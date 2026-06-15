---
title: Java Flame Graphs
url: https://www.brendangregg.com/blog/2014-06-12/java-flame-graphs.html
published: "2014-06-12T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-06-12/java-flame-graphs.html
---

# Java Flame Graphs

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

## Java Flame Graphs

12 Jun 2014

Java flame graphs are a hot new way to visualize CPU usage. I'll show how to create them using free and open source tools: Google's [lightweight-java-profiler](https://code.google.com/p/lightweight-java-profiler/wiki/GettingStarted) (code.google.com) and my [flame graph software](https://github.com/brendangregg/FlameGraph) (github). Hopefully, one day, flame graphs will be a standard feature on all Java profilers, alongside the call tree graphs that are standard today.

**UPDATE (Aug 2015)**: See the [Java in Flames](http://techblog.netflix.com/2015/07/java-in-flames.html) Netflix Tech Blog post for the latest and best method for Java flame graphs, which uses Linux perf\_events to show Java and system code paths together. I also describe the steps in [Java CPU Flame Graphs](/FlameGraphs/cpuflamegraphs.html#Java). This blog post refers to a Java-only JVMTI-based method, that while works, has the caveats described in those links.

Flame Graphs show the big picture. Here is [vert.x](http://vertx.io) serving a simple JavaScript program:

![](/blog/images/2014/cpu-vertx-flamegraph.zng)

Awesome! Mouse over elements to see details. Here is the direct [SVG](/blog/images/2014/cpu-vertx-flamegraph.svg), and a non-interactive [PNG](/blog/images/2014/cpu-vertx-flamegraph.png) version.

The y-axis is stack depth, and the x-axis spans the sample population. Each rectangle is a stack frame. Color is not important, it's randomized to differentiate frames. The ordering from left to right is also unimportant.

You look for the widest frames, from bottom up, and forks in the "flames", which indicate different code paths taken. See my [CPU flame graphs](http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html) page for a longer explanation, and the [flame graphs presentation](http://www.brendangregg.com/flamegraphs.html#Presentation) from USENIX/LISA\`13. Once you get the hang of these, you can quickly identify and quantify CPU usage.

The large tower on the left is the Mozilla Rhino JavaScript engine, which is eating 42.70% CPU (look for the lowest mozilla frame; the percentage is inclusive of all child frames above it). vert.x with Java doesn't need to run this engine, freeing up those CPU resources, and roughly doubling maximum possible performance.

Other details are also interesting: the large plateau in write0(), at 31.99%, is desirable – that's vert.x responding to requests and doing work. To improve the performance further (aside from tuning write0()), examine and reduce time spent in other code paths. Eliminating Rhino provides the biggest win.

Collect flame graphs over different days or software versions, and you can also quickly quantify performance changes by comparing them.

## Profile Collection

These flame graphs visualize sampled stack traces that were running on-CPU. You can use any profiler that gives you full stack traces, provided the profiler is accurate. I first tried CPU sampling using hprof, but found that it had [issues](http://www.brendangregg.com/blog/2014-06-09/java-cpu-sampling-using-hprof.html).

For this example, I used the [lightweight-java-profiler](https://code.google.com/p/lightweight-java-profiler/wiki/GettingStarted) by Jeremy Manson. This is an open source demonstration of an accurate profiling technique, and not a point-and-click production-ready product, so some assembly (and caution) is required. My steps were:

### 1\. Get software

```
svn checkout http://lightweight-java-profiler.googlecode.com/svn/trunk/ lightweight-java-profiler-read-only
cd lightweight-java-profiler-read-only

```

### 2\. Customize Makefile

I edited the Makefile using vi, and set BITS to 64, and added an include path for my system. My changes looked like this (diff), yours may vary depending on include paths required:

```
4c4
< BITS?=32
---
> BITS?=64
49c49
< INCLUDES=-I$(JAVA_HOME)/$(HEADERS) -I$(JAVA_HOME)/$(HEADERS)/$(UNAME)
---
> INCLUDES=-I$(JAVA_HOME)/$(HEADERS) -I$(JAVA_HOME)/$(HEADERS)/$(UNAME) -I/usr/include/x86_64-linux-gnu

```

### 3\. Build software:

```
make all

```

### 4\. Set agent

I added the following option when running java (change path to match yours):

```
-agentpath:/usr/local/lightweight-java-profiler-read-only/build-64/liblagent.so

```

This samples when java starts until it exits, writing the report to a traces.txt file, which has a similar layout to hprof. The flame graph software will read this traces.txt file.

There are some overheads to have this running. For my program it was negligible (~1% loss in request rate, and a 7% increase in CPU consumption, for which headroom was available), but your mileage will vary. See the later Customizing the Profiler section for reducing the sampling rate if needed.

## Flame Graph

Now to turn traces.txt into a flame graph:

```
git clone http://github.com/brendangregg/FlameGraph
cd FlameGraph
./stackcollapse-ljp.awk < ../traces.txt | ./flamegraph.pl > ../traces.svg

```

stackcollapse-ljp.awk is a simple awk program to convert the output of lightweight-java-profiler into the stack trace format that flame graph reads (one stack per line). The flamegraph.pl program has various options (list using -h) to customize the output, including changing the title.

Now open the traces.svg file in a browser.

## Customizing the Profiler

It's worth mentioning that you can configure some options in lightweight-java-profiler by editing the source. See src/globals.h for:

```
// Number of times per second that we profile
static const int kNumInterrupts = 100;

// Maximum number of stack traces
static const int kMaxStackTraces = 3000;

// Maximum number of frames to store from the stack traces sampled.
static const int kMaxFramesToCapture = 128;

```

I may be inclined to reduce the sampling rate from 100 hertz to 50, or lower, if I'd like to collect data over a long run (minutes), to reduce the sampling overheads. I'd also increase the maximum stack frames, if my code exceeded 128. Flame graphs need full stacks to be drawn.

There's also Richard Warburton's [honest-profiler](https://github.com/RichardWarburton/honest-profiler), which builds upon the same accurate profiling technique. Like the lightweight-java-profiler, some assembly is required.

While these profilers see inside Java, my ideal profiler would include native user-level and kernel stacks. This would allow us to include JVM GC code paths, as well as other JVM internals, and kernel internals.

Flame graphs already have a history of proving their usefulness in other areas, including for kernel performance, Node.JS, Ruby, Perl, and others. See the [updates](http://www.brendangregg.com/flamegraphs.html#Updates) section on my [Flame Graphs](http://www.brendangregg.com/flamegraphs.html) page.

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
