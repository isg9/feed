---
title: introducing griddy — wingolog
url: https://wingolog.org/archives/2008/07/31/introducing-griddy
published: "2008-07-31T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/07/31/introducing-griddy
---

# introducing griddy — wingolog

## [introducing griddy](/archives/2008/07/31/introducing-griddy)

31 July 2008 2:11 PM

- [window manager](/tags/window%20manager)
- [clutter](/tags/clutter)
- [guile](/tags/guile)
- [hack](/tags/hack)

[![](//wingolog.org/pub/griddy-repl-thumbnail.png)](//wingolog.org/pub/griddy.ogg)

*5m50s, 2.5 MB, Theora*

The linked video shows my hackbaby of the last couple months, a hackable 3D compositing window manager written in Scheme. It's not much code, but it's rather fun. I'm still not dogfooding with it yet -- you can see a pause around minute 4:30 or so where I have to unstick keys that Xephyr thinks are pressed.

And it's still a bit hard to compile, because you need bzr guile-gnome, and the latest guile-clutter release, and a guile 1.8 compiled as --with-threads if you want the emacs integration. But if you want to join in on the fun, or read some interesting code, check:

```
bzr branch http://wingolog.org/bzr/griddy

```

## related articles

- [twenty ten](/archives/2010/01/27/twenty-ten)
- [the good hack](/archives/2009/07/26/the-good-hack)
- [dispatch strategies in dynamic languages](/archives/2008/10/17/dispatch-strategies-in-dynamic-languages)
- [so you want to build a compositor](/archives/2008/07/26/so-you-want-to-build-a-compositor)
- [guile ? clutter ? quoi ?](/archives/2008/07/11/guile-clutter-quoi-)
- [monday evening dispatches](/archives/2007/09/24/monday-evening-dispatches)

### 12 responses

1. Martin says:[31 July 2008 4:20 PM](#67cc7dd4a481641f46eb46bb12a8ac77d9c30a4e)

   Kind of like ratpoison?

   http://www.nongnu.org/ratpoison/

2. [wingo](http://wingolog.org/) says:[31 July 2008 4:33 PM](#aa238204f1e867e7eb2b53b047628ebe58acbd16)

   @martin: More like StumpWM, ratpoison's successor, but with compositing.

3. Martin says:[31 July 2008 5:21 PM](#e850e37ebd72ea03a5e7034cc83e9c3c01a115c0)

   Cool, didn't know about that one. Easy access to search and execution a'la Gnome Launch Box would probably integrate well with a wm like this.

4. [wingo](http://wingolog.org/) says:[31 July 2008 6:20 PM](#cd9be4f3873e4975105956b602ceeee3aba1cc05)

   Yeah, you're totally right. One of my goals with this is to make it really easy to add such behaviours, in users' branches, or in their .griddy files (not yet implemented).

   For example the concept of organizing applications in a grid is something that is built off of the primitives -- you can replace that if you want, if you find something better. I want to make it easy for people (read: me :) to hack up the things that I want, without imposing them on others, but making them available -- and to have taste in the defaults.

   At some point I think I want to implement tiled windows like that family of wm's (ion, awesome, stumpwm, ratpoison, etc) does. But while I'm hacking it up, I should be able to do so without restarting the window manager. That kind of workflow pleases me immensely.

5. [wingo](http://wingolog.org/) says:[31 July 2008 6:21 PM](#94d3118dcddd717aa9ea6d241745724cead6ca0b)

   (basically i want emacs)

6. Jason says:[31 July 2008 7:10 PM](#ed33788bcfe7613abe7579ec555ecaa9a865d3b1)

   That's pretty nifty. I'm currently a Gnome and Stumpwm user, which is kind of a strange combination. It'll be interesting to see what a lisp-based tiling window manager by a Gnome developer ends up looking like.

7. Zeeshan Ali says:[31 July 2008 10:00 PM](#7e2710db712c393e36f2cf63698aea5ad0b5d9bb)

   Your coolness level has exceeded far beyond what is allowed under the law.

8. Chris Parker says:[1 August 2008 4:03 AM](#6989385d160d52a885b31ef12dcbbe567bd82e27)

   Please keep working on this - this is exactly what I have been looking for!

   I will send you money, buy you beer, do whatever. A real lispy window manager with a REAL repl would be amazing, and this looks amazing!

   Now if Gimp would goto a single-window mode for the rare times I have to use it!

9. [Zajcev Evgeny](http://xwem.org) says:[1 August 2008 10:12 AM](#398d27372cef2d912b4ac8ef1b783bed0c0a59eb)

   Hello, nice thing! But have you seen xwem window manager? it is fully integrated into (S)XEmacs (i.e. written in emacs lisp). It is possible to create full-featured (S)XEmacsen desktop environment using xwem. Currently it does not run under GNU Emacs, due to some (S)XEmacsen features used in xwem implementation, however xlib already ported to GNU Emacs and seems to work. If you are also, along griddy, interested in hacking xwem for GNU Emacs please e-mail me and cc to RMS.

   Thanks, i'm looking forward for griddy updates.

10. Jonas says:[1 August 2008 1:48 PM](#c11d72ac67b0487cdae2f5710eac12af36991dc8)

    Off topic: What app did you use to record the video?

11. [wingo](http://wingolog.org/) says:[1 August 2008 3:22 PM](#317f217faaf6ba6b97d52f5c90f5676f5c2f5581)

    @Jonas: Istanbul, from Zaheer Merali. I did the demo in a Xephyr, so I told it to capture output just from the xephyr window. It was just pointing and clicking, something I found to be quite pleasant.

    I tried to record sound but failed; in retrospect I think the failure was due to my mixer settings.

12. Fox Mulder says:[1 August 2008 8:35 PM](#09714dc214d6306c18a0b56f79abaf9e043d0607)

    This window manager is exactly what I want from my X server on cool Gentoo system. I wait some betas and believe that this project don't die :)

    Good Luck!

Comments are closed.
