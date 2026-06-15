---
title: Linux ftrace TCP Retransmit Tracing
url: https://www.brendangregg.com/blog/2014-09-06/linux-ftrace-tcp-retransmit-tracing.html
published: "2014-09-06T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-09-06/linux-ftrace-tcp-retransmit-tracing.html
---

# Linux ftrace TCP Retransmit Tracing

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

## Linux ftrace TCP Retransmit Tracing

06 Sep 2014

Why is my Linux system doing TCP retransmits? Lets see packet details and kernel state:

```
# ./tcpretrans
TIME     PID    LADDR:LPORT          -- RADDR:RPORT          STATE
05:16:44 3375   10.150.18.225:53874  R> 10.105.152.3:6001    ESTABLISHED
05:16:44 3375   10.150.18.225:53874  R> 10.105.152.3:6001    ESTABLISHED
05:16:54 4028   10.150.18.225:6002   R> 10.150.30.249:1710   ESTABLISHED
05:16:54 4028   10.150.18.225:6002   R> 10.150.30.249:1710   ESTABLISHED
05:16:54 4028   10.150.18.225:6002   R> 10.150.30.249:1710   ESTABLISHED
05:16:55 0      10.150.18.225:47115  R> 10.71.171.158:6001   ESTABLISHED
05:16:58 0      10.150.18.225:44388  R> 10.103.130.120:6001  ESTABLISHED
^C
Ending tracing...

```

Awesome!

tcpretrans is a script from my [perf-tools](https://github.com/brendangregg/perf-tools) collection, and uses [ftrace](/blog/2014-08-30/ftrace-the-hidden-light-switch.html) to dynamically instrument the tcp\_retransmit\_skb() kernel function. One reason this is awesome is that the overhead is negligible: it only adds instrumentation to the retransmit path. It's also using existing Linux kernel features, ftrace and kprobes, and doesn't even need kernel debuginfo.

**This does not trace every packet and then filter**, such as if I had used any tcpdump/libpcap/kernel-packet-filter approach, which can become painful for high packet rates. (I think it would also be lazy to trace every packet when you can just trace the kernel retransmit code path.)

Using a tracer like ftrace means I can also dig out kernel state, which is invisible to on-the-wire tracers like tcpdump. I only included the kernel STATE column, but anything kernel-side could be included.

The -s option will include the kernel stack trace that led to the TCP retransmit:

```
# ./tcpretrans -s
TIME     PID    LADDR:LPORT          -- RADDR:RPORT          STATE
06:21:10 19516  10.144.107.151:22    R> 10.13.106.251:32167  ESTABLISHED
 => tcp_fastretrans_alert
 => tcp_ack
 => tcp_rcv_established
 => tcp_v4_do_rcv
 => tcp_v4_rcv
 => ip_local_deliver_finish
 => ip_local_deliver
 => ip_rcv_finish
 => ip_rcv
 => __netif_receive_skb
 => netif_receive_skb
 => handle_incoming_queue
 => xennet_poll
 => net_rx_action
 => __do_softirq
 => call_softirq
 => do_softirq
 => irq_exit
 => xen_evtchn_do_upcall
 => xen_do_hypervisor_callback
[...]

```

In this case, the retransmit was sent after receiving a packet (ip\_rcv()), processing a TCP ACK (tcp\_ack()), and then by tcp\_fastretrans\_alert(). This is a TCP fast retransmit. Ie, these:

```
# netstat -s | grep -i retr
    242 segments retransmited
    46 fast retransmits
    2 forward retransmits
    3 retransmits in slow start

```

Here are a couple of timer-based retransmits for comparison:

```
06:38:45 0      10.11.172.162:7102   R> 10.10.153.60:47538   ESTABLISHED
 => tcp_write_timer_handler
 => tcp_write_timer
 => call_timer_fn
 => run_timer_softirq
 => __do_softirq
 => irq_exit
 => xen_evtchn_do_upcall
 => xen_hvm_callback_vector
 => default_idle
 => arch_cpu_idle
 => cpu_startup_entry
 => start_secondary
06:38:45 0      10.11.172.162:7102   R> 10.10.153.60:47539   ESTABLISHED
 => tcp_write_timer_handler
 => tcp_write_timer
 => call_timer_fn
 => run_timer_softirq
 => __do_softirq
 => irq_exit
 => xen_evtchn_do_upcall
 => xen_hvm_callback_vector
 => default_idle
 => arch_cpu_idle
 => cpu_startup_entry
 => rest_init
 => start_kernel
 => x86_64_start_reservations
 => x86_64_start_kernel

```

These come from a callback, and tcp\_write\_timer\_handler(). Timer-based retransmits are worse than fast retransmits, as they add timer-based latency to application requests. This is usually 1000 ms in Linux.

My tcpretrans ftrace-based tool is a hack, and may not work on some systems without alterations (it also doesn't support IPv6 yet). To dig out custom details like IP addresses as dotted quad strings, I really should be using a programmable tracer like SystemTap. However, I wanted to see if this was possible with just ftrace, as it would make it easier to use in my production environment (Netflix cloud).

It works like this:

1. Dynamically instrument tcp\_retransmit\_skb() using kprobes, and capture the %di register.
2. Assume the skb pointer is in register %di (to know for certain would require kernel debuginfo, which I don't normally have on production systems). On non-x86 systems, this may well be in another register, and this script will need editing.
3. Wait one second.
4. Read the kernel buffer of tcp\_retransmit\_skb() events, with skb pointers.
5. Read /proc/net/tcp, and cache socket details by skb.
6. Assume retransmits happen for long-lived sessions (> 1 second), and the session details would still have been in /proc/net/tcp when it was read.
7. Parse the kernel buffer of tcp\_retransmit\_skb() events and print retransmit events with session details from the /proc/net/tcp data we read earlier.
8. Goto 3.

On an earlier version, I read /proc/net/tcp synchronously when retransmits occurred, but on production systems with frequent retransmits and tens of thousands of open connections (making for a very large /proc/net/tcp), the CPU overhead of that approach was too high. The approach above only reads /proc/net/tcp once per second.

So, it was a gift to have socket details in /proc/net/tcp, and not need to dig them out using ftrace. That would be possible, but the script would become much more brittle without kernel debuginfo, as there would then be several assumptions about registers and struct offsets, rather than just skb being in %di. Without /proc/net/tcp, I'd only really attempt this *with* kernel debuginfo, where I could make it reasonbly reliable.

So far tcpretrans has proved quite useful to quickly get some details on TCP retransmits: not just source and destionation addresses and ports, but kernel state and stack traces.

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
