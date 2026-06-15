---
title: Linux bcc ext4 Latency Tracing
url: https://www.brendangregg.com/blog/2016-10-06/linux-bcc-ext4dist-ext4slower.html
published: "2016-10-06T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-10-06/linux-bcc-ext4dist-ext4slower.html
---

# Linux bcc ext4 Latency Tracing

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

## Linux bcc ext4 Latency Tracing

06 Oct 2016

Storage I/O performance issues are often studied at the block device layer, but instrumenting the file system instead provides more relevant metrics for understanding how applications are affected.

My ext4dist tool does this for the ext4 file system, and traces reads, writes, opens, and fsyncs, and summarizes their latency as a power-of-2 histogram. For example:

```
# ext4dist
Tracing ext4 operation latency... Hit Ctrl-C to end.
^C

operation = 'read'
     usecs               : count     distribution
         0 -> 1          : 1210     |****************************************|
         2 -> 3          : 126      |****                                    |
         4 -> 7          : 376      |************                            |
         8 -> 15         : 86       |**                                      |
        16 -> 31         : 9        |                                        |
        32 -> 63         : 47       |*                                       |
        64 -> 127        : 6        |                                        |
       128 -> 255        : 24       |                                        |
       256 -> 511        : 137      |****                                    |
       512 -> 1023       : 66       |**                                      |
      1024 -> 2047       : 13       |                                        |
      2048 -> 4095       : 7        |                                        |
      4096 -> 8191       : 13       |                                        |
      8192 -> 16383      : 3        |                                        |

operation = 'write'
     usecs               : count     distribution
         0 -> 1          : 0        |                                        |
         2 -> 3          : 0        |                                        |
         4 -> 7          : 0        |                                        |
         8 -> 15         : 75       |****************************************|
        16 -> 31         : 5        |**                                      |

operation = 'open'
     usecs               : count     distribution
         0 -> 1          : 1278     |****************************************|
         2 -> 3          : 40       |*                                       |
         4 -> 7          : 4        |                                        |
         8 -> 15         : 1        |                                        |
        16 -> 31         : 1        |                                        |

```

This output shows a bi-modal distribution for read latency, with a faster mode of less than 7 microseconds, and a slower mode of between 256 and 1023 microseconds. The count column shows how many events fell into that latency range. It's likely that the faster mode was a hit from the in-memory file system cache, and the slower mode is a read from a storage device (disk).

This "latency" is measured from when the operation was issued from the VFS interface to the file system, to when it completed. This spans everything: block device I/O (disk I/O), file system CPU cycles, file system locks, run queue latency, etc. This is a better measure of the latency suffered by applications reading from the file system than measuring this down at the block device interface. Measuring at the block device level is better suited for other uses: resource capacity planning.

Note that this tool only traces the common file system operations previously listed: other file system operations (eg, inode operations including getattr()) are not traced.

ext4dist is a [bcc](https://github.com/iovisor/bcc) tool that uses kernel dynamic tracing (via kprobes) with BPF. bcc is a front-end and a collection of tools that use new Linux enhanced BPF tracing capabilities.

I also wrote ext4slower, to trace these ext4 operations that are slower than a custom threshold. Eg, 1 millisecond:

```
# ext4slower 1
Tracing ext4 operations slower than 1 ms
TIME     COMM           PID    T BYTES   OFF_KB   LAT(ms) FILENAME
06:49:17 bash           3616   R 128     0           7.75 cksum
06:49:17 cksum          3616   R 39552   0           1.34 [
06:49:17 cksum          3616   R 96      0           5.36 2to3-2.7
06:49:17 cksum          3616   R 96      0          14.94 2to3-3.4
06:49:17 cksum          3616   R 10320   0           6.82 411toppm
06:49:17 cksum          3616   R 65536   0           4.01 a2p
06:49:17 cksum          3616   R 55400   0           8.77 ab
06:49:17 cksum          3616   R 36792   0          16.34 aclocal-1.14
06:49:17 cksum          3616   R 15008   0          19.31 acpi_listen
06:49:17 cksum          3616   R 6123    0          17.23 add-apt-repository
06:49:17 cksum          3616   R 6280    0          18.40 addpart
06:49:17 cksum          3616   R 27696   0           2.16 addr2line
06:49:17 cksum          3616   R 58080   0          10.11 ag
06:49:17 cksum          3616   R 906     0           6.30 ec2-meta-data
06:49:17 cksum          3616   R 6320    0          10.00 animate.im6
[...]

```

This is great for proving or exonerating the storage subsystem (file systems, volume managers, and disks) as a source of high latency events. Let's say you had occasional 100 ms application request outliers, and suspected it may be a single slow I/O (of 100 ms) or several (adding to 100 ms). Running "ext4slower 10" would print everything beyond 10 ms, proving or exonerating these theories. (Note that it could be thousands of sub-1 ms I/O, not caught by a "ext4slower 10".)

The output will be far too verbose, but you can also use a threshold of "0" to dump all events:

```
# ext4slower 0
Tracing ext4 operations
TIME     COMM           PID    T BYTES   OFF_KB   LAT(ms) FILENAME
06:58:05 supervise      1884   O 0       0           0.00 status.new
06:58:05 supervise      1884   W 18      0           0.02 status.new
06:58:05 supervise      1884   O 0       0           0.00 status.new
06:58:05 supervise      1884   W 18      0           0.01 status.new
06:58:05 supervise      15817  O 0       0           0.00 run
06:58:05 supervise      15817  R 92      0           0.00 run
06:58:05 supervise      15817  O 0       0           0.00 bash
06:58:05 supervise      15817  R 128     0           0.00 bash
06:58:05 supervise      15817  R 504     0           0.00 bash
06:58:05 supervise      15817  R 28      0           0.00 bash
06:58:05 supervise      15817  O 0       0           0.00 ld-2.19.so
[...]

```

Do run ext4dist first, to check the rate of file system operations. If it's millions per second, then running "ext4slower 0" will try to print millions of lines of output per second – probably not what you want, and will cost overhead on the system.

These tools are in [bcc](https://github.com/iovisor/bcc), and which has man pages (under /man/man8) and text files with more example screenshots (under /tools/\*\_example.txt). Each tool also has a USAGE message, eg:

```
# ext4slower -h
usage: ext4slower [-h] [-j] [-p PID] [min_ms]

Trace common ext4 file operations slower than a threshold

positional arguments:
  min_ms             minimum I/O duration to trace, in ms (default 10)

optional arguments:
  -h, --help         show this help message and exit
  -j, --csv          just print fields: comma-separated values
  -p PID, --pid PID  trace this PID only

examples:
    ./ext4slower             # trace operations slower than 10 ms (default)
    ./ext4slower 1           # trace operations slower than 1 ms
    ./ext4slower -j 1        # ... 1 ms, parsable output (csv)
    ./ext4slower 0           # trace all operations (warning: verbose)
    ./ext4slower -p 185      # trace PID 185 only

```

There are also equivalent tools in bcc for btrfs, xfs, and zfs (so far).

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
