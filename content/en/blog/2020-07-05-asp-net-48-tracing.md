---
layout: post
title: ASP.NET 4.8 Tracing
date: "2020-07-05"
slug: asp-net-48-tracing
---

In this longer blog post, I would like to explain contextual logging and request tracing in ASP.NET Core and old ASP.NET. In my current job, I maintain several ASP.NET 4.8 applications and some new ASP.NET Core ones. I needed to properly set up logging, but I couldn't find much information how to get request tracing working in old ASP.NET 4.8. I hope this article might help someone with the same problem.

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

## Why to do request tracing and contextual logging

In these examples, I will use logging library [Serilog](https://github.com/serilog/serilog) and [datalust Seq](https://datalust.co/) for centralized log management. I use both every day and I must say I'm verry happy with them, I think it os one of the best logging infrastructure available for .NET log management.

Most of information in this article is generic and apply to any logging framework which captures the contextual information.

## Serilog configuration

In our initial log configuration, we get only the basic request information logged.

![Default Serilog logging ASP.NET Core](/images/aspnet-tracing/00-netcore-without-context.png)

Also, if we don't change log levels for Microsoft loggers, lot of log messages will be logged for each request.

![Default Serilog logging ASP.NET Core](/images/aspnet-tracing/00-netcore-lot-of-clutter.png)

Let's switch log levels for Microsoft loggers:

```json
"MinimumLevel": {
    "Default": "Warning",
    "Override": {
        "Serilog": "Information",
        "System": "Warning",
        "Microsoft": "Warning",
        "Microsoft.EntityFrameworkCore": "Warning"
    }
},
```

From now on, there will be just single log message for each request. So much cleaner.

![Default Serilog logging ASP.NET Core](/images/aspnet-tracing/00-request-clutter-reduced-to-min.png)

And whad do we get in logs for subsequent requests?

![Default Serilog logging - Subsequent requests](/images/aspnet-tracing/00-subsequent-requests.png)

Not much, there is no information connecting these requests together.

Let's turn on `ContextEnricher` by adding following part into `Serilog` configuration in `appsettings.json`:

```json
"Enrich": [
    "FromLogContext"
],
```

Now, lets look at Seq output.

![Default Serilog logging - Subsequent requests](/images/aspnet-tracing/01-with-trace-context.png)

We got new properties - `RequestId`, `SpanId`, `TraceId`, `ParentId`.

So what are these?

- `ParentId` - identifies parent request. Empty for initial request.
- `SpanId` and `RequestId` - are both IDs of current request, just in different format.
- `TraceId` - this is probably the most important information. It identifies the chain of requests. As you can see, it is the same for all subsequent requests.


### How is trace context propagated

Let's look on headers which are sent in ASP.NET Core. In this example, we will use
Initial request is made from browser, so server gets the `User-Agent` and all the usual headers that browser sends.

```json
"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:78.0) Gecko/20100101 Firefox/78.0",
...
```

Second request is made from ASP.NET Core server again to ASP.NET Core server.
We can see `Request-Id` header being sent.

```json
"Host": "localhost:5101",
"Request-Id": "|5b714529-4035a02d51ef211e.1."
```

And third request is made again from ASP.NET Core to ASP.NET Core. Again, `Request-Id` is sent and we can see it's hierarchical format - it's first part is the same up to the second dot. Then there is `5b71452a_1.` appended.

```json
"Host": "localhost:5101",
"Request-Id": "|5b714529-4035a02d51ef211e.1.5b71452a_1."
```

If we do the same for full framework app, we don't get the `Request-Id` header.

## Adding tracing for ASP.NET Framework app

I did some digging to see how does ASP.NET Core handle passing the proper values to initialize activity context to see if I could reproduce it in old ASP.NET Framework 4.8.

On the client side, .NET Core automatically sends `Request-Id` header in HTTP requests from `HttpClient`. This is set in `DiagnosticsHandler` (more details in [dotnet/corefx repo](https://github.com/dotnet/corefx/blob/v3.1.5/src/System.Net.Http/src/System/Net/Http/DiagnosticsHandler.cs#L206)).

On the server side it automatically reads that value and properly initializes `Activity` in `HostingApplicationDiagnostics` (again, more details in [dotnet/aspnetcore repo](https://github.com/dotnet/aspnetcore/blob/v3.1.5/src/Hosting/Hosting/src/Internal/HostingApplicationDiagnostics.cs#L259)).

Luckily we can do similar steps manually in ASP.NET 4.8 application.

1. First step is to install `System.Diagnostics.DiagnosticSource` nugget package.

2. Create `ActivityManager` class

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

3. Configure `Global.asax.cs` handler

    ```csharp
    protected void Application_Start()
    {
        Activity.DefaultIdFormat = ActivityIdFormat.Hierarchical;
    }

    protected void Application_BeginRequest()
    {
        ActivityManager.StartActivity();
    }

    protected void Application_EndRequest()
    {
        ActivityManager.StopActivity();
    }
    ```

4. And for logging, create new Serilog enricher:

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

    this log enricher must be registered to serilog (for example with `.Enrich.With<TraceLogEnricher>()`).

Now, our application handles the incomming requests side and properly logs the tracing properties.

## Outgoing requests

Now to handle outgoing requests, we need to fill in `Request-Id` header for all HTTP requests comming from our application.

### Http Requests

There are many possible ways how to achieve that, we could for example use `DefaultRequestHeaders` property on `HttpClient`. Or we could use and configure `HttpClientFactory`. This is preferred method, so we will use that.

### WebService requests

We use also `.asmx` web services in our applications. Luckily, these can be easily modified to also include tracing.
Most of the generated services are created as partial classes. They have `GetWebRequest` method, which creates the WebRequest
 which is used to send request to web service. We can easily modify `GetWebRequest` method to fill in `Request-Id` headers.

```csharp
namespace RequestTracing.GeneratedWebServiceClient
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
