---
title: node.js Flame Graphs on Linux
url: https://www.brendangregg.com/blog/2014-09-17/node-flame-graphs-on-linux.html
published: "2014-09-17T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-09-17/node-flame-graphs-on-linux.html
---

# node.js Flame Graphs on Linux

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

## node.js Flame Graphs on Linux

17 Sep 2014

CPU [flame graphs](/flamegraphs.html) are a useful visualization application stack traces, allowing you to quickly identify and quantify what to tune to improve performance. For Node.js they have solved countless problems on systems which have DTrace for sampling stack traces. But what about Linux?

At Netflix we have node.js in production at scale, on Linux instances in AWS EC2, and we create flame graphs using Linux [perf\_events](/perf.html) and v8's --perf\_basic\_prof [option](https://codereview.chromium.org/70013002) (also works as --perf-basic-prof). In this quick blog post, I'll share how it works and how you can do it, and what needs to be fixed to improve it further.

## 1\. The problem

Using perf\_events to profile CPU usage on node.js 0.10.23:

![](/blog/images/2014/node.js_flamegraph_nosymbols_01023.svg)

It's interactive: mouse-over elements for details, and click the [SVG](/blog/images/2014/node.js_flamegraph_nosymbols_01023.svg) to zoom. The [CPU flame graphs](/FlameGraphs/cpuflamegraphs.html) page explains how to interpret these, and this was created using the instructions in the [Linux perf section](/FlameGraphs/cpuflamegraphs.html#perf).

This flame graph is partially-useful, as I can see system and v8 library symbols. However, it is missing JavaScript symbols (the blank rectangles), since v8, like the JVM, compiles and places symbols just in time (JIT).

## 2\. Linux perf\_events JIT support

In 2009, Linux [perf\_events](/perf.html) added [JIT symbol support](https://lkml.org/lkml/2009/6/8/499), so that symbols from language virtual machines like the JVM could be inspected. It works in the following amazingly simple way:

1. Your JIT application must be modified to create a /tmp/perf- *PID*.map file, which is a simple text database containing symbol addresses (in hex), sizes, and symbol names.
2. That's it.

perf already looks for the /tmp/perf- *PID*.map file, and if it finds it, it uses it for symbol translations. So only v8 needed to be modified.

## 3\. v8 --perf-basic-prof support

In November 2013, [v8 added perf\_events support](https://codereview.chromium.org/70013002), enabled using the --perf-basic-prof option. This made it into node v0.11.13. It works like this:

```
# ~/node-v0.11.13-linux-x64/bin/node --perf-basic-prof hello.js &
[1] 31441
# ls -l /tmp/perf-31441.map
-rw-r--r-- 1 root root 81920 Sep 17 20:41 /tmp/perf-31441.map
# tail /tmp/perf-31441.map
14cec4db98a0 f Stub:BinaryOpICWithAllocationSiteStub(ADD_CreateAllocationMementos:String*Generic->String)
14cec4db9920 f Stub:BinaryOpICWithAllocationSiteStub(ADD_CreateAllocationMementos:String*String->String)
14cec4db99a0 f Stub:BinaryOpICWithAllocationSiteStub(ADD_CreateAllocationMementos:String*Smi->String)
14cec4db9a20 22c LazyCompile:~nextTick node.js:389
14cec4db9cc0 156 Stub:KeyedLoadElementStub
14cec4db9e80 22 KeyedLoadIC:
14cec4db9f20 22 KeyedLoadIC:
14cec4db9fc0 56 Stub:DoubleToIStub
14cec4dba080 10c Stub:KeyedStoreElementStub

```

This text file is what perf\_events reads.

## 4\. node.js Flame Graphs

Now that we have node 0.11.13+ running with --perf-basic-prof, we can create a flame graph using:

```
$ sudo bash
# perf record -F 99 -p `pgrep -n node` -g -- sleep 30
# perf script > out.nodestacks01
# git clone --depth 1 http://github.com/brendangregg/FlameGraph
# cd FlameGraph
# ./stackcollapse-perf.pl < ../out.nodestacks01 | ./flamegraph.pl > ../out.nodestacks01.svg

```

You can also use [stackvis](https://www.npmjs.org/package/stackvis), by Dave Pacheco, a node.js implementation which has extra features.

Here's an example result:

![](/blog/images/2014/node.js_flamegraph_symbols_01113.svg)

Note the JavaScript symbols are now readable. Click the [SVG](/blog/images/2014/node.js_flamegraph_symbols_01113.svg) to zoom in. This actual flame graph isn't very interesting, as I'm just testing a dummy app to test out --perf-basic-prof.

Thanks to [Trevor Norris](https://twitter.com/trevnorris) for first posting the instructions for doing this in a short [gist](https://gist.github.com/trevnorris/9616784), which you may find useful to read. He also provides a script to facilitate this.

## WARNING: map file growth

We can currently only use --perf-basic-prof for short periods (hours), due to [bug 3453](https://code.google.com/p/v8/issues/detail?id=3453): the perf.map file can grow endlessly, eating Gbytes in a few days. It looks like symbols are moving location (they are supposed to stay put with --perf-basic-prof), causing the map file to keep growing.

UPDATE (2016): A new option, --perf\_basic\_prof\_only\_functions (or --perf-basic-prof-only-functions) was introduced to address this bug by only logging interesting types of symbols, cutting down on map file growth. If map file growth is a problem for you, try out this option instead.

## More

We're doing more at Netflix with node.js analysis. Stay tuned, and also see the [Netflix Tech Blog](http://techblog.netflix.com/).

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
