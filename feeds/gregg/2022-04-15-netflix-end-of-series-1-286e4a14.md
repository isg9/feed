---
title: Netflix End of Series 1
url: https://www.brendangregg.com/blog/2022-04-15/netflix-farewell-1.html
published: "2022-04-15T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2022-04-15/netflix-farewell-1.html
---

# Netflix End of Series 1

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

## Netflix End of Series 1

15 Apr 2022

A large and unexpected opportunity has come my way outside of Netflix that I've decided to try. Netflix has been the best job of my career so far, and I'll miss my colleagues and the culture.

![](/blog/images/2022/netflixlogo2014.jpg)

*offer letter logo (2014)*

[![](/blog/images/2022/javamixedmodeflamegraph.png)](https://netflixtechblog.com/java-in-flames-e763b3d32166)

*flame graphs (2014)*

[![](/blog/images/2022/bpftools-thumb.jpg)](/Perf/bcc_tracing_tools.png)

*eBPF tools (2014-2019)*

[![](/blog/images/2017/rxnetty_pmcs01.png)](/blog/2017-05-04/the-pmcs-of-ec2.html)

*PMC analysis (2017)*

[![](/blog/images/2022/netflixdesk2020-1920p.jpg)](/blog/images/2022/netflixdesk2020-1920p.jpg)

*my pandemic-abandoned*

*desk (2020); [office wall](/blog/images/2022/brendanwall.jpg)*

I joined Netflix in 2014, a company at the forefront of cloud computing with an attractive [work culture](/blog/2017-11-13/brilliant-jerks.html). It was the most challenging job among those I interviewed for. On the Netflix Java/Linux/EC2 stack there were no working mixed-mode flame graphs, no production safe dynamic tracer, and no PMCs: All tools I used extensively for advanced performance analysis. How would I do my job? I realized that this was a challenge I was best suited to fix. I could help not only Netflix but all customers of the cloud.

Since then I've done just that. I developed the original JVM changes to allow [mixed-mode flame graphs](http://techblog.netflix.com/2015/07/java-in-flames.html), I pioneered using [eBPF for observability](/blog/2015-05-15/ebpf-one-small-step.html) and helped develop the [front-ends and tools](/Perf/bcc_tracing_tools.png), and I worked with Amazon to get [PMCs enabled](/blog/2017-05-04/the-pmcs-of-ec2.html) and developed tools to use them. Low-level performance analysis is now possible in the cloud, and with it I've helped Netflix save a very large amount of money, mostly from service teams using flame graphs. There is also now a flourishing industry of observability products based on my work.

Apart from developing tools, much of my time has been spent helping teams with performance issues and evaluations. The Netflix stack is more diverse than I was expecting, and is explained in detail in the [Netflix tech blog](https://netflixtechblog.com/): The production cloud is AWS EC2, Ubuntu Linux, Intel x86, mostly Java with some Node.js (and other languages), microservices, Cassandra (storage), EVCache (caching), Spinnaker (deployment), Titus (containers), Apache Spark (analytics), Atlas (monitoring), FlameCommander (profiling), and at least a dozen more applications and workloads (but no 3rd party agents in the BaseAMI). The Netflix CDN runs FreeBSD and NGINX (not Linux: I published a Netflix-approved [footnote](/blog/images/2022/netflixcdn.png) in my last book to explain why). This diverse environment has always provided me with interesting things to explore, to understand, analyze, debug, and improve.

I've also used and helped develop many other technologies for debugging, primarily perf, Ftrace, eBPF (bcc and bpftrace), PMCs, MSRs, Intel vTune, and of course, [flame graphs](/flamegraphs.html) and [heat maps](/heatmaps.html). Martin Spier and I also created [Flame Scope](https://netflixtechblog.com/netflix-flamescope-a57ca19d47bb) while at Netflix, to analyze perturbations and variation in profiles.

I've also had the chance to do other types of work. For 18 months I joined the [CORE SRE team](/blog/2016-05-04/srecon2016-perf-checklists-for-sres.html) rotation, and was the primary contact for Netflix outages. It was difficult and fascinating work. I've also created internal training materials and classes, apart from my books. I've worked with awesome colleagues not just in cloud engineering, but also in open connect, studio, DVD, NTech, talent, immigration, HR, PR/comms, legal, and most recently ANZ content.

Last time I quit a job, I wanted to share publicly the reasons why I left, but I ultimately did not. I've since been asked many times why I resigned that job (not unlike [The Prisoner](https://en.wikipedia.org/wiki/The_Prisoner)) along with much speculation (none true). I wouldn't want the same thing happening here, and having people wondering if something bad happened at Netflix that caused me to leave: I had a great time and It's a great company!

I'm thankful for the opportunities and support I've had, especially from my former managers [Coburn](https://www.linkedin.com/in/coburnw/) and [Ed](https://www.linkedin.com/in/edwhunter/). I'm also grateful for the support for my work by other companies, technical communities, social communities (Twitter, HackerNews), conference organizers, and all who have liked my work, developed it further, and shared it with others. Thank you. I hope my last two books, [Systems Performance 2nd Ed](/systems-performance-2nd-edition-book.html) and [BPF Performance Tools](/bpf-performance-tools-book.html) serve Netflix well in my absence and everyone else who reads them.

I'll still be posting here in my next job. More on that soon...

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
