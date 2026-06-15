---
title: unintentional concurrency — wingolog
url: https://wingolog.org/archives/2022/07/20/unintentional-concurrency
published: "2022-07-20T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2022/07/20/unintentional-concurrency
---

# unintentional concurrency — wingolog

## [unintentional concurrency](/archives/2022/07/20/unintentional-concurrency)

20 July 2022 9:26 PM

- [garbage collection](/tags/garbage%20collection)
- [immix](/tags/immix)
- [guile](/tags/guile)
- [gnu](/tags/gnu)
- [igalia](/tags/igalia)
- [evacuation](/tags/evacuation)
- [concurrency](/tags/concurrency)

Good evening, gentle hackfolk. [Last time](https://wingolog.org/archives/2022/06/21/an-optimistic-evacuation-of-my-wordhoard) we talked about heuristics for when you might want to compact a heap. Compacting garbage collection is nice and tidy and appeals to our orderly instincts, and it enables heap shrinking and reallocation of pages to large object spaces and it can reduce fragmentation: all very good things. But evacuation is more expensive than just marking objects in place, and so a production garbage collector will usually just mark objects in place, and only compact or evacuate when needed.

Today's post is more details!

**dedication**

Just because it's been, oh, a couple decades, I would like to reintroduce a term I learned from [Marnanel](https://marnanel.org/) years ago on [advogato](https://advogato.org/), a nerdy group blog kind of a site. As I recall, there is a word that originates in the Oxbridge social environment, "narg", from "Not A Real Gentleman", and which therefore denotes things that not-real-gentlemen do: nerd out about anything that's not, like, fox-hunting or golf; or generally spending time on something not because it will advance you in conventional hierarchies but because you just can't help it, because you love it, because it is just your thing. Anyway, in the spirit of pursuits that are really not What One Does With One's Time, this post is dedicated to the word "nargery".

**side note, bis: immix-style evacuation versus mark-compact**

In my last post I described Immix-style evacuation, and noted that it might take a few cycles to fully compact the heap, and that it has a few pathologies: the heap might never reach full compaction, and that Immix might run out of free blocks in which to evacuate.

With these disadvantages, why bother? Why not just do a single mark-compact pass and be done? I implicitly asked this question last time but didn't really answer it.

For some people will be, yep, yebo, mark-compact is the right answer. And yet, there are a few reasons that one might choose to evacuate a fraction of the heap instead of compacting it all at once.

The first reason is object pinning. Mark-compact systems assume that all objects can be moved; you can't usefully relax this assumption. Most algorithms "slide" objects down to lower addresses, squeezing out the holes, and therefore every live object's address needs to be available to use when sliding down other objects with higher addresses. And yet, it would be nice sometimes to prevent an object from being moved. This is the case, for example, when you grant a foreign interface (e.g. a C function) access to a buffer: if garbage collection happens while in that foreign interface, it would be nice to be able to prevent garbage collection from moving the object out from under the C function's feet.

Another reason to want to pin an object is because of conservative root-finding. Guile currently uses the Boehm-Demers-Weiser collector, which conservatively scans the stack and data segments for anything that looks like a pointer to the heap. The garbage collector can't update such a global root in response to compaction, because you can't be sure that a given word is a pointer and not just an integer with an inconvenient value. In short, objects referenced by conservative roots need to be pinned. I would like to support precise roots at some point but part of my interest in Immix is to allow Guile to move to a better GC algorithm, without necessarily requiring precise enumeration of GC roots. Optimistic partial evacuation allows for the possibility that any given evacuation might fail, which makes it appropriate for conservative root-finding.

Finally, as moving objects has a cost, it's reasonable to want to only incur that cost for the part of the heap that needs it. In any given heap, there will likely be some data that stays live across a series of collections, and which, once compacted, can't be profitably moved for many cycles. Focussing evacuation on only the part of the heap with the lowest survival rates avoids wasting time on copies that don't result in additional compaction.

(I should admit one thing: sliding mark-compact compaction preserves allocation order, whereas evacuation does not. The memory layout of sliding compaction is more optimal than evacuation.)

**multi-cycle evacuation**

Say a mutator runs out of memory, and therefore invokes the collector. The collector decides for whatever reason that we should evacuate at least part of the heap instead of marking in place. How much of the heap can we evacuate? The answer depends primarily on how many free blocks you have reserved for evacuation. These are known-empty blocks that haven't been allocated into by the last cycle. If you don't have any, you can't evacuate! So probably you should keep some around, even when performing in-place collections. The Immix papers suggest 2% and that works for me too.

Then you evacuate some blocks. Hopefully the result is that after this collection cycle, you have more free blocks. But you haven't compacted the heap, at least probably not on the first try: not into 2% of total space. Therefore you tell the mutator to put any empty blocks it finds as a result of lazy sweeping during the next cycle onto the evacuation target list, and then the next cycle you have more blocks to evacuate into, and more and more and so on until after some number of cycles you fall below some overall heap fragmentation low-watermark target, at which point you can switch back to marking in place.

I don't know how this works in practice! In my test setups which triggers compaction at 10% fragmentation and continues until it drops below 5%, it's rare that it takes more than 3 cycles of evacuation until the heap drops to effectively 0% fragmentation. Of course I had to introduce fragmented allocation patterns into the microbenchmarks to even cause evacuation to happen at all. I look forward to some day soon testing with real applications.

**concurrency**

Just as a terminological note, in the world of garbage collectors, "parallel" refers to multiple threads being used by a garbage collector. Parallelism within a collector is essentially an implementation detail; when the world is stopped for collection, the mutator (the user program) generally doesn't care if the collector uses 1 thread or 15. On the other hand, "concurrent" means the collector and the mutator running at the same time.

Different parts of the collector can be concurrent with the mutator: for example, sweeping, marking, or evacuation. Concurrent sweeping is just a detail, because it just visits dead objects. Concurrent marking is interesting, because it can significantly reduce stop-the-world pauses by performing most of the computation while the mutator is running. It's tricky, as you might imagine; the collector traverses the object graph while the mutator is, you know, mutating it. But there are standard techniques to make this work. [Concurrent evacuation is a nightmare.](https://scholar.google.com/citations?view_op=view_citation&hl=en&user=HgSTO7oAAAAJ&sortby=pubdate&citation_for_view=HgSTO7oAAAAJ:kzcrU_BdoSEC) It's not that you can't implement it; you can. But it's very very hard to get an overall performance win from concurrent evacuation/copying.

So if you are looking for a good bargain in the marketplace of garbage collector algorithms, it would seem that you need to avoid concurrent copying/evacuation. It's an expensive product that would seem to not buy you very much.

All that is just a prelude to an observation that there is a funny source of concurrency even in some systems that don't see themselves as concurrent: mutator threads marking their own roots. To recall, when you stop the world for a garbage collection, all mutator threads have to somehow notice the request to stop, reach a safepoint, and then stop. Then the collector traces the roots from all mutators and everything they reference, transitively. Then you let the threads go again. Thing is, once you get more than a thread or four, stopping threads can take time. You'd be tempted to just have threads notice that they need to stop, then traverse their own stacks at their own safepoint to find their roots, then stop. But, this introduces concurrency between root-tracing and other mutators that might not have seen the request to stop. For marking, this concurrency can be fine: you are just setting mark bits, not mutating the roots. You might need to add an additional mark pattern that can be distinguished from marked-last-time and marked-the-time-before-but-dead-now, but that's a detail. Fine.

But if you instead start an evacuating collection, the gates of hell open wide and toothy maws and horns fill your vision. One thread could be stopping and evacuating the objects referenced by its roots, while another hasn't noticed the request to stop and is happily using the same objects: chaos! You are trying to make a minor optimization to move some work out of the stop-the-world phase but instead everything falls apart.

Anyway, this whole article was really to get here and note that you can't do ragged-stops with evacuation without supporting full concurrent evacuation. Otherwise, you need to postpone root traversal until all threads are stopped. Perhaps this is another argument that evacuation is expensive, relative to marking in place. In practice I haven't seen the ragged-stop effect making so much of a difference, but perhaps that is because evacuation is infrequent in my test cases.

Zokay? Zokay. Welp, this evening's nargery was indeed nargy. Happy hacking to all collectors out there, and until next time.

## related articles

- [coarse or lazy?](/archives/2022/08/07/coarse-or-lazy)
- [an optimistic evacuation of my wordhoard](/archives/2022/06/21/an-optimistic-evacuation-of-my-wordhoard)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)
- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)
- [blocks and pages and large objects](/archives/2022/06/20/blocks-and-pages-and-large-objects)

