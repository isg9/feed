---
title: USENIX/LISA 2014 New Tools and Old Secrets (perf-tools)
url: https://www.brendangregg.com/blog/2015-03-17/usenix-lisa-2014-linux-ftrace-perf-tools.html
published: "2015-03-17T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2015-03-17/usenix-lisa-2014-linux-ftrace-perf-tools.html
---

# USENIX/LISA 2014 New Tools and Old Secrets (perf-tools)

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

## USENIX/LISA 2014 New Tools and Old Secrets (perf-tools)

17 Mar 2015

At the last USENIX/LISA conference, I gave a talk on new Linux performance tools: my open source [perf-tools](https://github.com/brendangregg/perf-tools) collection. These use existing kernel frameworks, ftrace and perf\_events, which are built in to most Linux kernel distributions by default, including the Linux cloud instances I analyze at Netflix.

The slides are on [slideshare](http://www.slideshare.net/brendangregg/linux-performance-analysis-new-tools-and-old-secrets):

The talk was also videoed, which is on [youtube](https://www.youtube.com/watch?v=R4IKeMQhM0Y):

I have a long history of creating DTrace scripts, published in the DTraceToolkit and the DTrace book, some of which are shipped by default in some OSes. After switching to Linux, I've been looking for ways to port them over, and have found that ftrace and perf\_events – which are in the Linux kernel source – can provide many of the capabilities I need. My perf-tools collection provides scripts to facilitate their use.

Ftrace, in particular, seems to be a well-kept secret of the Linux kernel. It was created by Steven Rostedt, and has had virtually no marketing. The tools I'm creating can help raise awareness, by showing examples of what ftrace can do.

For example, my ftrace-based iosnoop:

```
# ./iosnoop
Tracing block I/O... Ctrl-C to end.
COMM             PID    TYPE DEV      BLOCK        BYTES     LATms
supervise        1809   W    202,1    17039968     4096       1.32
supervise        1809   W    202,1    17039976     4096       1.30
tar              14794  RM   202,1    8457608      4096       7.53
tar              14794  RM   202,1    8470336      4096      14.90
tar              14794  RM   202,1    8470368      4096       0.27
tar              14794  RM   202,1    8470784      4096       7.74
tar              14794  RM   202,1    8470360      4096       0.25
tar              14794  RM   202,1    8469968      4096       0.24
tar              14794  RM   202,1    8470240      4096       0.24
[...]

```

The output shows the tar command making a series of metadata reads (type == RM), with one taking 14.9 milliseconds. Like tcpdump, but for disks, this can be useful to track down odd performance behaviors that may involve sequences of I/O and their queueing effects.

The perf-tools collection contains several single-purpose tools, like iosnoop, which aim to do one thing and do it well (Unix philosophy).

There are also multi-purpose tools, which require more expertise to use, but are also more powerful. For example, I'll use funccount to count kernel calls beginning with "ip":

```
# ./funccount 'ip*'
Tracing "ip*"... Ctrl-C to end.
^C
FUNC                              COUNT
[...]
ip_mc_sf_allow                       70
ipv6_chk_mcast_addr                  72
ip_finish_output                    108
ip_local_out                        108
ip_output                           108
ip_queue_xmit                       108
ipv4_mtu                            216
ip_local_deliver                    229
ip_local_deliver_finish             229
ip_rcv                              229
ip_rcv_finish                       229
ipv4_dst_check                      513

Ending tracing...

```

This output shows 108 calls to ip\_output(), which sounds like a function for sending IP packets. Let's pull out some stack traces to see how we got here, using kprobe:

```
# ./kprobe -s 'p:ip_output' | head -50 > out.ip_output
# more out.ip_output
Tracing kprobe ip_output. Ctrl-C to end.
            sshd-19389 [003] d... 3042954.165578: ip_output: (ip_output+0x0/0x90)
            sshd-19389 [003] d... 3042954.165584:
 => ip_queue_xmit
 => tcp_transmit_skb
 => tcp_write_xmit
 => __tcp_push_pending_frames
 => tcp_sendmsg
 => inet_sendmsg
 => sock_aio_write
 => do_sync_write
 => vfs_write
 => SyS_write
 => system_call_fastpath
            sshd-19250 [001] d... 3042954.258718: ip_output: (ip_output+0x0/0x90)
            sshd-19250 [001] d... 3042954.258723:
 => ip_queue_xmit
[...]

```

Nice! We can see the path from the write() syscall to ip\_output(), through VFS and TCP. I redirected the output of kprobe to a file to avoid a feedback loop, as I'm logged in via SSH. The head command ensures that kprobe exits after capturing several stack traces, as we don't want to leave this running (overhead).

I can go further with kprobe, and inspect functions arguments as well. There are [examples of kprobe](https://github.com/brendangregg/perf-tools/blob/master/examples/kprobe_example.txt) and all other tools in the perf-tools collection. If need be, I can re-instrument the same kprobe one-liner using perf\_events, which has lower overhead.

While these tools have a polished interface (USAGE message, man page, examples file), their internals often make use of temporary hacks, awaiting more Linux kernel features. For example, iosnoop passes both I/O request and response timestamps to user-level, which then calculates the delta, when it would be more efficient to do this calculation in-kernel, and only pass the result.

In particular, I'm looking forward to eBPF (or a similar facility) being available in the kernel, so that I can do more kernel-level programming including the I/O latency calculation during tracepoints, and reduce the overhead of tracing. I'll update my tools to use eBPF when it is available, and I'll also be able to create more.

LISA is a great conference, although I arrived late as there was a clash with AWS re:Invent, which I also spoke at. I can at least see the talks I missed, since they were videoed: they are linked on the [conference program](https://www.usenix.org/conference/lisa14/conference-program).

I did arrive in time to see some talks, including Ben Rockwood's excellent [I Am SysAdmin (And So Can You!)](https://www.usenix.org/conference/lisa14/conference-program/presentation/rockwood), which I referenced. Despite the many hats I've worn, I still feel like a sysadmin, as I still use those skills day to day. Ben and I also took a moment to pose with Deirdré, who created the ponycorn mascots in my slidedeck, for a [photo](https://twitter.com/DeirdreS/status/533335956053454848). I hope to be back at LISA in 2015.

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
