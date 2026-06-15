---
title: 'whippet hacklog: adding freelists to the no-freelist space — wingolog'
url: https://wingolog.org/archives/2025/08/07/whippet-hacklog-adding-freelists-to-the-no-freelist-space
published: "2025-08-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2025/08/07/whippet-hacklog-adding-freelists-to-the-no-freelist-space
---

# whippet hacklog: adding freelists to the no-freelist space — wingolog

## [whippet hacklog: adding freelists to the no-freelist space](/archives/2025/08/07/whippet-hacklog-adding-freelists-to-the-no-freelist-space)

7 August 2025 3:02 PM

- [whippet](/tags/whippet)
- [guile](/tags/guile)
- [gc](/tags/gc)
- [bdw](/tags/bdw)
- [nofl](/tags/nofl)
- [membalancer](/tags/membalancer)
- [chase-lev](/tags/chase-lev)
- [freelists](/tags/freelists)
- [heuristics](/tags/heuristics)
- [heap growth](/tags/heap%20growth)

August greetings, comrades! Today I want to bookend some recent work on [my Immix-inspired garbage
collector](https://wingolog.org/archives/2025/07/08/guile-lab-notebook-on-the-move): firstly, an idea with muddled results, then a slog through heuristics.

### the big idea

My mostly-marking collector’s main space is called the [“nofl space”](https://arxiv.org/abs/2503.16971). Its name comes from its historical evolution from mark-sweep to mark-region: instead of sweeping unused memory to freelists and allocating from those freelists, sweeping is interleaved with allocation; “nofl” means “no free-list”. As it finds holes, the collector bump-pointer allocates into those holes. If an allocation doesn’t fit into the current hole, the collector sweeps some more to find the next hole, possibly fetching another block. Space for holes that are too small is effectively wasted as fragmentation; mutators will try again after the next GC. Blocks with lots of holes will be chosen for opportunistic evacuation, which is the heap defragmentation mechanism.

Hole-too-small fragmentation has bothered me, because it presents a potential pathology. You don’t know how a GC will be used or what the user’s allocation pattern will be; if it is a mix of medium (say, a kilobyte) and small (say, 16 bytes) allocations, one could imagine a medium allocation having to sweep over lots of holes, discarding them in the process, which hastens the next collection. Seems wasteful, especially for non-moving configurations.

So I had a thought: why not collect those holes into a [size-segregated
freelist](https://github.com/wingo/whippet/blob/main/src/nofl-holeset.h)? We just cleared the hole, the memory is core-local, and we might as well. Then before fetching a new block, the allocator slow-path can see if it can service an allocation from the second-chance freelist of holes. This decreases locality a bit, but maybe it’s worth it.

Thing is, I implemented it, and I don’t know if it’s worth it! It seems to interfere with evacuation, in that the blocks that would otherwise be most profitable to evacuate, because they contain many holes, are instead filled up with junk due to second-chance allocation from the freelist. I need to do more measurements, but I think my big-brained idea is a bit of a wash, at least if evacuation is enabled.

### heap growth

When running the new collector in Guile, we have a performance oracle in the form of BDW: it had better be faster for Guile to compile a Scheme file with the new nofl-based collector than with BDW. In this use case we have an additional degree of freedom, in that unlike the lab tests of nofl vs BDW, we don’t impose a fixed heap size, and instead allow heuristics to determine the growth.

BDW’s built-in heap growth heuristics are very opaque. You give it a heap multiplier, but as a divisor truncated to an integer. It’s very imprecise. Additionally, there are nonlinearities: BDW is relatively more generous for smaller heaps, because attempts to model and amortize tracing cost, and there are some fixed costs (thread sizes, static data sizes) that don’t depend on live data size.

Thing is, BDW’s heuristics work pretty well. For example, I had a process that ended with a heap of about 60M, for a peak live data size of 25M or so. If I ran my collector with a fixed heap multiplier, it wouldn’t do as well as BDW, because it collected much more frequently when the heap was smaller.

I ended up switching from the primitive “size the heap as a multiple of live data” strategy to live data plus a square root factor; this is like what Racket ended up doing in its simple implementation of [MemBalancer](https://marisa.moe/balancer.html). (I do have a proper implementation of MemBalancer, with time measurement and shrinking and all, but I haven’t put it through its paces yet.) With this fix I can meet BDW’s performance for my Guile-compiling-Guile-with-growable-heap workload. It would be nice to exceed BDW of course!

### parallel worklist tweaks

Previously, in parallel configurations, trace workers would each have a [Chase-Lev deque](https://inria.hal.science/hal-00802885/document) to which they could publish objects needing tracing. Any worker could steal an object from the top of a worker’s public deque. Also, each worker had a local, unsynchronized FIFO worklist, some 1000 entries in length; when this worklist filled up, the worker would publish its contents.

There is a pathology for this kind of setup, in which one worker can end up with a lot of work that it never publishes. For example, if there are 100 long singly-linked lists on the heap, and the worker happens to have them all on its local FIFO, then perhaps they never get published, because the FIFO never overflows; you end up not parallelising. This seems to be the case in one microbenchmark. I switched to not have local worklists at all; perhaps this was not the right thing, but who knows. Will poke in future.

### a hilarious bug

Sometimes you need to know whether a given address is in an object managed by the garbage collector. For the nofl space it’s pretty easy, as we have big slabs of memory; bisecting over the array of slabs is fast. But for large objects whose memory comes from the kernel, we don’t have that. (Yes, you can reserve a big ol’ region with `PROT_NONE` and such, and then allocate into that region; I don’t do that currently.)

Previously I had a splay tree for lookup. Splay trees are great but not so amenable to concurrent access, and parallel marking is one place where we need to do this lookup. So I prepare a sorted array before marking, and then bisect over that array.

Except a funny thing happened: I switched the bisect routine to return the start address if an address is in a region. Suddenly, weird failures started happening randomly. Turns out, in some places I was testing if bisection succeeded with an `int`; if the region happened to be 32-bit-aligned, then the nonzero 64-bit `uintptr_t` got truncated to its low 32 bits, which were zero. Yes, crusty reader, Rust would have caught this!

### fin

I want this new collector to work. Getting the growth heuristic good enough is a step forward. I am annoyed that second-chance allocation didn’t work out as well as I had hoped; perhaps I will find some time this fall to give a proper evaluation. In any case, thanks for reading, and hack at you later!

## related articles

- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)
- [partitioning ambiguous edges in guile](/archives/2025/04/25/partitioning-ambiguous-edges-in-guile)
- [whippet lab notebook: untagged mallocs, bis](/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis)
- [whippet lab notebook: on untagged mallocs](/archives/2025/03/04/whippet-lab-notebook-on-untagged-mallocs)

### One response

1. [wleslie](https://william-ml-leslie.id.au/) says:[13 August 2025 3:09 AM](#72a5e2c2f0d1cfba46e9d2a1dd0a48526cedbe69)

   You don’t know how a GC will be used or what the user’s allocation pattern will be

   This problem - not knowing what the user’s allocation pattern will be - really looks like a compilers problem to me, one that still feels like a largely untouched frontier. MLKit and later Rust made some inroads, but there’s so much space to experiment.

Comments are closed.
