---
title: 'CPI Flame Graphs: Catching Your CPUs Napping'
url: https://www.brendangregg.com/blog/2014-10-31/cpi-flame-graphs.html
published: "2014-10-31T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-10-31/cpi-flame-graphs.html
---

# CPI Flame Graphs: Catching Your CPUs Napping

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

## CPI Flame Graphs: Catching Your CPUs Napping

31 Oct 2014

When you see high CPU usage, you may be forgiven for believing that your CPUs must be furiously executing software, like obedient robots that never rest. In reality, you don't know what's happening from the CPU utilization alone. Your CPUs might be napping, during an instruction, while they wait for some resource.

Understanding what the CPUs are really doing helps direct performance tuning. The simplest way is to measure the average cycles-per-instruction (CPI) ratio: higher values mean it takes more cycles to complete instructions (often "stalled cycles" waiting on memory I/O). This is usually studied as a system-wide metric.

The following new visualization takes a [CPU flame graph](/FlameGraphs/cpuflamegraphs.html) and then assigns colors relative to CPI:

![](/blog/images/2014/cpi-freebsd-kernel.svg)

Wow! For the first time I can *see* where the stall cycles are, providing a clear visualization of CPU efficiency by function.

The width of each frame shows the time a function (or its children) was on-CPU, as with a normal CPU flame graph. The color now shows what that function was doing when it was on-CPU: running or stalled.

The color range in this example has been scaled to color the highest CPI blue (slowest instructions), and lowest CPI red (fastest instructions). Flame graphs also now do click to zoom (thanks Adrien Mahieux), as well as mouse-overs. (Direct [SVG](/blog/images/2014/cpi-freebsd-kernel.svg) link.)

This example is showing a FreeBSD kernel. The vm\_\* (virtual memory) functions have the slowest instructions, which is expected as they involve memory I/O to walk page tables, which may not cache as well as other workloads. The fastest instructions were in \_\_mtx\_lock\_sleep, which is also expected as it is a spin loop.

The most color saturated frames on the left (vm\_page\_alloc->\_\_mtx\_lock\_sleep) are being worked on by a fellow engineer in our OCA development team. Had that not already been the case, this CPI flame graph should have prompted their study and development.

There are two parts that make this possible: differential flame graphs, and measuring stacks on instructions and stall cycles. I'll briefly summarize these, with the latter using pmcstat(8) on FreeBSD (this approach is also possible on Linux, provided you have PMC access, which my current cloud instances don't!).

## Differential Flame Graphs

This is a new feature which I'll explain in a separate post (I also want to add some more options first). In summary, flamegraph.pl usually takes input like the following:

```
func_a;func_b;func_c 31
...

```

These are single line entries with a semi-colon delimited stack followed by a sample count.

A differential flame graph will be generated if you provide the following input:

```
func_a;func_b;func_c 31 35
...

```

This now has two value columns, which are intended to show before and after counts. The flame graph is sized using the second column (the "after", or "now"), and then colored using the delta of 2 - 1: positive is red, negative is blue. The difffolded.pl tool takes two folded-style profiles (generated using the stackcollapse scripts) and emits this two-column output.

For CPI flame graphs, the two value columns are:

- stall cycle count
- unhalted cycle count

The first is scaled so that its average is the same as the second column, and the full blue-red range is used. Otherwise, there's always more unhalted cycles, and the differential flame graph is just red.

## FreeBSD pmcstat

The data in this example was collected using FreeBSD's [pmcstat(8)](https://www.freebsd.org/cgi/man.cgi?query=pmcstat) using the following (on Intel):

```
# pmcstat -n 1000000 -S RESOURCE_STALLS.ANY -S CPU_CLK_UNHALTED.THREAD_P -O out.pmclog_cpi01 sleep 10
# pmcstat -R out.pmclog_cpi01 -z 32 -G out.pmcstat_cpi01
CONVERSION STATISTICS:
 #exec/elf                                12
 #samples/total                           123464
 #samples/unknown-function                2059
 #callchain/dubious-frames                4723
# ls -lh *cpi01
-rw-r--r--  1 root  wheel    14M Oct 29 04:24 out.pmclog_cpi01
-rw-r--r--  1 root  wheel   7.2M Oct 29 04:24 out.pmcstat_cpi01

```

