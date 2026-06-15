---
title: 'bcc: Taming Linux 4.3+ Tracing Superpowers'
url: https://www.brendangregg.com/blog/2015-09-22/bcc-linux-4.3-tracing.html
published: "2015-09-22T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2015-09-22/bcc-linux-4.3-tracing.html
---

# bcc: Taming Linux 4.3+ Tracing Superpowers

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

## bcc: Taming Linux 4.3+ Tracing Superpowers

22 Sep 2015

Here's a quick tour of some new open source tools which I demonstrated at the Silicon Valley Linux Technology meetup last night. These use the new eBPF capabilities added in recent Linux, including Linux 4.3.

Summarizing the distribution of disk I/O latency:

```
# ./biolatency
Tracing block device I/O... Hit Ctrl-C to end.
^C
     usecs           : count     distribution
       0 -> 1        : 0        |                                      |
       2 -> 3        : 0        |                                      |
       4 -> 7        : 0        |                                      |
       8 -> 15       : 0        |                                      |
      16 -> 31       : 0        |                                      |
      32 -> 63       : 0        |                                      |
      64 -> 127      : 1        |                                      |
     128 -> 255      : 12       |********                              |
     256 -> 511      : 15       |**********                            |
     512 -> 1023     : 43       |*******************************       |
    1024 -> 2047     : 52       |**************************************|
    2048 -> 4095     : 47       |**********************************    |
    4096 -> 8191     : 52       |**************************************|
    8192 -> 16383    : 36       |**************************            |
   16384 -> 32767    : 15       |**********                            |
   32768 -> 65535    : 2        |*                                     |
   65536 -> 131071   : 2        |*                                     |

```

Tracing per-disk I/O:

```
# ./biosnoop
TIME(s)        COMM           PID    DISK    T  SECTOR    BYTES   LAT(ms)
0.000004001    supervise      1950   xvda1   W  13092560  4096       0.74
0.000178002    supervise      1950   xvda1   W  13092432  4096       0.61
0.001469001    supervise      1956   xvda1   W  13092440  4096       1.24
0.001588002    supervise      1956   xvda1   W  13115128  4096       1.09
1.022346001    supervise      1950   xvda1   W  13115272  4096       0.98
1.022568002    supervise      1950   xvda1   W  13188496  4096       0.93
1.023534000    supervise      1956   xvda1   W  13188520  4096       0.79
1.023585003    supervise      1956   xvda1   W  13189512  4096       0.60

```

Tracing the open() syscall:

```
# ./opensnoop
PID    COMM               FD ERR PATH
17326  <...>               7   0 /sys/kernel/debug/tracing/trace_pipe
17358  run                 3   0 /lib/x86_64-linux-gnu/libtinfo.so.5
17358  run                 3   0 /lib/x86_64-linux-gnu/libdl.so.2
17358  run                 3   0 /lib/x86_64-linux-gnu/libc.so.6
17358  run                -1   6 /dev/tty
17358  run                 3   0 /proc/meminfo
17358  run                 3   0 /etc/nsswitch.conf

```

Counting VFS operation types:

```
# ./vfsstat
TIME         READ/s  WRITE/s CREATE/s   OPEN/s  FSYNC/s
18:35:35:       241       15        4       99        0
18:35:36:       232       10        4       98        0
18:35:37:       244       10        4      107        0
18:35:38:       235       13        4       97        0
18:35:39:      6749     2633        4     1446        0
18:35:40:       277       31        4      115        0

```

Counting kernel function calls per-second, that match "tcp\*send\*":

```
# ./funccount -i 1 'tcp*send*'
Tracing... Ctrl-C to end.

ADDR             FUNC                          COUNT
ffffffff816d2281 tcp_send_delayed_ack             30
ffffffff816d6c81 tcp_v4_send_check                31
ffffffff816c2f61 tcp_sendmsg                      31
ffffffff816bf851 tcp_send_mss                     31

ADDR             FUNC                          COUNT
ffffffff816d1db1 tcp_send_fin                      3
ffffffff816d0f71 tcp_send_ack                     18
ffffffff816d2281 tcp_send_delayed_ack            214
ffffffff816c2f61 tcp_sendmsg                     231
ffffffff816bf851 tcp_send_mss                    231
ffffffff816d6c81 tcp_v4_send_check               255

ADDR             FUNC                          COUNT
ffffffff816d0f71 tcp_send_ack                      2
ffffffff816d2281 tcp_send_delayed_ack              9
ffffffff816c2f61 tcp_sendmsg                      30
ffffffff816bf851 tcp_send_mss                     30

```

Timing tcp\_sendmsg() latency (call duration), in microseconds:

```
# ./funclatency -u tcp_sendmsg
Tracing tcp_sendmsg... Hit Ctrl-C to end.
^C
     usecs           : count     distribution
       0 -> 1        : 20778    |**************************************|
       2 -> 3        : 15429    |****************************          |
       4 -> 7        : 355      |                                      |
       8 -> 15       : 171      |                                      |
      16 -> 31       : 106      |                                      |
      32 -> 63       : 9        |                                      |
Detaching...

```

... all of these tools have man pages, and most have help messages as well:

