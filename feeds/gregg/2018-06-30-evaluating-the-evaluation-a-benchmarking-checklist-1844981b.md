---
title: 'Evaluating the Evaluation: A Benchmarking Checklist'
url: https://www.brendangregg.com/blog/2018-06-30/benchmarking-checklist.html
published: "2018-06-30T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2018-06-30/benchmarking-checklist.html
---

# Evaluating the Evaluation: A Benchmarking Checklist

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

## Evaluating the Evaluation: A Benchmarking Checklist

30 Jun 2018

A co-worker introduced me to Craig Hanson and Pat Crain's performance mantras, which neatly summarize much of what we do in performance analysis and tuning. They are:

**Performance mantras**

1. Don't do it
2. Do it, but don't do it again
3. Do it less
4. Do it later
5. Do it when they're not looking
6. Do it concurrently
7. Do it cheaper

These have inspired me to summarize another performance activity: evaluating benchmark accuracy. Accurate benchmarking rewards engineering investment that actually improves performance, but, unfortunately, inaccurate benchmarking is more common. I have spent much of my career refuting bad benchmarks, and have developed such a knack for it that prior employers would not publish benchmarks unless they were approved by me. Because benchmarking is so important for our industry, I'd like to share with you how I do it.

Perhaps you're choosing products based on benchmark results, or building a product and publishing your own benchmarks. I recommend using the following benchmarking questions as a checklist:

**Benchmarking checklist**

1. Why not double?
2. Was it tuned?
3. Did it break limits?
4. Did it error?
5. Does it reproduce?
6. Does it matter?
7. Did it even happen?

In more detail:

### 1\. Why not double?

If the benchmark reported 20k ops/sec, you should ask: why not 40k ops/sec? This is really asking "what's the limiter?" but in a way that motivates people to answer it. "What's the limiter?" sounds like a homework exercise of purely academic value. Finding the answer to "Why not double?" might motivate everyone to find and fix the limiter, and double the benchmark numbers! Answering this usually requires analysis using other observability tools while the benchmark is running (what I call " [active benchmarking](/activebenchmarking.html)").

### 2\. Was it tuned?

Was the benchmark and the benchmark target fully tuned, to best resemble a production experience? Imagine some product has a new performance-improving capability that was not turned on for the benchmark. I've seen this happen because the new capability is unfamiliar (because it's new!) so staff don't realize its importance and don't use it. Or they didn't even hear about it. Or they did, but didn't find any documentation on how to turn it on. This ends up like testing a manual car's top speed in 1st gear only.

### 3\. Did it break limits?

This is a sanity test. I've refuted many benchmarks by showing that they would require a network throughput that would far exceed the maximum network bandwidth (off by, for example, as much as 10x!). Networks, PCIe busses, CPU interconnects, memory busses, and storage devices (both throughput and IOPS), all have fixed limits. Networking is the easiest to check. In some cases, a benchmark may appear to exceed network bandwidth because it returns from a local memory cache instead of the remote target. If you test data transfers of a known size, and show their rate, you can do a simple sum to see the minimum throughput required, then compare that to network links.

### 4\. Did it error?

What percentage of operations errored? 0%, 1%, 100%? Benchmarks may be misconfigured, or they may be stress testing and pushing targets to error-inducing limits. Errors perform differently from normal operations, so a high error rate will skew the benchmark. If you develop a habit of reading only the operation rate and latency numbers from a lengthy benchmark report (or you have a shell script to do this that feeds a GUI), it's easy to miss other details in the report such as the error rate.

### 5\. Does it reproduce?

If you run the benchmark ten times, how consistent are the results? There may be variance (e.g., due to caching or turbo boost) or perturbations (e.g., system cron tasks, GC) that skew a single benchmark result. One way to check on Linux for system perturbations is via the load averages: let them reach "0.00, 0.00, 0.00" before you begin benchmarking, and if they don't get to zero, debug and find out why.

### 6\. Does it matter?

If I showed that the wheel on one car would fly apart if spun at 7,000 mph, and the wheel on another car at 9,000 mph, how much would that affect your purchasing decision? Probable very little! But I see this all the time in tech: benchmarks that spin some software component at unrealistic rates, in order to claim a "win" over a competitor. Automated kitchen-sink benchmarks can also be full of these. These kinds of benchmarks are misleading. You need to ask yourself: how much does this benchmark matter in the real world?

### 7\. Did it even happen?

Once, during a proof of concept, a client reported that latency was unacceptably high for the benchmark: over one second for each request! My colleague began analysis using a packet sniffer on the target, to see if packets were really taking that long to be returned by the server. He saw nothing. No packets. A firewall was blocking everything – the reported latency was the time it took the client to time out! Beware of misconfigurations where the benchmark doesn't actually run, but the benchmark client thinks it did.

I hope you find this methodology useful, and feel free to forward this to anyone doing or evaluating benchmarks. I'll add this methodology to my online collection: [Performance Analysis Methodology](http://www.brendangregg.com/methodology.html).

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
