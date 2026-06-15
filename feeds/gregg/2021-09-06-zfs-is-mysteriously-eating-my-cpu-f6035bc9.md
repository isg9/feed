---
title: ZFS Is Mysteriously Eating My CPU
url: https://www.brendangregg.com/blog/2021-09-06/zfs-is-mysteriously-eating-my-cpu.html
published: "2021-09-06T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2021-09-06/zfs-is-mysteriously-eating-my-cpu.html
---

# ZFS Is Mysteriously Eating My CPU

Brendan's site:

[Start Here](/overview.html)

[Homepage](/index.html)

[Blog](/blog/index.html)

[Sys Perf book](/systems-performance-2nd-edition-book.html)

[BPF Perf book](/bpf-performance-tools-book.html)

[Linux Perf](/linuxperf.html)

[eBPF Tools](/ebpf.html)

[perf Examples](/perf.html)

[Perf Methods](/methodology.html)

[USE Method](/usemethod.html)

[TSA Method](/tsamethod.html)

[Off-CPU Analysis](/offcpuanalysis.html)

[Active Bench.](/activebenchmarking.html)

[WSS Estimation](/wss.html)

[Flame Graphs](/flamegraphs.html)

[Flame Scope](/flamescope.html)

[Heat Maps](/heatmaps.html)

[Frequency Trails](/frequencytrails.html)

[Colony Graphs](/colonygraphs.html)

[DTrace Tools](/dtrace.html)

[DTraceToolkit](/dtracetoolkit.html)

[DtkshDemos](/dtkshdemos.html)

[Guessing Game](/guessinggame.html)

[Specials](/specials.html)

[Books](/books.html)

[Other Sites](/sites.html)

[![](/Images/sysperf2nd_bookcover_360.jpg)](/systems-performance-2nd-edition-book.html)

*[Systems Performance 2nd Ed.](/systems-performance-2nd-edition-book.html)*

[![](/Images/bpfperftools_bookcover_360.jpg)](/bpf-performance-tools-book.html)

*[BPF Performance Tools book](/bpf-performance-tools-book.html)*

Recent posts:

- 07 Feb 2026 »

[Why I joined OpenAI](/blog/2026-02-07/why-i-joined-openai.html)
- 05 Dec 2025 »

[Leaving Intel](/blog/2025-12-05/leaving-intel.html)
- 28 Nov 2025 »

[On "AI Brendans" or "Virtual Brendans"](/blog/2025-11-28/ai-virtual-brendans.html)
- 22 Nov 2025 »

