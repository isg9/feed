---
title: ephemeral success — wingolog
url: https://wingolog.org/archives/2022/12/15/ephemeral-success
published: "2022-12-15T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2022/12/15/ephemeral-success
---

# ephemeral success — wingolog

## [ephemeral success](/archives/2022/12/15/ephemeral-success)

15 December 2022 9:21 PM

- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [semispace](/tags/semispace)
- [implementation](/tags/implementation)
- [cheney](/tags/cheney)
- [virtual machines](/tags/virtual%20machines)
- [guile](/tags/guile)
- [gnu](/tags/gnu)
- [igalia](/tags/igalia)
- [ephemerons](/tags/ephemerons)
- [stack overflow](/tags/stack%20overflow)

Good evening, patient hackers :) Today finishes off my [series on
implementing ephemerons in a garbage
collector](https://wingolog.org/archives/2022/12/12/im-throwing-ephemeron-party-youre-invited).

Last time, we had a working solution for ephemerons, but it involved recursively visiting any pending ephemerons from within the `copy` routine—the bit of a semi-space collector that is called when traversing the object graph and we see an object that we hadn’t seen yet. This recursive visit could itself recurse, and so we could overflow the control stack.

The solution, of course, is “don’t do that”: instead of visiting recursively, enqueue the ephemeron for visiting later. Iterate, don’t recurse. But here we run into a funny problem: how do we add an ephemeron to a queue or worklist? It’s such a pedestrian question (“just... enqueue it?”) but I think it illustrates some of the particular concerns of garbage collection hacking.

#### speak, memory

The issue is that we are in the land of “can’t use my tools because I broke my tools with my tools”. You can’t make a standard `List` because you can’t allocate list nodes inside the tracing routine: if you had memory in which you could allocate, you wouldn’t be calling the garbage collector.

If the collector needs a data structure whose size doesn’t depend on the connectivity of the object graph, you can pre-allocate it in a reserved part of the heap. This adds memory overhead, of course; for a 1000 MB heap, say, you used to be able to make graphs 500 MB in size (for a semi-space collector), but now you can only do 475 MB because you have to reserve 50 MB (say) for your data structures. Another way to look at it is, if you have a 400 MB live set and then you allocate 2GB of garbage, if your heap limit is 500 MB you will collect 20 times, but if it’s 475 MB you’ll collect 26 times, which is more expensive. This is part of why GC algorithms are so primitive; implementors have to be stingy that we don’t get to have nice things / data structures.

However in the case of ephemerons, we will potentially need one worklist entry per ephemeron in the object graph. There is no optimal fixed size for a worklist of ephemerons. Most object graphs will have no or few ephemerons. Some, though, will have practically the whole heap.

For data structure needs like this, the standard solution is to reserve the needed space for a GC-managed data structure in the object itself. For example, for concurrent copying collectors, the GC might reserve a word in the object for a forwarding pointer, instead of just clobbering the first word. If you needed a GC-managed binary tree for a specific kind of object, you’d reserve two words. Again there are strong pressures to minimize this overhead, but in the case of ephemerons it seems sensible to make them pay their way on a per-ephemeron basis.

#### so let’s retake the thing

So sometimes we might need to put an ephemeron in a worklist. Let’s add a member to the ephemeron structure:

```
struct gc_ephemeron {
  struct gc_obj header;
  int dead;
  struct gc_obj *key;
  struct gc_obj *value;
  struct gc_ephemeron *gc_link; // *
};

```

Incidentally this also solves the problem of how to represent the `struct gc_pending_ephemeron_table`; just reserve 0.5% of the heap or so as a bucket array for a buckets-and-chains hash table, and use the `gc_link` as the intrachain links.

```
struct gc_pending_ephemeron_table {
  struct gc_ephemeron *resolved;
  size_t nbuckets;
  struct gc_ephemeron buckets[0];
};

```

An ephemeron can end up in three states, then:

1. Outside a collection: `gc_link` can be whatever.

2. In a collection, the ephemeron is in the pending ephemeron table: `gc_link` is part of a hash table.

3. In a collection, the ephemeron’s key has been visited, and the ephemeron is on the to-visit worklist; `gc_link` is part of the `resolved` singly-linked list.

Instead of phrasing the interface to ephemerons in terms of visiting edges in the graph, the verb is to *resolve* ephemerons. Resolving an ephemeron adds it to a worklist instead of immediately visiting any edge.

```
struct gc_ephemeron **
pending_ephemeron_bucket(struct gc_pending_ephemeron_table *table,
                         struct gc_obj *key) {
  return &table->buckets[hash_pointer(obj) % table->nbuckets];
}

void add_pending_ephemeron(struct gc_pending_ephemeron_table *table,
                           struct gc_obj *key,
                           struct gc_ephemeron *ephemeron) {
  struct gc_ephemeron **bucket = pending_ephemeron_bucket(table, key);
  ephemeron->gc_link = *bucket;
  *bucket = ephemeron;
}

void resolve_pending_ephemerons(struct gc_pending_ephemeron_table *table,
                                struct gc_obj *obj) {
  struct gc_ephemeron **link = pending_ephemeron_bucket(table, obj);
  struct gc_ephemeron *ephemeron;
  while ((ephemeron = *link)) {
    if (ephemeron->key == obj) {
      *link = ephemeron->gc_link;
      add_resolved_ephemeron(table, ephemeron);
    } else {
      link = &ephemeron->gc_link;
    }
  }
}

```

Copying an object may add it to the set of pending ephemerons, if it is itself an ephemeron, and also may resolve other pending ephemerons.

```
void resolve_ephemerons(struct gc_heap *heap, struct gc_obj *obj) {
  resolve_pending_ephemerons(heap->pending_ephemerons, obj);

  struct gc_ephemeron *ephemeron;
  if ((ephemeron = as_ephemeron(forwarded(obj)))
      && !ephemeron->dead) {
    if (is_forwarded(ephemeron->key))
      add_resolved_ephemeron(heap->pending_ephemerons,
                             ephemeron);
    else
      add_pending_ephemeron(heap->pending_ephemerons,
                            ephemeron->key, ephemeron);
  }
}

struct gc_obj* copy(struct gc_heap *heap, struct gc_obj *obj) {
  ...
  resolve_ephemerons(heap, obj); // *
  return new_obj;
}

```

Finally, we need to add something to the core collector to scan resolved ephemerons:

```
int trace_some_ephemerons(struct gc_heap *heap) {
  struct gc_ephemeron *resolved = heap->pending_ephemerons->resolved;
  if (!resolved) return 0;
  heap->pending_ephemerons->resolved = NULL;
  while (resolved) {
    resolved->key = forwarded(resolved->key);
    visit_field(&resolved->value, heap);
    resolved = resolved->gc_link;
  }
  return 1;
}

void kill_pending_ephemerons(struct gc_heap *heap) {
  struct gc_ephemeron *ephemeron;
  struct gc_pending_ephemeron_table *table = heap->pending_ephemerons;
  for (size_t i = 0; i < table->nbuckets; i++) {
    for (struct gc_ephemeron *chain = table->buckets[i];
         chain;
         chain = chain->gc_link)
      chain->dead = 1;
    table->buckets[i] = NULL;
  }
}

void collect(struct gc_heap *heap) {
  flip(heap);
  uintptr_t scan = heap->hp;
  trace_roots(heap, visit_field);
  do { // *
    while(scan < heap->hp) {
      struct gc_obj *obj = scan;
      scan += align_size(trace_heap_object(obj, heap, visit_field));
    }
  } while (trace_ephemerons(heap)); // *
  kill_pending_ephemerons(heap); // *
}

```

The result is... not so bad? It makes sense to make ephemerons pay their own way in terms of memory, having an internal field managed by the GC. In fact I must confess that in the implementation I have been woodshedding, I actually have three of these damn things; perhaps more on that in some other post. But the perturbation to the core algorithm is perhaps less than the original code. There are still some optimizations to make, notably postponing hash-table lookups until the whole strongly-reachable graph is discovered; but again, another day.

And with that, thanks for coming along with me for my journeys into ephemeron-space.

I would like to specifically thank Erik Corry and Steve Blackburn for their advice over the years, and patience with my ignorance; I can only imagine that it’s quite amusing when you have experience in a domain to see someone new and eager come in and make many of the classic mistakes. They have both had a kind of generous parsimony in the sense of allowing me to make the necessary gaffes but also providing insight where it can be helpful.

I’m thinking of many occasions but I especially appreciate the advice to start with a semi-space collector when trying new things, be it benchmarks or test cases or API design or new functionality, as it’s a simple algorithm, hard to get wrong on the implementation side, and perfect for bringing out any bugs in other parts of the system. In this case the difference between fromspace and tospace pointers has a material difference to how you structure the ephemeron implementation; it’s not something you can do just in a `trace_heap_object` function, as you don’t have the old pointers there, and the pending ephemeron table is indexed by old object addresses.

Well, until some other time, gentle hackfolk, do accept my sincerest waste disposal greetings. As always, yours in garbage, etc.,

## related articles

- [i'm throwing ephemeron party & you're invited](/archives/2022/12/12/im-throwing-ephemeron-party-youre-invited)
- [we iterate so that you can recurse](/archives/2022/12/11/we-iterate-so-that-you-can-recurse)
- [a simple semi-space collector](/archives/2022/12/10/a-simple-semi-space-collector)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)
- [finalizers, guardians, phantom references, et cetera](/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera)

