---
title: USENIX/LISA 2013 Metrics Workshop
url: https://www.brendangregg.com/blog/2014-05-16/usenix-lisa-2013-metrics-workshop.html
published: "2014-05-16T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-05-16/usenix-lisa-2013-metrics-workshop.html
---

# USENIX/LISA 2013 Metrics Workshop

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

## USENIX/LISA 2013 Metrics Workshop

16 May 2014

[![](/blog/images/2014/brendan-workshop-bw-600.png)](/blog/images/2014/brendan-workshop-bw-600.png)

At USENIX LISA'13 I helped run a [Metrics Workshop](https://www.usenix.org/conference/lisa13/workshops/metrics), along with Narayan Desai (Argonne National Laboratory), Kent Skaar (VMware, Inc.), Theo Schlossnagle (OmniTI), and Caskey Dickson (Google). This was an opportunity for many industry professionals to discuss problems with performance metrics and monitoring, and to propose and discuss solutions. It was a lot of fun, and was very useful to hear the different opinions and perspectives from those who attended.

We provided guidance for choosing more effective performance metrics, which involved helping people think more freely and creatively, instead of being bounded by the metrics that are currently or typically offered. I also covered key methodologies, including the USE Method, which provide a checklist of concise metrics that are designed to solve issues. I ended the day with five minute lightning talks on [statistics and visualizations](http://www.youtube.com/watch?v=HKmgdLcx8jQ).

There were about 30 participants, and [Deirdré Straughan](http://www.beginningwithi.com) videoed the entire day long event, which includes the talks by the other moderators. The videos are on [youtube](http://www.youtube.com/playlist?list=PL3oXECC9Rpm2BL0v1W7cgyj8eV9d44icc):

As an exercise, we identified several targets of performance monitoring, formed groups to propose ideal metrics, then presented and discussed these metrics. I've listed a summary of the metrics below, and also submitted them to the monitoringsucks project on [github](https://github.com/monitoringsucks/metrics-catalog/pull/1).

## Network Infrastructure

- Physical Infrastructure

  - bandwidth, utilization of individual links
  - CoS/QoS rate/drops
  - L2/L2 protocol health
  - churn
  - reachabality
- Per port:

  - packets/sec
  - packet size
  - buffer utilization
  - perf flow into:
  - app injection BW
  - app injectiov rate
  - app consumption rate
  - app consumption BW
- Component:

  - links
  - errors
  - latency
  - utilization
- Topology:

  - app to app latency
  - app to app low
  - symmetry

## Configuration

- Apps should export flags, to check for consistency

  - a metadata to show the target configuration
- Versioning:

  - ldd, libraries linked against
  - time a config was applied
- Platform Type:

  - server H/W
- Cost of Configuration

  - cost of configuration upload/download
  - time to deployment: security changes (high priority), vs others
  - CPU and RAM usage during configuration
- People

  - deployment report
- Hardware

  - current hardware
  - max expected performance
- Process

  - compliance measurement of configuration: percent of systems
- Failure

  - failure of configuration deployment
  - rollbacks, rollforward: config metric didn't apply
- OS flags

## Distributed system

- Perceived latency: service time and queueing
- Request rate
- Error rate
- Traffic origins
- Histogram of latencies for each server, for comparisons
- Visualizations:

  - heatmaps
  - for service
  - per server
  - per backend
  - system 'flame graph'
  - visualize traffic as graph, queue time, request flow

## Message Queueing

- Distribution of message latency (ns)
- Throughput
- Total number of ns
- Errors, drop, retransmits, discards
- Message fanout distribution (gain: ratio of input to put)
- For distribution message queues: see distributied systems
- Queue lengths
- Saturation: run out of space
- Resource constraints on queueing systems
- Last time of access

## Web servers

- Requests: referrer, origin, UA, resp code, count

  - origin
  - response code
- Req size: distribution
- Response Size: resp code, distribution
- Responce Count: resp code, counter
- Time To First Bite: resp code, distribution
- Time To Last Bite: resp code, distribution
- Active Workers: guage
- Worker Age: guage
- Connections: counter
- Process Metrics from host

## Application servers

- Total requests served, rate
- Latency:

  - time to serve a client
  - complete a client transaction
  - request queue time
- App error rate
- Error counts on backend H/W
- Bandwidth usage front and backend
- System load on primary application server: CPU, memory, disk, swapping
- Usage patterns:

  - which user, client time, session time, active vs idle time

## Databases

- Queries/sec
- \# of connections
- connections/sec
- avg time per query
- cache hit rate
- avg io latency
- aggregate io
- % of query time in io
- \# of locks
- \# of versions (for read consistency)
- terminated connects
- SQL statements
- cache evictions
- query errors by type
- saturation: plan to execute

  - queueing on pool
- change in number of executed plans
- latency of last checkpoint, and on-disk representation of wall log

  - (how much of DB to reply)
- checkpoint times

## Resources/Devices

- Utilization

  - per-device: eg, as a heat map for distribution over time
- Saturation

  - average queue length, or time waiting on queue
- Errors

Thanks to all those who attended and helped out!

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
