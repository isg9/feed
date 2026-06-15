---
title: Suboptimality — wingolog
url: https://wingolog.org/archives/2005/09/30/suboptimality
published: "2005-09-30T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2005/09/30/suboptimality
---

# Suboptimality — wingolog

## [Suboptimality](/archives/2005/09/30/suboptimality)

30 September 2005 1:13 PM

- [gnome](/tags/gnome)
- [gstreamer](/tags/gstreamer)

I have an old laptop set up in my living room with all my music on it. It's neat to have all my music ready for playing, right when I get home.

I'm pretty pleased with the setup, but I think that's because I adapt myself to the software. When my flatmates want to use it, I cringe at having to tell them things they shouldn't have to know.

(As an aside, I had the same experience teaching computers in Namibia. I couldn't bear to insult the kids by teaching them how to work around warts in the software. This was around Gnome 2.6 or so.)

So. Someone puts in a CD. A while later, although there's no indication something is happening, sound-juicer pops up. Cool. It might even have all of the tracks loaded from musicbrainz or something. But...

1. The process of getting tracks from wherever blocks the rest of the UI. I should be able to play while this is happening.

2. If I press 'play', nothing happens. I have to *select* the first track, then press play.

3. While playing, the extract button is insensitive. So it seems I can't extract while playing? Fine, maybe I can play while extracting... No, can't do that either... It should be possible -- the first thing you want to do with a new CD is play it, and ripping it would be a close second. Two birds, one stone would be nice.

4. If there is an error in the disc, the whole UI blocks for a long time.

5. When the ripping is finished, the disc is ejected. Great. When I put another in, it blocks the sound-juicer UI without any stated reason for 4-5 seconds, and then doesn't even read my new CD. I have to fiddle in menus to manually tell s-j to reread the disc. Duh...

6. Then, neither Rhythmbox nor Jamboree automatically adds these files to their libraries. There should be some way of adding files as they are finished writing, so that I can play them immediately. At the very least they should all be added when ripping finishes. Not sure how to do this though.

As a side note, when one flatmate wanted to add a disc to the library, she went to the File -> Add Folder... option in Jamboree's menu. This menu item has a nice "+" icon next to it, seems logical you should be able to add music there... Of course she got lost in the file chooser. Admittedly, this was when gnome-volume-manager wasn't working -- normally after putting in a CD sound-juicer just pops up without having to choose anything from a menu. (I'm running a psuedo-xfce desktop on that machine.)

I like sound-juicer a lot. I'll like it a lot better when these things are fixed. I'll take a look at fixing them myself, but I wanted to write them down while they were fresh in my mind.

## related articles

- [notes from the bosphorus](/archives/2008/07/09/notes-from-the-bosphorus)
- [dv capture](/archives/2005/10/05/dv-capture)
- [purity of intent](/archives/2005/03/24/98)
- [rococo, and then rubble](/archives/2012/06/08/rococo-and-then-rubble)
- [a schemer at jsconf.eu](/archives/2011/10/04/a-schemer-at-jsconf-eu)
- [to guadec!](/archives/2011/08/03/to-guadec)

Comments are closed.
