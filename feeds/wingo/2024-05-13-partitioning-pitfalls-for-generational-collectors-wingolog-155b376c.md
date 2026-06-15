---
title: partitioning pitfalls for generational collectors — wingolog
url: https://wingolog.org/archives/2024/05/13/partitioning-pitfalls-for-generational-collectors
published: "2024-05-13T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/05/13/partitioning-pitfalls-for-generational-collectors
---

# partitioning pitfalls for generational collectors — wingolog

## [partitioning pitfalls for generational collectors](/archives/2024/05/13/partitioning-pitfalls-for-generational-collectors)

13 May 2024 1:03 PM

- [v8](/tags/v8)
- [gc](/tags/gc)
- [generational collection](/tags/generational%20collection)
- [igalia](/tags/igalia)

You might have heard of long-pole problems: when you are packing a tent, its bag needs to be at least as big as the longest pole. (This is my preferred etymology; [there are
others](https://devblogs.microsoft.com/oldnewthing/20080805-00/?p=21373).) In garbage collection, the long pole is the longest chain of object references; there is no way you can algorithmically speed up pointer-chasing of (say) a long linked-list.

As a GC author, some of your standard tools can mitigate the long-pole problem, and some don’t apply.

*Parallelism* doesn’t help: a baby takes 9 months, no matter how many people you assign to the problem. You need to visit each node in the chain, one after the other, and having multiple workers available to process those nodes does not get us there any faster. Parallelism does help in general, of course, but doesn’t help for long poles.

You can apply *concurrency*: instead of stopping the user’s program (the *mutator*) to enumerate live objects, you trace while the mutator is running. This can happen on mutator threads, interleaved with allocation, via what is known as *incremental* tracing. Or it can happen in dedicated tracing threads, which is what people usually mean when they refer to *concurrent* tracing. Though it does impose some coordination overhead on the mutator, the pause-time benefits are such that most production garbage collectors do trace concurrently with the mutator, if they have the development resources to manage the complexity.

Then there is *partitioning*: instead of tracing the whole graph all the time, try to just trace part of it and just reclaim memory for that part. This bounds the size of a long pole—it can’t be bigger than the partition you trace—and so tracing a graph subset should reduce total pause time.

The usual way to partition is *generational*, in which the main program allocates into a *nursery*, and objects that survive a few collections then get promoted into *old space*. But there may be other partitions, for example to put [“large
objects”](https://wingolog.org/archives/2022/06/20/blocks-and-pages-and-large-objects) (for example, bigger than a few virtual memory pages) in their own section, to be managed with their own algorithm.

### partitions, take one

And that brings me to today’s article: generational partitioning has a failure mode which manifests itself as spurious out-of-memory. For example, in V8, running this small JavaScript program that repeatedly allocates a 1-megabyte buffer grows to consume all memory in the system and eventually panics:

```
while (1) new Uint8Array(1e6);

```

This is a funny result, to say the least. Let’s dig in a bit to see why.

First off, note that allocating a 1-megabyte Uint8Array makes a *large* *object*, because it is [bigger than half a V8
page](https://source.chromium.org/chromium/chromium/src/+/main:v8/src/common/globals.h;l=581-588), which is [256 kB on most
systems](https://source.chromium.org/chromium/chromium/src/+/main:v8/src/base/build_config.h;l=60-77.) There is a backing store for the array that will get allocated into the large object space, and then the JS object wrapper that gets allocated in the young generation of the regular object space.

Let’s imagine the heap consists of the nursery, the old space, and a large object space ( *lospace*, for short). (See the next section for a refinement of this model.) In the loop, we cause allocation in the nursery and the lospace. When the nursery starts to get full, we run a *minor* collection, to trace the newly allocated part of the object graph, possibly promoting some objects to the old generation.

Promoting an object adds to the byte count in the old generation. You could say that promoting causes allocation in the old-gen, though it might happen just on an accounting level if an object is promoted in place. In V8, old-gen allocation is subject to a [limit](https://source.chromium.org/chromium/chromium/src/+/main:v8/src/heap/heap.h;drc=41d53a863ef269e6d53fea38cc1d3feda6c3df78;l=2240), and that limit is set by a [gnarly series of
heuristics](https://source.chromium.org/chromium/chromium/src/+/main:v8/src/heap/heap.cc;l=2698;drc=41d53a863ef269e6d53fea38cc1d3feda6c3df78). Exceeding the limit will cause a *major* GC, in which the whole heap is traced.

So, minor collections cause major collections via promotion. But what if a minor collection never promotes? This can happen in request-oriented workflows, in which a request comes in, you handle it in JS, write out the result, and then handle the next. If the nursery is at least as large as the memory allocated in one request, no object will ever survive more than one collection. But in our `new Uint8Array(1e6)` example above, we are allocating to newspace and the lospace. If we never cause promotion, we will always collect newspace but never trigger the full GC that would also collect the large object space, triggering this out-of-memory phenomenon.

### partitions, take two

In general, the phenomenon we are looking for is nursery allocations that cause non-nursery allocations, where those non-nursery allocations will not themselves bring about a major GC.

In our example above, it was a typed array object with an associated lospace backing store, assuming the lospace allocation wouldn’t eventually trigger GC. But V8’s heap is not exactly like our model, for one because it actually has separate nursery and old-generation lospaces, and for two because allocation to the old-generation lospace does count towards the old-gen allocation limit. And yet, that simple does still blow through all memory. So what is the deal?

The answer is that V8’s heap now has [around two dozen spaces](https://source.chromium.org/chromium/chromium/src/+/main:v8/src/heap/heap.h;l=764-822), and allocations to some of those spaces escape the limits and allocation counters. In this case, [V8’s sandbox](https://v8.dev/blog/sandbox) makes the link between the JS typed array object and its backing store pass through a [table of
indirections](https://docs.google.com/document/d/1V3sxltuFjjhp_6grGHgfqZNK57qfzGzme0QTk0IXDHk/edit#heading=h.xzptrog8pyxf). At each allocation, we make an object in the nursery, allocate a backing store in the nursery lospace, and then allocate a new entry in the external pointer table, storing the index of that entry in the JS object. When the object needs to get its backing store, it dereferences the corresponding entry in the external pointer table.

Our tight loop above would therefore cause an allocation (in the external pointer table) that itself would not hasten a major collection.

Seen this way, one solution immediately presents itself: maybe we should find a way to make external pointer table entry allocations trigger a GC based on some heuristic, perhaps the old-gen allocated bytes counter. But then you have to actually trigger GC, and this is annoying and not always possible, as EPT entries can be allocated in response to a pointer write. [V8 hacker Michael Lippautz summed up the issue in a
comment](https://issues.chromium.org/issues/40643874#comment9) that can be summarized as “it’s gnarly”.

In the end, [it would be better if new-space allocations did not cause
old-space
allocations](https://issues.chromium.org/issues/40643874#comment24). We should separate the external pointer table (EPT) into old and new spaces. Because there is at most one “host” object that is associated with any EPT entry—if it’s zero objects, the entry is dead—then the heuristics that apply to host objects will do the right thing with regards to EPT entries.

A couple weeks ago, I [landed a
patch](https://source.chromium.org/chromium/_/chromium/v8/v8/+/90f276be1122336c6ff7b808054fb183af7a2a9e) to do this. It was much easier said than done; the patch went through upwards of 40 revisions, as it took me a while to understand the delicate interactions between the concurrent and parallel parts of the collector, which were themselves evolving as I was working on the patch.

The challenge mostly came from the fact that [V8 has two nursery
implementations that operate in different
ways](https://wingolog.org/archives/2023/12/08/v8s-mark-sweep-nursery).

The semi-space nursery (the *scavenger*) is a parallel stop-the-world collector, which is relatively straightforward... except that there can be a concurrent/incremental major trace in progress while the scavenger runs (a so-called *interleaved* minor GC). External pointer table entries have mark bits, which the scavenger collector will want to use to determine which entries are in use and which can be reclaimed, but the concurrent major marker will also want to use those mark bits. In the end we decided that if a major mark is in progress, an interleaved collection will neither mark nor sweep the external pointer table nursery; a major collection will finish soon, and reclaiming dead EPT entries will be its responsibility.

The [minor mark-sweep
nursery](https://wingolog.org/archives/2023/12/08/v8s-mark-sweep-nursery) does not run concurrently with a major concurrent/incremental trace, which is something of a relief, but it does run concurrently with the mutator. When we promote a page, we also need to promote all EPT entries for objects on that page. To keep track of which objects have external pointers, we had to add a new [remembered
set](https://source.chromium.org/chromium/chromium/src/+/main:v8/src/heap/memory-chunk-layout.h;l=33?q=survivor_to_external_pointer), built up during a minor trace, and cleared when the page is swept (also lazily / concurrently). Promoting a page [iterates through that set,
evacuating EPT entries to the old
space](https://source.chromium.org/chromium/chromium/src/+/main:v8/src/heap/minor-mark-sweep.cc;l=872?q=survivor_to_external_pointer). This is additional work during the stop-the-world pause, but hopefully it is not too bad.

To be honest I don’t have the performance numbers for this one. It rides the train this week to Chromium 126 though, and hopefully it fixes this problem in a robust way.

### partitions, take three

The V8 sandboxing effort has sprouted a number of [indirect
tables](https://docs.google.com/document/d/1FM4fQmIhEqPG8uGp5o9A-mnPB5BOeScZYpkHjo0KKA8/edit#heading=h.fg3qxf1x0p2q): [external
pointers](https://docs.google.com/document/d/1V3sxltuFjjhp_6grGHgfqZNK57qfzGzme0QTk0IXDHk/edit#heading=h.xzptrog8pyxf), [external buffers](https://issues.chromium.org/issues/42204529), [trusted
objects](https://docs.google.com/document/d/1IrvzL4uX_Zv0k2Iakdp_q_z33bj-qlYF5IesGpXW0fM/edit#heading=h.xzptrog8pyxf), and [code
objects](https://docs.google.com/document/d/1CPs5PutbnmI-c5g7e_Td9CNGh5BvpLleKCqUnqmD82k/edit#heading=h.xzptrog8pyxf). Also recently to better integrate with compressed pointers, there is also a [table for V8-to-managed-C++ (Oilpan)
references](https://chromium-review.googlesource.com/c/v8/v8/+/5392814). I expect the Oilpan reference table will soon grow a nursery along the same lines as the regular external pointer table.

In general, if you have a generational partition of your main object space, it would seem that you almost always need a generational partition of every other space. Otherwise either you cause new allocations to occur in what is effectively an old space, perhaps needlessly hastening a major GC, or you forget to track allocations in that space, leading to a memory leak. If the generational hypothesis applies for a wrapper object, it probably also applies for any auxiliary allocations as well.

### fin

I have a few cleanups remaining in this area but I am relieved to have landed this patch, and pleased to have spent time under the hood of V8’s collector. Many many thanks to Samuel Groß, Michael Lippautz, and Dominik Inführ for their review and patience. Until the next cycle, happy allocating!

## related articles

- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)
- [ahead-of-time wasm gc in wastrel](/archives/2026/02/06/ahead-of-time-wasm-gc-in-wastrel)
- [the last couple years in v8's garbage collector](/archives/2025/11/13/the-last-couple-years-in-v8s-garbage-collector)
- [an annoying failure mode of copying nurseries](/archives/2025/01/13/an-annoying-failure-mode-of-copying-nurseries)
- [v8's precise field-logging remembered set](/archives/2024/01/05/v8s-precise-field-logging-remembered-set)
- [v8's mark-sweep nursery](/archives/2023/12/08/v8s-mark-sweep-nursery)

Comments are closed.
