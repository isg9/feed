---
title: 'dynamic dispatch: a followup — wingolog'
url: https://wingolog.org/archives/2008/10/19/dynamic-dispatch-a-followup
published: "2008-10-19T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/10/19/dynamic-dispatch-a-followup
---

# dynamic dispatch: a followup — wingolog

## [dynamic dispatch: a followup](/archives/2008/10/19/dynamic-dispatch-a-followup)

19 October 2008 10:40 PM

- [guile](/tags/guile)
- [goops](/tags/goops)
- [clos](/tags/clos)
- [dispatch](/tags/dispatch)
- [dynamic](/tags/dynamic)
- [self](/tags/self)
- [pic](/tags/pic)
- [polymorphic inline caches](/tags/polymorphic%20inline%20caches)

It seems that the 8-hash technique for dynamic dispatch that I mentioned in my [last essay](//wingolog.org/archives/2008/10/17/dispatch-strategies-in-dynamic-languages) actually has a longer pedigree. At least 10 years before GOOPS' implementation, the always-excellent Gregor Kiczales wrote, with Luis H Rodriguez Jr.:

> If we increase the size class wrappers slightly, we can add more hash seeds to each wrapper. If n is the number of hash seeds stored in each wrapper, we can think of each generic function selecting some number x less than n and using the xth hash seed from each wrapper. Currently we store 8 hash seeds in each wrapper, resulting in very low average probe depths.
>
> The additional hash seeds increase the probability that a generic function will be able to have a low average probe depth in its memoization table. If one set of seeds doesn't produce a good distribution, the generic function can select one of the other sets instead. In effect, we are increasing the size of class wrappers in order to decrease the size of generic function memoization tables. This tradeoff is attractive since typical systems seem to have between three and five times as many generic functions as classes.
>
> [*Efficient method dispatch in PCL*](http://www2.parc.com/csl/groups/sda/publications/papers/Kiczales-Andreas-PCL/for-web.pdf)

So it seems that Mikael Djurfeldt, the GOOPS implementor, appears to have known about CLOS implementation strategies. But it's interesting how this knowledge percolates out -- it's not part of the computer science canon. When you read these papers, it's always "Personal communication from Dave Moon this" and "I know about this Kiczales paper that". (Now you do too.)

Also interesting about the Kiczales paper is the focus on the user, the programmer, in the face of redefinitions -- truly a different culture than the one that is dominant now.

**polymorphic inline caches buzz buzz buzz**

This reference comes indirectly via [Keith Rarick](http://xph.us/), who [writes](//wingolog.org/archives/2008/10/17/dispatch-strategies-in-dynamic-languages#80963571c8979abaf6b085a32e737c9dd73ffd90) to mention a *beautiful* paper by Hölzle, Chambers, and Ungar, introducing *polymorphic inline caches*, a mechanism to dispatch based on runtime types, as GOOPS does.

PICs take dispatch one step further: instead of indirect table lookups as GOOPS does, a PIC is a runtime-generated procedure that performs the lookups directly in code. This difference between data-driven processing and direct execution is the essence of compilation -- compilation pushes all of the caching and branching logic as close to the metal as possible.

Furthermore, PICs can be a source of data as well as a dispatch mechanism:

> The presence of PIC-based type information fundamentally alters the nature of optimization of dynamically-typed object-oriented languages. In “traditional” systems such as the current SELF compiler, type information is scarce, and consequently the compiler is designed to make the best possible use of the type information. This effort is expensive both in terms of compile time and compiled code space, since the heuristics in the compiler are tuned to spend time and space if it helps extract or preserve type information. In contrast, a PIC-based recompiling system has a veritable wealth of type information: every message has a set of likely receiver types associated with it derived from the previously compiled version’s PICs. The compiler’s heuristics and perhaps even its fundamental design should be reconsidered once the information in PICs becomes available \[...\].
>
> [Optimizing Dynamically-Typed Object-Oriented Programming Languages with Polymorphic Inline Caches](http://research.sun.com/self/papers/pics.html)

The salient point is that in latent-typed languages, *all of the static type analysis techniques that we know are insufficient*. Only runtime analysis and runtime recompilation can capture the necessary information for efficient compilation.

Read both of these articles! But if you just read one, make it the Ungar/Chambers/Hölzle -- it is well-paced, clearly-written, and illuminating.

Happy hacking!

## related articles

- [inline cache applications in scheme](/archives/2012/05/29/inline-cache-applications-in-scheme)
- [international lisp conference 2009 -- day one](/archives/2009/03/23/international-lisp-conference-day-one)
- [dispatch strategies in dynamic languages](/archives/2008/10/17/dispatch-strategies-in-dynamic-languages)
- [v8: a tale of two compilers](/archives/2011/07/05/v8-a-tale-of-two-compilers)
- [on-stack replacement in v8](/archives/2011/06/20/on-stack-replacement-in-v8)
- [what does v8 do with that loop?](/archives/2011/06/08/what-does-v8-do-with-that-loop)

### 2 responses

1. [Juho Snellman](http://jsnell.iki.fi) says:[20 October 2008 2:03 AM](#52b58fd8f97e9fdd982ce368314eda760a754db2)

   The percomputation of 8 separate hash seeds is indeed a very clever idea.

   But at least when this was last looked at in the context of SBCL, it turned out that it wasn't a particularily useful optimization. People were asked to run a bit of code that would print the dispatch cache statistics in any long-running Lisp images they had. I can't find the results right now (they're somewhere on the notoriously unsearchable paste.lisp.org), but the results were hugely skewed. Something along the lines of the first hash seed being used for 99% of generic functions, and nobody who ran the code having a case of a gf that was using anything beyond the third seed.

2. Mr Kevin says:[17 January 2014 7:31 PM](#adf3d3068593cc9236a01b00d55a862587df9527)

   The link above is dead. I believe I found the paper at http://www.cs.ucsb.edu/~urs/oocsb/papers/ecoop91.pdf.

Comments are closed.
