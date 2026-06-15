---
title: 'whippet lab notebook: untagged mallocs, bis — wingolog'
url: https://wingolog.org/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis
published: "2025-03-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis
---

# whippet lab notebook: untagged mallocs, bis — wingolog

## [whippet lab notebook: untagged mallocs, bis](/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis)

7 March 2025 1:47 PM

- [whippet](/tags/whippet)
- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [guile](/tags/guile)
- [bdw](/tags/bdw)
- [igalia](/tags/igalia)
- [conservative roots](/tags/conservative%20roots)
- [conservative tracing](/tags/conservative%20tracing)

[Earlier this
week](https://wingolog.org/archives/2025/03/04/whippet-lab-notebook-on-untagged-mallocs) I took an inventory of how [Guile](https://gnu.org/s/guile) uses the Boehm-Demers-Weiser (BDW) garbage collector, with the goal of making sure that I had replacements for all uses lined up in [Whippet](https://github.com/wingo/whippet). I categorized the uses into seven broad categories, and I was mostly satisfied that I have replacements for all except the last: I didn’t know what to do with untagged allocations: those that contain arbitrary data, possibly full of pointers to other objects, and which don’t have a header that we can use to inspect on their type.

But now I do! Today’s note is about how we can support untagged allocations of a few different kinds in Whippet’s [mostly-marking
collector](https://github.com/wingo/whippet/blob/main/doc/collector-mmc.md).

### inside and outside

Why bother supporting untagged allocations at all? Well, if I had my way, I wouldn’t; I would just slog through Guile and fix all uses to be tagged. There are only a finite number of use sites and I could get to them all in a month or so.

The problem comes for uses of [`scm_gc_malloc`](https://www.gnu.org/software/guile/docs/docs-2.0/guile-ref/Memory-Blocks.html) from outside `libguile` itself, in C extensions and embedding programs. These users are loathe to adapt to any kind of change, and garbage-collection-related changes are the worst. So, somehow, we need to support these users if we are not to break the Guile community.

### on intent

The problem with `scm_gc_malloc`, though, is that it is missing an expression of intent, notably as regards tagging. You can use it to allocate an object that has a tag and thus can be traced precisely, or you can use it to allocate, well, anything else. I think we will have to add an API for the tagged case and assume that anything that goes through `scm_gc_malloc` is requesting an untagged, conservatively-scanned block of memory. Similarly for `scm_gc_malloc_pointerless`: you could be allocating a tagged object that happens to not contain pointers, or you could be allocating an untagged array of whatever. A new API is needed there too for pointerless untagged allocations.

### on data

Recall that the mostly-marking collector can be built in a number of different ways: it can support conservative and/or precise roots, it can trace the heap precisely or conservatively, it can be generational or not, and the collector can use multiple threads during pauses or not. Consider a basic configuration with precise roots. You can make tagged pointerless allocations just fine: the trace function for that tag is just trivial. You would like to extend the collector with the ability to make *untagged* pointerless allocations, for raw data. How to do this?

Consider first that when the collector goes to trace an object, it can’t use bits inside the object to discriminate between the tagged and untagged cases. Fortunately though [the main space of the mostly-marking collector has one metadata byte for each 16 bytes of
payload](https://github.com/wingo/whippet/blob/main/src/nofl-space.h#L239). Of those 8 bits, 3 are used for the mark (five different states, allowing for future concurrent tracing), two for the [precise
field-logging write
barrier](https://wingolog.org/archives/2024/10/03/preliminary-notes-on-a-nofl-field-logging-barrier), one to indicate whether the object is pinned or not, and one to indicate the end of the object, so that we can determine object bounds just by scanning the metadata byte array. That leaves 1 bit, and we can use it to indicate untagged pointerless allocations. Hooray!

However there is a wrinkle: when Whippet decides the it should evacuate an object, it tracks the evacuation state in the object itself; the embedder has to provide an implementation of a [little state machine](https://github.com/wingo/whippet/blob/main/api/gc-embedder-api.h#L57-L64), allowing the collector to detect whether an object is forwarded or not, to claim an object for forwarding, to commit a forwarding pointer, and so on. We can’t do that for raw data, because all bit states belong to the object, not the collector or the embedder. So, we have to set the “pinned” bit on the object, indicating that these objects can’t move.

We could in theory manage the forwarding state in the metadata byte, but we don’t have the bits to do that currently; maybe some day. For now, untagged pointerless allocations are pinned.

### on slop

You might also want to support untagged allocations that contain pointers to other GC-managed objects. In this case you would want these untagged allocations to be scanned conservatively. We can do this, but if we do, it will pin all objects.

Thing is, conservative stack roots is a kind of a sweet spot in language run-time design. You get to avoid constraining your compiler, you avoid a class of bugs related to rooting, but you can still support compaction of the heap.

How is this, you ask? Well, consider that you can move any object for which we can precisely enumerate the incoming references. This is trivially the case for precise roots and precise tracing. For conservative roots, we don’t know whether a given edge is really an object reference or not, so we have to conservatively avoid moving those objects. But once you are done tracing conservative edges, any live object that hasn’t yet been traced is fair game for evacuation, because none of its predecessors have yet been visited.

But once you add conservatively-traced objects back into the mix, you don’t know when you are done tracing conservative edges; you could always discover another conservatively-traced object later in the trace, so you have to pin everything.

The good news, though, is that we have gained an easier migration path. I can now shove Whippet into Guile and get it running even before I have removed untagged allocations. Once I have done so, I will be able to allow for compaction / evacuation; things only get better from here.

Also as a side benefit, the mostly-marking collector’s heap-conservative configurations are now faster, because we have metadata attached to objects which allows tracing to skip known-pointerless objects. This regains an optimization that BDW has long had via its `GC_malloc_atomic`, used in Guile since time out of mind.

### fin

With support for untagged allocations, I think I am finally ready to start getting Whippet into Guile itself. Happy hacking, and see you on the other side!

## related articles

- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [partitioning ambiguous edges in guile](/archives/2025/04/25/partitioning-ambiguous-edges-in-guile)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)
- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)
- [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)

Comments are closed.
