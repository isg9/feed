---
title: Linux ftrace Function Counting
url: https://www.brendangregg.com/blog/2014-07-13/linux-ftrace-function-counting.html
published: "2014-07-13T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-07-13/linux-ftrace-function-counting.html
---

# Linux ftrace Function Counting

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

## Linux ftrace Function Counting

13 Jul 2014

Here's another capability I didn't think was possible. What kernel bio functions are being called?

```
# ./funccount 'bio_*'
Tracing "bio_*"... Ctrl-C to end.
^C
FUNC                              COUNT
bio_attempt_back_merge               26
bio_get_nr_vecs                     361
bio_alloc                           536
bio_alloc_bioset                    536
bio_endio                           536
bio_free                            536
bio_fs_destructor                   536
bio_init                            536
bio_integrity_enabled               536
bio_put                             729
bio_add_page                       1004

```

Great! How about the top 5 functions beginning with "tcp\_" every second?

```
# ./funccount -i 1 -t 5 'tcp_*'
Tracing "tcp_*". Top 5 only... Ctrl-C to end.

FUNC                              COUNT
tcp_cleanup_rbuf                    386
tcp_service_net_dma                 386
tcp_established_options             549
tcp_v4_md5_lookup                   560
tcp_v4_md5_do_lookup                890

FUNC                              COUNT
tcp_service_net_dma                 498
tcp_cleanup_rbuf                    499
tcp_established_options             664
tcp_v4_md5_lookup                   672
tcp_v4_md5_do_lookup               1071

[...]

```

Awesome!

Like my previous post on [perf Hacktograms](http://www.brendangregg.com/blog/2014-07-10/perf-hacktogram.html), this capability is also bread-and-butter for advanced tracers like SystemTap, ktap, and DTrace. But I'm not using those. I'm not even using perf\_events.

This is using **dynamic tracing** and **by-CPU in-kernel aggregations** on a standard Linux 3.2 kernel.

It does this using ftrace, automated by my [funccount](https://github.com/brendangregg/perf-tools/blob/master/kernel/funccount) script.

ftrace is part of the Linux kernel, and is included at compile time by various FTRACE CONFIG options (including CONFIG\_DYNAMIC\_FTRACE, which on my systems is already turned on). You can operate it via control files under /sys/kernel/debug/tracing. It's a little tricky to use, but gets many jobs done. See the kernel source documentation [trace/ftrace.txt](https://www.kernel.org/doc/Documentation/trace/ftrace.txt) for details.

## Why

Understanding function call rates can be a useful tool for debugging and performance analysis. It's also part of a regular procedure I use when tracing subsystems, especially unfamiliar ones.

Dynamic tracing is amazing, but it can be hard to know where to start, especially when faced with hundreds of functions in a subsystem of interest. By counting which functions are actually in use, you can narrow down the potential targets. So you can use funccount to find active functions, and then other tracers (including [perf\_events dynamic tracing](http://www.brendangregg.com/perf.html#DynamicTracing)) to probe them in more detail.

Another approach for identifying active kernel functions is to create a [perf\_events CPU Flame Graph](http://www.brendangregg.com/perf.html#FlameGraphs) based on stack trace samples. I'll usually start with this, since it costs lower (fixed) overhead, relative to the sample rate, and can also solve numerous problems right off the bat. I'd move onto function counts when I'm digging deeper into a subsystem.

## funccount

This is a simple script to automate ftrace. It does one thing: kernel function counting.

```
# ./funccount -h
USAGE: funccount [-hT] [-i secs] [-d secs] [-t top] funcstring
                 -d seconds      # total duration of trace
                 -h              # this usage message
                 -i seconds      # interval summary
                 -t top          # show top num entries only
                 -T              # include timestamp (for -i)
  eg,
        funccount 'vfs*'         # trace all funcs that match "vfs*"
        funccount -d 5 'tcp*'    # trace "tcp*" funcs for 5 seconds
        funccount -t 10 'ext3*'  # show top 10 "ext3*" funcs
        funccount -i 1 'ext3*'   # summary every 1 second
        funccount -i 1 -d 5 'ext3*' # 5 x 1 second summaries

```

I added it to my [perf-tools](https://github.com/brendangregg/perf-tools) collection on github.

It works by enabling the ftrace function profiler. It creates per-CPU summaries (which are efficient – no synchronization overheads when updating counts), which funccount combines for the report. ftrace already provides the ability to match wildcards for its function filter.

Not all functions are visible from ftrace and funccount. If you think a function should be included in the output, but is missing, check for it in /proc/kallsyms (kernel symbols) and /sys/kernel/debug/tracing/available\_filter\_functions (what ftrace can trace).

Dynamic tracing of kernel functions has been problematic in the past on Linux, with the risk of kernel panics. I haven't experienced them (I've run this script on 3.2 and 3.16), but wanted to warn you of this anyway: I'd use a test machine (with load) to try this out on first.

Thanks to Steven Rostedt and others who have been busy adding great capabilities to ftrace.

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
