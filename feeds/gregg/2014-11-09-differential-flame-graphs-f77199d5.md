---
title: Differential Flame Graphs
url: https://www.brendangregg.com/blog/2014-11-09/differential-flame-graphs.html
published: "2014-11-09T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-11-09/differential-flame-graphs.html
---

# Differential Flame Graphs

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

## Differential Flame Graphs

09 Nov 2014

How quickly can you debug a CPU performance regression? If your environment is complex and changing quickly, this becomes challenging with existing tools. If it takes a week to root cause a regression, the code may have changed multiple times, and now you have new regressions to debug.

Debugging CPU usage is easy in most cases by using [CPU flame graphs](/FlameGraphs/cpuflamegraphs.html). To debug regressions, I would load before and after flame graphs in separate browser tabs, and then blink between them like searching for [Pluto](http://en.wikipedia.org/wiki/Planets_beyond_Neptune#Discovery_of_Pluto). It got the job done, but I wondered about a better way.

Introducing **red/blue differential flame graphs**:

![](/blog/images/2014/zfs-flamegraph-diff.svg)

This is an interactive SVG (direct [link](/blog/images/2014/zfs-flamegraph-diff.svg)). The color shows **red for growth**, and **blue for reductions**.

The size and shape of the flame graph is the same as a CPU flame graph for the second profile (y-axis is stack depth, x-axis is population, and the width of each frame is proportional to its presence in the profile; the top edge is what's actually running on CPU, and everything beneath it is ancestry.)

In this example, a workload saw a CPU increase after a system update. Here's the CPU flame graph ( [SVG](/blog/images/2014/zfs-flamegraph-after.svg)):

![](/blog/images/2014/zfs-flamegraph-after.svg)

Normally, the colors are picked at random to differentiate frames and towers. Red/blue differential flame graphs use color to show the difference between two profiles.

The deflate\_slow() code and children were running more in the second profile, highlighted earlier as red frames. The cause was that ZFS compression was enabled in the system update, which it wasn't previously.

While this makes for a clear example, I didn't really need a differential flame graph for this one. Imagine tracking down subtle regressions, of less than 5%, and where the code is also more complex.

## Red/Blue Differential Flame Graphs

I've had many discussions about this for years, and finally wrote an implementation that I hope makes sense. It works like this:

1. Take stack profile 1.
2. Take stack profile 2.
3. Generate a flame graph using 2. (This sets the width of all frames using profile 2.)
4. Colorize the flame graph using the "2 - 1" delta. If a frame appeared more times in 2, it is red, less times, it is blue. The saturation is relative to the delta.

The intent is for use with before & after profiles, such as for **non-regression testing** or benchmarking code changes. The flame graph is drawn using the "after" profile (such that the frame widths show the current CPU consumption), and then colorized by the delta to show how we got there.

The colors show the difference that function directly contributed (eg, being on-CPU), not its children.

## Generation

I've pushed a simple implementation to github (see [FlameGraph](https://github.com/brendangregg/FlameGraph)), which includes a new program, difffolded.pl. To show how it works, here are the steps using Linux [perf\_events](http://www.brendangregg.com/perf.html) (you can use other profilers).

Collect profile 1:

```
# perf record -F 99 -a -g -- sleep 30
# perf script > out.stacks1

```

Some time later (or after a code change), collect profile 2:

```
# perf record -F 99 -a -g -- sleep 30
# perf script > out.stacks2

```

Now fold these profile files, and generate a differential flame graph:

```
$ git clone --depth 1 http://github.com/brendangregg/FlameGraph
$ cd FlameGraph
$ ./stackcollapse-perf.pl ../out.stacks1 > out.folded1
$ ./stackcollapse-perf.pl ../out.stacks2 > out.folded2
$ ./difffolded.pl out.folded1 out.folded2 | ./flamegraph.pl > diff2.svg

```

difffolded.pl operates on the "folded" style of stack profiles, which are generated by the stackcollapse collection of tools (see the files in [FlameGraph](https://github.com/brendangregg/FlameGraph)). It emits a three column output, with the folded stack trace and two value columns, one for each profile. E.g.:

```
func_a;func_b;func_c 31 33
[...]

```

This would mean the stack composed of "func\_a()->func\_b()->func\_c()" was seen 31 times in profile 1, and in 33 times in profile 2. If flamegraph.pl is handed this three column input, it will automatically generate a red/blue differential flame graph.

## Options

Some options you'll want to know about:

**difffolded.pl -n**: This normalizes the first profile count to match the second. If you don't do this, and take profiles at different times of day, then all the stack counts will naturally differ due to varied load. Everything will look red if the load increased, or blue if load decreased. The -n option balances the first profile, so you get the full red/blue spectrum.

**difffolded.pl -x**: This strips hex addresses. Sometimes profilers can't translate addresses into symbols, and include raw hex addresses. If these addresses differ between profiles, then they'll be shown as differences, when in fact the executed function was the same. Fix with -x.

**flamegraph.pl --negate**: Inverts the red/blue scale. See the next section.

## Negation

While my red/blue differential flame graphs are useful, there is a problem: if code paths vanish completely in the second profile, then there's nothing to color blue. You'll be looking at the current CPU usage, but missing information on how we got there.

One solution is to reverse the order of the profiles and draw a negated flame graph differential. E.g.:

![](/blog/images/2014/zfs-flamegraph-negated.svg)

Now the widths show the first profile, and the colors show what *will* happen. The blue highlighting on the right shows we're about to spend a lot less time in the CPU idle path. (Note that I usually filter out cpu\_idle from the folded files, by including a grep -v cpu\_idle.)

This also highlights the vanishing code problem (or rather, *doesn't* highlight), as since compression wasn't enabled in the "before" profile, there is nothing to color red.

This was generated using:

```
$ ./difffolded.pl out.folded2 out.folded1 | ./flamegraph.pl --negate > diff1.svg

```

Which, along with the earlier diff2.svg, gives us:

- **diff1.svg**: widths show the before profile, colored by what WILL happen
- **diff2.svg**: widths show the after profile, colored by what DID happen

If I were to automate this for non-regression testing, I'd generate and show both side by side.

## Update: Elided

Following on from negation, another solution for code paths that vanish in the second profile is to draw them separately as an "elided flame graph." Netflix's internal FlameCommander tool does this. I'll add screenshots when I get a chance, but for now a quick description of how it is presented:

- The red/blue differential flame graph is shown, and on the same page a text string: "X% elided"
- If you click the elided text, it takes you to the elided flame graph

I debated with colleagues (Martin, Jason) about always including the elided flame graph next to the differential flame graph, scaled appropriately. But I realized that most of the time the elided percentage is small, and in those cases the end user may not be very interested in seeing the elided paths. So just the percentage is shown instead and the end user can decide if that's large enough to merit clicking on for the elided flame graph. Not showing it by default helps reduce the number of rectangles to draw and improves browser performance.

## CPI Flame Graphs

I first used this code for my [CPI flame graphs](/blog/2014-10-31/cpi-flame-graphs.html), where instead of doing a difference between two profiles, I showed the difference between CPU cycles and stall cycles, which highlights what the CPUs were doing.

## Other Differential Flame Graphs

[![](/blog/images/2014/rm-flamegraph-diff.jpg)](http://www.slideshare.net/brendangregg/blazing-performance-with-flame-graphs/167)

There's other ways to do flame graph differentials. [Robert Mustacchi](http://dtrace.org/blogs/rm) experimented with [differentials](http://www.slideshare.net/brendangregg/blazing-performance-with-flame-graphs/167) a while ago, and used an approach similar to a colored code review: only the difference is shown, colored red for added (increased) code paths, and blue for removed (decreased) code paths. An example is on the right. It's a good idea, but in practice I found it a bit weird as the frame widths are now the difference only, which is hard to follow without the bigger picture context: a standard flame graph showing the full profile.

[![](/blog/images/2014/corpaul-flamegraph-diff.png)](https://github.com/corpaul/flamegraphdiff)

Cor-Paul Bezemer has created [flamegraphdiff](http://corpaul.github.io/flamegraphdiff/), which shows the difference flame graph along with two standard (bigger picture) flame graphs at the same time: a flame graph for the A and B profiles (before and after). See the [example](http://corpaul.github.io/flamegraphdiff/demos/dispersy/dispersy_diff.html) on the right, which has the before and after flame graphs at the top and the difference at the bottom. You can mouse-over frames in the differential, which highlights frames in all profiles. This solves the context problem, since you can see the standard flame graph profiles. I found it a bit slow for complex profiles with many frames, as it is more than doubling the number of rectangles to draw.

My red/blue flame graphs, Robert's hue differential, and Cor-Paul's triple-view, all have their strengths. These could be combined: the top two flame graphs in Cor-Paul's view could be my diff1.svg and diff2.svg. Then the bottom flame graph colored using Robert's approach. For consistency, the bottom flame graph could use the same palette range as mine: blue->white->red.

Flame graphs are spreading, and are now used by many companies. I wouldn't be surprised if there were already other implementations of flame graph differentials I didn't know about. (Leave a comment!)

## Conclusion

If you have problems with performance regressions, red/blue differential flame graphs may be the quickest way to find the root cause. These take a normal flame graph and then use colors to show the difference between two profiles: red for greater samples, and blue for fewer. The size and shape of the flame graph shows the current ("after") profile, so that you can easily see where the samples are based on the widths, and then the colors show how we got there: the profile difference.

These differential flame graphs could also be generated by a nightly non-regression test suite, so that performance regressions can be quickly debugged after the fact.

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
