---
title: Flame Graphs vs Tree Maps vs Sunburst
url: https://www.brendangregg.com/blog/2017-02-06/flamegraphs-vs-treemaps-vs-sunburst.html
published: "2017-02-06T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2017-02-06/flamegraphs-vs-treemaps-vs-sunburst.html
---

# Flame Graphs vs Tree Maps vs Sunburst

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

## Flame Graphs vs Tree Maps vs Sunburst

06 Feb 2017

Yesterday I posted about [flame graphs for file systems](/blog/2017-02-05/file-system-flame-graph.html), showing how they can visualize where disk space is consumed. Many people have [responded](https://news.ycombinator.com/item?id=13574825), citing other tools and visualizations they prefer: du, ncdu, treemaps, and the sunburst layout. Since there's so much interest in this subject, I've visualized the same files here (the source for linux 4.9.-rc5) in different ways for comparison.

## Flame Graphs

Using [FlameGraph](https://github.com/brendangregg/FlameGraph) ( [SVG](/blog/images/2017/files_linux49.svg)):

![](/blog/images/2017/files_linux49.svg)

While you can mouse-over and click to zoom, at first glance the long labeled rectangles tell the big picture, by comparing their lengths and looking at the longest first:

[![](/blog/images/2017/linux_flamegraph02.png)](/blog/images/2017/linux_flamegraph02.png)

The drivers directory looks like it's over 50%, with drivers/net about 15% of the total. Many small rectangles are too thin to label, and, they also matter less overall. You can imagine printing the flame graph on paper, or including a screen shot in a slide deck, and it will still convey many high level details in not much space. Here's an [example](https://twitter.com/ppcelery/status/828779560376299521) someone just posted to twitter.

## Tree Map

Using [GrandPerspective](https://sourceforge.net/projects/grandperspectiv/?source=typ_redirect) (on OSX):

[![](/blog/images/2017/linux_treemap01.png)](/blog/images/2017/linux_treemap01.png)

What can you tell on first glance? Not those big picture details (drivers 50%, etc). You can mouse over tree map boxes to get more details, which this screenshot doesn't convey. It is, however, easier to see that there are a handful of large files with those boxes in the top left, which are under drivers/gpu/drm/amd.

Using [Baobab](http://www.marzocca.net/linux/baobab/index.html) on Linux:

[![](/blog/images/2017/linux_treemap03.png)](/blog/images/2017/linux_treemap03.png)

You can see that the drivers directory is large from the tree list on the left, which includes mini bar graphs for a visual line length comparison (good). You can't see into subdirectories without clicking to expand.

[![](/blog/images/2017/linux_treemap05.png)](/blog/images/2017/linux_treemap03.png)

Here I've highlighted the drivers/net box. What percentage is that of total from first glance? It's a little bit more difficult to compare sizes than lengths (compare to earlier).

This is also missing labels, when compared to the flame graph, although other tree maps like [SpaceMonger](http://www.aplusfreeware.com/categories/LFWV/SpaceMonger.html) do have them. An advantage to all tree maps is that we can more easily use vertical space.

## Sunburst

Using [Baobab](http://www.marzocca.net/linux/baobab/index.html) on Linux:

[![](/blog/images/2017/linux_sunburst01.png)](/blog/images/2017/linux_sunburst01.png)

This is a flame graph (which is an adjacency diagram with an inverted icicle layout), using polar coordinates. It is very pretty, and as someone said "it always wows". Sunbursts are the new pie chart.

[![](/blog/images/2017/linux_sunburst04.png)](/blog/images/2017/linux_sunburst04.png)

Deeper slices exaggerate their size, and look visually larger. The smaller-looking slice 2 in this image represents 27.7 Mbytes, whereas the larger-looking slice 1 is 25.6 Mbytes. This visualization is actually showing that slice 1 is smaller than slice 2, although I bet most people would think it was the other way around! The problem is that to understand this correctly requires the comparison of angles, instead of lengths or areas, which has been evaluated as a more difficult perceptive task.

## ncdu

```
--- /home/bgregg/linux-4.9-rc5 -----------------------------------------------
                         /..
  405.1 MiB [##########] /drivers
  139.1 MiB [###       ] /arch
   37.5 MiB [          ] /fs
   36.0 MiB [          ] /include
   35.8 MiB [          ] /Documentation
   32.6 MiB [          ] /sound
   27.8 MiB [          ] /net
   14.7 MiB [          ] /tools
    7.5 MiB [          ] /kernel
    6.0 MiB [          ] /firmware
    3.7 MiB [          ] /lib
    3.4 MiB [          ] /scripts
    3.3 MiB [          ] /mm
    3.2 MiB [          ] /crypto
    2.4 MiB [          ] /security
    1.1 MiB [          ] /block
  968.0 KiB [          ] /samples
[...]

```

This does have ASCII bar charts for line length comparisons, but it's only showing one directory level at a time.

## du

```
$ du -hs * | sort -hr
406M    drivers
140M    arch
38M fs
36M include
36M Documentation
33M sound
28M net
15M tools
7.5M    kernel
6.1M    firmware
3.7M    lib
3.5M    scripts
3.4M    mm
3.2M    crypto
2.4M    security
1.2M    block
968K    samples
[...]

```

Requires reading. Although this is so quick it's my usual starting point.

## Which to use

If you're designing a file system usage tool, which should you use? Ideally, I'd make flame graphs, tree maps, and sunbursts all available as different ways to understand the same data set. For the default view, I'd probably use the flame graph, but I'd want to check with many sample file systems to ensure it really works best with the data it's visualizing.

For more about flame graphs see my ACMQ article [The Flame Graph](http://queue.acm.org/detail.cfm?id=2927301) ( [CACM](http://dl.acm.org/citation.cfm?id=2909476)), and for more about different visualizations see the ACMQ article [A Tour through the Visualization Zoo](http://queue.acm.org/detail.cfm?id=1805128) ( [CACM](http://dl.acm.org/citation.cfm?id=1743546.1743567)), by Jeffrey Heer, Michael Bostock, and Vadim Ogievetsky, especially the section on Hierarchies.

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
