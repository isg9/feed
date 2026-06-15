---
title: TCP Tracepoints
url: https://www.brendangregg.com/blog/2018-03-22/tcp-tracepoints.html
published: "2018-03-22T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2018-03-22/tcp-tracepoints.html
---

# TCP Tracepoints

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

## TCP Tracepoints

22 Mar 2018

TCP tracepoints have arrived in Linux! Linux 4.15 added five of them, and 4.16 (not quite released yet) added two more ( `tcp:tcp_probe`, and `sock:inet_sock_set_state` – a socket tracepoint that can be used for TCP analysis). We now have:

```
# perf list 'tcp:*' 'sock:inet*'

List of pre-defined events (to be used in -e):

  tcp:tcp_destroy_sock                               [Tracepoint event]
  tcp:tcp_probe                                      [Tracepoint event]
  tcp:tcp_receive_reset                              [Tracepoint event]
  tcp:tcp_retransmit_skb                             [Tracepoint event]
  tcp:tcp_retransmit_synack                          [Tracepoint event]
  tcp:tcp_send_reset                                 [Tracepoint event]

  sock:inet_sock_set_state                           [Tracepoint event]

```

This includes one that's versatile: `sock:inet_sock_set_state`. It can be used to track when the kernel changes the state of a TCP session, such as from TCP\_SYN\_SENT to TCP\_ESTABLISHED. One example use is my [tcplife](http://www.brendangregg.com/blog/2016-11-30/linux-bcc-tcplife.html) tool in the open source [bcc](https://github.com/iovisor/bcc) collection:

```
# tcplife
PID   COMM       LADDR           LPORT RADDR           RPORT TX_KB RX_KB MS
22597 recordProg 127.0.0.1       46644 127.0.0.1       28527     0     0 0.23
3277  redis-serv 127.0.0.1       28527 127.0.0.1       46644     0     0 0.28
22598 curl       100.66.3.172    61620 52.205.89.26    80        0     1 91.79
22604 curl       100.66.3.172    44400 52.204.43.121   80        0     1 121.38
22624 recordProg 127.0.0.1       46648 127.0.0.1       28527     0     0 0.22
3277  redis-serv 127.0.0.1       28527 127.0.0.1       46648     0     0 0.27
[...]

```

I wrote tcplife before this tracepoint existed, so I had to use kprobes (kernel dynamic tracing) of the tcp\_set\_state() kernel function. That works, but it's relying on various kernel implementation details that may change from one kernel version to the next. To keep tcplife working, it would need to include different code every time the kernel changed, which would become difficult to maintain and enhance. Imagine needing to test changes on several different kernel versions, because tcplife has special code for each!

Tracepoints are considered a "stable API," so their details shouldn't change from one kernel version to the next, making programs that use them easier to maintain. I say "shouldn't" on purpose, because I consider these "best effort" and not "set in stone." If they are considered set in stone, then it will be harder to convince kernel maintainers to accept new tracepoints (for good reason). As a case in point: `tcp:tcp_set_state` was added in 4.15, and then `sock:inet_sock_set_state` was added in 4.16. Since the sock one is a superset, the tcp one was disabled in 4.16 and will be removed. We try to avoid changing tracepoints like this, but in this case it was short-lived and removed before anyone had used it.

tcplife isn't a great example of using tracepoints anyway, as it goes beyond the tracepoint API in three places (tx and rx bytes, and best-effort process context on tracepoints), so it may still need some maintenance. But it's a large improvement over the kprobes version, and other tools can stick to the tracepoints API only.

Another way to show `sock:inet_sock_set_state` is to compare it with kprobes on tcp\_set\_state() using Sasha Goldshtein's bcc trace tool. The first command uses kprobes, and the second the tracepoint:

```
# trace -t -I net/sock.h 'p::tcp_set_state(struct sock *sk) "%llx: %d -> %d", sk, sk->sk_state, arg2'
TIME     PID     TID     COMM            FUNC             -
2.583525 17320   17320   curl            tcp_set_state    ffff9fd7db588000: 7 -> 2
2.584449 0       0       swapper/5       tcp_set_state    ffff9fd7db588000: 2 -> 1
2.623158 17320   17320   curl            tcp_set_state    ffff9fd7db588000: 1 -> 4
2.623540 0       0       swapper/5       tcp_set_state    ffff9fd7db588000: 4 -> 5
2.623552 0       0       swapper/5       tcp_set_state    ffff9fd7db588000: 5 -> 7
^C
# trace -t 't:sock:inet_sock_set_state "%llx: %d -> %d", args->skaddr, args->oldstate, args->newstate'
TIME     PID     TID     COMM            FUNC             -
1.690191 17308   17308   curl            inet_sock_set_state ffff9fd7db589800: 7 -> 2
1.690798 0       0       swapper/24      inet_sock_set_state ffff9fd7db589800: 2 -> 1
1.727750 17308   17308   curl            inet_sock_set_state ffff9fd7db589800: 1 -> 4
1.728041 0       0       swapper/24      inet_sock_set_state ffff9fd7db589800: 4 -> 5
1.728063 0       0       swapper/24      inet_sock_set_state ffff9fd7db589800: 5 -> 7
^C

```