[Intel is listening, don't waste your shot](/blog/2025-11-22/intel-is-listening.html)
- 17 Nov 2025 »

[Third Stage Engineering](/blog/2025-11-17/third-stage-engineering.html)
- 04 Aug 2025 »

[When to Hire a Computer Performance Engineering Team (2025) part 1 of 2](/blog/2025-08-04/when-to-hire-a-computer-performance-engineering-team-2025-part1.html)
- 22 May 2025 »

[3 Years of Extremely Remote Work](/blog/2025-05-22/3-years-of-extremely-remote-work.html)
- 01 May 2025 »

[Doom GPU Flame Graphs](/blog/2025-05-01/doom-gpu-flame-graphs.html)
- 29 Oct 2024 »

[AI Flame Graphs](/blog/2024-10-29/ai-flame-graphs.html)
- 22 Jul 2024 »

[No More Blue Fridays](/blog/2024-07-22/no-more-blue-fridays.html)
- 24 Mar 2024 »

[Linux Crisis Tools](/blog/2024-03-24/linux-crisis-tools.html)
- 17 Mar 2024 »

[The Return of the Frame Pointers](/blog/2024-03-17/the-return-of-the-frame-pointers.html)
- 10 Mar 2024 »

[eBPF Documentary](/blog/2024-03-10/ebpf-documentary.html)
- 28 Apr 2023 »

[eBPF Observability Tools Are Not Security Tools](/blog/2023-04-28/ebpf-security-issues.html)
- 01 Mar 2023 »

[USENIX SREcon APAC 2022: Computing Performance: What's on the Horizon](/blog/2023-03-01/computer-performance-future-2022.html)
- 17 Feb 2023 »

[USENIX SREcon APAC 2023: CFP](/blog/2023-02-17/srecon-apac-2023.html)
- 02 May 2022 »

[Brendan@Intel.com](/blog/2022-05-02/brendan-at-intel.html)
- 15 Apr 2022 »

[Netflix End of Series 1](/blog/2022-04-15/netflix-farewell-1.html)
- 09 Apr 2022 »

[TensorFlow Library Performance](/blog/2022-04-09/tensorflow-library-performance.html)
- 19 Mar 2022 »

[Why Don't You Use ...](/blog/2022-03-19/why-dont-you-use.html)

[Blog index](/blog/index.html)

[About](/blog/about.html)

[RSS](/blog/rss.xml)

# [Brendan Gregg's Blog](/blog/index.html)

[home](/blog/index.html)

## ZFS Is Mysteriously Eating My CPU

06 Sep 2021

A microservice team asked me for help with a mysterious issue. They claimed that the ZFS file system was consuming 30% of CPU capacity. I summarized this case study at [Kernel Recipes](https://youtu.be/UVM3WX8Lq2k?t=133) in 2017; it is an old story that's worth resharing here.

## 1\. Problem Statement

The microservice was for metrics ingestion and had recently updated their base OS image (BaseAMI). After doing so, they claimed that ZFS was now eating over 30% of CPU capacity. My first thought was that they were somehow mistaken: I worked on ZFS internals at Sun Microsystems, and unless it is badly misconfigured there's no way it can consume that much CPU.

I have been surprised many times by unexpected performance issues, so I thought I should check their instances anyway. At the very least, I could show that I took it seriously enough to check it myself. I should also be able to identify the real CPU consumer.

## 2\. Monitoring

I started with the cloud-wide monitoring tool, [Atlas](https://netflixtechblog.com/introducing-atlas-netflixs-primary-telemetry-platform-bd31f4d8ed9a), to check high-level CPU metrics. These included a breakdown of CPU time into percentages for "usr" (user: applications) and "sys" (system: the kernel).

I was surprised to find a whopping 38% of CPU time was in sys, which is highly unusual for cloud workloads at Netflix. This supported the claim that ZFS was eating CPU, but how? Surely this is some other kernel activity, and not ZFS.

## 3\. Next Steps

I'd usually SSH to instances for deeper analysis, where I could use mpstat(1) to confirm the usr/sys breakdown and perf(1) to begin profiling on-CPU kernel code paths. But since Netflix has tools (previously [Vector](https://netflixtechblog.com/introducing-vector-netflixs-on-host-performance-monitoring-tool-c0d3058c3f6f), now FlameCommander) that allow us to easily fetch flame graphs from our cloud deployment UI, I thought I'd jump to the chase. Just for illustration, this shows the Vector UI and a typical cloud flame graph:

[![](/blog/images/2021/vector2flamegraph.png)](/blog/images/2021/vector2flamegraph.png)

Note that this sample flame graph is dominated by Java, shown by the green frames.

## 4\. Flame Graph

Here's the CPU flame graph from one of the problem instances:

[![](/blog/images/2021/zfsflamegraph-crop-red.png)](/blog/images/2021/zfsflamegraph-red.svg)

The kernel CPU time pretty obvious, shown as two orange towers on the left and right. (The other colors are yellow for C++, and red for other user-level code.)

Zooming in to the left kernel tower:

[![](/blog/images/2021/zfsflamegraph-zoom-crop.png)](/blog/images/2021/zfsflamegraph-zoom.png)

This is arc\_reclaim\_thread! I worked on this code back at Sun. So this really is ZFS, they were right!

The ZFS Adapative Replacement Cache (ARC) is the main memory cache for the file system. The arc\_reclaim\_thread runs arc\_adjust() to evict memory from the cache to keep it from growing too large, and to maintain a threshold of free memory that applications can quickly use. It does this periodically or when woken up by low memory conditions. In the past I've seen the arc\_reclaim\_thread eat too much CPU when a tiny file system record size was used (e.g., 512 bytes) creating millions of tiny buffers. But that was basically a misconfiguration. The default size is 128 Kbytes, and people shouldn't be tuning below 8 Kbytes.

The right kernel tower enters spl\_kmem\_cache\_reap\_now(), another ZFS memory freeing function. I imagine this is related to the left tower (e.g., contending for the same locks).

But first: Why is ZFS in use?

## 5\. Configuration

There was only one use of ZFS so far at Netflix that I knew of: A new infrastructure project was using ZFS for containers. This led me to a theory: If they were quickly churning through containers, they would also be churning through ZFS file systems, and this might mean that many old pages needed to be cleaned out of the cache. Ahh, it makes sense now.

I told them my theory, confident I was on the right path. But they replied: "We aren't using containers." Ok, then how *are* you using ZFS? I did not expect their answer:

> "We aren't using ZFS."

What!? Yes you are, I can see the arc\_reclaim\_thread in the flame graph. It doesn't run for fun! It's only on CPU because it's evicting pages from the ZFS ARC. If you aren't using ZFS, there are no pages in the ARC, so it shouldn't run.

They were confident that they weren't using ZFS at all. The flame graph defied logic. I needed to prove to them that they were indeed using ZFS somehow, and figure out why.

## 6\. cd & ls

I should be able to debug this using nothing more than the cd and ls(1) commands. cd to the file system and ls(1) to see what's there. The file names should be a big clue as to its use.

First, finding out where the ZFS file systems are mounted:

```
df -h
mount
zfs list

```

This showed nothing! No ZFS file systems were currently mounted. I tried another instance and saw the same thing. Huh?

Ah, but containers may have been created previously and since destroyed, hence no remaining file systems now. How can I tell if ZFS has ever been used?

## 7\. arcstats

I know, arcstats! The kernel counters that track ZFS statistics, including ARC hits and misses. Viewing them:

```
# cat /proc/spl/kstat/zfs/arcstats
name                            type data
hits                            4    0
misses                          4    0
demand_data_hits                4    0
demand_data_misses              4    0
demand_metadata_hits            4    0
demand_metadata_misses          4    0
prefetch_data_hits              4    0
prefetch_data_misses            4    0
prefetch_metadata_hits          4    0
prefetch_metadata_misses        4    0
mru_hits                        4    0
mru_ghost_hits                  4    0
mfu_hits                        4    0
mfu_ghost_hits                  4    0
deleted                         4    0
mutex_miss                      4    0
evict_skip                      4    0
evict_not_enough                4    0
evict_l2_cached                 4    0
evict_l2_eligible               4    0
[...]

```

Unbelievable! All the counters were zero! ZFS really wasn't in use, ever! But at the same time, it was eating over 30% of CPU capacity! Whaaat??

The customer had been right all along. ZFS was straight up eating CPU, and for no reason.

How can a file system *that's not in use at all* consume 38% CPU? I'd never seen this before. This was a mystery.

## 8\. Code Analysis

I took a closer look at the flame graph and the paths involved, and saw that the CPU code paths led to get\_random\_bytes() and extract\_entropy(). These were new to me. Browsing the [source code](https://github.com/openzfs/zfs/blob/4e33ba4c389f59b74138bf7130e924a4230d64e9/module/zfs/arc.c) and change history I found the culprit.

The ARC contains lists of cached buffers for different memory types. A performance feature ("multilist") had been added that split the ARC lists into one per CPU, to reduce lock contention on multi-CPU systems. Sounds good, as that should improve performance. But what happens when you want to evict some memory? You need to pick one of those CPU lists. Which one? You could go through them in a round-robin fashion, but the developer thought it better to pick one at random.

*Cryptographically-secure random.*

The kicker was that ZFS wasn't even in use. The ARC was detecting low system memory and then adjusting its size accordingly, at which point it'd discover it was zero size already and didn't need to do anything. But this was after randomly selecting a zero-sized list, using a CPU-expensive random number generator.

I filed this as ZFS [issue #6531](https://github.com/openzfs/zfs/issues/6531). I believe the first fix was to have the arc\_reclaim\_thread bail out earlier if ZFS wasn't in use, and not enter list selection. The ARC has since had many changes, and I haven't heard of this issue again.

---

Click here for Disqus comments (ad supported).

*You are welcome to comment here, but I've been meaning to switch comment systems one day and I don't know yet if I can preserve existing comments (I'll try to find a way).*

[comments powered by Disqus](http://disqus.com)

---

Site Navigation

[![](/Images/sysperf2nd_bookcover_360.jpg)](/systems-performance-2nd-edition-book.html)

*[Systems Performance 2nd Ed.](/systems-performance-2nd-edition-book.html)*

[![](/Images/bpfperftools_bookcover_360.jpg)](/bpf-performance-tools-book.html)

*[BPF Performance Tools book](/bpf-performance-tools-book.html)*

Recent posts:

- 07 Feb 2026 »

[Why I joined OpenAI](/blog/2026-02-07/why-i-joined-openai.html)
- 05 Dec 2025 »

[Leaving Intel](/blog/2025-12-05/leaving-intel.html)
- 28 Nov 2025 »

[On "AI Brendans" or "Virtual Brendans"](/blog/2025-11-28/ai-virtual-brendans.html)
- 22 Nov 2025 »

[Intel is listening, don't waste your shot](/blog/2025-11-22/intel-is-listening.html)
- 17 Nov 2025 »

[Third Stage Engineering](/blog/2025-11-17/third-stage-engineering.html)
- 04 Aug 2025 »

[When to Hire a Computer Performance Engineering Team (2025) part 1 of 2](/blog/2025-08-04/when-to-hire-a-computer-performance-engineering-team-2025-part1.html)
- 22 May 2025 »

[3 Years of Extremely Remote Work](/blog/2025-05-22/3-years-of-extremely-remote-work.html)
- 01 May 2025 »

[Doom GPU Flame Graphs](/blog/2025-05-01/doom-gpu-flame-graphs.html)
- 29 Oct 2024 »

[AI Flame Graphs](/blog/2024-10-29/ai-flame-graphs.html)
- 22 Jul 2024 »

[No More Blue Fridays](/blog/2024-07-22/no-more-blue-fridays.html)
- 24 Mar 2024 »

[Linux Crisis Tools](/blog/2024-03-24/linux-crisis-tools.html)
- 17 Mar 2024 »

[The Return of the Frame Pointers](/blog/2024-03-17/the-return-of-the-frame-pointers.html)
- 10 Mar 2024 »

[eBPF Documentary](/blog/2024-03-10/ebpf-documentary.html)
- 28 Apr 2023 »

[eBPF Observability Tools Are Not Security Tools](/blog/2023-04-28/ebpf-security-issues.html)
- 01 Mar 2023 »

[USENIX SREcon APAC 2022: Computing Performance: What's on the Horizon](/blog/2023-03-01/computer-performance-future-2022.html)
- 17 Feb 2023 »

[USENIX SREcon APAC 2023: CFP](/blog/2023-02-17/srecon-apac-2023.html)
- 02 May 2022 »

[Brendan@Intel.com](/blog/2022-05-02/brendan-at-intel.html)
- 15 Apr 2022 »

[Netflix End of Series 1](/blog/2022-04-15/netflix-farewell-1.html)
- 09 Apr 2022 »

[TensorFlow Library Performance](/blog/2022-04-09/tensorflow-library-performance.html)
- 19 Mar 2022 »

[Why Don't You Use ...](/blog/2022-03-19/why-dont-you-use.html)

[Blog index](/blog/index.html)

[About](/blog/about.html)

[RSS](/blog/rss.xml)

---

Brendan's site:

[Start Here](/overview.html)

[Homepage](/index.html)

[Blog](/blog/index.html)

[Sys Perf book](/systems-performance-2nd-edition-book.html)

[BPF Perf book](/bpf-performance-tools-book.html)

[Linux Perf](/linuxperf.html)

[eBPF Tools](/ebpf.html)

[perf Examples](/perf.html)

[Perf Methods](/methodology.html)

[USE Method](/usemethod.html)

[TSA Method](/tsamethod.html)

[Off-CPU Analysis](/offcpuanalysis.html)

[Active Bench.](/activebenchmarking.html)

[WSS Estimation](/wss.html)

[Flame Graphs](/flamegraphs.html)

[Flame Scope](/flamescope.html)

[Heat Maps](/heatmaps.html)

[Frequency Trails](/frequencytrails.html)

[Colony Graphs](/colonygraphs.html)

[DTrace Tools](/dtrace.html)

[DTraceToolkit](/dtracetoolkit.html)

[DtkshDemos](/dtkshdemos.html)

[Guessing Game](/guessinggame.html)

[Specials](/specials.html)

[Books](/books.html)

[Other Sites](/sites.html)

Copyright 2025 Brendan Gregg.

[About this blog](/blog/about.html)
