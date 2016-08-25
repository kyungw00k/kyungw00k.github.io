---
layout: post
title: git bisect
date: 2016-01-20 09:42:35.000000000 +09:00
type: post
categories:
- git
---

https://robots.thoughtbot.com/git-bisect

```sh
# get it ready
git bisect start
git bisect good c09c728
git bisect bad e6a0692

# give git a command to run against each commit
git bisect run command.sh
```
