---
title: cvs-bisect — wingolog
url: https://wingolog.org/archives/2006/01/20/cvs-bisect
published: "2006-01-20T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2006/01/20/cvs-bisect
---

# cvs-bisect — wingolog

## [cvs-bisect](/archives/2006/01/20/cvs-bisect)

20 January 2006 10:06 PM

- [hacks](/tags/hacks)

Simple script to do a binary search over your CVS repository to find out at when you serviced the pooch. Similar in spirit to [git-bisect](http://www.kernel.org/pub/software/scm/git/docs/git-bisect.html), although much more pedestrian.

Fetch it here: [cvs-bisect](//wingolog.org/pub/cvs-bisect). A sample run looks like this:

```
$ cat my-test
#!/bin/bash
test `cat CVS/Tag` \< D2005.11.06 || exit 1
$ cvs-bisect 2005-08-01 Now ./my-test
* Checking that the check fails on 2006-01-20 17:44:52...
* Checking that the check succeeds on 2005-08-01 00:00:00...
* Updating tree to 2005-10-26 09:22:26...
* Running check
* Check passed
* Updating tree to 2005-12-08 13:03:39...
* Running check
* Check failed
* Updating tree to 2005-11-16 22:43:02...
* Running check
* Check failed
* Updating tree to 2005-11-06 03:32:44...
* Running check
* Check failed
* Updating tree to 2005-10-31 17:57:35...
* Running check
* Check passed
* Updating tree to 2005-11-03 10:45:09...
* Running check
* Check passed
* Updating tree to 2005-11-04 19:08:56...
* Running check
* Check passed
* Updating tree to 2005-11-05 11:20:50...
* Running check
* Check passed
* Updating tree to 2005-11-05 19:26:47...
* Running check
* Check passed
* Updating tree to 2005-11-05 23:29:45...
* Running check
* Check passed
* Updating tree to 2005-11-06 01:31:14...
* Running check
* Check failed
* Updating tree to 2005-11-06 00:30:29...
* Running check
* Check passed
* Updating tree to 2005-11-06 01:00:51...
* Running check
* Check failed
* Bisection stopped.

Command:
  "./my-test"
started to fail somewhere between 2005-11-06 00:30:29 and 2005-11-06 01:00:51.

```

In other toolchain-related hacks, I have become so addicted to the [new browse.cgi pages](http://bugzilla.gnome.org/browse.cgi?product=GStreamer) on bugzilla that I couldn't bear to use the [trac](http://www.edgewall.com/trac/) installation we have at work. So, a couple of macros later and I have a [poor man's browse.cgi](http://core.fluendo.com/flumotion/trac/wiki/Dashboard) running. Makes trac into a more liveable place to spend your time.

I threw the macros into a [page on the trac wiki](http://core.fluendo.com/flumotion/trac/wiki/TracMacros), if others are interested.

## related articles

- [bread crumbs](/archives/2007/06/08/bread-crumbs)
- [steal our code](/archives/2007/05/03/steal-our-code)
- [tape ate the player](/archives/2006/05/26/tape-ate-the-player)
- [it can't be all that pretty](/archives/2006/05/19/it-cant-be-all-that-pretty)
- [lost my wallet and feeling fine](/archives/2006/05/14/lost-my-wallet-and-feeling-fine)
- [Google profiler a nice gesture but.](/archives/2005/11/01/google-profiler-a-nice-gesture-but)

Comments are closed.
