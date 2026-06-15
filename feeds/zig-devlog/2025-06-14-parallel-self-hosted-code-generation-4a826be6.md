---
title: Parallel Self-Hosted Code Generation
url: https://ziglang.org/devlog/2025/#2025-06-14
published: "2025-06-14T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2025/#2025-06-14
---

# Parallel Self-Hosted Code Generation

June 14, 2025

# [Parallel Self-Hosted Code Generation](\#2025-06-14)

Author: Matthew Lugg

Less than a week ago, we finally turned on the x86\_64 backend by default for Debug builds on Linux and macOS. Today, we’ve got a big performance improvement to it: we’ve parallelized the compiler pipeline even more!

These benefits do not affect the LLVM backend, because it uses a lot more shared state; in fact, it is still limited to one thread, where every other backend was able to use two threads even before this change. But for the self-hosted backends, machine code generation is essentially an isolated task, so we can run it in parallel with everything else, and even run multiple code generation jobs in parallel with one another. The generated machine code then all gets glued together on the linker thread at the end. This means we end up with one thread performing semantic analysis, arbitrarily many threads performing code generation, and one thread performing linking. Parallelizing this phase is particularly beneficial because instruction selection for x86\_64 is incredibly complex due to the architecture’s huge variety of extensions and instructions.

This worked culminated in a [pull request](https://github.com/ziglang/zig/pull/24124), created by myself and Jacob, which was merged a couple of days ago. It was a fair amount of work, because a lot of internal details of the compiler pipeline needed reworking to completely isolate machine code generation from the linker. But it was all worth it in the end for the performance gains! Using the self-hosted x86\_64 backend, we saw anywhere from 5% through to 50% improvements in wall-clock time for compiling Zig projects. For example, Andrew reports being able to build the Zig compiler itself (excluding linking LLVM, which would add a couple of seconds to the time) in 10 seconds or less:

```
Benchmark 1 (32 runs): [... long command to build compiler with old compiler ...]
  measurement          mean ± σ            min … max           outliers         delta
  wall_time          13.8s  ± 71.4ms     13.7s … 13.8s           0 ( 0%)        0%
  peak_rss           1.08GB ± 18.3MB    1.06GB … 1.10GB          0 ( 0%)        0%
  cpu_cycles          109G  ± 71.2M      109G  …  109G           0 ( 0%)        0%
  instructions        240G  ± 48.3M      240G  …  240G           0 ( 0%)        0%
  cache_references   6.42G  ± 7.31M     6.41G  … 6.42G           0 ( 0%)        0%
  cache_misses        450M  ± 1.02M      449G  …  451G           0 ( 0%)        0%
  branch_misses       422M  ±  783K      421M  …  423M           0 ( 0%)        0%
Benchmark 2 (34 runs): [... long command to build compiler with new compiler ...]
  measurement          mean ± σ            min … max           outliers         delta
  wall_time         10.00s  ± 32.2ms     9.96s … 10.0s           0 ( 0%)        ⚡- 27.4% ±  0.9%
  peak_rss           1.35GB ± 18.6MB    1.34GB … 1.37GB          0 ( 0%)        💩+ 25.7% ±  3.9%
  cpu_cycles         95.1G  ±  371M     94.8G  … 95.5G           0 ( 0%)        ⚡- 12.8% ±  0.6%
  instructions        191G  ± 7.30M      191G  …  191G           0 ( 0%)        ⚡- 20.6% ±  0.0%
  cache_references   5.93G  ± 33.3M     5.90G  … 5.97G           0 ( 0%)        ⚡-  7.5% ±  0.9%
  cache_misses        417M  ± 4.55M      412M  …  421M           0 ( 0%)        ⚡-  7.2% ±  1.7%
  branch_misses       391M  ±  549K      391M  …  392M           0 ( 0%)        ⚡-  7.3% ±  0.4%

```

As another data point, I measure a 30% improvement in the time taken to build a simple “Hello World”:

```
Benchmark 1 (15 runs): /home/mlugg/zig/old-master/build/stage3/bin/zig build-exe hello.zig
  measurement          mean ± σ            min … max           outliers         delta
  wall_time           355ms ± 4.04ms     349ms …  361ms          0 ( 0%)        0%
  peak_rss            138MB ±  359KB     138MB …  139MB          0 ( 0%)        0%
  cpu_cycles         1.61G  ± 16.4M     1.59G  … 1.65G           0 ( 0%)        0%
  instructions       3.20G  ± 57.8K     3.20G  … 3.20G           0 ( 0%)        0%
  cache_references    113M  ±  450K      112M  …  113M           0 ( 0%)        0%
  cache_misses       10.5M  ±  122K     10.4M  … 10.8M           0 ( 0%)        0%
  branch_misses      9.73M  ± 39.2K     9.67M  … 9.79M           0 ( 0%)        0%
Benchmark 2 (21 runs): /home/mlugg/zig/master/build/stage3/bin/zig build-exe hello.zig
  measurement          mean ± σ            min … max           outliers         delta
  wall_time           244ms ± 4.35ms     236ms …  257ms          1 ( 5%)        ⚡- 31.5% ±  0.8%
  peak_rss            148MB ±  909KB     146MB …  149MB          2 (10%)        💩+  7.3% ±  0.4%
  cpu_cycles         1.47G  ± 12.5M     1.45G  … 1.49G           0 ( 0%)        ⚡-  8.7% ±  0.6%
  instructions       2.50G  ±  169K     2.50G  … 2.50G           1 ( 5%)        ⚡- 22.1% ±  0.0%
  cache_references    106M  ±  855K      105M  …  108M           1 ( 5%)        ⚡-  5.6% ±  0.4%
  cache_misses       9.67M  ±  145K     9.35M  … 10.0M           2 (10%)        ⚡-  8.3% ±  0.9%
  branch_misses      9.23M  ± 78.5K     9.09M  … 9.39M           0 ( 0%)        ⚡-  5.1% ±  0.5%

```

By the way, I’m a real sucker for some good `std.Progress` output, so I can’t help but mention how much I enjoy just *watching* the compiler now, and seeing all the work that it’s doing:

Even with these numbers, we’re still far from done in the area of compiler performance. Future improvements to our self-hosted linkers, as well as in the code which emits a function into the final binary, could help to speed up linking, which is now sometimes the bottleneck of compilation speed (you can actually see this bottleneck in the asciinema above). We also want to [improve the quality of the machine code we emit](https://github.com/ziglang/zig/issues/24144), which not only makes Debug binaries perform better, but (perhaps counterintutively) should further speed up linking. Other performance work on our radar includes decreasing the amount of work the compiler does at the very end of compilation (its “flush” phase) to eliminate another big chunk of overhead, and (in the more distant future) parallelizing semantic analysis.

Perhaps most significantly of all, incremental compilation – which has been a long-term investment of the Zig project for many years – is getting pretty close to being turned on by default in some cases, which will allow small changes to [rebuild in milliseconds](https://www.youtube.com/clip/Ugkxjn7L0hEfN1XLfH1soaUdCksG3FvJkXIS). By the way, remember that you can try out incremental compilation and start reaping its benefits *right now*, as long as you’re okay with possible compiler bugs! Check out [the tracking issue](https://github.com/ziglang/zig/issues/21165) if you want to learn more about that.

That’s enough rambling – I hope y’all are as excited about these improvements as we are. Zig’s compilation speed is the best it’s ever been, and hopefully the worst it’ll ever be again ;)
