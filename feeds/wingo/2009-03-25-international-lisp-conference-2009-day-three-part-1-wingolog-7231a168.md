---
title: international lisp conference 2009, day three, part 1 — wingolog
url: https://wingolog.org/archives/2009/03/25/international-lisp-conference-day-three-part-
published: "2009-03-25T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/03/25/international-lisp-conference-day-three-part-
---

# international lisp conference 2009, day three, part 1 — wingolog

## [international lisp conference 2009, day three, part 1](/archives/2009/03/25/international-lisp-conference-day-three-part-)

25 March 2009 2:49 PM

- [lisp](/tags/lisp)
- [ilc](/tags/ilc)
- [guile](/tags/guile)
- [scheme](/tags/scheme)

[Another day](//wingolog.org/archives/2009/03/24/international-lisp-conference-day-two), [another parenthetical note](//wingolog.org/archives/2009/03/23/international-lisp-conference-day-one). It's probably worth repeating that these are just personal impressions, take them with a salt lick.

François-René Rideau spoke about [XCVB](http://common-lisp.net/project/xcvb/), a packaging system for Common Lisp. Slides [here](http://common-lisp.net/project/xcvb/doc/ilc09-xcvb-slides.html). From what I gather, XCVB tries to make separate compilation of Lisp libraries more robust via mechanisms that isolate compile-time side-effects.

More interesting was where he headed at the end of his talk, and expanded on in a [lightning talk](http://fare.tunes.org/computing/fare-ilc09-lightning.html): that the things that we do are intertwined with the stories we tell others and ourselves. The bad stories are about things. The better stories are about people making things.

Next up was a presentation by Gary King from Franz Inc., about [AllegroGraph](http://agraph.franz.com/), a massive RDF triple-store capable of storing and searching *billions and billions* of triples. Specifically, as far as I could tell, it was about adding temporal and geospatial capabilities to queries -- so you could ask "Of the people that Bob called last week, who has a friend within 5 miles of the Boston area?"

It was quite impressive, but it makes me queasy too. Who has billions and billions of facts and needs to query them? Well, the CIA, the NSA, and similar organizations. Others too of course, epidemiologists for example, but that doesn't ease the quease.

Next up, something totally different. John Hastings lives in Nebraska and has a problem with grasshoppers -- they tend to eat up all of the grass that cattle graze on. Ranchers out there would douse their land with chemicals at any word of a grasshopper outbreak. But that's not good for the land, and it's also expensive. So he wrote an expert system to help ranchers say what the conditions are where they live, in Lisp, and help them decide what to do. The program gets deployed into peoples' web browsers via one of the Lisp-on-the-JVM projects. Voilà fewer chemicals!

Next, a talk by an engineer at Cadence Design Systems, on how they use Lisp to lay out certain kinds of electronics called "systems-in-a-package", such as the circuit boards and the amplifiers and conditioners and such. His point appeared to be that giving the user a way to extend and specialize certain designs via generic functions lets them deliver applications without source, but still allow some extensibility. I didn't care for the "freedom subtracted" aspect, but seeing more about chip design processes was interesting.

Jorge Vallejos from the Vrije Universiteit in Brussels took the floor next. His talk was on distributed computing within the Actors paradigm, whereby objects on different machines can reference and send messages to each other. The interesting part was how he used actors as containers for unadorned objects, and used generic functions to "invoke" remote objects. The meta-object protocol allowed him to extend base Lisp to have remote asynchronous procedure calls, while keeping a natural Lisp feel. It was fine, but something about RPC as a distributed programming metaphor doesn't sit right with me. I probably missed something.

Didier Verna followed, with a delightful talk comparing the speed of Common Lisp and C++. He works at a lab in France that does lots of numerical processing on images, and is the only Lisper in a roomful of C++ wizards. The funny thing is that he says he doesn't care about speed, because things are fast enough, but that he was tired of getting shit from his C++-loving coworkers, so he decided to do some measuring.

This guy is serious. This is his second paper in a projected series of 5 or so; his last [installment](http://www.lrde.epita.fr/~didier/research/verna.06.ecoop-slides.pdf) showed that Lisp and C are comparable in speed. This talk compared instantiation time for C++ objects to instantiation of Lisp structures and CLOS objects, showing again that Lisp was comparable in speed with C++. Unfortunately the paper and slides don't appear to be online; I'll ask him about this.

Also, Verna gave a lightning talk in which he explained that Lisp is Jazz and Jazz is Lisp. When asked by the audience about other things, he explained that they were Jazz too. Also Lisp is Aikido, and [buy his CD](http://www.didierverna.com/).

\\* \\* \\*

In a later post, I'll get to Sussman's talk on evolvability and robust design. It will take some thinking to summarize.

## related articles

- [international lisp conference -- day two](/archives/2009/03/24/international-lisp-conference-day-two)
- [international lisp conference 2009 -- day one](/archives/2009/03/23/international-lisp-conference-day-one)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [quasiconf 2012: lisp @ froscon](/archives/2012/08/15/quasiconf-2012-lisp-froscon)
- [the very merry month of may](/archives/2009/05/10/the-very-merry-month-of-may)
- [dispatch strategies in dynamic languages](/archives/2008/10/17/dispatch-strategies-in-dynamic-languages)

### 2 responses

1. Nathan Myers says:[25 March 2009 6:32 PM](#5f24538333f57293d7403706d1d2669f715afc08)

   Re Didier Verna reporting Lisp achieving image processing performance of at 60% of C ... that's a very low bar. People used to use Fortran because it was much faster than C; now they use C++ because it's faster than Fortran. Fortran didn't get slower. It's no surprise when you say Verna doesn't care about speed -- he also doesn't know about speed.

   I don't know how pretending is supposed to benefit the Lisp community. Maybe it's just traditional.

2. Ricky Youngblood says:[1 May 2009 10:41 PM](#6b9b5bdbf21fdd82d22d6973112d8677f9619636)

   The masses miss you.

Comments are closed.
