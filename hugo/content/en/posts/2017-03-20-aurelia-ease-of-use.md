---
published: true
tags: [javascript, typescript]
---
On my last project I had a chance to work with Aurelia. I needed to use some frontend framework capable of being easily used with existing server rendered application. This single requirement automatically crossed out Angular, so I had to look for some alternative. Since I knew little bit about Aurelia and I found out it is easily embeddable ito existing app, I decided to use it.

I think it was right decision and it has been quite pleasant to work with Aurelia.

### CLI
I started with downlading Aurelia CLI tool. I read about it in [Early March Mega Release article](http://blog.aurelia.io/2017/03/07/early-march-mega-release/) and I thought I will give it a try. It was quite easy to use. Creating new project was as simple as `au new`. I specified name of project, language and it was ready to go! Great!  

### First component
I have never worked with aurelia before, but I have quite a lot of experience with Angular, so I believed it won't be a probem. And I was right! I created new component using `au generate element`. Calling components `element` was little bit confusing for me, but it is just a name. Not a big deal. I generated component, looked at it and it was quite obvious how to use it.
Only thing I had to get used to is to enclose all template HTML into `<template>` tag. No problem at all.

Then I played with `<require>` little bit and found out way how to avoid need for it - registering components in configuration file as [global resources](http://aurelia.io/hub.html#/doc/article/aurelia/templating/latest/templating-html-behaviors-introduction/5), similarily as it is done in angular. You don't have to do it if you don't want to - you can stick to `<require>` tag. But you can, if you want to get rid of `<require>` boilerplate. Big + for Aurelia.

### Embedding into existing application
Using Aurelia with existing server rendered application is quite easy. I quickly went through [documentation](http://aurelia.io/hub.html#/doc/article/aurelia/framework/latest/app-configuration-and-startup/8) and I was able to get it running in almost no time. This is great! Really great! Applause for Aurelia!

### Documentation
I already mentioned documentation several times. To be honest, I don't like Aurelia's documentation much - it is really very very complex and it wasn't easy for me to orient in it. (At least if you want to read it chapter by chapter). It feels too complex for me. Also I was struggling with selecting the right role :) Instead of reading it chapter by chapter, I used search to find things I needed. Search is really very helpful and it can help you to quicky find what you need. 

But I didn't really read much of the documentation. I just wrote the code and when I needed something, I used search. The biggest surprise for me was how much I was able to code without really needing the documentation. I just wrote TypeScript code and some HTML. Then I quickly searched in documentation how to bind variables to elements, how to use events, how to write if statements and loops and it was basically it.

I got working Aurelia application with only about 20 minutes spent in documentation! That is AWESOME!

### Installing libraries
Another think I really enjoyed was an ease of installing new plugins. I needed to use Aurelia HTTP client. So I used `au install aurelia-http-client` and it installed and configured everything for me. It even showed me several ways how to import the library in TypeScript! I was impressed once again.


### Overall impression
Experience with using Aurelia was really great for me. I felt very productive and I find Aurelia very convenient. Even though I have about 1 year experience with Angular, I felt more productive with Aurelia right away - and it was the first time I used that framework! I think Rob Eisenberg got it right and did really great job on Aurelia. I can definitely recommend Aurelia - even to people who are new to frontend development. 
