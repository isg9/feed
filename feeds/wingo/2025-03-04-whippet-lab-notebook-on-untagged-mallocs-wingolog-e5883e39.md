---
title: 'whippet lab notebook: on untagged mallocs — wingolog'
url: https://wingolog.org/archives/2025/03/04/whippet-lab-notebook-on-untagged-mallocs
published: "2025-03-04T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2025/03/04/whippet-lab-notebook-on-untagged-mallocs
---

# whippet lab notebook: on untagged mallocs — wingolog

## [whippet lab notebook: on untagged mallocs](/archives/2025/03/04/whippet-lab-notebook-on-untagged-mallocs)

4 March 2025 3:42 PM

- [whippet](/tags/whippet)
- [gc](/tags/gc)
- [bdw](/tags/bdw)
- [guile](/tags/guile)
- [igalia](/tags/igalia)
- [malloc](/tags/malloc)

Salutations, populations. Today’s note is more of a work-in-progress than usual; I have been finally starting to look at getting [Whippet](https://github.com/wingo/whippet) into [Guile](https://gnu.org/s/guile), and there are some open questions.

### inventory

I started by taking a look at how Guile uses the [Boehm-Demers-Weiser
collector](https://github.com/ivmai/bdwgc)‘s API, to make sure I had all my bases covered for an eventual switch to something that was not BDW. I think I have a good overview now, and have divided the parts of BDW-GC used by Guile into seven categories.

#### implicit uses

Firstly there are the ways in which Guile’s run-time and compiler depend on BDW-GC’s behavior, without actually using BDW-GC’s API. By this I mean principally that we assume that any reference to a GC-managed object from any thread’s stack will keep that object alive. The same goes for references originating in global variables, or static data segments more generally. Additionally, we rely on GC objects not to move: references to GC-managed objects in registers or stacks are valid across a GC boundary, even if those references are outside the GC-traced graph: all objects are pinned.

Some of these “uses” are internal to Guile’s implementation itself, and thus amenable to being changed, albeit with some effort. However some escape into the wild via Guile’s API, or, as in this case, as implicit behaviors; these are hard to change or evolve, which is why I am putting my hopes on Whippet’s [mostly-marking
collector](https://github.com/wingo/whippet/blob/main/doc/collector-mmc.md), which allows for conservative roots.

#### defensive uses

Then there are the uses of BDW-GC’s API, not to accomplish a task, but to protect the mutator from the collector: [`GC_call_with_alloc_lock`](https://github.com/ivmai/bdwgc/blob/master/include/gc/gc.h#L1608), explicitly enabling or disabling GC, calls to `sigmask` that take BDW-GC’s use of POSIX signals into account, and so on. BDW-GC can stop any thread at any time, between any two instructions; for most users is anodyne, but if ever you use weak references, things start to get really gnarly.

Of course a new collector would have its own constraints, but switching to cooperative instead of pre-emptive safepoints would be a welcome relief from this mess. On the other hand, we will require client code to explicitly mark their threads as inactive during calls in more cases, to ensure that all threads can promptly reach safepoints at all times. Swings and roundabouts?

#### precise tracing

Did you know that the Boehm collector allows for precise tracing? It does! It’s slow and truly gnarly, but when you need precision, precise tracing nice to have. (This is the [`GC_new_kind`](https://github.com/ivmai/bdwgc/blob/master/include/gc/gc_mark.h) interface.) Guile uses it to mark Scheme stacks, allowing it to avoid treating unboxed locals as roots. When it loads compiled files, Guile also adds some sliced of the mapped files to the root set. These interfaces will need to change a bit in a switch to Whippet but are ultimately internal, so that’s fine.

What is not fine is that Guile allows C users to hook into precise tracing, notably via [`scm_smob_set_mark`](https://www.gnu.org/software/guile/docs/docs-2.0/guile-ref/Smobs.html). This is not only the wrong interface, not allowing for copying collection, but these functions are just truly gnarly. I don’t know know what to do with them yet; are our external users ready to forgo this interface entirely? We have been working on them over time, but I am not sure.

#### reachability

Weak references, weak maps of various kinds: the implementation of these in terms of BDW’s API is incredibly gnarly and ultimately unsatisfying. We will be able to replace all of these with ephemerons and tables of ephemerons, which are natively supported by Whippet. The same goes with finalizers.

The same goes for constructs built on top of finalizers, such as [guardians](https://wingolog.org/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera); we’ll get to reimplement these on top of nice Whippet-supplied primitives. Whippet allows for resuscitation of finalized objects, so all is good here.

#### misc

There is a long list of miscellanea: the interfaces to explicitly trigger GC, to get statistics, to control the number of marker threads, to initialize the GC; these will change, but all uses are internal, making it not a terribly big deal.

I should mention one API concern, which is that BDW’s state is all implicit. For example, when you go to allocate, you don’t pass the API a handle which you have obtained for your thread, and which might hold some thread-local freelists; BDW will instead load thread-local variables in its API. That’s not as efficient as it could be and Whippet goes the explicit route, so there is some additional plumbing to do.

Finally I should mention the true miscellaneous BDW-GC function: `GC_free`. Guile exposes it via an API, `scm_gc_free`. It was already vestigial and we should just remove it, as it has no sensible semantics or implementation.

#### allocation

That brings me to what I wanted to write about today, but am going to have to finish tomorrow: the actual allocation routines. BDW-GC provides two, essentially: `GC_malloc` and `GC_malloc_atomic`. The difference is that “atomic” allocations don’t refer to other GC-managed objects, and as such are well-suited to raw data. Otherwise you can think of atomic allocations as a pure optimization, given that BDW-GC mostly traces conservatively anyway.

From the perspective of a user of BDW-GC looking to switch away, there are two broad categories of allocations, tagged and untagged.

Tagged objects have attached metadata bits allowing their type to be inspected by the user later on. This is the happy path! We’ll be able to write a `gc_trace_object` function that takes any object, does a switch on, say, some bits in the first word, dispatching to type-specific tracing code. As long as the object is sufficiently initialized by the time the next safepoint comes around, we’re good, and given cooperative safepoints, the compiler should be able to ensure this invariant.

Then there are untagged allocations. Generally speaking, these are of two kinds: temporary and auxiliary. An example of a temporary allocation would be growable storage used by a C run-time routine, perhaps as an unbounded-sized alternative to `alloca`. Guile uses these a fair amount, as they compose well with non-local control flow as occurring for example in exception handling.

An auxiliary allocation on the other hand might be a data structure only referred to by the internals of a tagged object, but which itself never escapes to Scheme, so you never need to inquire about its type; it’s convenient to have the lifetimes of these values managed by the GC, and when desired to have the GC automatically trace their contents. Some of these should just be folded into the allocations of the tagged objects themselves, to avoid pointer-chasing. Others are harder to change, notably for mutable objects. And the trouble is that for external users of `scm_gc_malloc`, I fear that we won’t be able to migrate them over, as we don’t know whether they are making tagged mallocs or not.

### what is to be done?

One conventional way to handle untagged allocations is to manage to fit your data into other tagged data structures; V8 does this in many places with instances of FixedArray, for example, and Guile should do more of this. Otherwise, you make new tagged data types. In either case, all auxiliary data should be tagged.

I think there may be an alternative, which would be just to support the equivalent of untagged `GC_malloc` and `GC_malloc_atomic`; but for that, I am out of time today, so type at y’all tomorrow. Happy hacking!

## related articles

- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [whippet lab notebook: untagged mallocs, bis](/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)
- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)

Comments are closed.
