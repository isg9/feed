---
title: updates — wingolog
url: https://wingolog.org/archives/2005/07/08/updates
published: "2005-07-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/07/08/updates
---

# updates — wingolog

## [updates](/archives/2005/07/08/updates)

8 July 2005 5:48 PM

- [hacks](/tags/hacks)

A few things. Too lazy to write complete sentences. (A disease that I should cure at some point.)

**hacking**

Hacking a lot on GStreamer, for the 0.9 version. Core approaching stable, with some possible exceptions for format (caps) renegotiation originating from downstream elements, e.g. from the video output. Have been porting plugins and hacking core, find myself porting more now -- probably a good thing.

Dogfooding via [Jamboree](http://developer.imendio.com/wiki/Jamboree), another itunes clone with an exceptionally clean codebase. Can be checked out via `cvs -d :pserver:anonymous@anoncvs.gnome.org:/cvs/gnome co -r jamboree-gst-0-9 jamboree`. Up-to-date gstreamer, gst-plugins-base necessary.

Designed a network clock synchronization algorithm using windowed least squares. Tested on simulator written in scheme (lazy lists + tail recursion), output piped to python program, graphed with the most excellent [matplotlib](http://matplotlib.sf.net). Very configurable, example run [here](//wingolog.org/pub/network-clock.png). Source [here](http://cvs.freedesktop.org/gstreamer/gstreamer/tests/network-clock.scm?rev=1.6&view=markup) [here](http://cvs.freedesktop.org/gstreamer/gstreamer/tests/network-clock-utils.scm?rev=1.4&view=markup) [here](http://cvs.freedesktop.org/gstreamer/gstreamer/tests/plot-data?rev=1.2&view=markup).

```
$ ./print-backtraces
usage: ./print-backtraces program-name

```

A [nifty script](//wingolog.org/pub/print-backtraces). Connects to a running process named `program-name`, prints backtraces in all threads, exits. s,pidof,/sbin/pidof/, recommended for fedora users.

**life**

Sister came last weekend, pleasant. Going to San Fermines this weekend. Sky squeezing out raindrops, not very successful. Saw star wars, acting certificate of actor playing annakin must be revoked. Ordered ibook today. Chillin like a villain.

## related articles

- [bread crumbs](/archives/2007/06/08/bread-crumbs)
- [steal our code](/archives/2007/05/03/steal-our-code)
- [tape ate the player](/archives/2006/05/26/tape-ate-the-player)
- [it can't be all that pretty](/archives/2006/05/19/it-cant-be-all-that-pretty)
- [lost my wallet and feeling fine](/archives/2006/05/14/lost-my-wallet-and-feeling-fine)
- [cvs-bisect](/archives/2006/01/20/cvs-bisect)

Comments are closed.
