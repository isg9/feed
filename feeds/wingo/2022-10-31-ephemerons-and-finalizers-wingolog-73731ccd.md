---
title: ephemerons and finalizers — wingolog
url: https://wingolog.org/archives/2022/10/31/ephemerons-and-finalizers
published: "2022-10-31T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2022/10/31/ephemerons-and-finalizers
---

# ephemerons and finalizers — wingolog

## [ephemerons and finalizers](/archives/2022/10/31/ephemerons-and-finalizers)

31 October 2022 12:21 PM

- [garbage collection](/tags/garbage%20collection)
- [ephemerons](/tags/ephemerons)
- [finalizers](/tags/finalizers)
- [finalization](/tags/finalization)
- [guardians](/tags/guardians)
- [weak references](/tags/weak%20references)
- [js](/tags/js)
- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [igalia](/tags/igalia)

Good day, hackfolk. Today we continue the series on garbage collection with some notes on ephemerons and finalizers.

**conjunctions and disjunctions**

First described in a [1997 paper by Barry
Hayes](https://dl.acm.org/doi/abs/10.1145/263700.263733), which attributes the invention to George Bosworth, ephemerons are a kind of weak key-value association.

Thinking about the problem abstractly, consider that the garbage collector’s job is to keep live objects and recycle memory for dead objects, making that memory available for future allocations. Formally speaking, we can say:

- An object is live if it is in the root set

- An object is live it is referenced by any live object.

This circular definition uses the word *any*, indicating a disjunction: a single incoming reference from a live object is sufficient to mark a referent object as live.

Ephemerons augment this definition with a conjunction:

- An object *V* is live if, for an ephemeron *E* containing an association betweeen objects *K* and *V*, both *E* and *K* are live.

This is a more annoying property for a garbage collector to track. If you happen to mark *K* as live and then you mark *E* as live, then you can just continue to trace *V*. But if you see *E* first and then you mark *K*, you don’t really have a direct edge to *V*. (Indeed this is one of the main purposes for ephemerons: associating data with an object, here *K*, without actually modifying that object.)

During a trace of the object graph, you can know if an object is definitely alive by checking if it was visited already, but if it wasn’t visited yet that doesn’t mean it’s *not* live: we might just have not gotten to it yet. Therefore one common implementation strategy is to wait until tracing the object graph is done before tracing ephemerons. But then we have another annoying problem, which is that tracing ephemerons can result in finding more live ephemerons, requiring another tracing cycle, and so on. Mozilla’s Steve Fink wrote a [nice article on
this
issue](https://blog.mozilla.org/sfink/2022/06/09/ephemeron-tables-aka-javascript-weakmaps/) earlier this year, with some mitigations.

**finalizers aren’t quite ephemerons**

All that is by way of introduction. If you just have an object graph with strong references and ephemerons, our definitions are clear and consistent. However, if we add some more features, we muddy the waters.

Consider finalizers. The basic idea is that you can attach one or a number of finalizers to an object, and that when the object becomes unreachable (not live), the system will invoke a function. One way to imagine this is a global association from finalizable object *O* to finalizer *F*.

As it is, this definition is underspecified in a few ways. One, what happens if *F* references *O*? It could be a GC-managed closure, after all. Would that prevent *O* from being collected?

Ephemerons solve this problem, in a way; we could trace the table of finalizers like a table of ephemerons. In that way *F* would only be traced if *O* is live already, so that by itself it wouldn’t keep *O* alive. But then if *O* becomes dead, you’d want to invoke *F*, so you’d need it to be live, so reachability of finalizers is not quite the same as ephemeron-reachability: indeed logically all *F* values in the finalizer table are live, because they all will be invoked at some point.

In the end, if *F* references *O*, then *F* actually keeps *O* alive. Whether this prevents *O* from being finalized depends on our definition for finalizability. We could say that an object is finalizable if it is found to be unreachable after a full trace, and the finalizers *F* are in the root set. Or we could say that an object is finalizable if it is unreachable after a partial trace, in which finalizers are not themselves in the initial root set, and instead we trace them after determining the finalizable set.

Having finalizers in the initial root set is unfortunate: there’s no quick check you can make when adding a finalizer to signal this problem to the user, and it’s very hard to convey to a user exactly how it is that an object is referenced. You’d have to add lots of gnarly documentation on top of the already [unavoidable](https://wingolog.org/archives/2012/02/16/unexpected-concurrency) [gnarliness](https://www.gnu.org/software/guile/manual/html_node/Foreign-Object-Memory-Management.html) that you already had to write. But, perhaps it is a local maximum.

Incidentally, you might think that you can get around these issues by saying “don’t reference objects from their finalizers”, and that’s true in a way. However it’s not uncommon for finalizers to receive the object being finalized as an argument; after all, it’s that object which probably encapsulates the information necessary for its finalization. Of course this can lead to the finalizer prolonging the longevity of an object, perhaps by storing it to a shared data structure. This is a risk for correct program construction ( [the finalized object might
reference live-but-already-finalized
objects](http://shiftleft.com/mirrors/www.hpl.hp.com/techreports/2002/HPL-2002-335.pdf)), but not really a burden for the garbage collector, except in that it’s a serialization point in the collection algorithm: you trace, you compute the finalizable set, then you have to trace the finalizables again.

**ephemerons vs finalizers**

The gnarliness continues! Imagine that *O* is associated with a finalizer *F*, and also, via ephemeron *E*, some auxiliary data *V*. Imagine that at the end of the trace, *O* is unreachable and so will be dead. Imagine that *F* receives *O* as an argument, and that *F* looks up the association for *O* in *E*. Is the association to *V* still there?

Guile’s [documentation](https://www.gnu.org/software/guile/manual/html_node/Guardians.html) on [guardians](https://dl.acm.org/doi/abs/10.1145/173262.155110), a finalization-like facility, specifies that weak associations (i.e. ephemerons) remain in place when an object becomes collectable, though I think in practice this has been broken since Guile switched to the BDW-GC collector some 20 years ago or so and I would like to fix it.

One nice solution falls out if you prohibit resuscitation by not including finalizer closures in the root set and not passing the finalizable object to the finalizer function. In that way you will never be able to look up *E* × *O* ⇒ *V*, because you don’t have *O*. This is the path that JavaScript has taken, for example, with [WeakMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap) and [FinalizationRegistry](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/FinalizationRegistry).

However if you allow for resuscitation, for example by passing finalizable objects as an argument to finalizers, I am not sure that there is an optimal answer. Recall that with resuscitation, the trace proceeds in three phases: first trace the graph, then compute and enqueue the finalizables, then trace the finalizables. When do you perform the conjunction for the ephemeron trace? You could do so after the initial trace, which might augment the live set, protecting some objects from finalization, but possibly missing ephemeron associations added in the later trace of finalizable objects. Or you could trace ephemerons at the very end, preserving all associations for finalizable objects (and their referents), which would allow more objects to be finalized at the same time.

Probably if you trace ephemerons early you will also want to trace them later, as you would do so because you think ephemeron associations are important, as you want them to prevent objects from being finalized, and it would be weird if they were not present for finalizable objects. This adds more serialization to the trace algorithm, though:

1. (Add finalizers to the root set?)

2. Trace from the roots

3. Trace ephemerons?

4. Compute finalizables

5. Trace finalizables (and finalizer closures if not done in 1)

6. Trace ephemerons again?

These last few paragraphs are the reason for today’s post. It’s not clear to me that there is an optimal way to compose ephemerons and finalizers in the presence of resuscitation. If you add finalizers to the root set, you might prevent objects from being collected. If you defer them until later, you lose the optimization that you can skip steps 5 and 6 if there are no finalizables. If you trace (not-yet-visited) ephemerons twice, that’s overhead; if you trace them only once, the user could get what they perceive as premature finalization of otherwise reachable objects.

In Guile I think I am going to try to add finalizers to the root set, pass the finalizable to the finalizer as an argument, and trace ephemerons twice if there are finalizable objects. I think this wil minimize incoming bug reports. I am bummed though that I can’t eliminate them by construction.

Until next time, happy hacking!

## related articles

- [are ephemerons primitive?](/archives/2022/11/28/are-ephemerons-primitive)
- [finalizers, guardians, phantom references, et cetera](/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)
- [javascript weakmaps should be iterable](/archives/2024/08/19/javascript-weakmaps-should-be-iterable)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)

### One response

1. Taylor R Campbell says:[6 November 2022 3:48 PM](#3d96c3793a2d1d5a67f7ef9317d21398d382c6f3)

   MIT Scheme addresses this problem in a much simpler way. It may not help if you already have a lot of code using the existing finalizer API, but I thought I’d share it anyway.

   Instead of associating a traceable object with a procedure that performs destruction using the object, MIT Scheme associates a traceable object with an *underlying resource* together with a procedure that performs destruction using *only the underlying resource*.

   For example, suppose you have a Scheme object representing an open file on Unix, where the underlying resource is represented by the file descriptor number. Instead of writing

   (add-finalizer! f (lambda (f) (syscall:close (file-descriptor-number f)))),

   you write

   (add-finalizer! f (file-descriptor-number f) (lambda (d) (syscall:close d))).

   This way there’s never any need to worry about resuscitating the object at all – it can’t happen. The garbage collector can actually destroy the object and safely recycle its memory before the finalizer procedure runs.

   MIT Scheme’s GC itself only knows how to break weak references (and ephemerons, but they’re not relevant here: a list of (cons (weak-ref traceable-object) underlying-resource) is enough) and invoke a hook once it has completed, which the runtime system uses to go through the list of finalizer procedures and call them as needed.

   (The API looks a little different in practice: in MIT Scheme, a finalizer is a *collection* of (object, resource) pairs with a single associated destruction procedure, so it’s more like (define finalizer (make-gc-finalizer syscall:close file? file-descriptor-number set-file-descriptor-number!)), and then when you create a file, (add-to-gc-finalizer! finalizer f). But that’s a minor API difference.)

Comments are closed.
