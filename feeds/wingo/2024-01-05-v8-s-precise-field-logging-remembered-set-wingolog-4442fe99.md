---
title: v8's precise field-logging remembered set — wingolog
url: https://wingolog.org/archives/2024/01/05/v8s-precise-field-logging-remembered-set
published: "2024-01-05T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/01/05/v8s-precise-field-logging-remembered-set
---

# v8's precise field-logging remembered set — wingolog

## [v8's precise field-logging remembered set](/archives/2024/01/05/v8s-precise-field-logging-remembered-set)

5 January 2024 9:44 AM

- [v8](/tags/v8)
- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [remembered sets](/tags/remembered%20sets)
- [whippet](/tags/whippet)
- [igalia](/tags/igalia)
- [generational](/tags/generational)
- [compilers](/tags/compilers)

A *remembered set* is used by a garbage collector to identify graph edges between partitioned sub-spaces of a heap. The canonical example is in generational collection, where you allocate new objects in *newspace*, and eventually promote survivor objects to *oldspace*. If most objects die young, we can focus GC effort on newspace, to avoid traversing all of oldspace all the time.

Collecting a subspace instead of the whole heap is sound if and only if we can identify all live objects in the subspace. We start with some set of *roots* that point into the subspace from outside, and then traverse all links in those objects, but only to other objects within the subspace.

The roots are, like, global variables, and the stack, and registers; and in the case of a partial collection in which we identify live objects only within newspace, also any link into newspace from other spaces (oldspace, in our case). This set of inbound links is a *remembered* *set*.

There are a few strategies for maintaining a remembered set. Generally speaking, you start by implementing a write barrier that intercepts all stores in a program. Instead of:

```
obj[slot] := val;

```

You might abstract this away:

```
write_slot(obj, sizeof obj, &obj[slot], val);

```

As you can see, it’s quite an annoying transformation to do by hand; typically you will want some sort of language-level abstraction that lets you keep the more natural syntax. C++ can do this pretty well, or if you are implementing a compiler, you just add this logic to the code generator.

Then the actual write barrier... well its implementation is twingled up with implementation of the remembered set. The simplest variant is a *card-marking* scheme, whereby the heap is divided into equal-sized power-of-two-sized *cards*, and each card has a bit. If the heap is also divided into blocks (say, 2 MB in size), then you might divide those blocks into 256-byte cards, yielding 8192 cards per block. A barrier might look like this:

```
void write_slot(ObjRef obj, size_t size,
                SlotAddr slot, ObjRef val) {
  obj[slot] := val; // Start with the store.

  uintptr_t block_size = 1<<21;
  uintptr_t card_size = 1<<8;
  uintptr_t cards_per_block = block_size / card_size;

  uintptr_t obj_addr = obj;
  uintptr_t card_idx = (obj_addr / card_size) % cards_per_block;

  // Assume remset allocated at block start.
  void *block_start = obj_addr & ~(block_size-1);
  uint32_t *cards = block_start;

  // Set the bit.
  cards[card_idx / 32] |= 1 << (card_idx % 32);
}

```

Then when marking the new generation, you visit all cards, and for all marked cards, trace all outbound links in all live objects that begin on the card.

Card-marking is simple to implement and simple to statically allocate as part of the heap. Finding marked cards takes time proportional to the size of the heap, but you hope that the constant factors and SIMD minimize this cost. However iterating over objects within a card can be costly. You hope that there are few old-to-new links but what do you know?

