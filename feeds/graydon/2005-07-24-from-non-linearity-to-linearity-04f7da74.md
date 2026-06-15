---
title: from non-linearity to linearity
url: https://graydon2.dreamwidth.org/41086.html
published: "2005-07-24T20:39:31Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:41086
---

# from non-linearity to linearity

I've been studying a family of programming language work I've had a hard time naming, until recently. By browsing the delightful [ATTaPL](http://www.amazon.com/exec/obidos/tg/detail/-/0262162288), I've learned that this family of work is called "substructural types", which means "type systems which intentionally omit a structural rule from the simply typed lambda calculus" (exchange, weakening, or contraction). These restrictions essentially mean that some form of *resource accounting* is injected into the type system, or (in more subtle systems) a semilattice or finite automaton of valid "type-state" or "role" transitions is described. Concretely: you don't get to reuse (or fail to use, or copy, or mutate) a variable willy-nilly, only in permitted circumstances. It has a lot of direct implications in terms of designing languages which are free from concurrency problems or garbage collection, or in permitting users to describe resource-use protocols. In general, it's about "making programs typefully-aware of resources".

Most of these systems are research-y, but I'm hopeful the idea will wend its way into mainstream languages *someday*. Variations on the theme include [imperative linear types](http://citeseer.csail.mit.edu/fahndrich02adoption.html), [object typestates](http://citeseer.csail.mit.edu/deline04typestates.html), [role types](http://xxx.lanl.gov/abs/cs.PL/0408013), [unique pointers](http://citeseer.ist.psu.edu/84758.html), [linear-typed assembly language](http://citeseer.ist.psu.edu/hawblitzel04lowlevel.html), [time regions](http://citeseer.ist.psu.edu/632070.html), [alias types](http://citeseer.ist.psu.edu/smith99alias.html), a couple of race-free [dialects](http://www.research.ibm.com/people/d/dfb/papers/Bacon00Guava.ps) of [java](http://www.eecs.umich.edu/~bchandra/publications/oopsla02.pdf), and -- making abysmally clear the agonizing pace of research transfer -- an absolutely *lovely* typestate-checking (and garbage-collection-free) language from the early 90s called [hermes](http://www.research.ibm.com/people/d/dfb/hermes-publications.html), which I only recently ran across.

I also got a little MIDI cable to connect my miserable ugly synthesizer to the computer. This means I can now use [seq24](http://www.filter24.org/seq24/), which is an absolute *riot*, and strongly embraces my preference for *intentionally limited* musical instruments, which force you to make concrete decisions.

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=41086) comments
