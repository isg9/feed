---
title: v8's mark-sweep nursery — wingolog
url: https://wingolog.org/archives/2023/12/08/v8s-mark-sweep-nursery
published: "2023-12-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/12/08/v8s-mark-sweep-nursery
---

# v8's mark-sweep nursery — wingolog

## [v8's mark-sweep nursery](/archives/2023/12/08/v8s-mark-sweep-nursery)

8 December 2023 2:34 PM

- [v8](/tags/v8)
- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [javascript](/tags/javascript)
- [igalia](/tags/igalia)
- [mark-sweep](/tags/mark-sweep)
- [generational gc](/tags/generational%20gc)

Today, a followup to [yesterday’s note](https://wingolog.org/archives/2023/12/07/the-last-5-years-of-v8s-garbage-collector) with some more details on V8’s new young-generation implementation, *minor mark-sweep* or *MinorMS*.

A caveat again: these observations are just from reading the code; I haven’t run these past the MinorMS authors yet, so any of these details might be misunderstandings.

The MinorMS nursery consists of *pages*, each of which is 256 kB, unless huge-page mode is on, in which case they are 2 MB. The total default size of the nursery is 72 MB by default, or 144 MB if [pointer
compression](https://v8.dev/blog/pointer-compression) is off.

There can be multiple threads allocating into the nursery, but let’s focus on the [main
allocator](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/main/src/heap/main-allocator.h), which is used on the main thread. Nursery allocation is bump-pointer, whether in a MinorMS page or scavenger semi-space. Bump-pointer regions are called *linear allocation buffers*, and often abbreviated as `Lab` in the source, though the class is [`LinearAllocationArea`](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/main/src/heap/linear-allocation-area.h).

If the current bump-pointer region is too small for the current allocation, the nursery implementation finds another one, or triggers a collection. For the MinorMS nursery, each page collects the set of allocatable spans in a free list; if the free-list is non-empty, it pops off one entry as the current and tries again.

Otherwise, MinorMS needs another page, and specifically a *swept page*: a page which has been visited since the last GC, and whose spans of unused memory have been collected into a free-list. There is a concurrent sweeping task which should usually run ahead of the mutator, but if there is no swept page available, the allocator might need to sweep some. This logic is in [`MainAllocator::RefillLabMain`](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/main/src/heap/main-allocator.cc#625).

Finally, if all pages are swept and there’s no Lab big enough for the current allocation, we trigger collection from the roots. The initial roots are the *remembered set*: pointers from old objects to new objects. Most of the trace happens concurrently with the mutator; when the nursery utilisation rises over 90%, V8 will kick off concurrent marking tasks.

Then once the mutator actually runs out of space, it pauses, drains any pending marking work, marks conservative roots, then drains again. I am not sure whether MinorMS with conservative stack scanning visits the whole C/C++ stacks or whether it manages to install some barriers (i.e. “don’t scan deeper than 5 frames because we collected then, and so all older frames are older”); dunno. All of this logic is in [`MinorMarkSweepCollector::MarkLiveObjects`](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/main/src/heap/minor-mark-sweep.cc#618).

Marking traces the object graph, setting object mark bits. It does not trace pages. However, the MinorMS space promotes in units of pages. So how to decide what pages to promote? The answer is that *sweeping* partitions the MinorMS pages into empty, recycled, aging, and promoted pages.

Empty pages have no surviving objects, and are very useful because they can be given back to the operating system if needed or shuffled around elsewhere in the system. If they are re-used for allocation, they do not need to be swept.

Recycled pages have some survivors, but not many; MinorMS keeps the page around for allocation in the next cycle, because it has enough empty space. By default, a page is recyclable if it has 50% or more free space after a minor collection, or 30% after a major collection. MinorMS also promotes a page eagerly if in the last cycle, we only managed to allocate into 30% or less of its empty space, probably due to fragmentation. These pages need to be swept before re-use.

Finally, MinorMS doesn’t let pages be recycled indefinitely: after 4 minor cycles, a page goes into the *aging* pool, in which it is kept unavailable for allocation for one cycle, but is not yet promoted. This allows any new allocations on that page in the previous cycle age out and probably die, preventing premature tenuring.

And that’s it. Next time, a note on a way in which generational collectors can run out of memory. Have a nice weekend, hackfolk!

## related articles

- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)
- [the last couple years in v8's garbage collector](/archives/2025/11/13/the-last-couple-years-in-v8s-garbage-collector)
- [the last 5 years of V8's garbage collector](/archives/2023/12/07/the-last-5-years-of-v8s-garbage-collector)
- [ahead-of-time wasm gc in wastrel](/archives/2026/02/06/ahead-of-time-wasm-gc-in-wastrel)
- [v8's precise field-logging remembered set](/archives/2024/01/05/v8s-precise-field-logging-remembered-set)
- [the sticky mark-bit algorithm](/archives/2022/10/22/the-sticky-mark-bit-algorithm)

### One response

1. Hubert says:[9 December 2023 4:42 PM](#2ad090193483b63fc92b87982bbdb5daa57ac9a4)

   The markdown parser can make it challenging to write comments to Andy Wingo’s excellent blog.

   A simple recipe that seems to work for me is to: 1) start each new paragraph with a space (before the first letter of text), and 2) write at least two paragraphs (each started by a space, separate paragraphs with a blank line).

   I hope this helps others who may have faced similar challenges.

Comments are closed.
