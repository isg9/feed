---
title: Linux bcc/eBPF tcpdrop
url: https://www.brendangregg.com/blog/2018-05-31/linux-tcpdrop.html
published: "2018-05-31T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2018-05-31/linux-tcpdrop.html
---

# Linux bcc/eBPF tcpdrop

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

## Linux bcc/eBPF tcpdrop

31 May 2018

While debugging a production issue of kernel-based TCP packet drops, I remembered seeing a new function added in Linux 4.7 by Eric Dumazet (Google) called tcp\_drop(), which I can trace using kprobes and bcc/eBPF. This lets me fetch extra context to explain why these drops are happening. Eg:

```
# tcpdrop
TIME     PID    IP SADDR:SPORT          > DADDR:DPORT          STATE (FLAGS)
05:46:07 82093  4  10.74.40.245:50010   > 10.74.40.245:58484   ESTABLISHED (ACK)
    tcp_drop+0x1
    tcp_rcv_established+0x1d5
    tcp_v4_do_rcv+0x141
    tcp_v4_rcv+0x9b8
    ip_local_deliver_finish+0x9b
    ip_local_deliver+0x6f
    ip_rcv_finish+0x124
    ip_rcv+0x291
    __netif_receive_skb_core+0x554
    __netif_receive_skb+0x18
    process_backlog+0xba
    net_rx_action+0x265
    __softirqentry_text_start+0xf2
    irq_exit+0xb6
    xen_evtchn_do_upcall+0x30
    xen_hvm_callback_vector+0x1af

05:46:07 85153  4  10.74.40.245:50010   > 10.74.40.245:58446   ESTABLISHED (ACK)
    tcp_drop+0x1
    tcp_rcv_established+0x1d5
    tcp_v4_do_rcv+0x141
    tcp_v4_rcv+0x9b8
    ip_local_deliver_finish+0x9b
    ip_local_deliver+0x6f
    ip_rcv_finish+0x124
    ip_rcv+0x291
    __netif_receive_skb_core+0x554
    __netif_receive_skb+0x18
    process_backlog+0xba
    net_rx_action+0x265
    __softirqentry_text_start+0xf2
    irq_exit+0xb6
    xen_evtchn_do_upcall+0x30
    xen_hvm_callback_vector+0x1af

[...]

```

This is [tcpdrop](https://github.com/iovisor/bcc/pull/1790), a new tool I've written for the open source [bcc](https://github.com/iovisor/bcc) project. It shows source and destination packet details, as well as TCP session state (from the kernel), TCP flags (from the packet TCP header), and the kernel stack trace that led to this drop. That stack trace helps answer the why (you'll need to browse the code behind those functions to make sense of it). This is also information that's not on the wire, so you can never see this using packet sniffers (libpcap, tcpdump, etc).

I can't help but highlight this small but significant change from Eric's patch ( [tcp: increment sk\_drops for dropped rx packets](https://patchwork.ozlabs.org/patch/604910/)):

```
@@ -6054,7 +6061,7 @@  int tcp_rcv_state_process(struct sock *sk, struct sk_buff *skb)

    if (!queued) {
 discard:
-       __kfree_skb(skb);
+       tcp_drop(sk, skb);
    }
    return 0;

```

\_\_kfree\_skb() is called from many paths to free socket buffers, including routine codeapths. Tracing it was too noisy: you'd have your packet drop code paths lost among many normal ones. But with the new tcp\_drop() function, I can trace just the TCP drops. I've already suggested some enhancements to Eric today at netconf18, such as adding a "reason" argument somewhere that I can trace for a more human description of why the packet was dropped. Maybe tcp\_drop() should become a tracepoint too.

Here's some more code worth mentioning, this time some eBPF/C from my tcpdrop tool:

```
[...]
    // pull in details from the packet headers and the sock struct
    u16 family = sk->__sk_common.skc_family;
    char state = sk->__sk_common.skc_state;
    u16 sport = 0, dport = 0;
    u8 tcpflags = 0;
    struct tcphdr *tcp = skb_to_tcphdr(skb);
    struct iphdr *ip = skb_to_iphdr(skb);
    bpf_probe_read(&sport, sizeof(sport), &tcp->source);
    bpf_probe_read(&dport, sizeof(dport), &tcp->dest);
    bpf_probe_read(&tcpflags, sizeof(tcpflags), &tcp_flag_byte(tcp));
    sport = ntohs(sport);
    dport = ntohs(dport);

    if (family == AF_INET) {
        struct ipv4_data_t data4 = {.pid = pid, .ip = 4};
        bpf_probe_read(&data4.saddr, sizeof(u32), &ip->saddr);
        bpf_probe_read(&data4.daddr, sizeof(u32), &ip->daddr);
        data4.dport = dport;
        data4.sport = sport;
        data4.state = state;
        data4.tcpflags = tcpflags;
        data4.stack_id = stack_traces.get_stackid(ctx, 0);
        ipv4_events.perf_submit(ctx, &data4, sizeof(data4));
[...]

```

My prior tcp tools in bcc have made do with struct sock members alone (eg, [tcplife](http://www.brendangregg.com/blog/2016-11-30/linux-bcc-tcplife.html)). But this time I needed packet info to see TCP flags, and the direction of the packet. So it's the first time I've accessed TCP and IP headers in eBPF. I added skb\_to\_tcphdr() and skb\_to\_iphdr() in tcpdrop to help with this, as well as a new tcp bcc library for later processing in Python. I'm sure this code will get reused (and improved) over time.

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
