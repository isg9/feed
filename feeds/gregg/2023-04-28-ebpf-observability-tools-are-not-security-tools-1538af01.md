---
title: eBPF Observability Tools Are Not Security Tools
url: https://www.brendangregg.com/blog/2023-04-28/ebpf-security-issues.html
published: "2023-04-28T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2023-04-28/ebpf-security-issues.html
---

# eBPF Observability Tools Are Not Security Tools

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

## eBPF Observability Tools Are Not Security Tools

28 Apr 2023

eBPF has many uses in improving computer security, but just taking eBPF observability tools as-is and using them for security monitoring would be like driving your car into the ocean and expecting it to float.

Observability tools are designed have the lowest overhead possible so that they are safe to run in production while analyzing an active performance issue. Keeping overhead low can require tradeoffs in other areas: tcpdump(8), for example, will drop packets if the system is overloaded, resulting in incomplete visibility. This creates an obvious security risk for tcpdump(8)-based security monitoring: An attacker could overwhelm the system with mostly innocent packets, hoping that a few malicious packets get dropped and are left undetected. Long ago I encountered systems which met strict security auditing requirements with the following behavior: If the kernel could not log an event, it would immediately **halt**! While this was vulnerable to DoS attacks, it met the system's security auditing non-repudiation requirements, and logs were 100% complete.

There are ways to evade detection in other tools as well, like top(1) (since it samples processes and relies on its comm field) and even ls(1) (putting escape characters in files). Rootkits do this. These techniques have been known in the industry for decades and haven't been "fixed" because they aren't "broken." They are cars, not boats. Similar methods can be used to evade detection in the eBPF bcc and bpftrace observability tools as well: overwhelming them with events, doing time-of-check-time-of-use attacks (TOCTOU), escape characters, etc.

When will the eBPF community "fix" these tools? Well, when will Tesla fix my Model 3 so I can drive it under the Oakland bridge instead of over it? (I joke, and I don't drive a Tesla.) What you actually want is a security monitoring tool that meets a different set of requirements. Trying to adapt observability tools into security tools generally increases overhead (e.g., adding extra probes) which negates the main reason I developed these using eBPF in the first place. That would be like taking the wheels off a car to help make it float. There are other issues as well, like decreasing maintainability when moving probes from stable tracepoints to unstable inner workings for TOU tracing. Had I written these as security tools to start with, I would have done them differently: I'd start with LSM hooks, use a plugin model instead of standalone CLI tools, support configurable policies for event drop behavior, optimize event logging (which we still haven't [done](https://github.com/iovisor/bcc/issues/1033)), and lots more.

None of this should be news to experienced security engineers. I'm writing this post because others see the tools and examples I've shared and believe that, with a bit of shell scripting, they could have a good security monitoring product. I get that it looks that way, but in reality there's a bunch of work to do. Ideally I'd link to an example in bcc for security monitoring (we could create a subdirectory for them) but that currently doesn't exist. In the meantime my best advice is: If you are making a security monitoring product, hire a good security engineer (e.g., someone with solid pen-testing experience).

BPF for security monitoring was first explored by myself and a Netflix security engineer, Alex Maestretti, in a [2017 BSides talk](https://www.brendangregg.com/Slides/BSidesSF2017_BPF_security_monitoring) (some slides below). Since then I've worked with other security engineers on the topic (hi Michael, Nabil, Sargun, KP). (I also did security work many years ago, so I'm not completely new to the topic.)

![](http://www.brendangregg.com/Slides/BSidesSF2017_BPF_security_monitoring/BSidesSF2017_BPF_security_monitoring_013.jpg)

![](http://www.brendangregg.com/Slides/BSidesSF2017_BPF_security_monitoring/BSidesSF2017_BPF_security_monitoring_014.jpg)

*[BSidesSF2017 BPF security monitoring: Alex Maesretti, Brendan Gregg](/Slides/BSidesSF2017_BPF_security_monitoring)*

There is potential for an awesome eBPF security product, and it's not just the visibility that's valuable (all those arrows) it's also the low overhead. These slides included our [overhead evaluation](https://www.brendangregg.com/Slides/BSidesSF2017_BPF_security_monitoring/#17) showing bcc/eBPF was far more efficient than auditd or go-audit. (It was pioneering work, but unfortunately the slides are all we have: Alex, I, and others left Netflix before open sourcing it.) There are now other eBPF security products, including open source projects (e.g., [tetragon](https://github.com/cilium/tetragon)), but I don't know enough about them all to have a recommendation.

Note that I'm talking about the observability tools here and not the eBPF kernel runtime itself, which has been designed as a secure sandbox. Nor am I talking about privilege escalation, since to run the tools you already need root access (that car has sailed!).

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
