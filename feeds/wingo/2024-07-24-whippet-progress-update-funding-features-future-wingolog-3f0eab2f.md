---
title: 'whippet progress update: funding, features, future — wingolog'
url: https://wingolog.org/archives/2024/07/24/whippet-progress-update-funding-features-future
published: "2024-07-24T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/07/24/whippet-progress-update-funding-features-future
---

# whippet progress update: funding, features, future — wingolog

## [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)

24 July 2024 9:19 AM

- [whippet](/tags/whippet)
- [gc](/tags/gc)
- [nlnet](/tags/nlnet)
- [garbage collection](/tags/garbage%20collection)
- [ngi0](/tags/ngi0)
- [igalia](/tags/igalia)
- [guile](/tags/guile)
- [finalizers](/tags/finalizers)
- [semi-space](/tags/semi-space)
- [cheney](/tags/cheney)
- [wasm2c](/tags/wasm2c)
- [ocaml](/tags/ocaml)
- [bdw](/tags/bdw)

Greets greets! Today, an update on recent progress in [Whippet](https://github.com/wingo/whippet), including sponsorship, a new collector, and a new feature.

### the lob, the pitch

But first, a reminder of what the haps: Whippet is a garbage collector library. The target audience is language run-time authors, particularly “small” run-times: [wasm2c](https://github.com/WebAssembly/wabt/blob/main/wasm2c/README.md), [Guile](https://gnu.org/s/guile), [OCaml](https://ocaml.org), and so on; to a first approximation, the [kinds of projects that currently use the
Boehm-Demers-Weiser
collector](https://github.com/ivmai/bdwgc/wiki/Known-clients).

The pitch is that if you use Whippet, you get a low-fuss small dependency to [vendor into your source
tree](https://github.com/wingo/whippet/blob/main/doc/manual.md#subtree-merges) that offers you access to a choice of advanced garbage collectors: not just the conservative mark-sweep collector from BDW-GC, but also copying collectors, an Immix-derived collector, generational collectors, and so on. You can choose the GC that fits your problem domain, like [Java
people have done for many
years](https://developers.redhat.com/articles/2021/11/02/how-choose-best-java-garbage-collector). The Whippet API is designed to be a [no-overhead
abstraction](https://github.com/wingo/whippet/blob/main/doc/manual.md#heaps-and-mutators) that decouples your language run-time from the specific choice of GC.

I co-maintain Guile and will be working on integrating Whippet in the next months, and have high hopes for success.

### [bridgeroos](https://en.wikipedia.org/wiki/Eurobridges_Spijkenisse)!

I’m delighted to share that Whippet was granted financial support from the European Union via the [NGI zero core](https://nlnet.nl/core/) fund, administered by the Dutch non-profit, [NLnet
foundation](https://nlnet.nl/). See the [NLnet project
page](https://nlnet.nl/project/Whippet/) for the overview.

This funding allows me to devote time to Whippet to bring it from proof-of-concept to production. I’ll finish the [missing
features](https://github.com/wingo/whippet?tab=readme-ov-file#missing-features-before-guile-can-use-whippet), spend some time adding tracing support, measuring performance, and sanding off any rough edges, then work on integrating Whippet into Guile.

This bloggery is a first update of the progress of the funded NLnet project.

### a new collector!

I landed a new collector a couple weeks ago, a [*parallel copying*
*collector*](https://github.com/wingo/whippet/blob/main/src/pcc.c) (PCC). It’s like a [semi-space
collector](https://wingolog.org/archives/2022/12/10/a-simple-semi-space-collector), in that it always evacuates objects (except [large
objects](https://wingolog.org/archives/2022/06/20/blocks-and-pages-and-large-objects), which are managed in their own space). However instead of having a single global bump-pointer allocation region, it breaks the heap into 64-kB blocks. In this way it supports multiple mutator threads: mutators do local bump-pointer allocation into their own block, and when their block is full, they fetch another from the global store.

The block size is 64 kB, but really it’s 128 kB, because each block has two halves: the active region and the copy reserve. It’s a copying collector, after all. Dividing each block in two allows me to easily grow and shrink the heap while ensuring there is always enough reserve space.

Blocks are allocated in 64-MB aligned slabs, so you get 512 blocks in a slab. The first block in a slab is used by the collector itself, to keep metadata for the rest of the blocks, for example a chain pointer allowing blocks to be collected in lists, a saved allocation pointer for partially-filled blocks, whether the block is paged in or out, and so on.

The PCC not only supports parallel mutators, it can also trace in parallel. This mechanism works somewhat like allocation, in which multiple trace workers compete to evacuate objects into their local allocation buffers; when an allocation buffer is full, the trace worker grabs another, just like mutators do.

However, unlike the simple semi-space collector which uses a Cheney grey worklist, the PCC uses the [fine-grained work-stealing parallel
tracer](https://github.com/wingo/whippet/blob/main/src/parallel-tracer.h) originally developed for Whippet’s Immix-like collector. Each trace worker maintains a [local queue of objects that need
tracing](https://github.com/wingo/whippet/blob/main/src/local-worklist.h), which currently has 1024 entries. If the local queue becomes full, the worker will publish 3/4 of those entries to the worker’s [shared
worklist](https://github.com/wingo/whippet/blob/main/src/shared-worklist.h). When a worker runs out of local work, it will first try to remove work from its own shared worklist, then will try to steal from other workers.

Of course, because threads compete to evacuate objects, we have to use [atomic compare-and-swap instead of simple forwarding pointer
updates](https://github.com/wingo/whippet/blob/main/doc/manual.md#forwarding-objects); if you only have one mutator thread and are OK with just one tracing thread, you can avoid the ~30% performance penalty that atomic operations impose. The PCC generally starts to win over a semi-space collector when it can trace with 2 threads, and gets better with each thread you add.

I sometimes wonder whether the worklist should contain grey edges or grey objects. [MMTk](https://www.mmtk.io/) seems to do the former, and bundles edges into work packets, which are the unit of work sharing. I don’t know yet what is best and look forward to experimenting once I have better benchmarks.

Anyway, maintaining an external worklist is cheating in a way: unlike the Cheney worklist, this memory is not accounted for as part of the heap size. If you are targetting a microcontroller or something, probably you need to choose a different kind of collector. Fortunately, Whippet enables this kind of choice, as it contains a number of collector implementations.

What about benchmarks? Well, I’ll be doing those soon in a more rigorous way. For now I will just say that it seems to behave as expected and I am satisfied; it is useful to have a performance oracle against which to compare other collectors.

### finalizers!

This week I landed support for [finalizers](https://github.com/wingo/whippet/blob/main/api/gc-finalizer.h)!

Finalizers work in all the collectors: semi, pcc, whippet, and the BDW collector that is a shim to use BDW-GC behind the Whippet API. They have a well-defined relationship with [ephemerons](https://wingolog.org/archives/2022/10/31/ephemerons-and-finalizers) and are [multi-priority, allowing embedders to build guardians or
phantom references on
top](https://wingolog.org/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera).

In the future I should do some more work to make finalizers support generations, if the collector is generational, allowing a minor collection to avoid visiting finalizers for old objects. But this is a straightforward extension that will happen at some point.

### future!

And that’s the end of this update. Next up, I am finally going to tackle [heap resizing, using the membalancer
approach](https://wingolog.org/archives/2023/01/27/three-approaches-to-heap-sizing). Then basic Windows and Mac support, then I move on to the tracing and performance measurement phase. Onwards and upwards!

## related articles

- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [whippet lab notebook: untagged mallocs, bis](/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [preliminary notes on a nofl field-logging barrier](/archives/2024/10/03/preliminary-notes-on-a-nofl-field-logging-barrier)
- [whippet progress update: feature-complete!](/archives/2024/09/18/whippet-progress-update-feature-complete)

### One response

1. Hello says:[9 August 2024 5:54 PM](#798996ef2d4799a7c9b378b4a49662db073c3a70)

   Nice article, haven’t read in long time..

Comments are closed.
