---
published: false
---
## Angular Router

In this article, I would like to focus on several problems I had with Angular Router component. Even though this article is criticism. Angular developers are creating Angular with some use cases in mind and it is not so generic common use framework as we might think. Angular Router is just not as universal as one might think.

### Hierarchical routes
Hierarchical routes is very interesting aspect of Angular router. It allows you to define children of routes and create routes hiearchy. I've been very excited about that and started using it quite a lot almost immediatelly after I learned about it. 

At first, I created top level route with LoggedIn guard which checks if user is logged in and can go to given route. Then I started using something what I call layout routes.

Beware - params and data are not inherited. At least not always. There are exceptions for that. They are inherited in one of those cases: 

- parent route is componentless
- parent route is empty

Unfortunately those limitations were too restrictie for my use case and I had to use some workaround. After trying out several things, I ended up injecting ActivatedRoute and Router into basically all of my components. I listened for NavigationEnd event of Router and when it occured, I used ActivatedRoute.snapshot and passed it to my service which then goes through all parent and child routes and creates one flat route with all parameters and data. This is very ugly solution but it works. And that is most important factor for me right now.

But I'm starting to wory about reduced testability of such components. Mocking ActivatedRoute.data (or params) is now not enough anymore, I have to mock also Router.events. And I'm starting to feel I'm breaking SOLID principle. Components are not behaving like black box anymore - they use data from routes without explicitly saying that. It feels almost like service locator pattern (which is being considered as anti pattern and is preferred to be replaced with constructor or attribute DI).


#### Changing route to componentless
Child routes automatically inherit, you cannot prevent it. You have to deal with it. IMHO current situation with hierarchical routes is so bad, it makes me think it would be better if routes NEVER inherited params and data. It would be at least consistent, if not anything else at all. Now I ended up with having my custom written workaround which works perfectly fine, when data are not inherited and I need them to be, but when they suddenly do inherit, it is problem to prevent that.


### Reducing hierarchical routes boilerplate, test havoc
I came with idea to reduce boilerplate with resolving hierarchical routes. I added most of it into Application component and don't depend on much of routing in other components. But it made test havoc - my prepared code for mocking routes could not be used in a same manner anymore.


Sure, we might get architecture of our application wrong, but working with Angular router is huge pain in the a**. 

