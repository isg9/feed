---
title: No-Libc Zig Now Outperforms Glibc Zig
url: https://ziglang.org/devlog/2025/#2025-02-07
published: "2025-02-07T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2025/#2025-02-07
---

# No-Libc Zig Now Outperforms Glibc Zig

February 07, 2025

# [No-Libc Zig Now Outperforms Glibc Zig](\#2025-02-07)

Author: Andrew Kelley

Alright, I know I’m supposed to be focused on issue triage and merging PRs for the upcoming release this month, but in my defense, I do some of my best work while procrastinating.

Jokes aside, this week we had CI failures due to Zig’s debug allocator creating too many memory mappings. This was interfering with Jacob’s work on the x86 backend, so I spent the time to [rework the debug allocator](https://github.com/ziglang/zig/pull/20511#issuecomment-2638356298).

Since this was a chance to eliminate the dependency on a compile-time known page size, I based my work on contributor archbirdplus’s patch to add runtime-known page size support to the Zig standard library. With this change landed, it means Zig finally works on Asahi Linux. My fault for originally making page size compile-time known. Sorry about that!

Along with detecting page size at runtime, the new implementation no longer memsets each page to 0xaa bytes then back to 0x00 bytes, no longer searches when freeing, and no longer depends on a treap data structure. Instead, the allocation metadata is stored inline, on the page, using a pre-cached lookup table that is computed at compile-time:

```zig
/// This is executed only at compile-time to prepopulate a lookup table.
fn calculateSlotCount(size_class_index: usize) SlotIndex {
    const size_class = @as(usize, 1) << @as(Log2USize, @intCast(size_class_index));
    var lower: usize = 1 << minimum_slots_per_bucket_log2;
    var upper: usize = (page_size - bucketSize(lower)) / size_class;
    while (upper > lower) {
        const proposed: usize = lower + (upper - lower) / 2;
        if (proposed == lower) return lower;
        const slots_end = proposed * size_class;
        const header_begin = mem.alignForward(usize, slots_end, @alignOf(BucketHeader));
        const end = header_begin + bucketSize(proposed);
        if (end > page_size) {
            upper = proposed - 1;
        } else {
            lower = proposed;
        }
    }
    const slots_end = lower * size_class;
    const header_begin = mem.alignForward(usize, slots_end, @alignOf(BucketHeader));
    const end = header_begin + bucketSize(lower);
    assert(end <= page_size);
    return lower;
}

```

It’s pretty nice because you can tweak some global constants and then get optimal slot sizes. That assert at the end means if the constraints could not be satisfied you get a compile error. Meanwhile in C land, equivalent code has to resort to handcrafted lookup tables. Just look at the top of malloc.c from musl:

```c
const uint16_t size_classes[] = {
	1, 2, 3, 4, 5, 6, 7, 8,
	9, 10, 12, 15,
	18, 20, 25, 31,
	36, 42, 50, 63,
	72, 84, 102, 127,
	146, 170, 204, 255,
	292, 340, 409, 511,
	584, 682, 818, 1023,
	1169, 1364, 1637, 2047,
	2340, 2730, 3276, 4095,
	4680, 5460, 6552, 8191,
};

```

Not nearly as nice to experiment with different size classes. The water’s warm, Rich, come on in! 😛

Anyway, as a result of reworking this allocator, not only does it work with runtime-known page size, and avoid creating too many memory mappings, it also performs significantly better than before. The motivating test case for these changes was this degenerate ast-check task, with a debug compiler:

```
Benchmark 1 (3 runs): master/bin/zig ast-check ../lib/compiler_rt/udivmodti4_test.zig
  measurement          mean ± σ            min … max           outliers         delta
  wall_time          22.8s  ±  184ms    22.6s  … 22.9s           0 ( 0%)        0%
  peak_rss           58.6MB ± 77.5KB    58.5MB … 58.6MB          0 ( 0%)        0%
  cpu_cycles         38.1G  ± 84.7M     38.0G  … 38.2G           0 ( 0%)        0%
  instructions       27.7G  ± 16.6K     27.7G  … 27.7G           0 ( 0%)        0%
  cache_references   1.08G  ± 4.40M     1.07G  … 1.08G           0 ( 0%)        0%
  cache_misses       7.54M  ± 1.39M     6.51M  … 9.12M           0 ( 0%)        0%
  branch_misses       165M  ±  454K      165M  …  166M           0 ( 0%)        0%
Benchmark 2 (3 runs): branch/bin/zig ast-check ../lib/compiler_rt/udivmodti4_test.zig
  measurement          mean ± σ            min … max           outliers         delta
  wall_time          20.5s  ± 95.8ms    20.4s  … 20.6s           0 ( 0%)        ⚡- 10.1% ±  1.5%
  peak_rss           54.9MB ±  303KB    54.6MB … 55.1MB          0 ( 0%)        ⚡-  6.2% ±  0.9%
  cpu_cycles         34.8G  ± 85.2M     34.7G  … 34.9G           0 ( 0%)        ⚡-  8.6% ±  0.5%
  instructions       25.2G  ± 2.21M     25.2G  … 25.2G           0 ( 0%)        ⚡-  8.8% ±  0.0%
  cache_references   1.02G  ±  195M      902M  … 1.24G           0 ( 0%)          -  5.8% ± 29.0%
  cache_misses       4.57M  ±  934K     3.93M  … 5.64M           0 ( 0%)        ⚡- 39.4% ± 35.6%
  branch_misses       142M  ±  183K      142M  …  142M           0 ( 0%)        ⚡- 14.1% ±  0.5%

```

I didn’t stop there, however. Even though I had release tasks to get back to, this left me *itching* to make a fast allocator - one that was designed for multi-threaded applications built in ReleaseFast mode.

It’s a tricky problem. A fast allocator needs to avoid contention by storing thread-local state, however, it does not directly learn when a thread exits, so one thread must periodically attempt to reclaim another thread’s resources. There is also the producer-consumer pattern - one thread only allocates while one thread only frees. A naive implementation would never reclaim this memory.

Inspiration struck, and [200 lines of code later](https://github.com/ziglang/zig/blob/42dbd35d3e16247ee68d7e3ace0da3778a1f5d37/lib/std/heap/SmpAllocator.zig) I had a working implementation… after Jacob helped me find a couple logic bugs.

I created [Where in the World Did Carmen’s Memory Go?](https://github.com/andrewrk/CarmensPlayground) and used it to test a couple specific usage patterns. Idea here is to over time collect a robust test suite, do fuzzing, benchmarking, etc., to make it easier to try out new Allocator ideas in Zig.

After getting good scores on those contrived tests, I turned to the real world use cases of the Zig compiler itself. Since it can be built with and without libc, it’s a great way to test the performance delta between the two.

Here’s that same degenerate case above, but with a release build of the compiler - glibc zig vs no libc zig:

```
Benchmark 1 (32 runs): glibc/bin/zig ast-check ../lib/compiler_rt/udivmodti4_test.zig
  measurement          mean ± σ            min … max           outliers         delta
  wall_time           156ms ± 6.58ms     151ms …  173ms          4 (13%)        0%
  peak_rss           45.0MB ± 20.9KB    45.0MB … 45.1MB          1 ( 3%)        0%
  cpu_cycles          766M  ± 10.2M      754M  …  796M           0 ( 0%)        0%
  instructions       3.19G  ± 12.7      3.19G  … 3.19G           0 ( 0%)        0%
  cache_references   4.12M  ±  498K     3.88M  … 6.13M           3 ( 9%)        0%
  cache_misses        128K  ± 2.42K      125K  …  134K           0 ( 0%)        0%
  branch_misses      1.14M  ±  215K      925K  … 1.43M           0 ( 0%)        0%
Benchmark 2 (34 runs): SmpAllocator/bin/zig ast-check ../lib/compiler_rt/udivmodti4_test.zig
  measurement          mean ± σ            min … max           outliers         delta
  wall_time           149ms ± 1.87ms     146ms …  156ms          1 ( 3%)        ⚡-  4.9% ±  1.5%
  peak_rss           39.6MB ±  141KB    38.8MB … 39.6MB          2 ( 6%)        ⚡- 12.1% ±  0.1%
  cpu_cycles          750M  ± 3.77M      744M  …  756M           0 ( 0%)        ⚡-  2.1% ±  0.5%
  instructions       3.05G  ± 11.5      3.05G  … 3.05G           0 ( 0%)        ⚡-  4.5% ±  0.0%
  cache_references   2.94M  ± 99.2K     2.88M  … 3.36M           4 (12%)        ⚡- 28.7% ±  4.2%
  cache_misses       48.2K  ± 1.07K     45.6K  … 52.1K           2 ( 6%)        ⚡- 62.4% ±  0.7%
  branch_misses       890K  ± 28.8K      862K  … 1.02M           2 ( 6%)        ⚡- 21.8% ±  6.5%

```

Outperforming glibc!

And finally here’s the entire compiler building itself:

```
Benchmark 1 (3 runs): glibc/bin/zig build -Dno-lib -p trash
  measurement          mean ± σ            min … max           outliers         delta
  wall_time          12.2s  ± 99.4ms    12.1s  … 12.3s           0 ( 0%)        0%
  peak_rss            975MB ± 21.7MB     951MB …  993MB          0 ( 0%)        0%
  cpu_cycles         88.7G  ± 68.3M     88.7G  … 88.8G           0 ( 0%)        0%
  instructions        188G  ± 1.40M      188G  …  188G           0 ( 0%)        0%
  cache_references   5.88G  ± 33.2M     5.84G  … 5.90G           0 ( 0%)        0%
  cache_misses        383M  ± 2.26M      381M  …  385M           0 ( 0%)        0%
  branch_misses       368M  ± 1.77M      366M  …  369M           0 ( 0%)        0%
Benchmark 2 (3 runs): SmpAllocator/fast/bin/zig build -Dno-lib -p trash
  measurement          mean ± σ            min … max           outliers         delta
  wall_time          12.2s  ± 49.0ms    12.2s  … 12.3s           0 ( 0%)          +  0.0% ±  1.5%
  peak_rss            953MB ± 3.47MB     950MB …  957MB          0 ( 0%)          -  2.2% ±  3.6%
  cpu_cycles         88.4G  ±  165M     88.2G  … 88.6G           0 ( 0%)          -  0.4% ±  0.3%
  instructions        181G  ± 6.31M      181G  …  181G           0 ( 0%)        ⚡-  3.9% ±  0.0%
  cache_references   5.48G  ± 17.5M     5.46G  … 5.50G           0 ( 0%)        ⚡-  6.9% ±  1.0%
  cache_misses        386M  ± 1.85M      384M  …  388M           0 ( 0%)          +  0.6% ±  1.2%
  branch_misses       377M  ±  899K      377M  …  378M           0 ( 0%)        💩+  2.6% ±  0.9%

```

I feel that this is a key moment in the Zig project’s trajectory. [This last piece of the puzzle](https://github.com/ziglang/zig/pull/22808) marks the point at which the language and standard library has become *strictly better* to use than C and libc.

While other languages build on top of libc, Zig instead has conquered it!
