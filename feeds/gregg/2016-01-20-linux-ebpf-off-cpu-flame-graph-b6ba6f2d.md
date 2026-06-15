---
title: Linux eBPF Off-CPU Flame Graph
url: https://www.brendangregg.com/blog/2016-01-20/ebpf-offcpu-flame-graph.html
published: "2016-01-20T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-01-20/ebpf-offcpu-flame-graph.html
---

# Linux eBPF Off-CPU Flame Graph

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

## Linux eBPF Off-CPU Flame Graph

20 Jan 2016

CPU profiles, which can be visualized as a [CPU flame graph](http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html), can solve a wide range of CPU usage issues. Off-CPU analysis, which can be visualized as [Off-CPU time flame graphs](http://www.brendangregg.com/FlameGraphs/offcpuflamegraphs.html), can begin to solve the rest. As I recently hacked [stack traces for eBPF](/blog/2016-01-18/ebpf-stack-trace-hack.html), I can now start to explore off-CPU profiling using core Linux capabilities.

## Off-CPU Time Flame Graph

This is for a Linux kernel build, followed by a CPU profile flame graph:

![](/blog/images/2016/off-cpu-flame-graph-build.svg)

![](/blog/images/2016/system-cpu-flame-graph-build.svg)

Click to zoom, or browse the [Off-CPU](/blog/images/2016/off-cpu-flame-graph-build.svg) and [CPU](/blog/images/2016/system-cpu-flame-graph-build.svg) flame graph SVGs directly. To keep these examples simple, I'm only showing kernel time.

Seeing both on-CPU and off-CPU flame graphs shows you the full picture: what's consuming CPUs, what's blocked, and by how much. The x-axis for the CPU flame graph shows the sample population. The x-axis for the off-CPU time flame graph shows the time threads were blocked.

This example off-CPU time flame graph has the thread names at the bottom, then the stack traces. During the Linux kernel compile, we can see "as" and "make" were blocked in vfs\_read(), likely reading from the filesystem. "gcc" and "sh" were waiting on child processes, as we'd expect. Plus there are many other threads waiting for work in poll routines. Since many threads can be blocked in parallel, they can add up to hundreds of seconds of blocked time for this 30 second profile.

## Why eBPF Is So Important

Off-CPU profiling has been a prickly problem that can be explained with some math:

- A typical CPU profile using a system profiler may involve 99 stack trace samples per second, on all CPUs. If this were a 16 CPU instance, that's 1,584 samples per second. Linux [perf\_events](http://www.brendangregg.com/perf.html) writes every sample to a perf.data file (in groups for efficiency), and usually handles this rate with low overhead.
- An off-CPU profile may involve tracing the scheduler task switch routine, and recording timestamps, calculating off-CPU time, and saving this with stack traces. Scheduler events scale with load, so instead of maybe 1,584 stack samples per second, this can be over 100,000 or 1 million per second. Overhead can become a big problem fast, which can make this prohibitive for production use.

This is where the new eBPF functionality comes in: my offcputime script uses eBPF to sum off-CPU time by kernel stack trace in kernel context for efficiency. Only the summary is passed to user-level.

Dumping every event to a file (eg, perf\_event's perf.data) will become problematic before it reaches 100,000 events per second, as it will likely cost noticeable CPU resources and start dropping events. By summarizing in kernel using eBPF, we can keep going, making this and a whole lot more practical. There's still overhead, and you should test and quantify before production use, but it should be the least possible.

With my [eBPF stack hack](/blog/2016-01-18/ebpf-stack-trace-hack.html), I'm able to try out off-CPU flame graphs now, at least for kernel stacks, on x86\_64, and up to 20 frames deep. In a future Linux version (4.5?), eBPF stack tracing should be supported properly, and not have these limitations.

## offcputime

The off-CPU flame graph was generated using offcputime from [bcc tools](https://github.com/iovisor/bcc#tracing). It can emit folded output suitable for flame graph generation. Eg, for a 30 second profile:

```
# ./offcputime -f 30 > out.offcpustacks01

```

A -u option will print only user-mode stacks. For example, this off-CPU flame graph is for an idle system, where I ran "sleep 3" in one window, and "vmstat 1" in another [SVG](/blog/images/2016/off-cpu-flame-graph-idle.svg):

![](/blog/images/2016/off-cpu-flame-graph-idle.svg)

Can you find the sleep command? (Try the Search button in the top right.) The blocked time for "vmstat 1" can be seen on the right, which adds to 5 seconds (I stopped the profile between 5 and 6 seconds). Both sleep and vmstat are blocked on a timer, which makes sense.

The commands I used to create this were:

```
# ./offcputime -uf > out.offcpustacks02
^C
# ./flamegraph.pl --color=io --countname=us --width=900 \
    --title="Off-CPU Time Flame Graph: idle system" < out.offcpustacks02 > offcpu.svg

```

flamegraph.pl is from [FlameGraph](https://github.com/brendangregg/FlameGraph).

## Other Tracers

I previously posted about using [perf\_events for off-CPU flame graphs](/blog/2015-02-26/linux-perf-off-cpu-flame-graph.html), along with a warning about the prohibitive overhead, and a suggestion that this should be done using eBPF.

DTrace and SystemTap can also summarize in kernel, and can be used for off-CPU analysis. Yichun Zhang first published off-CPU time flame graphs in his [Introduction to off-CPU Time Flame Graphs](http://agentzh.org/misc/slides/off-cpu-flame-graphs.pdf) (PDF) talk, and has the SystemTap scripts in his [nginx-systemtap-toolkit](https://github.com/openresty/nginx-systemtap-toolkit). Frits Hoogland more recently published [stapflame](https://github.com/FritsHoogland/stackflame) as a proof of concept, which uses perf\_events for CPU profiling and SystemTap for off-CPU time with Oracle context.

## Future Work

eBPF needs stack trace support, for both kernel and user stacks, for generating complete off-CPU time flame graphs. Perhaps by the time you're reading this, it already exists (check the implementation of [offcputime](https://github.com/iovisor/bcc/blob/master/tools/offcputime)). Other eBPF features will help as well: once profiling (timed sampling) is available, eBPF could do in kernel aggregations of the CPU profile as well. It'll be great to have this in the core Linux kernel.

**Update 2017: eBPF stack support arrived in Linux 4.8, and offcputime in bcc has been updated to use it.**

More information about blocked stacks should also be included. Sometimes the off-CPU stack is a sufficient clue as to the source of latency, but in many cases it isn't: you're blocked on a file descriptor (sys\_poll), a mutex, or a conditional variable. I'll explore this in an upcoming post.

Another area of future work is how to best show CPU and off-CPU flame graphs together. See my [hot/cold flame graphs](/FlameGraphs/hotcoldflamegraphs.html) page for a summary of this work.

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
