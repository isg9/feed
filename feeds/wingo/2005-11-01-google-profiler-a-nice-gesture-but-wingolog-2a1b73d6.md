---
title: Google profiler a nice gesture but. — wingolog
url: https://wingolog.org/archives/2005/11/01/google-profiler-a-nice-gesture-but
published: "2005-11-01T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/11/01/google-profiler-a-nice-gesture-but
---

# Google profiler a nice gesture but. — wingolog

## [Google profiler a nice gesture but.](/archives/2005/11/01/google-profiler-a-nice-gesture-but)

1 November 2005 4:28 PM

- [hacks](/tags/hacks)

So Google released a set of [performance-measuring tools](http://goog-perftools.sf.net) as free software. There's a really fast allocator, a heap usage profiler, a cpu profiler, and some other goodies.

The CPU profiler is touted as being able to correctly profile multithreaded applications, which sounds excellent. Unfortunately it seems to be only for LinuxThreads, the threading library used with pre-2.6 kernels, which is really old. Because threads had different PID's under LinuxThreads, it was easy to do per-thread profiling via setitimer(2). Each thread would get its own timer signal because it was seen as a separate process (I think -- could be wrong here).

However as it is the google profiler is mostly useless for profiling multithreaded applications on modern systems. Furthermore I couldn't get it to work at all on my x86-64 machine, either via LD\_PRELOADing the library or via real linking -- it caused segmentation faults, and the processing script complained about my 8-byte pointers.

There are more gotchas with the other tools. All of the voodoo is implemented for i386, some of it for x86-64, a tiny bit for ARM\_3, but nothing else. They don't know which direction a ChangeLog is supposed to go, and there's no CVS.

I feel like there must be a funny tension at Google. On the one side there is the desire of the hackers there to be professionals, sharing information with their fellow hackers for the advancement of their field. On the other hand there is a rigid information flow policy whereby the brains that Google bought (converse: brains that sold themselves?) contain knowledge that is proprietary to the company, which the company does not want shared. Surely the professional desire is fulfilled somewhat via internal recognition, but there is more to be desired. Hence the perftools code drop, although still a gesture on Google's own terms.

My reaction at this point is twofold, one that I wish that Google were investing in the modern toolchain instead, and two that I am a lucky fellow to hack free software for a living. It would suck to code from within a fortress.

## related articles

- [bread crumbs](/archives/2007/06/08/bread-crumbs)
- [steal our code](/archives/2007/05/03/steal-our-code)
- [tape ate the player](/archives/2006/05/26/tape-ate-the-player)
- [it can't be all that pretty](/archives/2006/05/19/it-cant-be-all-that-pretty)
- [lost my wallet and feeling fine](/archives/2006/05/14/lost-my-wallet-and-feeling-fine)
- [cvs-bisect](/archives/2006/01/20/cvs-bisect)

Comments are closed.
