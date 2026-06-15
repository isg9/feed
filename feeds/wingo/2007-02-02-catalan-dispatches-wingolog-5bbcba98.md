---
title: catalan dispatches — wingolog
url: https://wingolog.org/archives/2007/02/02/catalan-dispatches
published: "2007-02-02T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2007/02/02/catalan-dispatches
---

# catalan dispatches — wingolog

## [catalan dispatches](/archives/2007/02/02/catalan-dispatches)

2 February 2007 8:48 PM

- [scheme](/tags/scheme)
- [gnome](/tags/gnome)
- [guile](/tags/guile)
- [calçots](/tags/cal%C3%A7ots)
- [hack](/tags/hack)
- [soundscrape](/tags/soundscrape)
- [fosdem](/tags/fosdem)

**calçotada**

[![](//wingolog.org/photos//2007/1/28/DSC01759.normal.JPG)](//wingolog.org/index.py/tags/calçotada)

*ebullient onion*

Headed down to Valls last weekend with some folks, to pay our respects to the [humble spring onion](http://en.wikipedia.org/wiki/Calçot/). Six euros for a load of edible onion, a cup of romesco sauce for dipping, bread, an orange, and half a bottle of wine. Delicious!

**fosdem**

I decided to go to [FOSDEM](http://fosdem.org/) again this year. It's too hard to resist belgian food and beverage.

This time I'll be giving a talk on hacking GNOME applications with Scheme via the [guile-gnome](http://www.gnu.org/software/guile-gnome/) bindings. I dig on programming in Scheme -- it's quite refreshing, mentally. So anyone who's not turned off by minority languages should come by and see what's up. As long as you're hacking for fun there's no reason not to choose your tools.

**copyright 2003,2004,2007**

I'm still plodding away on reviving an old synthesizer project. I had it going OK back in the GStreamer 0.6/0.8 days, but time pulled the bits out from beneath me. In the last month I've been focusing on getting the low-latency, pull-mode GStreamer stuff going right.

The current state of pull-mode scheduling in GStreamer is a bit shaky. It hasn't really been designed. I've been working a little bit on this, but without some code review it might not work out. I've ported audio sinks to work in pull mode, and ladspa plugins also. Wim already handled basetransform in pull mode. Apart from that I've had to write interleaver and deinterleaver elements, and I just committed a libsndfile sink that can operate in either mode. That's aside from the 100 or so elements that I have in the synthesizer project itself.

Add to that bugfixes and updates across g-wrap and guile-gnome and I'm lucky to have reached about 30% hack time on the synthesizer itself. Maybe it will be worth it when things are finished? Maybe GStreamer won't change so much in the future? Who knows. I think about the Jokosher people, who seem to have gotten far because they can't or won't hack the stack, while I can't seem to leave it alone. At least I never got into hacking GTK+ or GLib.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [fscons 2011: free software, free society](/archives/2011/11/28/fscons-2011-free-software-free-society)
- [twenty ten](/archives/2010/01/27/twenty-ten)
- [the good hack](/archives/2009/07/26/the-good-hack)
- [dispatch strategies in dynamic languages](/archives/2008/10/17/dispatch-strategies-in-dynamic-languages)
- [brought to you by torres viña sol](/archives/2007/11/10/brought-to-you-by-torres-vina-sol)

Comments are closed.
