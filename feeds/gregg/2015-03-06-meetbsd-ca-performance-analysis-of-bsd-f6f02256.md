---
title: 'MeetBSD CA: Performance Analysis of BSD'
url: https://www.brendangregg.com/blog/2015-03-06/performance-analysis-bsd.html
published: "2015-03-06T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2015-03-06/performance-analysis-bsd.html
---

# MeetBSD CA: Performance Analysis of BSD

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

## MeetBSD CA: Performance Analysis of BSD

06 Mar 2015

System performance analysis is an enormous topic, and it's a challenge to know what can be analyzed and how. At the last [MeetBSD CA](https://www.meetbsd.com/), I gave a talk titled "Performance Analysis" where I discussed five key facets of this topic: observability tools, methodologies, benchmarking, profiling, and tracing. I'd recommend this talk for anyone working in the BSD family of operating systems.

For the talk I created an observability tools diagram for FreeBSD:

[![](/Perf/freebsd_observability_tools.png)](/Perf/freebsd_observability_tools.png)

I have this, and my [Linux](http://www.brendangregg.com/linuxperf.html) ones, printed out and hung up around my desk. It helps jog the memory, especially when I'm switching between operating systems.

The slides for my talk are on [slideshare](http://www.slideshare.net/brendangregg/meetbsd2014-performance-analysis):

The talk was also videoed (thanks, iXsystems), which is on [youtube](https://www.youtube.com/watch?v=uvKMptfXtdo):

Apart from the slide content, my talk included additional discussion and live demos of profiling the Netflix Open Connect Appliances (OCAs), which stream our content. I even did some cscope browsing of kernel source, to understand tracing targets.

FreeBSD has the most advanced performance analysis toolset, which includes pmcstat(8) for CPU performance monitoring counter (PMC) analysis, and DTrace for static and dynamic tracing. On Linux I can get the same jobs done, but it sometimes feels like I can't get out of second gear. On FreeBSD, I can fly.

For pmcstat(8), I came up with a neat diagram for the PMC groups:

[![](/Perf/freebsd_pmc_groups.png)](/Perf/freebsd_pmc_groups.png)

I'm pretty happy with how this turned out. I do want to improve it next time I use it, and try to highlight frontend vs backend stall cycles, since that can be a useful approach for CPU analysis.

This is also an example of how more developed performance analysis is on FreeBSD: I tried to create this diagram for Linux perf\_events for my SCALE 13x talk, but it was lacking so many PMC groups that I didn't use it. Linux can get the job done, but more often you'll need to use the raw PMC counter specifications from the Intel manual. Stuck in second gear again.

Just before the talk I published a collection of DTrace scripts ( [DTrace-tools](https://github.com/brendangregg/DTrace-tools)) which I wrote for the Netflix OCAs, and are installed by default. I'll find a better home for these (FreeBSD wiki or source).

In the talk I said I should have written a script beforehand for measuring storage I/O size, without using the DTrace io provider, as there is a bug in its translator (involving struct types). Here's one way, by instrumenting GEOM using dynamic tracing:

```
# dtrace -n 'fbt::g_disk_start:entry { @ = quantize(args[0]->bio_length); }'
dtrace: description 'fbt::g_disk_start:entry ' matched 1 probe
^C

           value  ------------- Distribution ------------- count
            2048 |                                         0
            4096 |@@                                       196
            8192 |@                                        147
           16384 |                                         4
           32768 |                                         1
           65536 |                                         32
          131072 |                                         0
          262144 |                                         0
          524288 |                                         0
         1048576 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@    4799
         2097152 |                                         0

```

This histogram shows that most of the I/O fell into the 1-2 Mbyte bucket, which explains the 1 millisecond minimum latency from my demo. These Netflix OCAs are tuned for streaming workloads.

I enjoyed meeting others in the BSD community at MeetBSD CA, and watching the talks. I was a little surprised at how welcoming, happy, and *relaxed* the BSD community is. I've been to Linux events recently (SCALE is an exception) where the mood felt a bit tense: hallway conversations were about Linus's attitude, and whether that's ok, and how much people hate or love systemd. Things are currently much more relaxed in BSD land!

I hope you like my talk. Check out the [playlist](https://www.youtube.com/playlist?list=PLb87fdKUIo8TijlK7TuBeRMgylGaN9Ec9) for other MeetBSD CA talks. The day after this talk, I gave another on FreeBSD flame graphs, which I'll describe in a follow up post.

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
