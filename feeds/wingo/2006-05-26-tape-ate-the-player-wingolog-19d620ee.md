---
title: tape ate the player — wingolog
url: https://wingolog.org/archives/2006/05/26/tape-ate-the-player
published: "2006-05-26T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2006/05/26/tape-ate-the-player
---

# tape ate the player — wingolog

## [tape ate the player](/archives/2006/05/26/tape-ate-the-player)

26 May 2006 7:55 PM

- [america](/tags/america)
- [hacks](/tags/hacks)
- [aikido](/tags/aikido)
- [streaming](/tags/streaming)

**Home**

I've had a series of fine days. A festa major in a small street near Gràcia, a meatful party at the house in which no glass was broken, and for the first time I was chosen as the person on whom the aikido instructor would demonstrate a technique. Granted, 15 of the best folks at the school were gone to Japan to participate in the [All Japan Aikido Demonstration](http://www.aikikai.or.jp/eng/info/2006/info_enbu.htm), but hey, I take my moments as they come.

**Other home**

I'll be flying out on Sunday to North Carolina. After running around a bit, I fly to San Francisco from Thursday to Thursday, hopefully my visa situation is all ready for me to go to Washington on Friday, back to Charlotte on Saturday, and hop the aluminum airtubebus back to Spain on the Sunday two weeks after arriving.

That is the theory. Practice may or may not follow the theory.

**Work**

I realized the other day that I've been hacking proprietary software for going on two months now. Not NDA-proprietary, not copyright-proprietary, but just proprietary. Software to run the Flumotion streaming server on a large cluster of machines. Proprietary in the sense that it's only useful to some organization planning to host streams from many customers, streaming to many many listeners.

It's been interesting, although I have to figure out how to get back to free software or otherwise rationalize my existence.

**Hack**

I wrote a lossless ogg/theora+vorbis cutter a few weeks ago and never said anything about it. It has a very simple installation process: just [click here](http://webcvs.freedesktop.org/*checkout*/gstreamer/gst-python/examples/remuxer.py?rev=1.7). You'll need up-to-date gstreamer, gst-plugins-base, and gst-python. Ubuntu Dapper will do.

![](//wingolog.org/pub/video-munger.png)*mungeariffic*

For example, if you like the [video on vilanova](http://stream.fluendo.com/archive/6uadec/vilanova_presentation.ogg), but just wanted the part about the festa major, cut cut cut and you have a [lossless cut of the video](//wingolog.org/pub/vilanova_presentation-remuxed.ogg).

I'm pretty sure it produces correct files, although in general it outputs a few P frames before the first I frame. While correct, this confuses some common players until the first keyframe is processed. Ignore the man behind the curtain.

## related articles

- [bread crumbs](/archives/2007/06/08/bread-crumbs)
- [it can't be all that pretty](/archives/2006/05/19/it-cant-be-all-that-pretty)
- [canonical source](/archives/2005/01/08/canonical-source)
- [digital interfaces](/archives/2009/11/07/digital-interfaces)
- [read what i wrote on my shirt](/archives/2009/10/08/read-what-i-wrote-on-my-shirt)
- [another (turkey) bites the dust](/archives/2008/11/25/another-turkey-bites-the-dust)

Comments are closed.
