---
title: up to the date — wingolog
url: https://wingolog.org/archives/2005/06/17/up-to-the-date
published: "2005-06-17T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/06/17/up-to-the-date
---

# up to the date — wingolog

## [up to the date](/archives/2005/06/17/up-to-the-date)

17 June 2005 2:16 PM

- [random](/tags/random)
- [hacks](/tags/hacks)

**work**

Spent the last few days chasing down a doubt-your-compiler, hate-your-life bug. Segfaults deep in the python interpreter, but always in the same place -- in python code, inside twisted. Only reproduceable after connecting a few hundred clients to a flumotion server. I didn't realize how it was getting to me until getting home yesterday, pounding headache, feeling like I'd just been hit by a truck.

Turns out I caused it, and the [fix](http://cvs.freedesktop.org/gstreamer/gst-python/gst/gstcaps.override?r1=1.1.2.6&r2=1.1.2.8&only_with_tag=BRANCH-GSTREAMER-0_8) was minute. I don't know what more to say about it.

**other**

Last weekend the aikido dojo here hosted a seminar with Peter Bernath of [Florida Aikikai](http://www.floridaaikikai.com/). Wonderful teacher, precise and fluid. This weekend in BCN there's the [Sónar](http://sonar.es) festival, which I hope to catch a bit of. No outside of work hacking going on tho.

## related articles

- [powda](/archives/2005/03/20/powda)
- [bread crumbs](/archives/2007/06/08/bread-crumbs)
- [steal our code](/archives/2007/05/03/steal-our-code)
- [tape ate the player](/archives/2006/05/26/tape-ate-the-player)
- [it can't be all that pretty](/archives/2006/05/19/it-cant-be-all-that-pretty)
- [lost my wallet and feeling fine](/archives/2006/05/14/lost-my-wallet-and-feeling-fine)

Comments are closed.
