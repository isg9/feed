---
title: ephemerons vs generations in whippet — wingolog
url: https://wingolog.org/archives/2025/01/09/ephemerons-vs-generations-in-whippet
published: "2025-01-09T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2025/01/09/ephemerons-vs-generations-in-whippet
---

# ephemerons vs generations in whippet — wingolog

## [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)

9 January 2025 10:15 AM

- [whippet](/tags/whippet)
- [gc](/tags/gc)
- [ephemerons](/tags/ephemerons)
- [finalizers](/tags/finalizers)
- [parallelism](/tags/parallelism)
- [tracing](/tags/tracing)
- [evacuation](/tags/evacuation)
- [guile](/tags/guile)
- [igalia](/tags/igalia)
- [garbage collection](/tags/garbage%20collection)

Happy new year, hackfolk! Today, a note about [ephemerons](https://wingolog.org/tags/ephemerons). I thought I was done with them, but it seems they are not done with me. The question at hand is, how do we efficiently and correctly implement ephemerons in a generational collector? [Whippet](https://github.com/wingo/whippet)‘s answer turns out to be simple but subtle.

### on oracles

The deal is, I want to be able to evaluate different collector constructions and configurations, and for that I need a performance oracle: a known point in performance space-time against which to compare the unknowns. For example, I want to know how a [sticky mark-bit
approach to generational collection](https://wingolog.org/archives/2022/10/22/the-sticky-mark-bit-algorithm) does relative to the conventional state of the art. To do that, I need to build a conventional system to compare against! If I manage to do a good job building the conventional evacuating nursery, it will have similar performance characteristics as other nurseries in other unlike systems, and thus I can use it as a point of comparison, even to systems I haven’t personally run myself.

So I am adapting the [parallel copying collector I described last
July](https://wingolog.org/archives/2024/07/24/whippet-progress-update-funding-features-future) to have generational support: a copying (evacuating) young space and a copying old space. Ideally then I’ll be able to build a collector with a copying young space (nursery) but a mostly-marking [nofl](https://wingolog.org/archives/2024/08/25/whippet-update-faster-evacuation-eager-sweeping-of-empty-blocks) old space.

### notes on a copying nursery

A copying nursery has different operational characteristics than a sticky-mark-bit nursery, in a few ways. One is that a sticky mark-bit nursery will promote all survivors at each minor collection, leaving the nursery empty when mutators restart. This has the pathology that objects allocated just before a minor GC aren’t given a chance to “die young”: a sticky-mark-bit GC over-promotes.

Contrast that to a copying nursery, which can decide to promote a survivor or leave it in the young generation. In Whippet the current strategy for the parallel-copying nursery I am working on is to keep freshly allocated objects around for another collection, and only promote them if they are live at the next collection. We can do this with a cheap per-block flag, set if the block has any survivors, which is the case if it was allocated into as part of evacuation during minor GC. This gives objects enough time to die young while not imposing much cost in the way of recording per-object ages.

Recall that during a GC, all inbound edges from outside the graph being traced must be part of the root set. For a minor collection where we just trace the nursery, that root set must include all old-to-new edges, which are maintained in a data structure called the [remembered
set](https://wingolog.org/archives/2024/01/05/v8s-precise-field-logging-remembered-set). Whereas for a sticky-mark-bit collector the remembered set will be empty after each minor GC, for a copying collector this may not be the case. An existing old-to-new remembered edge may be unnecessary, because the target object was promoted; we will clear these old-to-old links at some point. (In practice this is done either in bulk during a major GC, or the next time the remembered set is visited during the root-tracing phase of a minor GC.) Or we could have a new-to-new edge which was not in the remembered set before, but now because the source of the edge was promoted, we must adjoin this old-to-new edge to the remembered set.

To preserve the invariant that all edges into the nursery are part of the roots, we have to pay special attention to this latter kind of edge: we *could* (should?) remove old-to-promoted edges from the remembered set, but we *must* add promoted-to-survivor edges. The field tracer has to have specific logic that applies to promoted objects during a minor GC to make the necessary remembered set mutations.

### other object kinds

In Whippet, “small” objects (less than 8 kilobytes or so) are allocated into block-structed spaces, and large objects have their own space which is managed differently. Notably, large objects are never moved. There is generational support, but it is currently like the sticky-mark-bit approach: any survivor is promoted. Probably we should change this at some point, at least for collectors that don’t eagerly promote all objects during minor collections.

### finalizers?

Finalizers keep their target objects alive until the finalizer is run, which effectively makes [each finalizer part of the root set](https://github.com/wingo/whippet/blob/main/src/gc-finalizer.c#L220-L237). Ideally we would have a separate finalizer table for young and old objects, but currently Whippet just has one table, which we always fully traverse at the end of a collection. This effectively adds the finalizer table to the remembered set. This is too much work—there is no need to visit finalizers for old objects in a minor GC—but it’s not incorrect.

### ephemerons

So what about ephemerons? Recall that [an ephemeron is an object
*E* × *K* ⇒ *V* in which there is an edge from *E* to *V* if and only if
both *E* and *K* are
live](https://wingolog.org/archives/2022/10/31/ephemerons-and-finalizers). Implementing this conjunction is surprisingly gnarly; you really want to discover live ephemerons while tracing rather than maintaining a global registry as we do with finalizers. Whippet’s algorithm is derived from [what SpiderMonkey
does](https://blog.mozilla.org/sfink/2022/06/09/ephemeron-tables-aka-javascript-weakmaps/), but [extended to be
parallel](https://wingolog.org/archives/2023/01/24/parallel-ephemeron-tracing).

The question is, how do we implement ephemeron-reachability while also preserving the invariant that all old-to-new edges are part of the remembered set?

For Whippet, the answer turns out to be simple: an ephemeron *E* is never older than its *K* or *V*, by construction, and we never promote *E* without also promoting (if necessary) *K* and *V*. (Ensuring this second property is somewhat delicate.) In this way you never have an old *E* and a young *K* or *V*, so no edge from an ephemeron need ever go into the remembered set. We still need to run the ephemeron tracing algorithm for any ephemerons discovered as part of a minor collection, but we don’t need to fiddle with the remembered set. Phew!

### conclusion

As long all promoted objects are older than all survivors, and that all ephemerons are younger than the objects referred to by their key and value edges, Whippet’s [parallel ephemeron tracing
algorithm](https://wingolog.org/archives/2023/01/24/parallel-ephemeron-tracing) will efficiently and correctly trace ephemeron edges in a generational collector. This applies trivially for a sticky-mark-bit collector, which always promotes and has no survivors, but it also holds for a copying nursery that allows for survivors after a minor GC, as long as all survivors are younger than all promoted objects.

Until next time, happy hacking in 2025!

## related articles

- [parallel ephemeron tracing](/archives/2023/01/24/parallel-ephemeron-tracing)
- [whippet update: faster evacuation, eager sweeping of empty blocks](/archives/2024/08/25/whippet-update-faster-evacuation-eager-sweeping-of-empty-blocks)
- [finalizers, guardians, phantom references, et cetera](/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [an annoying failure mode of copying nurseries](/archives/2025/01/13/an-annoying-failure-mode-of-copying-nurseries)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)

### One response

1. k4r4b3y says:[12 January 2025 9:46 PM](#9426117d2b01675da608ca51c19281fde8623c63)

   Hello Wingo. I’ve been trying to hack at your tekuti codebase – trying to adopt it, and use it for my own static blog.

   However, I find it quite difficult to get it up and running. There seems to be a lot of problems with it if one tries to use Tekuti to start a blog with no content in its initial condition. Most of the times the content bogosity classifier seems to give errors and cause no user comments to show up. Also, while testing it locally, I noticed that my second blog post, or my third blog post do not show up in the localhost:8080 rendering.

   Are you aware of these issues? Could you lend me some help me in overcoming those?

Comments are closed.
