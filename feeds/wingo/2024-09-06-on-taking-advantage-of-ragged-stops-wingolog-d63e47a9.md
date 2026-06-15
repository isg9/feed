---
title: on taking advantage of ragged stops — wingolog
url: https://wingolog.org/archives/2024/09/06/on-taking-advantage-of-ragged-stops
published: "2024-09-06T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/09/06/on-taking-advantage-of-ragged-stops
---

# on taking advantage of ragged stops — wingolog

## [on taking advantage of ragged stops](/archives/2024/09/06/on-taking-advantage-of-ragged-stops)

6 September 2024 12:15 PM

- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [concurrency](/tags/concurrency)
- [ragged stop](/tags/ragged%20stop)
- [parallelism](/tags/parallelism)
- [whippet](/tags/whippet)
- [guile](/tags/guile)

Many years ago I read one of those Cliff Click “here’s what I learned” articles in which he was giving advice about garbage collector design, and one of the recommendations was that at a GC pause, running mutator threads should cooperate with the collector by identifying roots from their own stacks. You can read a similar assertion in their VEE2005 paper, [*The Pauseless GC*
*Algorithm*](https://www.usenix.org/legacy/events/vee05/full_papers/p46-click.pdf), though this wasn’t the source of the information.

One motivation for the idea was locality: a thread’s stack is already local to a thread. Then specifically in the context of a pauseless collector, you need to avoid races between the collector and the mutator for a thread’s stack, and having the thread visit its own stack neatly handles this problem.

However, I am not so interested any more in (so-called) pauseless collectors; though I have not measured myself, I am convinced enough by the arguments in the [*Distilling the real costs of production garbage*
*collectors*](https://ieeexplore.ieee.org/abstract/document/9804613) paper, which finds that state of the art pause-minimizing collectors actually increase both average and p99 latency, relative to a well-engineered collector with a pause phase. So, the racing argument is not so relevant to me, because a pause is happening anyway.

There was one more argument that I thought was interesting, which was that having threads visit their own stacks is a kind of cheap parallelism: the mutator threads are already there, they might as well do some work; it could be that it saves time, if other threads haven’t seen the safepoint yet. Mutators exhibit a *ragged stop*, in the sense that there is no clean cutoff time at which all mutators stop simultaneously, only a time after which no more mutators are running.

[Visiting roots during a ragged stop introduces concurrency between the mutator
and the
collector](https://wingolog.org/archives/2022/07/20/unintentional-concurrency), which is not exactly free; notably, it prevents objects marked while mutators are running from being evacuated. Still, it could be worth it in some cases.

Or so I thought! Let’s try to look at the problem analytically. Consider that you have a system with *N* processors, a stop-the-world GC with *N* tracing threads, and *M* mutator threads. Let’s assume that we want to minimize GC latency, as defined by the time between GC is triggered and the time that mutators resume. There will be one *triggering thread* that causes GC to begin, and then *M*–1 *remote* *threads* that need to reach a safepoint before the GC pause can begin.

The total amount of work that needs to be done during GC can be broken down into *rootsi*, the time needed to visit roots for mutator *i*, and then *graph*, the time to trace the transitive closure of live objects. We want to know whether it’s better to perform *rootsi* during the ragged stop or in the GC pause itself.

Let’s first look to the case where *M* is 1 (just one mutator thread). If we visit roots before the pause, we have

latencyragged,M=1=roots0+graphN

Which is to say, thread 0 triggers GC, visits its own roots, then enters the pause in which the whole graph is traced by all workers with maximum parallelism. It may be that graph tracing doesn’t fully parallelize, for example if the graph has a long singly-linked list, but the parallelism with be maximal within the pause as there are *N* workers tracing the graph.

If instead we visit roots within the pause, we have:

latencypause,M=1=roots0+graphN

This is strictly better than the ragged-visit latency.

If we have two threads, then we will need to add in some delay, corresponding to the time it takes for remote threads to reach a safepoint. Let’s assume that there is a maximum period (in terms of instructions) at which [a mutator will check for
safepoints](https://wingolog.org/archives/2023/10/16/on-safepoints). In that case the worst-case delay will be a constant, and we add it on to the latency. Let us assume also there are more than two threads available. The marking-roots-during-the-pause case it’s easiest to analyze:

latencypause,M=2=delay+roots0+roots1+graphN

In this case, a ragged visit could win: while the triggering thread is waiting for the remote thread to stop, it could perform *roots0*, moving the work out of the pause, reducing pause time, and reducing latency, for free.

latencyragged,M=2=delay+roots1+graphN

However, we only have this win if the root-visiting time is smaller than the safepoint delay; otherwise we are just prolonging the pause. Thing is, you don’t know in general. If indeed the root-visiting time is short, relative to the delay, we can assume the *roots* elements of our equation are 0, and so the choice to mark during ragged stop doesn’t matter at all! If we assume instead that root-visiting time is long, then it is suboptimally parallelised: under-parallelised if we have more than *M* cores, oversubscribed if *M* is greater than *N*, and needlessly serializing before the pause while it waits for the last mutator to finish marking its roots. What’s worse, root-visiting actually slows down *delay*, because the oversubscribed threads compete with visitation for CPU time.

So in summary, I plan to switch away from doing GC work during the ragged stop. It is complexity that doesn’t pay. Onwards and upwards!

## related articles

- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [whippet update: faster evacuation, eager sweeping of empty blocks](/archives/2024/08/25/whippet-update-faster-evacuation-eager-sweeping-of-empty-blocks)
- [parallel ephemeron tracing](/archives/2023/01/24/parallel-ephemeron-tracing)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)

### One response

1. Brendan MacDonell says:[7 September 2024 6:48 PM](#69eec3377e59961fe0e22c7ee8c57247f4667341)

   I’m not really a GC guy, but I suspect that the fraction of the roots in core-private caches makes a large difference in the benefit of doing GC during a ragged stop. In the extreme, if a thread’s full stack was recently modified and resident in the local L1+L2, then walking it in the mutator will be just as fast as walking it in N other threads — the rate at which the core can transfer data from its caches to its own registers is the same as it can transfer it to the interconnect. And in this extreme case, if you have N mutator threads running on N cores, then it will be faster to walk each thread’s stack locally than to shuffle the work of walking the roots across N tracing threads, since the total interconnect bandwidth is usually lower than the sum of the core-private cache bandwidths. This is especially true for systems made of multiple chiplets or with multiple sockets, where inter-chiplet and inter-socket bandwidth is limited, and the L3 caches are not shared between chiplets / sockets.

   Of course, it’s hard to say how much this matters in general — it’s very much dependent on the mutator and the hardware it’s running on. But for workloads that mostly perform minor collections, have 1:1 thread:core affinity, and are running on large systems, I would expect improved locality from visiting roots during a ragged stop to have a (small) positive impact on pause time.

Comments are closed.
