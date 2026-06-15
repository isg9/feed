---
title: Finding and Understanding Bugs in C Compilers
url: https://blog.regehr.org/archives/492
published: "2011-03-18T21:39:28Z"
feed: regehr
guid: http://blog.regehr.org/?p=492
---

# Finding and Understanding Bugs in C Compilers

Today we finished preparing the camera-ready version of our [paper that will appear in PLDI 2011](http://www.cs.utah.edu/~regehr/papers/pldi11-preprint.pdf). I’m pretty happy with it. Here’s the abstract:

> Compilers should be correct. To improve the quality of C compilers, we created Csmith, a randomized test-case generation tool, and spent three years using it to find compiler bugs. During this period we reported more than 325 previously unknown bugs to compiler developers. Every compiler we tested was found to crash and also to silently generate wrong code when presented with valid input. In this paper we present our compiler-testing tool and the results of our bug-hunting study. Our first contribution is to advance the state of the art in compiler testing. Unlike previous tools, Csmith generates programs that cover a large subset of C while avoiding the undefined and unspecified behaviors that would destroy its ability to automatically find wrong-code bugs. Our second contribution is a collection of qualitative and quantitative results about the bugs we have found in open-source C compilers.

Actually, as of today the total is 344 bugs reported but at some point we had to freeze the statistics in the paper.