Be careful with pmcstat(8): you can generate a lot of output quickly, which in turn can perturb the performance of what you are measuring. I only measured for 10 seconds (sleep 10), and only one event every million (-n 1000000). Despite this, the raw output file is 14 Mbytes.

In this example I measured kernel stacks (-S), and up to 32 frames deep. Note that I did need to set kern.hwpmc.callchaindepth=32 in /boot/loader.conf and reboot, as my stacks were at first always being truncated to 8 frames.

pmstat -G was used to create a callchain text file. It looks like this:

```
# more out.pmcstat_cpi01
@ CPU_CLK_UNHALTED.THREAD_P [10867 samples]

07.86%  [854]      __mtx_lock_sleep @ /boot/kernel/kernel
 48.95%  [418]       vm_page_alloc
  99.76%  [417]        vm_page_grab
   99.04%  [413]         allocbuf
    100.0%  [413]          getblk
     95.16%  [393]           cluster_rbuild
      100.0%  [393]            cluster_read
       100.0%  [393]             ffs_read
        100.0%  [393]              VOP_READ_APV
         100.0%  [393]               vn_read
          100.0%  [393]                vn_io_fault
           100.0%  [393]                 aio_daemon
            100.0%  [393]                  fork_exit
     04.84%  [20]            cluster_read
      100.0%  [20]             ffs_read
[...]
@ CPU_CLK_UNHALTED.THREAD_P [10856 samples]
[...]
@ RESOURCE_STALLS.ANY [3462 samples]

06.15%  [213]      sf_buf_mext @ /boot/kernel/kernel
 100.0%  [213]       mb_free_ext
  100.0%  [213]        m_freem
   54.46%  [116]         reclaim_tx_descs
    100.0%  [116]          t4_eth_tx
[...]

```

For each counter, and then each CPU, the samples are printed as call chains.

## CPI Flame Graph

To make a CPI flame graph from this call chain output, I needed to separate out the counters into separate files. I did this in about 60 seconds using vi, creating:

- out.pmcstat\_cycles: all the sections titled CPU\_CLK\_UNHALTED.THREAD\_P
- out.pmcstat\_stalls: all the sections titled RESOURCE\_STALLS.ANY

Yes, I should just write some Perl/awk to do this. I will next time.

I then converted these to the folded format, using stackcollapse-pmc.pl (by Ed Maste):

```
$ cat out.pmcstat_cycles | ./stackcollapse-pmc.pl > out.pmcstat_cycles.folded
$ cat out.pmcstat_stalls | ./stackcollapse-pmc.pl > out.pmcstat_stalls.folded

```

Finally, making the CPI flame graph:

```
$ ./difffolded.pl -n out.pmcstat_stall.folded out.pmcstat_cycles.folded | \
    ./flamegraph.pl --title "CPI Flame Graph: blue=stalls, red=instructions" --width=900 > cpi.svg

```

That's the [SVG](/blog/images/2014/cpi-freebsd-kernel.svg) at the top of this page.

All the flame graph software (flamegraph.pl, stackcollapse-pmc.pl, difffolded.pl) can be found in the [Flame Graph](https://github.com/brendangregg/FlameGraph) collection on github.

## Conclusion

There is much you can do as either a software developer or system administrator once you know that CPU is either memory-bound or instruction-bound, resulting in small to large performance improvements. CPU flame graphs are already a great way to visualize how your software is using the CPUs. CPI flame graphs use color to show what the CPUs are really doing: their efficiency by function.

PS. This weekend I'm speaking at [MeetBSD](https://www.meetbsd.com/) on performance analysis. I will be sure to mention pmcstat(8), which can do a lot more than just stalls and cycles.

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
