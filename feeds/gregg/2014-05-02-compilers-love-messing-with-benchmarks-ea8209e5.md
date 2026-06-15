---
title: Compilers Love Messing With Benchmarks
url: https://www.brendangregg.com/blog/2014-05-02/compilers-love-messing-with-benchmarks.html
published: "2014-05-02T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-05-02/compilers-love-messing-with-benchmarks.html
---

# Compilers Love Messing With Benchmarks

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

## Compilers Love Messing With Benchmarks

02 May 2014

If you want accurate and trustworthy benchmarks, you need to perform [active benchmarking](http://www.brendangregg.com/activebenchmarking.html), as everything, **including compilers**, can mess with your benchmark. In this post I'll describe a common example.

There seems to be a trend, especially in cloud computing, for evaluating server instances using kitchen-sink benchmarks. These drag together old and new benchmark tools and automates their execution. UnixBench is a common inclusion – the original kitchen-sink micro-benchmark from 1984 – which is nowadays [trivial to run](http://blog.serverbear.com/tutorials/unixbench-ubuntu/):

```
wget http://byte-unixbench.googlecode.com/files/UnixBench5.1.3.tgz
tar xvf UnixBench5.1.3.tar.gz
cd UnixBench5.1.3
./Run

```

The "./Run" command will compile and launch UnixBench using its default Makefile, which includes:

```
## Very generic
#OPTON = -O

## For Linux 486/Pentium, GCC 2.7.x and 2.8.x
#OPTON = -O2 -fomit-frame-pointer -fforce-addr -fforce-mem -ffast-math \
#   -m486 -malign-loops=2 -malign-jumps=2 -malign-functions=2

## For Linux, GCC previous to 2.7.0
#OPTON = -O2 -fomit-frame-pointer -fforce-addr -fforce-mem -ffast-math -m486

#OPTON = -O2 -fomit-frame-pointer -fforce-addr -fforce-mem -ffast-math \
#   -m386 -malign-loops=1 -malign-jumps=1 -malign-functions=1

## For Solaris 2, or general-purpose GCC 2.7.x
OPTON = -O2 -fomit-frame-pointer -fforce-addr -ffast-math -Wall

## For Digital Unix v4.x, with DEC cc v5.x
#OPTON = -O4
#CFLAGS = -DTIME -std1 -verbose -w0

```

I didn't edit this. That's what you get in version 5.1.3, the version everyone is using. Take a closer look.

## Makefiles

The UnixBench Makefile sets OPTON: a list of compiler options. (I'm happier if I assume that OPTON is short for Option-ON, and isn't an egregious typo.)

The default OPTON is for Solaris 2 (the last release of which was 1997). How many users don't realize this and let UnixBench use these defaults, vs, how many choose "Very generic" or one of the Linux options?

The problem is that people may end up using different compiler options, which can massively change the benchmark results. If you try and compare your UnixBench results with others found online, without realizing that these were compiled differently, you may be seriously misled. That is, if this even makes a difference...

## Optimizations

Consider the following two Linux servers, and the first UnixBench result, Dhrystone 2 (more is better):

server A: 21775273

server B: 35773059

Server B is 64% faster, I bet you'd rather buy that! ... Except that these are the *same server*: B is compiled with the default OPTON (Solaris 2) and A is "Very generic" (a lower optimization level).

Fortunately this Dhrystone 2 result was the largest difference from the 24 benchmarks from UnixBench, and the difference to the final UnixBench "Index Score" summary was less than 5% (at least, in my case, this could be higher for you). Unfortunately, Dhrystone 2 is listed first in the UnixBench report, so it can be eye-catching.

What's happened is that UnixBench has benchmarked GCC (specifically, compiler options), as well as the target of the study. Mistakes like this are very common.

Perhaps, for UnixBench, it's not so bad. Perhaps everyone uses "./Run" and compiles with the Solaris 2 OPTON, so that even the Dhrystone 2 results can be compared accurately.

## Versions

Now consider the following two servers, this time compiled with the same default OPTON (Solaris 2):

server A: 24932516

server B: 35773059

So I bet you'd like to buy... ok, yes, they are the same server again. The difference this time is the GCC version: Server A is gcc 4.1.2, and B is gcc 4.6.3. I suppose I owe a GCC developer a beer or some other gesture of thanks, at least for when they aren't [obfuscating my assembly](http://www.brendangregg.com/blog/2014-04-27/let-me-obfuscate-that-for-you.html).

Wouldn't everyone generally be running the same GCC version? Absolutely not. Right now I happen to be testing the performance of different server instances on AWS EC2, and a quick check shows I have:

```
gcc (GCC) 4.8.2 20131212 (Red Hat 4.8.2-7)
gcc (Ubuntu/Linaro 4.6.3-1ubuntu5) 4.6.3
gcc (GCC) 4.1.2 20080704 (Red Hat 4.1.2-54)

```

So if I didn't pay attention to the compiler, any results comparing systems would be bogus and misleading. Short of running the exact same binary, you need the compilers to match, or, you need to treat the compiler as part of the tested target.

> To compare system benchmarks, you need the same compiler options and compiler version.

A workaround for this is to ship and test binaries (although this approach may miss out on new processor instructions). For example, as was done with these [dhrystone results](http://www.roylongbottom.org.uk/dhrystone%20results.htm). Interestingly, they also share non-optimized Dhrystone results, to better understand compiler differences.

## Index Score

The final report from UnixBench has a "System Benchmarks Index Score": a single number to summarize the system's performance. The USAGE file describes it as:

```
The BYTE Index
==============

The purpose of this test is to provide a basic indicator of the performance
of a Unix-like system; hence, multiple tests are used to test various
aspects of the system's performance.  These test results are then compared
to the scores from a baseline system to produce an index value, which is
generally easier to handle than the raw sores.  The entire set of index
values is then combined to make an overall index for the system.

Since 1995, the baseline system has been "George", a SPARCstation 20-61
with 128 MB RAM, a SPARC Storage Array, and Solaris 2.3, whose ratings
were set at 10.0.

```

Yes, it's old, but that's not a problem (e.g., [VAX MIPS](http://dictionary.reference.com/browse/vax+mips)). A problem, again, is that the compiler options and version are part of the score, and yet, this score is used to compare servers – not compilers.

For example (and there are many I could pick from), [ServerBear](http://serverbear.com) have a [VPS UnixBench Leaderboard](http://serverbear.com/benchmarks/vps) for these Index Scores, stating:

> If you're in the market for a VPS with a high system performance then the UnixBench score is one of the metrics you should be paying attention to.

They are nice enough to share their [source code](https://github.com/Crowd9/Benchmark) that gathers these UnixBench scores (much more trustworthy than vendors who don't!). It looks like UnixBench is executed via "./Run", which compiles using whatever gcc version the system has already or is added from its default package repository. Which for the systems I'm testing right now, means three different versions of gcc.

## Improving UnixBench

So while this is a neat example of compiler differences, how do we actually fix UnixBench? Well, I'm not sure it's really a problem with UnixBench. The project [website](https://code.google.com/p/byte-unixbench/) does warn you about this:

> The results will depend not only on your hardware, but on your operating system, libraries, and even compiler.

As does the USAGE file under the "Interpreting the Results" section, which even suggests:

> So you may want to make sure that all your test systems are running the same version of the OS; or at least publish the OS and compuiler versions with your results.

Good advice, but I frequently see UnixBench results used without this information.

I think the biggest problem is how people are using UnixBench and other benchmarks: fire-and-forget, hoping for a quick result. And doing so without reading the documentation, or performing [active benchmarking](http://www.brendangregg.com/activebenchmarking.html) to confirm what was measured.

UnixBench could help by making `-O0` the default OPTON, reducing compiler differences. This does not – as is commonly believed – turn off all optimizations (zero optimizations). It uses fewer of them. The optimization I described in my [let me obfuscate that for you](http://www.brendangregg.com/blog/2014-04-27/let-me-obfuscate-that-for-you.html) post was still enabled at -O0, for example. Even so, -O0 may make the differences negligible.

## Avoiding Trouble

If you want to compare different servers using benchmarks that you compile, you need the compilers to match, or you need to take that into consideration. This should be something you unearth by following an [active benchmarking](http://www.brendangregg.com/activebenchmarking.html) approach, where you study and understand what the benchmark really does.

As for UnixBench: if it does encourage you to switch to a new system because it has a higher UnixBench Index Score, you might want to try upgrading your compiler first!

(I never discussed problems with the UnixBench micro-benchmarks themselves – maybe next time.)

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
