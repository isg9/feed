---
title: 'EuroBSDcon: System Performance Analysis Methodologies'
url: https://www.brendangregg.com/blog/2017-10-28/bsd-performance-analysis-methodologies.html
published: "2017-10-28T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2017-10-28/bsd-performance-analysis-methodologies.html
---

# EuroBSDcon: System Performance Analysis Methodologies

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

## EuroBSDcon: System Performance Analysis Methodologies

28 Oct 2017

For my first trip to Paris I gave the closing keynote at [EuroBSDcon 2017](https://2017.eurobsdcon.org/about/) on performance methodologies, using FreeBSD 11.1 as an analysis target. In the past I've shared similar methodologies applied to other operating systems, and finished porting them to BSD for this talk. It was a few days of work, which is really not bad. That's a virtue of these methodologies: once you learn them, you can apply them to anything throughout your career, and it doesn't take too much time to re-apply them.

The video is on [youtube](https://www.youtube.com/watch?v=ay41Uq1DrvM):

Here are the [slides](/Slides/EuroBSDcon2017_SystemMethodology) or as a [PDF](/Slides/EuroBSDcon2017_SystemMethodology.pdf):

firstprevnextlast/permalink/zoom

FreeBSD has an excellent range of analysis tools, and this was an opportunity to show them off. Among the new content I developed for the talk was a **FreeBSD performance checklist**:

1. `uptime` → load averages
2. `dmesg -a | tail` → kernel errors
3. `vmstat 1` → overall stats by time
4. `vmstat -P` → CPU balance
5. `ps -auxw` → process usage
6. `iostat -xz 1` → disk I/O
7. `systat -ifstat` → network I/O
8. `systat -netstat` → TCP stats
9. `top` → process overview
10. `systat -vmstat` → system overview

I also developed a new tool to support my [thread state analysis](http://www.brendangregg.com/tsamethod.html) methodology on FreeBSD, [tstates.d](https://github.com/brendangregg/DTrace-tools/blob/master/sched/tstates.d):

```
# ./tstates.d
Tracing scheduler events... Ctrl-C to end.
^C
Time (ms) per state (read script for info):
COMM             PID       CPU  RUNQ    SLP    USL   SUS   SWP   LCK   IWT   YLD
irq15: ata1      12          0     0      0      0     0     0     0 15024     0
[...]
sleep            877         0     0    505      0     0     0     0     0     0
bufdaemon        19          0    11      0  15057     0     0     0     0     0
sleep            879         0     0   2614      0     0     0     0     0     0
devd             523         0     0  15024      0     0     0     0     0     0
syncer           21          1     9      0  15055     0     0     0     0     0
fsck_ufs         878         1     0      0     10     0     0     0     0     0
fsck             836         1     0     12      0     0     0     0     0     0
dd               883         2     0      0      0     0     0     0     0     0
bufspacedaemon   20          3     5      0  15019     0     0     0     0     0
dtrace           873         3    23  15980      0     0     0     0     0     0
sh               881         4     0      3      1     0     0     0     0     0
csh              865         5     7  13882      0     0     0     0     0     0
rand_harvestq    6           8    20      0  15846     0     0     0     0     0
kernel           0          29    15      0      0     0     0     0     0     0
cam              4          45    14      0      0     0     0     0     0     0
sshd             863        52    85  13757      0     0     0     0     0     0
intr             12         79   192      0      0     0     0     0     0     0
cksum            876      1591   177      0    234     0     0     0     0     0
idle             11      14114  1902      0      0     0     0     0     0     0

```

This tool breaks down thread time into different states by tracing scheduler events (which can have noticeable overhead: measure in a lab environment before use). The states are:

- **CPU**: on-CPU
- **RUNQ**: Waiting on a CPU run queue
- **SLP**: Interruptible sleep
- **USL**: Uninterruptible sleep (eg, disk I/O)
- **SUS**: Suspended
- **SWP**: Swapped
- **LCK**: Waiting for a lock
- **IWT**: Waiting for an interrupt
- **YLD**: Yield

I added USL since the talk to split out disk I/O from the sleep state. The output above includes a `sleep 0.5` command, and a `cksum`.

EuroBSDcon was a great conference, and I had a lot of fun catching up with the BSD folk and meeting new people. If you missed my talk, you can see it online above, and I hope you find it useful.

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
