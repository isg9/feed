---
title: Antelopes — wingolog
url: https://wingolog.org/archives/2004/09/19/the-countdown-begins
published: "2004-09-19T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2004/09/19/the-countdown-begins
---

# Antelopes — wingolog

## [Antelopes](/archives/2004/09/19/the-countdown-begins)

19 September 2004 6:53 PM

- [namibia](/tags/namibia)
- [scheme](/tags/scheme)
- [gnome](/tags/gnome)

### Regarding wildebeests

Got out some hopefully final prereleases of guile-gnome today. If all is well, we can release tomorrow afternoon or so. Doing things the GNU way takes a lot of non-hacking work: AUTHORS must record who wrote and changed *every file*, actual pieces of paper need to be signed for changes to go in, etc. I'm fudging on this release, though. There are still some unassigned copyrights that need to be dealt with. Oh well!

I want to be able to write panel applets in Guile. To do that, we need to wrap CORBA (via ORBit), then bonobo and bonoboui, and finally libpanel-applet. Fortunately, Martin Baulig had already done ORBit in the original guile-gobject, but the inevitable ORBit and guile-gnome changes broke it over time. After a bit of hacking and GDB last night, I was pleasantly surprised to see it working for the first time. With luck, I might have a panel applet by the end of the week. Just in time for bonobo to be deprecated, I guess :/

Oddly enough, the GNU coding standards suggest that apps provide a CORBA interface. Pretty bizarre. CORBA is complicated, yo. DBus shows that less is more. Still, those GNU documents are an accumulation of wisdom, or perhaps more buzz-wordy "best practice". There's something there.

In a Namibian connection, both penguins and gnus can be found in this country. Perhaps these African hacks aren't so out of place after all!

Speaking of which...

### The countdown begins

86 days, only about 25 of which will actually be spent teaching. Time accelerates; December 15th is the day. I think I'll fly to Spain to see my sister, then back to the states after Christmas. Anxiety!

## related articles

- [photobloggin](/archives/2004/10/17/photobloggin)
- [schoolnet, guile-gnome](/archives/2004/07/15/schoolnet-guile-gnome)
- [focus](/archives/2004/02/11/advogato-29)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a schemer at jsconf.eu](/archives/2011/10/04/a-schemer-at-jsconf-eu)
- [poverty, riches](/archives/2009/05/14/poverty-riches)

Comments are closed.
