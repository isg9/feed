---
title: coarse or lazy? — wingolog
url: https://wingolog.org/archives/2022/08/07/coarse-or-lazy
published: "2022-08-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2022/08/07/coarse-or-lazy
---

# coarse or lazy? — wingolog

## [coarse or lazy?](/archives/2022/08/07/coarse-or-lazy)

7 August 2022 9:44 AM

- [garbage collection](/tags/garbage%20collection)
- [immix](/tags/immix)
- [guile](/tags/guile)
- [gnu](/tags/gnu)
- [igalia](/tags/igalia)
- [evacuation](/tags/evacuation)
- [concurrency](/tags/concurrency)
- [sweeping](/tags/sweeping)

**sweeping, coarse and lazy**

One of the things that had perplexed me about the Immix collector was [how to effectively defragment the heap via evacuation while keeping just 2-3% of space as free blocks for an evacuation reserve](https://wingolog.org/archives/2022/06/21/an-optimistic-evacuation-of-my-wordhoard). The [original Immix paper](http://users.cecs.anu.edu.au/~steveb/pubs/papers/immix-pldi-2008.pdf) states:

> To evacuate the object, the collector uses the same allocator as the mutator, continuing allocation right where the mutator left off. Once it exhausts any unused recyclable blocks, it uses any completely free blocks. By default, immix sets aside a small number of free blocks that it never returns to the global allocator and only ever uses for evacuating. This *headroom* eases defragmentation and is counted against immix's overall heap budget. By default immix reserves 2.5% of the heap as compaction headroom, but \[...\] is fairly insensitive to values ranging between 1 and 3%.

To Immix, a "recyclable" block is partially full: it contains surviving data from a previous collection, but also some holes in which to allocate. But when would you have recyclable blocks at evacuation-time? Evacuation occurs as part of collection. Collection usually occurs when there's no more memory in which to allocate. At that point any recyclable block would have been allocated into already, and won't become recyclable again until the next trace of the heap identifies the block's surviving data. Of course after the next trace they could become "empty", if no object survives, or "full", if all lines have survivor objects.

In general, after a full allocation cycle, you don't know much about the heap. If you could easily know where the live data and the holes were, a garbage collector's job would be much easier :) Any algorithm that starts from the assumption that you know where the holes are can't be used before a heap trace. So, I was not sure what the Immix paper is meaning here about allocating into recyclable blocks.

Thinking on it again, I realized that Immix might trigger collection early sometimes, before it has exhausted the previous cycle's set of blocks in which to allocate. As we discussed earlier, [there is a case in which you might want to trigger an early compaction](https://wingolog.org/archives/2022/06/21/an-optimistic-evacuation-of-my-wordhoard): when a large object allocator runs out of blocks to decommission from the immix space. And if one evacuating collection didn't yield enough free blocks, you might trigger the next one early, reserving some recyclable and empty blocks as evacuation targets.

**when do you know what you know: lazy and eager**

Consider a basic question, such as "how many bytes in the heap are used by live objects". In general you don't know! Indeed you often never know precisely. For example, concurrent collectors often have some amount of "floating garbage" which is unreachable data but which survives across a collection. And of course you don't know the difference between floating garbage and precious data: if you did, you would have collected the garbage.

Even the idea of "when" is tricky in systems that allow parallel mutator threads. Unless the program has a total ordering of mutations of the object graph, there's no one timeline with respect to which you can measure the heap. Still, Immix is a stop-the-world collector, and since such collectors synchronously trace the heap while mutators are stopped, these are times when you can exactly compute properties about the heap.

Let's retake the question of measuring live bytes. For an evacuating semi-space, knowing the number of live bytes after a collection is trivial: all survivors are packed into to-space. But for a mark-sweep space, you would have to compute this information. You could compute it at mark-time, while tracing the graph, but doing so takes time, which means delaying the time at which mutators can start again.

Alternately, for a mark-sweep collector, you can compute free bytes at sweep-time. This is the phase in which you go through the whole heap and return any space that wasn't marked in the last collection to the allocator, allowing it to be used for fresh allocations. This is the point in the garbage collection cycle in which you can answer questions such as "what is the set of recyclable blocks": you know what is garbage and you know what is not.

