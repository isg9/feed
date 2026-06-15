---
title: conservative gc can be faster than precise gc — wingolog
url: https://wingolog.org/archives/2024/09/07/conservative-gc-can-be-faster-than-precise-gc
published: "2024-09-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/09/07/conservative-gc-can-be-faster-than-precise-gc
---

# conservative gc can be faster than precise gc — wingolog

## [conservative gc can be faster than precise gc](/archives/2024/09/07/conservative-gc-can-be-faster-than-precise-gc)

7 September 2024 10:00 AM

- [garbage collection](/tags/garbage%20collection)
- [gc](/tags/gc)
- [conservative scanning](/tags/conservative%20scanning)
- [bdw](/tags/bdw)
- [guile](/tags/guile)
- [precise roots](/tags/precise%20roots)
- [root-finding](/tags/root-finding)
- [whippet](/tags/whippet)
- [immix](/tags/immix)

Should your garbage collector be precise or conservative? The prevailing wisdom is that precise is always better. Conservative GC can retain more objects than strictly necessary, making GC slow: GC has to more frequently, and it has to trace a larger heap on each collection. However the calculus is not as straightforward as most people think, and indeed there are some reasons to expect that conservative root-finding can result in faster systems.

(I have made / relayed some of these arguments before but I feel like a dedicated article can make a contribution here.)

### problem precision

Let us assume that by *conservative GC* we mean conservative root-finding, in which the collector assumes that any integer on the stack that happens to be a heap address indicates a reference on the object containing that address. The address doesn’t have to be at the start of the object. Assume that objects on the heap are traced precisely; contrast to BDW-GC which generally traces both the stack and the heap conservatively. Assume a collector that will pin referents of conservative roots, but in which objects not referred to by a conservative root can be moved, as in [Conservative
Immix](https://dl.acm.org/doi/10.1145/2660193.2660198) or Whippet’s [`stack-conservative-mmc`
collector](https://github.com/wingo/whippet/blob/main/doc/collector-mmc.md#conservative-stack-scanning).

With that out of the way, let’s look at some reasons why conservative GC might be faster than precise GC.

### smaller lifetimes

A compiler that does precise root-finding will typically output a side-table indicating which slots in a stack frame hold references to heap objects. These lifetimes aren’t always precise, in the sense that although they precisely enumerate heap references, those heap references might actually not be used in the continuation of the stack frame. When GC occurs, it might mark more objects as live than are actually live, which is the imputed disadvantage of conservative collectors.

This is most obviously the case when you need to explicitly register roots with some kind of handle API: the handle will typically be kept live until the scope ends, but that might be an overapproximation of lifetime. A compiler that can assume conservative stack scanning may well exhibit more precision than it would if it needed to emit stack maps.

### no run-time overhead

For generated code, stack maps are great. But if a compiler needs to call out to C++ or something, it needs to precisely track roots in a [run-time data
structure](https://github.com/v8/v8/blob/main/src/handles/handles.h). This is overhead, and conservative collectors avoid it.

### smaller stack frames

A compiler may partition spill space on a stack into a part that contains pointers to the heap and a part containing numbers or other unboxed data. This may lead to larger stack sizes than if you could just re-use a slot for two purposes, if the lifetimes don’t overlap. A similar concern applies for compilers that partition registers.

### no stack maps

The need to emit stack maps is annoying for a compiler and makes binaries bigger. Of course it’s necessary for precise roots. But then there is additional overhead when tracing the stack: for each frame on the stack, you need to look up the stack map for the return continuation, which takes time. It may be faster to just test if words on the stack might be pointers to the heap.

### unconstrained compiler

Having to make stack maps is a constraint imposed on the compiler. Maybe if you don’t need them, the compiler could do a better job, or you could use a different compiler entirely. A conservative compiler can sometimes have better codegen, for example by the use of interior pointers.

### anecdotal evidence

The [Conservative Immix](https://dl.acm.org/doi/10.1145/2660193.2660198) paper shows that conservative stack scanning can beat precise scanning in some cases. I have reproduced these results with [`parallel-stack-conservative-mmc` compared to
`parallel-mmc`](https://github.com/wingo/whippet/blob/main/doc/collector-mmc.md#conservative-stack-scanning). It’s small—maybe a percent—but it was a surprising result to me and I thought others might want to know.

Also, Apple’s JavaScriptCore uses conservative stack scanning, and [V8 is looking at switching to it](https://wingolog.org/archives/2023/12/07/the-last-5-years-of-v8s-garbage-collector). Funny, right?

### conclusion

When it comes to designing a system with GC, don’t count out conservative stack scanning; the tradeoffs don’t obviously go one way or the other, and conservative scanning might be the right engineering choice for your system.

## related articles

- [on safepoints](/archives/2023/10/16/on-safepoints)
- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)
- [partitioning ambiguous edges in guile](/archives/2025/04/25/partitioning-ambiguous-edges-in-guile)

### 3 responses

1. solrize says:[8 September 2024 10:29 AM](#8e5a5d6cc2425af3cbfcd8e3f2accb000b90477c)

   The conservative GC might run faster than a precise one, but it can’t compact memory, so maybe the application runs slower due to fragmentation and cache misses. Lisp-like languages can even save a bunch of memory from cdr-coding. I know the Lisp machine did that, but idk if any present-day software implementations do.

2. Tommy Reilly says:[9 September 2024 11:54 AM](#85977e6a7432ab8cddd2625a80e68db2eff51046)

   The GC released in FlashPlayer 7 used this model: https://github.com/adobe/avmplus/MMgc

   Worked well enough for us. We definitely worried about false positives with large heaps on 32bit systems but hard to see that being an issue on 64 bit. If I remember correctly I think we arranged the stack marking to occur on frame boundaries so we wouldn’t scan any active frames with user code. Stacks are really dirty.

3. Nick Barnes says:[10 September 2024 8:42 AM](#6d3c2898ded7bc6bd19f0cf674cd452be26acf83)

   As noted in the article (“Assume that objects on the heap are traced precisely”) collectors can be conservative on the stack but precise otherwise. This opens the door to so-called “mostly-copying collectors” (Bartlett etc) which can still copy / compact / etc. There are a bunch of tricks, especially on 64-bit platforms (where there’s so much address space to play with), to manage allocation efficiently and reduce the amount of incorrect pinning.

Comments are closed.
