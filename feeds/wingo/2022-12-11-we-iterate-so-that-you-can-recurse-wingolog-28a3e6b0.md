---
title: we iterate so that you can recurse — wingolog
url: https://wingolog.org/archives/2022/12/11/we-iterate-so-that-you-can-recurse
published: "2022-12-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2022/12/11/we-iterate-so-that-you-can-recurse
---

# we iterate so that you can recurse — wingolog

## [we iterate so that you can recurse](/archives/2022/12/11/we-iterate-so-that-you-can-recurse)

11 December 2022 9:19 PM

- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [semispace](/tags/semispace)
- [implementation](/tags/implementation)
- [cheney](/tags/cheney)
- [virtual machines](/tags/virtual%20machines)
- [guile](/tags/guile)
- [gnu](/tags/gnu)
- [igalia](/tags/igalia)
- [recursion](/tags/recursion)
- [stack overflow](/tags/stack%20overflow)

Sometimes when you see an elegant algorithm, you think “looks great, I just need it to also do X”. Perhaps you are able to build X directly out of what the algorithm gives you; fantastic. Or, perhaps you can alter the algorithm a bit, and it works just as well while also doing X. Sometimes, though, you alter the algorithm and things go pear-shaped.

Tonight’s little note builds on [yesterday’s semi-space collector
article](https://wingolog.org/archives/2022/12/10/a-simple-semi-space-collector) and discusses an worse alternative to the Cheney scanning algorithm.

To recall, we had this `visit_field` function that takes a edge in the object graph, as the address of a field in memory containing a `struct gc_obj*`. If the edge points to an object that was already copied, `visit_field` updates it to the forwarded address. Otherwise it copies the object, thus computing the new address, and then updates the field.

```
struct gc_obj* copy(struct gc_heap *heap, struct gc_obj *obj) {
  size_t size = heap_object_size(obj);
  struct gc_obj *new_obj = (struct gc_obj*)heap->hp;
  memcpy(new_obj, obj, size);
  forward(obj, new_obj);
  heap->hp += align_size(size);
  return new_obj;
}

void visit_field(struct gc_obj **field, struct gc_heap *heap) {
  struct gc_obj *from = *field;
  struct gc_obj *to =
    is_forwarded(from) ? forwarded(from) : copy(heap, from);
  *field = to;
}

```

Although a newly copied object is in tospace, all of its fields still point to fromspace. The Cheney scan algorithm later visits the fields in the newly copied object with `visit_field`, which both discovers new objects and updates the fields to point to tospace.

One disadvantage of this approach is that the order in which the objects are copied is a bit random. Given a hierarchical memory system, it’s better if objects that are accessed together in time are close together in space. This is an impossible task without instrumenting the actual data access in a program and then assuming future accesses will be like the past. Instead, the generally-accepted solution is to ensure that objects that are *allocated* close together in time be adjacent in space. The bump-pointer allocator in a semi-space collector provides this property, but the evacuation algorithm above does not: it would need to preserve allocation order, but instead its order is driven by graph connectivity.

I say that the copying algorithm above is random but really it favors a breadth-first traversal; if you have a binary tree, first you will copy the left and the right nodes of the root, then the left and right children of the left, then the left and right children of the right, then grandchildren, and so on. Maybe it would be better to keep parent and child nodes together? After all they are probably allocated that way.

So, what if we change the algorithm:

```
struct gc_obj* copy(struct gc_heap *heap, struct gc_obj *obj) {
  size_t size = heap_object_size(obj);
  struct gc_obj *new_obj = (struct gc_obj*)heap->hp;
  memcpy(new_obj, obj, size);
  forward(obj, new_obj);
  heap->hp += align_size(size);
  trace_heap_object(new_obj, heap, visit_field); // *
  return new_obj;
}

void visit_field(struct gc_obj **field, struct gc_heap *heap) {
  struct gc_obj *from = *field;
  struct gc_obj *to =
    is_forwarded(from) ? forwarded(from) : copy(heap, from);
  *field = to;
}

void collect(struct gc_heap *heap) {
  flip(heap);
  trace_roots(heap, visit_field);
}

```

Here we favor a depth-first traversal: we eagerly call `trace_heap_object` within `copy`. No need for the Cheney scan algorithm; tracing does it all.

The thing is, this works! It might even have better performance for some workloads, depending on access patterns. And yet, nobody does this. Why?

Well, consider a linked list with a million nodes; you’ll end up with a million recursive calls to `copy`, as visiting each link eagerly traverses the next. While I am [all about unbounded
recursion](https://wingolog.org/archives/2014/03/17/stack-overflow), an infinitely extensible stack is something that a language runtime has to provide to a user, and here we’re deep into implementing-the-language-runtime territory. At some point a user’s deep heap graph is going to cause a gnarly system failure via stack overflow.

Ultimately stack space needed by a GC algorithm counts towards collector memory overhead. In the case of a semi-space collector you already need twice the amount memory as your live object graph, and if you recursed instead of iterated this might balloon to 3x or more, depending on the heap graph shape.

Hey that’s my note! All this has been context for some future article, so this will be on the final exam. Until then!

## related articles

- [ephemeral success](/archives/2022/12/15/ephemeral-success)
- [i'm throwing ephemeron party & you're invited](/archives/2022/12/12/im-throwing-ephemeron-party-youre-invited)
- [a simple semi-space collector](/archives/2022/12/10/a-simple-semi-space-collector)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)

### One response

1. Taylor R Campbell says:[12 December 2022 1:33 PM](#1754c9275e1ff3c55fceb8d7429d783929bebe7a)

   Hold up – if you have the space for a queue to do a breadth-first traversal, surely you can find the space for a stack to do a depth-first traversal?

   The trick with the well-known algorithm of Cheney is to reuse the newspace as a contiguous queue of objects to be scanned.

   The trick with the little-known algorithm of Clark, used by T 3, is to reuse the oldspace as a discontiguous stack of objects to be scanned.

   When you copy an object from the oldspace to the newspace in Cheney’s algorithm, you reuse the oldspace storage for a forwarding pointer – but the rest of the storage for the object remains forever untouched!

   When you copy an object from the oldspace to the newspace in Clark’s algorithm, you reuse the oldspace storage for a forwarding pointer *and* a pointer to the object you were copying when you found it, as well as (for objects longer than pairs) a cursor into how much of the object you’ve copied so far.

   That way, once you’ve traced all the cdrs of a list (or down to a leaf of some other kind of tree), you can unwind the stack of objects you were copying to pick up each object where you left off, and do this iteratively until you’ve emptied the stack.

   Douglas W. Clark, “An Efficient List-Moving Algorithm Using Constant Workspace”, Communications of the ACM 19(6), June 1976. https://dl.acm.org/doi/pdf/10.1145/360238.360247

   (I’d leave a reference to the historical T source code too but it isn’t really amenable to URLs at the moment; perhaps that can be fixed at some point!)

Comments are closed.
