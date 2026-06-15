---
title: Linux eBPF/bcc uprobes
url: https://www.brendangregg.com/blog/2016-02-08/linux-ebpf-bcc-uprobes.html
published: "2016-02-08T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-02-08/linux-ebpf-bcc-uprobes.html
---

# Linux eBPF/bcc uprobes

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

## Linux eBPF/bcc uprobes

08 Feb 2016

User-level dynamic tracing support was just added to bcc\[1\], a front-end to Linux eBPF\[2\]. As a spooky example, let's trace interactive commands entered on all running bash shells, system-wide:

```
# ./bashreadline
TIME      PID    COMMAND
05:28:25  21176  ls -l
05:28:28  21176  date
05:28:35  21176  echo hello world
05:28:43  21176  foo this command failed
05:28:45  21176  df -h
05:29:04  3059   echo another shell
05:29:13  21176  echo first shell again

```

This even sees commands that failed. bash doesn't need to be run in any special debug mode for this to work: all running bash shells are instrumented immediately, and new ones. You can walk up to a system that's never run eBPF before, and say: "so what's bash doing?" and then see. It's like a superpower.

Here's the entire [bashreadline](https://github.com/iovisor/bcc/blob/master/tools/bashreadline.py) bcc/eBPF program:

This is tracing the return of the readline() function from /bin/bash, using a uretprobe (user-level return probe). The uretprobe runs a custom eBPF program, printret(), which prints the returned string. Because eBPF programs only operate on their own stack memory (improving safety), we need to use bpf\_probe\_read() to pull in the string for later operations (bpf\_trace\_printk()). This may go away: there's a lot of work in bcc to automatically do the bpf\_probe\_read()s for you, so you can write tools more easily.

This currently accesses the return value from the x86\_64 `%ax` register\[3\], however, bcc should really provide an alias for this (eg, "rval"; it's bug #225\[4\]).

### gethostlatency

As another example, gethostlatency traces name lookups (DNS) system-wide:

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
06:11:28  29475  curl           1.00 foo

```

This time it's tracing three libc library functions, system wide: getaddrinfo(), gethostbyname(), and gethostbyname2(). The relevant code from the program is:

```
b.attach_uprobe(name="c", sym="getaddrinfo", fn_name="do_entry")
b.attach_uprobe(name="c", sym="gethostbyname", fn_name="do_entry")
b.attach_uprobe(name="c", sym="gethostbyname2", fn_name="do_entry")
b.attach_uretprobe(name="c", sym="getaddrinfo", fn_name="do_return")
b.attach_uretprobe(name="c", sym="gethostbyname", fn_name="do_return")
b.attach_uretprobe(name="c", sym="gethostbyname2", fn_name="do_return")

```

This attaches custom eBPF functions to the library function calls and function returns, via uprobes and uretprobes. The name="c" is referring to libc. Tracing both calls and returns was necessary to save and retrieve timestamps for calculating the latency. Check out the full program: [gethostlatency](https://github.com/iovisor/bcc/blob/master/tools/gethostlatency.py).

### Summary

User-level dynamic tracing is a new bcc/eBPF feature. I demonstrated it by tracing user-level functions and system library functions, and with two new tools: bashreadline and gethostlatency.

We have some work to improve uprobe usage, but that you can now use them at all is a big milestone.

### References & Links

1. [https://github.com/iovisor/bcc](https://github.com/iovisor/bcc)
2. [http://www.brendangregg.com/blog/2015-05-15/ebpf-one-small-step.html](http://www.brendangregg.com/blog/2015-05-15/ebpf-one-small-step.html)
3. [https://en.wikipedia.org/wiki/X86\_calling\_conventions#System\_V\_AMD64\_ABI](https://en.wikipedia.org/wiki/X86_calling_conventions#System_V_AMD64_ABI)
4. [https://github.com/iovisor/bcc/issues/225](https://github.com/iovisor/bcc/issues/225)

Thanks Brenden Blanco (PLUMgrid) for uprobe support!

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
