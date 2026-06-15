---
title: perf Heat Maps
url: https://www.brendangregg.com/blog/2014-07-01/perf-heat-maps.html
published: "2014-07-01T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-07-01/perf-heat-maps.html
---

# perf Heat Maps

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

## perf Heat Maps

01 Jul 2014

This is disk latency of an AWS EC2 c3.large instance (ubuntu), shown as a [latency heat map](/HeatMaps/latency.html):

![](/blog/images/2014/out.lat.png)

Woah, aren't these disks supposed to be SSDs?

Mouse over pixels for details. Here is the [SVG](/blog/images/2014/out.lat.svg), and a zoomed [SVG to 15ms](/blog/images/2014/out.latzoom.svg). Time is on the x-axis, and latency on the y-axis. The color shows how many I/O fell into that time and latency range: darker for more.

There is a latency spike at the 63 second mark, which exceeds 237 ms. Yikes. Around the 43 second mark a cloud of latency begins, reaching around 30 ms, which is also worrying for SSDs.

Perhaps there is a simple explanation: load. Bursts of I/O could cause the spikes, thanks to queueing on the device. And the latency cloud could be explained by an increase in the rate of requested I/O at the 43 second mark. I generated line graphs to check these theories:

[![](/blog/images/2014/storageio.png)](/blog/images/2014/storageio.png)

This shows no workload bursts or increases that would explain the changes in latency.

I was creating an example of a disk latency heat map, and as a simple workload I ran tar(1) to archive the entire system. This turned out to be more than I had bargained for. Explaining it is pretty interesting, and demonstrates the different types of disk I/O heat maps you can generate on Linux.

## Latency Heat Map Generation

In my previous post on Linux [perf static tracepoints](http://www.brendangregg.com/blog/2014-06-29/perf-static-tracepoints.html), I showed how the block:block\_rq\_complete probe provided a wealth of useful info by default. Along with the block:block\_rq\_issue probe, you can use the issue to complete tracepoint timestamps to calculate I/O time, or "latency".

Here's how I captured the disk events:

```
# perf record -e block:block_rq_issue -e block:block_rq_complete -a sleep 120
[ perf record: Woken up 217 times to write data ]
[ perf record: Captured and wrote 54.507 MB perf.data (~2381448 samples) ]
# perf script > out.blockio
# more out.blockio
     tar 16072 [001] 2199495.030133: block:block_rq_issue: 202,16 R 0 () 21997888 + 8 [tar]
 swapper     0 [000] 2199495.030286: block:block_rq_complete: 202,16 R () 21997888 + 8 [0]
     tar 16072 [001] 2199495.030327: block:block_rq_issue: 202,16 R 0 () 21997896 + 8 [tar]
 swapper     0 [000] 2199495.030512: block:block_rq_complete: 202,16 R () 21997896 + 8 [0]
[...]

```

This uses `perf` to instrument both tracepoints for 120 seconds, then dumps the capture file to out.blockio: a text file of the data.

Here's the commands I used to generate the previous heatmaps. This includes trace2heatmap.pl, which you can download from my github [HeatMap](https://github.com/brendangregg/HeatMap) project.

```
$ awk '{ gsub(/:/, "") } $5 ~ /issue/ { ts[$6, $10] = $4 }
    $5 ~ /complete/ { if (l = ts[$6, $9]) { printf "%.f %.f\n", $4 * 1000000,
    ($4 - l) * 1000000; ts[$6, $10] = 0 } }' out.blockio > out.lat_us
$ ./trace2heatmap.pl --unitstime=us --unitslat=us --grid out.lat_us > out.lat.svg
$ ./trace2heatmap.pl --unitstime=us --unitslat=us --grid --maxlat=15000 \
    --title="Latency Heat Map: 15ms max" out.lat_us > out.latzoom.svg

```

The trace2heatmap.pl program takes two columns: a timestamp and a latency. I used awk to associate block\_rq\_issue timestamps with block\_rq\_complete, based on the device ID and offset, so that latency can be calculated from their timestamps.

You might need to adjust the field numbers ($4, $5, ...) to match your output, as the perf\_events tracepoints can change between kernel versions. This kernel version is 3.14.5.

If you want, you can combine steps to skip the intermediate files. I find them handy to browse, to look for patterns event by event.

## Size Heat Map

There are two main workload factors that cause SSDs to perform differently: reads vs writes, and I/O size. As this is archiving an entire system, it might have encountered larger files at the 43 second mark, changing the I/O sizes used.

I/O size information is part of the capture I already have: field 11, in sectors. Building an I/O size heat map:

```
$ awk '{ gsub(/:/, "") } $5 ~ /complete/ { print $4 * 1000000, ($11 * 512) }' \
    out.blockio > out.sizes
$ ./trace2heatmap.pl --unitstime=us --unitslat=bytes out.sizes \
    --title="Size Heat Map" > out.sizes.svg

```

Produces ( [SVG](/blog/images/2014/out.sizes.svg)):

![](/blog/images/2014/out.sizes.png)

The y-axis is now I/O size. In this case, that's not consistent with the latency seen. I/O size actually tends to be smaller after 43 seconds, not larger, which should mean quicker I/O...

## Offset Heat Map

It's worth showing the following while I'm here, although it shouldn't make much difference for SSDs: the disk I/O offset heat map. This uses the sector offset field from the block\_rq\_complete probe.

```
$ awk '{ gsub(/:/, "") } $5 ~ /complete/ { print $4 * 1000000, ($9 * 512) / (1024 * 1024) }' \
    out.blockio > out.offset
$ ./trace2heatmap.pl --unitstime=us --unitslat=Mbytes out.offset \
    --title="Offset Heat Map" > out.offset.svg

```

Produces ( [SVG](/blog/images/2014/out.offset.svg)):

![](/blog/images/2014/out.offset.png)

You can use this to quickly identify random disk I/O vs sequential, based on the spread of the offset distribution. For rotational disks this can make a 10x difference to the delivered latency (although to do this properly, you should generate a heat map for each disk). For SSDs it shouldn't matter: no mechanical disk arm to move, or platter to wait for rotation.

Something does happen at the 43 second mark. Before that, I/O looks quite random across the 16 Gbyte range (size of each SSD). After that, the pattern shows more sequential components. In fact, it looks a bit like a file system optimizing placement for rotational disks. Am I sure that I'm only looking at SSDs here?

## SSD Latency Only

My archive workload read all file systems, including the boot drive. For this instance, that's rotational magnetic disks. I can exclude it from my heat map by filtering the device ID, which was "202,1" (you'll need to adjust this to match your instance type):

