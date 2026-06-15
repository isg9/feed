---
title: gst-editor again — wingolog
url: https://wingolog.org/archives/2002/06/18/advogato-4
published: "2002-06-18T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2002/06/18/advogato-4
---

# gst-editor again — wingolog

## [gst-editor again](/archives/2002/06/18/advogato-4)

18 June 2002 10:58 AM

- [gstreamer](/tags/gstreamer)

**gstreamer**

Well, [thomasvs](http://advogato.org/person/thomasvs/), for better or for worse, switched the default scheduler in GStreamer over to the one that uses my new cothreads package. For the record, cothreads are a sort of low-level user-space thread that require manual scheduling, and are how exceptions and tail-recursion work. Sorta.

Anyway, it's been causing problems; for some reason it doesn't run on a lot of redhat boxen. An odd problem that I'm looking into, I haven't had it fail on my box in a long time. I suppose it's for the better, giving it the trial by fire. Who knows ;)

**gst-editor**

It actually runs pipelines now, and can do many neat things. Still some bugs to work out, though...

## related articles

- [notes from the bosphorus](/archives/2008/07/09/notes-from-the-bosphorus)
- [catching memory leaks with valgrind's massif](/archives/2008/05/05/catching-memory-leaks-with-valgrinds-massif)
- [why god made buttermilk](/archives/2006/12/17/why-god-made-buttermilk)
- [radical adults](/archives/2006/12/15/radical-adults)
- [cypherpunks writecode](/archives/2006/07/21/cypherpunks-writecode)
- [plane from spain](/archives/2005/12/21/plane-from-spain)

Comments are closed.