In [Whippet](https://github.com/wingo/whippet) I have been struggling a bit with [sticky-mark-bit generational
marking](https://wingolog.org/archives/2022/10/22/the-sticky-mark-bit-algorithm), in which new and old objects are not spatially partitioned. Sometimes generational collection is a win, but in benchmarking I find that often it isn’t, and I think [Whippet’s card-marking
barrier](https://github.com/wingo/whippet/blob/main/api/gc-api.h#L187) is at fault: it is simply too imprecise. Consider firstly that our write barrier applies to stores to slots in all objects, not just those in oldspace; a store to a new object will mark a card, but that card may contain old objects which would then be re-scanned. Or consider a store to an old object in a more dense part of oldspace; scanning the card may incur more work than needed. It could also be that Whippet is being too aggressive at re-using blocks for new allocations, where it should be limiting itself to blocks that are very sparsely populated with old objects.

### what v8 does

There is a tradeoff in write barriers between the overhead imposed on stores, the size of the remembered set, and the precision of the remembered set. Card-marking is relatively low-overhead and usually small as a fraction of the heap, but not very precise. It would be better if a remembered set recorded objects, not cards. And it would be even better if it recorded slots in objects, not just objects.

V8 takes this latter strategy: it has per-block remembered sets which record slots containing “interesting” links. All of the above words were to get here, to take a brief look at its remembered set.

The main operation is [`RememberedSet::Insert`](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/main/src/heap/remembered-set.h#92). It takes the `MemoryChunk` (a block, in our language from above) and the address of a slot in the block. Each block has a remembered set; in fact, [six remembered
sets](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/main/src/heap/memory-chunk-layout.h#25) for some reason. The remembered set itself is a [`SlotSet`](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/main/src/heap/slot-set.h), whose interesting operations come from [`BasicSlotSet`](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/main/src/heap/base/basic-slot-set.h).

The structure of a slot set is a [bitvector partitioned into
equal-sized, possibly-empty
buckets](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/main/src/heap/base/basic-slot-set.h#44). There is one bit per slot in the block, so in the limit the size overhead for the remembered set may be 3% (1/32, assuming compressed pointers). Currently [each bucket is 1024 bits (128
bytes)](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/main/src/heap/base/basic-slot-set.h#265), plus the 4 bytes for the bucket pointer itself.

[Inserting into the slot
set](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/main/src/heap/base/basic-slot-set.h#109) will first allocate a bucket (using C++ `new`) if needed, then [load the
“cell” (32-bit
integer)](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/main/src/heap/base/basic-slot-set.h#130) containing the slot. There is a template parameter declaring whether this is an atomic or normal load. Finally, if the slot bit in the cell is not yet set, V8 will set the bit, possibly using atomic compare-and-swap.

In the language of Blackburn’s [*Design and analysis of field-logging*
*write*
*barriers*](https://users.cecs.anu.edu.au/~steveb/pubs/papers/fieldbarrier-ismm-2019.pdf), I believe this is a field-logging barrier, rather than the bit-stealing slot barrier described by Yang et al in the 2012 [*Barriers*
*Reconsidered, Friendlier*
*Still!*](https://users.cecs.anu.edu.au/~steveb/pubs/papers/barrier-ismm-2012.pdf). Unlike Blackburn’s field-logging barrier, however, this remembered set is implemented completely on the side: there is no in-object remembered bit, nor remembered bits for the fields.

On the one hand, V8’s remembered sets are precise. There are some tradeoffs, though: they require off-managed-heap dynamic allocation for the buckets, and [traversing the remembered
sets](https://chromium.googlesource.com/v8/v8.git/+/refs/heads/main/src/heap/minor-mark-sweep.cc#162) takes time proportional to the whole heap size. And, should V8 ever switch its [minor mark-sweep generational
collector](https://wingolog.org/archives/2023/12/08/v8s-mark-sweep-nursery) to use sticky mark bits, the lack of a spatial partition could lead to similar problems as I am seeing in Whippet. I will be interested to see what they come up with in this regard.

Well, that’s all for today. Happy hacking in the new year!

## related articles

- [ahead-of-time wasm gc in wastrel](/archives/2026/02/06/ahead-of-time-wasm-gc-in-wastrel)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)
- [baffled by generational garbage collection](/archives/2025/02/09/baffled-by-generational-garbage-collection)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [six thoughts on generating c](/archives/2026/02/09/six-thoughts-on-generating-c)
- [the last couple years in v8's garbage collector](/archives/2025/11/13/the-last-couple-years-in-v8s-garbage-collector)

Comments are closed.
