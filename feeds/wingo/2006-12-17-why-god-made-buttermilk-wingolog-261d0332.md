---
title: why god made buttermilk — wingolog
url: https://wingolog.org/archives/2006/12/17/why-god-made-buttermilk
published: "2006-12-17T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2006/12/17/why-god-made-buttermilk
---

# why god made buttermilk — wingolog

## [why god made buttermilk](/archives/2006/12/17/why-god-made-buttermilk)

18 December 2006 0:27 AM

- [scheme](/tags/scheme)
- [gstreamer](/tags/gstreamer)
- [guile](/tags/guile)
- [food](/tags/food)

**morning**

I would like to note an underexplored territory that is ripe for the munching, the "southern buttermilk biscuit -- mallorcan [sobrassada](http://en.wikipedia.org/wiki/Sobrassada)" interface. Spreadable sausage on a biscuit. God did truly shine his face on my kitchen this morning.

**guile**

I got guile-gstreamer working today, which is hot. [Version 0.9.90](http://article.gmane.org/gmane.lisp.guile.gtk/539) is totally released. The last bit that I had to do was to leave "guile mode" when the wrapper calls C functions, and to enter guile mode when GClosures are invoked. That is what it takes to be thread-friendly with Guile; while multiple guile threads can be running at the same time, they have to not block, because GC or thread joining requires cooperation from all threads.

So, multithreaded GStreamer and Guile. I'm pleased, this is already more than the old 0.8 wrapper did. Also the process of porting to 0.10 was mostly removing crufty code, which says nice things about the state of GStreamer-the-library.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [radical adults](/archives/2006/12/15/radical-adults)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)

### One response

1. [tiffany](http://comingsoon) says:[24 December 2006 10:38 AM](#234)

   dude, did you use my biscuit recipe??

   hey...i need yer help. er advice. i have decided to start a website for my columns and other writings. i got dreamweaver and ivan's gonna teach me some website-makin basics and php, whatever that is.

   so, i checked my domain name. it's available. but whomto should i go where for web hosting? what do you use?

   lurve,

   tiff

Comments are closed.
