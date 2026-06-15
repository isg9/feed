---
title: Preemption vs. Event Ordering
url: https://blog.regehr.org/archives/506
published: "2011-04-05T20:53:11Z"
feed: regehr
guid: http://blog.regehr.org/?p=506
---

# Preemption vs. Event Ordering

There’s no doubt that concurrent programming is hard, but it’s not always clear exactly why. Generally, eliminating [data races](https://blog.regehr.org/archives/490) is not the hard part. On the other hand, dealing with event ordering problems can be extremely difficult. To put that another way, if we remove preemption from the picture by programming with non-preemptive threads or a single-threaded event loop, much of the difficulty of concurrent programming remains.

In 1998/99, while doing an internship at Microsoft Research, I wrote some moderately complicated code for the NT kernel. Since kernel debugging was a pain, I wrote a simple event-driven simulator that emulated the kernel’s execution environment well enough to support running my code in user mode. I was depressed to learn that debugging code in the single-threaded simulator was not hugely easier than debugging the live version. The crux of the issue was atomicity: not “concurrency atomicity” but rather something like “invariant atomicity.” The kernel code I was writing was not only highly stateful, but also quite incremental, meaning that event handlers would often drop what they were doing and other handlers would pick up the job later. Thus, event handlers were seeing extremely unclean system states. The invariants were more complicated than I could easily grasp, which of course was the root of the problem. Software systems using callbacks can be surprisingly hard to get right for similar reasons, and I suspect that many problems with error-handling code have a similar root cause.

In contrast with my NT kernel simulator example, an event driven system will be far easier to get right when stable states of data structures coincide nicely with event boundaries. Adding concurrency/parallelism may not make program development any more difficult if the points of interaction between concurrent flows coincide with relatively stable points in the system’s invariants.

One part of the solution to event ordering problems is to design systems where the invariants are well-structured and unclean system states are exposed to as little code as possible. Another part is to use programming languages and environments that have better support for explicit invariants. Lamentably, almost all of my development these days is either systems programming or scripting — both domains where invariant-based programming (design by contract for example) is weakly supported.
