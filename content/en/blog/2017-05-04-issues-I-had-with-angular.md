---
published: false
title: Issues I had with Angular
---

This article is just list of issues I had with Angular. I will keep updating it when I discover some new issue.

## Problems I encountered during development (bugs or designed behavior)

- Route data are not hierarchically resolved and it cannot be enforced
- Route data are sometimes hierarchically resolved (when route is componentless/empty) and it cannot be prevented
- `<ng-content>` cannot be properly used in container which wraps it into `*ngIf`
- `[routeLink]` cannot be properly used with `<ng-content>` projected component

## Other problems

- By using `Router` you usually end up with very tight coupling of components to `ActivatedRoute` and `Router`
- there is no proper documentation how to test components with `OnPush` change detection
- there are quite a lot of issues with AOT and it took very long time until someone documented requirements for it (and it was done by someone outside Angular team)
- i18n cannot be currently used outside of template
- there is no proper documentation for quite a lot of things
  - proper usage of Router (notes about hierarchical routes..)
  - proper use of `<ng-content>`
  - lazy loading modules requires [angular2-route-loader](https://github.com/angular/angular.io/issues/2801) webpack loader (not documented)
- quite a lot of reported bugs are considered to be features or designed behavior by Angular team
- Angular team is not very good in explaining things (especially design decisions, which are not documented)
