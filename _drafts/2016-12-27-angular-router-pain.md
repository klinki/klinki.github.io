---
published: false
---

In this article, I would like to focus on several problems I had with Angular Router component. One could say I criticize Router because it doesn't fit to my use case. Of course, Angular developers are creating Angular with some use cases in mind. It is not so generic common use framework and it doesn't fit all use cases at all. Angular Router is great example of that - it is barely as universal as one might think.

But part of problem I have with it, is it hasn't been properly communicated out how to properly use Router. More I use it, more problems I have with it and I even started to think there is no proper way how to use it :(

### Hierarchical routes
Hierarchical routes is very interesting aspect of Angular router. It allows you to define children of routes and create routes hiearchy. I've been very excited about that and started using it quite a lot almost immediatelly after I learned about it.

At first, I created top level route with `LoggedIn` `Guard` which checks if user is logged in and can go to given route. Then I started using something what I call layout routes.

Beware - params and data are not inherited. At least not always. There are exceptions for that. They are inherited in one of those cases: 

- parent route is componentless
- parent route is empty

Unfortunately those limitations were too restrictive for my use case and I had to use some workaround. After trying out several things, I ended up injecting `ActivatedRoute` and `Router` into basically all of my components. I listened for `NavigationEnd` event of `Router` and when it occured, I used `ActivatedRoute.snapshot` and passed it to my service which then goes through all parent and child routes and creates one flat route with all parameters and data. This is very ugly solution but it works. And that is most important factor for me right now.

But I'm starting to wory about reduced testability of such components. Mocking `ActivatedRoute.data` (or params) is now not enough anymore, I have to mock also `Router.events`. And I'm starting to feel I'm breaking SOLID principle. Components are not behaving like black box anymore - they use data from routes without explicitly saying that. It feels almost like service locator pattern (which is being considered as anti-pattern and is preferred to be replaced with constructor or attribute DI).


#### Changing route to componentless
When parent is componentless (or has empty path) child routes automatically inherit, you cannot prevent it. You have to deal with it. IMHO current situation with hierarchical routes is so bad, it makes me think it would be better if routes NEVER inherited params and data. It would be at least consistent, if not anything else at all. Now I ended up with having my custom written workaround which works perfectly fine, when data are not inherited and I need them to be, but when they suddenly do inherit, it is problem to prevent that.


### Reducing hierarchical routes boilerplate, test havoc
I came with idea to reduce boilerplate with resolving hierarchical routes. I added most of code into `Application` component and don't depend on much of routing in other components. But it made test havoc - my prepared code for mocking routes could not be used in a same manner anymore. And I had to make brand new workarounds for tests.

Sure, we might get architecture of our application wrong, but working with Angular router is huge pain in the a**. 

