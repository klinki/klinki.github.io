---
published: false
---
In this article I would like to make some very high level comparison of Aurelia and Angular. I have limited experience with Aurelia,
 so I won't go into deep details, I will focus on the basic things I already used.

## Much less boilerplate
I have noticed Aurelia doesn't need as much boilerplate as Angular does. For example component in Aurelia could look like this:

```javascript
export class AureliaComponent {
   @bindable property = 'Hello World';
}
```

On the other hand, component in Angular looks like this:

```javascript
@Component({
    templateUrl: './angular-component.html',
    selector: 'app-angular-component'
})
export class AngularComponent {
   property = 'Hello World';
}
```
In Aurelia you don't have to use any `@Component` annotation and you don't have to specify path to template file. It is nice and short.
Of course you could specify template path if you want to, but it nicely defaults to file with the same name as component, just with `.html` extension.

In Angular you typically have to register component into some module. In Aurelia you don't have to, you can use it with `<require>` tag. But you can register it as global and then you don't have to repeat `<require>` everywhere. Aurelia provides more flexibility here.


### Services
In Angular, you have to register all services to module providers. In Aurelia, you don't. You just simply use them and use `@autoinject()` annotation which tells DI container to automatically resolve dependencies for you. Angular has similar annotation `@Injectable()` but requires registering services in module.


### Templating
Aurelia is using simpler syntax in templates. For printing output, it uses standard `${expression}` syntax. Angular uses it's own (but quite common in some other frameworks too) syntax `{{expression}}`.

For data and event binding, Aurelia uses `data.bind` which is intelligently resolved as two way binding for form inputs, or one way binding in other cases. It also provides `data.one-way` and `data.two-way` bindings. For events it uses `event.delegate` or `event.trigger`. For if conditions, it uses `if.bind`, for loops `for.repeat`. Quite simple pattern and easy to get used to.

Angular on the other hand came with its own syntax. One way binding is `[data]`. Two way binding uses bannana in the box syntax `[(data)]`. Event binding uses `(event)`. These are 3 different syntax expressions just for data and event binding. For conditions, it uses `*ngIf`. For loops `*ngFor`. Remember the asterisk `*` it is important part of the expression! (I don't really know why - well, documentation explains what it does, but I don't understand why Angular team decided to use this syntax). But if you want to use it on `<template>` tag, then it is different - For if statement on `<template>` you have to use `<template [ngIf]="">`. For loop condition, it is even more cucumbersome: `<template ngFor let-variable [ngForOf]="array">`. Readability of that is very bad.


Another thing is when you want to modify attributes. In Angular you can use either `attr="{{experession}}"` inside attribute, or `[attr.attribute]="expression"`. In Aurelia, you simply use `attr="${expression}"`.


### Aurelia pure functions
There was one thing in Aurelia which surprised me and took me some time to find out. Functions called from templates are IMHO something like pure functions and they reevaluate only when input changes. But what if you use some component instance variable inside? Nope, they don't reevaluate until you pass the variable as input parameter.

This surprised me a lot, because in Angular it behaves completely different - Angular would reevaluate such a function again, when component property has changed.

It was surprising for me, but I think I understand the reason and I agree with it. Functions are pure and without side effects. For the same input, they should give the same output. And they don't depend on some hidden internal state.


### Conclusion
I just scratched a surface. I don't really have enough experience with Aurelia to write deeper comparison. I might try to implement tour of heroes in it though, just for fun and comparison :)
