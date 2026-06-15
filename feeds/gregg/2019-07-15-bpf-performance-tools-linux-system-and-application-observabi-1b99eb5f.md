---
title: 'BPF Performance Tools: Linux System and Application Observability (book)'
url: https://www.brendangregg.com/blog/2019-07-15/bpf-performance-tools-book.html
published: "2019-07-15T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2019-07-15/bpf-performance-tools-book.html
---

# BPF Performance Tools: Linux System and Application Observability (book)

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

## BPF Performance Tools: Linux System and Application Observability (book)

15 Jul 2019

BPF (eBPF) tracing is a superpower that can analyze everything, and I'll show you how in my upcoming book [BPF Performance Tools: Linux System and Application Observability](http://www.brendangregg.com/bpf-performance-tools-book.html), coming soon from Addison Wesley. The book includes over 150 BPF observability tools that you can run to find performance wins and troubleshoot software, and also shows you how to write your own. Over one hundred of these BPF tools are newly-developed for this book; you can see many of them in this diagram:

[![](/blog/images/2019/bpf_performance_tools.png)](/blog/images/2019/bpf_performance_tools_book.png)

I've found this diagram to be a useful reminder and checklist when debugging production issues at Netflix, and I have it printed on a wall so I can glance at it. You may find it helps you, too, which is why it is on the [cover](/blog/images/2019/bpfperftools_bookcover_500.png).

Since BPF does so many things, it is becoming a technology name and no longer an acronym. It originated as Berkeley Packet Filter (BPF), an in-kernel execution engine that processes a virtual instruction set, and has been extended (aka eBPF) for providing a safe way to extend kernel functionality. BPF is in the Linux kernel.

## Why Your Book?

