---
title: yuletide notes — wingolog
url: https://wingolog.org/archives/2007/12/23/yuletide-notes
published: "2007-12-23T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2007/12/23/yuletide-notes
---

# yuletide notes — wingolog

## [yuletide notes](/archives/2007/12/23/yuletide-notes)

23 December 2007 7:41 PM

- [original](/tags/original)
- [thanksgiving](/tags/thanksgiving)
- [turkey](/tags/turkey)
- [hack](/tags/hack)
- [correfoc](/tags/correfoc)
- [sant andreu](/tags/sant%20andreu)
- [home](/tags/home)
- [tags](/tags/tags)
- [f-spot](/tags/f-spot)

**notes from the gaining daylight**

It is alternately Christmas Eve's Eve, or the day of the Barça--Real Madrid game, depending on your religion.

My neighborhood had its yearly festival earlier this month, oddly scheduled in the winter. There was a [correfoc](http://en.wikipedia.org/wiki/Correfoc):

[![](//wingolog.org/photos//2007/12/08/DSC02284.thumb.JPG)](//wingolog.org/photos/index.py/photos/2661)[![](//wingolog.org/photos//2007/12/08/DSC02280.thumb.JPG)](//wingolog.org/photos/index.py/photos/2657)[![](//wingolog.org/photos//2007/12/08/DSC02296.thumb.JPG)](//wingolog.org/photos/index.py/photos/2673)[more...](//wingolog.org/photos/index.py/rolls/55)

In other December news, I did end up cooking thanksgiving, with a turkey, stuffing, mashed potatoes, a dozen guests, day-after gumbo, etc. Sixth in a row outside the US -- perhaps next year I can get someone else to cook it for me.

And, in the vein of home cooking, I fly home to North Carolina tomorrow, if the plane gods permit.

**photo gallery hacking**

My web photo gallery software, [Original](//wingolog.org/software/original/), tries to present a taggy interface to your photos [(example)](//wingolog.org/photos/index.py/tags/). The tag cloud is really interesting when it's small, because each new addition of photos makes a notable impact on the shape of the tag cloud. However after you have about a thousand photos or so, it's more difficult to see your new photos, and how they relate to older ones.

Since about version 0.4.0 or so, the excellent [F-Spot](http://f-spot.org/) marks photos that you import as being in a "roll" -- each import makes a new roll. This is pretty interesting information, allowing a more [chronological presentation](//wingolog.org/photos/index.py/rolls/) of photos.

Rolls also take burden off of the tagging interface -- without some kind of chronological discretization, tags get misused for navigation. The gallery [index](//wingolog.org/photos/) used to show the [large tag cloud](//wingolog.org/photos/index.py/tags/) just so photos with lesser-used tags can be found. Now, with rolls available, one can search for photos via the two axes of time and tag, as well as get more [targeted tag listings](//wingolog.org/photos/index.py/rolls/55).

The one problem now is all of the photos that you have in your f-spot library that were imported before roll information began to be recorded. I wrote an f-spot extension to handle this case, which is currently still [sitting around in bugzilla](http://bugzilla.gnome.org/show_bug.cgi?id=497136). If you trust me, you can install it by copying the [built DLL](//wingolog.org/pub/RetroactiveRoll.dll) into your `~/.gnome2/f-spot/addins/` directory. You use it by selecting a set of photos, right clicking on them, and selecting "Assign to new roll" or whatever it says. Good luck, and bugs to that bugzilla report.

## related articles

- [another (turkey) bites the dust](/archives/2008/11/25/another-turkey-bites-the-dust)
- [autumn sweater](/archives/2006/11/20/autumn-sweater)
- [twenty ten](/archives/2010/01/27/twenty-ten)
- [read what i wrote on my shirt](/archives/2009/10/08/read-what-i-wrote-on-my-shirt)
- [the good hack](/archives/2009/07/26/the-good-hack)
- [dispatch strategies in dynamic languages](/archives/2008/10/17/dispatch-strategies-in-dynamic-languages)

Comments are closed.