```
$ awk '{ gsub(/:/, "") } $5 ~ /issue/ && $6 != "202,1" { ts[$6, $10] = $4 }
    $5 ~ /complete/ { if (l = ts[$6, $9]) { printf "%.f %.f\n", $4 * 1000000,
    ($4 - l) * 1000000; ts[$6, $10] = 0 } }' out.blockio > out.ssd_lat
$ ./trace2heatmap.pl --unitstime=us --unitslat=us \
    --title="Latency Heat Map: SSDs only" out.ssd_lat > out.ssd.svg

```

This produces a heat map that shows SSD I/O only ( [SVG](/blog/images/2014/out.ssd.svg)):

![](/blog/images/2014/out.ssd.png)

All right! So the latency cloud and spikes after 43 were fore rotational disk I/O only, not SSD.

## Read Latency Only

I've seen such latency spikes before, caused by flushing batches of writes. Filtering out writes:

```
$ awk '{ gsub(/:/, "") } $5 ~ /issue/ && $7 ~ /R/ { ts[$6, $10] = $4 }
    $5 ~ /complete/ { if (l = ts[$6, $9]) { printf "%.f %.f\n", $4 * 1000000,
    ($4 - l) * 1000000; ts[$6, $10] = 0 } }' out.blockio > out.read_lat
$ ./trace2heatmap.pl --unitstime=us --unitslat=us out.read_lat > out.read.svg

```

Produces ( [SVG](/blog/images/2014/out.read.svg)):

![](/blog/images/2014/out.read.png)

Ah, latency spikes gone. These are bursts of writes, likely file system flushes. Also notice that the reads were unaffected: what's likely happening is that the disk devices are prioritizing reads over writes.

## Conclusion

While the main storage on my c3.large is SSDs, my example workload accessed the rotational boot disk, changing the latency distribution dramatically. It's an easy configuration mistake to watch out for. There were also occasions where bursts of writes to this disk occurred, likely file system flushes, causing spikes in latency. These are normal, and usually don't affect synchronous I/O (drives can queue and service these differently).

These behaviors were visualized using heat maps for latency, I/O size, and I/O offset. These were generated using Linux [perf\_events](/perf.html) to capture disk I/O events, some awk to process the output, and my [heat map](https://github.com/brendangregg/HeatMap) (github) software. You can do this too. Usual warning applys about running commands as root in production: understand what you are doing first.

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
