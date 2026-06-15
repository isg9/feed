---
title: Who is waking the waker? (Linux chain graph prototype)
url: https://www.brendangregg.com/blog/2016-02-05/ebpf-chaingraph-prototype.html
published: "2016-02-05T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-02-05/ebpf-chaingraph-prototype.html
---

# Who is waking the waker? (Linux chain graph prototype)

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

## Who is waking the waker? (Linux chain graph prototype)

05 Feb 2016

[![](/blog/images/2016/offwake-sshd.png)](/blog/images/2016/offwake-sshd.png)

*Off-wake flame graph excerpt*

My previous post introduced off-wake time flame graphs\[1\], which marry off-CPU stack traces with wakeup stack traces. These provide better insight as to why threads have blocked. However, they don't solve everything.

An example from my previous post is on the right. This shows "sshd", with the off-CPU stack at the bottom and the waker stack on top ( [full SVG](/blog/images/2016/tar-offwaketime-flamegraph.svg)). So sshd is blocked on sys\_select(), which is woken up by kworker threads (kworker/u16:2) doing tty\_receive\_buf handling. Sounds like sshd is reading from a pipe. But who is at the other end of the pipe?

The problem becomes: who is waking the waker? And who is waking them?

Can't I trace *all* the waker stacks, and assemble the *entire chain*?

## Chain Graph Prototype

Chain graphs show why your threads have blocked, along with the entire chain of wakeup stacks that led to their resumption. So far they have been a fanciful idea that I introduced and prototyped for a USENIX LISA 2013 talk on flame graphs\[2\]\[3\]\[4\]. With Linux eBPF, we're getting closer to these becoming a practical tool. eBPF can trace with very low overhead, and store and retrieve stack traces in kernel probe context.

Here's an eBPF prototype which has two levels of waker stacks, and includes all threads ( [SVG](/blog/images/2016/chaingraph-2levels.svg)):

![](/blog/images/2016/chaingraph-2levels.svg)

Many of these are idle, waiting for work on a file descriptor via a sys\_poll() or sys\_select(), and woken up by a timer interrupt. The widths are the total time the off-CPU stack was blocked until its wakeup. Since this sums all time, it visualizes where time is spent on average, and identifies possible tuning targets.

[![](/blog/images/2016/chaingraph-2level-legend.png)](/blog/images/2016/chaingraph-2level-legend.png)

*Chain graph layout*

Try clicking on "sshd" (bottom to the left of supervise). Wow! "vmstat" woke up "kworker/u16:0", which woke up "sshd", and this was for 9.0 seconds in total. I had a "vmstat 1" running in one shell, and we're seeing 9 x 1 second updates during this 10 second profile. This layout is shown on the right.

All roads lead to metal.

You can imagine going further and showing all wakeup stacks, not just the first two. I'd prototyped this for my LISA talk, and found that all paths eventually lead to metal: hardware interrupts or timers wake up everything. The path from your application to metal explains why it blocked for so long.

The program I'm using, [chaintest](https://gist.github.com/brendangregg/c67039252268ec5e66ba), is a work in progress and not yet in bcc tools\[5\]. Perhaps by the time you are reading this it will be finished, and published as "chaingraph".

## Current Issues

You may not care much about the current issues right now, which will become out of date as eBPF and bcc are developed. For completion, here are the current issues and things that need work:

- I limited waker stacks to 7 frames each, as my stack trace hack\[6\] hits the MAX\_BPF\_STACK limit (512). We need eBPF stack support, which Alexei Starovoitov is working on (he's also now at Facebook).
- This prototype is kernel stacks only. eBPF stack support will give us the user-stacks too.
- It's currently only showing the last chain of wakeups. Needs to handle more complex chains.
- All threads are shown, which leads to some duplication. Eg, thread A shows A->B->C, and thread B shows (to some extent) B->C; we can probably exclude thread B.
- Cleanup per-thread timestamps and stacks, to avoid exhausting a BPF\_HASH table (10240 default).

The greatest challenge may be keeping the overhead of this tool low enough to be acceptable. eBPF helps as it has low overhead, and I am doing all in-kernel summaries, however, scheduler events can be very frequent. This area needs more work, testing, and quantification.

## Future

Imagine chain graphs: a visualization that explains not only why application threads blocked, but shows the full reason as to why they blocked for so long: the full wakeup chain. If we can use these in production, they should solve many system issues, and will be complementary to CPU flame graphs.

Chain graphs won't solve everything (latency outliers is one example of a performance issue not addressed by these...yet), but they will be a good start. It's been worth prototyping these on Linux now, while eBPF and bcc are still in development, to give us a use case to work towards.

## References & Links

1. [http://www.brendangregg.com/blog/2016-02-01/linux-wakeup-offwake-profiling.html](http://www.brendangregg.com/blog/2016-02-01/linux-wakeup-offwake-profiling.html)
2. [https://www.usenix.org/conference/lisa13/technical-sessions/plenary/gregg](https://www.usenix.org/conference/lisa13/technical-sessions/plenary/gregg)
3. [https://youtu.be/nZfNehCzGdw?t=1h21m48s](https://youtu.be/nZfNehCzGdw?t=1h21m48s)
4. [http://www.brendangregg.com/FlameGraphs/offcpuflamegraphs.html#ChainGraph](http://www.brendangregg.com/FlameGraphs/offcpuflamegraphs.html#ChainGraph)
5. [https://github.com/iovisor/bcc#tracing](https://github.com/iovisor/bcc#tracing)
6. [http://www.brendangregg.com/blog/2016-01-18/ebpf-stack-trace-hack.html](http://www.brendangregg.com/blog/2016-01-18/ebpf-stack-trace-hack.html)

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
