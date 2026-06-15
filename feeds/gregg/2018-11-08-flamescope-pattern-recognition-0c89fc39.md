---
title: FlameScope Pattern Recognition
url: https://www.brendangregg.com/blog/2018-11-08/flamescope-pattern-recognition.html
published: "2018-11-08T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2018-11-08/flamescope-pattern-recognition.html
---

# FlameScope Pattern Recognition

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

## FlameScope Pattern Recognition

08 Nov 2018

Flamescope is a new open source performance visualization tool that uses [subsecond offset heat maps](/HeatMaps/subsecondoffset.html) and [flame graphs](/flamegraphs.html) to analyze **periodic** activity, **variance**, and **perturbations**. We posted this on the Netflix TechBlog, [Netflix FlameScope](https://medium.com/netflix-techblog/netflix-flamescope-a57ca19d47bb), and the tool is on [github](https://github.com/Netflix/flamescope). While flame graphs are well understood, subsecond offset heat maps are not (they are another visualization I invented a while ago). FlameScope should help adoption.

In summary: the x-axis are columns of whole seconds, and the y-axis is the fraction within each second, grouped as buckets (boxes). The box color scales to show the number of events that happened at that second and subsecond: darker is more. Here's a real subsecond offset heat map of CPU samples:

[![](/blog/images/2018/flamescope-heatmap09-crop-annotated.png)](/blog/images/2018/flamescope-heatmap09-crop-annotated.png)

What can you identify in this? In this post I'll draw some synthetic subsecond offset heat maps for CPU samples, so I can separate out and show you patterns. In the real flame scope tool, these patterns can be selected and a flame graph generated, to show the responsible code paths (I'm not demonstrating flame graphs here).

## Periodic Activity

### 1\. One thread, once per second

![](/blog/images/2018/scaled_heatmap_flat1.png)

A thread wakes up at the same offset each second, does a few milliseconds of work, then goes back to sleep.

### 2\. One thread, twice per second

![](/blog/images/2018/scaled_heatmap_flat2.png)

The wakeups are exactly 500 ms apart. It could be two threads, but chances are it's one, given the 500 ms offset.

### 3\. Two threads

![](/blog/images/2018/scaled_heatmap_flat2b.png)

This looks like two threads waking up once per second.

### 4\. One busy-wait thread, once per second

![](/blog/images/2018/scaled_heatmap_busywait1.png)

This thread does about 20 ms of work, then sleeps for a whole second (1000 ms). It's a common pattern, and results in the wakeup offset creeping forward each second.

### 5\. One busy-wait thread, twice per second

![](/blog/images/2018/scaled_heatmap_busywait2.png)

The wakeups are now 500 ms apart. Chances are it's one thread, twice per second.

### 6\. One heavy-busy-wait thread

![](/blog/images/2018/scaled_heatmap_heavy1.png)

The slope is higher, as it's doing more work per second, looks like about 80 ms.

### 7\. One less-busy-wait thread

![](/blog/images/2018/scaled_heatmap_light1.png)

The slope is less, as it's doing less work per second, probably a few milliseconds.

### 8\. One busy-wait thread, once every five seconds

![](/blog/images/2018/scaled_heatmap_busywait5sec1.png)

Now it's only waking up once every 5 seconds.

If you're already wondering, I think we can calculate the CPU busy time on each wakeup based on the angle and the spacing between wakeups:

busy\_time = (1000ms / (heatmap\_rows x spacing)) x tan(angle)

So my earlier 45 degree line would be:

busy\_time = (1000ms / (50 x 1)) x tan(45) = 20ms

## Variance

### 9\. CPUs 100% utilized

![](/blog/images/2018/scaled_heatmap_pegged.png)

What it would look like if you maxed out all CPUs perfectly.

### 10\. CPUs 50% utilized

![](/blog/images/2018/scaled_heatmap_hurl90.png)

Real world workloads look more like this. It's composed of short requests, random arrivals.

### 11\. CPUs 25% utilized

![](/blog/images/2018/scaled_heatmap_hurl50.png)

Same workload at 25%.

### 12\. CPUs 5% utilized

![](/blog/images/2018/scaled_heatmap_hurl10.png)

Same workload at 5%.

### 13\. Load increasing

![](/blog/images/2018/scaled_heatmap_hurl50busier1.png)

Over the span of 2 minutes, the load has become heavier.

### 14\. Variable load

![](/blog/images/2018/scaled_heatmap_5secbusy1.png)

Every 30 seconds there is 5 seconds of heavier work.

## Perturbations

### 15\. CPU perturbations

![](/blog/images/2018/scaled_heatmap_perturb1.png)

Every so often, all CPUs max out for 100s of ms. (Eg, GC).

### 16\. CPU blocking

![](/blog/images/2018/scaled_heatmap_perturb2.png)

Every so often, all CPUs idle for 100s of ms. (Eg, I/O).

### 17\. Single-threaded blocking

![](/blog/images/2018/scaled_heatmap_perturb3.png)

Every so often, all CPUs idle except one (pink lines, not white) (Eg, global locks).

The last pattern is especially interesting: it happens when all threads are blocked on a lock, which is held by one thread that's running on-CPU. What's that thread doing? Just click the pink line in FlameScope and now you're looking at its flame graph. This makes analyzing a difficult performance problem trivial.

## Conclusion

Now what can you see?:

![](/blog/images/2018/flamescope-heatmap09-crop.png)

You now speak flame scope. In the real flame scope tool, you can select each of those patterns, and a flame graph will be shown that explains the code paths responsible.

Tonight I'm giving a talk at the LinkedIn performance meetup on FlameScope with my colleague Martin Spier, who has done most of the development work for the Netflix/FlameScope tool.

Good luck FlameScoping, and please share screenshots of the interesting patterns you encounter!

Brendan

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
