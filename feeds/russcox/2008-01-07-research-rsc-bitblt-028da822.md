---
title: 'research!rsc: Bitblt'
url: https://research.swtch.com/bitblt
published: "2008-01-07T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/bitblt
---

# research!rsc: Bitblt

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Bitblt Russ Cox January 7, 2008 *research.swtch.com/bitblt* Posted on Monday, January 7, 2008.

In the 1980s, bitmap graphics used a powerful primitive called bitblt. Bitblt combined a rectangle of a source image with a similarly-sized rectangle in a destination image using a boolean function and replaced the destination rectangle with the boolean result.

Dan Ingalls invented bitblt for the Alto workstation developed at Xerox PARC. Rob Pike and Bart Locanthi used bitblt as the graphics primitive and the name for the Blit, the first graphical terminal for Unix.

At the 1984 SIGGRAPH conference, Ingalls, Leo Guibas (also at Xerox PARC), and Pike gave a course on the history, use, and implementation of bitblt. The [course notes from 1984](http://pdos.csail.mit.edu/%7Ersc/pike84bitblt.pdf) (pdf, 68 pages, 5MB) have a wealth of information about bitblt, including area-filling, bitmap rotation, bitmap magnification, implementation via on-the-fly code generation, line-drawing, text manipulation, and more.
