---
title: a first stab at modern texinfo — wingolog
url: https://wingolog.org/archives/2003/11/14/advogato-22
published: "2003-11-14T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2003/11/14/advogato-22
---

# a first stab at modern texinfo — wingolog

## [a first stab at modern texinfo](/archives/2003/11/14/advogato-22)

14 November 2003 7:14 AM

- [namibia](/tags/namibia)
- [scheme](/tags/scheme)

**guile-gobject**

The library we use to make the scheme<->C connection, g-wrap, is now in CVS. Andreas Rottman has done some interesting things with it to extend it to use libffi extensively. That's important for us, as the gtk wrapper is about 140 KB now, which gcc-3.3 -O2 chokes on. The changes shrink it to about 20 KB.

So, I hope I hope I hope a g-wrap release comes soon. Then other parts of the stack can be released: guile-gobject (to be renamed to guile-gnome), guile-gstreamer, and finally soundscrape.

**soundscrape**

That's the stack for me anyway. While waiting on that to happen, I had some interesting ideas about help interfaces. The existing gnome help interface, calling an external program to display a file, is a bit inadequate: It takes a while to start up, you can't dynamically add help, the interface is disjoint from the program, and finally I'd like to have docs in texinfo format.

Texinfo traditionally has its own problems: `inf` sucks, to put it politely, and you can't read it while you're running the program.

So I wrote a new help browser. It's all written in scheme. As an input, it takes nodes of "structured text", a simple kind of s-expression markup. The structured-text->gtk-txt-buffer code is rather elegant, I think, but the texinfo->nodes-of-structured-text code is somewhat nasty. Anyhow, I got a [screenshot of it](http://ambient.2y.net/soundscrape/shots/14-nov-2003.png) last night running on some docs of the guile distro. This code is in `(soundscrape ui help)` in soundscrape cvs, although it might find a home in guile-gnome at some point.

I think this is an interesting development: it brings texinfo into the world of graphical applications. Docbook is just too much of a PITA to write ;)

**teaching**

Today was the last day of school before examinations. In a couple of classes, I had learners write down "I Can't" statements on pieces of paper. We then folded them and put them in a box. We solemnly and sillily (a word?) marched outside, dug a hole in the sand, then burned and buried the papers. I read a eulogy to "I Can't", through giggling and all from the learners. I think they liked it, though. We were all grinning.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [poverty, riches](/archives/2009/05/14/poverty-riches)
- [culture and code](/archives/2004/12/06/culture-and-code)
- [cromulation](/archives/2004/11/24/cromulation)
- [photobloggin](/archives/2004/10/17/photobloggin)
- [Antelopes](/archives/2004/09/19/the-countdown-begins)

Comments are closed.
