---
title: foreign interfaces — wingolog
url: https://wingolog.org/archives/2007/05/24/foreign-interfaces
published: "2007-05-24T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2007/05/24/foreign-interfaces
---

# foreign interfaces — wingolog

## [foreign interfaces](/archives/2007/05/24/foreign-interfaces)

25 May 2007 1:57 AM

- [scheme](/tags/scheme)
- [gnome](/tags/gnome)
- [guile](/tags/guile)
- [ffi](/tags/ffi)
- [mintzkov](/tags/mintzkov)

**ffi**

I've been looking at different Schemes in their interactions with C.

Using C to interact with MzScheme: [extreme badness](http://download.plt-scheme.org/doc/insidemz/insidemz-Z-H-1.html#node_chap_1).

On the other hand [their FFI module](http://download.plt-scheme.org/doc/foreign/foreign.html), while very low level, seems doable. Using `foreign` is Scheme programming, which I haven't been doing much of lately. It's very similar to Python's [ctypes](http://docs.python.org/dev/lib/module-ctypes.html) module, which is at one level disappointing, because the necessary information is there to construct wrappers at compile-time. Perhaps MzScheme has some cleverness I don't know about.

I released [guile-gnome-platform 2.15.92](http://article.gmane.org/gmane.lisp.guile.gtk/684). A couple more tweaks and that will be off my brain and I can move up the stack to making a synthesizer, maybe.

I found this in my wordpress today, apparently written a while ago:

**Results 1 - 7 of about 2 for broodrock. (0.15 seconds)**

Am enjoying mintzkov.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [closedgl particle simulation in guile](/archives/2013/02/16/opengl-particle-simulation-in-guile)
- [twenty ten](/archives/2010/01/27/twenty-ten)
- [brought to you by torres viña sol](/archives/2007/11/10/brought-to-you-by-torres-vina-sol)
- [happy birthday gnome!](/archives/2007/08/14/happy-birthday-gnome)
- [documenting language bindings](/archives/2007/07/18/documenting-language-bindings)

Comments are closed.
