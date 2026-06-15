---
title: 'Coloring Flame Graphs: Code Hues'
url: https://www.brendangregg.com/blog/2017-07-30/coloring-flamegraphs-code-type.html
published: "2017-07-30T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2017-07-30/coloring-flamegraphs-code-type.html
---

# Coloring Flame Graphs: Code Hues

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

## Coloring Flame Graphs: Code Hues

30 Jul 2017

I recently improved flame graph code coloring. If you're automating or implementing flame graphs, this is a small detail that may interest you. (For an intro to flame graphs, see my [website](http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html#Java) and [github](https://github.com/brendangregg/FlameGraph).)

First, a confession. Code-type coloring was a regex hack that took five minutes. In late 2014 I was modifying the JDK to preserve the frame pointer so that traditional stack walkers and profilers would work (an example of the problem is [here](/blog/images/2017/out.vertx01.svg), where Java methods lack ancestry). After I fixed the frame pointer, profiling Java looked like this ( [SVG](/blog/images/2017/out.brendanjdk_vertx03.svg)):

![](/blog/images/2017/out.brendanjdk_vertx03.svg)

It worked! Java methods now had ancesty (stack depth), and appear as towers.

I was delighted and showed my colleagues straight away. Amer Ather, another performance engineer at Netflix, suggested I color the Java and kernel frames differently. He was only back at his desk for five minutes when I called him back ( [SVG](/blog/images/2017/out.brendanjdk_vertx03color.svg)):

![](/blog/images/2017/out.brendanjdk_vertx03color.svg)

Done. (I also stripped the extra L from Java symbols.)

My hack was the following eight lines of code:

```
        if (defined $type and $type eq "java") {
                if ($name =~ /::/) {            # C++
                        $type = "yellow";
                } elsif ($name =~ m:/:) {       # Java (match "/" in path)
                        $type = "green"
                } else {                        # system
                        $type = "red";
                }

```

The "java" $type is from the command line option: `--color=java`. The $name is the function name. Here are some sample function names:

- **Java**

`io/netty/channel/nio/NioEventLoop;.run`

`org/mozilla/classfile/ClassFileWriter;.addLoadConstant`
- **C++** `JavaCalls::call_helper`

`JavaThread::thread_main_inner`
- **C** `tcp_v4_do_rcv`

`start_thread`

`write`

If you cast your regular expression eye over these, you'll quickly see patterns. If it contains "::" it's C++, "/" it's Java, else it's C. And that's what I coded.

It mostly worked. But I've noticed the odd case where it gets things wrong. Sometimes the profiled Java symbols use "." instead of "/" as a delimiter. Or, somehow, I have Java methods that lack any package delimiter, so were colored red. I had similar issues with JIT'd code for Node.js.

Revisiting how flame graphs for Linux perf are generated (full instructions in [Java Flame Graphs](http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html#Java)):

```
perf record -F 49 -a -g -- sleep 30; ./jmaps
perf script | ./stackcollapse-perf.pl | grep -v cpu_idle | ./flamegraph.pl --color=java > out.svg

```

It's beginning with the output of `perf script` (later perf versions added a way to emit a folded summary directly). Here is some truncated `perf script` output:

```
java  4811 cpu-clock:
    ffffffff8100122a hypercall_page ([kernel.kallsyms])
    ffffffff8100aca2 check_events ([kernel.kallsyms])
    ffffffff8104dffe __wake_up_sync_key ([kernel.kallsyms])
    ffffffff8152f86e sock_def_readable ([kernel.kallsyms])
[...]
    ffffffff81662142 system_call_fastpath ([kernel.kallsyms])
        7f62aadf2f7d write (/lib/x86_64-linux-gnu/libc-2.15.so)
        7f62961a5e8b Lsun/nio/ch/FileDispatcherImpl;.write0(Ljava/io/FileDescriptor;JI)I (/tmp/perf-4637.map)
        7f629619dd64 Lsun/nio/ch/SocketDispatcher;.write(Ljava/io/FileDescriptor;JI)I (/tmp/perf-4637.map)
        7f62961b3330 Lsun/nio/ch/IOUtil;.writeFromNativeBuffer(Ljava/io/FileDescriptor;Ljava/nio/ByteBuffer;JLsun/nio/ch/NativeDispatcher;)I (/tmp/perf-4637.map)
[...]
        7f62aa3b1618 JavaThread::thread_main_inner() (/mnt/openjdk8/build/linux-x86_64-normal-server-release/jdk/lib/amd64/server/libjvm.so)
        7f62aa3b186c JavaThread::run() (/mnt/openjdk8/build/linux-x86_64-normal-server-release/jdk/lib/amd64/server/libjvm.so)
        7f62aa272bf2 java_start(Thread*) (/mnt/openjdk8/build/linux-x86_64-normal-server-release/jdk/lib/amd64/server/libjvm.so)
        7f62aa8f2e9a start_thread (/lib/x86_64-linux-gnu/libpthread-2.15.so)

```

The stackcollapse-perf.pl tool plucks out the symbol name (second column) and discards everything else. But the last column – the segment printed in ( ) – provides more details for identifying code types. Eg:

- **`[kernel.kallsyms]`**: kernel code (I could also match the addr vs the kernel base address for this)
- **`/tmp/perf-PID.map`**: JIT'd code (Java, Node.js, ...)

This is what I made use of recently, by adding an `--all` option to stackcollapse-perf.pl to turn on all annotations. Annotations are inspired by the "\[k\]" annotations seen in `perf report --stdio` output. I append them after the function name, so `tcp_sengmsg` becomes `tcp_sengmsg_[k]`, and that annotation is used and then stripped by flamegraph.pl.

Annontation suffixes:

- **`_[k]`**: kernel
- **`_[j]`**: JIT
- **`_[i]`**: inlined function
- **`_[w]`**: waker stack (for [offwake](http://www.brendangregg.com/blog/2016-02-01/linux-wakeup-offwake-profiling.html) or [chain](http://www.brendangregg.com/blog/2016-02-05/ebpf-chaingraph-prototype.html) graphs)

Making use of both annotations and pattern matching, the "java" palette is now:

- green: JIT (Java, Node.js, ...)
- aqua: inlined
- yellow: C++
- orange: kernel
- red: native (user-level)

If you're automating flame graphs using my original tools, you might want to consider adding `--all` to the normal workflow for annotations. These are currently used by the "java" and "js" palettes. Eg:

```
perf record -F 49 -a -g -- sleep 30; ./jmaps
perf script | ./stackcollapse-perf.pl --all | grep -v cpu_idle | ./flamegraph.pl --color=java > out.svg

```

If you are using a different profiler (not Linux perf), you might want to consider enhancing its stackcollapse program to have an option to turn on annotations (or I can do it next time I use them). If you are implementing your own flame graph software, you might want to add similar color hues for code types.

Finally, it should be clear that changing the hue of code based on a regex is a trivial change to flamegraph.pl. You could add custom rules to your version to highlight your team's code, for example.

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
