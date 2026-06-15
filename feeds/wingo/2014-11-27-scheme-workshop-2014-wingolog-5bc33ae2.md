---
title: scheme workshop 2014 — wingolog
url: https://wingolog.org/archives/2014/11/27/scheme-workshop-2014
published: "2014-11-27T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2014/11/27/scheme-workshop-2014
---

# scheme workshop 2014 — wingolog

## [scheme workshop 2014](/archives/2014/11/27/scheme-workshop-2014)

27 November 2014 5:48 PM

- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [igalia](/tags/igalia)
- [gnu](/tags/gnu)
- [adaptive optimization](/tags/adaptive%20optimization)
- [javascript](/tags/javascript)

I just got back from the US, and after sleeping for 14 hours straight I'm in a position to type about stuff again. So welcome back to the solipsism, France and internet! It is good to see you on a properly-sized monitor again.

I had the enormously pleasurable and flattering experience of being invited to keynote this year's [Scheme Workshop](http://homes.soic.indiana.edu/jhemann/scheme-14/) last week in DC. Thanks to [John Clements](http://www.brinckerhoff.org/clements/home/), [Jason Hemann](http://hemann.pl/), and the rest of the committee for making it a lovely experience.

My talk was on what Scheme can learn from JavaScript, informed by my work in JS implementations over the past few years; you can [download the slides as a PDF](//wingolog.org/pub/scheme-workshop-2014-slides.pdf). I managed to record audio, so here goes nothing:

*55 minutes, [vorbis](http://wingolog.org/pub/scheme-workshop-2014-audio.ogg) or [mp3](http://wingolog.org/pub/scheme-workshop-2014-audio.mp3)*

It helps to follow along with the slides. Some day I'll augment my slide-rendering stuff to synchronize a sequence of SVGs with audio, but not today :)

The invitation to speak meant a lot to me, for complicated reasons. See, Scheme was born out of academic research labs, and to a large extent that's been its spiritual core for the last 40 years. My way to the temple was as a money-changer, though. While working as a teacher in northern Namibia in the early 2000s, fleeing my degree in nuclear engineering, trying to figure out some future life for myself, for some reason I was recording all of my expenses in Gnucash. Like, all of them, petty cash and all. 50 cents for a fat-cake, that kind of thing.

I got to thinking "you know, I bet I don't spend any money on Tuesdays." See, there was nothing really to spend money on in the village besides fat cakes and boiled eggs, and I didn't go into town to buy things except on weekends or later in the week. So I thought that it would be neat to represent that as a chart. Gnucash didn't have such a chart but I knew that they were implemented in Guile, as part of this wave of Scheme consciousness that swept the GNU project in the nineties, and that I should in theory be able to write it myself.

Problem was, I also didn't have internet in the village, at least then, and I didn't know Scheme and I didn't really know Gnucash. I think what I ended up doing was just monkey-typing out something that looked like the rest of the code, getting terrible errors but hey, it eventually worked. I [submitted the code](http://lists.gnucash.org/pipermail/gnucash-devel/2003-March/008796.html), many years ago now, some of the worst code you'll read today, but they did end up incorporating it into Gnucash and to my knowledge that report is still there.

I got more into programming, but still through the back door, so to speak. I had done some free software work before going to Namibia, on [GStreamer](http://gstreamer.freedesktop.org/), and wanted to build a programmable modular synthesizer with it. I read about [Supercollider](http://audiosynth.com/), and decided I wanted to do something like that but with the "unit generators" defined in GStreamer and orchestrated with Scheme. If I knew then that Scheme could be fast, I probably would have started on an entirely different course of things, but that did at least result in gainful employment doing unrelated GStreamer things, if [not a synthesizer](https://web.archive.org/web/20090311133225/http://ambient.2y.net/soundscrape/).

Scheme became my dominant language for writing programs. It was fun, and the need to re-implement a bunch of things wasn't a barrier at all -- rather a fun challenge. After a while, though, speed was becoming a problem. It became apparent that the only way to speed up Guile would be to replace its AST interpreter with a compiler. Thing is, I didn't know how to write one! Fortunately there was [previous work by Keisuke Nishida](http://www.sourceware.org/ml/guile/2000-07/msg00418.html), jetsam from the nineties wave of Scheme consciousness. I read and read that code, mechanically monkey-typed it into compilation, and slowly reworked it into Guile itself. In the end, around 2009, Guile was faster and I found myself its co-maintainer to boot.

Scheme has been a back door for me for work, too. I randomly met [Kwindla Hultman-Kramer](http://allafrica.com/staff/kwindla/) in Namibia, and we found Scheme to be a common interest. Some four or five years later I ended up working for him with the great folks at [Oblong](http://oblong.com/). As my interest in compilers grew, and it grew as I learned more about Scheme, I wanted something closer there, and that's what I've been doing in [Igalia](https://igalia.com/) for the last few years. My first contact there was a former Common Lisp person, and since then many contacts I've had in the JS implementation world have been former Schemers.

So it was a delight when the invitation came to speak (keynote, no less!) the Scheme Workshop, behind the altar instead of in the foyer.

I think it's clear by now that Scheme as a language and a community isn't moving as fast now as it was in 2000 or even 2005. That's good because it reflects a certain maturity, and makes the lore of the tribe easier to digest, but bad in that people tend to ossify and focus on past achievements rather than future possibility. Ehud Lamm quoted Nietzche earlier today on Twitter:

> By searching out origins, one becomes a crab. The historian looks backward; eventually he also believes backward.

So it is with Scheme and Schemers, to an extent. I hope my talk at the conference inspires some young Schemer to make an adaptively optimized Scheme, or to solve the self-hosted adaptive optimization problem. Anyway, as users I think we should end the era of contorting our code to please compilers. Of course some discretion in this area is always necessary but there's little excuse for actively bad code.

Happy hacking with Scheme, and *au revoir*!

## related articles

- [two paths, one peak: a view from below on high-performance language implementations](/archives/2015/11/03/two-paths-one-peak-a-view-from-below-on-high-performance-language-implementations)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [design notes on inline caches in guile](/archives/2018/02/07/design-notes-on-inline-caches-in-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)

### 2 responses

1. tomás zerolo says:[4 December 2014 10:03 AM](#878b5d714d8352f6cc083666d8c04560e04cac25)

   Thanks, Andy for doing all that cool stuff I'd like to do -- and thanks for writing about it!

2. [marty alain](http://epsilonwiki.free.fr/alphawiki_2/) says:[15 December 2014 12:44 PM](#85c3feb9a9f08e9d073a0ea6c6bce270c05747eb)

   Hi Andy Wingo,

   I was in Washington for your keynote in the Scheme'14 workshop. This is what I wrote during this workshop : http://epsilonwiki.free.fr/epsilonwiki/index.php?view=wash\_20141119. Well, as you can see, it was not very nice from me to write such a things, but it was my feelings. OK, probably because my paper had been rejected. Reading your post in this page today helps me to better understand your work and your investment in nice things. Bravo again. I don't think anymore that you are a "tête à claque"...

   Alain Marty

Comments are closed.
