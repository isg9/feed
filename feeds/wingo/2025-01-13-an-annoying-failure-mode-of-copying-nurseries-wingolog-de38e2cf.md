---
title: an annoying failure mode of copying nurseries — wingolog
url: https://wingolog.org/archives/2025/01/13/an-annoying-failure-mode-of-copying-nurseries
published: "2025-01-13T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2025/01/13/an-annoying-failure-mode-of-copying-nurseries
---

# an annoying failure mode of copying nurseries — wingolog

## [an annoying failure mode of copying nurseries](/archives/2025/01/13/an-annoying-failure-mode-of-copying-nurseries)

13 January 2025 9:59 AM

- [gc](/tags/gc)
- [evacuation](/tags/evacuation)
- [whippet](/tags/whippet)
- [nurseries](/tags/nurseries)
- [block-structured heaps](/tags/block-structured%20heaps)
- [guile](/tags/guile)
- [igalia](/tags/igalia)
- [generational collection](/tags/generational%20collection)
- [garbage collection](/tags/garbage%20collection)

I just found a funny failure mode in the [Whippet](https://github.com/wingo/whippet) garbage collector and thought readers might be amused.

Say you have a semi-space nursery and a semi-space old generation. Both are block-structured. You are allocating live data, say, a long linked list. Allocation fills the nursery, which triggers a minor GC, which decides to keep everything in the nursery another round, because that’s policy: Whippet gives new objects another cycle in which to potentially become unreachable.

This causes a funny situation!

Consider that the first minor GC doesn’t actually free anything. But, like, *nothing*: it’s impossible to allocate anything in the nursery after collection, so you run another minor GC, which promotes everything, and you’re back to the initial situation, wash rinse repeat. Copying generational GC is strictly a pessimization in this case, with the additional insult that it doesn’t preserve object allocation order.

Consider also that because [copying collectors with block-structured heaps
are
unreliable](https://wingolog.org/archives/2024/07/10/copying-collectors-with-block-structured-heaps-are-unreliable), any one of your minor GCs might require more blocks after GC than before. Unlike in the case of a major GC in which this essentially indicates out-of-memory, either because of a mutator bug or because the user didn’t give the program enough heap, for minor GC this is just what we expect when allocating a long linked list.

Therefore we either need to allow a minor GC to allocate fresh blocks – very annoying, and we have to give them back at some point to prevent the nursery from growing over time – or we need to maintain some kind of margin, corresponding to the maximum amount of fragmentation. Or, or, we allow evacuation to fail in a minor GC, in which case we fall back to promotion.

Anyway, I am annoyed and amused and I thought others might share in one or the other of these feelings. Good day and happy hacking!

## related articles

- [whippet update: faster evacuation, eager sweeping of empty blocks](/archives/2024/08/25/whippet-update-faster-evacuation-eager-sweeping-of-empty-blocks)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)
- [parallel ephemeron tracing](/archives/2023/01/24/parallel-ephemeron-tracing)
- [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)

Comments are closed.
