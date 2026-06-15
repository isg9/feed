---
title: Where has my disk space gone? Flame graphs for file systems
url: https://www.brendangregg.com/blog/2017-02-05/file-system-flame-graph.html
published: "2017-02-05T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2017-02-05/file-system-flame-graph.html
---

# Where has my disk space gone? Flame graphs for file systems

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

## Where has my disk space gone? Flame graphs for file systems

05 Feb 2017

My laptop was recently running low on available disk space, and it was a mystery as to why. I have different tools to explore the file system, including running the "find / -ls" command from a terminal, but they can be time consuming to use. I wanted a big picture view of space by directories, subdirectories, and so on.

I've created a simple open source tool to do this, using flame graphs as the final visualization. To demonstrate, here's the space consumed by the Linux 4.9-rc5 source code. Click to zoom, and Ctrl-F to search ( [SVG](/blog/images/2017/files_linux49.svg)):

![](/blog/images/2017/files_linux49.svg)

If you are new to flame graphs, see my [flame graphs](http://www.brendangregg.com/flamegraphs.html) page. In this case, width corresponds to total size. I created them for visualizing stack traces, but since they are a generic hierarchical visualization (technically an [adjacency diagram with an inverted icicle layout](http://queue.acm.org/detail.cfm?id=2927301)), they are suited for the hierarchy of directories as well.

I've also used this to diagnose a similar problem with a friend's laptop, which turned out to be due to a backup application consuming space in a directory completely unknown to them.

The following sections show how to create one yourself. Start by opening a terminal session so you can use the command line.

### Using git

If you have the "git" command, you can fetch the [FlameGraph](https://github.com/brendangregg/FlameGraph) repository and run the commands from it:

```
git clone https://github.com/brendangregg/FlameGraph
cd FlameGraph
./files.pl /Users | ./flamegraph.pl --hash --countname=bytes > out.svg

```

Then open out.svg in a browser. Change "/Users" to be the directory you want to visualize. This could be "/" for everything (provided you don't have removable storage or network file systems mounted, which if you do, it would include them in the report by accident).

### Without git

If you don't have git, you can download the two Perl programs straight from github: [files.pl](https://raw.githubusercontent.com/brendangregg/FlameGraph/master/files.pl) and [flamegraph.pl](https://raw.githubusercontent.com/brendangregg/FlameGraph/master/flamegraph.pl), either using wget or download them via your browser (save as). The steps can then be:

```
chmod 755 files.pl flamegraph.pl
./files.pl /Users | ./flamegraph.pl --hash --countname=bytes > out.svg

```

Again, change "/Users" to be the directory you want to visualize, then open out.svg in a browser.

### Linux source example

For reference, the Linux source example I included above was created using:

```
files.pl linux-4.9-rc5 | flamegraph.pl --hash --countname=bytes \
    --title="Linux source tree by file size" --width=800 > files_linux49.svg

```

You can customize the flame graph using options:

```
$ ./flamegraph.pl -h
Option h is ambiguous (hash, height, help)
USAGE: ./flamegraph.pl [options] infile > outfile.svg

    --title       # change title text
    --width       # width of image (default 1200)
    --height      # height of each frame (default 16)
    --minwidth    # omit smaller functions (default 0.1 pixels)
    --fonttype    # font type (default "Verdana")
    --fontsize    # font size (default 12)
    --countname   # count type label (default "samples")
    --nametype    # name type label (default "Function:")
    --colors      # set color palette. choices are: hot (default), mem, io,
                  # wakeup, chain, java, js, perl, red, green, blue, aqua,
                  # yellow, purple, orange
    --hash        # colors are keyed by function name hash
    --cp          # use consistent palette (palette.map)
    --reverse     # generate stack-reversed flame graph
    --inverted    # icicle graph
    --negate      # switch differential hues (blue<->red)
    --help        # this message

    eg,
    ./flamegraph.pl --title="Flame Graph: malloc()" trace.txt > graph.svg

```

You might need to know about the --minwidth option: rectangles thinner than this (1/10th of a pixel when zoomed out) will be elided, to conserve space in the SVG. But that can mean things are missing when you zoom in. If it's a problem, you can set minwidth to 0.

Update: see my follow-on post [Flame Graphs vs Tree Maps vs Sunburst](http://www.brendangregg.com/blog/2017-02-06/flamegraphs-vs-treemaps-vs-sunburst.html).

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
