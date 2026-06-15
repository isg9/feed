---
title: 'Learn eBPF Tracing: Tutorial and Examples'
url: https://www.brendangregg.com/blog/2019-01-01/learn-ebpf-tracing.html
published: "2019-01-01T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2019-01-01/learn-ebpf-tracing.html
---

# Learn eBPF Tracing: Tutorial and Examples

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

## Learn eBPF Tracing: Tutorial and Examples

01 Jan 2019

At the Linux Plumber's conference there were at least 24 talks on eBPF. It has quickly become not just an invaluable technology, but also an in-demand skill. Perhaps you'd like a new year's resolution: learn eBPF!

eBPF should stand for something meaningful, like Virtual Kernel Instruction Set (VKIS), but due to its origins it is extended Berkeley Packet Filter. It can be used for many things: network performance, firewalls, security, tracing, and device drivers. Some of these have plenty of free documentation online, like for tracing, and others not yet. The term tracing refers to performance analysis and observability tools that can produce per-event info. You may have already use a tracer: `tcpdump` and `strace` are specialized tracers.

In this post I'll cover learning eBPF for tracing, grouped into content for beginner, intermediate, and advanced users. In summary:

- Beginner: run [bcc](https://github.com/iovisor/bcc) tools
- Intermediate: develop [bpftrace](https://github.com/iovisor/bpftrace) tools
- Advanced: develop [bcc](https://github.com/iovisor/bcc) tools, contribute to bcc & bpftrace

Update: I have a new book about eBPF tracing, published by Addison Wesley: [BPF Performance Tools: Linux System and Application Observability](http://www.brendangregg.com/bpf-performance-tools-book.html).

# Beginner

## 1\. What is eBPF, bcc, bpftrace, and iovisor?

**eBPF** does to Linux what JavaScript does to HTML. (Sort of.) So instead of a static HTML website, JavaScript lets you define mini programs that run on events like mouse clicks, which are run in a safe virtual machine in the browser. And with eBPF, instead of a fixed kernel, you can now write mini programs that run on events like disk I/O, which are run in a safe virtual machine in the kernel. In reality, eBPF is more like the v8 virtual machine that runs JavaScript, rather than JavaScript itself. eBPF is part of the Linux kernel.

Programming in eBPF directly is incredibly hard, the same as coding in v8 bytecode. But no one codes in v8: they code in JavaScript, or often a framework on top of JavaScript (jQuery, Angular, React, etc). It's the same with eBPF. People will use it and code in it via frameworks. For tracing, the main ones are **[bcc](https://github.com/iovisor/bcc)** and **[bpftrace](https://github.com/iovisor/bpftrace)**. These don't live in the kernel code base, they live in a Linux Foundation project on github called **iovisor**.

## 2\. What is an example of eBPF tracing?

This eBPF-based tool shows completed TCP sessions, with their process ID (PID) and command name (COMM), sent and received bytes (TX\_KB, RX\_KB), and duration in milliseconds (MS):

```
# tcplife
PID   COMM       LADDR           LPORT RADDR           RPORT TX_KB RX_KB MS
22597 recordProg 127.0.0.1       46644 127.0.0.1       28527     0     0 0.23
3277  redis-serv 127.0.0.1       28527 127.0.0.1       46644     0     0 0.28
22598 curl       100.66.3.172    61620 52.205.89.26    80        0     1 91.79
22604 curl       100.66.3.172    44400 52.204.43.121   80        0     1 121.38
22624 recordProg 127.0.0.1       46648 127.0.0.1       28527     0     0 0.22
3277  redis-serv 127.0.0.1       28527 127.0.0.1       46648     0     0 0.27
22647 recordProg 127.0.0.1       46650 127.0.0.1       28527     0     0 0.21
3277  redis-serv 127.0.0.1       28527 127.0.0.1       46650     0     0 0.26
[...]

```

eBPF *did not* make this possible – I can rewrite tcplife to use older kernel technologies. But if I did, we'd never run such a tool in production due to the performance overhead, security issues, or both. What eBPF did was make this tool *practical*: it is efficient and secure. For example, it does not trace every packet like older techniques, which can add too much performance overhead. Instead, it only traces TCP session events, which are much less frequent. This makes the overhead so low we can run this tool in production, 24x7.

## 3\. How do I use it?

For beginners, try the tools from bcc. See the [bcc install instructions](https://github.com/iovisor/bcc/blob/master/INSTALL.md) for your OS. On Ubuntu, it may be like:

```
# sudo apt-get update
# sudo apt-get install bpfcc-tools
# sudo /usr/share/bcc/tools/opensnoop
PID    COMM               FD ERR PATH
25548  gnome-shell        33   0 /proc/self/stat
10190  opensnoop          -1   2 /usr/lib/python2.7/encodings/ascii.x86_64-linux-gnu.so
10190  opensnoop          -1   2 /usr/lib/python2.7/encodings/ascii.so
10190  opensnoop          -1   2 /usr/lib/python2.7/encodings/asciimodule.so
10190  opensnoop          18   0 /usr/lib/python2.7/encodings/ascii.py
10190  opensnoop          19   0 /usr/lib/python2.7/encodings/ascii.pyc
25548  gnome-shell        33   0 /proc/self/stat
29588  device poll         4   0 /dev/bus/usb
^C

```

There I finished by running opensnoop to test that the tools worked. If you get this far, you've used eBPF!

Companies including Netflix and Facebook have bcc installed on all servers by default, and maybe you'll want to as well.

## 4\. Is there a beginner tutorial?

Yes, I created a bcc tutorial, which is a good starting point for beginners to eBPF tracing:

- [bcc Tutorial](https://github.com/iovisor/bcc/blob/master/docs/tutorial.md)

As a beginner, you do not need to write any eBPF code. bcc comes with over 70 tools that you can use straight away. The tutorial steps you through eleven of these: execsnoop, opensnoop, ext4slower (or btrfs\*, xfs\*, zfs\*), biolatency, biosnoop, cachestat, tcpconnect, tcpaccept, tcpretrans, runqlat, and profile.

Once you've tried these, you just need to be aware that many more exist:

[![](http://www.brendangregg.com/Perf/bcc_tracing_tools.png)](http://www.brendangregg.com/Perf/bcc_tracing_tools.png)

These are also fully documented with man pages and example files. The example files (\*\_example.txt in bcc/tools) show screenshots with explanations: for example, [biolatency\_example.txt](https://github.com/iovisor/bcc/blob/master/tools/biolatency_example.txt). I wrote many of these example files (and man pages, and tools), which are like an extra 50 blog posts you'll find in the bcc repo.

What's missing is production examples. I wrote all this documentation when eBPF was so new it was only available on our test instances, so most of the examples are synthetic. Over time we'll add real world examples, and it's an area beginners can help: if you solve an issue with a tool, consider publishing a post to share the screenshots, or adding them to the example files.

# Intermediate

At this point you should have bcc running and have tried the tools, and you're interested in customizing them and writing your own tools. The best path is to switch to bpftrace, which has a high-level language that is *much* easier to learn. The downside is that it's not as customizable as bcc, so you may eventually run into limitations and want to switch back to bcc.

See the [bpftrace install instructions](https://github.com/iovisor/bpftrace/blob/master/INSTALL.md). It's a newer project, so it isn't packaged everywhere at the time of writing this post. In the future, it should just be an `apt-get install bpftrace` or equivalent.

## 1\. bpftrace Tutorial

I developed a tutorial that teaches bpftrace via a series of one-liners:

- [bpftrace One-Liners Tutorial](https://github.com/iovisor/bpftrace/blob/master/docs/tutorial_one_liners.md)

This provides 12 lessons that will teach you bpftrace bit by bit. An example screenshot:

```
# bpftrace -e 'tracepoint:syscalls:sys_enter_open { printf("%d %s\n", pid, str(args->filename)); }'
Attaching 1 probe...
181 /proc/cpuinfo
181 /proc/stat
1461 /proc/net/dev
1461 /proc/net/if_inet6
^C

```

That's using the open syscall tracepoint to trace the PID and path opened.

## 2\. bpftrace Reference Guide

For more on bpftrace, I created the reference guide which has examples for the syntax, probes, and builtins:

- [bpftrace Reference Guide](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md)

It's terse on purpose: as much as possible I fit a topic heading, summary, and screenshot all on one screen. If you look up something and need to hit page down several times, I think it's too long.

## 3\. bpftrace Examples

There are over 20 tools in the bpftrace repository that you can browse as examples:

- [bpftrace Tools](https://github.com/iovisor/bpftrace/tree/master/tools)

For example:

```
# cat tools/biolatency.bt
[...]
BEGIN
{
    printf("Tracing block device I/O... Hit Ctrl-C to end.\n");
}

kprobe:blk_account_io_start
{
    @start[arg0] = nsecs;
}

kprobe:blk_account_io_completion
/@start[arg0]/

{
    @usecs = hist((nsecs - @start[arg0]) / 1000);
    delete(@start[arg0]);
}

```

Like bcc, these tools have man pages and example files. For example, [biolatency\_example.txt](https://github.com/iovisor/bpftrace/blob/master/tools/biolatency_example.txt).

# Advanced

## 1\. Learn bcc Development

I've created two documents to help here:

- [bcc Python Developer Tutorial](https://github.com/iovisor/bcc/blob/master/docs/tutorial_bcc_python_developer.md)
- [bcc Reference Guide](https://github.com/iovisor/bcc/blob/master/docs/reference_guide.md)

There's also lots of examples under bcc/tools/\*.py. bcc tools are of two parts: the BPF code for the kernel, written in C, and the user-space tool written in Python (or lua, or C++). Developing bcc tools is somewhat advanced, and may involve some gritty kernel or application internals.

## 2\. Contribute

Help is appreciated:

- [bcc issues](https://github.com/iovisor/bcc/issues)
- [bpftrace issues](https://github.com/iovisor/bpftrace/issues)

For bpftrace, I created a [bpftrace Internals Development Guide](https://github.com/iovisor/bpftrace/blob/master/docs/internals_development.md). It gets hard as you're coding in llvm IR, but if you're up for a challenge...

There's also the kernel eBPF (aka BPF) engine: if you browse bcc and bpftrace issues, you'll see some requests for enhancements there. Eg, the [bpftrace kernel tag](https://github.com/iovisor/bpftrace/issues?q=is%3Aissue+is%3Aopen+label%3Akernel). Also watch the [netdev](https://www.spinics.net/lists/netdev/) mailing list for the latest kernel BPF developments, which are added to net-next before they are merged into mainline Linux.

Apart from writing code, you can also contribute with testing, packaging, blog posts, and talks.

# Summary

eBPF does many things. In this post I covered how to learn eBPF for tracing and performance analysis. In summary:

- Beginner: run [bcc](https://github.com/iovisor/bcc) tools
- Intermediate: develop [bpftrace](https://github.com/iovisor/bpftrace) tools
- Advanced: develop [bcc](https://github.com/iovisor/bcc) tools, contribute to bcc & bpftrace

I also have a dedicate page on **[eBPF Tracing Tools](http://www.brendangregg.com/ebpf.html)** that covers these in more detail. Good luck!

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
