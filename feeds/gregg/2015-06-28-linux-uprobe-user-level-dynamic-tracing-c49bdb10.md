---
title: 'Linux uprobe: User-Level Dynamic Tracing'
url: https://www.brendangregg.com/blog/2015-06-28/linux-ftrace-uprobe.html
published: "2015-06-28T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2015-06-28/linux-ftrace-uprobe.html
---

# Linux uprobe: User-Level Dynamic Tracing

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

## Linux uprobe: User-Level Dynamic Tracing

28 Jun 2015

One of the features I've been looking forward to on newer Linux kernels is **uprobes**: user-level dynamic tracing, which was added to [Linux 3.5](http://kernelnewbies.org/Linux_3.5#head-95fccbb746226f6b9dfa4d1a48801f63e11688de) and improved in [Linux 3.14](http://kernelnewbies.org/Linux_3.14#head-ca18fd90b3cee1181d74251909e0dda6934b5add). It lets you trace user-level functions; for example, the return of the readline() function from all running bash shells, with the returned string:

```
# ./uprobe 'r:bash:readline +0($retval):string'
Tracing uprobe readline (r:readline /bin/bash:0x8db60 +0($retval):string). Ctrl-C to end.
 bash-11886 [003] d... 19601837.001935: readline: (0x41e876 <- 0x48db60) arg1="ls -l"
 bash-11886 [002] d... 19601851.008409: readline: (0x41e876 <- 0x48db60) arg1="echo "hello world""
 bash-11886 [002] d... 19601854.099730: readline: (0x41e876 <- 0x48db60) arg1="df -h"
 bash-11886 [002] d... 19601858.805740: readline: (0x41e876 <- 0x48db60) arg1="cd .."
 bash-11886 [003] d... 19601898.378753: readline: (0x41e876 <- 0x48db60) arg1="foo bar"
^C
Ending tracing...

```

This is snooping on all entered commands from all shells, by instrumenting bash code dynamically. I didn't need to restart any of the bash shells to do this.

Library functions can also be traced. Here's all calls to libc sleep(), with the argument (number of seconds):

```
# ./uprobe 'p:libc:sleep %di'
Tracing uprobe sleep (p:sleep /lib/x86_64-linux-gnu/libc-2.15.so:0xbf130 %di). Ctrl-C to end.
      svscan-2134  [002] d... 19602517.962925: sleep: (0x7f2dba562130) arg1=0x5
      svscan-2134  [002] d... 19602522.963082: sleep: (0x7f2dba562130) arg1=0x5
        cron-923   [002] d... 19602524.187733: sleep: (0x7f3e26d9e130) arg1=0x3c
      svscan-2134  [002] d... 19602527.963267: sleep: (0x7f2dba562130) arg1=0x5
[...]

```

So svcscan is sleeping for 5 seconds, and cron for 60 (0x3c = 60 decimal).

