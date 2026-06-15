---
title: Java Warmup
url: https://www.brendangregg.com/blog/2016-09-28/java-warmup.html
published: "2016-09-28T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-09-28/java-warmup.html
---

# Java Warmup

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

## Java Warmup

28 Sep 2016

I gave a talk at JavaOne last week on flame graphs ( [slides here](http://www.slideshare.net/brendangregg/java-performance-analysis-on-linux-with-flame-graphs)), and in Q&A was asked about Java warmup. I wanted to show some flame graphs that illustrated it, but didn't have them on hand. Here they are.

These are 30 second flame graphs from a Java microservice warming up and beginning to process load. The following 30 flame graphs (3 rows of 10) have been shrunk too small to read the frames, but at this level you can see how processing changes in the first 10 minutes then settles down. Click to zoom (a little):

[![](/blog/images/2016/java_flamegraph_warmup_montage.png)](/blog/images/2016/java_flamegraph_warmup_montage.png)

The hues are: yellow is C++, green is Java methods, orange is kernel code, and red is other user-level code. I'm using the Linux perf profiler, which can see everything, including kernel code execution with Java context. JVM profilers (which are what almost everyone is using) cannot do this, and have blind spots.

Now I'll zoom in to the first two:

### Time = 0 - 30s

![](/blog/images/2016/java_flamegraph_warmup_000.svg)

This is the first 30 seconds of Java after launch.

On the left, the Java heap is being pre-allocated by the JVM code (C++, yellow). The os::pretouch\_memory() function is triggering page faults (kernel code, orange). Click Universe::initialize\_heap to get a breakdown of where time (CPU cycles) are spent. Neat, huh?

Because you're probably wondering, here's clear\_page\_c\_c() from arch/x86/lib/clear\_page\_64.S:

```
ENTRY(clear_page_c_e)
    movl $4096,%ecx
    xorl %eax,%eax
    rep stosb
    ret
ENDPROC(clear_page_c_e)

```

It uses REP and XORL instructions to wipe a page (4096 bytes) of memory. And that's a performance optimization added to Linux [by Intel](https://github.com/torvalds/linux/commit/e365c9df2f2f001450decf9512412d2d5bd1cdef#diff-e66daa94d10db3737c64b090770a16c2) (Linux gets such micro-optimizations frequently). Now zoom back out using "Reset Zoom" (top left) or click a lower frame.

The tower on the right has many "Interpreter" frames in red (user-mode): these are Java methods that likely haven't hit the CompileThreshold tunable yet (by default: 10000). If these methods keep being called, they'll switch from being interpreted (executed by the JVM's "Interpreter" function) to being complied and executed natively. At that point, the compiled methods will be running from their own addresses which can be profiled and translated: in the Java flame graph, they'll be colored green.

The tower in the middle around C2Compiler::compile\_method() shows the C2 compiler working on compiling Java just-in-time.

### Time = 30 - 60s

![](/blog/images/2016/java_flamegraph_warmup_001.svg)

We're done initializing the heap (that tower is now gone), but we're still compiling: about 55% of the CPU cycles are in the C2 compiler, and lots of Interpreter frame are still present. Click CompileBroker::compiler\_thread\_loop to zoom in to the compiler. I can't help but want to dig into these functions further, to look for optimizations in the compiler itself. Zooming back out...

There's a new tower, on the left, containing ParNewGenTask::work. Yes, GC has kicked in by the 30-60s profile. (Not all frames there are colored yellow, but they should be, it's just a bug in the flame graph software.)

See the orange frames in the top right? That's the kernel deleting files, called from java/io/UnixFileSystem:::delete0(). It's called so much that its CPU footprint is showing up in the profile. How interesting. So what's going on? The answer should be down the stack, but it's still Interpreter frames. I could do another warmup run and use my tracing tools ( [perf-tools](https://github.com/brendangregg/perf-tools) or [bcc](https://github.com/iovisor/bcc)). But right now I haven't done that so I don't know -- a mystery. I'd guess log rotation.

The frames with "\[perf-79379.map\]": these used to say "unknown", but a recent flame graph change defaults to the library name if the function symbol couldn't be found. In this case, it's not a library, but a map file which should have Java methods. So why weren't these symbols in it? The way I'm currently doing Java symbol translation is to take snapshots at the end of the profile, and it's possible those symbols were recompiled and moved by that point.

See the "Interpreter" tower on the far left? Notice how they don't begin with start\_thread(), java\_start(), etc? This is likely a truncated stack, due to the default setting of Linux perf to only capture the top 127 frames. Truncated stacks break flame graphs: it doesn't know where to merge them. This will be tunable in Linux 4.8 (kernel.perf\_event\_max\_stack). I'd bed this tower really belongs on as part of the tallest Interpreter tower on the right -- if you can imagine moving the frames over there and making it wider to fit.

### Time = 90+s

Going back to the montage, you can see different phases of warmup come and go, and different patterns of Java methods in green for each of them:

[![](/blog/images/2016/java_flamegraph_warmup_montage.png)](/blog/images/2016/java_flamegraph_warmup_montage.png)

I was hoping to document and include more of the warmup, as each stage can be clearly seen and inspected (and there are more interesting things to describe, including a lock contention issue in CFLS\_LAB::alloc() that showed up), but this post would get very long. I'll skip ahead to the warmed-up point:

### Time = 600 - 630s

[![](/blog/images/2016/java_flamegraph_warmup_020.png)](/blog/images/2016/java_flamegraph_warmup_020.png)

At the 10 minute mark, the microservice is running its production workload. Over time, the compiler tower shrinks further and we spend more time running Java code. (This image is a PNG, not a SVG.)

The small green frames on the left are more evidence of truncated stacks.

### Generation

To generate these flame graphs, I ran the following the same moment I began load:

```
for i in `seq 100 130`; do perf record -F 99 -a -g -- sleep 30; ./jmaps
    perf script > out.perf_warmup_$i.txt; echo $i done; done

```

The [jmaps](https://raw.githubusercontent.com/brendangregg/Misc/master/java/jmaps) program is an unsupported helper script for automating a symbol dump using Java perf-map-agent.

Then I post processed with the [FlameGraph](https://github.com/brendangregg/FlameGraph) software:

```
for i in `seq 100 130`; do ./FlameGraph/stackcollapse-perf.pl --kernel \
    < out.perf_warmup_$i.txt | grep '^java' | grep -v cpu_idle | \
    ./FlameGraph/flamegraph.pl --color=java --width=800 --minwidth=0.5 \
    --title="at $(((i - 100) * 30)) seconds" > warmup_$i.svg; echo $i done; done

```

This is using the older way of profiling with Linux perf: via a perf.data file and "perf script". A newer way has been in development that uses BPF for efficient in-kernel summaries, but that's a topic for another post...

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
