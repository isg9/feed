---
title: Fuzzing Linux Kernel Modules?
url: https://blog.regehr.org/archives/588
published: "2011-09-12T20:06:02Z"
feed: regehr
guid: http://blog.regehr.org/?p=588
---

# Fuzzing Linux Kernel Modules?

I’ve been thinking about what would be the best way to fuzz-test a Linux kernel module, for example a filesystem. Of course this can be done in the context of a live kernel, but for a variety of reasons I’d prefer to run the LKM in user space. At the source level, the interface to an LKM seems a little hairy, but at the object level they are really simple. So, a reasonable approach would seem to be to write a user-space loader for compiled LKMs and then just call the object code directly. At that point it would become necessary to write a set of shims to support each class of device driver and then fuzzing could start.

Anyway, I’m curious to see what people think about this idea before I go off and hack. I did some random web searching and didn’t turn up an existing implementation of this idea, though of course there are plenty of resources on testing various parts of Linux.
