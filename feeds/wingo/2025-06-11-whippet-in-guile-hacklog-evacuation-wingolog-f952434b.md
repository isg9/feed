---
title: 'whippet in guile hacklog: evacuation — wingolog'
url: https://wingolog.org/archives/2025/06/11/whippet-in-guile-hacklog-evacuation
published: "2025-06-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2025/06/11/whippet-in-guile-hacklog-evacuation
---

# whippet in guile hacklog: evacuation — wingolog

## [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)

11 June 2025 8:56 PM

- [guile](/tags/guile)
- [whippet](/tags/whippet)
- [gc](/tags/gc)
- [gnu](/tags/gnu)
- [igalia](/tags/igalia)
- [evacuation](/tags/evacuation)
- [immix](/tags/immix)

Good evening, hackfolk. A quick note this evening to record a waypoint in my efforts to improve Guile’s memory manager.

So, I got Guile running on top of the [Whippet](https://github.com/wingo/whippet/) API. This API can be implemented by a number of concrete garbage collector implementations. The implementation backed by the Boehm collector is fine, as expected. The implementation that uses the [bump-pointer-allocation-into-holes
strategy](https://wingolog.org/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth) is less good. The minor reason is heap sizing heuristics; I still get it wrong about when to grow the heap and when not to do so. But the major reason is that non-moving Immix collectors appear to have pathological fragmentation characteristics.

Fragmentation, for our purposes, is memory under the control of the GC which was free after the previous collection, but which the current cycle failed to use for allocation. I have the feeling that for the non-moving Immix-family collector implementations, fragmentation is much higher than for size-segregated freelist-based mark-sweep collectors. For an allocation of, say, 1024 bytes, the collector might have to scan over many smaller holes until you find a hole that is big enough. This wastes free memory. Fragmentation memory is not gone—it is still available for allocation!—but it won’t be allocatable until after the current cycle when we visit all holes again. In Immix, fragmentation wastes allocatable memory during a cycle, hastening collection and causing more frequent whole-heap traversals.

The value proposition of Immix is that if there is too much fragmentation, you can just go into evacuating mode, and probably improve things. I still buy it. However I don’t think that non-moving Immix is a winner. I still need to do more science to know for sure. I need to fix Guile to support the stack-conservative, heap-precise version of the Immix-family collector which will allow for evacuation.

So that’s where I’m at: a load of gnarly Guile refactors to allow for precise tracing of the heap. I probably have another couple weeks left until I can run some tests. Fingers crossed; we’ll see!

## related articles

- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [an annoying failure mode of copying nurseries](/archives/2025/01/13/an-annoying-failure-mode-of-copying-nurseries)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [whippet update: faster evacuation, eager sweeping of empty blocks](/archives/2024/08/25/whippet-update-faster-evacuation-eager-sweeping-of-empty-blocks)
- [parallel ephemeron tracing](/archives/2023/01/24/parallel-ephemeron-tracing)

Comments are closed.
