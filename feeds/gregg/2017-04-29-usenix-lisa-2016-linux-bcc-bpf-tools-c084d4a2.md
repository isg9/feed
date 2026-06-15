---
title: USENIX/LISA 2016 Linux bcc/BPF Tools
url: https://www.brendangregg.com/blog/2017-04-29/usenix-lisa-2016-bcc-bpf-tools.html
published: "2017-04-29T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2017-04-29/usenix-lisa-2016-bcc-bpf-tools.html
---

# USENIX/LISA 2016 Linux bcc/BPF Tools

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

## USENIX/LISA 2016 Linux bcc/BPF Tools

29 Apr 2017

For USENIX LISA 2016 I gave a talk that was years in the making, on Linux bcc/BPF analysis tools.

> "Time to rethink the kernel" - Thomas Graf

Thomas has been using BPF to create new network and application security technologies (project [Cilium](https://github.com/cilium/cilium)), and build something that's starting to look like microservices in the kernel ( [video](https://www.youtube.com/watch?v=ilKlmTDdFgk)). I'm using it for advanced performance analysis tools that do tracing and profiling. Enhanced BPF might still be new, but it's already delivering new technologies, and making us rethink what we can do with the kernel.

My LISA 2016 talk begins with a 15 minute demo, showing the progression from ftrace, then perf\_events, to BPF (due to the audio/video settings, this demo is a little hard to follow in the full video, but there's a separate recording of just the demo here: [Linux tracing 15 min demo](https://www.youtube.com/watch?v=GsMs3n8CB6g)). Below is the full talk video ( [youtube](https://www.youtube.com/watch?v=UmOU3I36T2U&t=1151s)):

Here are the [slides](/Slides/LISA2016_BPF_tools_16_9.pdf) or as a [PDF](/Slides/LISA2016_BPF_tools_16_9.pdf):

firstprevnextlast/permalink/zoom

## Installing bcc/BPF

To try out BPF for performance analysis you'll need to be on a newer kernel: at least 4.4, preferably 4.9. The main front end is currently [bcc](https://github.com/iovisor/bcc) (BPF compiler collection), and there are [install instructions](https://github.com/iovisor/bcc/blob/master/INSTALL.md) on github, which keep getting improved. For Ubuntu, installation is:

```
echo "deb [trusted=yes] https://repo.iovisor.org/apt/xenial xenial-nightly main" | sudo tee /etc/apt/sources.list.d/iovisor.list
sudo apt-get update
sudo apt-get install bcc-tools

```

There's currently a pull request to add snap instructions, as there are nightly builds for snappy as well.

## Listing bcc/BPF Tools

This install will add various performance analysis and debugging tools to /usr/share/bcc/tools. Since some require a very recent kernel (4.6, 4.7, or 4.9), there's a subdirectory, /usr/share/bcc/tools/old, which has some older versions of the same tools that work on Linux 4.4 (albeit with some caveats).

```
# ls /usr/share/bcc/tools
argdist       cpudist            filetop         offcputime   solisten    tcptop    vfsstat
bashreadline  cpuunclaimed       funccount       offwaketime  sslsniff    tplist    wakeuptime
biolatency    dcsnoop            funclatency     old          stackcount  trace     xfsdist
biosnoop      dcstat             gethostlatency  oomkill      stacksnoop  ttysnoop  xfsslower
biotop        deadlock_detector  hardirqs        opensnoop    statsnoop   ucalls    zfsdist
bitesize      doc                killsnoop       pidpersec    syncsnoop   uflow     zfsslower
btrfsdist     execsnoop          llcstat         profile      tcpaccept   ugc
btrfsslower   ext4dist           mdflush         runqlat      tcpconnect  uobjnew
cachestat     ext4slower         memleak         runqlen      tcpconnlat  ustat
cachetop      filelife           mountsnoop      slabratetop  tcplife     uthreads
capable       fileslower         mysqld_qslower  softirqs     tcpretrans  vfscount

```

Just by listing the tools, you might spot something you want to start with (ext4\*, tcp\*, etc). Or you can browse the following diagram:

[![](http://www.brendangregg.com/Perf/bcc_tracing_tools.png)](/Perf/bcc_tracing_tools.png)

## Using bcc/BPF

If you don't have a good starting point, in the [bcc Tutorial](https://github.com/iovisor/bcc/blob/master/docs/tutorial.md) I included a generic checklist of the first ten tools to try. I also included this in my LISA talk:

1. execsnoop
2. opensnoop
3. ext4slower (or btrfs\*, xfs\*, zfs\*)
4. biolatency
5. biosnoop
6. cachestat
7. tcpconnect
8. tcpaccept
9. tcpretrans
10. runqlat
11. profile

Most of these have usage messages, and are easy to use. They'll need to be run as root. For example, `execsnoop` to trace new processes:

```
# /usr/share/bcc/tools/execsnoop
PCOMM            PID    PPID   RET ARGS
grep             69460  69458    0 /bin/grep -q g2.
grep             69462  69458    0 /bin/grep -q p2.
ps               69464  58610    0 /bin/ps -p 308
ps               69465  100871   0 /bin/ps -p 301
sleep            69466  58610    0 /bin/sleep 1
sleep            69467  100871   0 /bin/sleep 1
run              69468  5160     0 ./run
[...]

```

And `biolatency` to record an in-kernel histogram of disk I/O latency:

```
# /usr/share/bcc/tools/biolatency
Tracing block device I/O... Hit Ctrl-C to end.
^C
     usecs               : count     distribution
         0 -> 1          : 0        |                                        |
         2 -> 3          : 0        |                                        |
         4 -> 7          : 0        |                                        |
         8 -> 15         : 0        |                                        |
        16 -> 31         : 0        |                                        |
        32 -> 63         : 0        |                                        |
        64 -> 127        : 0        |                                        |
       128 -> 255        : 0        |                                        |
       256 -> 511        : 64       |**********                              |
       512 -> 1023       : 248      |****************************************|
      1024 -> 2047       : 29       |****                                    |
      2048 -> 4095       : 18       |**                                      |
      4096 -> 8191       : 42       |******                                  |
      8192 -> 16383      : 20       |***                                     |
     16384 -> 32767      : 3        |                                        |

```

Here's its USAGE message:

```
# /usr/share/bcc/tools/biolatency -h
usage: biolatency [-h] [-T] [-Q] [-m] [-D] [interval] [count]

Summarize block device I/O latency as a histogram

positional arguments:
  interval            output interval, in seconds
  count               number of outputs

optional arguments:
  -h, --help          show this help message and exit
  -T, --timestamp     include timestamp on output
  -Q, --queued        include OS queued time in I/O time
  -m, --milliseconds  millisecond histogram
  -D, --disks         print a histogram per disk device

examples:
    ./biolatency            # summarize block I/O latency as a histogram
    ./biolatency 1 10       # print 1 second summaries, 10 times
    ./biolatency -mT 1      # 1s summaries, milliseconds, and timestamps
    ./biolatency -Q         # include OS queued time in I/O time
    ./biolatency -D         # show each disk device separately

```

In /usr/share/bcc/tools/docs or the [tools subdirectory](https://github.com/iovisor/bcc/tree/master/tools) on github, you'll find \_example.txt files for every tool which have screenshots and discussion. Check them out! There are also man pages under man/man8.

For more information, please watch my LISA talk at the top of this post when you get a chance, where I explain Linux tracing, BPF, bcc, and tour various tools.

## What's Next?

My prior talk at LISA 2014 was [New Tools and Old Secrets (perf-tools)](/blog/2015-03-17/usenix-lisa-2014-linux-ftrace-perf-tools.html), where I showed similar performance analysis tools using ftrace, an older tracing framework in Linux. I'm still using ftrace, not just for older kernels, but for times where it's more efficient (eg, kernel function counting using the `funccount` tool). BPF is programmatic, and can do things that ftrace can't.

Doing ftrace at LISA 2014, then BPF at LISA 2016, you might wonder I'll propose for LISA 2018. We'll see. I could be covering a higher-level BPF front-end (eg, [ply](https://github.com/iovisor/ply), if it gets finished), or a BPF GUI (eg, via Netflix Vector), or I could be focused on something else entirely. Tracing was my priority when Linux lacked various capabilities, but now that's done, there are other important technologies to work on...

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
