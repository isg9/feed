---
title: 'Linux bcc/BPF tcplife: TCP Lifespans'
url: https://www.brendangregg.com/blog/2016-11-30/linux-bcc-tcplife.html
published: "2016-11-30T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-11-30/linux-bcc-tcplife.html
---

# Linux bcc/BPF tcplife: TCP Lifespans

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

## Linux bcc/BPF tcplife: TCP Lifespans

30 Nov 2016

"i really wish i had a command line tool that would give me stats on TCP connection lengths on a given port"

```
# ./tcplife -D 80
PID   COMM       LADDR           LPORT RADDR           RPORT TX_KB RX_KB MS
27448 curl       100.66.11.247   54146 54.154.224.174  80        0     1 263.85
27450 curl       100.66.11.247   20618 54.154.164.22   80        0     1 243.62
27452 curl       100.66.11.247   11480 54.154.43.103   80        0     1 231.16
27454 curl       100.66.11.247   31382 54.154.15.7     80        0     1 249.95
27456 curl       100.66.11.247   33416 52.210.59.223   80        0     1 545.72
27458 curl       100.66.11.247   16406 52.30.140.35    80        0     1 222.29
27460 curl       100.66.11.247   11634 52.30.133.135   80        0     1 217.52
27462 curl       100.66.11.247   25660 52.30.126.182   80        0     1 250.81
[...]

```

That's tracing destination port 80, you can also trace local port (-L), or trace all ports (default behavior).

The quote and good idea is from Julia Evans on [twitter](https://twitter.com/b0rk/status/765666624968003584), and I added it as a tool to the Linux BPF-based [bcc](https://github.com/iovisor/bcc) open source collection.

The output of tcplife, short for TCP lifespan, shows not just the duration (MS == milliseconds) but also throughput statistics: TX\_KB for Kbytes transmitted, and RX\_KB for Kbytes received. It should be useful for performance and security analysis, and network debugging.

## I am NOT tracing packets

The overheads can cost too much to examine every packet, especially on servers that process millions of packets per second. To do this I'm not using tcpdump, libpcap, tethereal, or any network sniffer.

So how'd I do it? There were four challenges:

### 1\. Measuring lifespans

The current version of tcplife uses kernel dynamic tracing (kprobes) of tcp\_set\_state(), and looks for the duration from an early state (eg, TCP\_ESTABLISHED) to TCP\_CLOSE. State changes have a much lower frequency than packets, so this approach greatly reduces overhead. But it ends up tricker than it sounds, since it's tracing the Linux implementation of TCP, which isn't guaranteed to use tcp\_set\_state() for every state transition. But it works well enough for now, and we can add a stable tracepoint for TCP state transitions to future kernels so that this will always work. (Or, if that proves untenable, tracepoints for the creation and destruction of TCP sessions or sockets.)

### 2\. Fetching addresses and ports

tcp\_set\_state() has `struct sock *sk` as an argument, and we can dig the details from that.

### 3\. Fetching throughput statistics

Those TX\_KB and RX\_KB columns. This was only possible in the last couple of years thanks to the RFC-4898 additions to `struct tcp_info` in the Linux kernel: tcpi\_bytes\_acked and tcpi\_bytes\_received. I discussed these in my [tcptop](http://www.brendangregg.com/blog/2016-10-15/linux-bcc-tcptop.html) blog post.

### 4\. Showing task context

It's useful to show the PID and COMM (process name) with these connections, but TCP state changes aren't guaranteed to happen in the correct task context, so we can't just fetch the currently running task information. Nor is there a cached PID and comm in `struct sock` to print out. So I'm caching the task context on TCP state changes where it's usually valid, by virtue of implementation. It works, but is another area where we can do better, and can be addressed if and when we add stable TCP tracepoints.

That's how tcplife works right now. It the future it may change and improve, especially if more TCP tracepoints become available. I'm also not the first to try this kind of TCP event tracing: Facebook already have their own BPF TCP event tool, although I think their implementation is a little different.

Here's the full USAGE message for tcplife:

```
# ./tcplife -h
usage: tcplife [-h] [-T] [-t] [-w] [-s] [-p PID] [-L LOCALPORT]
               [-D REMOTEPORT]

Trace the lifespan of TCP sessions and summarize

optional arguments:
  -h, --help            show this help message and exit
  -T, --time            include time column on output (HH:MM:SS)
  -t, --timestamp       include timestamp on output (seconds)
  -w, --wide            wide column output (fits IPv6 addresses)
  -s, --csv             comma seperated values output
  -p PID, --pid PID     trace this PID only
  -L LOCALPORT, --localport LOCALPORT
                        comma-separated list of local ports to trace.
  -D REMOTEPORT, --remoteport REMOTEPORT
                        comma-separated list of remote ports to trace.

examples:
    ./tcplife           # trace all TCP connect()s
    ./tcplife -t        # include time column (HH:MM:SS)
    ./tcplife -w        # wider colums (fit IPv6)
    ./tcplife -stT      # csv output, with times & timestamps
    ./tcplife -p 181    # only trace PID 181
    ./tcplife -L 80     # only trace local port 80
    ./tcplife -L 80,81  # only trace local ports 80 and 81
    ./tcplife -D 80     # only trace remote port 80

```

And there is more [example output](https://github.com/iovisor/bcc/blob/master/tools/tcplife_example.txt) in bcc, along with a [man page](https://github.com/iovisor/bcc/blob/master/man/man8/tcplife.8). For more about bcc and BPF (eBPF), see the collection of documents on my [Linux performance page](http://www.brendangregg.com/linuxperf.html#Documentation).

The only catch is that tcplife does need newer kernels (say, 4.4).

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
