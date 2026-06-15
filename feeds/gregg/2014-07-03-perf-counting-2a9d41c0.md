---
title: perf Counting
url: https://www.brendangregg.com/blog/2014-07-03/perf-counting.html
published: "2014-07-03T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-07-03/perf-counting.html
---

# perf Counting

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

## perf Counting

03 Jul 2014

"I just want to know how many times this is called."... This is a question I'm often asking for kernel, device, and application events, and there is an efficient way to answer them: using Linux `perf` `stat`.

The `perf` tool (aka perf\_events) has different modes of operation. My previous examples on [CPU sampling](http://www.brendangregg.com/blog/2014-06-22/perf-cpu-sample.html), [static tracepoints](http://www.brendangregg.com/blog/2014-06-29/perf-static-tracepoints.html), and [heat maps](http://www.brendangregg.com/blog/2014-07-01/perf-heat-maps.html), used a mode perf\_events calls "sampling", where a binary perf.data file is written containing event data. Another mode, "counting", summarizes events in-kernel and passes the summary to user space. This is more efficient, costing less overhead in terms of CPU and storage.

### Process Counts

How many processes are being created and destroyed? Using `perf` `stat` to *count* the sched\_process tracepoints for 5 seconds:

```
# perf stat -e 'sched:sched_process_*' -a sleep 5

 Performance counter stats for 'system wide':

                20      sched:sched_process_free                     [100.00%]
                21      sched:sched_process_exit                     [100.00%]
                37      sched:sched_process_wait                     [100.00%]
                20      sched:sched_process_fork                     [100.00%]
                41      sched:sched_process_exec                     [100.00%]
                 0      sched:sched_process_hang

       5.001391328 seconds time elapsed

```

Neat. So there were 20 fork()s and 21 exit()s during 5 seconds. I used -a to match on all CPUs, and `sleep 5` as a dummy command to set the counting duration. You can use `-p PID` instead, to match a process, and skip the sleep command so that `perf` runs until Ctrl-C.

This works for any event (see `perf` `list`) including all static and dynamic tracepoints.

### Syscall Counts

Here's syscalls:

```
# perf stat -e 'syscalls:sys_enter_*' -a sleep 5 | awk '$1'
# started on Wed Jul  2 23:39:50 2014
 Performance counter stats for 'system wide':
                60      syscalls:sys_enter_socket                    [99.95%]
                68      syscalls:sys_enter_connect                    [99.95%]
               148      syscalls:sys_enter_epoll_wait                    [99.97%]
                 8      syscalls:sys_enter_statfs                    [99.97%]
                18      syscalls:sys_enter_dup2                      [99.98%]
                18      syscalls:sys_enter_getcwd                    [99.98%]
               155      syscalls:sys_enter_select                    [99.98%]
               226      syscalls:sys_enter_poll                      [99.98%]
[...]

```

I used awk to strip out syscalls that had zero counts. If that doesn't work for you, add "-o /dev/stdout" to the `perf` command, which can be needed for older versions (eg, Linux 3.2). Another issue I've had with older versions is the need to increase the file descriptor limit (ulimit -n), when matching this many probes.

This one-liner can be a quick way to determine which syscalls are in use, before moving to using `perf` `record` for a closer look, eg, with arguments and stack traces.

### Interval Summary

`perf` `stat` can also print an interval summary in more recent versions, using `-I` and a duration in milliseconds. For example, showing the per-second rate of context switches:

```
# perf stat -I 1000 -e sched:sched_switch -a sleep 5
#           time             counts unit events
     1.000205453                314      sched:sched_switch
     2.000456051                290      sched:sched_switch
     3.000644420                322      sched:sched_switch
     4.000836009                305      sched:sched_switch
     5.001022382                198      sched:sched_switch
     5.001442140                  7      sched:sched_switch

```

Nice. Again, this works for any event.

### By CPU-ID

You can also decompose by CPU id:

```
# perf stat -A -e sched:sched_switch -a sleep 5

 Performance counter stats for 'system wide':

CPU0                   495      sched:sched_switch
CPU1                   632      sched:sched_switch

       5.001377216 seconds time elapsed

```

### Filtering

Finally, --filter can be added to only increment the counter based on a boolean test. For example, counting read() syscalls where the requested size is greater than 4 Kbytes:

```
# perf stat -e syscalls:sys_enter_read --filter 'count > 4096' -a sleep 5

 Performance counter stats for 'system wide':

                 1      syscalls:sys_enter_read

       5.001407932 seconds time elapsed

```

Notice that I used a "count" variable. Where did that come from, and what else is there?

You can figure out many of the variables by reading the static tracepoint definitions in the kernel source (see the example in my [static tracepoints](http://www.brendangregg.com/blog/2014-06-29/perf-static-tracepoints.html) post). Another way is to print their format files:

```
# cat /sys/kernel/debug/tracing/events/syscalls/sys_enter_read/format
name: sys_enter_read
ID: 509
format:
    field:unsigned short common_type;   offset:0;   size:2; signed:0;
    field:unsigned char common_flags;   offset:2;   size:1; signed:0;
    field:unsigned char common_preempt_count;   offset:3;   size:1; signed:0;
    field:int common_pid;   offset:4;   size:4; signed:1;

    field:int nr;   offset:8;   size:4; signed:1;
    field:unsigned int fd;  offset:16;  size:8; signed:0;
    field:char * buf;   offset:24;  size:8; signed:0;
    field:size_t count; offset:32;  size:8; signed:0;

print fmt: "fd: 0x%08lx, buf: 0x%08lx, count: 0x%08lx", ((unsigned long)(REC->fd)), ((unsigned long)(REC->buf)), ((unsigned long)(REC->count))

```

If it's a dynamic tracepoint, you can use `perf` `probe` `-V` to list available variables. I wish there was a similar option for static tracepoints, as well.

So that's a quick tour of basic perf\_events counting capabilities. More is possible; see my [perf\_events](/perf.html) page and the [perf\_events wiki](https://perf.wiki.kernel.org/index.php/Main_Page).

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