Both are showing the same output. For reference:

- 1: TCP\_ESTABLISHED
- 2: TCP\_SYN\_SENT
- 3: TCP\_SYN\_RECV
- 4: TCP\_FIN\_WAIT1
- 5: TCP\_FIN\_WAIT2
- 6: TCP\_TIME\_WAIT
- 7: TCP\_CLOSE
- 8: TCP\_CLOSE\_WAIT

I know, I know, I should just add that as a lookup hash and...a little while later, here's a new tool I just contributed to bcc – [tcpstate](https://github.com/iovisor/bcc/pull/1639) – that does the translations, and shows per-state durations:

```
# tcpstates
SKADDR           C-PID C-COMM     LADDR           LPORT RADDR         RPORT OLDSTATE    -> NEWSTATE    MS
ffff9fd7e8192000 22384 curl       100.66.100.185  0     52.33.159.26  80    CLOSE       -> SYN_SENT    0.000
ffff9fd7e8192000 0     swapper/5  100.66.100.185  63446 52.33.159.26  80    SYN_SENT    -> ESTABLISHED 1.373
ffff9fd7e8192000 22384 curl       100.66.100.185  63446 52.33.159.26  80    ESTABLISHED -> FIN_WAIT1   176.042
ffff9fd7e8192000 0     swapper/5  100.66.100.185  63446 52.33.159.26  80    FIN_WAIT1   -> FIN_WAIT2   0.536
ffff9fd7e8192000 0     swapper/5  100.66.100.185  63446 52.33.159.26  80    FIN_WAIT2   -> CLOSE       0.006
^C

```

I'm demonstrating this on Linux 4.16, after Yafang Shao wrote an [enhancement](https://lkml.org/lkml/2017/12/6/494) to show all state transitions, instead of just the ones the kernel implements. Here's what it used to look like on 4.15:

```
# trace -I net/sock.h -t 'p::tcp_set_state(struct sock *sk) "%llx: %d -> %d", sk, sk->sk_state, arg2'
TIME     PID    TID    COMM         FUNC             -
3.275865 29039  29039  curl         tcp_set_state    ffff8803a8213800: 7 -> 2
3.277447 0      0      swapper/1    tcp_set_state    ffff8803a8213800: 2 -> 1
3.786203 29039  29039  curl         tcp_set_state    ffff8803a8213800: 1 -> 8
3.794016 29039  29039  curl         tcp_set_state    ffff8803a8213800: 8 -> 7
^C
# trace -t 't:tcp:tcp_set_state "%llx: %d -> %d", args->skaddr, args->oldstate, args->newstate'
TIME     PID    TID    COMM         FUNC             -
2.031911 29042  29042  curl         tcp_set_state    ffff8803a8213000: 7 -> 2
2.035019 0      0      swapper/1    tcp_set_state    ffff8803a8213000: 2 -> 1
2.746864 29042  29042  curl         tcp_set_state    ffff8803a8213000: 1 -> 8
2.754193 29042  29042  curl         tcp_set_state    ffff8803a8213000: 8 -> 7

```

Back to 4.16, here's the current list of tracepoints, with arguments, via bcc's tplist tool:

```
# tplist -v 'tcp:*'
tcp:tcp_retransmit_skb
    const void * skbaddr;
    const void * skaddr;
    __u16 sport;
    __u16 dport;
    __u8 saddr[4];
    __u8 daddr[4];
    __u8 saddr_v6[16];
    __u8 daddr_v6[16];
tcp:tcp_send_reset
    const void * skbaddr;
    const void * skaddr;
    __u16 sport;
    __u16 dport;
    __u8 saddr[4];
    __u8 daddr[4];
    __u8 saddr_v6[16];
    __u8 daddr_v6[16];
tcp:tcp_receive_reset
    const void * skaddr;
    __u16 sport;
    __u16 dport;
    __u8 saddr[4];
    __u8 daddr[4];
    __u8 saddr_v6[16];
    __u8 daddr_v6[16];
tcp:tcp_destroy_sock
    const void * skaddr;
    __u16 sport;
    __u16 dport;
    __u8 saddr[4];
    __u8 daddr[4];
    __u8 saddr_v6[16];
    __u8 daddr_v6[16];
tcp:tcp_retransmit_synack
    const void * skaddr;
    const void * req;
    __u16 sport;
    __u16 dport;
    __u8 saddr[4];
    __u8 daddr[4];
    __u8 saddr_v6[16];
    __u8 daddr_v6[16];
tcp:tcp_probe
    __u8 saddr[sizeof(struct sockaddr_in6)];
    __u8 daddr[sizeof(struct sockaddr_in6)];
    __u16 sport;
    __u16 dport;
    __u32 mark;
    __u16 length;
    __u32 snd_nxt;
    __u32 snd_una;
    __u32 snd_cwnd;
    __u32 ssthresh;
    __u32 snd_wnd;
    __u32 srtt;
    __u32 rcv_wnd;
# tplist -v sock:inet_sock_set_state
sock:inet_sock_set_state
    const void * skaddr;
    int oldstate;
    int newstate;
    __u16 sport;
    __u16 dport;
    __u16 family;
    __u8 protocol;
    __u8 saddr[4];
    __u8 daddr[4];
    __u8 saddr_v6[16];
    __u8 daddr_v6[16];

```