```
# ./funclatency -h
usage: funclatency [-h] [-p PID] [-i INTERVAL] [-T] [-u] [-m] [-r] pattern

Time kernel funcitons and print latency as a histogram

positional arguments:
  pattern               search expression for kernel functions

optional arguments:
  -h, --help            show this help message and exit
  -p PID, --pid PID     trace this PID only
  -i INTERVAL, --interval INTERVAL
                        summary interval, seconds
  -T, --timestamp       include timestamp on output
  -u, --microseconds    microsecond histogram
  -m, --milliseconds    millisecond histogram
  -r, --regexp          use regular expressions. Default is "*" wildcards
                        only.

examples:
    ./funclatency do_sys_open       # time the do_sys_open() kenel function
    ./funclatency -u vfs_read       # time vfs_read(), in microseconds
    ./funclatency -m do_nanosleep   # time do_nanosleep(), in milliseconds
    ./funclatency -mTi 5 vfs_read   # output every 5 seconds, with timestamps
    ./funclatency -p 181 vfs_read   # time process 181 only
    ./funclatency 'vfs_fstat*'      # time both vfs_fstat() and vfs_fstatat()

```

## Linux 4.3+, eBPF

What's new in Linux 4.3 is the ability to print strings from Extended Berkeley Packet Filters (eBPF) programs. This is only a small addition, but one I needed for many tools. eBPF is a virtual machine for running user-defined, sandboxed bytecode, with maps for data storage. I wrote about it in [eBPF: One Small Step](http://www.brendangregg.com/blog/2015-05-15/ebpf-one-small-step.html).

eBPF enhances Linux tracing, allowing mini programs to be executed on tracing events. In my tools above, eBPF lets me tag events with custom timestamps, store histograms, filter events, and only emit summarized info to user-level. These capabilities give me the info I want, with the lowest possible overhead cost.

While eBPF provides amazing superpowers, there is a catch: it's hard to use via its assembly or C interface. The challenge attracts me, but it can be a brutal experience, especially if you write eBPF assembly directly (eg, see bpf\_insn\_prog\[\] from [sock\_example.c](https://github.com/torvalds/linux/blob/master/samples/bpf/sock_example.c); I've yet to code one of these from scratch that compiles). The C interface is better (see other examples in [samples/bpf](https://github.com/torvalds/linux/tree/master/samples/bpf)), but it's still laborious and difficult to use.

## Enter bcc

[![](/blog/images/2015/tracing_landscape_sep2015.png)](http://www.slideshare.net/brendangregg/velocity-2015-linux-perf-tools)

The BPF Compiler Collection ( [bcc](https://github.com/iovisor/bcc)) project provides a front-end for eBPF, making it easier to write programs. It uses C for the back-end instrumentation, and Python for the front-end interface. My tools at the start of this post all use bcc, and can be found in the [tools directory](https://github.com/iovisor/bcc/tree/master/tools) of bcc on github. Also browse that directory for the \_example.txt files.

I've modified my diagram on the right (from [Velocity 2015](http://www.slideshare.net/brendangregg/velocity-2015-linux-perf-tools/136)) to show the role that bcc plays: it improves the useability of eBPF.

It's still early days for bcc, and right now it isn't easy to setup and use, even once you're on Linux 4.3+ (eg, here are my own [notes](https://gist.github.com/brendangregg/cfa482acb71aa577789c)). In the future, this should be as simple as a package add. And this will be adding user-level software only: the kernel parts (eBPF) are already in the Linux kernel.

Just as one example of bcc, here's the full code to my [biolatency tool](https://github.com/iovisor/bcc/blob/master/tools/biolatency), which I showed at the top of this post:

A lot of this code is logic for processing command-line arguments. The C instrumentation code is defined in-line, as bpf\_text. There are a few things I'd like improved, like switching to tracepoints instead of kprobes, but this is not bad so far. I can write tools in this.

bcc can also do a lot more than my examples here: it can also be used for advanced network traffic control. For more about those capabilities, see the [IO Visor](https://www.iovisor.org/) project.

Even if few people learn bcc programming, it should see success through the use of its tools. At a company like Netflix, it will only take a few of us learning bcc/eBPF to have a big impact for the company: we can develop tools for others to use, and plugins for our other analysis software (eg, [Vector](http://techblog.netflix.com/2015/04/introducing-vector-netflixs-on-host.html)). A lot of what we create is open sourced, so others will benefit as well.

## Other tracers and tools

bcc won't be the only interface to eBPF. There is already work on bringing it to Linux perf\_events. It would also be great to see a tracer with a grammar, like SystemTap or ktap, support eBPF, which would make ad hoc tools even easier to write. (Perhaps bcc will provide its own grammer in the future.) eBPF should make other enhancements possible for the other tracers.

I previously created [perf-tools](https://github.com/brendangregg/perf-tools), a collection of mostly ftrace-based tracing tools for Linux systems. With eBPF and bcc, I'll eventually switch some of these tools to eBPF, where they will have more features, be easier to maintain, and have lower overhead. For example, my perf-tools [iolatency](https://github.com/brendangregg/perf-tools/blob/master/iolatency) tool, which is equivalent to biolatency, processed every disk event in user-level, costing measurable overhead (which I warned about in the tool and its man page). The overhead for the bcc biolatency version should be negligible.

With bcc/eBPF, I'll also be creating many new tools that were previously impossible (or impractical) to do.

*Thanks to Alexei Starovoitov and Brenden Blanco of PLUMgrid, and others, for developing eBPF and bcc, and Deirdré Straughan for edits to this post.*

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
