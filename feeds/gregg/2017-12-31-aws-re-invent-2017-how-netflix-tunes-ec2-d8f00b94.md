---
title: 'AWS re:Invent 2017: How Netflix Tunes EC2'
url: https://www.brendangregg.com/blog/2017-12-31/reinvent-netflix-ec2-tuning.html
published: "2017-12-31T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2017-12-31/reinvent-netflix-ec2-tuning.html
---

# AWS re:Invent 2017: How Netflix Tunes EC2

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

## AWS re:Invent 2017: How Netflix Tunes EC2

31 Dec 2017

My last talk for 2017 was at AWS re:Invent, on "How Netflix Tunes EC2 Instances for Performance," an updated version of my [2014](/blog/2015-03-03/performance-tuning-linux-instances-on-ec2.html) talk. There was so much demand for it this year that I had three overflow rooms streaming it, and people still couldn't get in. (I shouldn't let this go to my head, as there were 42,000 attendees at re:Invent looking for something to see!) Fortunately, it was videoed for those who missed it.

A video of the talk is on [youtube](https://www.youtube.com/watch?v=89fYOo1V2pA):

Here are the [slides](/Slides/AWSreInvent2017_performance_tuning_EC2) or as a [PDF](/Slides/AWSreInvent2017_performance_tuning_EC2.pdf):

firstprevnextlast/permalink/zoom

I love this talk as I get to share more about what the Performance and Operating Systems team at Netflix does, rather than just my work. Our team looks after the BaseAMI, kernel tuning, OS performance tools and profilers, and self-service tools like Vector. We're not the only people doing performance and performance tuning at Netflix either: all the development teams do performance work. We help where we can.

My talk included a section on Linux kernel tunables, as follows. **WARNING: These tunables were developed in late 2017, for Ubuntu Xenial instances on EC2.**

CPU

```
schedtool –B PID

```

Virtual Memory

```
vm.swappiness = 0       # from 60

```

Huge Pages

```
# echo madvise > /sys/kernel/mm/transparent_hugepage/enabled

```

NUMA

```
kernel.numa_balancing = 0

```

File System

```
vm.dirty_ratio = 80                     # from 40
vm.dirty_background_ratio = 5           # from 10
vm.dirty_expire_centisecs = 12000       # from 3000
mount -o defaults,noatime,discard,nobarrier …

```

Storage I/O

```
/sys/block/*/queue/rq_affinity  2
/sys/block/*/queue/scheduler        noop
/sys/block/*/queue/nr_requests  256
/sys/block/*/queue/read_ahead_kb    256
mdadm –chunk=64 ...

```

Networking

```
net.core.somaxconn = 1000
net.core.netdev_max_backlog = 5000
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_wmem = 4096 12582912 16777216
net.ipv4.tcp_rmem = 4096 12582912 16777216
net.ipv4.tcp_max_syn_backlog = 8096
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 10240 65535
net.ipv4.tcp_abort_on_overflow = 1    # maybe

```

Hypervisor (Xen)

```
echo tsc > /sys/devices/system/clocksource/clocksource0/current_clocksource

```

Not a lot has changed with these tunables since my [2014](/blog/2015-03-03/performance-tuning-linux-instances-on-ec2.html) talk.

What I was most excited for was the launch of a new EC2 hypervisor, which I referred to in the video as the "c5 hypervisor". Later that night the real name was released: the "Nitro" hypervisor, as well as the bare metal instance type. My last post on [Introducing Nitro](http://www.brendangregg.com/blog/2017-11-29/aws-ec2-virtualization-2017.html) explained it and the hypervisor development journey.

Many other Netflix staff spoke at re:Invent ( [list here](https://medium.com/netflix-techblog/netflix-at-aws-re-invent-2017-79384f525367)). Here are the talks from my immediate colleagues in building F, level 2 at Netflix:

- Vadim Filanovsky (perf team) co-presented [Auto Scaling Made Easy: How Target Tracking Scaling Policies Hit the Bullseye](https://www.youtube.com/watch?v=qOLOhvlpMyU)
- Dave Hahn (CORE team) gave an updated [A Day in the Life of a Netflix Engineer III](https://www.youtube.com/watch?v=T_D1G42G0dE)
- Nora Jones (chaos team) gave a keynote on [Why We Need More Chaos - Chaos Engineering, That Is](https://www.youtube.com/watch?v=rgfww8tLM0A), as well as a talk [Performing Chaos at Netflix Scale](https://www.youtube.com/watch?v=LaKGx0dAUlo)
- Casey Rosenthal (traffic and chaos) [Models of Availability](https://www.youtube.com/watch?v=xc_PZ5OPXcc)
- John Bennett (networking) co-presented [How Netflix Monitors Applications in Near Real-Time with Amazon Kinesis](https://www.youtube.com/watch?v=tuDt7eLC5n4)
- Donovan Fritz and Joel Kodama (networking) [A Day in the Life of a Cloud Network Engineer at Netflix](https://www.youtube.com/watch?v=8C9xNVYbCVk)
- Alex Maestretti (security) co-presented [SecOps 2021 Today: Using AWS Services to Deliver SecOps](https://www.youtube.com/watch?v=ZoCJ_VIsLQs)
- Will Bengtson (security) co-presented [Best Practices for Managing Security Operations on AWS](https://www.youtube.com/watch?v=gjrcoK8T3To)
- Patrick Kelley and Travis McPeak (security) [Using Access Advisor to Strike the Balance Between Security and Usability](https://www.youtube.com/watch?v=gp4z0dCRTS0)
- Andrew Spyker (Titus) co-presented [Elastic Load Balancing Deep Dive and Best Practices](https://www.youtube.com/watch?v=9TwkMMogojY)
- Andrew Park and Sebastien de Larquier [Tooling Up for Efficiency: DIY Solutions @ Netflix](https://www.youtube.com/watch?v=U01QSQJmJbQ)
- Rajan Mittal and Andrew Park [Why Regional Reserved Instances Are a Game Changer for Netflix](https://www.youtube.com/watch?v=i1EW6zmFbSM)
- Monal Daxini [Netflix Keystone SPaaS: Real-time Stream Processing as a Service](https://www.youtube.com/watch?v=p8qSWE_nAAE)
- Our department director Coburn Watson [Walking the tightrope: Balancing Innovation, Reliability, Security, and Efficiency](https://www.youtube.com/watch?v=00awAtGTyJ0)

Check them out. It's awesome to see my coworkers on the big stage doing great!

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
