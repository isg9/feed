---
title: fold, xml, presentations — wingolog
url: https://wingolog.org/archives/2007/07/11/fold-xml-presentations
published: "2007-07-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2007/07/11/fold-xml-presentations
---

# fold, xml, presentations — wingolog

## [fold, xml, presentations](/archives/2007/07/11/fold-xml-presentations)

11 July 2007 4:56 PM

- [guile](/tags/guile)
- [svg](/tags/svg)
- [presentations](/tags/presentations)
- [xml](/tags/xml)
- [fold](/tags/fold)
- [org mode](/tags/org%20mode)
- [emacs](/tags/emacs)
- [jornadespl](/tags/jornadespl)

Good afternoon, world. I have a bit to write about; we'll see how much of it manages to escape my brain into this page.

**fold, xml, presentations**

To recap a bit, I've been trying to avoid powerpoint-style applications, especially OpenOffice. OpenOffice is good for some people but it hurts my brain.

After poking about for a while, I decided to make presentations with SVG graphics, with each layer being one slide. I made presentations in Inkscape, and then wrote [a tool](//wingolog.org/pub/svg-split) (in Guile) to split the SVG file into layers, and then make a PDF out of that.

However, the process was not satisfying: it was too hard to enter in the text, and I always had to think about alignment. I wanted to be able to make an SVG presentation programmatically. It turns out that the what has become the standard Scheme XML manipulation combinator, [`pre-post-order`](http://okmij.org/ftp/papers/SXSLT.ps.gz), is not well-suited to performing layout operations.

This is because `pre-post-order` effectively maps across XML child elements instead of single-threadedly passing a layout seed through the operation. So I started looking how to recast `pre-post-order` as an extended tree fold (foldts from Oleg's SSAX paper). I wrote up the results in a paper that I submitted to [SFP 2007](http://sfp2007.ift.ulaval.ca/), [Applications of Fold to XML Transformation](//wingolog.org/pub/fold-and-xml-transformation.pdf). We'll see if it gets accepted, it's a bit of an odd endeavor.

Of course, the next time I gave a presentation, I would have to figure out how to use the algorithms that I had worked on, otherwise it would be wasted effort. The occasion was the [Jornades del Programari Lliure](http://jornadespl.org) last week in Girona. Following a [recommendation by a kind commenter](//wingolog.org/archives/2007/02/19/presentations-and-svg#comment-292), I decided to write the presentation in Emacs' Org Mode, then transform its XML export into the presentation language that I defined, then use the algorithms that I presented to transform that into SVG. After the presentation, I cleaned up the scripts I had, and packaged them into a library that I just released, [Guile-Present](//wingolog.org/software/guile-present/).

Guile-Present is OK. You can write your presentation in Emacs, with all of the fluidity that comes with plain text:

```
# -*- mode: org; fill-column: 34 -*-
#+TITLE: Medios de (re)producción: GStreamer

* Plan

GStreamer: Multimedia para todos

@fluendo.com: Qué es lo que
estamos haciendo

* GStreamer

** ¿Qué es GStreamer?

 * Una librería multimedia y un
   conjunto de plugins

```

Then, just run [org-to-pdf-presentation](//wingolog.org/software/guile-present/doc/org-to-pdf-presentation/) on your org file, and you have a PDF ready to present ( [example](//wingolog.org/pub/jornadespl-es.pdf); [example in english](//wingolog.org/pub/jornadespl-en.pdf)). The PDF output is clean but not pretty. I hope to improve that in the future (read: the next time I give a presentation). I also need to fix some rendering bugs in the HTML documentation, but time is with me these days.

## related articles

- [presentations and svg](/archives/2007/02/19/presentations-and-svg)
- [sir talks-a-lot](/archives/2023/12/12/sir-talks-a-lot)
- [recent developments in guile](/archives/2010/04/02/recent-developments-in-guile)
- [wikipedia & guile](/archives/2009/10/01/wikipedia-guile)
- [a brief history of guile](/archives/2009/01/07/a-brief-history-of-guile)
- [allocate, memory (part one of n)](/archives/2008/04/10/allocate-memory-part-one-of-n)

### 7 responses

1. Tobias says:[11 July 2007 5:31 PM](#875)

   The Link to the last PDF is wrong. Its jornadespl-en.pdf and not jornadaspl-en.pdf.

2. dr d says:[11 July 2007 6:18 PM](#877)

   and how can this be extended to include LaTeXy like formulas?

3. yosch says:[11 July 2007 6:44 PM](#878)

   Neat approach. I like using http://code.100allora.it/pdfcube or http://keyjnote.sourceforge.net/ for the added eye-candy and usability effects.

4. [Marius Gedminas](http://gedmin.as) says:[12 July 2007 0:11 AM](#879)

   Have you ever tried MagicPoint?

5. Rob says:[12 July 2007 1:07 AM](#880)

   Thanks for the links yosch! Keyjnote looks awesome.

6. [wingo](http://wingolog.org/) says:[12 July 2007 2:54 AM](#881)

   Tobias: links fixed, thanks :)

7. phineas gage says:[13 July 2007 10:25 AM](#893)

   Maybe I just don't see it, but what does this have to do with sailing?

Comments are closed.
