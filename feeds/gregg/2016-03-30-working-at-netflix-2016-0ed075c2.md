---
title: Working at Netflix 2016
url: https://www.brendangregg.com/blog/2016-03-30/working-at-netflix-2016.html
published: "2016-03-30T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-03-30/working-at-netflix-2016.html
---

# Working at Netflix 2016

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

## Working at Netflix 2016

30 Mar 2016

Some companies project an image of an engineering paradise, where their technologies are awesome, the staff are stunning, and the keyboards are made of gold. But after joining, you discover that the technologies are crummy, the staff can be terrible, and your right shift key is sticky. It's a company where dumb stuff happens but nobody can fix it, and the future outlook is dull. The paradise was a facade.

Netflix is not like that.

After two years I still find Netflix an amazing place to work. What's surprised me most is the lack of unpleasant surprises. Before I joined, Netflix looked awesome, but I wondered if there was some catch or downside that I'd only learn after working here. I have yet to find one.

How does Netflix do it? I think the [culture deck](http://www.slideshare.net/reed2001/culture-1798664) goes a long way to explain how, and it's still true. In short: everyone is professional, we work well together, innovate (freedom and responsibility), and have a good work/life balance. It works well.

This is also the most challenging job of my life, in part because of what I've chosen to work on. So far I've worked on kernel and hypervisor internals, various runtimes and databases, created many new performance tools, and performed distributed systems analysis as an SRE.

## SRE

Me?...an SRE? Yes, I've volunteered to join the on-call rotation for the Core SRE team, where I get to do distributed systems analysis and SRE work. Core SRE is the central SRE team at Netflix. My colleague, Dave Hahn, gave a good talk about it at [AWS re:Invent](https://www.youtube.com/watch?v=-mL3zT1iIKw).

Working on Core issues at Netflix has been a great opportunity for me to grow professionally, expanding from my typical specialty of systems analysis to doing distributed systems analysis at scale. Others on the performance team (which I'm on) also help Core, and our teams are co-located. The kinds of work we do is similar – troubleshooting complex software written by others.

Netflix is composed of many microservices, and each have their own teams who are on-call. Much of the time, service teams have set up early alerts so that they are notified and can fix issues before they affect customers. Core gets paged when there is a customer impacting issue.

When we get paged, we'll often find that a service team is already working on the issue and about to deploy a fix. Sometimes, however, we find that no one is working the issue – you're *it*! You have to figure out what – in the massive Netflix ecosystem – is broken, and help get it fixed ASAP. At the very least, Twitter is a slot machine of people saying that Netflix is down. If it's a really bad outage, major news networks are speculating about what Netflix is doing to fix it. They're talking about *you*, and the decisions you make in the next five minutes. Pressure's on!

I was both excited and a little terrified to do Core on-call, but for a few reasons it's been easier than I thought:

- Bad incidents are rare (thanks to planning, architecture, and regular Chaos testing).
- I'm never working an incident alone: other staff join to help, and I can page others.
- We have staff who are experts at distributed systems analysis, from whom I can learn.
- We have specialized tools that make this kind of analysis much easier.
- I'm only on-call about 3-4 days per month.

This has helped me gain some experience in distributed systems analysis at massive scale, and of the new methodologies and metrics involved: upstream vs downstream latency, timeout and failover statistics, error rates, auto-correlations, etc. I look forward to learning more and getting better at this – it's been a highlight so far of my time at Netflix.

## Other Highlights

Some other highlights from two years at Netflix:

- Building Java CPU flame graphs, helping take this from a crazy idea (modifying hotspot) to production.
- Developing new flame graph features: differentials, and using them for CPI flame graphs, and search.
- Developing a toolkit of Linux perf and ftrace tools: perf-tools, and using them in production.
- Getting some experience with FreeBSD on systems running heavy workloads.
- Learning some Xen internals, and contributing a Xen patch.
- rxNetty vs Tomcat analysis: getting deep on new vs old technologies.
- Many conference talks, including my first BSD conferences, JavaOne, and LinuxCon.
- Contributing to Linux BPF and developing new bcc tracing tools.
- Playing cricket on the Netflix cricket team!
- Working with awesome staff.

Plus a few more highlights I can’t talk about (yet). I also have many interesting projects I'll be working on next, which makes for a long todo list. More highlights to come.

## Should you work here?

Check the [culture deck](http://www.slideshare.net/reed2001/culture-1798664). It's not for everyone, and that's ok – being open allows people to self select. You can also read my own perspective about working at Netflix in a [previous post](/blog/2015-01-20/working-at-netflix.html), where I talked more about culture and recruiting. It's still true. And as with my last post, no one at Netflix asked me to write this one. I'm writing because I think our company culture is worth talking about, whether it helps you join us, or if it helps you improve the culture at your own company.

[![](/blog/images/2016/brendan_desk.jpg)](/blog/images/2016/brendan_desk.jpg)

*My desk in building D*

If the Netflix culture and company do sound attractive, and you're a senior professional with relevant skills and a top performer, I'd recommend checking our [jobs page](https://jobs.netflix.com/jobs). Most of these jobs are based in Los Gatos, where we've just opened two new buildings and two more are on the way. These have staff at desks in a semi-open office layout, with private places you can visit to work.

A problem you might have, if you are a senior engineer, is the phenomenon of "the more you know, the more you don't know". If you've dug deep on technologies in the past, you may have been exposed to their vast complexities, and how much you've yet to learn. This can hurt your confidence, and that's a problem when applying for jobs. I don't know of a great solution other than to be self aware that you're falling into this trap, as well as chatting to Netflix staff about this.

As with our engineers, we try to hire the best recruiters in the industry. So if you only talk to one recruiter this year, make it a Netflix recruiter!

Good luck, and come say hi if you join (I'm in D2 near a window overlooking the basketball court). You might also see me at the upcoming [SREcon16](https://www.usenix.org/conference/srecon16/), where I'm giving a talk on performance checklists for SREs.

UPDATE: [Working at Netflix 2017](http://www.brendangregg.com/blog/2017-05-16/working-at-netflix-2017.html).

*Update: Many people have been emailing me their resumes. I'm glad they are interested in working at Netflix, but I'm an engineer, not a hiring manager. Please use jobs.netflix.com, which will send your details to the right people.*

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
