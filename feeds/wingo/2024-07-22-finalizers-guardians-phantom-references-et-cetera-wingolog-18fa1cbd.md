---
title: finalizers, guardians, phantom references, et cetera — wingolog
url: https://wingolog.org/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera
published: "2024-07-22T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera
---

# finalizers, guardians, phantom references, et cetera — wingolog

## [finalizers, guardians, phantom references, et cetera](/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera)

22 July 2024 9:27 AM

- [guile](/tags/guile)
- [garbage collection](/tags/garbage%20collection)
- [gc](/tags/gc)
- [whippet](/tags/whippet)
- [finalizers](/tags/finalizers)
- [ephemerons](/tags/ephemerons)
- [weak references](/tags/weak%20references)
- [igalia](/tags/igalia)
- [finalization](/tags/finalization)

Consider [guardians](https://dl.acm.org/doi/abs/10.1145/173262.155110). Compared to finalizers, in which the [cleanup procedures are run concurrently
with the mutator, by the garbage
collector](https://wingolog.org/archives/2012/02/16/unexpected-concurrency), guardians allow the mutator to control concurrency. See [Guile’s
documentation](https://www.gnu.org/software/guile/manual/html_node/Guardians.html) for more notes. Java’s [PhantomReference](https://docs.oracle.com/javase/8/docs/api/java/lang/ref/PhantomReference.html) / [ReferenceQueue](https://docs.oracle.com/javase/8/docs/api/java/lang/ref/ReferenceQueue.html) seems to be similar in spirit, though the details differ.

### questions

If we want guardians, how should we implement them in [Whippet](https://github.com/wingo/whippet)? How do they relate to [ephemerons and
finalizers](https://wingolog.org/archives/2022/10/31/ephemerons-and-finalizers)?

It would be a shame if guardians were primitive, as they are a relatively niche language feature. Guile has them, yes, but really what Guile has is bugs: because Guile implements guardians on top of [BDW-GC’s
finalizers](https://github.com/ivmai/bdwgc/blob/master/docs/finalization.md) (without topological ordering), all the other things that finalizers might do in Guile (e.g. closing file ports) might run at the same time as the objects protected by guardians. For the specific object being guarded, this isn’t so much of a problem, because when you put an object in the guardian, it arranges to prepend the guardian finalizer before any existing finalizer. But when a whole clique of objects becomes unreachable, objects referred to by the guarded object may be finalized. So the object you get back from the guardian might refer to, say, already-closed file ports.

The guardians-are-primitive solution is to add a special guardian pass to the collector that will identify unreachable guarded objects. In this way, the transitive closure of all guarded objects will be already visited by the time finalizables are computed, protecting them from finalization. This would be sufficient, but is it necessary?

### answers?

Thinking more abstractly, perhaps we can solve this issue and others with a set of finalizer priorities: a collector could have, say, 10 finalizer priorities, and run the [finalizer
fixpoint](https://wingolog.org/archives/2022/10/31/ephemerons-and-finalizers) once per priority, in order. If no finalizer is registered at a given priority, there is no overhead. A given embedder (Guile, say) could use priority 0 for guardians, priority 1 for “normal” finalizers, and ignore the other priorities. I think such a facility could also support other constructs, including Java’s phantom references, weak references, and reference queues, especially when combined with ephemerons.

Anyway, all this is a note for posterity. Are there other interesting mutator-exposed GC constructs that can’t be implemented with a combination of ephemerons and multi-priority finalizers? Do let me know!

## related articles

- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [are ephemerons primitive?](/archives/2022/11/28/are-ephemerons-primitive)
- [ephemerons and finalizers](/archives/2022/10/31/ephemerons-and-finalizers)
- [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)
- [parallel ephemeron tracing](/archives/2023/01/24/parallel-ephemeron-tracing)

Comments are closed.
