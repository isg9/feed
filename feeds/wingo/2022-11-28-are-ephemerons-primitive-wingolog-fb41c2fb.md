---
title: are ephemerons primitive? — wingolog
url: https://wingolog.org/archives/2022/11/28/are-ephemerons-primitive
published: "2022-11-28T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2022/11/28/are-ephemerons-primitive
---

# are ephemerons primitive? — wingolog

## [are ephemerons primitive?](/archives/2022/11/28/are-ephemerons-primitive)

28 November 2022 9:11 PM

- [mark functions](/tags/mark%20functions)
- [bdw](/tags/bdw)
- [garbage collection](/tags/garbage%20collection)
- [ephemerons](/tags/ephemerons)
- [finalizers](/tags/finalizers)
- [finalization](/tags/finalization)
- [guardians](/tags/guardians)
- [weak references](/tags/weak%20references)
- [js](/tags/js)
- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [igalia](/tags/igalia)

Good evening :) A quick note, tonight: I’ve long thought that [ephemerons](https://wingolog.org/archives/2022/10/31/ephemerons-and-finalizers) are primitive and can’t be implemented with mark functions and/or finalizers, but today I think I have a counterexample.

For context, one of the goals of the [GC implementation I have been
working on on](https://github.com/wingo/whippet-gc) is to replace [Guile](https://gnu.org/s/guile)‘s current use of the [Boehm-Demers-Weiser (BDW) conservative
collector](https://github.com/ivmai/bdwgc). Of course, changing a garbage collector for a production language runtime is risky, and for Guile one of the mitigation strategies for this work is that the new collector is behind an abstract API whose implementation can be chosen at compile-time, without requiring changes to user code. That way we can first switch to BDW-implementing-the-new-GC-API, then switch the implementation behind that API to something else.

Abstracting GC is a tricky problem to get right, and I thank the [MMTk
project](https://www.mmtk.io/) for showing that this is possible – you have user-facing APIs that need to be implemented by concrete collectors, but also extension points so that the user can provide some compile-time configuration too, for example to provide field-tracing visitors that take into account how a user wants to lay out objects.

Anyway. As we discussed last time, ephemerons are usually have explicit support from the GC, so we need an ephemeron abstraction as part of the abstract GC API. The question is, can BDW-GC provide an implementation of this API?

I think the answer is “yes, but it’s very gnarly and will kill performance so bad that you won’t want to do it.”

**the contenders**

Consider that the primitives that you get with BDW-GC are custom mark functions, run on objects when they are found to be live by the mark workers; disappearing links, a kind of weak reference; and finalizers, which receive the object being finalized, can allocate, and indeed can resurrect the object.

BDW-GC’s finalizers are a powerful primitive, but not one that is useful for implementing the “conjunction” aspect of ephemerons, as they cannot constrain the marker’s idea of graph connectivity: a finalizer can only prolong the life of an object subgraph, not cut it short. So let’s put finalizers aside.

Weak references have a tantalizingly close kind of conjunction property: if the weak reference itself is alive, *and* the referent is also otherwise reachable, then the weak reference can be dereferenced. However this primitive only involves the two objects E and K; there’s no way to then condition traceability of a third object V to E and K.

We are left with mark functions. These are an extraordinarily powerful interface in BDW-GC, but somewhat expensive also: not inlined, and going against the grain of what BDW-GC is really about (heaps in which the majority of all references are conservative). But, OK. They way they work is, your program allocates a number of GC “kinds”, and associates mark functions with those kinds. Then when you allocate objects, you use those kinds. BDW-GC will call your mark functions when tracing an object of those kinds.

Let’s assume firstly that you have a kind for ephemerons; then when you go to mark an ephemeron E, you mark the value V only if the key K has been marked. Problem solved, right? Only halfway: you also have to handle the case in which E is marked first, then K. So you publish E to a global hash table, and... well. You would mark V when you mark a K for which there is a published E. But, for that you need a hook into marking V, and V can be any object...

So now we assume additionally that *all* objects are allocated with user-provided custom mark functions, and that all mark functions check if the marked object is in the published table of pending ephemerons, and if so marks values. This is essentially what a proper ephemeron implementation would do, though there are some optimizations one can do to avoid checking the table for each object before the mark stack runs empty for the first time. In this case, yes you can do it! Additionally if you register disappearing links for the K field in each E, you can know if an ephemeron E was marked dead in a previous collection. Add a pre-mark hook (something BDW-GC provides) to clear the pending ephemeron table, and you are in business.

**yes, but no**

So, it is possible to implement ephemerons with just custom mark functions. I wouldn’t want to do it, though: missing the mostly-avoid-pending-ephemeron-check optimization would be devastating, and really what you want is support in the GC implementation. I think that for the BDW-GC implementation in [whippet](https://github.com/wingo/whippet-gc) I’ll just implement weak-key associations, in which the value is always marked strongly unless the key was dead on a previous collection, using disappearing links on the key field. That way a (possibly indirect) reference from a value V to a key K can indeed keep K alive, but oh well: it’s a conservative approximation of what should happen, and not worse than what Guile has currently.

Good night and happy hacking!

## related articles

- [ephemerons and finalizers](/archives/2022/10/31/ephemerons-and-finalizers)
- [finalizers, guardians, phantom references, et cetera](/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera)
- [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)

### 3 responses

1. [Daphne Preston-Kendal](http://dpk.io/) says:[29 November 2022 3:34 AM](#99d531c5551f68118d29fa012d04abf35775902a)

   Apropos ephemerons as primitive, I’d be interested to know if my implementation for Gerbil based on Gambit’s wills (boxes providing weak referencing and finalization for one value each) is somehow incorrect: https://github.com/vyzo/gerbil/blob/99e7f6cbb1b00e0b490a7e8c3f1debf9f10c7ef7/src/std/srfi/124.ss

   How it works conceptually (using SRFI 124 terminology) is that an ephemeron consists of a boolean broken flag, a will for the key, and a will for the datum. The will for the datum is merely there so that cycles from the datum back to the key are handled properly, i.e. weakly — as long as the broken flag isn’t set (which would happen if the key was finalized before the datum), it simply resurrects the ephemeron if the GC attempts to execute its action. The will for the key actually carries out the breaking, setting the broken flag and nullifying the datum.

   This probably falls under ‘you wouldn’t want to do this too much in practice’ performance-wise, but since the previous SRFI 124 implementation wasn’t actually weak referencing, this takes the only weak referencing primitive Gambit has to implement ephemerons with (hopefully) correct semantics. I hacked it together quite quickly so there might be some flaw in it which both vyzo and I failed to see. Since you didn’t think finalization was enough to correctly implement them, maybe you already know what’s wrong here.

2. [wingo](https://wingolog.org) says:[29 November 2022 9:04 AM](#ecaa2e76c42b689d82471b408a1e84493f981013)

   Humm!!! Thanks for the note Daphne, this is an elegant solution. I think it may have a problem though; consider:

   ```
   (define x (list 'x))
   (define y (list 'y))
   (define ex (make-ephemeron x y))
   (define ey (make-ephemeron y 42))
   (set! y #f)
   (##gc)
   (eqv? 42 (ephemeron-datum ey))

   ```

   Let’s assume that after the GC, all the will actions have time to run. At the GC, y is only weak-reachable and so its will execute: the one in ex, which is fine, but also the one in ey, marking ey dead. Does this sound right to you?

3. [Daphne Preston-Kendal](http://dpk.io/) says:[2 December 2022 8:29 AM](#be5cce9d433db97ee060af061b4a713fc726871d)

   Ahh, yes, that is an issue. I think it could be fixed by making ephemerons somehow aware of one another instead of all independent, but that would be something of a hack. I’ll think about it.

   But probably it would be better to just encourage the Gambit developers to switch from single-slot wills to XEmacs/Haskell-style ephemerons with an extra finalizer slot. (I also have it on my to-do list to propose an extension to SRFI 124 with such finalizer procedures.)

Comments are closed.
