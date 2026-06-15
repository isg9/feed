---
title: Why Don't You Use ...
url: https://www.brendangregg.com/blog/2022-03-19/why-dont-you-use.html
published: "2022-03-19T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2022-03-19/why-dont-you-use.html
---

# Why Don't You Use ...

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

## Why Don't You Use ...

19 Mar 2022

Working for a famous tech company, I get asked a lot "Why don't you use technology X?" X may be an application, programming language, operating system, hypervisor, processor, or tool. It may be because:

- It performs poorly.
- It is too expensive.
- It is not open source.
- It lacks features.
- It lacks a community.
- It lacks debug tools.
- It has serious bugs.
- It is poorly documented.
- It lacks timely security fixes.
- It lacks subject matter expertise.
- It's developed for the wrong audience.
- Our custom internal solution is good enough.
- Its longevity is uncertain: Its startup may be dead or sold soon.
- We know under NDA of a better solution.
- We know other bad things under NDA.
- Key contributors told us it was doomed.
- It made sense a decade ago but doesn't today.
- It made false claims in articles/blogs/talks and has lost credibility.
- It tolerates brilliant jerks and has no effective CoC.
- Our lawyers won't accept its T&Cs or license.

It's rarely because we don't know about it or don't understand it. Sometimes we do know about it, but have yet to find time to check it out. For big technical choices, it is often the result of an internal evaluation that involved various teams, and for some combination of the above reasons. Companies typically do not share these internal product evaluations publicly.

> It's easier to say what you *do use and why*, than what you *don't use and why not*.

I used to be the outsider asking the big companies, and their silence would drive me nuts. Do they not know about this technology? Why wouldn't they use it?...I finally get it now having seen it from the other side. They are usually well aware of various technologies but have reasons not to use them which they usually won't share.

## Why companies won't say why

**Private reasons**:

- It is too expensive.
- We know under NDA of a better solution.
- We know other bad things under NDA.
- Key contributors told us it was doomed.

These have all happened to me. Technology X may be too expensive because we're using another technology with a special discount that's confidential. If the reasons are under NDA, then I also cannot share it. In one case I was interested in a technology, only to have the CEO of the company that developed it, under NDA, tell me that he was abandoning it. They had not announced this publicly. I've also privately chatted with key technology contributors at conferences who are looking for something different to work on, because they believe their own technology is doomed.

At some companies, whatever reason may just be considered competitive knowledge and thus confidential.

**Complicated reasons**:

- It performs poorly.
- Our lawyers won't accept its T&Cs or license ( [example](/blog/2014-05-17/free-as-in-we-own-your-ip.html)).
- Our custom internal solution is good enough.
- It made sense a decade ago but doesn't today.
- It tolerates brilliant jerks and has no effective CoC.
- It lacks subject matter expertise.
- It's developed for the wrong audience.

These are complicated and time consuming to explain properly, and it may not be a good use of engineering time. If you rushed a quick answer instead, you can put the company in a worse position than by saying nothing: that of providing a *weak* explanation. Those vested in the technology will find it easy to attack your response – and have more time, energy, and interest in doing so – which can make your company look worse.

I've found one of these reasons is common: Discovering that a product was built without subject matter expertise. I've seen startups that do nothing new and nothing well in a space that's crying for new or better solutions. I usually find out later that the development team has no prior domain experience. It's complicated to explain, even to the developer team, as they lack the background to best understand why they missed the mark.

A common reason I've seen at smaller companies is when an internal solution is good enough. Adopting new technologies isn't "free": It takes (their limited) engineering time away from other projects, it adds technical debt, and it may add security risk (agents running as root). In this case, the other technology just doesn't have enough features or improved performance to justify a switch.

**Safer reasons**:

- It is poorly documented.
- It lacks features.
- It lacks debug tools.
- It has serious bugs.
- etc.

Those are objective and easy to discuss, right? Sort of. There are usually multiple reasons not to use a product, which might include all three: "safe", "complicated", and "private." It can be misleading to only mention the safe reasons. Also, if you do, and they get fixed, people will feel let down that you don't switch.

**Bad reasons**:

- Not invented here syndrome.
- Pride/ego.
- Corporate politics.
- etc.

I didn't include them in the above list, but there can be some poor reasons behind technology choices. At large tech companies (with many staff and their collective expertise) I'd expect that most technical choices will have valid reasons, even if the company won't share them, and poor reasons are the exception and not the rule. People think it's poor technical choices ten times more than it actually is.

Just as an example, I've been asked many times why my company uses FreeBSD for its CDN, with the assumption that there must be some dumb reason we didn't choose Linux. No, we do regular Linux vs FreeBSD production tests (which I've helped analyze) and FreeBSD is faster for that workload. For a long time we just avoided this question. I finally got approval to mention it in a footnote in my last book (SysPerf2 page 124).

The only sure way to find out why a company doesn't use a given technology is to join that company and then ask the right team. Otherwise, try asking questions that are safer to answer. For example: "Are you aware of technology X?," and "Are you aware that technology X claims to be better at Y?". This can at least rule out ignorance.

Update: This was discussed on [Hacker News](https://news.ycombinator.com/item?id=30755861), which included some other reasons, including:

- Engineers don't know X and can't hire X engineers (certainly true at smaller companies).
- The cost of switching outweighs the benefits.
- Legacy.

And I liked the comment about "...random drive-by comments where the person is really just advertising x, rather than being genuinely curious."

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
