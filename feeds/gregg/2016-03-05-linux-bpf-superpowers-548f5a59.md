---
title: Linux BPF Superpowers
url: https://www.brendangregg.com/blog/2016-03-05/linux-bpf-superpowers.html
published: "2016-03-05T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-03-05/linux-bpf-superpowers.html
---

# Linux BPF Superpowers

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

## Linux BPF Superpowers

05 Mar 2016

Last month I spoke at Facebook's [Performance @Scale](https://performanceatscale.splashthat.com/) event about Linux BPF Superpowers. These are coming to Linux in the 4.x series, and I've been using them in new open source performance tools.

Video is on [Facebook](https://www.facebook.com/atscaleevents/videos/1693888610884236/) (30 mins):

> [Linux 4.x Performance: Using BPF Superpowers (Brendan Gregg)](https://www.facebook.com/atscaleevents/videos/1693888610884236/)
>
> Posted by [At Scale](https://www.facebook.com/atscaleevents/) on Friday, February 26, 2016

Slides are on [slideshare](http://www.slideshare.net/brendangregg/linux-bpf-superpowers) ( [PDF](/Slides/PerformanceAtScale2016_LinuxBPFSuperpowers.pdf)):

We've stopped calling it eBPF (extended Berkeley Packet Filter), and are now just calling it BPF, although we need a better backronym: Bytecode Probe Framework? Naming things is hard.

BPF is the in-kernel bytecode machine that can be used for tracing, virtual networks, and more. Alexei Starovoitov is the lead developer (he's now at Facebook), and there are developers from several companies contributing, including myself at Netflix, Daniel Borkmann at Cisco, and Brenden Blanco at PLUMgrid.

As an example of BPF, I opened with off-CPU analysis, and why BPF was making new things possible. I summarized some other examples as well, including gethostlatency, which instruments DNS lookups system wide without needing to restart anything:

```
# ./gethostlatency
TIME      PID    COMM          LATms HOST
06:10:24  28011  wget          90.00 www.iovisor.org
06:10:28  28127  wget           0.00 www.iovisor.org
06:10:41  28404  wget           9.00 www.netflix.com
06:10:48  28544  curl          35.00 www.netflix.com.au
06:11:10  29054  curl          31.00 www.plumgrid.com
06:11:16  29195  curl           3.00 www.facebook.com
06:11:25  29404  curl          72.00 foo

```

gethostlatency, and the other tools I demonstrated, are in [bcc tools](https://github.com/iovisor/bcc#tools), which is a Python front end for BPF. For this talk I created a diagram of all the bcc tracing tools so far:

[![](https://raw.githubusercontent.com/iovisor/bcc/master/images/bcc_tracing_tools_2016.png)](https://raw.githubusercontent.com/iovisor/bcc/master/images/bcc_tracing_tools_2016.png)

So many of my favourites (from other tracing languages) now have equivalents in bcc, which is pretty exciting. Tools like execsnoop, opensnoop, ext4slower, tcpretrans, tcpconnect, and runqlat.

These bcc tools are still in development and require at least Linux 4.1, which many people aren't running yet. You can think of them as a preview of things to come. But they are coming sooner rather than later: Ubuntu 16.04 (for example) will have a 4 series kernel, and isn't far away.

Please watch my talk video above, and check out the [other talk videos](https://code.facebook.com/posts/997355933686556/performance-scale-2016-recap/) which were pretty interesting as well (although that link plays the low-res versions in Chrome; high-res versions, like I linked to above, do exist).

Thanks to Facebook for having me – it was a great event.

## Links from the talk

iovisor bcc:

- [https://github.com/iovisor/bcc](https://github.com/iovisor/bcc)
- [http://www.brendangregg.com/blog/2015-09-22/bcc-linux-4.3-tracing.html](http://www.brendangregg.com/blog/2015-09-22/bcc-linux-4.3-tracing.html)
- [http://blogs.microsoft.co.il/sasha/2016/02/14/two-new-ebpf-tools-memleak-and-argdist/](http://blogs.microsoft.co.il/sasha/2016/02/14/two-new-ebpf-tools-memleak-and-argdist/)

BPF Off-CPU, Wakeup, Off-Wake & Chain Graphs:

- [http://www.brendangregg.com/blog/2016-01-20/ebpf-offcpu-flame-graph.html](http://www.brendangregg.com/blog/2016-01-20/ebpf-offcpu-flame-graph.html)
- [http://www.brendangregg.com/blog/2016-02-01/linux-wakeup-offwake-profiling.html](http://www.brendangregg.com/blog/2016-02-01/linux-wakeup-offwake-profiling.html)
- [http://www.brendangregg.com/blog/2016-02-05/ebpf-chaingraph-prototype.html](http://www.brendangregg.com/blog/2016-02-05/ebpf-chaingraph-prototype.html)

Linux Performance:

- [http://www.brendangregg.com/linuxperf.html](http://www.brendangregg.com/linuxperf.html)

Linux perf\_events:

- [https://perf.wiki.kernel.org/index.php/Main\_Page](https://perf.wiki.kernel.org/index.php/Main_Page)
- [http://www.brendangregg.com/perf.html](http://www.brendangregg.com/perf.html)

Flame Graphs:

- [http://techblog.netflix.com/2015/07/java-in-flames.html](http://techblog.netflix.com/2015/07/java-in-flames.html)
- [http://www.brendangregg.com/flamegraphs.html](http://www.brendangregg.com/flamegraphs.html)

Netflix Tech Blog on Vector:

- [http://techblog.netflix.com/2015/04/introducing-vector-netflixs-on-host.html](http://techblog.netflix.com/2015/04/introducing-vector-netflixs-on-host.html)

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