The first TCP tracepoint added was [tcp:tcp\_retransmit\_skb](https://github.com/torvalds/linux/commit/e086101b150ae8e99e54ab26101ef3835fa9f48d#diff-e8cb18ec8a94424be48ed548686783cd) by Cong Wang (Twitter). He cited my kprobe-based [tcpretrans](https://github.com/brendangregg/perf-tools/blob/master/net/tcpretrans) tool from [perf-tools](https://github.com/brendangregg/perf-tools) as an example consumer. Song Liu (Facebook) added five more tracepoints, including [tcp:tcp\_set\_state](https://github.com/torvalds/linux/commit/e8fce23946b7e7eadf25ad78d8207c22903dfe27#diff-e8cb18ec8a94424be48ed548686783cd) which is now `sock:inet_sock_set_state`. Thanks to Cong and Song, and also David S. Miller (networking maintainer) for accepting these and providing feedback on the ongoing tcp tracepoint work.

During development I talked to Song (and Alexei Starovoitov) about the recent additions, so I already have an idea about why these exist and their use. Some rough notes for the current TCP tracepoints:

- **tcp:tcp\_retransmit\_skb**: Traces retransmits. Useful for understanding network issues including congestion. Will be used by my tcpretrans tools instead of kprobes.
- **tcp:tcp\_retransmit\_synack**: Tracing SYN/ACK retransmits. I imagine this could be used for a type of DoS detection (SYN flood triggering SYN/ACKs and then retransmits). This is separate from tcp:tcp\_retransmit\_skb because this event doesn't have an skb.
- **tcp:tcp\_destroy\_sock**: Needed by any program that summarizes details in-memory for a TCP session, which would be keyed by the sock address. This probe can be used to know that the session has ended, so that sock address is about to be reused and any summarized data so far should be consumed and then deleted.
- **tcp:tcp\_send\_reset**: This traces a RST send during a valid socket, to diagnose those type of issues.
- **tcp:tcp\_receive\_reset**: Trace a RST receive.
- **tcp:tcp\_probe**: for TCP congestion window tracing, which also allowed an older TCP probe module to be deprecated and removed. This was [added by](https://lkml.org/lkml/2017/12/19/154) Masami Hiramatsu, and merged in Linux 4.16.
- **sock:inet\_sock\_set\_state**: Can be used for many things. The tcplife tool is one, but also my tcpconnect and tcpaccept bcc tools can be converted to use this tracepoint. We could add separate `tcp:tcp_connect` and `tcp:tcp_accept` tracepoints (or `tcp:tcp_active_open` and `tcp:tcp_passive_open`), but `sock:inet_sock_set_state` can be used for this.

Use of these tracepoints is much preferred over packet capture approaches (libpcap), as tracepoints should cost lower overhead and can expose useful kernel state that's not on the wire.

I can imagine how useful these TCP tracepoints will be, as I designed and used similar tracepoints many years ago: my [DTrace TCP provider](http://www.brendangregg.com/DTrace/DTrace_Network_Providers.html) which I demonstrated at [CEC2006](http://www.brendangregg.com/DTrace/CEC2006_demo.html). I originally split out TCP state changes into a probe for each state, but by the time this was merged it became a single [tcp:::state-change](https://docs.oracle.com/cd/E36784_01/html/E36846/glhmv.html) probe, as we now have in Linux via the sock tracepoint.

What's next? Tracepoints for `tcp:tcp_send` and `tcp:tcp_receive` may be handy, but special attention must be paid to the tiny overhead they can add (both enabled and especially disabled), since send/receive can be a very frequent activity. More tracepoints for error conditions would be useful too, such as for the connection refused path, which may be helpful for analyzing DoS attacks.

If you are interested in adding TCP tracepoints, I'd recommend coding a kprobe solution to start with as the proof of concept, and getting some production experience with it. This is the role my prior kprobe tools played. A kprobe solution will show whether a tracepoint would be that useful, and if so, help make the case for its inclusion with the Linux kernel maintainers.

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
