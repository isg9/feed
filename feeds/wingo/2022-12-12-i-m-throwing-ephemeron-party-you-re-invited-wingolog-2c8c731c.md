---
title: i'm throwing ephemeron party & you're invited — wingolog
url: https://wingolog.org/archives/2022/12/12/im-throwing-ephemeron-party-youre-invited
published: "2022-12-12T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2022/12/12/im-throwing-ephemeron-party-youre-invited
---

# i'm throwing ephemeron party & you're invited — wingolog

## [i'm throwing ephemeron party & you're invited](/archives/2022/12/12/im-throwing-ephemeron-party-youre-invited)

12 December 2022 2:02 PM

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

Good day, hackfolk. Today’s note tries to extend our semi-space collector with support for ephemerons. Spoiler alert: we fail in a subtle and interesting way. See if you can spot it before the end :)

Recall that, as we concluded in [an earlier
article](https://wingolog.org/archives/2022/11/28/are-ephemerons-primitive), a memory manager needs to incorporate ephemerons as a core part of the tracing algorithm. Ephemerons are not macro-expressible in terms of object trace functions.

Instead, to support ephemerons we need to augment our core trace routine. When we see an ephemeron *E*, we need to check if the key *K* is already visited (and therefore live); if so, we trace the value *V* directly, and we’re done. Otherwise, we add *E* to a global table of pending ephemerons *T*, indexed under *K*. Finally whenever we trace a new object *O*, ephemerons included, we look up *O* in *T*, to trace any pending ephemerons for *O*.

So, taking our [semi-space
collector](https://wingolog.org/archives/2022/12/10/a-simple-semi-space-collector) as a workbench, let’s start by defining what an ephemeron is.

```
struct gc_ephemeron {
  struct gc_obj header;
  int dead;
  struct gc_obj *key;
  struct gc_obj *value;
};

enum gc_obj_kind { ..., EPHEMERON, ... };

static struct gc_ephemeron* as_ephemeron(struct gc_obj *obj) {
  uintptr_t ephemeron_tag = NOT_FORWARDED_BIT | (EPHEMERON << 1);
  if (obj->tag == ephemeron_tag)
    return (struct gc_ephemeron*)obj;
  return NULL;
}

```

First we need to allow the GC to know when an object is an ephemeron or not. This is somewhat annoying, as you would like to make this concern entirely the responsibility of the user, and let the GC be indifferent to the kinds of objects it’s dealing with, but it seems to be unavoidable.

The heap will need some kind of data structure to track pending ephemerons:

```
struct gc_pending_ephemeron_table;

struct gc_heap {
  ...
  struct gc_pending_ephemeron_table *pending_ephemerons;
}

struct gc_ephemeron *
pop_pending_ephemeron(struct gc_pending_ephemeron_table*,
                      struct gc_obj*);
void
add_pending_ephemeron(struct gc_pending_ephemeron_table*,
                      struct gc_obj*, struct gc_ephemeron*);
struct gc_ephemeron *
pop_any_pending_ephemeron(struct gc_pending_ephemeron_table*);

```

Now let’s define a function to handle ephemeron shenanigans:

```
void visit_ephemerons(struct gc_heap *heap, struct gc_obj *obj) {
  // We are visiting OBJ for the first time.
  // OBJ is the old address, but it is already forwarded.
  ASSERT(is_forwarded(obj));

  // First, visit any pending ephemeron for OBJ.
  struct gc_ephemeron *ephemeron;
  while ((ephemeron =
            pop_pending_ephemeron(heap->pending_ephemerons, obj))) {
    ASSERT(obj == ephemeron->key);
    ephemeron->key = forwarded(obj);
    visit_field(&ephemeron->value, heap);
  }

  // Then if OBJ is itself an ephemeron, trace it.
  if ((ephemeron = as_ephemeron(forwarded(obj)))
      && !ephemeron->dead) {
    if (is_forwarded(ephemeron->key)) {
      ephemeron->key = forwarded(ephemeron->key);
      visit_field(&ephemeron->value, heap);
    } else {
      add_pending_ephemeron(heap->pending_ephemerons,
                            ephemeron->key, ephemeron);
    }
  }
}

struct gc_obj* copy(struct gc_heap *heap, struct gc_obj *obj) {
  ...
  visit_ephemerons(heap, obj); // *
  return new_obj;
}

```

We wire it into the `copy` routine, as that’s the bit of the collector that is called only once per object and which has access to the old address. We actually can’t process ephemerons during the Cheney field scan, as there we don’t have old object addresses.

Then at the end of collection, we kill any ephemeron whose key hasn’t been traced:

```
void kill_pending_ephemerons(struct gc_heap *heap) {
  struct gc_ephemeron *ephemeron;
  while ((ephemeron =
            pop_any_pending_ephemeron(heap->pending_ephemerons)))
    ephemeron->dead = 1;
}

void collect(struct gc_heap *heap) {
  // ...
  kill_pending_ephemerons(heap);
}

```

First observation: Gosh, this is quite a mess. It’s more code than the core collector, and it’s gnarly. There’s a hash table, for goodness’ sake. Goodbye, elegant algorithm!

Second observation: Well, at least it works.

Third observation: Oh. It works in the same way as [tracing in the
`copy`
routine](https://wingolog.org/archives/2022/12/11/we-iterate-so-that-you-can-recurse) works: well enough for shallow graphs, but catastrophically for arbitrary graphs. Calling `visit_field` from within `copy` introduces unbounded recursion, as tracing one value can cause more ephemerons to resolve, ad infinitum.

Well. We seem to have reached a dead-end, for now. Will our hero wrest victory from the jaws of defeat? Tune in next time for find out: same garbage time (unpredictable), same garbage channel (my wordhoard). Happy hacking!

## related articles

- [ephemeral success](/archives/2022/12/15/ephemeral-success)
- [we iterate so that you can recurse](/archives/2022/12/11/we-iterate-so-that-you-can-recurse)
- [a simple semi-space collector](/archives/2022/12/10/a-simple-semi-space-collector)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)
- [finalizers, guardians, phantom references, et cetera](/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera)

Comments are closed.
