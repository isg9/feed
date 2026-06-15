---
title: pre-tenuring in v8 — wingolog
url: https://wingolog.org/archives/2026/01/05/pre-tenuring-in-v8
published: "2026-01-05T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2026/01/05/pre-tenuring-in-v8
---

# pre-tenuring in v8 — wingolog

## [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)

5 January 2026 3:38 PM

- [v8](/tags/v8)
- [javascript](/tags/javascript)
- [garbage collection](/tags/garbage%20collection)
- [gc](/tags/gc)
- [generational collection](/tags/generational%20collection)
- [igalia](/tags/igalia)
- [compilers](/tags/compilers)

Hey hey happy new year, friends! Today I was going over some V8 code that touched *pre-tenuring*: allocating objects directly in the old space instead of the nursery. I knew the theory here but I had never looked into the mechanism. Today’s post is a quick overview of how it’s done.

### allocation sites

In a JavaScript program, there are a number of source code locations that allocate. Statistically speaking, any given allocation is likely to be short-lived, so generational garbage collection partitions freshly-allocated objects into their own space. In that way, when the system runs out of memory, it can preferentially reclaim memory from the nursery space instead of groveling over the whole heap.

But you know what they say: there are lies, damn lies, and statistics. Some programs are outliers, allocating objects in such a way that they don’t die young, or at least not young enough. In those cases, allocating into the nursery is just overhead, because minor collection won’t reclaim much memory (because too many objects survive), and because of useless copying as the object is scavenged within the nursery or promoted into the old generation. It would have been better to eagerly tenure such allocations into the old generation in the first place. (The more I think about it, the funnier *pre-tenuring* is as a term; what if some PhD programs could pre-allocate their graduates into named chairs? Is going straight to industry the equivalent of dying young? Does collaborating on a paper with a full professor imply a write barrier? But I digress.)

Among the set of allocation sites in a program, a subset should pre-tenure their objects. How can we know which ones? There is a literature of static techniques, but this is JavaScript, so the answer in general is dynamic: we should observe how many objects survive collection, organized by allocation site, then optimize to assume that the future will be like the past, falling back to a general path if the assumptions fail to hold.

### my runtime doth object

The high-level overview of how V8 implements pre-tenuring is based on per-program-point *AllocationSite* objects, and per-allocation *AllocationMemento* objects that point back to their corresponding AllocationSite. Initially, V8 doesn’t know what program points would profit from pre-tenuring, and instead allocates everything in the nursery. Here’s a quick picture:

