---
title: Csmith Released
url: https://blog.regehr.org/archives/510
published: "2011-04-13T02:36:06Z"
feed: regehr
guid: http://blog.regehr.org/?p=510
---

# Csmith Released

[Here is Csmith](http://embed.cs.utah.edu/csmith/), our randomized C program generator. My dream is that it will be a force for good by unleashing a world of hurt upon low-quality C compilers everywhere (it is not uncommon for Csmith to crash a previously-untested tool on the very first try). High-quality C compilers, such as the latest versions of GCC and Clang, will correctly translate many thousands of random programs.

I wanted to release Csmith under the [CRAPL](http://matt.might.net/articles/crapl/), but nobody else in my group thought this was as funny as I do. Actually we put quite a bit of work into creating a stable, usable release. Even so, Csmith is a complex piece of software and there are certainly some lurking problems that we’ll have to fix in subsequent releases.
