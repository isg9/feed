---
title: FlameScope Origin
url: https://www.brendangregg.com/blog/2018-12-15/flamescope-origin.html
published: "2018-12-15T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2018-12-15/flamescope-origin.html
---

# FlameScope Origin

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

## FlameScope Origin

15 Dec 2018

I've previously written about the [FlameScope](https://netflixtechblog.com/netflix-flamescope-a57ca19d47bb) tool and how it can help you identify [patterns](/blog/2018-11-08-flamescope-pattern-recognition.markdown) in profiles, especially CPU stack samples, and from those patterns generate flame graphs. I've also spoken about the origin of FlameScope in talks; here's a quick post to share the details here as well.

## 1\. Problem Statement

At Netflix my colleague Vadim was debugging an intermittent performance issue where application request latency increased briefly once every 15 minutes. It was a big issue. We picked one service in particular to analyze, "ums".

Here is how it looked in our cloud-wide monitoring tool, Atlas (redacted):

[![](/blog/images/2018/atlas-ums01red.png)](/blog/images/2018/atlas-ums01red.png)

The pillars show noticeable increases in latency.

We also looked at a number of other metrics to see if there was a correlation. CPU utilization did increase a little, however, the workload (as measured by the operation rate: NumCompleted) did not.

## 2\. Capturing a Sample

Vadim kept collecting 3-minute perf profiles while watching the Atlas latency metrics.

```
# perf record -F 49 -a -g -- sleep 180
# perf script > ums.stacks01

```

He finally saw the latency jump up while collecting a profile. We now had a 3-minute profile to study.

## 3\. Single CPU Runs

Latency popping up every 15 minutes or so sounds like garbage collection, but it didn't match Java GC activity.

To identify the problem, I wanted to analyze the `perf script` output, which is essentially a list of samples with their timestamps, metadata, and stack traces. I had been working on a [perf2runs](https://www.brendangregg.com/FlameGraphs/perf2runs-pl.txt) tool that would identify "single CPU runs," which I'd used to find issues before (such as a code cache one I discussed at the start of my [YOW!](https://www.youtube.com/watch?v=tAY8PnfrS_k) talk). But I kept thinking about other ways to slice and dice this output.

## 4\. Slicing and Dicing into Time Ranges

I came up with another simple tool, [range-perf.pl](https://github.com/brendangregg/FlameGraph/blob/master/range-perf.pl), which could filter time ranges from `perf script` output. I had built up a command line like this:

```
$ cat ums.stacks01 | ./range-perf.pl 0 10 | ./stackcollapse-perf.pl | grep -v cpu_idle | ./flamegraph.pl --hash --color=java > ums.first10.svg
```

I then did this in a loop and made separate flame graphs for every ten second range, hoping one would look different and explain the issue. It looked promising as it was starting to reveal some variation. I then tried a time range of one second, producing 180 flame graphs, and after browsing many of these we eventually found the issue: periodic application cache refreshes. But the whole process seemed time consuming, and had a granularity of one second. What if I needed 100 ms flame graphs to really focus on the perturbations? There would be way too many flame graphs to browse through (1,800).

## 5\. Visualizing CPU Utilization Down to 20 ms

We knew there was a slight correlation with CPU utilization, and in a way that information is captured in the perf profile (by filtering out idle samples). If we could visualize the CPU utilization in the profile over time, we could then select the most interesting ranges for a flame graph. I tried line graphs first, but ultimately found my [subsecond-offset heat map](/HeatMaps/subsecondoffset.html) visualization to be most useful as it revealed more patterns and easily visualized down to the 20 ms range. It looked like this ( [svg](/blog/images/2018/flamescope-heatmap.svg)):

[![](/blog/images/2018/flamescope-heatmap.svg)](/blog/images/2018/flamescope-heatmap.svg)

If you click this svg you should get some interactivity with this visualization, where you can hover over pixels and see time and sample counts. It's showing seconds as full columns, painted bottom to top, and fractions of seconds as the rows. The color depth shows how many CPU samples fell into each range. Each of these blocks represents about 20 milliseconds in time. (See [subsecond-offset heat map](/HeatMaps/subsecondoffset.html) for a greater explanation.)

There are three patterns in the above visualization:

[![](/blog/images/2018/flamescope-heatmap02.png)](/blog/images/2018/flamescope-heatmap02.png)

A) Three vertical bands of higher CPU usage spaced 60 seconds apart

B) Various short bursts of about 60 milliseconds of high CPU usage, appearing as short red stripes.

C) A burst of CPU usage at the start of the profile ( `perf` doing some initializing? Surely not).

## 6\. FlameScope

I enhanced this visualization so that we could click on time ranges and it would produce a flame graph just for that range. We called this tool "FlameScope" ( [my prototypes](https://github.com/brendangregg/FlameScope)). That way we quickly identified what all of these were:

A) Servo (Atlas monitoring)

B) Garbage collection

C) An application cache refresh

It was (C) we had found earlier the hard way by slicing and dicing the perf profile. It was far far easier to just "see it" in the heatmap and select that range. I used this same profile for many of other talks and blog posts about flame scope.

Martin Spier and myself (mostly Martin) then rewrote this into [Netflix FlameScope](https://netflixtechblog.com/netflix-flamescope-a57ca19d47bb), published on [github](https://github.com/Netflix/flamescope).

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
