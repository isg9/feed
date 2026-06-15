---
title: notes from the bosphorus — wingolog
url: https://wingolog.org/archives/2008/07/09/notes-from-the-bosphorus
published: "2008-07-09T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/07/09/notes-from-the-bosphorus
---

# notes from the bosphorus — wingolog

## [notes from the bosphorus](/archives/2008/07/09/notes-from-the-bosphorus)

9 July 2008 11:56 AM

- [guadec](/tags/guadec)
- [gnome](/tags/gnome)
- [istanbul](/tags/istanbul)
- [gstreamer](/tags/gstreamer)
- [abi](/tags/abi)

My camera broke; I have only words.

**the golden horn**

Istanbul is a town full of wonder. And wander: around the cobbled streets of the old town, in the morning, in the evening, alive at all hours.

Last night I shared a water-pipe under the bridge, looking back on the night silhouette of the old town, smoke rings dissipating over the water. As we were walking back to the hostel some hours later, [Dimitris](http://dimitris.glezos.com/weblog/) noted the waft of baking bread, the start of a new day.

I stepped out on Sunday morning to the GStreamer mini-summit, but was waylaid by the Blue Mosque. Outside it is grey, hard stone and spired; inside it is lush and tactile, the carpet creeping up between your toes. I believe that space can have rhythm. In that place there was a rich visual soundscape, tiled motifs repeating on the macro level, fractally recursing into micro-vegetation, a symphony of space and lines. I stumbled out into the blue sky.

**gstreamer, breakage**

The GStreamer summit was pretty good. We decided to switch to git (from CVS), once some issues are ironed out regarding history and our "common" submodule. We also decided that at some point we should do a new development cycle, but that we needed reasons for doing so; the idea would be to develop a number of features that cannot be done with 0.10 in experimental branches, and once there are enough branches, we pull them together in a quick 0.11 and from there to 0.12 or 1.0. This would be a process that could take a year or two.

In that regard, some interesting points were brought up regarding GLib and GTK+'s plan to break ABI for version 3. The problem is that any library that depends on GLib will break ABI as well, and that includes GStreamer. So given that we will need to break ABI to depend on GLib 3, that gives us a good timetable for a next development series, corresponding to GLib 3's release in about 16 months. I suspect that many projects will want to do the same.

## related articles

- [to guadec!](/archives/2011/08/03/to-guadec)
- [planetary rotations](/archives/2010/08/08/planetary-rotations)
- [guadec ho!](/archives/2009/07/02/guadec-ho)
- [regarding decadence](/archives/2008/06/10/regarding-decadence)
- [gnome in the age of decadence](/archives/2008/06/07/gnome-in-the-age-of-decadence)
- [to birmingham!](/archives/2007/07/14/to-birmingham)

### 3 responses

1. [Anonymous Coward](http://www.anonymous.com) says:[9 July 2008 3:32 PM](#6f02d0f3f931129fb875819df4a1f3231cb577e4)

   I would be nice to see C++ bindings planned too...

2. Jeff Ollie says:[10 July 2008 2:02 AM](#c238ad6be37f38a0dced56252ac4db65b3a9ecbc)

   Yay to switching to git!

3. [Ricky Younglood](http://dictionary.reference.com/browse/git) says:[14 July 2008 7:57 AM](#58e255bfe072f8b3eba5e9aa7ac5a526cb72215e)

   Seriously, who really cares about git?

   Some details that I think you may have left out. The tea and beer in Istanbul are one of the biggest jokes out there. The call to prayer is pretty incredible, especially when you start sleeping through it.

   Anonymous Coward: Do it yourself.

Comments are closed.
