---
title: preliminary notes on a nofl field-logging barrier — wingolog
url: https://wingolog.org/archives/2024/10/03/preliminary-notes-on-a-nofl-field-logging-barrier
published: "2024-10-03T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/10/03/preliminary-notes-on-a-nofl-field-logging-barrier
---

# preliminary notes on a nofl field-logging barrier — wingolog

## [preliminary notes on a nofl field-logging barrier](/archives/2024/10/03/preliminary-notes-on-a-nofl-field-logging-barrier)

3 October 2024 8:54 AM

- [gc](/tags/gc)
- [whippet](/tags/whippet)
- [guile](/tags/guile)
- [garbage collection](/tags/garbage%20collection)
- [write barriers](/tags/write%20barriers)
- [nlnet](/tags/nlnet)
- [igalia](/tags/igalia)
- [field-logging](/tags/field-logging)
- [whiffle](/tags/whiffle)
- [lxr](/tags/lxr)

When you have a generational collector, you aim to trace only the part of the object graph that has been allocated recently. To do so, you need to keep a *remembered set*: a set of old-to-new edges, used as roots when performing a minor collection. A language run-time maintains this set by adding *write barriers*: little bits of collector code that run when a mutator writes to a field.

Whippet’s [nofl
space](https://wingolog.org/archives/2024/08/25/whippet-update-faster-evacuation-eager-sweeping-of-empty-blocks) is a block-structured space that is appropriate for use as an old generation or as part of a sticky-mark-bit generational collector. It used to have a card-marking write barrier; see my [article diving into V8’s new
write
barrier](https://wingolog.org/archives/2024/01/05/v8s-precise-field-logging-remembered-set), for more background.

Unfortunately, when running [whiffle](https://github.com/wingo/whiffle) benchmarks, I was seeing no improvement for generational configurations relative to whole-heap collection. Generational collection was doing fine in my tiny microbenchmarks that are part of Whippet itself, but when translated to larger programs (that aren’t yet proper macrobenchmarks), it was a lose.

I had planned on doing some serious tracing and instrumentation to figure out what was happening, and thereby correct the problem. I still plan on doing this, but instead for this issue I used the old noggin technique instead: just, you know, thinking about the thing, eventually concluding that unconditional card-marking barriers are inappropriate for sticky-mark-bit collectors. As I mentioned in the earlier article:

> An unconditional card-marking barrier applies to stores to slots in all objects, not just those in oldspace; a store to a new object will mark a card, but that card may contain old objects which would then be re-scanned. Or consider a store to an old object in a more dense part of oldspace; scanning the card may incur more work than needed. It could also be that Whippet is being too aggressive at re-using blocks for new allocations, where it should be limiting itself to blocks that are very sparsely populated with old objects.

That’s three problems. The second is well-known. But the first and last are specific to sticky-mark-bit collectors, where pages mix old and new objects.

### a precise field-logging write barrier

Back in 2019, Steve Blackburn’s paper [*Design and Analysis of*
*Field-Logging Write*
*Barriers*](https://www.steveblackburn.org/pubs/papers/fieldbarrier-ismm-2019.pdf) took a look at the state of the art in precise barriers that record not regions of memory that have been updated, but the precise edges (fields) that were written to. He ends up re-using this work later in [the 2022
LXR paper](https://www.steveblackburn.org/pubs/papers/lxr-pldi-2022.pdf) (see §3.4), where the write barrier is used for deferred reference counting and a snapshot-at-the-beginning (SATB) barrier for concurrent marking. All in all field-logging seems like an interesting strategy. Relative to card-marking, work during the pause is much less: you have a precise buffer of all fields that were written to, and you just iterate that, instead of iterating objects. Field-logging does impose some mutator cost, but perhaps the payoff is worth it.

To log each old-to-new edge precisely once, you need a bit per field indicating whether the field is logged already. Blackburn’s 2019 write barrier paper used bits in the object header, if the object was small enough, and otherwise bits before the object start. This requires some cooperation between the collector, the compiler, and the run-time that I wasn’t ready to pay for. The 2022 LXR paper was a bit vague on this topic, saying just that it used “a side table”.

In Whippet’s nofl space, we have a side table already, [used for a number
of purposes](https://github.com/wingo/whippet/blob/main/src/nofl-space.h#L187-L246):

1. Mark bits.

2. Iterability / interior pointers: is there an object at a given address? If so, it will have a recognizable bit pattern.

3. End of object, to be able to sweep without inspecting the object itself

4. Pinning, allowing a mutator to prevent an object from being evacuated, for example because a hash code was computed from its address

5. A hack to allow fully-conservative tracing to identify ephemerons at trace-time; this re-uses the pinning bit, since in practice such configurations never evacuate

6. Bump-pointer allocation into holes: the mark byte table serves the purpose of Immix’s line mark byte table, but at finer granularity. Because of this though, it is swept lazily rather than eagerly.

7. Generations. Young objects have a bit set that is cleared when they are promoted.

Well. Why not add another thing? The nofl space’s granule size is two words, so we can use two bits of the byte for field logging bits. If there is a write to a field, a barrier would first check that the object being written to is old, and then check the log bit for the field being written. The old check will be to a byte that is nearby or possibly the same as the one to check the field logging bit. If the bit is unsert, we call out to a slow path to actually record the field.

### preliminary results

I disassembled the fast path as compiled by GCC and got something like this on x86-64, in AT&T syntax, for the young-generation test:

```
mov    %rax,%rdx
and    $0xffffffffffc00000,%rdx
shr    $0x4,%rax
and    $0x3ffff,%eax
or     %rdx,%rax
testb  $0xe,(%rax)

```

The first five instructions compute the location of the mark byte, from the address of the object (which is known to be in the nofl space). If it has any of the bits in `0xe` set, then it’s in the old generation.

Then to test a field logging bit it’s a similar set of instructions. In one of my tests the data type looks like this:

```
struct Node {
  uintptr_t tag;
  struct Node *left;
  struct Node *right;
  int i, j;
};

```

Writing the `left` field will be in the same granule as the object itself, so we can just test the byte we fetched for the logging bit directly with `testb` against `$0x80`. For `right`, we should be able to know it’s in the same slab (aligned 4 MB region) and just add to the previously computed byte address, but the C compiler doesn’t know that right now and so recomputes. This would work better in a JIT. Anyway I think these bit-swizzling operations are just lost in the flow of memory accesses.

For the general case where you don’t statically know the offset of the field in the object, you have to compute which bit in the byte to test:

```
mov    %r13,%rcx
mov    $0x40,%eax
shr    $0x3,%rcx
and    $0x1,%ecx
shl    %cl,%eax
test   %al,%dil

```

Is it good? Well, it improves things for my whiffle benchmarks, relative to the card-marking barrier, seeing a 1.05×-1.5× speedup across a range of benchmarks. I suspect the main advantage is in avoiding the “unconditional” part of card marking, where a write to a new object could cause old objects to be added to the remembered set. There are still quite a few whiffle configurations in which the whole-heap collector outperforms the sticky-mark-bit generational collector, though; I hope to understand this a bit more by building a more classic semi-space nursery, and comparing performance to that.

Implementation links: [the barrier
fast-path](https://github.com/wingo/whippet/blob/main/api/gc-api.h#L231-L247), [the slow
path](https://github.com/wingo/whippet/blob/main/src/mmc.c#L910-L923), and the [sequential store
buffers](https://github.com/wingo/whippet/blob/main/src/field-set.h). (At some point I need to make it so that allocating edge buffers in the field set causes the nofl space to page out a corresponding amount of memory, so as to be honest when comparing GC performance at a fixed heap size.)

Until next time, onwards and upwards!

## related articles

- [baffled by generational garbage collection](/archives/2025/02/09/baffled-by-generational-garbage-collection)
- [whippet progress update: feature-complete!](/archives/2024/09/18/whippet-progress-update-feature-complete)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)

Comments are closed.
