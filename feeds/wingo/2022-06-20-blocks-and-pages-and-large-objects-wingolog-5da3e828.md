---
title: blocks and pages and large objects — wingolog
url: https://wingolog.org/archives/2022/06/20/blocks-and-pages-and-large-objects
published: "2022-06-20T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2022/06/20/blocks-and-pages-and-large-objects
---

# blocks and pages and large objects — wingolog

## [blocks and pages and large objects](/archives/2022/06/20/blocks-and-pages-and-large-objects)

20 June 2022 2:59 PM

- [garbage collection](/tags/garbage%20collection)
- [immix](/tags/immix)
- [guile](/tags/guile)
- [gnu](/tags/gnu)
- [igalia](/tags/igalia)
- [mmap](/tags/mmap)

Good day! In a [recent dispatch](https://wingolog.org/archives/2022/06/15/defragmentation) we talked about the fundamental garbage collection algorithms, also introducing the Immix mark-region collector. Immix mostly leaves objects in place but can move objects if it thinks it would be profitable. But when would it decide that this is a good idea? Are there cases in which it is necessary?

I promised to answer those questions in a followup article, but I didn't say which followup :) Before I get there, I want to talk about paged spaces.

**enter the multispace**

We mentioned that Immix divides the heap into blocks (32kB or so), and that no object can span multiple blocks. "Large" objects -- defined by Immix to be more than 8kB -- go to a separate "large object space", or "lospace" for short.

Though the implementation of a large object space is relatively simple, I found that it has some points that are quite subtle. Probably the most important of these points relates to heap size. Consider that if you just had one space, implemented using mark-compact maybe, then the procedure to allocate a 16 kB object would go:

1. Try to bump the allocation pointer by 16kB. Is it still within range? If so we are done.

2. Otherwise, collect garbage and try again. If after GC there isn't enough space, the allocation fails.

In step (2), collecting garbage could decide to grow or shrink the heap. However when evaluating collector algorithms, you generally want to avoid dynamically-sized heaps.

**cheatery**

Here is where I need to make an embarrassing admission. In my role as co-maintainer of the [Guile](https://gnu.org/s/guile) programming language implementation, I have long noodled around with benchmarks, comparing Guile to Chez, Chicken, and other implementations. It's good fun. However, I only realized recently that I had a magic knob that I could turn to win more benchmarks: simply make the heap bigger. Make it start bigger, make it grow faster, whatever it takes. For a program that does its work in some fixed amount of total allocation, a bigger heap will require fewer collections, and therefore generally take less time. (Some amount of collection may be good for performance as it improves locality, but this is a marginal factor.)

Of course I didn't really go wild with this knob but it now makes me doubt all benchmarks I have ever seen: are we really using benchmarks to select for fast implementations, or are we in fact selecting for implementations with cheeky heap size heuristics? Consider even any of the common allocation-heavy JavaScript benchmarks, DeltaBlue or Earley or the like; to win these benchmarks, web browsers are incentivised to have large heaps. In the real world, though, a more parsimonious policy might be more appreciated by users.

Java people have known this for quite some time, and are therefore used to fixing the heap size while running benchmarks. For example, people will measure the minimum amount of memory that can allow a benchmark to run, and then configure the heap to be a constant multiplier of this minimum size. The [MMTK](https://mmtk.io) garbage collector toolkit can't even grow the heap at all currently: it's an important feature for production garbage collectors, but as they are just now migrating out of the research phase, heap growth (and shrinking) hasn't yet been a priority.

**lospace**

So now consider a garbage collector that has two spaces: an Immix space for allocations of 8kB and below, and a large object space for, well, larger objects. How do you divide the available memory between the two spaces? Could the balance between immix and lospace change at run-time? If you never had large objects, would you be wasting space at all? Conversely is there a strategy that can also work for only large objects?

Perhaps the answer is obvious to you, but it wasn't to me. After much reading of the MMTK source code and pondering, here is what I understand the state of the art to be.

1. Arrange for your main space -- Immix, mark-sweep, whatever -- to be block-structured, and able to dynamically decomission or recommission blocks, perhaps via [`MADV_DONTNEED`](https://man7.org/linux/man-pages/man2/madvise.2.html). This works if the blocks are even multiples of the underlying OS page size.

2. Keep a counter of however many bytes the lospace currently has.

3. When you go to allocate a large object, increment the lospace byte counter, and then round up to number of blocks to decommission from the main paged space. If this is more than are currently decommissioned, find some empty blocks and decommission them.

4. If no empty blocks were found, collect, and try again. If the second try doesn't work, then the allocation fails.

5. Now that the paged space has shrunk, lospace can allocate. You can use the system `malloc`, but probably better to use `mmap`, so that if these objects are collected, you can just `MADV_DONTNEED` them and keep them around for later re-use.

6. After GC runs, explicitly return the memory for any object in lospace that wasn't visited when the object graph was traversed. Decrement the lospace byte counter and possibly return some empty blocks to the paged space.

There are some interesting aspects about this strategy. One is, the memory that you return to the OS doesn't need to be contiguous. When allocating a 50 MB object, you don't have to find 50 MB of contiguous free space, because any set of blocks that adds up to 50 MB will do.

Another aspect is that this adaptive strategy can work for any ratio of large to non-large objects. The user doesn't have to manually set the sizes of the various spaces.

This strategy does assume that address space is larger than heap size, but only by a factor of 2 (modulo fragmentation for the large object space). Therefore our risk of running afoul of user resource limits and kernel overcommit heuristics is low.

The one underspecified part of this algorithm is... did you see it? "Find some empty blocks". If the main paged space does lazy sweeping -- only scanning a block for holes right before the block will be used for allocation -- then after a collection we don't actually know very much about the heap, and notably, we don't know what blocks are empty. (We could know it, of course, but it would take time; you could traverse the line mark arrays for all blocks while the world is stopped, but this increases pause time. The original Immix collector does this, however.) In the system I've been working on, instead I have it so that if a mutator finds an empty block, it puts it on a separate list, and then takes another block, only allocating into empty blocks once all blocks are swept. If the lospace needs blocks, it sweeps eagerly until it finds enough empty blocks, throwing away any nonempty blocks. This causes the next collection to happen sooner, but that's not a terrible thing; this only occurs when rebalancing lospace versus paged-space size, because if you have a constant allocation rate on the lospace side, you will also have a complementary rate of production of empty blocks by GC, as they are recommissioned when lospace objects are reclaimed.

What if your main paged space has ample space for allocating a large object, but there are no empty blocks, because live objects are equally peppered around all blocks? In that case, often the application would be best served by growing the heap, but maybe not. In any case in a strict-heap-size environment, we need a solution.

But for that... let's pick up another day. Until then, happy hacking!

## related articles

- [an optimistic evacuation of my wordhoard](/archives/2022/06/21/an-optimistic-evacuation-of-my-wordhoard)
- [coarse or lazy?](/archives/2022/08/07/coarse-or-lazy)
- [unintentional concurrency](/archives/2022/07/20/unintentional-concurrency)
- [defragmentation](/archives/2022/06/15/defragmentation)
- [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)
- [copying collectors with block-structured heaps are unreliable](/archives/2024/07/10/copying-collectors-with-block-structured-heaps-are-unreliable)

Comments are closed.
