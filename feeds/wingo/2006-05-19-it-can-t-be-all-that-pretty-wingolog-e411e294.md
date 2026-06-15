---
title: it can't be all that pretty — wingolog
url: https://wingolog.org/archives/2006/05/19/it-cant-be-all-that-pretty
published: "2006-05-19T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2006/05/19/it-cant-be-all-that-pretty
---

# it can't be all that pretty — wingolog

## [it can't be all that pretty](/archives/2006/05/19/it-cant-be-all-that-pretty)

19 May 2006 4:06 PM

- [america](/tags/america)
- [hacks](/tags/hacks)
- [photos](/tags/photos)
- [tagging](/tags/tagging)
- [profiling](/tags/profiling)

**Photos**

Went through and tagged the rest of my photos, resulting in an [enormous tag cloud](//wingolog.org/photos/). The thresholds can be tweaked but I think it's rather interesting.

As far as the software goes, recent changes: the bottom row of random thumbnails are only recent photos; fixed some url-encoding issues so all tag names should work now. I keep claiming I won't do any more web hacking but I imagine I'll implement view counts for photos.

**States**

Going home next Sunday for visa issues; hopefully will be returning Spain-side within a couple of weeks. However in the meantime I'm pretty excited about heading out to visit some folks in San Francisco. Sweetness.

**Hacks**

[Steel Bank Common Lisp](http://www.sbcl.org/) has an [excellent profiler](http://groups.google.com/group/comp.lang.lisp/msg/ed551830b6ba8697). I mentioned this [before](//wingolog.org/archives/2006/01/02/slime). It's statistical, driven by SIGPROF, so that the stack samples that it takes are evenly spaced in program execution time. That means that if you have more samples in one function, that your program spent more time there. A simple idea, used also by the statprof profiler I hacked on for guile and python, among many others.

Juho Snellman did a nice hack recently, making the profiler ticks driven by allocations instead of SIGPROF. The result is an [allocation profiler](http://jsnell.iki.fi/blog/archive/2006-05-14-statistical-allocation-profiler-for-sbcl.html) that will tell you which parts of your code are allocating memory the most. In garbage-collected languages this is often one of the best ways to microoptimize, as the cost of GC depends on the amount of allocation. Nice hack, Juho!

Meanwhile back at the bit ranch, holy shit! It seems that as early as 2003, a fellow named Alex Yakovlev wrote a Scheme interpreter in JavaScript with everything -- call/cc, tail recursion, multiple values, dynamic-wind, a syntax-rules based macro system, everything. Then recently Chris Double [ported it from IE6 to Firefox](http://www.bluishcoder.co.nz/2006/05/scheme-implementation-in-javascript.html), leaving us with a nifty [repl demo](http://www.bluishcoder.co.nz/jsscheme/), replete with an erlang-like [library for concurrency programming](http://www.bluishcoder.co.nz/2006/05/lightweight-threads-in-browser.html). Wow. Very neat. But oh, for a common VM.

*come back from san francisco, it can't be all that pretty*

## related articles

- [bread crumbs](/archives/2007/06/08/bread-crumbs)
- [tape ate the player](/archives/2006/05/26/tape-ate-the-player)
- [lost my wallet and feeling fine](/archives/2006/05/14/lost-my-wallet-and-feeling-fine)
- [canonical source](/archives/2005/01/08/canonical-source)
- [tracepoints: gnarly but worth it](/archives/2025/02/14/tracepoints-gnarly-but-worth-it)
- [a whiff of whiffle](/archives/2023/11/16/a-whiff-of-whiffle)

Comments are closed.