BPF is a new in-demand technology, and I expect that, over time, a dozen or more books about it may be published – as happens for other hot new technologies. Guess how many books there are about Docker? I lost count at [30](https://www.amazon.com/s?k=docker&rh=n%3A5&ref=nb_sb_noss)! Some other future BPF books may be good, and I'll recommend them as appropriate. To answer the question why you should buy *my* book:

- It is about using BPF for performance analysis, and is written by a veteran performance engineer and technical trainer (myself).
- At over 700 pages, it covers CPUs, memory, disks, file systems, networking, multiple languages, applications, containers, hypervisors, security analysis, and the Linux kernel.
- It contains over 150 tools (often with source code), including over 100 brand-new tools mostly developed by myself, so that you can get started right away finding performance wins in any of those areas.
- It also covers over 30 traditional tools (iostat(1), perf(1), etc) so you can use the right tool for the job, even when that tool is not BPF.
- It goes far beyond what you can find online, which I previously summarized ( [learn eBPF tracing](http://www.brendangregg.com/blog/2019-01-01/learn-ebpf-tracing.html)).
- It focuses on the recommended BPF tracing front ends: [BCC](https://github.com/iovisor/bcc) and [bpftrace](https://github.com/iovisor/bpftrace).
- It shows production screenshots and use cases from a major site (Netflix), including gotchas we've hit in production, such as fixing stacks and symbols, and managing overhead.
- It was developed with the help of many members and leaders of the BPF community.
- I've been helping lead BPF observability since 2014, and I'm a major contributor and maintainer for the front-ends. Among other things I created the tutorials and reference guides.
- This is also my fourth book to cover tracing.

[![](/blog/images/2019/bpfperftools_bookcover_500.png)](/bpf-performance-tools-book.html)

I was previously a professional technical trainer for Sun Educational Services, where my classes included DTrace, another tracing tool similar to BPF tracing. I learned firsthand how to bring beginners from different backgrounds up to speed. Early courses covered only the DTrace language, and expected students to figure out what to do with it. I found it was more effective to teach by example: explain the goals and targets of performance analysis, and then share real tools to meet those goals, and how to develop new ones. I've followed this proven approach in my book.

I've taken a break from training, but I have an opportunity to start again at Netflix, and I'll be using this book as materials for an internal BPF course. This course will be internal only: you'll need to join Netflix to attend my class! (Although I'll submit a short version for USENIX LISA/SREcon.)

## bpftrace for the Win

We (the BPF community) began planning this book in 2017, and posted an [announcement](https://github.com/iovisor/bcc/issues/1506) about it so that anyone looking could [find it](/blog/images/2019/google-ebpf-book-4jan2019.png). At the time, BCC was great for complex tools, but it was still in development and changing. Also, no higher-level front end had been completed, but we knew we needed one beyond BCC. Since then, BCC has largely stabilized, and [bpftrace](https://github.com/iovisor/bpftrace) has become a mature high-level front-end: one that is built from the ground up for BPF. bpftrace is already in use for production analysis at companies including Netflix, Facebook, and Shopify. It allows tool source to be so short and concise it can be included in the book in its entirety. It's like including pseudocode that you can run.

[![](/BPF/bpf-book-draft.jpg)](/BPF/bpf-book-draft.jpg)

*draft copy*

The last nine months have been hectic as I and others have added features to bpftrace, fixed bugs, and stabilized the API. The analysis of so many things unearthed many bugs and missing parts; fixing these was good not only for the book, but also for bpftrace. A special thanks to Mary Marchini, Willian Gasper, Alastair Robertson, Dale Hamel, Dan Xu, Augusto Mecking Caringi, and others, for all their work in these months. I'm also grateful that my fiancée, Deirdré Straughan, could find the time outside of her AWS job to do technical copy-edit. Thanks also to Alastair Robertson for creating bpftrace, Alexei Starovoitov and Daniel Borkmann for creating eBPF, and over forty other people involved in reviewing and contributing to this book.

## A New Era for Linux Observability

As you'll see, this book is the start of a new era for Linux observability: the power to easily see anything and everything, in production. A time where you can pose arbitrary questions of the system, and it can answer them. A time where you are free to think about performance in new ways, and you can fetch hard data to support or disprove your new ideas.

## Isn't BPF still evolving?

It is, but I had a static goal for the book: to get all of the included 150+ tracing tools to work. After many months of development, we've reached that milestone. I also involved the main BPF developers so I could document their planned work, to make this book as future-proof as possible.

BPF Type Format (BTF) is the only large change expected for tracing in the nearish future. It will provide kernel struct information beyond the linux-headers package, and is summarized in the book. A few of the tools declare missing structs as a workaround, and some years from now it will be possible to remove those workarounds.

What I expect to break sooner is the kprobe-based source code in the book. kprobes is considered an unstable API as it instruments kernel functions, and a kernel engineer may rename a function in a later kernel, breaking the tool. This is why the tools use the stable tracepoint interface whenever possible. But it's not always possible, and this is the nature of tracing. This is not, however, the hardest problem you'll face. The hardest problem is knowing what to do with BPF tracing, and even broken tools are a source of useful ideas. I will create a GitHub repository for the book tools as-is plus updated versions, and allow everyone to submit updates as they are found to be needed.

## What about Systems Performance?

[Systems Performance](http://www.brendangregg.com/sysperfbook.html) has sold very well (thanks, everyone!). It's recommended reading for new hires at several companies, including some teams at Facebook and SUSE. It is the encyclopedia of performance, covering many topics with a balance of theory and practice. BPF Performance Tools focuses on one topic only – observability – and goes deeper. The BPF book is about doing analysis right now, with these tools and capabilities. Now that I've finished it, I can do a second edition of Systems Performance (not right away…).

## Good Luck

BPF is the hot new technology, and it is an extreme privilege to have written the book on it. I was determined to give you the best book imaginable, and not waste this opportunity. I hope you enjoy it, and it helps you find valuable performance wins. Good luck!

For URLs where to find the book and updates, please see my website [BPF Performance Tools (book)](http://www.brendangregg.com/bpf-performance-tools-book.html).

Discussion on [hacker news](https://news.ycombinator.com/item?id=20442200)

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
