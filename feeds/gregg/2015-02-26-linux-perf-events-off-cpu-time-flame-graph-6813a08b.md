---
title: Linux perf_events Off-CPU Time Flame Graph
url: https://www.brendangregg.com/blog/2015-02-26/linux-perf-off-cpu-flame-graph.html
published: "2015-02-26T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2015-02-26/linux-perf-off-cpu-flame-graph.html
---

# Linux perf_events Off-CPU Time Flame Graph

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

## Linux perf\_events Off-CPU Time Flame Graph

26 Feb 2015

I've been asked how to do these several times, so here's a quick blog post.

[CPU Flame Graphs](http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html) are great for understanding CPU usage, but what about performance issues of latency, when programs are blocked and not running on-CPU? There's a generic methodology to attack these, which I've called [Off-CPU Analysis](http://www.brendangregg.com/offcpuanalysis.html). This involves instrumenting call stacks and timestamps when the scheduler blocks a thread, and when it later wakes it up. There are even [Off-CPU Time Flame Graphs](http://www.brendangregg.com/FlameGraphs/offcpuflamegraphs.html) to visualize this blocked time, which are the counterpart to on-CPU flame graphs.

Off-CPU time flame graphs may solve (say) 60% of the issues, with the remainder requiring walking the thread wakeups to find root cause. I explained off-CPU time flame graphs, this wakeup issue, and additional work, in my LISA13 talk on flame graphs ( [slides](http://www.slideshare.net/brendangregg/blazing-performance-with-flame-graphs/122), [youtube](http://youtu.be/nZfNehCzGdw?t=1h11m52s)).

Here I'll show one way to do off-CPU time flame graphs using Linux perf\_events. Example (click to zoom):

![](/blog/images/2015/offcpu-perf.svg)

Unlike the CPU flame graph, in this graph the width spans the total duration that a code path was sleeping. A "sleep 1" command was caught, shown on the far right as having slept for 1,000 ms.

I was discussing how to do these with Nicolas on [github](https://github.com/brendangregg/FlameGraph/issues/47#issuecomment-76298637), as he had found that perf\_events could almost do this using the `perf inject -s` feature, when I noticed perf\_events has added "-f period" to perf script (added in about 3.17). That simplifies things a bit, so the steps can be:

```
# perf record -e sched:sched_stat_sleep -e sched:sched_switch \
    -e sched:sched_process_exit -a -g -o perf.data.raw sleep 1
# perf inject -v -s -i perf.data.raw -o perf.data
# perf script -f comm,pid,tid,cpu,time,period,event,ip,sym,dso,trace | awk '
    NF > 4 { exec = $1; period_ms = int($5 / 1000000) }
    NF > 1 && NF <= 4 && period_ms > 0 { print $2 }
    NF < 2 && period_ms > 0 { printf "%s\n%d\n\n", exec, period_ms }' | \
    ./stackcollapse.pl | \
    ./flamegraph.pl --countname=ms --title="Off-CPU Time Flame Graph" --colors=io > offcpu.svg

```

Update: on more recent kernels, use "perf script -F ..." instead of "perf script -f ...". Your kernel will also need CONFIG\_SCHEDSTATS for the tracepoints to all be present, which may be missing (eg, RHEL7).

The awk I'm using merely turns the perf script output into something stackcollapse.pl understands. This whole perf inject workflow strikes me as a bit weird, and this step could be skipped by doing your own processing of the perf script output (more awk!), to stitch together events. I'd do this if I were on older kernels, that lacked the perf script -f period.

**Warning**: scheduler events can be very frequent, and the overhead of dumping them to the file system (perf.data) may be prohibitive in production environments. Test carefully. This is also why I put a "sleep 1" in the perf record (the dummy command that sets the duration), to start with a small amount of trace data. If I had to do this in production, I'd consider other tools that could summarize data in-kernel to reduce overhead, including perf\_events once it supports more in-kernel programming (eBPF).

There may be some scenarios where the current perf\_events overhead is acceptable, especially if you match on a single PID of interest (in perf record, using "-p PID" instead of "-a").

**Update 1**: Since Linux 4.5 you may need the following for this to work:

```
# echo 1 > /proc/sys/kernel/sched_schedstats
```

**Update 2**: Since Linux 4.6 you can use BPF to do this *much* more efficiently: aggregating the stacks in-kernel context (using the BPF stack trace feature in 4.6), and only passing the summary to user-level. I developed the offcputime tool [bcc](https://github.com/iovisor/bcc) to do this. I also wrote a post about it, [Off-CPU eBPF Flame Graph](http://www.brendangregg.com/blog/2016-01-20/ebpf-offcpu-flame-graph.html), although that was written before Linux 4.6's stack trace support, so I was unwinding stacks the hard way.

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
