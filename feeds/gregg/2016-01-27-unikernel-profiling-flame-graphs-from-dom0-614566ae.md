---
title: 'Unikernel Profiling: Flame Graphs from dom0'
url: https://www.brendangregg.com/blog/2016-01-27/unikernel-profiling-from-dom0.html
published: "2016-01-27T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-01-27/unikernel-profiling-from-dom0.html
---

# Unikernel Profiling: Flame Graphs from dom0

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

## Unikernel Profiling: Flame Graphs from dom0

27 Jan 2016

Is a unikernel an impenetrable black box, resistant to profilers, tracers, fire, and poison? At [SCaLE14x](https://www.socallinuxexpo.org/scale/14x) this weekend there was a full day track on unikernels, after which I was asked about unikernel profiling and tracing. I'm not an expert on the topic, and wasn't able to answer these questions at the time, however, I've since taken a quick look using [MirageOS](https://mirage.io/) and [Xen](http://www.xenproject.org).

In this post I'll share a proof of concept for profiling a Xen domU unikernel from dom0. I didn't find any recent examples of doing this (there are some 2006 oprofile patches). This is just a PoC, not yet a polished product. Although, by the time you read this, there may be something better based on this work (check comments).

I'll begin by running a unikernel as a normal process for profiling, and then as a Xen guest.

## Normal Process Profiling

Voila, a CPU flame graph ( [SVG](/blog/images/2016/mirageos-www-flamegraph.svg))!

![](/blog/images/2016/mirageos-www-flamegraph.svg)

This is the MirageOS [www example](https://mirage.io/wiki/mirage-www), compiled and executed in "Unix binary" mode, where it runs as a normal process under Linux -- no hypervisor:

```
# mirage configure --unix
# make
# ./mir-www &

```

I then profiled the process using Linux [perf\_events](http://www.brendangregg.com/perf.html). See the OCaml docs [Using perf on Linux](http://ocaml.org/learn/tutorials/performance_and_profiling.html#UsingperfonLinux), and my docs on [CPU flame graphs](http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html). The steps were:

```
# perf record -F 99 -a --call-graph=dwarf -- sleep 10
# git clone https://github.com/brendangregg/FlameGraph
# perf script | ./FlameGraph/stackcollapse-perf.pl --kernel | \
    ./FlameGraph/flamegraph.pl --color=java > out.svg

```

The `--call-graph=dwarf` only works on newish Linux. On older Linux you'll need to use a `+fp` (frame pointer) version of the ocaml compiler. I also used the java palette here, because it highlights kernel-mode as orange and user-mode as red. These will not be separate modes once I start running this under Xen.

Does Unix binary profiling matter? A few scenarios come to mind:

- The developer writes code that is picked up by a build system and compiled straight to Xen. In this scenario, there is no chance for this type of profiling.
- Same as above, but with an extra build step added for the Unix binary, for running with a test suite and profiler before Xen compilation.
- A developer may use Unix binary builds as a normal step for testing.

In the latter two cases, this type of profiling can be used to catch a variety of issues, before running under Xen. You can also use any other tools to debug the binary unikernel ( [strace](http://www.brendangregg.com/blog/2014-05-11/strace-wow-much-syscall.html), etc).

This Unix binary build will execute differently to a Xen build for a number of reasons. A Xen unikernel will use a single address space without context switching, hypercalls for various resources, and will run a different body of kernel code (TCP/IP, etc). It's very possible that some performance issues may only manifest when running under Xen, so you will want to be able to profile it there as well.

## xenctl stack walking

As I found no examples of stack trace profiling, I did start wondering if this was even possible. The following shows one approach I've found that works; I now suspect other approaches are also possible.

### 1\. Compile unikernel with frame pointers

There's more than one way to walk a stack. The easiest is usually via frame pointers, provided they are preserved by the compiler.

For MirageOS, the default ocaml compiler omits the frame pointer, but it can be returned by switching to a `+fp` ocaml compiler (this is like needing `-fno-omit-frame-pointer` for gcc). The steps I used were (also follow the [mirage install](https://mirage.io/wiki/install) docs):

```
# opam switch 4.02.3+fp
# eval `opam config env`
# opam install mirage

```

Then I compiled and ran the unikernel as a Xen guest, going back to the [console](https://mirage.io/wiki/hello-world) example (which I hacked to make it hotter on-CPU):

```
# mirage configure --xen
# make
# xl create console.xl

```

### 2\. Create a symbol file

This can be anything that has memory addresses and function names, and will be used to translate the instruction pointer addresses in stack traces. If you don't have this working, you'll get hexadecimal numbers.

I found an `objdump` of the MirageOS compiled object to be suitable (didn't even need transposing, although it may for other unikernels/builds), and used some awk process the output:

```
dom0# objdump -x mir-console.xen | awk '{ print $1, $2, $NF }' > /tmp/xen.map
dom0# more /tmp/xen.map
[...]
00000000000d04c0 l caml_negf_mask
00000000000d04d0 l caml_absf_mask
00000000000f4378 l camlConsole_xen__370
00000000000f4398 l camlConsole_xen__371
00000000000f43b8 l camlConsole_xen__372
[...]

```

### 3\. Take a stack trace snapshot

The xenctx command can be executed from dom0 to dump registers and the call stack of a domU. You might find it under /usr/lib/xen-4.4/bin/xenctx, or in the Xen source under tools/xentrace.

Here I'm using the `-C` option prints all CPUs, `-s` to specify my symbol file, and a domU ID of 26. I'd guess for some unikernels you may need to use `-k` to change the kernel base address, although I didn't need to here:

```
dom0# xenctx -C -s /tmp/xen.map 26
vcpu0:
rip: 0000000000049ad7 camlLwt__return_1319+0x27
flags: 00000206 i nz p
rsp: 000000000019fe50
rax: 000000000014e198   rcx: 0000000000350020   rdx: 000000000004a464
rbx: 0000000000000001   rsi: 0000000000007270   rdi: 000000000044e9c8
rbp: 000000000019fe50    r8: 000000000000000d    r9: 000000000000000d
r10: 000000000015e5f0   r11: 0000000000000010   r12: 00000000000f1150
r13: 0000000000590c78   r14: 000000000019feb0   r15: 000000000044e9c0
 cs: e033    ss: e02b    ds: 0000    es: 0000
 fs: 0000 @ 0000000000000000 .comment
 gs: 0000 @ 000000000015f4b0/0000000000000000 cpu0_pda/ .comment
Code (instr addr 00049ad7)
d2 46 10 00 4c 3b 38 72 24 49 8d 7f 08 48 c7 47 f8 00 04 00 00 <48> 89 1f 48 8d 47 10 48 c7 40 f8

Stack:
 000000000019fe70 0000000000007312 000000000044e9e8 0000000000000003
 000000000019fe80 0000000000007154 000000000019fe90 000000000000725d
 000000000019fea0 000000000000358d 000000000019ff50 00000000000b07b2
 0000000000000000 00000000000b07e9 0000000000000000 0000000000000001
 0000000000000000 00000000000af871 00000008000f4000 0000000000000000

Call Trace:
                    [<0000000000049ad7>] camlLwt__return_1319+0x27 <--
000000000019fe58:   [<0000000000007312>] camlUnikernel____pa_lwt_loop_1376+0x72
000000000019fe68:   [<0000000000000003>] _start+0x3
000000000019fe78:   [<0000000000007154>] camlMain__fun_1404+0x14
000000000019fe88:   [<000000000000725d>] camlMain__entry+0xcd
000000000019fe98:   [<000000000000358d>] caml_program+0x58d
000000000019fea8:   [<00000000000b07b2>] caml_start_program+0x4a
000000000019feb0:   [<0000000000000000>] .comment
[...]

```

Excellent, we have a call trace and translated frames. If I can collect many of these, I'm profiling!

Unikernels make profiling a bit easier, for a couple of reasons:

- No separate kernel-/user-mode stacks. We only need to walk one stack for everything.
- No separate processes. We only need one symbol map for translating addresses.

I thought I might have to write my own stack walker, but xenctx can already do this. If I execute xenctx many times, I'll have a rough PoC.

## Dom0 to DomU: Proof of Concept

Here's a CPU flame graph profile of my simple test program, compiled as a Unix binary and profiled using Linux perf\_events ( [SVG](/blog/images/2016/mirageos-looper-perf-flamegraph.svg)):

![](/blog/images/2016/mirageos-looper-perf-flamegraph.svg)

Now the same program as a Xen domU unikernel, profiled using xenctx in a loop ( [SVG](/blog/images/2016/mirageos-looper-xenctx-flamegraph.svg)):

![](/blog/images/2016/mirageos-looper-xenctx-flamegraph.svg)

I used a xenctx wrapper ( [xenctx\_prof.sh](https://gist.github.com/brendangregg/5c9d55de3ca3c6737933)) which uses awk to scrape the output. I'd executed it, and made the flame graph, in the following vastly inefficient way:

```
dom0# (for i in `seq 1 1000`; do
    ./xenctx_prof.sh -s /tmp/xen.map 26; echo 1; echo; sleep 0.01; done) > out.stacks
dom0# ./stackcollapse.pl < out.stacks | ./flamegraph.pl > out.svg
...

```

Please don't actually run this. See the next section.

## Dom0 to DomU: Profiler

The xenctx source can be made into a real profiler by adding something like:

> for (int sample = 0; sample < max; sample++) { ... print stack only ...; usleep(10000); }

Command line options can also be added: I'd suggest "-F frequency", and a "-T duration". I'd also test it to get a handle on its overhead and safety, and then document expectations clearly. It does do a xc\_domain\_pause(), which I'm guessing introduces latency. This may also need some optimization to reduce that latency.

Unfortunately, developing a unikernel profiler isn't a priority for me (my company is not currently using unikernels), so won't be doing this right now. You are welcome to do this yourself, if this interest you!

## DomU profiler

What if you don't have access to dom0? Ideally, we have a domU-from-domU profiler.

For MirageOS, the ocaml compiler source includes stack trace routines for printing exception stacks. This can serve as example code for walking stack traces, and could be made into a profiler interface that worked similar to tools like Java Flight Recorder / Java Mission Control. These attach to a port to enable and disable profiling, and to collect profile data.

## Summary

I've spent a few hours with MirageOS, and was able to profile a unikernel as a Unix binary with traditional tools (Linux perf\_events), and then develop a proof of concept dom0 to domU unikernel profiler, for profiling a Xen guest unikernel. For both approaches, I was able to create CPU flame graphs.

I'm tempted to spend a bit more time and polish this dom0 profiler, rewriting it entirely in C. However, it may be more practical to write a domU-from-domU profiler instead. I wouldn't be surprised if one already existed (I'd been exploring the dom0 approach).

For more reading about unikernel observability, check out Thomas Leonard's [trace visualizations](http://roscidus.com/blog/blog/2014/10/27/visualising-an-asynchronous-monad/).

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
