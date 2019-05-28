---
published: true
title: Migrating to Hugo
slug: migrating-to-hugo
date: "2019-05-13"
---
I decided to migrate my blog tu Hugo. One of the main reasons was to support multiple languages, since I decided
I want to write also in Czech from time to time.

It hasn't been exactly easy transition as I hoped for, but I will describe what I did and how it worked out.

## Conversion tool

I tried to use scripts mentioned on the [Hugo official documentation](https://gohugo.io/tools/migrations/#jekyll), but wasn't much successful with them. I also tried `hugo import jekyll`. This one had better results. I had to modify the blog posts little bit anyway.

## Preserving file name

One of the biggest issues I had was to preserve file name and parse date and title from it. Hugo ignores all files
without title in frontmatter for me. I tried to configure it based on discussions, github and official documentation
but didn't succeed.

Posts must have title, slug and date filled in, it cannot be parsed from a filename. What a pitty :(

## BaseUrl variables in markdown files

In jekyll, I had `{{baseUrl}}` variables in markdown files. I needed to get these to Hugo. It was quite a problem,
so at the end I deleted the variables and let the browser handle URLs starting with `/` on its own. It worked quite well.

## URL aliases

There was actually one thing I was very happy about. It was very simple to configure URL aliases, so old links wouldn't break.
It was as easy as adding following code to frontmatter:

```yaml
aliases:
    - my-cool-url-alias
```

## Picking up a template

Surprisingly, another big issue for me was picking a template. I had just one simple requirement: I needed it to work.
The less configuration it require, the better. And I don't even need anything special, for the beginning, I would be very
happy with pretty much same template as I used in jekyll. I've found [hugo-now](https://github.com/mikeblum/hugo-now)
template, but it wasn't looking good. Home page didn't show anything, links didn't work and the source code style
was totally broken.

I jumped over quite a lot of other templates with more or less success after I ended up using current [Mainroad theme](https://github.com/Vimux/Mainroad/).

## Code highlighting

I still have issues with turning on code higlighting, so this is one of the areas I need to work on.

## Deployment

There is another major change after migration to Hugo. It requires deployemt. Previosly, jekyll was natively supported by Github pages.
Now, hugo is not, so I had to set up travis-ci job to build it and deploy new changes. Luckily there are quite few articles how to do that. I used similar approach as is described on this blog post: <https://www.sidorenko.io/post/2018/12/hugo-on-github-pages-with-travis-ci/>.
