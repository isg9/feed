---
title: bread crumbs — wingolog
url: https://wingolog.org/archives/2007/06/08/bread-crumbs
published: "2007-06-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2007/06/08/bread-crumbs
---

# bread crumbs — wingolog

## [bread crumbs](/archives/2007/06/08/bread-crumbs)

8 June 2007 9:21 PM

- [scheme](/tags/scheme)
- [hacks](/tags/hacks)
- [aikido](/tags/aikido)
- [profiling](/tags/profiling)
- [guile](/tags/guile)
- [sailing](/tags/sailing)
- [cairo](/tags/cairo)
- [valgrind](/tags/valgrind)
- [callgrind](/tags/callgrind)
- [charting](/tags/charting)

**hack**

I have been a creature of the machine lately. My current project is Guile profiling, but it proves to have many distracting components. For example:

```
cumulative   self      total
 percent    percent    calls    file:function
   17.81     15.94       4200   gc-card.c:scm_i_sweep_card[/opt/guile/lib/libguile.so.17.0.1]
 1272.79      7.47      74218   eval.c:ceval[/opt/guile/lib/libguile.so.17.0.1]
    7.13      7.13     491863   ???:pthread_getspecific[/lib/libpthread-2.5.so]
    6.19      6.19     191068   ???:pthread_mutex_lock[/lib/libpthread-2.5.so]

```

I wrote a [callgrind format parser](//wingolog.org/pub/callgrind-output-gprof) in Scheme, which produces what to me are more readable summaries. A bit pointless but interesting.

[![](//wingolog.org/software/guile-charting/guile-benchmarks.png)](//wingolog.org/software/guile-charting/)

In the same vein, envious of the [benchmark graphs of bazaar](http://bazaar-vcs.org/Performance/0.15), I made a [hack to make charts in Guile](//wingolog.org/software/guile-charting/), using [Guile-Cairo](http://home.gna.org/guile-cairo/) (of which I released version 1.3.91). I probably wouldn't have done it if my Gnumeric didn't have some kind of endianness problem with its colors, but I'm pleased with how things turned out.

**existence**

Time passes too quickly to write about it! Climbed [pedraforca](http://www.summitpost.org/mountain/rock/150651/pedraforca.html), in a trip in which the mountain gods did smile upon us. Left my camera in the car, what a fool I am.

Last weekend was beautiful, my first time in Italy, meeting up with a friend in Florence. My robot overlords have not allowed me time yet for picture-postage, but they will relent yet.

Not until after this weekend though: the [third year in a row](//wingolog.org/index.php?s=bernath) in which I'll be able to catch [Peter Bernath](http://www.floridaaikikai.com/staff.htm), and then in the afternoons I have my first sailing lessons. A bit absurd to fill the time so much, but it will be a pleasant change from the bar--recover-from-bar cycle.

## related articles

- [visualizing statistical profiles with chartprof](/archives/2009/02/09/visualizing-statistical-profiles-with-chartprof)
- [an in-depth look at the performance of guile's web server](/archives/2012/03/08/an-in-depth-look-at-the-performance-of-guiles-web-server)
- [\-\\*\- text -\*-](/archives/2007/05/22/text)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [twenty ten](/archives/2010/01/27/twenty-ten)

### One response

1. Rob says:[8 June 2007 11:37 PM](#634)

   Your hacks are pretty cool, they make me want to learn lisp.

Comments are closed.
