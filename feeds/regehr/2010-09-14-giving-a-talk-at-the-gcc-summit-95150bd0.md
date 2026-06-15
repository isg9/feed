---
title: Giving a Talk at the GCC Summit
url: https://blog.regehr.org/archives/267
published: "2010-09-14T02:47:31Z"
feed: regehr
guid: http://blog.regehr.org/?p=267
---

# Giving a Talk at the GCC Summit

Last month I proposed giving a paper presentation at the next [GCC Developers’ Summit](https://www.gccsummit.org/2010/) about our compiler bug finding work. Happily, it was accepted and in late October I’ll head up to Ottawa to tell them what we’ve been doing and what we plan to do.  Here’s the abstract:

> About 60 wrong-code and crash bugs in GCC that we have reported have been fixed, including 21 that were categorized as P1. All of these bugs were found using differential random testing: generating a C program, compiling it using multiple compilers and/or optimization levels, and comparing the results. Although most of our tests have been run on x86 and x86-64 platforms, most of the bugs occurred in target-independent code.
>
> This talk will start out showing a few examples of bugs we have found. Next, I’ll talk about the main challenges in using random testing to find compiler bugs: avoiding C’s undefined behaviors and creating reduced test cases that can actually be reported. Third, I’ll present some results about the observed rates of crash and wrong-code errors in GCC 4.x releases. Finally, I’ll discuss some future directions including testing more language features and more targets.

I’m excited to give this talk since our work only matters if the people on the receiving end of our bug reports care about them. I’ll post more details from the event, or soon after.
