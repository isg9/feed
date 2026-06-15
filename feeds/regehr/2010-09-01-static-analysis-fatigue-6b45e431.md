---
title: Static Analysis Fatigue
url: https://blog.regehr.org/archives/259
published: "2010-09-01T20:41:17Z"
feed: regehr
guid: http://blog.regehr.org/?p=259
---

# Static Analysis Fatigue

![](https://blog.regehr.org/wp-content/uploads/2010/09/bug.jpg)My student Peng and I have been submitting lots of bug reports to maintainers of open source software packages. These bugs were found using [Peng’s integer undefined behavior detector](https://blog.regehr.org/archives/226). We’ve found problems in OpenSSL, BIND, Perl, Python, PHP, GMP, GCC, and many others.

As we reported these bugs, I noticed developers doing something funny: in many cases, their first reaction was something like:

> Please stop bothering us with these stupid static analysis results!

They said this despite the fact that in the initial bug report, I usually pointed out that not only were these results from actual tests, but they were from the tests provided by the developers themselves!  [This interaction with PHP’s main developer](http://bugs.php.net/bug.php?id=52550) is a good example.

What can we learn from this?  I take away a few different messages:

1. From the developer’s point of view, static analysis results can suck. As these tools become smarter, they consider more and longer code paths. In a very real sense, this makes their results more difficult to reason about. It should go without saying that bug reports that are hard to reason about are not very actionable.
2. The developers of popular open-source software projects are suffering from static analysis fatigue: a syndrome brought on by seeing too many bug reports from too many random tools, too few of which are actionable. If I made a living or a career out of static analysis, I’d be seriously worried about this.
3. Many developers are still in the dark about C’s integer undefined behaviors. This is a constant surprise to me given that GCC and other common compilers have evaluated “(x+1)>x” to “true” for quite a while now.
4. The best bug reports, by far, are those that are accompanied by a failure-inducing input. I think the reason that we’ve been running into developer confusion is that we’re telling people that their own “make check” is a failure-inducing input, which people don’t want to hear since when they run it, everything looks fine. Some of these problems will go away when our undefined behavior checker makes it into an LLVM release. Lacking the checker people can still reproduce our results, but only by manually adding assertions.
5. Developers are prideful about their projects. This is as it should be, unless it blinds them to useful bug reports.

Regarding static analysis fatigue, there can be no silver bullet. People developing these tools must take false-positive elimination very seriously (and certainly some of them have). My preference for the time being is to simply not work on static analysis for bug-finding, but rather to work on the testing side. It is more satisfying and productive in many ways.
