---
title: international lisp conference 2009 -- day one — wingolog
url: https://wingolog.org/archives/2009/03/23/international-lisp-conference-day-one
published: "2009-03-23T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/03/23/international-lisp-conference-day-one
---

# international lisp conference 2009 -- day one — wingolog

## [international lisp conference 2009 -- day one](/archives/2009/03/23/international-lisp-conference-day-one)

23 March 2009 3:36 PM

- [lisp](/tags/lisp)
- [ilc](/tags/ilc)
- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [clojure](/tags/clojure)
- [clos](/tags/clos)
- [goops](/tags/goops)

Greetings from Cambridge! The American one, that is. I'm at the [International Lisp Conference](http://international-lisp-conference.org/), which is an OK set of talks, but an excellent group of almost 200 Lisp hackers. What follows is not strictly journalism, but rather reflections of a more subjective nature.

Sunday was reserved for "tutorials", four sessions of an hour and a half in length, with an expository and relaxed flavor. There were three sessions running at a time, which made for some hard choices.

The first session that I went to was "Monads for the Working Lisp Programmer", which started off by explaining monads, and in the second session looked at their uses. As a working Lisp programmer, I guess it's good to be able to talk down Haskell weenies, but something about the complexity of the "plumbing" doesn't sit right. People keep talking about separating the "plumbing" from the computation -- as if programming somehow involves clogged pipes or something. The presenters did a good job, but "different strokes, different folks". I jetted out after the first session.

Next I decided to drop in on the second session of Pascal Constanza's tutorial on CLOS, generic functions, and the meta-object protocol. It was very clearly presented, and even though I've been working with GOOPS (a CLOS-a-like) for a long time, I came away with some style tips. Also seeing his LispWorks environment was very interesting. So much of programming is workflow and environment, something that often requires you to look over the shoulder of a master. I was a bit frustrated though, as I came with questions but it seemed Pascal had a curriculum he wanted to stick to. I'll corner him later.

Actually, that's the best thing about this conference, this ability to corner smart people. So far they don't have that harried, ragged look yet.

In the afternoon, I went to Rich Hickey's Clojure tutorials (parts 2 and 3). What a treat. Hickey's instincts and good taste as a language designer are evident. All languages deserve to have his persistent data structures. His discussion of state and identity was particularly lucid, as well. (Hickey say that the talk on state and time was basically the same one he gave at InfoQ.)

Personally I don't care for the JVM, and I feel Clojure's worth as a general-purpose language is actually hampered by the lack of tail recursion, but I really think that his design ideas should percolate out to all the dynamic languages of the world.

Now it's Monday morning, and I'm sitting in on the plenary technical talks and software demonstrations. They're OK, but thus far yesterday had a better feel. I'm very much looking forward to the afternoon sessions.

## related articles

- [international lisp conference 2009, day three, part 1](/archives/2009/03/25/international-lisp-conference-day-three-part-)
- [international lisp conference -- day two](/archives/2009/03/24/international-lisp-conference-day-two)
- [dispatch strategies in dynamic languages](/archives/2008/10/17/dispatch-strategies-in-dynamic-languages)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [quasiconf 2012: lisp @ froscon](/archives/2012/08/15/quasiconf-2012-lisp-froscon)

### One response

1. Patrick says:[10 September 2011 11:48 PM](#f4f91aabed19d1932e01bba97fc72ce76fd39883)

   What would a Clojure-like collections/sequences implementation (e.g. list, vector, map, set) look like in Guile Scheme, in the sense of those data structures all responding to a common interface of count, first, rest, map, fold/reduce, and various other functions? Would they be objects and multimethods in GOOPS? (scheme/clojure newbie)

Comments are closed.