### 2 responses

1. [Tim Chevalier](https://blogs.igalia.com/tjc) says:[21 July 2022 6:20 PM](#110e63b2520dcf3d9d10d90ee10e53025a17fad5)

   I'm confused about "mutator threads marking their own roots" -- what's the interface for the mutator to do the marking? Does the compiler insert calls to some GC library function (in the systems you're talking about here)?

2. [wingo](https://wingolog.org) says:[22 July 2022 7:31 AM](#6cbfe79ad0683cd760127c41fe6d9ddd8d873b08)

   Hum, good question -- a brief answer: the life of a mutator is not just the user program. It also necessarily hooks into the garbage collector to allocate new objects, to write new references into existing objects (via a write barrier), and sometimes when reading object fields (a read barrier; very expensive, mostly applies to concurrent-copying collectors).

   But there's more. A collector with precise roots might have to keep some ongoing bookkeeping information about where references are stored on the stack. (This is the C++ "handle" API in V8 or SpiderMonkey, for example. Most systems manage to avoid this overhead in JIT-generated code though, by keeping side tables indicating where roots are stored at the points at which a thread can stop.) This is also mutator overhead. And when a thread triggers garbage collection, it effectively does so by handing off control to the GC, but within the same thread -- via a library call.

   And finally, to answer your question, in systems with parallel mutator threads, all threads have to somehow arrange to stop when other threads trigger GC. The BDW collector that Guile uses does so via signals -- brutally effective. But in systems with precise roots, the precondition is that when the thread stops, you need to be able to enumerate all storage locations that can hold references to the heap. Usually that is only possible at safepoints, where you have root information in a side table. So you arrange for each thread to sometimes check for safepoints.

   Once a mutator thread is stopped, you can identify its roots, because they won't change any more. You could do that from whatever thread caused GC, or from worker threads. But the thread that is best-suited to do it is the stopping mutator thread itself, because presumably that data is all local to the CPU that is running the thread. It's cheaper for a thread to scan its own stack than for a "foreign" thread to do so. So usually as part of whatever library routine from the GC that a mutator calls to stop itself, the thread marks its own stack. But since all threads don't notice that they need to stop at the same time, you get a "ragged stop" effect: some threads have moved on to the mark-your-stack phase, but some are still running the user program.

Comments are closed.