Though you could sweep during the stop-the-world pause, you don't have to; sweeping only touches dead objects, so it is correct to allow mutators to continue and then sweep as the mutators run. There are two general strategies: spawn a thread that sweeps as fast as it can (concurrent sweeping), or make mutators sweep as needed, just before they allocate (lazy sweeping). But this introduces a lag between when you know and what you know—your count of total live heap bytes describes a time in the past, not the present, because mutators have moved on since then.

For most collectors with a sweep phase, deciding between eager (during the stop-the-world phase) and deferred (concurrent or lazy) sweeping is very easy. You don't immediately need the information that sweeping allows you to compute; it's quite sufficient to wait until the next cycle. Moving work out of the stop-the-world phase is a win for mutator responsiveness (latency). Usually people implement lazy sweeping, as it is naturally incremental with the mutator, naturally parallel for parallel mutators, and any sweeping overhead due to cache misses can be mitigated by immediately using swept space for allocation. The case for concurrent sweeping is less clear to me, but if you have cores that would otherwise be idle, sure.

**eager coarse sweeping**

Immix is interesting in that it chooses to sweep eagerly, during the stop-the-world phase. Instead of sweeping irregularly-sized objects, however, it sweeps over its "line mark" array: one byte for each 128-byte "line" in the mark space. For 32 kB blocks, there will be 256 bytes per block, and line mark bytes in each 4 MB slab of the heap are packed contiguously. Therefore you get relatively good locality, but this just mitigates a cost that other collectors don't have to pay. So what does eager marking over these coarse 128-byte regions buy Immix?

Firstly, eager sweeping buys you eager identification of empty blocks. If your large object space needs to steal blocks from the mark space, but the mark space doesn't have enough empties, it can just trigger collection and then it knows if enough blocks are available. If no blocks are available, you can grow the heap or signal out-of-memory. If the lospace (large object space) runs out of blocks before the mark space has used all recyclable blocks, that's no problem: evacuation can move the survivors of fragmented blocks into these recyclable blocks, which have also already been identified by the eager coarse sweep.

Without eager empty block identification, if the lospace runs out of blocks, firstly you don't know how many empty blocks the mark space has. Sweeping is a kind of wavefront that moves through the whole heap; empty blocks behind the wavefront will be identified, but those ahead of the wavefront will not. Such a lospace allocation would then have to either wait for a concurrent sweeper to advance, or perform some lazy sweeping work. The expected latency of a lospace allocation would thus be higher, without eager identification of empty blocks.

Secondly, eager sweeping might reduce allocation overhead for mutators. If allocation just has to identify holes and not compute information or decide on what to do with a block, maybe it go brr? Not sure.

**lines, lines, lines**

The original Immix paper also notes a relative insensitivity of the collector to line size: 64 or 256 bytes could have worked just as well. This was a somewhat surprising result to me but I think I didn't appreciate all the roles that lines play in Immix.

Obviously line size affect the worst-case fragmentation, though this is mitigated by evacuation (which evacuates objects, not lines). This I got from the paper. In this case, smaller lines are better.

Line size affects allocation-time overhead for mutators, though which way I don't know: scanning for holes will be easier with fewer lines in a block, but smaller lines would contain more free space and thus result in fewer collections. I can only imagine though that with smaller line sizes, average hole size would decrease and thus medium-sized allocations would be harder to service. Something of a wash, perhaps.

However if we ask ourselves the thought experiment, why not just have 16-byte lines? How crazy would that be? I think the impediment to having such a precise line size would mainly be Immix's eager sweep, as a fine-grained traversal of the heap would process much more data and incur possibly-unacceptable pause time overheads. But, in such a design you would do away with some other downsides of coarse-grained lines: a side table of mark bytes would make the line mark table redundant, and you eliminate much possible "dark matter" hidden by internal fragmentation in lines. You'd need to defer sweeping. But then you lose eager identification of empty blocks, and perhaps also the ability to evacuate into recyclable blocks. What would such a system look like?

Readers that have gotten this far will be pleased to hear that I have made some investigations in this area. But, this post is already long, so let's revisit this in another dispatch. Until then, happy allocations in all regions.

## related articles

- [unintentional concurrency](/archives/2022/07/20/unintentional-concurrency)
- [an optimistic evacuation of my wordhoard](/archives/2022/06/21/an-optimistic-evacuation-of-my-wordhoard)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)
- [whippet update: faster evacuation, eager sweeping of empty blocks](/archives/2024/08/25/whippet-update-faster-evacuation-eager-sweeping-of-empty-blocks)
- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)

Comments are closed.
