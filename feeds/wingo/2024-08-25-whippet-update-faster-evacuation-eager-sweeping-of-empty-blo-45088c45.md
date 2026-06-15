---
title: 'whippet update: faster evacuation, eager sweeping of empty blocks — wingolog'
url: https://wingolog.org/archives/2024/08/25/whippet-update-faster-evacuation-eager-sweeping-of-empty-blocks
published: "2024-08-25T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/08/25/whippet-update-faster-evacuation-eager-sweeping-of-empty-blocks
---

# whippet update: faster evacuation, eager sweeping of empty blocks — wingolog

## [whippet update: faster evacuation, eager sweeping of empty blocks](/archives/2024/08/25/whippet-update-faster-evacuation-eager-sweeping-of-empty-blocks)

25 August 2024 8:29 PM

- [guile](/tags/guile)
- [garbage collection](/tags/garbage%20collection)
- [gc](/tags/gc)
- [whippet](/tags/whippet)
- [igalia](/tags/igalia)
- [block-structured heaps](/tags/block-structured%20heaps)
- [evacuation](/tags/evacuation)
- [parallelism](/tags/parallelism)
- [sweeping](/tags/sweeping)
- [atomics](/tags/atomics)

Good evening. Tonight, notes on things I have learned recently while hacking on the [Whippet GC
library](https://github.com/wingo/whippet).

### service update

For some time now, the name Whippet has referred to three things. Firstly, it is the [project as a
whole](https://github.com/wingo/whippet), consisting of an include-only garbage collection library containing a compile-time configurable choice of specific collector implementations. Also, it is the name of a [specific Immix-derived
collector](https://github.com/wingo/whippet/blob/main/doc/collector-whippet.md). Finally, it is the name of a specific *space* within that collector, in which objects are mostly marked in place but can be evacuated if appropriate.

Well, naming being one of the two hard problems of computer science, I can only ask for forgiveness and understanding. I have started fixing this situation with the third component, renaming the *whippet space* to the *nofl space*. Nofl stands for no-free-list, indicating that it’s a (mostly) mark space but which does bump-pointer allocation instead of using freelists. Also, it stands for novel, in the sense that as far as I can tell, it is a design that hasn’t been tried yet.

### unreliable evacuation

Following [Immix](https://dl.acm.org/doi/10.1145/1379022.1375586), the nofl space has always had *optimistic evacuation*. It prefers to mark objects in place, but if fragmentation gets too high, it will try to defragment by evacuating sparse blocks into a small set of empty blocks reserved for this purpose. If the evacuation reserve fills up, nofl will dynamically switch to marking in place.

My previous implementation was a bit daft: some subset of blocks would get marked as being evacuation targets, and evacuation would allocate into those blocks in ascending address order. Parallel GC threads would share a single global atomically-updated evacuation allocation pointer. As you can imagine, this was a serialization bottleneck; I initially thought it wouldn’t be so important but for some workloads it is.

I had chosen this strategy to maximize utilization of the evacuation reserve; if you had 8 GC workers, each allocating into their own block, their blocks won’t all be full at the end of GC; that would waste space.

But [reliability](https://wingolog.org/archives/2024/07/10/copying-collectors-with-block-structured-heaps-are-unreliable) turns out to be unimportant. It’s more important to let parallel GC threads do their thing without synchronization, to the extent possible.

Also, this serialized allocation discipline imposed some complexity on the nofl space implementation; the evacuation allocator was not like the “normal” allocator. With worker-local allocation buffers, these two allocators are now essentially the same. (They differ in that the normal allocator interleaves lazy sweeping with allocation, and can allocate into blocks with survivors from the last GC instead of requiring empty blocks.)

### eager sweeping

Another initial bad idea I had was to lean too much on lazy sweeping as a design principle. The idea was that deferring sweeping work until just before an allocator would write to a block would minimize cache overhead (you page in a block right when you will use it) and provide for workload-appropriate levels of parallelism (multiple mutator threads naturally parallelize sweeping).

Lazy sweeping was very annoying when it came to discovery of empty blocks. Empty blocks are precious: they can be returned to the OS if needed, they are useful for evacuation, and they have nice allocation properties, in that you can just do bump-pointer from beginning to end.

Nofl was discovering empty blocks just in time, from the allocator. If the allocator acquired a new block and found that it was empty, it would return it to a special list of empty blocks. Only if all sweepable pages were exhausted would an allocator use an empty block. But to prevent an allocator from pausing forever, there was a limit to the number of empty swept blocks that would be returned to the collector (10, as it happens); an 11th empty swept block would be given to a mutator for allocation. And so on and so on. Complicated, and you only know the number of empty blocks yielded by the last collection when the whole next allocation cycle has finished.

The obvious solution is some kind of separate mark on blocks, in addition to a mark on objects. I didn’t do it initially out of fear of overhead; marking is a fast path. The implementation I ended up making was a packed bitvector, with one bit per 64 kB block, at the beginning of each 4 MB slab of blocks. The beginning blocks are for metadata like this. For reasons, I don’t have the space for full bytes. When marking an object, if I see that a block’s mark is unset, I do an `atomic_fetch_or_explicit` on the byte with `memory_order_relaxed` ordering. In this way I only do the atomic op very rarely. It seems that [on ARMv8.1 there is actually an instruction to do atomic bit
setting](https://godbolt.org/z/a3E8KP7f6); everywhere else it’s a garbage compare-and-swap thing, but on my x64 machine it’s fine.

Then after collection, during the pause, if I see a block is unmarked, I move it directly to the empty set instead of sweeping it. (I could probably do this concurrently outside the pause, but that would be for another day.)

And the overhead? Negative! Negative, in the sense that because I don’t have to sweep empty blocks, and that (for some workloads) collection can produce a significant-enough fraction of empty blocks, I actually see speedups with this strategy, relative to lazy sweeping. It also simplifies the allocator (no need for that return-the-11th-block logic).

The only wrinkle is as regards generational collection: nofl currently uses the [sticky mark bit
algorithm](https://wingolog.org/archives/2022/10/22/the-sticky-mark-bit-algorithm), which has to be applied also to block marks. Subtle, but not complicated.

### fin

Next up is growing and shrinking the nofl-using Whippet collector (which might need another name), using the [membalancer
algorithm](https://wingolog.org/archives/2023/01/27/three-approaches-to-heap-sizing), and then I think I will be ready to move on to getting Whippet into Guile. Until then, happy hacking!

## related articles

- [an annoying failure mode of copying nurseries](/archives/2025/01/13/an-annoying-failure-mode-of-copying-nurseries)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [parallel ephemeron tracing](/archives/2023/01/24/parallel-ephemeron-tracing)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)
- [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)

### One response

1. jpfr says:[26 August 2024 3:38 AM](#0a07dfe4c8dc5f4c7950c9991aac3ed4343acf02)

   The Emacs project introduced a new garbage collector, MPS, a few months back. There is still some fallout and crashing bugs are discussed on the mailing list. They might be interested in whippet if it is shown to be smaller/faster/robust with less cognitive overhead for developers.

Comments are closed.
