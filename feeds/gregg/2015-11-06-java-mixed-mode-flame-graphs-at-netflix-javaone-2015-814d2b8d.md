---
title: Java Mixed-Mode Flame Graphs at Netflix, JavaOne 2015
url: https://www.brendangregg.com/blog/2015-11-06/java-mixed-mode-flame-graphs.html
published: "2015-11-06T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2015-11-06/java-mixed-mode-flame-graphs.html
---

# Java Mixed-Mode Flame Graphs at Netflix, JavaOne 2015

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

## Java Mixed-Mode Flame Graphs at Netflix, JavaOne 2015

06 Nov 2015

At JavaOne this year I gave a talk on Java mixed-mode flame graphs, which we are rolling out at Netflix for CPU analysis and more. These make use of a new feature in JDK8u60: -XX:+PreserveFramePointer, which allows system profilers, including Linux perf\_events, to capture stack traces.

The slides are on [slideshare](http://www.slideshare.net/brendangregg/javaone-2015-java-mixedmode-flame-graphs):

The talk was not recorded, however, low quality audio and video was captured, which is on [youtube](https://www.youtube.com/watch?v=BHA65BqlqSk) (the screen capture missed the demos, so they were taken from a handheld video):

These flame graphs can be generated entirely with open source software, and I included the instructions in the talk. As it requires a number of steps, you'll likely want to automate it so that it becomes push-button. We've been doing that for our open source [Vector](http://techblog.netflix.com/2015/04/introducing-vector-netflixs-on-host.html) instance analysis tool.

In this talk I introduced some new types of Java flame graphs, that visualize stack traces leading to:

- **Page faults**: To show what Java or JVM code triggered main memory (resident memory) to grow. This explains growth in the "RES" column you see in top(1).
- **Context switches**: To show why Java is blocking: what code paths lead to Java leaving the CPU. This can identify locks, I/O, sleeps, and other events. (Caveat: if it includes some random paths that shouldn't block, the reason may be kernel involuntary context switches due to CPU demand: however, you should already have identified those through basic CPU monitoring before using a context switch flame graph!).
- **Disk I/O requests**: To show Java and system code paths that lead to issuing a disk I/O request.
- **TCP events**: To show Java code paths that lead to initializing TCP sessions with connect() or accept(), or sending packets (receiving TCP packets happens asynchronously, so Java stack traces aren't present; they can be identified by moving up to the socket layer and syscalls if necessary).
- **CPU cache misses**: To show Java and JVM paths that lead to last level cache (LLC) misses, visualizing physical memory access by Java. Similar flame graphs can also be generated for memory stall cycles, helping the developers locate where to improve memory access (techniques for this were discussed in other sessions at JavaOne both this and last year).
- **CPI flame graph**: To show a CPU flame graph decorated with cycles-per-instruction as a color dimension. You might be more familiar with instructions-per-cycle (IPC), which is just the same metric inverted.

I was most excited to get the CPI flame graph working, which is the first time they've worked on Linux. An example flame graph is below (click to zoom, or view the [SVG](/blog/images/2015/cpi-flamegraph-java.svg)):

![](/blog/images/2015/cpi-flamegraph-java.svg)

Red paths are instruction heavy (low CPI), blue paths are stall cycle heavy (likely memory stalls; high CPI). If you zoom in on the Java code, you can find the function AbstractByteBuf:.forEachByteAsc0 is instruction heavy (shown as red), and many other paths, especially involving reading and writing memory, are more stall cycle heavy (shown as blue). I'd previously blogged about [CPI flame graphs on FreeBSD](http://www.brendangregg.com/blog/2014-10-31/cpi-flame-graphs.html), which we've used for analysis on the Netflix Open Connect Appliances (our CDN).

For more background on these new types of Java profiling, see my [Java in Flames](http://techblog.netflix.com/2015/07/java-in-flames.html) post on the Netflix Tech Blog. For flame graphs in general, see my [Flame Graphs](/flamegraphs.html) page and the [CPU Flame Graphs](/FlameGraphs/cpuflamegraphs.html) examples.

Thanks to the conference organizers for having me, and those that attended my talk. There was a lot of good technical content at JavaOne, and I'm still catching sessions I missed from the live stream recordings. Hopefully they will be eventually organized like the [JavaOne 2014 sessions](https://www.oracle.com/javaone/sessions.html), making them easy to browse and watch.

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
