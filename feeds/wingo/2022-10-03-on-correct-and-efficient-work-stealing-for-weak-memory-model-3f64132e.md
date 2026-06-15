---
title: on "correct and efficient work-stealing for weak memory models" — wingolog
url: https://wingolog.org/archives/2022/10/03/on-correct-and-efficient-work-stealing-for-weak-memory-models
published: "2022-10-03T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2022/10/03/on-correct-and-efficient-work-stealing-for-weak-memory-models
---

# on "correct and efficient work-stealing for weak memory models" — wingolog

## [on "correct and efficient work-stealing for weak memory models"](/archives/2022/10/03/on-correct-and-efficient-work-stealing-for-weak-memory-models)

3 October 2022 8:24 AM

- [concurrency](/tags/concurrency)
- [atomics](/tags/atomics)
- [parallelism](/tags/parallelism)
- [garbage collection](/tags/garbage%20collection)
- [work stealing](/tags/work%20stealing)
- [igalia](/tags/igalia)

Hello all, a quick post today. Inspired by [Rust as a Language for High Performance GC Implementation](http://users.cecs.anu.edu.au/~steveb/pubs/papers/rust-ismm-2016.pdf) by Yi Lin et al, a few months ago I had a look to see how the basic Rust concurrency facilities that they used were implemented.

One of the key components that Lin et al used was a Chase-Lev work-stealing double-ended queue (deque). The 2005 article [Dynamic Circular Work-Stealing Deque](https://www.dre.vanderbilt.edu/~schmidt/PDF/work-stealing-dequeue.pdf) by David Chase and Yossi Lev is a nice read defining this data structure. It's used when you have a single producer of values, but multiple threads competing to claim those values. This is useful when implementing per-CPU schedulers or work queues; each CPU pushes on any items that it has to its own deque, and pops them also, but when it runs out of work, it goes to see if it can steal work from other CPUs.

The 2013 paper [Correct and Efficient Work-Stealing for Weak Memory Models](https://fzn.fr/readings/ppopp13.pdf) by Nhat Min Lê et al updates the Chase-Lev paper by relaxing the concurrency primitives from the original big-hammer sequential-consistency operations used in the Chase-Lev paper to an appropriate mix of C11 relaxed, acquire/release, and sequentially-consistent operations. The paper therefore has a C11 translation of the original algorithm, and a proof of correctness. It's quite pleasant. Here's the [a version in Rust's `crossbeam` crate](https://docs.rs/crossbeam/0.3.2/src/crossbeam/sync/chase_lev.rs.html#11-605), and [here's the same thing in C](https://github.com/wingo/whippet-gc/blob/main/parallel-tracer.h#L14).

I had been using this updated C11 Chase-Lev deque implementation for a while with no complaints in a parallel garbage collector. Each worker thread would keep a local unsynchronized work queue, which when it grew too large would donate half of its work to a per-worker Chase-Lev deque. Then if it ran out of work, it would go through all the workers, seeing if it could steal some work.

My use of the deque was thus limited to only the [`push`](https://github.com/wingo/whippet-gc/blob/main/parallel-tracer.h#L158-L170) and [`steal`](https://github.com/wingo/whippet-gc/blob/main/parallel-tracer.h#L213-L230) primitives, but not [`take`](https://github.com/wingo/whippet-gc/blob/main/parallel-tracer.h#L187-L211) (using the language of the Lê et al paper). `take` is like `steal`, except that it takes values from the producer end of the deque, and it can't run concurrently with `push`. In practice `take` only used by the the thread that also calls `push`. Cool.

Well I thought, you know, before a worker thread goes to steal from some other thread, it might as well see if it can do a cheap `take` on its own deque to see if it could take back some work that it had previously offloaded there. But here I ran into a bug. A brief internet search didn't turn up anything, so here we are to mention it.

Specifically, there is a bug in the Lê et al paper that is not in the Chase-Lev paper. The original paper is in Java, and the C11 version is in, well, C11. The issue is.... integer overflow! In brief, `push` will increment `bottom`, and `steal` increments `top`. `take`, on the other hand, can decrement `bottom`. It uses `size_t` to represent `bottom`. I think you see where this is going; if you `take` on an empty deque in the initial state, you create a situation that looks just like a deque with `(size_t)-1` elements, causing garbage reads and all kinds of delightful behavior.

The funny thing is that I looked at the proof and I looked at the industrial applications of the deque and I thought well, I just have to transcribe the algorithm exactly and I'll be golden. But it just goes to show that proving one property of an algorithm doesn't necessarily imply that the algorithm is correct.

## related articles

- [whippet update: faster evacuation, eager sweeping of empty blocks](/archives/2024/08/25/whippet-update-faster-evacuation-eager-sweeping-of-empty-blocks)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [on taking advantage of ragged stops](/archives/2024/09/06/on-taking-advantage-of-ragged-stops)
- [the last 5 years of V8's garbage collector](/archives/2023/12/07/the-last-5-years-of-v8s-garbage-collector)
- [parallel ephemeron tracing](/archives/2023/01/24/parallel-ephemeron-tracing)

### One response

1. [Samuel Michael Squire](https://github.com/samsquire/) says:[3 October 2022 7:16 PM](#26302de0d4f0523b9cd7b1007b60f996ca26cd2b)

   You might enjoy this multiconsumer multiproducer ringbuffer in C++. I ported it to Java

   https://www.linuxjournal.com/content/lock-free-multi-producer-multi-consumer-queue-ring-buffer

   https://github.com/samsquire/multiversion-concurrency-control

   You might also enjoy this whitepaper

   Wait-Free Queues With Multiple Enqueuers and Dequeuers

   This implementation doesn't steal but helps other workers.

Comments are closed.
