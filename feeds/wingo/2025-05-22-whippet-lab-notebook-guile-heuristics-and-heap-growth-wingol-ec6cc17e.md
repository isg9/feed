---
title: 'whippet lab notebook: guile, heuristics, and heap growth — wingolog'
url: https://wingolog.org/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth
published: "2025-05-22T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth
---

# whippet lab notebook: guile, heuristics, and heap growth — wingolog

## [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)

22 May 2025 10:05 AM

- [guile](/tags/guile)
- [whippet](/tags/whippet)
- [bdw](/tags/bdw)
- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [nofl](/tags/nofl)
- [igalia](/tags/igalia)
- [fragmentation](/tags/fragmentation)
- [heuristics](/tags/heuristics)

Greets all! Another brief note today. I have gotten Guile working with one of the [Nofl](https://arxiv.org/abs/2503.16971)-based collectors, specifically the one that scans all edges conservatively ( [`heap-conservative-mmc` /
`heap-conservative-parallel-mmc`](https://github.com/wingo/whippet/blob/main/doc/collector-mmc.md#conservative-heap-scanning)). Hurrah!

It was a pleasant surprise how easy it was to switch—from the user’s point of view, you just pass `--with-gc=heap-conservative-parallel-mmc` to Guile’s build (on the `wip-whippet` branch); when developing I also pass `--with-gc-debug`, and I had a couple bugs to fix—but, but, there are still some issues. Today’s note thinks through the ones related to heap sizing heuristics.

### growable heaps

Whippet has [three heap sizing
strategies](https://wingolog.org/archives/2023/01/27/three-approaches-to-heap-sizing): fixed, growable, and adaptive ( [MemBalancer](https://marisa.moe/balancer.html)). The adaptive policy is the one I would like in the long term; it will grow the heap for processes with a high allocation rate, and shrink when they go idle. However I won’t really be able to test heap shrinking until I get precise tracing of heap edges, which will allow me to evacuate sparse blocks.

So for now, Guile uses the growable policy, which attempts to size the heap so it is at least as large as the live data size, times some multiplier. The multiplier currently defaults to 1.75×, but can be set on the command line via the `GUILE_GC_OPTIONS` environment variable. For example to set an initial heap size of 10 megabytes and a 4× multiplier, you would set `GUILE_GC_OPTIONS=heap-size-multiplier=4,heap-size=10M`.

Anyway, I have run into problems! The fundamental issue is fragmentation. Consider a 10MB growable heap with a 2× multiplier, consisting of a sequence of 16-byte objects followed by 16-byte holes. You go to allocate a 32-byte object. This is a small object (8192 bytes or less), and so it goes in the Nofl space. A Nofl mutator holds on to a block from the list of sweepable blocks, and will sequentially scan that block to find holes. However, each hole is only 16 bytes, so we can’t fit our 32-byte object: we finish with the current block, grab another one, repeat until no blocks are left and we cause GC. GC runs, and after collection we have an opportunity to grow the heap: but the heap size is already twice the live object size, so the heuristics say we’re all good, no resize needed, leading to the same sweep again, leading to a livelock.

I actually ran into this case during Guile’s bootstrap, while allocating a 7072-byte vector. So it’s a thing that needs fixing!

### observations

The root of the problem is fragmentation. One way to solve the problem is to remove fragmentation; using a semi-space collector comprehensively resolves the issue, [modulo
any block-level fragmentation](https://wingolog.org/archives/2024/07/10/copying-collectors-with-block-structured-heaps-are-unreliable).

However, let’s say you have to live with fragmentation, for example because your heap has ambiguous edges that need to be traced conservatively. What can we do? Raising the heap multiplier is an effective mitigation, as it increases the average hole size, but for it to be a comprehensive solution in e.g. the case of 16-byte live objects equally interspersed with holes, you would need a multiplier of 512× to ensure that the largest 8192-byte “small” objects will find a hole. I could live with 2× or something, but 512× is too much.

We could consider changing the heap organization entirely. For example, most mark-sweep collectors (BDW-GC included) partition the heap into blocks whose allocations are of the same size, so you might have some blocks that only hold 16-byte allocations. It is theoretically possible to run into the same issue, though, if each block only has one live object, and the necessary multiplier that would “allow” for more empty blocks to be allocated is of the same order (256× for 4096-byte blocks each with a single 16-byte allocation, or even 4096× if your blocks are page-sized and you have 64kB pages).

My conclusion is that practically speaking, if you can’t deal with fragmentation, then it is impossible to just rely on a heap multiplier to size your heap. It is certainly an error to live-lock the process, hoping that some other thread mutates the graph in such a way to free up a suitable hole. At the same time, if you have configured your heap to be growable at run-time, it would be bad policy to fail an allocation, just because you calculated that the heap is big enough already.

It’s a shame, because we lose a mooring on reality: “how big will my heap get” becomes an unanswerable question because the heap might grow in response to fragmentation, which is not deterministic if there are threads around, and so we can’t reliably compare performance between different configurations. Ah well. If reliability is a goal, I think one needs to allow for evacuation, one way or another.

### for nofl?

In this concrete case, I am still working on a solution. It’s going to be heuristic, which is a bit of a disappointment, but here we are.

My initial thought has two parts. Firstly, if the heap is growable but cannot defragment, then we need to reserve some empty blocks after each collection, even if reserving them would grow the heap beyond the configured heap size multiplier. In that way we will always be able to allocate into the Nofl space after a collection, because there will always be some empty blocks. How many empties? Who knows. Currently Nofl blocks are 64 kB, and the largest “small object” is 8kB. I’ll probably try some constant multiplier of the heap size.

The second thought is that searching through the entire heap for a hole is a silly way for the mutator to spend its time. Immix will reserve a block for *overflow allocation*: if a medium-sized allocation (more than 256B and less than 8192B) fails because no hole in the current block is big enough—note that Immix’s holes have 128B granularity—then the allocation goes to a dedicated *overflow block*, which is taken from the empty block set. This reduces fragmentation (holes which were not used for allocation because they were too small).

Nofl should probably do the same, but given its finer granularity, it might be better to sweep over a variable number of blocks, for example based on the logarithm of the allocation size; one could instead sweep over `clz(min-size)–clz(size)` blocks before taking from the empty block list, which would at least bound the sweeping work of any given allocation.

### fin

Welp, just wanted to get this out of my head. So far, my experience with this Nofl-based heap configuration is mostly colored by live-locks, and otherwise its implementation of a growable heap sizing policy seems to be more tight-fisted regarding memory allocation than BDW-GC’s implementation. I am optimistic though that I will be able to get precise tracing sometime soon, as measured in development time; the problem as always is fragmentation, in that I don’t have a hole in my calendar at the moment. Until then, sweep on Wayne, cons on Garth, onwards and upwards!

## related articles

- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet hacklog: adding freelists to the no-freelist space](/archives/2025/08/07/whippet-hacklog-adding-freelists-to-the-no-freelist-space)
- [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)
- [whippet lab notebook: untagged mallocs, bis](/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)
- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)

### One response

1. Mikael Djurfeldt says:[22 May 2025 7:51 PM](#822b9b95731965fd3d1a383848c6c7b440371b2e)

   Great to follow this development! A maybe naive question from someone who never studied GC:s for real: I can see how a heap size based on live objects + fragmentation leads to livelocks. Maybe one shouldn’t, then, base heap size on live objects? From the perspective of allocating a block which doesn’t fit in spaces in-between, all of the heap behaves as live objects. Furthermore, the size of live objects + holes is always the same order as live objects.

   If there was a cheap way of knowing the size of live objects + holes, one could simply use the multiplier on that. If not: When we have determined that we can’t find free space, we know that the heap should grow (as you discussed above). Maybe apply the multiplier to the present heap size, then, instead of the live objects?

Comments are closed.
