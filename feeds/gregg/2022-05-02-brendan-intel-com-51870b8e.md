---
title: Brendan@Intel.com
url: https://www.brendangregg.com/blog/2022-05-02/brendan-at-intel.html
published: "2022-05-02T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2022-05-02/brendan-at-intel.html
---

# Brendan@Intel.com

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

## Brendan@Intel.com

02 May 2022

I'm thrilled to be joining Intel to work on the performance of everything, apps to metal, with a focus on cloud computing. It's an exciting time to be joining: The geeks are back with [Pat Gelsinger](https://twitter.com/PGelsinger) and [Greg Lavender](https://twitter.com/GregL_Intel) as the CEO and CTO; new products are launching including the Sapphire Rapids processor; there are more competitors, which will drive innovation and move the whole industry forward more quickly; and Intel are building new fabs on US soil. It's a critical time to join, and an honour to do so as an Intel fellow, based in Australia.

My dream is to turn computer performance analysis into a science, one where we can completely understand the performance of everything: of applications, libraries, kernels, hypervisors, firmware, and hardware. These were the opening words of my 2019 [AWS re:Invent talk](https://www.youtube.com/watch?v=16slh29iN1g), which I followed by demonstrating rapid on-the-fly dynamic instrumentation of the Intel wireless driver. With the growing complexities of our industry, both hardware and software offerings, it has become increasingly challenging to find the root causes for system performance problems. I dream of solving this: to be able to observe everything and to provide complete answers to any performance question, for any workload, any operating system, and any hardware type.

A while ago I began exploring the idea of building this performance analysis capability for a major cloud, and use it to help find performance improvements to make that cloud industry-leading. The question was: Which cloud should I make number one? I'm grateful for the companies who explored this idea with me and offered good opportunities. I wasn't thinking Intel to start with, but after spending much time talking to Greg and other Intel leaders, I realized the massive opportunity and scope of an Intel role: I can work on new performance and debugging technologies for everything from apps down to silicon, across all xPUs (CPUs, GPUs, IPUs, etc.), and have a massive impact on the world. It was the most challenging of the options before me, just as Netflix was when I joined it, and at that point it gets hard for me to say no. I want to know if I can do this. Why climb the highest mountain?

Intel is a deeply technical company and a leader in high performance computing, and I'm excited to be working with others who have a similar appetite for deep technical work. I'll also have the opportunity to hire and mentor staff and build a team of the world's best performance and debugging engineers. My work will still involve hands-on engineering, but this time with resources. I was reminded of this while interviewing with other companies: One interviewer who had studied my work asked "How many staff report to you?" "None." He kept returning to this question. I got the feeling that he didn't actually believe it, and thought if he asked enough times I'd confess to having a team. (I'm reminded of my days at Sun Microsystems, where the joke was that I must be a team of clones to get so much done – so I made a [picture](https://www.brendangregg.com/Images/brendan_clones2006.jpg).) People have helped me build things (I have detailed Acknowledgement lists in my books), but I've always been an individual contributor. I now have an opportunity at Intel to grow further in my career, and to help grow other people. Teaching others is another passion of mine – it's what drives me to write books, create training materials, and write blog posts here.

I'll still be working on eBPF and other open source projects, and I'm glad that Intel's leadership has committed to continue supporting [open source](https://www.linkedin.com/pulse/open-letter-ecosystem-pat-gelsinger). Intel has historically been a top contributor to the Linux kernel, and supports countless other projects. Many of the performance tools I've worked on are open source, in particular eBPF, which plays an important role for understanding the performance of everything. eBPF is in development for Windows as well (not just Linux where it originated).

For cloud computing performance, I'll be working on projects such as the [Intel DevCloud](https://www.intel.com/content/www/us/en/developer/tools/devcloud/overview.html), run by [Markus Flierl](https://www.linkedin.com/in/markus-flierl-375185/), Corporate VP, General Manager, Intel Developer Cloud Platforms. I know Markus from Sun and I'm glad to be working for him at Intel.

While at Netflix in the last few years I've had more regular meetings with Intel than any other company, to the point where we had started joking that I already worked for Intel. I've found them to be not only the deepest technical company – capable of analysis and debugging at a mind-blowing atomic depth – but also professional and a pleasure to work with. This became especially clear after I recently worked with another hardware vendor, who were initially friendly and supportive but after evaluations of their technology went poorly became bullying and misleading. You never know a company (or person) until you see them on their worst day. Over the years I've seen Intel on good days and bad, and they have always been professional and respectful, and work hard to do right by the customer.

A bonus of the close relationship between Intel and Netflix, and my focus on cloud computing at Intel, is that I'll likely continue to help the performance of the Netflix cloud, as well as other clouds (that might mean you!). I'm looking forward to meeting new people in this bigger ecosystem, making computers faster everywhere, and understanding the performance of everything.

The title of this post is indeed my email address (I also used to be [Brendan@Sun.com](mailto:Brendan@Sun.com)).

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
