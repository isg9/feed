---
title: 'SREcon: Performance Checklists for SREs 2016'
url: https://www.brendangregg.com/blog/2016-05-04/srecon2016-perf-checklists-for-sres.html
published: "2016-05-04T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-05-04/srecon2016-perf-checklists-for-sres.html
---

# SREcon: Performance Checklists for SREs 2016

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

## SREcon: Performance Checklists for SREs 2016

04 May 2016

When Netflix is down, minutes matter, and there's little time for traditional performance engineering. At [SREcon16 Santa Clara](https://www.usenix.org/conference/srecon16) I gave the closing address on performance checklists for SREs. Checklists are vital for this kind of work, and are often implemented at Netflix as custom dashboards of selected metrics.

This was my first talk about my SRE work at Netflix, where I've joined the on-call rotation for the CORE incident response team. I began by summarizing the difference between performance engineering (where I spend most of my time, and is the team I'm on), and SRE incident response for performance issues.

The video is on [youtube](https://www.youtube.com/watch?v=zxCWXNigDpA) and [usenix.org](https://www.usenix.org/conference/srecon16/program/presentation/gregg):

And the slides are on [slideshare](http://www.slideshare.net/brendangregg/srecon-2016-performance-checklists-for-sres):

I summarized a dozen checklists in talk, as well as methodologies to derive them. They are roughly sorted in intended order of use: starting with cloud-wide dashboards and ending with Linux specific checklists.

The first two checklists are our Performance and Reliability Engineering (PRE) Triage Checklist, a shared document, and then predash, a custom dashboard. These are Netflix specific, and show how we begin this type of analysis. I thought for a moment that they were too specific to Netflix, but wanted to include them anyway for completeness.

I've reproduced the Linux checklists below, which should be implemented as GUI dashboards. Check the presentation for eight other checklists.

## 6\. Linux Perf Analysis in 60s

1. **`uptime`** ⟶ load averages
2. **`dmesg -T | tail`** ⟶ kernel errors
3. **`vmstat 1`** ⟶ overall stats by time
4. **`mpstat -P ALL 1`** ⟶ CPU balance
5. **`pidstat 1`** ⟶ process usage
6. **`iostat -xz 1`** ⟶ disk I/O
7. **`free -m`** ⟶ memory usage
8. **`sar -n DEV 1`** ⟶ network I/O
9. **`sar -n TCP,ETCP 1`** ⟶ TCP stats
10. **`top`** ⟶ check overview

These are explained in the post [Linux Performance Analysis in 60 seconds](http://techblog.netflix.com/2015/11/linux-performance-analysis-in-60s.html) from the Netflix tech blog.

## 7\. Linux Disk Checklist

1. **`iostat -xz 1`** ⟶ any disk I/O? if not, stop looking
2. **`vmstat 1 `** ⟶ is this swapping? or, high sys time?
3. **`df -h `** ⟶ are file systems nearly full?
4. **`ext4slower 10`** ⟶ (zfs\*, xfs\*, etc.) slow file system I/O?
5. **`bioslower 10`** ⟶ if so, check disks
6. **`ext4dist 1`** ⟶ check distribution and rate
7. **`biolatency 1`** ⟶ if interesting, check disks
8. **`cat /sys/devices/…/ioerr_cnt `** ⟶ (if available) errors
9. **`smartctl -l error /dev/sda1 `** ⟶ (if available) errors

Another short checklist. Won't solve everything. ext4slower/dist, bioslower/latency, are from [bcc/BPF tools](https://github.com/iovisor/bcc).

## 8\. Linux Network Checklist

1. **`sar -n DEV,EDEV 1`** ⟶ at interface limits? or use nicstat
2. **`sar -n TCP,ETCP 1`** ⟶ active/passive load, retransmit rate
3. **`cat /etc/resolv.conf`** ⟶ it's always DNS
4. **`mpstat -P ALL 1`** ⟶ high kernel time? single hot CPU?
5. **`tcpretrans`** ⟶ what are the retransmits? state?
6. **`tcpconnect`** ⟶ connecting to anything unexpected?
7. **`tcpaccept`** ⟶ unexpected workload?
8. **`netstat -rnv`** ⟶ any inefficient routes?
9. **check firewall config** ⟶ anything blocking/throttling?
10. **`netstat -s`** ⟶ play 252 metric pickup

tcp\*, are from bcc/BPF tools.

## 9\. Linux CPU Checklist

1. **`uptime`** ⟶ load averages
2. **`vmstat 1`** ⟶ system-wide utilization, run q length
3. **`mpstat -P ALL 1`** ⟶ CPU balance
4. **`pidstat 1`** ⟶ per-process CPU
5. **CPU flame graph** ⟶ CPU profiling
6. **CPU subsecond offset heat map** ⟶ look for gaps
7. **`perf stat -a -- sleep 10`** ⟶ IPC, LLC hit ratio

htop can do 1-4. I'm tempted to add execsnoop for short-lived processes (it's in [perf-tools](https://github.com/brendangregg/perf-tools) or bcc/BPF tools).

For more about SRE at Netflix, see my colleague Jonah Horowitz's talk [Netflix: 190 Countries and 5 CORE SREs](https://www.usenix.org/conference/srecon16/program/presentation/horowitz). We're also hiring SREs (keep an eye on Netflix [jobs](https://jobs.netflix.com/jobs)). For other talks about SRE (Site Reliability Engineering), see the SREcon16 [program](https://www.usenix.org/conference/srecon16/program).

This was my first SREcon and I found it very useful and informative, particularly to see what SRE really means to different companies. Thanks to USENIX and the organizers for a great conference!

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
