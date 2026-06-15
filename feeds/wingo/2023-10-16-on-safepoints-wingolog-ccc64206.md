---
title: on safepoints — wingolog
url: https://wingolog.org/archives/2023/10/16/on-safepoints
published: "2023-10-16T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/10/16/on-safepoints
---

# on safepoints — wingolog

## [on safepoints](/archives/2023/10/16/on-safepoints)

16 October 2023 1:46 PM

- [garbage collection](/tags/garbage%20collection)
- [gc](/tags/gc)
- [immix](/tags/immix)
- [whippet](/tags/whippet)
- [safepoints](/tags/safepoints)
- [bdw](/tags/bdw)
- [tee-ball](/tags/tee-ball)
- [flubt](/tags/flubt)
- [guile](/tags/guile)

Hello all, a brief note today. For context, my big project over the last year and a half or so is [Whippet](https://github.com/wingo/whippet), a new garbage collector implementation. If everything goes right, Whippet will finding a better point on the space/time tradeoff curve than [Guile](https://gnu.org/s/guile/)‘s current garbage collector, [BDW-GC](https://github.com/ivmai/bdwgc/), and will make it into Guile, well, sometime.

But, testing a garbage collector is... hard. Methodologically it’s very tricky, though there has been some [recent progress in assessing
collectors in a more systematic
way](https://users.cecs.anu.edu.au/~steveb/pubs/papers/lbo-ispass-2022.pdf). Ideally of course you test against a real application, and barring that, against a macrobenchmark extracted from a real application. But garbage collectors are deeply intertwined with language run-times; to maximize the insight into GC performance, you need to minimize everything else, and to minimize non-collector cost, that means relying on a high-performance run-time (e.g. the JVM), which... is hard! It’s hard enough to get toy programs to work, but testing against a beast like the JVM is not easy.

In the case of Guile, it’s even more complicated, because the BDW-GC is a conservative collector. BDW-GC doesn’t require precise annotation of stack roots, and scans intraheap edges conservatively (unless you go out of your way to do otherwise). How can you test this against a semi-space collector if Guile can’t precisely enumerate all object references? The Immix-derived collector in Whippet can be configured to run with conservative stack roots (and even heap links), but how then to test the performance tradeoff of conservative vs precise tracing?

In my first iterations on Whippet, I hand-coded some benchmarks in C, starting with the classic [`gcbench`](https://hboehm.info/gc/gc_bench.html). I used [stack-allocated linked lists for precise
roots](https://github.com/wingo/whippet/blob/main/benchmarks/simple-roots-api.h), when run in precise-rooting mode. But, this is excruciating and error-prone; trying to write some more tests, I ran up against a wall of gnarly nasty C code. Not fun, and I wasn’t sure that it was representative in terms of workload; did the handle discipline have some kind of particular overhead? The usual way to do things in a high-performance system is to instead have *stack maps*, where the collector is able to precisely find roots on the stack and registers using a side table.

Of course the usual solution if something is not fun is to make it into a tools problem, so I wrote a little Scheme-to-C compiler, [Whiffle](https://github.com/wingo/whiffle/), purpose-built for testing Whippet. It’s a [baseline-style
compiler](https://wingolog.org/archives/2020/06/03/a-baseline-compiler-for-guile) that uses the C stack for control (function call and return), and a packed side stack of temporary values. The goal was to be able to generate some C that I could include in the collector’s benchmarks; I’ve gotten close but haven’t done that final step yet.

Anyway, because its temporaries are all in a packed stack, Whippet can always traverse the roots precisely. This means I can write bigger benchmark programs without worry, which will allow me to finish testing the collector. As a nominally separate project, Whiffle also tests that Whippet’s API is good for embedding. (Yes, the names are similar; if I thought that in the future I would need to say much more about Whiffle, I would rename it!)

I was able to [translate over the sparse benchmarks that Whippet already
had into Scheme](https://github.com/wingo/whiffle/tree/main/examples), and set about checking that they worked as expected. Which they did... mostly.

### the bug

The Whippet interface abstracts over different garbage collector implementations; the choice of which collector to use is made at compile-time. There’s the basic semi-space copying collector, with a large object space and ephemerons; there’s the Immix derived “whippet” collector (yes, it shares the same name); and there’s a shim that provides the Whippet API via the BDW-GC.

The benchmarks are multithreaded, except when using the semi-space collector which only supports one mutator thread. (I should fix that at some point.) I tested the benchmarks with one mutator thread, and they worked with all collectors. Yay!

Then I tested with multiple mutator threads. The Immix-derived collectors worked fine, in both precise and conservative modes. The BDW-GC collector... did not. With just one mutator thread it was fine, but with multiple threads I would get segfaults deep inside libgc.

### the side quest

Over on the fediverse, Daphe Preston-Kendal asks, [how does Whiffle deal
with tail calls?](https://c.im/@dpk/111094069310103918) The answer is, “sloppily”. All of the generated function code has the same prototype, so return-calls from one function to another *should* just optimize to jumps, and indeed they do: at `-O2`. For `-O0`, this doesn’t work, and sometimes you do want to compile in that mode (no optimizations) when investigating. I wish that C had a `musttail` attribute, and I use GCC too much to rely on [the one that only Clang
supports](https://clang.llvm.org/docs/AttributeReference.html#musttail).

I found the `-foptimize-sibling-calls` flag in GCC and somehow convinced myself that it was a sufficient replacement, even when otherwise at `-O0`. However most of my calls are indirect, and I think this must have fooled GCC. Really not sure. But I *was* sure, at one point, until a week later I realized that the reason that the `call` instruction was segfaulting was because it couldn’t store the return address in `*$rsp`, because I had blown the stack. That was embarrassing, but realizing that and trying again at -O2 didn’t fix my bug.

### the second side quest

Finally I recompiled `bdw-gc`, which I should have done from the beginning. (I use [Guix](https://guix.gnu.org/) on my test machine, and I wish I could have entered into an environment that had a specific set of dependencies, but with debugging symbols and source code; there are many things about Guix that I would think should help me develop software but which don’t. Probably there is a way to do this that I am unaware of.)

After recompiling, it became clear that BDW-GC was trying to scan a really weird range of addresses for roots, and accessing unmapped memory. You would think that this would be a common failure mode for BDW-GC, but really, this *never* happens: it’s an astonishingly reliable piece of software for what it does. I have used it for almost 20 years and its problems are elsewhere. Anyway, I determined that the thread that it was scanning was... well it was stuck somewhere in some `rr` support library, which... why was that anyway?

You see, the problem was that since my strange spooky segfaults on stack overflow, I had been living in [`rr`](https://rr-project.org/) for a week or so, because I needed to do hardware watchpoints and reverse-continue. But, somehow, strangely, oddly, *`rr` was corrupting* *my program*: it worked fine when not run in `rr`. Somehow `rr` caused GCC to grab the wrong `$rsp` on a remote thread.

### the actual bug

I had found the actual bug in the shower, some days before, and fixed it later that evening. Consider that both precise Immix and conservative Immix are working fine. Consider that conservative Immix is, well, stack-conservative just like BDW-GC. Clearly Immix finds the roots correctly, why didn’t BDW-GC? What is the real difference?

Well, friend, you have read the article title, so perhaps you won’t be surprised when I say “safepoints”. A safepoint is a potential stopping place in a program. At a safepoint, the overall state of the program can be inspected by some independent part of the program. In the case of garbage collection, safepoints are places where heap object references in a thread’s stack can be enumerated.

For my Immix-derived collectors, safepoints are cooperative: the only place a thread will stop for collection is in an allocation call, or if the thread has explicitly signalled to the collector that is doing a blocking operation such as a thread join. Whiffle usually keeps the stack pointer for the side value stack in a register, but for allocating operations that need to go through the slow path, it makes sure to [write that stack pointer into a shared data
structure](https://github.com/wingo/whiffle/blob/main/include/whiffle/vm.h#L178), so that other threads can read it if they need to walk its stack.

The BDW collector doesn’t work this way. It can’t rely on cooperation from the host. Instead, it installs a signal handler for the process that can suspend and resume any thread at any time. When BDW-GC needs to trace the heap, it stops all threads, finds the roots for those threads, and then traces the graph.

I had installed a custom [BDW-GC mark function for Whiffle
stacks](https://github.com/wingo/whiffle/blob/main/runtime/whiffle-gc.h#L172-L184) so that even though they were in anonymous mmap’d memory that BDW-GC doesn’t usually trace for roots, Whiffle would be sure to let BDW-GC know about them. That mark function used the safepoints that I was already tracking to determine which part of the side stack to mark.

But here’s the thing: *cooperative safepoints can be lazy, but* *preemptive safepoints must be eager*. For BDW-GC, it’s not enough to ensure that the thread’s stack is traversable at allocations: it must be traversable *at all times*, because a signal can stop a thread at any time.

Safepoints also differ on the axis of precision: precise safepoints must include no garbage roots, whereas conservative safepoints can be sloppy. For precise safepoints, if you bump the stack pointer to allocate a new frame, you can’t include the slots in that frame in the root set until they have values. Precision has a cost to the compiler and to the run-time in side table size or shared-data-structure update overhead. As far as I am aware, no production collector uses fully preemptive, precise roots. (Happy to be corrected, if people have counterexamples; I know that this used to be the case, but my understanding is that since multi-threaded mutators have been common, people have mostly backed away from this design.)

Concretely, as I was using a stack discipline to allocate temporaries, I had to [*shrink* the precise
safepoint](https://github.com/wingo/whiffle/blob/main/include/whiffle/vm.h#L121-L124) at allocation sites, but I was neglecting to [*expand* the
conservative safepoint](https://github.com/wingo/whiffle/blob/main/include/whiffle/vm.h#L126C24-L133) whenever the stack pointer would be restored.

### coda

When I was a kid, baseball was not my thing. I failed out at the earlier phase of tee-ball, where the ball isn’t thrown to you by the pitcher but is instead on a kind of rubber stand. As often as not I would fail to hit the ball and instead buckle the tee under it, making an unsatisfactory “flubt” sound, causing the ball to just fall to the ground. I probably would have persevered if I hadn’t also caught a ground ball to the face one day while playing right field, knocking me out and giving me a nosebleed so bad that apparently they called off the game because so many other players were vomiting. I woke up in the entryway of the YMCA and was given a lukewarm melted push-pop.

Anyway, “flubt” is the sound I heard in my mind when I realized that `rr` had been perturbing my use of BDW-GC, and instead of joy and relief when I finally realized that it worked already, it tasted like melted push-pop. It be that way sometimes. Better luck next try!

## related articles

- [conservative gc can be faster than precise gc](/archives/2024/09/07/conservative-gc-can-be-faster-than-precise-gc)
- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)
- [partitioning ambiguous edges in guile](/archives/2025/04/25/partitioning-ambiguous-edges-in-guile)

Comments are closed.