![diagram of linear allocation buffer containing interleaved objects and allocation mementos](https://wingolog.org/pub/v8-allocation-sites.png)*A linear allocation buffer containing objects allocated with allocation mementos*

Here we show that there are two allocation sites, `Site1` and `Site2`. V8 is currently allocating into a linear allocation buffer (LAB) in the nursery, and has allocated three objects. After each of these objects is an `AllocationMemento`; in this example, `M1` and `M3` are `AllocationMemento` objects that point to `Site1` and `M2` points to `Site2`. When V8 allocates an object, it [increments the “created”
counter on the corresponding
`AllocationSite`](https://source.chromium.org/chromium/chromium/src/+/main:v8/src/heap/factory.cc;l=343-345;drc=56535b80c32d3a618b8fc8cfd7b1afdd3862e1b2) (if available; it’s possible an allocation comes from C++ or something where we don’t have an `AllocationSite`).

When the free space in the LAB is too small for an allocation, V8 gets another LAB, or collects if there are no more LABs in the nursery. When V8 does a minor collection, as the scavenger visits objects, it will [look to see if the object is followed by an
AllocationMemento](https://source.chromium.org/chromium/chromium/src/+/main:v8/src/heap/pretenuring-handler-inl.h;l=73-150). If so, it dereferences the memento to find the `AllocationSite`, then increments its “found” counter, and adds the `AllocationSite` to a set. [Once an AllocationSite has had 100
allocations](https://source.chromium.org/chromium/chromium/src/+/main:v8/src/heap/pretenuring-handler.cc;l=158), it is enqueued for a pre-tenuring decision; [sites with 85%
survival](https://source.chromium.org/chromium/chromium/src/+/main:v8/src/heap/pretenuring-handler.cc;l=50) get marked for pre-tenuring.

If an allocation site is marked as needing pre-tenuring, the code in which it is embedded it will get de-optimized, and then next time it is optimized, the code generator arranges to allocate into the old generation instead of the default nursery.

Finally, if a major collection collects more than 90% of the old generation, V8 [resets all pre-tenured allocation
sites](https://source.chromium.org/chromium/chromium/src/+/main:v8/src/heap/heap.cc;l=3034-3054), under the assumption that pre-tenuring was actually premature.

### tenure for me but not for thee

What kinds of allocation sites are eligible for pre-tenuring? Sometimes it depends on object kind; wasm memories, for example, are almost always long-lived, so they are always pre-tenured. Sometimes it depends on who is doing the allocation; allocations from the bootstrapper, literals allocated by the parser, and many allocations from C++ go straight to the old generation. And sometimes the compiler has enough information to determine that pre-tenuring might be a good idea, as when it [generates a store of a fresh object to a field in an known-old
object](https://source.chromium.org/chromium/chromium/src/+/main:v8/src/maglev/maglev-graph-builder.cc;l=4719).

But otherwise I thought that the whole AllocationSite mechanism would apply generally, to any object creation. It turns out, nope: it seems to only apply to object literals, array literals, and `new Array`. Weird, right? I guess it makes sense in that these are the ways to create objects that also creates the field values at creation-time, allowing the whole block to be allocated to the same space. If instead you make a pre-tenured object and then initialize it via a sequence of stores, this would likely create old-to-new edges, preventing the new objects from dying young while incurring the penalty of copying and write barriers. Still, I think there is probably some juice to squeeze here for pre-tenuring of class-style allocations, at least in the optimizing compiler or in short inline caches.

I suspect this state of affairs is somewhat historical, as the AllocationSite mechanism seems to have originated with [typed array
storage strategies](https://dl.acm.org/doi/10.1145/2509136.2509531) and V8’s “boilerplate” object literal allocators; both of these predate per-AllocationSite pre-tenuring decisions.

### fin

Well that’s adaptive pre-tenuring in V8! I thought the “just stick a memento after the object” approach is pleasantly simple, and if you are only bumping creation counters from baseline compilation tiers, it likely amortizes out to a win. But does the restricted application to literals point to a fundamental constraint, or is it just accident? If you have any insight, let me know :) Until then, happy hacking!

## related articles

- [the last couple years in v8's garbage collector](/archives/2025/11/13/the-last-couple-years-in-v8s-garbage-collector)
- [v8's precise field-logging remembered set](/archives/2024/01/05/v8s-precise-field-logging-remembered-set)
- [v8's mark-sweep nursery](/archives/2023/12/08/v8s-mark-sweep-nursery)
- [the last 5 years of V8's garbage collector](/archives/2023/12/07/the-last-5-years-of-v8s-garbage-collector)
- [ahead-of-time wasm gc in wastrel](/archives/2026/02/06/ahead-of-time-wasm-gc-in-wastrel)
- [an annoying failure mode of copying nurseries](/archives/2025/01/13/an-annoying-failure-mode-of-copying-nurseries)

### One response

1. Jan says:[5 January 2026 4:51 PM](#8df0364e7d1ab774e717d41dc2c557d23e162a0e)

   We have a similar pre-tenuring scheme in SpiderMonkey. We also use it in some other cases such as DOM nodes/reflectors (bug 1892764), and have a bug on file to apply it to `new Foo()` for user-defined constructors (bug 1966773).

   The Splay benchmark (in Octane/JetStream) benefits a lot from pre-tenuring.

Comments are closed.
