---
title: Linux BPF/bcc Road Ahead, March 2016
url: https://www.brendangregg.com/blog/2016-03-28/linux-bpf-bcc-road-ahead-2016.html
published: "2016-03-28T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-03-28/linux-bpf-bcc-road-ahead-2016.html
---

# Linux BPF/bcc Road Ahead, March 2016

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

## Linux BPF/bcc Road Ahead, March 2016

28 Mar 2016

This summarizes the current state of Linux BPF for system tracing, and the bcc front-end. BPF is a Linux mainline technology for event tracing and manipulation (formally eBPF for extended BPF), which has had many features added in the Linux 4.x series. bcc is an open source Python front-end.

## Linux Now (4.6-rc1)

The latest Linux kernel (4.6-rc1) supports the following:

- dynamic tracing, kernel-level (BPF support for kprobes)
- dynamic tracing, user-level (BPF support for uprobes)
- filtering (via BPF programs)
- debug output (bpf\_trace\_printk())
- per-event output (bpf\_perf\_event\_output())
- basic variables (global & per-thread variables, via BPF maps)
- associative arrays (via BPF maps)
- frequency counting (via BPF maps)
- histograms (power-of-2, linear, and custom, via BPF maps)
- timestamps and time deltas (bpf\_ktime\_get\_(), and BPF programs)
- stack traces, kernel (BPF stackmap)\*
- stack traces, user (BPF stackmap)\*

And in user-level, the bcc package currently adds:

- debug output (Python with BPF.trace\_pipe() and BPF.trace\_fields())
- per-event output (BPF\_PERF\_OUTPUT macro and BPF.open\_perf\_buffer())
- interval output (BPF.get\_table() and table.clear())
- histogram printing (table.print\_log2\_hist())
- C struct navigation, kernel-level (maps to bpf\_probe\_read())
- symbol resolution, kernel-level (ksym(), ksymaddr())
- symbol resolution, user-level (usymaddr())\*
- BPF tracepoint support (via bcc Tracepoint.enable\_tracepoint())\*
- BPF stack trace support (incl. walk method for stack flames)\*
- various other helper macros and functions
- many tools (under /tools)

\\* these were added in the last few [weeks](https://github.com/iovisor/bcc/commits/master) (thanks Alexei Starovoitov, Sasha Goldshtein, and Vicent Marti).

## The Road Ahead

The following are the major items left to add to the Linux kernel, for BPF-based tracing:

- static tracing, kernel-level (BPF support for tracepoints [#232](https://github.com/iovisor/bcc/issues/232); in progress by Alexei Starovoitov)
- timed sampling events (BPF support [#330](https://github.com/iovisor/bcc/issues/330))
- overwrite ring buffers (in progress by Wang Nan)

And to add to bcc:

- static tracing, user-level (USDT probes via uprobes [#327](https://github.com/iovisor/bcc/issues/327); in progress by Sasha Goldshtein)
- more helper routines for ease of use, but not too many (in progress by Brenden Blanco and others)
- more tools (in progress by me and others)
- more tests

Wow. I first drafted this post a few months ago, when it felt like we were at the half way mark. Many things have been completed since then, and I've updated this post. It's getting really exciting – the finish is in sight.

## Other Misc Work

There's also some other related and optional work worth mentioning:

- llvm BPF debug enhancements (incl. better error messages)
- llvm 32-bit subregisters
- BPF accounting (/proc for what is running)
- GUI for bcc (heat maps, flame graphs, etc; Netflix Vector will be one)
- TUI for bcc (like atop)
- bcc built-in compiler instead of llvm/clang
- Higher-level language support (see [#425](https://github.com/iovisor/bcc/issues/425))
- BPF/bcc optimizations, minor features, and bug fixes as needed (in progress by Alexei, Brenden, ...).

Plus new features we haven't thought of yet.

## How You Can Help

### 1\. Promotion

Most people have yet to hear of BPF or bcc. You can help by promoting them, especially to anyone doing performance work. Here are some links so far to share:

- [https://github.com/iovisor/bcc#tools](https://github.com/iovisor/bcc#tools) browse the "example" links
- [http://www.brendangregg.com/blog/2016-03-05/linux-bpf-superpowers.html](http://www.brendangregg.com/blog/2016-03-05/linux-bpf-superpowers.html) has BPF links
- [https://www.kernel.org/doc/Documentation/networking/filter.txt](https://www.kernel.org/doc/Documentation/networking/filter.txt)

The 2nd URL has my recent BPF talk ( [Facebook](https://www.facebook.com/atscaleevents/videos/1693888610884236/), 30 mins):

> [Linux 4.x Performance: Using BPF Superpowers (Brendan Gregg)](https://www.facebook.com/atscaleevents/videos/1693888610884236/)
>
> Posted by [At Scale](https://www.facebook.com/atscaleevents/) on Friday, February 26, 2016

### 2\. Kernel BPF work

If you've done Linux kernel work before and can contribute in this area, keep an eye on lkml posts containing BPF, and become familiar with the samples under samples/bpf. You can run these standalone. To get started, I get [llvm setup](https://gist.github.com/tuxology/357d8826e97eb72c9277) in /opt/local then do this with a pre-built kernel source:

```
cd linux-4....
vi samples/bpf/Makefile   # change LLC to LLC=/opt/local/llvm/bin/llc or whatever
make samples/bpf/
cd samples/bpf/
./tracex1

```

You can also contribute with code reviews and testing, which is often the bottleneck.

### 3\. bcc Python/C/assembly work

Browse the [bcc issues](https://github.com/iovisor/bcc/issues) and help out where you can. If you are very good at Python, or C, or assembly, you might notice areas that can be improved just by browsing the code. Our focus so far has been getting BPF and bcc capable on just x86\_64, but there's going to be a lot of additional work to get it working on all processor types. Eg, aliases for registers ( [#225](https://github.com/iovisor/bcc/issues/225)).

### 4\. Distro Packaging

There need to be bcc packages for different Linux distros and repositories. bcc has a build-time dependency on clang/llvm, but when packaged right it shouldn't have any install-time dependencies – it should be an easy package add.

### 5\. bcc Testing

Test the tools in bcc, and file detailed bugs as you find them.

### 6\. bcc Documentation

There isn't much documentation yet on bcc/BPF, as it's an evolving interface. In the coming months the interface will be settling down, and we can write a reference guide, a tutorial, specific distro install instructions, etc. If you're totally new to bcc/BPF, you'll have a valuable perspective as you'll encounter every detail that must be learned in order to use it. Make notes, and please help with the docs.

In particular, the Ubuntu 16.04LTS release will be the first Ubuntu release with substantial BPF support, and we need docs to show how people can get started. Not everything is available in its 4.4 kernel, but there's enough for a significant preview of BPF capabilities. A couple of dozen tools, at least, should work.

### 7\. Use cases

I wouldn't encourage everyone to start coding their own bcc tools just yet, as the interface still has rough edges. But if you have some tolerance for really gritty engineering work, then publishing early use cases will help the project in many ways.

If you do try bcc and find it too frustrating or difficult, then check again in a few months. It keeps getting better.

Thanks to everyone who has contributed so far!

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
