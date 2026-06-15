---
title: perf CPU Sampling
url: https://www.brendangregg.com/blog/2014-06-22/perf-cpu-sample.html
published: "2014-06-22T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-06-22/perf-cpu-sample.html
---

# perf CPU Sampling

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

## perf CPU Sampling

22 Jun 2014

top(1) shows a process is burning CPU – what do you do next? Depending on the application and context, you might use an application profiler to see why, or reads its logs, or even kill it. However, there's one answer that works for any application, even the kernel: use a *system profiler* like Linux `perf` (aka perf\_events).

`perf` isn't some random tool: it's part of the Linux kernel, and is actively developed and enhanced. It is also powerful: it can instrument hardware counters, static tracepoints, and dynamic tracepoints.

In this post I'll restrain myself to one feature: CPU sampling. Lets pretend this is our target:

```
top - 04:38:41 up 29 days, 9 min,  2 users,  load average: 2.82, 3.26, 1.67
Tasks: 133 total,   9 running, 123 sleeping,   0 stopped,   1 zombie
Cpu(s): 28.0%us, 72.0%sy,  0.0%ni,  0.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   7629464k total,  1481328k used,  6148136k free,   285412k buffers
Swap:        0k total,        0k used,        0k free,   566712k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
 6336 root      20   0  117m  976  488 R   25  0.0   0:01.35 fio
 6338 root      20   0  117m  972  484 R   25  0.0   0:01.35 fio
 6339 root      20   0  117m  976  488 R   25  0.0   0:01.35 fio
 6340 root      20   0  117m  976  488 R   25  0.0   0:01.33 fio
 6342 root      20   0  117m  976  488 R   25  0.0   0:01.33 fio
 6335 root      20   0  117m  972  484 R   25  0.0   0:01.32 fio
 6341 root      20   0  117m  972  484 R   25  0.0   0:01.33 fio
 6337 root      20   0  117m  960  472 R   24  0.0   0:01.31 fio
 2337 bgregg-t  20   0  149m 5608 4436 S    0  0.1   3:42.93 postgres
[...]

```

This top(1) output shows several `fio` processes eating 25% CPU each. Why? What are they doing?

### 1\. Check perf is installed

With no arguments, it should print a help message:

```
# perf

 usage: perf [--version] [--help] COMMAND [ARGS]

 The most commonly used perf commands are:
[...]

```

If it's not there, you may find it can be added from the linux-tools-common package. It's also under tools/perf in the Linux kernel source.

### 2\. Profile CPUs

```
# sudo perf record -F 99 -a -g -- sleep 20
[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.560 MB perf.data (~24472 samples) ]

```

Options are:

- `-F 99`: sample at 99 Hertz (samples per second). I'll sometimes sample faster than this (up to 999 Hertz), but that also costs overhead. 99 Hertz should be negligible. Also, the value '99' and not '100' is to avoid lockstep sampling, which can produce skewed results.
- `-a`: samples on all CPUs. Without it, it only samples a supplied command or PID.
- `-g`: include stack traces.
- `--`: skips providing a -g argument (in newer `perf` versions, -g can pick the stack unwinding method).
- `sleep 20`: a dummy command, used to set the duration of our sampling.

As `perf` tells you, it writes a perf.data file.

### 3\. Read profile

```
# sudo perf report -n --stdio
[...]
# Overhead  Samples     Command       Shared Object                             Symbol
# ........ ..........  ........  ..................  .................................
#
    20.97%        208       fio  [kernel.kallsyms]   [k] hypercall_page
                 |
                 --- hypercall_page
                     check_events
                    |
                    |--63.94%-- 0x7fff695c398f
                    |
                    |--18.27%-- 0x7f0c5b72bd2d
                    |
                     --17.79%-- 0x7f0c5b72c46d

    14.21%        141       fio  [kernel.kallsyms]   [k] copy_user_generic_string
                 |
                 --- copy_user_generic_string
                     do_generic_file_read.constprop.33
                     generic_file_aio_read
                     do_sync_read
                     vfs_read
                     sys_read
                     system_call_fastpath
                     0x7f0c5b72bd2d

    10.79%        107       fio  [vdso]              [.] 0x7fff695c398f
                 |
                 --- 0x7fff695c398f
                     clock_gettime
[...]

```

You can just run `perf report` for the interactive text user-interface, and drill down stacks using the arrow keys. I find that mode laborious, and usually use this `--stdio` mode instead. `-n` prints sample counts.

The percentages show the breakdowns at each level. Multiply them to see the percentage for each leaf. For example, 0x7fff695c398f (whatever that is) was sampled 20.97% x 63.94% of the time (= %13.40). If you want `perf` to do the multiplications for you, and always show the absolute percentages, use `-g graph`.

The code `perf` is showing may be alien to you, but it shouldn't take long to learn at least the "hottest" (most frequently sampled) stacks. Often the function names are enough of a clue. `clock_gettime` is probably ... getting the time. Can `fio` avoid doing that, or do it differently, to eliminate this overhead? (yes, in this case.)

### Common Issues

Hexidecimal numbers are printed if perf\_events can't translate the symbols, which can happen with stripped binaries, or JIT'd code. For the former, look for dbgsym packages (debug symbols), or recompile and don't strip applications, and also make sure the kernel has CONFIG\_KALLSYMS. For the latter, that's a bigger topic I'll write about another time (perf's JIT support).

Incomplete stacks usually mean -fomit-frame-pointer was used – a compiler optimization that makes little positive difference in the real world, but breaks stack profilers. Always compile with -fno-omit-frame-pointer. More recent `perf` has a `-g dwarf` option, to use the alternate libunwind/dwarf method for retrieving stacks.

While it can be a bit of work to get full stacks with symbols, a partially working profile can be enough to solve some problems. For example, I may not be able to see Java methods in the JVM, but I can see JVM system library usage, kernel CPU usage, and GC.

### Flame Graphs

If the output of `perf report` is too long to read quickly, you can reprocess the perf.data file using `perf script` and visualize it using [perf Flame Graphs](http://www.brendangregg.com/perf.html#FlameGraphs). I use them all the time.

### But wait, there's more!

I wanted to write a simple, short example of `perf`, that wasn't long and intimidating. There's a **lot** more to perf\_events: see my [perf examples](http://www.brendangregg.com/perf.html) page and the official [perf wiki](https://perf.wiki.kernel.org/index.php/Main_Page).

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
