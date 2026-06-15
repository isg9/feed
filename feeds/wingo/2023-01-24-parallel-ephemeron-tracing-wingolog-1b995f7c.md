---
title: parallel ephemeron tracing — wingolog
url: https://wingolog.org/archives/2023/01/24/parallel-ephemeron-tracing
published: "2023-01-24T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/01/24/parallel-ephemeron-tracing
---

# parallel ephemeron tracing — wingolog

## [parallel ephemeron tracing](/archives/2023/01/24/parallel-ephemeron-tracing)

24 January 2023 10:48 AM

- [garbage collection](/tags/garbage%20collection)
- [gc](/tags/gc)
- [ephemerons](/tags/ephemerons)
- [tracing](/tags/tracing)
- [evacuation](/tags/evacuation)
- [whippet](/tags/whippet)
- [guile](/tags/guile)
- [igalia](/tags/igalia)
- [parallelism](/tags/parallelism)

Hello all, and happy new year. Today’s note continues the [series on
implementing ephemerons in a garbage
collector](https://wingolog.org/tags/ephemerons).

In our [last
dispatch](https://wingolog.org/archives/2022/12/15/ephemeral-success) we looked at a serial algorithm to trace ephemerons. However, production garbage collectors are parallel: during collection, they trace the object graph using multiple worker threads. Our problem is to extend the ephemeron-tracing algorithm with support for multiple tracing threads, without introducing stalls or serial bottlenecks.

Recall that we ended up having to define a table of pending ephemerons:

```
struct gc_pending_ephemeron_table {
  struct gc_ephemeron *resolved;
  size_t nbuckets;
  struct gc_ephemeron *buckets[0];
};

```

This table holds *pending* ephemerons that have been visited by the graph tracer but whose keys haven’t been found yet, as well as a singly-linked list of *resolved* ephemerons that are waiting to have their values traced. As a global data structure, the pending ephemeron table is a point of contention between tracing threads that we need to design around.

### a confession

Allow me to confess my sins: things would be a bit simpler if I didn’t allow tracing workers to race.

As background, if your GC supports marking in place instead of always evacuating, then there is a mark bit associated with each object. To reduce the overhead of contention, a common strategy is to actually use a whole byte for the mark bit, and to write to it using relaxed atomics (or even raw stores). This avoids the cost of a compare-and-swap, but at the cost that multiple marking threads might see that an object’s mark was unset, go to mark the object, and think that they were the thread that marked the object. As far as the mark byte goes, that’s OK because everybody is writing the same value. The object gets pushed on the to-be-traced grey object queues multiple times, but that’s OK too because tracing should be idempotent.

This is a common optimization for parallel marking, and it doesn’t have any significant impact on other parts of the GC–except ephemeron marking. For ephemerons, because the state transition isn’t simply from unmarked to marked, we need more coordination.

### high level

The parallel ephemeron marking algorithm modifies the serial algorithm in just a few ways:

1. We have an atomically-updated *state* field in the ephemeron, used to know if e.g. an ephemeron is pending or resolved;

2. We use separate fields for the *pending* and *resolved* links, to allow for concurrent readers across a state change;

3. We introduce “traced” and “claimed” states to resolve races between parallel tracers on the same ephemeron, and track the “epoch” at which an ephemeron was last traced;

4. We remove resolved ephemerons from the pending ephemeron hash table lazily, and use atomic swaps to pop from the resolved ephemerons list;

5. We have to re-check key liveness after publishing an ephemeron to the pending ephemeron table.

Regarding the first point, there are four possible values for the ephemeron’s state field:

```
enum {
  TRACED, CLAIMED, PENDING, RESOLVED
};

```

The state transition diagram looks like this:

```
  ,----->TRACED<-----.
 ,         | ^        .
,          v |         .
|        CLAIMED        |
|  ,-----/     \---.    |
|  v               v    |
PENDING--------->RESOLVED

```

With this information, we can start to flesh out the ephemeron object itself:

```
struct gc_ephemeron {
  uint8_t state;
  uint8_t is_dead;
  unsigned epoch;
  struct gc_ephemeron *pending;
  struct gc_ephemeron *resolved;
  void *key;
  void *value;
};

```

The *state* field holds one of the four state values; *is\_dead* indicates if a live ephemeron was ever proven to have a dead key, or if the user explicitly killed the ephemeron; and *epoch* is the GC count at which the ephemeron was last traced. Ephemerons are born `TRACED` in the current GC epoch, and the collector is responsible for incrementing the current epoch before each collection.

### algorithm: tracing ephemerons

When the collector first finds an ephemeron, it does a compare-and-swap (CAS) on the state from `TRACED` to `CLAIMED`. If that succeeds, we check the epoch; if it’s current, we revert to the `TRACED` state: there’s nothing to do.

(Without marking races, you wouldn’t need either `TRACED` or `CLAIMED` states, or the epoch; it would be implicit in the fact that the ephemeron was being traced at all that you had a `TRACED` ephemeron with an old epoch.)

So now we have a `CLAIMED` ephemeron with an out-of-date epoch. We update the epoch and clear the *pending* and *resolved* fields, setting them to NULL. If, then, the ephemeron *is\_dead*, we are done, and we go back to `TRACED`.

Otherwise we check if the *key* has already been traced. If so we forward it (if evacuating) and then trace the value edge as well, and transition to `TRACED`.

Otherwise we have a live *E* but we don’t know about *K*; this ephemeron is pending. We transition *E*‘s state to `PENDING` and add it to the front of *K*’s hash bucket in the pending ephemerons table, using CAS to avoid locks.

We then have to re-check if *K* is live, after publishing *E*, to account for other threads racing to mark to *K* while we mark *E*; if indeed *K* is live, then we transition to `RESOLVED` and push *E* on the global resolved ephemeron list, using CAS, via the *resolved* link.

So far, so good: either the ephemeron is fully traced, or it’s pending and published, or (rarely) published-then-resolved and waiting to be traced.

### algorithm: tracing objects

The annoying thing about tracing ephemerons is that it potentially impacts tracing of all objects: any object could be the key that resolves a pending ephemeron.

When we trace an object, we look it up in the pending ephemeron hash table. But, as we traverse the chains in a bucket, we also load each node’s state. If we find a node that’s not in the `PENDING` state, we atomically forward its predecessor to point to its successor. This is correct for concurrent readers because the end of the chain is always reachable: we only skip nodes that are not `PENDING`, nodes never become `PENDING` after they transition away from being `PENDING`, and we only add `PENDING` nodes to the front of the chain. We even leave the *pending* field in place, so that any concurrent reader of the chain can still find the tail, even when the ephemeron has gone on to be `RESOLVED` or even `TRACED`.

(I had thought I would need [Tim Harris’ atomic list
implementation](https://timharris.uk/papers/2001-disc.pdf), but it turns out that since I only ever insert items at the head, having annotated links is not necessary.)

If we find a `PENDING` ephemeron that has *K* as its key, then we CAS its state from `PENDING` to `RESOLVED`. If this works, we CAS it onto the front of the resolved list. (Note that we also have to forward the key at this point, for a moving GC; this was a bug in my original implementation.)

### algorithm: resolved ephemerons

Periodically a thread tracing the graph will run out of objects to trace (its mark stack is empty). That’s a good time to check if there are resolved ephemerons to trace. We atomically exchange the global resolved list with `NULL`, and then if there were resolved ephemerons, then we trace their values and transition them to `TRACED`.

At the very end of the GC cycle, we sweep the pending ephemeron table, marking any ephemeron that’s still there as *is\_dead*, transitioning them back to `TRACED`, clearing the buckets of the pending ephemeron table as we go.

### nits

So that’s it. There are some drawbacks, for example that this solution takes at least three words per ephemeron. Oh well.

There is also an annoying point of serialization, which is related to the lazy ephemeron resolution optimization. Consider that checking the pending ephemeron table on every object visit is overhead; it would be nice to avoid this. So instead, we start in “lazy” mode, in which pending ephemerons are never resolved by marking; and then once the mark stack / grey object worklist fully empties, we sweep through the pending ephemeron table, checking each ephemeron’s key to see if it was visited in the end, and resolving those ephemerons; we then switch to “eager” mode in which each object visit could potentially resolve ephemerons. In this way the cost of ephemeron tracing is avoided for that part of the graph that is strongly reachable. However, with parallel markers, would you switch to eager mode when any thread runs out of objects to mark, or when all threads run out of objects? You would get greatest parallelism with the former, but you run the risk of some workers prematurely running out of data, but when there is still a significant part of the strongly-reachable graph to traverse. If you wait for all threads to be done, you introduce a serialization point. There is a related question of when to pump the resolved ephemerons list. But these are engineering details.

Speaking of details, there are some gnarly pitfalls, particularly that you have to be very careful about pre-visit versus post-visit object addresses; for a semi-space collector, visiting an object will move it, so for example in the pending ephemeron table which by definition is keyed by pre-visit (fromspace) object addresses, you need to be sure to trace the ephemeron key for any transition to `RESOLVED`, and there are a few places this happens (the re-check after publish, sweeping the table after transitioning from lazy to eager, and when resolving eagerly).

### implementation

If you’ve read this far, you may be interested in the [implementation](https://github.com/wingo/whippet-gc/blob/main/gc-ephemeron.c); it’s only a few hundred lines long. It took me quite a while to whittle it down!

Ephemerons are challenging from a software engineering perspective, because they are logically a separate module, but they interact both with users of the GC and with the collector implementations. It’s tricky to find the abstractions that work for all GC algorithms, whether they mark in place or move their objects, and whether they mark the heap precisely or if there are some conservative edges. But if this is the sort of thing that interests you, voilà the [API for
users](https://github.com/wingo/whippet-gc/blob/main/gc-ephemeron.h) and the [API to and from collector
implementations](https://github.com/wingo/whippet-gc/blob/main/gc-ephemeron-internal.h).

And, that’s it! I am looking forward to climbing out of this GC hole, one blog at a time. There are just a few more features before I can seriously attack integrating this into Guile. Until the next time, happy hacking :)

## related articles

- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [whippet update: faster evacuation, eager sweeping of empty blocks](/archives/2024/08/25/whippet-update-faster-evacuation-eager-sweeping-of-empty-blocks)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [an annoying failure mode of copying nurseries](/archives/2025/01/13/an-annoying-failure-mode-of-copying-nurseries)
- [finalizers, guardians, phantom references, et cetera](/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera)
- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)

Comments are closed.
