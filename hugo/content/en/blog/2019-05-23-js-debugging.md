---
layout: post
title: Chrome JavaScript debugging
date: "2019-05-23"
slug: chrome-js-debugging
---

## Ignoring external libraries

Chrome debugger allows you to ignore library code and do the stepping only on your custom code.

To activate that feature, do the following:

1. Open dev tools
1. Click on three dots menu on the right side
1. Select **Settings** from the menu and find **Blackboxing**
1. Check the **Blackbox content scripts** checkbox and add a pattern using **Add pattern** button.
(I usually use that with `node_modules` as a pattern).

This is very useful when debugging Angular application with lot of internal calls to Angular itself, Zone.js and RxJS. You can skip all these internal calls. It also allows you to simply debug RxJS Observable subscriptions.

I found this information on [Markus Edenhauser's blog](https://blog.edenhauser.com/tell-chrome-debugger-to-ignore-libraries/), thank you!

## Monitoring events

In Chrome there is also very easy way to monitor all DOM events. You can use built-in function `monitorEvents(object[, events])`.

```javascript
monitorEvents(document.body) // monitor body events
monitorEvents($0, 'focus') // monitor inspected element focus events
monitorEvents(document.body.querySelectorAll('input')) // monitor all events on all inputs on the page
```

To stop monitoring events, use `umonitorEvents`.
Thanks [sidonaldson on StackOverflow](https://stackoverflow.com/questions/7439570/how-do-you-log-all-events-fired-by-an-element-in-jquery/18850523#18850523) for this.

## Overriding native JS functions

Several times I had to debug code calling native JS functions. Mostly `scroll` and `focus`. I needed a way to place breakpoint into or right before the function call. To do that, I override native functions with my own adapter. It simplifies debugging, because I can put a breakpoint into adapter and see a stacktrace and find out from where it has been called.

Here is an example how to override such a function:

```typescript
function monkeyPatchObjectFunction(object: any, functionName: string) {
    const originalFunction = object[functionName];
    object[functionName] = function() {
      console.log(`Called function: ${functionName} with args: ${arguments}`);
      return originalFunction.apply(this, arguments);
    };
  }
```

```javascript
monkeyPatchObjectFunction(Element.prototype, 'focus');
monkeyPatchObjectFunction(window, 'scroll');
```

## $ shortcuts

DOM related shortcuts:

- `$0`...`$4` reference to last selected element from inspection panel.   
`$0` is currently selected, `$1` is selected before and so on... In Firefox only the most recent one (`$0`) works.
- `$(selector, [startNode])` is an alias for `document.querySelector()`.  
Selects single node with specified CSS selector (example: `$('#main-heading')`)
- `$$(selector, [startNode])` is an alias for `document.querySelectorAll()`.  
Selects all nodes with specified CSS selector (example: `$$('.sub-heading')`).
- `$x(path, [startNode])` allows you to select DOM elements by XPath expression.  
Example: `$x('//img[@alt]')` selects all images with `alt` attribute.

Not DOM related:

- `$_` last evaluated expression value.  
    If you type in for example `2+2` it evaluates as `4` and saves the result into `$_` variable.

## Other useful debugging functions

- `copy(object)` copies a string representation of the specified object into clipboard.
- `debug(function)` triggers debugger when function is called
- `monitor(function)` logs function calls with parameters into console

More information is available on [Google Chrome Devtools page](https://developers.google.com/web/tools/chrome-devtools/console/utilities).

## Sources

- <https://blog.edenhauser.com/tell-chrome-debugger-to-ignore-libraries/>
- <https://stackoverflow.com/questions/7439570/how-do-you-log-all-events-fired-by-an-element-in-jquery/18850523#18850523>
- <https://developers.google.com/web/tools/chrome-devtools/console/utilities>
