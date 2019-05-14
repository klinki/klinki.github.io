---
published: true
title: What I have learned this week (06. - 12. 03. 2017)
date: "2017-03-08"
slug: what-i-have-learned-3
tags: [learning]
aliases:
    - /what-i-have-learned-3
---
## Properly commenting workarounds
Sometimes you need to do some workaround - maybe because of some bug, or lack of features in framework you use... There might be many reasons for it. It is necessary to properly document such a workaround and describe reasons why you used it.

It might seem obvious at the beginning, but when you return to that code after some time, you might not understand it rigt away. And maybe, if it is an ugly workaround, which it probably is, you might have a tendency to refactor it. Which is esentially good idea, but if workaround is properly documented (especially reason for it) it could give you hint how to do such a refactoring. You could for instance check, if the reason for it is still valid.


## Git clean -i
Sometimes in happens you have multiple unversioned files in your working directory and you want to get rid of them. But you don't want to delete all of them, only some. For that, there is a great `git clean -i` command. It allows you to filter out files, which you want to keep.
