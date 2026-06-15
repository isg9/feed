---
title: 'From Clouds to Roots: Performance Analysis at Netflix'
url: https://www.brendangregg.com/blog/2014-09-27/from-clouds-to-roots.html
published: "2014-09-27T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-09-27/from-clouds-to-roots.html
---

# From Clouds to Roots: Performance Analysis at Netflix

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

## From Clouds to Roots: Performance Analysis at Netflix

27 Sep 2014

How do companies like Netflix, Google, and Facebook do root cause performance analysis in their cloud environments? These companies use Linux heavily, so are they using SystemTap or another tracer? Netflix is hosted on EC2, how do they do low level CPU profiling, if Xen guests can't access the CPU counters? ... And if these companies don't do these kinds of profiling, aren't they losing millions because of it?

On Thursday it was my privilege to speak again at [Surge](http://surge.omniti.com/2014), the scalability and performance conference. My talk was titled [From Clouds to Roots](http://surge.omniti.com/2014/speakers/speakers.html?gregg-brendan%5D), and showed how Neflix does root cause performance analysis, and answers these questions and more. The slides are on [slideshare](http://www.slideshare.net/brendangregg/netflix-from-clouds-to-roots):

And the video is on [youtube](http://www.youtube.com/watch?v=H-E0MQTID0g):

For background, I began by explaining the Netflix fault-tolerant architecture: how this can automatically work around performance issues, and what kinds of new issues this can introduce. Then I summarized key cloud-wide performance tools, and how we use these to isolate issues to an instance. And finally instance analysis, with the low-level tools we run to find the root cause.

Summary of cloud-wide performance tools:

[![](/blog/images/2014/netflix_cloud_perf_analysis.png)](/blog/images/2014/netflix_cloud_perf_analysis.png)

Summary of instance performance tools:

[![](/blog/images/2014/linux_EC2_perf_tools.png)](/blog/images/2014/linux_EC2_perf_tools.png)

People liked the talk, although it surprised some. One comment was "it didn't feel like a Brendan talk". Yes, I've expanded my scope at Netflix, and I'm working on higher-level analysis now, not just low-level performance tools. It's exciting for me, and I feel I have a better and more balanced view of performance analysis, which I captured in this talk.

Keep in mind that you aren't necessarily expected to do the low-level performance analysis yourself. I summarized the likely real scenarios at the end of the talk:

- A. Your company has one or more people who do advanced perf analysis (perf team). Ask them to do it.
- B. You actually are that person. Do it.
- C. You buy a product that does it. Ask them to add low-level capabilities.

Perhaps the most valuable take-away of this talk is that you be aware that such low-level analysis is possible, so that if you are in situation (A) or (C), you can ask for it. I've found that performance monitoring companies often build products based on what customers think they want. However, what customers think they want is bounded by what they know is possible. You really want products that make low-level analysis easy, including flame graphs and latency heat maps. Some monitoring products already provide these (eg, Circonus, AppNeta), and at Netflix we're building Vector.

Last year at Surge, I saw a great talk by [Coburn Watson](http://twitter.com/coburnw) titled [How Netflix Maximizes Scalability, Resilience, and Engineering Velocity in the Cloud](http://www.youtube.com/watch?v=7-13wV3WO8Q), which I recommend. His relaxed style threw me at first: he's able to talk very casually about complex performance issues in distributed environments, because he actually understands them extremely well. This year, he's now my manager at Netflix, and he's also still hiring. Great at performance and want to work with both Coburn and myself? Let him [know](mailto:cwatson@netflix.com). :-)

For more about low level performance analysis tools, see my [previous blog posts](/blog), and my [perf-tools](https://github.com/brendangregg/perf-tools) and [msr-cloud-tools](https://github.com/brendangregg/msr-cloud-tools) collections on github.

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
