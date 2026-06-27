---
title: Three kinds of memory leaks
url: https://blog.nelhage.com/post/three-kinds-of-leaks/
published: "2018-04-29T15:30:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/three-kinds-of-leaks/
---

# Three kinds of memory leaks

So, you’ve got a program that’s using more and more over time as it runs. Probably you can immediately identify this as a likely symptom of a memory leak. But when we say “memory leak”, what do we actually mean? In my experience, apparent memory leaks divide into three broad categories, each with somewhat different behavior, and requiring distinct tools and approaches to debug. This post aims to describe these classes, and provide tools and techniques for figuring out both which class you’re dealing with, and how to find the leak.
