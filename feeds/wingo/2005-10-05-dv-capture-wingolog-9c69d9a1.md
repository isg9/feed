---
title: dv capture — wingolog
url: https://wingolog.org/archives/2005/10/05/dv-capture
published: "2005-10-05T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/10/05/dv-capture
---

# dv capture — wingolog

## [dv capture](/archives/2005/10/05/dv-capture)

5 October 2005 8:56 PM

- [gnome](/tags/gnome)
- [gstreamer](/tags/gstreamer)

Having settled a number of issues with [Flumotion](http://flumotion.net/) and GStreamer 0.9, I sat down on Monday to check on the condition of DV capture over firewire. I already knew a bit of what to expect: kernel deadlocks. Ever since I got this SMP machine, that's what I see. Turns out DV capture deadlocks on all SMP machines I could find. I suppose I have a talent for breaking machines :-)

But, [Mr. Love's book](http://www.amazon.com/gp/product/0672327201/104-6457860-4079909?v=glance&n=283155&v=glance) in hand, I set out to figure out what was going on so I could submit a decent bug report, or at least figure out if someone else had already solved the issue. Ended up learning all kinds of neat things about spinlocks and interrupts, eventually (two days later) coming up with a patch that Works For Me (tm).

On the GStreamer side of things, it's the time of the week when the new summary comes out. Catch it on the mailing list, or over at [planet gnome news](http://planet.gnome.org/news/).

Speaking of planets, hello [p.g.o](http://planet.gnome.org/)!

## related articles

- [notes from the bosphorus](/archives/2008/07/09/notes-from-the-bosphorus)
- [Suboptimality](/archives/2005/09/30/suboptimality)
- [purity of intent](/archives/2005/03/24/98)
- [rococo, and then rubble](/archives/2012/06/08/rococo-and-then-rubble)
- [a schemer at jsconf.eu](/archives/2011/10/04/a-schemer-at-jsconf-eu)
- [to guadec!](/archives/2011/08/03/to-guadec)

Comments are closed.
