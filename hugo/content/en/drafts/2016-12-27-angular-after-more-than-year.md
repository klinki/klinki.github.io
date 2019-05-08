---
published: false
---
## Angular 2 after more than 1 year

It is more than year, since I started using Angular 2. Before I've been using Angular 1 in my job and I had quite a lot of experience with it and I also new about some of its problems (mainly performance related). So I was super excited about Angular 2 when I started. I was also very happy about Angular 2 being developed in TypeScript, since I'm huge fan of it.

When I started with Angular 2, in that time still in beta phase, there was little documentation. I had to google all the time and filter out some old and obsolete articles, since lot of things have been changing quite a lot. I started using Angular for my diploma thesis. I used only several things of the framework - mainly components, services and DI. I didn't use a Router at that time. The most worrisome thing for me was bundling. I spent tens of hours trying to figure out how to bundle my application and serve it in as few files as possible. Unfortunately I never managed to get the expected result, so I ended up using just TypeScript compiler with SystemJS module loader and hundreds of files being loaded.

Now, after one year, I had to face similar problem in my current job. We had existing application and we wanted to get bundling into one file in some reasonable time. And again, I hit very similar problem. We experimented with Angular CLI, but we decided not to use it, since we believed it wasn't mature enough. (Thank God we didn't use it, there has been some problems especially with version 22). Again, I spent tens of hours trying to get something running. This time, I managed to get webpack 2 up and running. But it wasn't pleasant experience. Again, I was facing not having enough documentation, endless googling and filtering out obsolete articles, because in JavaScript world, everything what is older than 1 moth, is outdated already.


Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.