### One response

1. Taylor R Campbell says:[17 December 2022 8:16 PM](#f2bd5d4a04184ff75655200611620ff00a69e647)

   MIT Scheme uses a similar technique, sketched in pseudo-Scheme at https://mumble.net/~campbell/scheme/ephemeron-hash.scm and implemented for real in https://git.savannah.gnu.org/cgit/mit-scheme.git/tree/src/microcode/gcloop.c?id=37fb28dbe3722c4e6560ace2deaaac699fcd53ca and https://git.savannah.gnu.org/cgit/mit-scheme.git/tree/src/microcode/memmag.c?id=37fb28dbe3722c4e6560ace2deaaac699fcd53ca.

   There are some minor differences. It is split into three stages rather than two:

   (1) Scan from the GC roots as usual (including recursive processing of Cheney queue), building the hash table along the way but not queueing any ephemerons to be resolved yet.

   (2) For each live ephemeron, if its key has been proven live, remove it from the hash table and queue it to be resolved.

   (3) For each queued ephemeron, resolve it and scan its datum, queueing more to be resolved as they are proven intact.

   Whether separating stages (1) and (2) is an optimization, like I think you’re suggesting, isn’t clear to me; if memory serves, I wrote it this way to make it easy to reason about more than to make it fast.

   There’s one other thing: The garbage collector counts the number n of live ephemerons, so that after it has finished a garbage collection, it can allocate a new array of O(n) hash table buckets in the newspace for the *next* garbage collection as soon as it’s done with the last one. This allocation is guaranteed to succeed, because the old array has been freed in the last garbage collection!

   The primitives to create a batch of b ephemerons (normally, for make-ephemeron, this is b=1, but for fasload you might have b>1 if ephemerons got stored in an image) will reallocate the array, roughly doubling it, if there isn’t enough free space in it for n + b ephemerons.

   If there isn’t enough free space in the heap, the primitives will record b before aborting and triggering a GC. The GC will then actually try to allocate an array of O(n + b) buckets, but if it can’t, it will fall back to allocating an array of O(n) buckets and trigger remediation such as GC daemons to clear space or report a heap-full error.

   This way:

   (a) The array is always large enough for hash table operations used by the GC to take expected O(1) time (or would be, if the hash function were uniform random; right now, it’s just addr % nbuckets).

   (b) make-ephemeron takes amortized O(1) time (and fasload takes amortized O(b) time for b ephemerons in a fasl file).

   (c) make-ephemeron (and fasload) never gets into a GC loop when heap space runs out.

   By the way, in contrast to your musings in the penultimate paragraph, MIT Scheme actually *does not* distinguish between oldspace and newspace pointers in its semispace stop & copy GC. Instead there is a separate newspace and tospace, but nothing points into the tospace. The tospace is just a temporary region where the newspace is built up and then memcpied to the real heap, where oldspace *and* newspace pointers both point into – so you can’t tell oldspace or newspace pointers apart. But the algorithm doesn’t need to tell them apart.

   (This newspace/tospace distinction is for...hysterical raisins going back to the 24-bit address space on Motorola 68k CPUs in the 1980s like the space shuttle dimensions determined by the width of a Roman horse’s rear.)

   P.S. I had to fake out the marxdown parser to prevent it from generating li elements, which the tetuki filter prohibits. Maybe ul/ol/li should be allowed in filters.scm?

Comments are closed.
