---
title: the sticky mark-bit algorithm — wingolog
url: https://wingolog.org/archives/2022/10/22/the-sticky-mark-bit-algorithm
published: "2022-10-22T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2022/10/22/the-sticky-mark-bit-algorithm
---

# the sticky mark-bit algorithm — wingolog

## [the sticky mark-bit algorithm](/archives/2022/10/22/the-sticky-mark-bit-algorithm)

22 October 2022 8:42 AM

- [garbage collection](/tags/garbage%20collection)
- [mark-sweep](/tags/mark-sweep)
- [sticky mark bit](/tags/sticky%20mark%20bit)
- [whippet](/tags/whippet)
- [demers](/tags/demers)
- [javascriptcore](/tags/javascriptcore)
- [igalia](/tags/igalia)
- [gc](/tags/gc)

Good day, hackfolk!

# The Sticky Mark-Bit Algorithm

Also an intro to mark-sweep GC

7 Oct 2022 – Igalia

Andy Wingo

A funny post today; I gave an internal presentation at work recently describing the so-called “sticky mark bit” algorithm. I figured I might as well post it here, as a gift to you from your local garbage human.

## Automatic Memory Management

“Don’t `free`, the system will do it for you”

Eliminate a class of bugs: use-after-free

Relative to bare `malloc`/ `free`, qualitative performance improvements

- cheap bump-pointer allocation
- cheap reclamation/recycling
- better locality

Continuum: `bmalloc` / `tcmalloc` grow towards GC

Before diving in though, we start with some broad context about *automatic memory management*. The term mostly means “garbage collection” these days, but really it describes a component of a system that provides fresh memory for new objects and *automatically* reclaims memory for objects that won’t be needed in the program’s future. This stands in contrast to *manual memory management*, which relies on the programmer to `free` their objects.

