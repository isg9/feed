---
title: Driving Mr. Schaller — wingolog
url: https://wingolog.org/archives/2004/08/19/driving-mr-schaller
published: "2004-08-19T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2004/08/19/driving-mr-schaller
---

# Driving Mr. Schaller — wingolog

## [Driving Mr. Schaller](/archives/2004/08/19/driving-mr-schaller)

19 August 2004 10:47 AM

- [namibia](/tags/namibia)
- [gnome](/tags/gnome)

### supercollider

For the last year and a half, I've been coding up a program similar to [SuperCollider](http://audiosynth.com/). Of course, first I had to work on the [language](http://home.gna.org/guile-gnome/) [in which](http://home.gna.org/guile-lib/) [to write](http://gstreamer.net/) such a program, but I'm back. [Soundscrape](http://ambient.2y.net/soundscrape/) is on the verge of being released.

It was with some trepidation, then, that I noted [SuperCollider's appearance in debian SID](http://packages.debian.org/supercollider/). I mean, I've been trying all this time to reproduce this program, expanding on it in the process, and here it is, only an apt-get away. I swallowed my pride and selected it.

A few hours later, it was on my computer. There are three binaries: one for the server, one for a REPL client, and one for a graphical frontend.

I guess I'm lucky. I don't really have anything to worry about. One, the GUI is written in Tcl/Tk, not SC's own SmallTalk dialect. So SC is still not graphically extensible from within the SC language. (SC had an unfortunate run-in with a contagiously proprietary GUI library, which made the GUI code unreleaseable.) Whereas, I wrote the Guile bindings for Gnome libraries. +1 soundscrape.

Secondly, the frontend is [really primitive](http://ambient.2y.net/wingo/shots/supercollider-linux.png). Running the language frontend was no help, either; I couldn't get it to do anything, and no help was forthcoming. Whereas, soundscrape has an extremely rich programming environment, from command-line help to a complete help browser. The browser can read the [(thorough) tutorial](http://ambient.2y.net/soundscrape/docs/tutorial/), as well as providing an indexed reference for every binding and unit generator in the system.

However, I really need to release. By the end of the month, at the latest.

### vacation

Ever go on a long vacation with a friend, only to find out that you really can't stand each other on the road? Ever do it with someone you had never met before in person?

Fortunately, that was not the case with me and [Christian Schaller](http://advogato.org/person/Uraeus/). Besides an astounding inability to recognize good music (Jim+Jennie and the Pinetops are hot!), Christian's a good guy and we travelled well together. We rented a car, made our own roads, got lost, found our way, picked up travellers, crashed at friends', taxied teachers, saw thousand-year old plants and rock engravings six times that age, got sandblasted, saw shipwrecks and Germans, got stuck in sand, almost drove off of cliffs, drove on the wrong side of the road, and mercilessly dispatched an army of Windhoek Lagers. Photos can be had over on [ambient](http://ambient.2y.net/photos/). Look for my stuff at the bottom of the page.

It was a good trip. We got endless ribbing from friends, though -- "This is Wingo and Christian. They met over the internet."

## related articles

- [turkles](/archives/2004/11/30/turkles)
- [weenies](/archives/2004/10/29/weenies)
- [bonobo-slay](/archives/2004/10/20/bonobo-slay)
- [photobloggin](/archives/2004/10/17/photobloggin)
- [Antelopes](/archives/2004/09/19/the-countdown-begins)
- [heading out](/archives/2004/07/29/heading-out)

Comments are closed.