**uprobe** is a tool I wrote for the [perf-tools](https://github.com/brendangregg/perf-tools) collection, to explore *uprobes* via Linux [ftrace](http://lwn.net/Articles/370423/) – the built-in tracer. (uprobe the user-level counterpart of my kprobe tool, which traces kernel functions.) uprobe is an experimental tool, and only works on newer kernels (more on this in a bit). Its real value – and why I'm blogging about it – is to show what capabilities are built into the Linux kernel. You may only ever use uprobes through another tracer (perf\_events, SystemTap, LTTng, ...), and that's fine.

## uprobe One-Liners

Here are the one-liners listed in the help message (-h) and man page, which summarize uprobe features:

```
# Trace readline() calls in all running "bash" executables:
uprobe p:bash:readline

# Trace readline() with explicit executable path:
uprobe p:/bin/bash:readline

# Trace the return of readline() with return value as a string:
uprobe 'r:bash:readline +0($retval):string'

# Trace sleep() calls in all running libc shared libraries:
uprobe p:libc:sleep

# Trace sleep() with register %di (x86):
uprobe 'p:libc:sleep %di'

# Trace this address (use caution: must be instruction aligned):
uprobe p:libc:0xbf130

# Trace gettimeofday() for PID 1182 only:
uprobe -p 1182 p:libc:gettimeofday

# Trace the return of fopen() only when it returns NULL:
uprobe 'r:libc:fopen file=$retval' 'file == 0'

```

Arguments to entry functions can be read from their register locations. If that's too much of a nuisance, consider Linux [perf\_events](http://www.brendangregg.com/perf.html) with debuginfo (or other tracers, like SystemTap), where the variable names can be used directly.

## Warnings

I was hoping to use uprobes on the Linux 3.13 kernels I'm now debugging, but have **frequently hit issues where the target process either crashes or enters an endless spin loop**. I'm not too surprised, since uprobes is relatively new kernel code. If I'm really unlucky, the server needs a reboot, as I've wedged multiple processes. These bugs seem to have been fixed by Linux 4.0 (maybe earlier). For that reason, uprobe won't run on kernels older than 4.0 (without -F to force). Maybe that's pessimistic, and it should be 3.18 or something.

You're unlikely to need this, but I should also warn against using the raw address mode, eg "p:libc: **0xbf130**", unless you know what you are doing. uprobe is simple and does not check instruction alignment. If you use the wrong address, say, mid-way through a multi-byte instruction, the target process will either crash or be in a corrupted state. Other tools, like perf\_events, use debuginfo to check instruction alignment.

Other than that, the typical warning about dynamic tracing applys: you can trace too much, slowing the target process. If that's a problem, you can try a more efficient tracer: eg, Linux perf\_events, and (coming up) eBPF. You can also try uprobe with the "-d duration" option, which uses in-kernel buffering during the specified duration (at the sacrifice of live updates).

If you want to try uprobe, use a test environment first.

## More Examples

See the [uprobe examples](https://github.com/brendangregg/perf-tools/blob/master/examples/uprobe_example.txt) file I included in perf-tools, and the [man page](https://github.com/brendangregg/perf-tools/blob/master/man/man8/uprobe.8).

## uprobe Internals

The inspiration for writing uprobe came from reading the documentation in the Linux kernel, [uprobetracer.txt](https://www.kernel.org/doc/Documentation/trace/uprobetracer.txt):

```
Following example shows how to dump the instruction pointer and %ax register
at the probed text address. Probe zfree function in /bin/zsh:

    # cd /sys/kernel/debug/tracing/
    # cat /proc/`pgrep zsh`/maps | grep /bin/zsh | grep r-xp
    00400000-0048a000 r-xp 00000000 08:03 130904 /bin/zsh
    # objdump -T /bin/zsh | grep -w zfree
    0000000000446420 g    DF .text  0000000000000012  Base        zfree

  0x46420 is the offset of zfree in object /bin/zsh that is loaded at
  0x00400000. Hence the command to uprobe would be:

    # echo 'p:zfree_entry /bin/zsh:0x46420 %ip %ax' > uprobe_events

  And the same for the uretprobe would be:

    # echo 'r:zfree_exit /bin/zsh:0x46420 %ip %ax' >> uprobe_events

```

(This example uses zsh instead of bash, which might be safer when experimenting with instrumentation than doing so with the default login shell!)

These steps are a lot of mucking around (and there are additional steps: probes need clean-up, too), and beg to be automated. Fortunately, it already has been: apart from my uprobe tool, you can use these from the Linux perf command (from [perf\_events](http://www.brendangregg.com/perf.html)).

## Using perf

For comparison, here's tracing using the standard Linux tracer, perf:

```
# perf probe -x /bin/bash 'readline%return +0($retval):string'
Added new event:
  probe_bash:readline  (on readline%return in /bin/bash with +0($retval):string)

You can now use it in all perf tools, such as:

    perf record -e probe_bash:readline -aR sleep 1

# perf record -e probe_bash:readline -a
^C[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.259 MB perf.data (2 samples) ]

# perf script
 bash 26239 [003] 283194.152199: probe_bash:readline: (48db60 <- 41e876) arg1="ls -l"
 bash 26239 [003] 283195.016155: probe_bash:readline: (48db60 <- 41e876) arg1="date"
# perf probe --del probe_bash:readline
Removed event: probe_bash:readline

```

The perf command provides better error checking, lower overhead, and other features; but is more cumbersome to use. I wrote uprobe to test out uprobes with ftrace, and to have a more rapid tool to explore this type of tracing. There are also some things it can do that perf can't, but more on that in another post...

UPDATE: In my next post, I covered [hacking USDT with ftrace](/blog/2015-07-03/hacking-linux-usdt-ftrace.html) for user static probes (eg, those in Node.js, MySQL, etc), and included more examples of uprobe.

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
