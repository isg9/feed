---
title: Self-Hosted x86 Backend is Now Default in Debug Mode
url: https://ziglang.org/devlog/2025/#2025-06-08
published: "2025-06-08T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2025/#2025-06-08
---

# Self-Hosted x86 Backend is Now Default in Debug Mode

June 08, 2025

# [Self-Hosted x86 Backend is Now Default in Debug Mode](\#2025-06-08)

Author: Andrew Kelley

Now, when you target x86\_64, by default, Zig will use its own x86 backend rather than using LLVM to lower a bitcode file to an object file.

The default is not changed on Windows yet, because more COFF linker work needs to be done first.

The x86 backend is now passing 1987 behavior tests, versus 1980 passed by the LLVM backend. In reality there are 2084 behavior tests, but the extra ones there are generally redundant with LLVM’s own test suite for its own x86 backend, so we only run those when testing with self-hosted x86. Anyway, my point is that Zig’s x86 backend is now *more robust* than its LLVM backend in terms of implementing the Zig language.

Why compete with LLVM on code generation? There are [a handful of reasons](https://ziggit.dev/t/can-someone-explain-why-zig-is-moving-away-from-llvm-but-in-simple-way/1226/6?u=andrewrk), but mainly, because we can dramatically outperform LLVM at compilation speed.

```
Benchmark 1 (6 runs): zig build-exe hello.zig -fllvm
  measurement          mean ± σ            min … max           outliers         delta
  wall_time           918ms ± 32.8ms     892ms …  984ms          0 ( 0%)        0%
  peak_rss            214MB ±  629KB     213MB …  215MB          0 ( 0%)        0%
  cpu_cycles         4.53G  ± 12.7M     4.52G  … 4.55G           0 ( 0%)        0%
  instructions       8.50G  ± 3.27M     8.50G  … 8.51G           0 ( 0%)        0%
  cache_references    356M  ± 1.52M      355M  …  359M           0 ( 0%)        0%
  cache_misses       75.6M  ±  290K     75.3M  … 76.1M           0 ( 0%)        0%
  branch_misses      42.5M  ± 49.2K     42.4M  … 42.5M           0 ( 0%)        0%
Benchmark 2 (19 runs): zig build-exe hello.zig
  measurement          mean ± σ            min … max           outliers         delta
  wall_time           275ms ± 4.94ms     268ms …  283ms          0 ( 0%)        ⚡- 70.1% ±  1.7%
  peak_rss            137MB ±  677KB     135MB …  138MB          0 ( 0%)        ⚡- 36.2% ±  0.3%
  cpu_cycles         1.57G  ± 9.60M     1.56G  … 1.59G           0 ( 0%)        ⚡- 65.2% ±  0.2%
  instructions       3.21G  ±  126K     3.21G  … 3.21G           1 ( 5%)        ⚡- 62.2% ±  0.0%
  cache_references    112M  ±  758K      110M  …  113M           0 ( 0%)        ⚡- 68.7% ±  0.3%
  cache_misses       10.5M  ±  102K     10.4M  … 10.8M           1 ( 5%)        ⚡- 86.1% ±  0.2%
  branch_misses      9.22M  ± 52.0K     9.14M  … 9.31M           0 ( 0%)        ⚡- 78.3% ±  0.1%

```

For a larger project like the Zig compiler itself, it takes the time down from 75 seconds to 20 seconds.

We’re *only just getting started*. We’ve already started work [fully parallelizing code generation](https://asciinema.org/a/722533). We’re also just a few linker enhancements and bug fixes away from making incremental compilation stable and robust in combination with this backend. There is still low hanging fruit for improving the generated x86 code quality. And we’re looking at aarch64 next - work that is expected to be accelerated thanks to our new Legalize pass.

The CI has finished building the respective commit, so you can try this out yourself by fetching the latest master branch build from [the download page](/download/).

Finally, here’s a gentle reminder that Zig Software Foundation is a 501(c)(3) non-profit that funds its development with donations from generous people like you. If you like what we’re doing, please [help keep us financially sustainable](/zsf/)!
