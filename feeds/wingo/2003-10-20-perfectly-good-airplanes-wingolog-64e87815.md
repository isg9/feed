---
title: perfectly good airplanes — wingolog
url: https://wingolog.org/archives/2003/10/20/advogato-20
published: "2003-10-20T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2003/10/20/advogato-20
---

# perfectly good airplanes — wingolog

## [perfectly good airplanes](/archives/2003/10/20/advogato-20)

20 October 2003 11:12 AM

- [namibia](/tags/namibia)
- [scheme](/tags/scheme)

**guile-gobject**

I put in some custom bindings for GtkTreeView and GtkTextView (and associated classes). Now we have a guile-gtk-demo, and there's a guile-implemented tree model there too (modeled after [jamesh](http://advogato.org/person/jamesh/)'s python tree model).

I started to wrap libvte so many graphical repl windows can be created, but that's going slow. Looks like I need to wrap GdkEvent specially before I can capture keypresses and feed them to the terminal. Also I'm kindof bummed that I can't use readline, as it stores state in static variables. Maybe if I fork and run the readline in a separate guile process, but that sucks.

**soundscrape**

Not so much work going on there. I've sketched out what I want the graphical part to look like, but I need to wrap that graphical repl first.

**life**

Went skydiving last weekend. It was pretty cool. It was my first jump so they had me attached to a "static line" which actually pulled out the main chute. The free fall was just about a second or two. Everything went fine, my lines weren't even twisted. A wind change put me down into a field though. Maybe I'll go again at the end of November.

A weird time, though. A lot of those (white) people were constantly badmouthing the black folks (where I live and with whom I work). I'm used to dealing with it from growing up in North Carolina, so I personally could still enjoy myself. But those people live in a strange world.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [poverty, riches](/archives/2009/05/14/poverty-riches)
- [culture and code](/archives/2004/12/06/culture-and-code)
- [cromulation](/archives/2004/11/24/cromulation)
- [photobloggin](/archives/2004/10/17/photobloggin)
- [Antelopes](/archives/2004/09/19/the-countdown-begins)

Comments are closed.
