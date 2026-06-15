---
title: Linux 4.9's Efficient BPF-based Profiler
url: https://www.brendangregg.com/blog/2016-10-21/linux-efficient-profiler.html
published: "2016-10-21T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-10-21/linux-efficient-profiler.html
---

# Linux 4.9's Efficient BPF-based Profiler

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

## Linux 4.9's Efficient BPF-based Profiler

21 Oct 2016

BPF-optimized profiling arrived in Linux 4.9-rc1 ( [patchset](https://lkml.org/lkml/2016/9/1/831)), allowing the kernel to profile via timed sampling and summarize stack traces. Here's how I explained it recently in a talk alongside other prior perf methods for CPU [flame graph](http://www.brendangregg.com/flamegraphs.html) generation:

![](/blog/images/2016/linux-profiling-perf-bpf.png)

Linux 4.9 skips needing the perf.data file entirely, and its associated overheads. I wrote about this as a missing BPF feature [in March](/blog/2016-03-28/linux-bpf-bcc-road-ahead-2016.html). It is now done.

In detail:

- **Linux 2.6**: `perf record` dumps all stack samples to a binary perf.data file for post-processing in user space. This is copied via a ring buffer, and perf wakes up an optimal number of times to read that buffer, so this has already been optimized somewhat. With low frequency sampling (<50 Hertz), it would be hard to measure the runtime overhead. The post-processing, however, could take up to several seconds of CPU time (and some disk I/O for perf.data) for a busy application and many CPUs.
- **Linux 4.5**: The prior use of stackcollapse-perf.pl was to frequency count stack traces, but really, the perf command has this capability, it just couldn't emit output in the folded format (stack traces on single lines, with frames delimitered by ";"). Linux 4.5 added a "-g folded" option to perf report, making this workflow more efficient. I blogged about this [previously](/blog/2016-04-30/linux-perf-folded.html).
- **Linux 4.9**: BPF was enhanced (eBPF) to attach to perf\_events, which are created by perf\_event\_open() with a PERF\_EVENT\_IOC\_SET\_BPF ioctl() to attach a BPF program. Voilà! Timed samples can now run BPF programs. BPF already had the capability to walk stack traces and frequency count them, added in Linux 4.6, we just needed this capability to attach it to timed samples.

I developed profile.py for [bcc tools](https://github.com/iovisor/bcc) as a BPF profiler (see [example output](https://github.com/iovisor/bcc/blob/master/tools/profile_example.txt)), which uses this new Linux 4.9 support. Here's a truncated screenshot:

```
# ./profile.py 10
Sampling at 49 Hertz of all threads by user + kernel stack for 10 secs.
[...]
    ffffffff81414025 copy_user_enhanced_fast_string
    ffffffff817b0774 tcp_sendmsg
    ffffffff817dd145 inet_sendmsg
    ffffffff8173ea48 sock_sendmsg
    ffffffff8173eae5 sock_write_iter
    ffffffff81232db3 __vfs_write
    ffffffff81233438 vfs_write
    ffffffff812348a5 sys_write
    ffffffff818737bb entry_SYSCALL_64_fastpath
    00007fae0a2614fd [unknown]
    0000000000020000 [unknown]
    -                iperf (21262)
        132

    ffffffff81414025 copy_user_enhanced_fast_string
    ffffffff8174e76b skb_copy_datagram_iter
    ffffffff817addb3 tcp_recvmsg
    ffffffff817dd0ae inet_recvmsg
    ffffffff8173e65d sock_recvmsg
    ffffffff8173e89a SYSC_recvfrom
    ffffffff8173fb9e sys_recvfrom
    ffffffff818737bb entry_SYSCALL_64_fastpath
    00007f9c16cf58bf recv
    -                iperf (21266)
        200

```

The output are stack traces, process details, and a single number: the number of times this stack trace was sampled. The entire stack trace is used as a key in an in-kernel hash, so that only the summary is emitted to user space.

You can use my profile tool to output folded format directly, for flame graph generation. Here I'm also including -d, for stack delimiters, which the flame graph software will color grey:

```
# ./profile.py -df 10
[...]
iperf;[unknown];[unknown];-;entry_SYSCALL_64_fastpath;sys_write;vfs_write;__vfs_write;sock_write_iter;sock_sendmsg;inet_sendmsg;tcp_sendmsg;copy_user_enhanced_fast_string 140
iperf;recv;-;entry_SYSCALL_64_fastpath;sys_recvfrom;SYSC_recvfrom;sock_recvmsg;inet_recvmsg;tcp_recvmsg;skb_copy_datagram_iter;copy_user_enhanced_fast_string 177

```

This can be read directly by flamegraph.pl -- it doesn't need stackcollapse-perf.pl (as pictured in the slide).

Here's the USAGE message for profile, which shows other capabilities:

```
# ./profile.py -h
usage: profile.py [-h] [-p PID] [-U | -K] [-F FREQUENCY] [-d] [-a] [-f]
                  [--stack-storage-size STACK_STORAGE_SIZE]
                  [duration]

Profile CPU stack traces at a timed interval

positional arguments:
  duration              duration of trace, in seconds

optional arguments:
  -h, --help            show this help message and exit
  -p PID, --pid PID     profile this PID only
  -U, --user-stacks-only
                        show stacks from user space only (no kernel space
                        stacks)
  -K, --kernel-stacks-only
                        show stacks from kernel space only (no user space
                        stacks)
  -F FREQUENCY, --frequency FREQUENCY
                        sample frequency, Hertz (default 49)
  -d, --delimited       insert delimiter between kernel/user stacks
  -a, --annotations     add _[k] annotations to kernel frames
  -f, --folded          output folded format, one line per stack (for flame
                        graphs)
  --stack-storage-size STACK_STORAGE_SIZE
                        the number of unique stack traces that can be stored
                        and displayed (default 2048)

examples:
    ./profile             # profile stack traces at 49 Hertz until Ctrl-C
    ./profile -F 99       # profile stack traces at 99 Hertz
    ./profile 5           # profile at 49 Hertz for 5 seconds only
    ./profile -f 5        # output in folded format for flame graphs
    ./profile -p 185      # only profile threads for PID 185
    ./profile -U          # only show user space stacks (no kernel)
    ./profile -K          # only show kernel space stacks (no user)

```

I have an older version of this tool in the bcc tools/old directory, which works on Linux 4.6 to 4.8 using an old kprobe trick, although one that had caveats.

Thanks to Alexei Starovoitov (Facebook) for getting the kernel BPF profiling support done, and Teng Qin (Facebook) for adding bcc support yesterday.

We could have developed BPF profiling sooner, but since the 2.6 style of profiling was somewhat sufficient, BPF effort has been on other higher priority areas. But now that BPF profiling has arrived, it means more than just saving CPU (and disk I/O) resources: imagine real-time flame graphs!

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
