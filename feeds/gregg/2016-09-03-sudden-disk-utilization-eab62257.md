---
title: Sudden Disk Utilization
url: https://www.brendangregg.com/blog/2016-09-03/sudden-disk-busy.html
published: "2016-09-03T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-09-03/sudden-disk-busy.html
---

# Sudden Disk Utilization

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

## Sudden Disk Utilization

03 Sep 2016

*These are rough notes.*

Disk %busy went to 80% on a jenkins host. Why? Cloud-wide monitoring (Atlas) showed this for an instance:

[![](/blog/images/2016/diskutil1.png)](/blog/images/2016/diskutil1.png)

Disk utilization was well over 80% and steady (1 minute averages).

I ran some CLI tools:

```
# iostat –x 1
[…]
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           5.37    0.00    0.77    0.00    0.00   93.86

Device:   rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
xvda        0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
xvdb        0.00     0.00    0.00    0.00     0.00     0.00     0.00     0.00    0.00    0.00    0.00   0.00   0.00
xvdj        0.00     0.00  139.00    0.00  1056.00     0.00    15.19     0.88    6.19    6.19    0.00   6.30  87.60
[…]

```

This shows an average disk I/O latency of about 6 milliseconds (await). Not unusual as an average for rotational disks and random disk I/O.

I also used my kernel tracing perf-tools to measure disk I/O latency histograms:

```
# /apps/perf-tools/bin/iolatency 10
Tracing block I/O. Output every 10 seconds. Ctrl-C to end.

  >=(ms) .. <(ms)   : I/O      |Distribution                          |
       0 -> 1       : 421      |######################################|
       1 -> 2       : 95       |#########                             |
       2 -> 4       : 48       |#####                                 |
       4 -> 8       : 108      |##########                            |
       8 -> 16      : 363      |#################################     |
      16 -> 32      : 66       |######                                |
      32 -> 64      : 3        |#                                     |
      64 -> 128     : 7        |#                                     |
^C

```

Looks like 7200 RPM disks, bi-modal, with some queueing. But disks normally behave like this under high load. This points towards a problem of workload and not the target.

```
# /apps/perf-tools/bin/iosnoop
Tracing block I/O. Ctrl-C to end.
COMM         PID    TYPE DEV      BLOCK        BYTES     LATms
java         30603  RM   202,144  1670768496   8192       0.28
cat          6587   R    202,0    1727096      4096      10.07
cat          6587   R    202,0    1727120      8192      10.21
cat          6587   R    202,0    1727152      8192      10.43
java         30603  RM   202,144  620864512    4096       7.69
java         30603  RM   202,144  584767616    8192      16.12
java         30603  RM   202,144  601721984    8192       9.28
java         30603  RM   202,144  603721568    8192       9.06
java         30603  RM   202,144  61067936     8192       0.97
java         30603  RM   202,144  1678557024   8192       0.34
java         30603  RM   202,144  55299456     8192       0.61
java         30603  RM   202,144  1625084928   4096      12.00
java         30603  RM   202,144  618895408    8192      16.99
java         30603  RM   202,144  581318480    8192      13.39
java         30603  RM   202,144  1167348016   8192       9.92
java         30603  RM   202,144  51561280     8192      22.17
[...]

```

Again, looks bi-modal, with I/O taking 10 ms and higher. Mostly java. I should check the stacks that issued I/O, but wait...what's TYPE=M?

I'm printing a standard flag string from the kernel – the rwbs field – which is encoded that way. I saw it asked online once what the characters mean. I replied that they weren't documented, and you had to read the kernel code. They said I was wrong, they were documented in a man page. Huh. Well I’m an idiot. What man page? It was my own man page.

```
# perf record -e block:block_rq_issue --filter rwbs ~ "*M*" -g -a
# perf report -n –stdio
[...]
# Overhead       Samples       Command      Shared Object                Symbol
# ........  ............  ............  .................  ....................
#
    70.70%           251          java  [kernel.kallsyms]  [k] blk_peek_request
                    |
                    --- blk_peek_request
                        do_blkif_request
                        __blk_run_queue
                        queue_unplugged
                        blk_flush_plug_list
                        blk_finish_plug
                        _xfs_buf_ioapply
                        xfs_buf_iorequest
                       |
                       |--88.84%-- _xfs_buf_read
                       |          xfs_buf_read_map
                       |          |
                       |          |--87.89%-- xfs_trans_read_buf_map
                       |          |          |
                       |          |          |--97.96%-- xfs_imap_to_bp
                       |          |          |          xfs_iread
                       |          |          |          xfs_iget
                       |          |          |          xfs_lookup
                       |          |          |          xfs_vn_lookup
                       |          |          |          lookup_real
                       |          |          |          __lookup_hash
                       |          |          |          lookup_slow
                       |          |          |          path_lookupat
                       |          |          |          filename_lookup
                       |          |          |          user_path_at_empty
                       |          |          |          user_path_at
                       |          |          |          vfs_fstatat
                       |          |          |          |
                       |          |          |          |--99.48%-- SYSC_newlstat
                       |          |          |          |          sys_newlstat
                       |          |          |          |          system_call_fastpath
                       |          |          |          |          __lxstat64
                       |          |          |          |Lsun/nio/fs/UnixNativeDispatcher;.lstat0
                       |          |          |          |          0x7f8f963c847c

```

All coming from stat(). The developers then realized they had enabled an app called "DiskUsageMonitor", which constantly stat()s the file system to track used space.

Disabling DiskUsageMonitor fixed the issue immediately:

[![](/blog/images/2016/diskutil2.png)](/blog/images/2016/diskutil2.png)

It's not the first time I've seen a monitoring tool cause a performance issue, and it won't be the last.

As this was a neat short example of using kernel tracing tools, I've used it in a talk at Facebook. It predates eBPF being available on these production hosts, so the perf-tools I was using are perf and Ftrace based.

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