Of course, automatic memory management ensures some valuable system-wide properties, like lack of [use-after-free
vulnerabilities](https://twitter.com/LazyFishBarrel). But also by enlarging the scope of the memory management system to include full object lifetimes, we gain some potential speed benefits, for example eliminating any cost for `free`, in the case of e.g. a semi-space collector.

## Automatic Memory Management

Two strategies to determine live object graph

- Reference counting
- Tracing

What to do if you trace

- Mark, and then sweep or compact
- Evacuate

Tracing O(n) in live object count

I should mention that reference counting is a form of automatic memory management. It’s not enough on its own; unreachable cycles in the object reference graph have to be detected either by a heap tracer or broken by weak references.

It used to be that we GC nerds made fun of reference counting as being an expensive, half-assed solution that didn’t work very well, but there have been some [fundamental advances in the state of the
art](https://twitter.com/stevemblackburn/status/1518386133746728960) in the last 10 years or so.

But this talk is more about the other kind of memory management, which involves periodically tracing the graph of objects in the heap. Generally speaking, as you trace you can do one of two things: *mark* the object, simply setting a bit indicating that an object is live, or *evacuate* the object to some other location. If you mark, you may choose to then *compact* by sliding all objects down to lower addresses, squeezing out any holes, or you might *sweep* all holes into a free list for use by further allocations.

## Mark-sweep GC (1/3)

```
freelist := []

allocate():
  if freelist is empty: collect()
  return freelist.pop()

collect():
  mark()
  sweep()
  if freelist is empty: abort
```

Concretely, let’s look closer at mark-sweep. Let’s assume for the moment that all objects are the same size. Allocation pops fresh objects off a freelist, and collects if there is none. Collection does a mark and then a sweep, aborting if sweeping yielded no free objects.

## Mark-sweep GC (2/3)

```
mark():
  worklist := []
  for ref in get_roots():
    if mark_one(ref):
      worklist.add(ref)
  while worklist is not empty:
    for ref in trace(worklist.pop()):
      if mark_one(ref):
        worklist.add(ref)

sweep():
  for ref in heap:
    if marked(ref):
      unmark_one(ref)
    else
      freelist.add(ref)
```

Going a bit deeper, here we have some basic implementations of `mark` and `sweep`. Marking starts with the *roots*: edges from outside the automatically-managed heap indicating a set of initial live objects. You might get these by maintaining a stack of objects that are currently in use. Then it traces references from these roots to other objects, until there are no more references to trace. It will visit each live object exactly once, and so is O(n) in the number of live objects.

Sweeping requires the ability to iterate the heap. With the precondition here that `collect` is only ever called with an empty freelist, it will clear the mark bit from each live object it sees, and otherwise add newly-freed objects to the global freelist. Sweep is O(n) in total heap size, but some optimizations can amortize this cost.

## Mark-sweep GC (3/3)

```
marked := 1

get_tag(ref):
  return *(uintptr_t*)ref
set_tag(ref, tag):
  *(uintptr_t*)ref = tag

marked(ref):
  return (get_tag(ref) & 1) == marked
mark_one(ref):
  if marked(ref): return false;
  set_tag(ref, (get_tag(ref) & ~1) | marked)
  return true
unmark_one(ref):
  set_tag(ref, (get_tag(ref) ^ 1))
```

Finally, some details on how you might represent a mark bit. If a *ref* is a pointer, we could store the mark bit in the first word of the objects, as we do here. You can choose instead to store them in a side table, but it doesn’t matter for today’s example.

## Observations

Freelist implementation crucial to allocation speed

Non-contiguous allocation suboptimal for locality

World is stopped during `collect()`: “GC pause”

`mark` O(n) in live data, `sweep` O(n) in total heap size

Touches a lot of memory

The salient point is that these O(n) operations happen when the world is stopped. This can be noticeable, even taking seconds for the largest heap sizes. It sure would be nice to have the benefits of GC, but with lower pause times.

## Optimization: rotate mark bit

```
flip():
  marked ^= 1

collect():
  flip()
  mark()
  sweep()
  if freelist is empty: abort

unmark_one(ref):
  pass
```

Avoid touching mark bits for live data

Incidentally, before moving on, I should mention an optimization to mark bit representation: instead of clearing the mark bit for live objects during the sweep phase, we could just choose to flip our interpretation of what the mark bit means. This allows `unmark_one` to become a no-op.

## Reducing pause time

*Parallel tracing*: parallelize `mark`. Clear improvement, but speedup depends on object graph shape (e.g. linked lists).

*Concurrent tracing*: `mark` while your program is running. Tricky, and not always a win (“Retrofitting Parallelism onto OCaml”, ICFP 2020).

*Partial tracing*: `mark` only a subgraph. Divide space into regions, record inter-region links, collect one region only. Overhead to keep track of inter-region edges.

Now, let’s revisit the pause time question. What can we do about it? In general there are three strategies.

## Generational GC

Partial tracing

Two spaces: nursery and oldgen

Allocations in nursery (usually)

Objects can be *promoted*/ *tenured* from nursery to oldgen

Minor GC: just trace the nursery

Major GC: trace nursery and oldgen

“Objects tend to die young”

Overhead of old-to-new edges offset by less amortized time spent tracing

Today’s talk is about partial tracing. The basic idea is that instead of tracing the whole graph, just trace a part of it, ideally a small part.

A simple and effective strategy for partitioning a heap into subgraphs is generational garbage collection. The idea is that objects tend to die young, and that therefore it can be profitable to focus attention on collecting objects that were allocated more recently. You therefore partition the heap graph into two parts, young and old, and you generally try to trace just the young generation.

The difficulty with partitioning the heap graph is that you need to maintain a set of inter-partition edges, and you do so by imposing overhead on the user program. But a generational partition minimizes this cost because you never do an only-old-generation collection, so you don’t need to remember new-to-old edges, and mutations of old objects are less common than new.

## Generational GC

Usual implementation: semispace nursery and mark-compact oldgen

Tenuring via evacuation from nursery to oldgen

Excellent locality in nursery

Very cheap allocation (bump-pointer)

But... evacuation requires all incoming edges to an object to be updated to new location

Requires precise enumeration of all edges

Usually the generational partition is reflected in the address space: there is a nursery and it is in these pages and an oldgen in these other pages, and never the twain shall meet. To *tenure* an object is to actually move it from the nursery to the old generation. But moving objects requires that the collector be able to enumerate all incoming edges to that object, and then to have the collector update them, which can be a bit of a hassle.

## JavaScriptCore

No precise stack roots, neither in generated nor C++ code

Compare to V8’s `Handle<>` in C++, stack maps in generated code

Stack roots *conservative*: integers that happen to hold addresses of objects treated as object graph edges

(Cheaper implementation strategy, can eliminate some bugs)

Specifically in JavaScriptCore, the JavaScript engine of WebKit and the Safari browser, we have a problem. JavaScriptCore uses a technique known as “conservative root-finding”: it just iterates over the words in a thread’s stack to see if any of those words might reference an object on the heap. If they do, JSC conservatively assumes that it is indeed a reference, and keeps that object live.

Of course a given word on the stack could just be an integer which happens to be an object’s address. In that case we would hold on to too much data, but that’s not so terrible.

Conservative root-finding is again one of those things that GC nerds like to make fun of, but the pendulum seems to be swinging back its way; perhaps another article on that some other day.

## JavaScriptCore

Automatic memory management eliminates use-after-free...

...except when combined with manual memory management

Prevent type confusion due to reuse of memory for object of different shape

`addrof`/ `fakeobj` primitives: `phrack.org/issues/70/3.html`

Type-segregated heaps

No evacuation: no generational GC?

The other thing about JSC is that it is constantly under attack by malicious web sites, and that any bug in it is a step towards hackers taking over your phone. Besides bugs inside JSC, there are bugs also in the objects exposed to JavaScript from the web UI. Although use-after-free bugs are impossible with a fully traceable object graph, references to and from DOM objects might not be traceable by the collector, instead referencing GC-managed objects by reference counting or weak references or even manual memory management. Bugs in these interfaces are a source of exploitable vulnerabilities.

In brief, there seems to be a decent case for trying to mitigate use-after-free bugs. Beyond the nuclear option of not freeing, one step we could take would be to avoid re-using memory between objects of different shapes. So you have a heap for objects with 3 fields, another objects with 4 fields, and so on.

But it would seem that this mitigation is at least somewhat incompatible with the usual strategy of generational collection, where we use a semi-space nursery. The nursery memory gets re-used all the time for all kinds of objects. So does that rule out generational collection?

## Sticky mark bit algorithm

```
collect(is_major=false):
  if is_major: flip()
  mark(is_major)
  sweep()
  if freelist is empty:
    if is_major: abort
    collect(true)

mark(is_major):
  worklist := []
  if not is_major:
    worklist += remembered_set
    remembered_set := []
  ...
```

Turns out, you can generationally partition a mark-sweep heap.

Recall that to visit each live object, you trace the heap, setting mark bits. To visit them all again, you have to clear the mark bit between traces. Our first `collect` implementation did so in `sweep`, via `unmark_one`; then with the optimization we switched to clear them all before the next trace in `flip()`.

Here, then, the trick is that you just don’t clear the mark bit between traces for a minor collection (tracing just the nursery). In that way all objects that were live at the previous collection are considered the old generation. Marking an object is tenuring, in-place.

There are just two tiny modifications to mark-sweep to implement sticky mark bit collection: one, flip the mark bit only on major collections; and two, include a *remembered set* in the roots for minor collections.

## Sticky mark bit algorithm

Mark bit from previous trace “sticky”: avoid `flip` for minor collections

Consequence: old objects not traced, as they are already marked

Old-to-young edges: the “remembered set”

Write barrier

```
write_field(object, offset, value):
  remember(object)
  object[offset] = value
```

The remembered set is maintained by instrumenting each write that the program makes with a little call out to code from the garbage collector. This code is the *write barrier*, and here we use it to add to the set of objects that might reference new objects. There are many ways to implement this write barrier but that’s a topic for another day.

## JavaScriptCore

Parallel GC: Multiple collector threads

Concurrent GC: `mark` runs while JS program running; “riptide”; interaction with write barriers

*Generational GC*: in-place, non-moving GC generational via sticky mark bit algorithm

Alan Demers, “Combining generational and conservative garbage collection: framework and implementations”, POPL ’90

So returning to JavaScriptCore and the general techniques for reducing pause times, I can summarize to note that it does them all. It traces both in parallel and concurrently, and it tries to trace just newly-allocated objects using the sticky mark bit algorithm.

## Conclusions

A little-used algorithm

Motivation for JSC: conservative roots

Original motivation: conservative roots; write barrier enforced by OS-level page protections

Revived in “Sticky Immix”

Better than nothing, not quite as good as semi-space nursery

I find that people that are interested in generational GC go straight for the semispace nursery. There are some advantages to that approach: allocation is generally cheaper in a semispace than in a mark space, locality among new objects is better, locality after tenuring is better, and you have better access locality during a nursery collection.

But if for some reason you find yourself unable to enumerate all roots, you can still take advantage of generational collection via the sticky mark-bit algorithm. It’s a simple change that improves performance, as long as you are able to insert write barriers on all heap object mutations.

The challenge with a sticky-mark-bit approach to generations is avoiding the O(n) sweep phase. There are a few strategies, but more on that another day perhaps.

And with that, presentation done. Until next time, happy hacking!

## related articles

- [ahead-of-time wasm gc in wastrel](/archives/2026/02/06/ahead-of-time-wasm-gc-in-wastrel)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [a whippet waypoint](/archives/2025/05/09/a-whippet-waypoint)
- [whippet lab notebook: untagged mallocs, bis](/archives/2025/03/07/whippet-lab-notebook-untagged-mallocs-bis)
- [tracepoints: gnarly but worth it](/archives/2025/02/14/tracepoints-gnarly-but-worth-it)

Comments are closed.
