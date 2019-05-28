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
I tried to use

## Preserving file name
One of the biggest issues I had was to preserve file name and parse date and title from it. Hugo ignores all files
without title in frontmatter for me. I tried to configure it based on discussion, github and official documentation
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

## Code highlighting
Turn on class highlight.

## Directory structure
## Layout
