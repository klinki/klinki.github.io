---
layout: post
title: ASP.NET 4.8 Tracing
date: "2020-07-05"
slug: asp-net-48-tracing
---

Outline:

1. Problem statement
2. Serilog configuration
3. Context enrichment
4. Showing results for chained requests:
    a. Core -> Core
    b. Fw -> Fw
    c. Fw -> Core
    d. Core -> Fw
5. Adding Activtity and passing Request-Id
6. Showing results again


In my current job, I maintain several ASP.NET 4.8 applications and some new ASP.NET Core ones.

I use [datalust Seq](https://datalust.co/) for centralized log management. I must say I'm verry happy with this log management tool
 and I think it os one of the best tools available for log management. As logging library I use Serilog.

`TraceId`, `SpanId`, `ParentId`

These 3 properties are super useful for debugging failed requests. `TraceId` remans the same for all related subsequent requests in the system.

Install `System.Diagnostics.DiagnosticSource` nugget package.

```csharp
```

I did some digging to see how does ASP.NET Core handle passing the proper values to initialize activity context.

On the client side, ASP.NET Core automatically sends `Request-Id` header in HTTP requests from `HttpClient`.
On the server side it automatically reads that value and properly initializes Activity.

```csharp
protected void Application_Start()
{
    Activity.DefaultIdFormat = ActivityIdFormat.Hierarchical;
}
```

```csharp
public static class ActivityManager
{
    public static void StartActivity()
    {
        if (Activity.Current == null)
        {
            var activity = new Activity("Default Activity");

            string parentIdFromHeaders = HttpContext.Current?.Request.Headers[GetRequestIdHeaderName()];
            if (!string.IsNullOrEmpty(parentIdFromHeaders))
            {
                activity.SetParentId(parentIdFromHeaders);
            }

            activity.Start();
            Activity.Current = activity;

            // Sometimes I had issues with Activity.Current being empty even though I set it
            // So just to be sure, I add it also to HttpContext Items.
            HttpContext.Current?.Items.Add("Activity", activity);
        }
    }

    public static void StopActivity()
    {
        GetActivity()?.Stop();
    }

    public static Activity GetActivity()
    {
        Activity activity = Activity.Current ?? (Activity)HttpContext.Current.Items["Activity"];
        return activity;
    }

    public static string GetRequestIdHeaderName()
    {
        return "Request-Id";
    }

    public static string GetRequestId()
    {
        Activity activity = GetActivity();

        if (activity != null)
        {
            string activityId = activity.Id;
            return activityId;
        }

        // For the rare cases when something happens and activity is not set
        // Try to read Request-Id first, if none, then create new GUID
        return HttpContext.Current?.Request.Headers.Get(GetRequestIdHeaderName())
                ?? Guid.NewGuid().ToString().Replace("-", "");
    }
}
```

```csharp
protected void Application_BeginRequest()
{
    ActivityManager.StartActivity();
}

protected void Application_EndRequest()
{
    ActivityManager.StopActivity();
}
```

and for logging, I created new Serilog enricher:

```csharp
public class TraceLogEnricher : ILogEventEnricher
{
    public void Enrich(LogEvent logEvent, ILogEventPropertyFactory propertyFactory)
    {
        Activity activity = ActivityManager.GetActivity();

        if (activity != null)
        {
            string parentId = activity.ParentId;
            string rootId = activity.RootId;
            string activityId = activity.Id;

            logEvent.AddPropertyIfAbsent(new LogEventProperty("SpanId", new ScalarValue(activityId)));
            logEvent.AddPropertyIfAbsent(new LogEventProperty("ParentId", new ScalarValue(parentId)));
            logEvent.AddPropertyIfAbsent(new LogEventProperty("TraceId", new ScalarValue(rootId)));           }
    }
}
```

this log enricher must be registered `.Enrich.With<TraceLogEnricher>()`.

This handles the incomming requests side na properly logs the tracing properties.

## Outgoing requests

Now to handle outgoing requests, we need to fill in `Request-Id` header for all HTTP requests comming from our application.

### Http Requests

There are many possible ways how to achieve that, you could for example use `DefaultRequestHeaders` property on `HttpClient`.
Or you could use `HttpClientFactory`

### WebService requests

We use also `.asmx` web services in our applications. Luckily, these can be easily modified to also include tracing.
Most of the generated services are created as partial classes. They have `GetWebRequest` method, which creates the WebRequest
 which is used to send request to web service. We can easily modify `GetWebRequest` method to fill in `Request-Id` headers.

```csharp
namespace RequestTracing.com.emclient.licensing
{
    public partial class Service
    {
        protected override WebRequest GetWebRequest(Uri uri)
        {
            var request = base.GetWebRequest(uri);

            request.Headers.Add(ActivityManager.GetRequestIdHeaderName(), ActivityManager.GetRequestId());

            return request;
        }
    }
```

and voila, now our web services support tracing!

## W3C Trace Context

All examples here have been using `ActivityIdFormat.Hierarchical` activity format. There is new [W3C Trace Context](https://www.w3.org/TR/trace-context/) standard with new format. It can be easily switched by setting `Activity.DefaultIdFormat = ActivityIdFormat.W3C`
 (which will soon be default for .NET Core).

To properly levarage W3C format, there should be used multiple tracing headers, `traceparent` and `tracestate`.
`traceparent` header has following format: `version-format "-" trace-id "-" parent-id "-" trace-flags`.

`StartActivity` and `GetRequestId`