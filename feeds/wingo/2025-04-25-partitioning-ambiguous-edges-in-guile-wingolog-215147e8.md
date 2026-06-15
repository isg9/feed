---
title: partitioning ambiguous edges in guile — wingolog
url: https://wingolog.org/archives/2025/04/25/partitioning-ambiguous-edges-in-guile
published: "2025-04-25T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2025/04/25/partitioning-ambiguous-edges-in-guile
---

# partitioning ambiguous edges in guile — wingolog

## [partitioning ambiguous edges in guile](/archives/2025/04/25/partitioning-ambiguous-edges-in-guile)

25 April 2025 3:00 PM

- [guile](/tags/guile)
- [bdw](/tags/bdw)
- [whippet](/tags/whippet)
- [garbage collection](/tags/garbage%20collection)
- [gc](/tags/gc)
- [conservative tracing](/tags/conservative%20tracing)
- [ambiguous edges](/tags/ambiguous%20edges)
- [continuations](/tags/continuations)
- [delimited continuations](/tags/delimited%20continuations)

Today, some more words on memory management, on the practicalities of a system with conservatively-traced references.

The context is that I have finally started banging [Whippet](https://github.com/wingo/whippet) into [Guile](https://gnu.org/s/guile), initially in a configuration that continues to use the conservative Boehm-Demers-Weiser (BDW) collector behind the scene. In that way I can incrementally migrate over all of the uses of the BDW API in Guile to use Whippet API instead, and then if all goes well, I should be able to switch Whippet to use another GC algorithm, probably the [mostly-marking collector
(MMC)](https://github.com/wingo/whippet/blob/main/doc/collector-mmc.md). MMC scales better than BDW for multithreaded mutators, and it can eliminate fragmentation via Immix-inspired optimistic evacuation.

### problem statement: how to manage ambiguous edges

A garbage-collected heap consists of memory, which is a set of addressable locations. An object is a disjoint part of a heap, and is the unit of allocation. A field is memory within an object that may refer to another object by address. Objects are nodes in a directed graph in which each edge is a field containing an object reference. A root is an edge into the heap from outside. Garbage collection reclaims memory from objects that are not reachable from the graph that starts from a set of roots. Reclaimed memory is available for new allocations.

In the course of its work, a collector may want to relocate an object, moving it to a different part of the heap. The collector can do so if it can update all edges that refer to the object to instead refer to its new location. Usually a collector arranges things so all edges have the same representation, for example an aligned word in memory; updating an edge means replacing the word’s value with the new address. Relocating objects can improve locality and reduce fragmentation, so it is a good technique to have available. (Sometimes we say evacuate, move, or compact instead of relocate; it’s all the same.)

Some collectors allow *ambiguous* edges: words in memory whose value may be the address of an object, or might just be scalar data. Ambiguous edges usually come about if a compiler doesn’t precisely record which stack locations or registers contain GC-managed objects. Such ambiguous edges must be traced *conservatively*: the collector adds the object to its idea of the set of live objects, as if the edge were a real reference. This tracing mode isn’t supported by all collectors.

Any object that might be the target of an ambiguous edge cannot be relocated by the collector; a collector that allows conservative edges cannot rely on relocation as part of its reclamation strategy. Still, if the collector can know that a given object will not be the referent of an ambiguous edge, relocating it is possible.

How can one know that an object is not the target of an ambiguous edge? We have to partition the heap somehow into possibly-conservatively-referenced and definitely-not-conservatively-referenced. The two ways that I know to do this are spatially and temporally.

Spatial partitioning means that regardless of the set of root and intra-heap edges, there are some objects that will never be conservatively referenced. This might be the case for a type of object that is “internal” to a language implementation; third-party users that may lack the discipline to precisely track roots might not be exposed to objects of a given kind. Still, link-time optimization tends to weather these boundaries, so I don’t see it as being too reliable over time.

Temporal partitioning is more robust: if all ambiguous references come from roots, then if one traces roots before intra-heap edges, then any object not referenced after the roots-tracing phase is available for relocation.

### kinds of ambiguous edges in guile

So let’s talk about Guile! Guile uses BDW currently, which considers edges to be ambiguous by default. However, given that objects carry type tags, Guile can, with relatively little effort, switch to precisely tracing most edges. “Most”, however, is not sufficient; to allow for relocation, we need to *eliminate* intra-heap ambiguous edges, to confine conservative tracing to the roots-tracing phase.

Conservatively tracing references from C stacks or even from static data sections is not a problem: these are roots, so, fine.

Guile currently traces Scheme stacks almost-precisely: its compiler emits stack maps for every call site, which uses liveness analysis to only mark those slots that are Scheme values that will be used in the continuation. However it’s possible that any given frame is marked conservatively. The most common case is when using the BDW collector and a thread is pre-empted by a signal; then its most recent stack frame is likely not at a safepoint and indeed is likely undefined in terms of Guile’s VM. It can also happen if there is a call site within a VM operation, for example to a builtin procedure, if it throws an exception and recurses, or causes GC itself. Also, when [per-instruction
traps](https://www.gnu.org/software/guile/manual/html_node/Low_002dLevel-Traps.html) are enabled, we can run Scheme between any two Guile VM operations.

So, Guile could change to trace Scheme stacks fully precisely, but this is a lot of work; in the short term we will probably just trace Scheme stacks as roots instead of during the main trace.

However, there is one more significant source of ambiguous roots, and that is reified continuation objects. Unlike active stacks, these have to be discovered during a trace and cannot be partitioned out to the root phase. For delimited continuations, these consist of a slice of the Scheme stack. Traversing a stack slice precisely is less problematic than for active stacks, because it isn’t in motion, and it is captured at a known point; but we will have to deal with stack frames that are pre-empted in unexpected locations due to exceptions within builtins. If a stack map is missing, probably the solution there is to reconstruct one using local flow analysis over the bytecode of the stack frame’s function; time-consuming, but it should be robust as we do it elsewhere.

Undelimited continuations (those captured by `call/cc`) contain a slice of the C stack also, for historical reasons, and there we can’t trace it precisely at all. Therefore either we disable relocation if there are any live undelimited continuation objects, or we eagerly pin any object referred to by a freshly captured stack slice.

### fin

If you want to follow along with the Whippet-in-Guile work, see the [wip-whippet](https://git.savannah.gnu.org/cgit/guile.git/log/?h=wip-whippet) branch in Git. I’ve bumped its version to 4.0 because, well, why the hell not; if it works, it will certainly be worth it. Until next time, happy hacking!

## related articles

- [whippet lab notebook: untagged mallocs, bis](/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)
- [conservative gc can be faster than precise gc](/archives/2024/09/07/conservative-gc-can-be-faster-than-precise-gc)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)

Comments are closed.
