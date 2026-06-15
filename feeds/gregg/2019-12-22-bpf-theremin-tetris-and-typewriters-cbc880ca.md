---
title: BPF Theremin, Tetris, and Typewriters
url: https://www.brendangregg.com/blog/2019-12-22/bpf-theremin.html
published: "2019-12-22T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2019-12-22/bpf-theremin.html
---

# BPF Theremin, Tetris, and Typewriters

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

## BPF Theremin, Tetris, and Typewriters

22 Dec 2019

For my AWS re:Invent talk on BPF Performance Analysis at Netflix, I began with a demo of "BPF superpowers" (aka eBPF). The video is on [youtube](https://youtu.be/16slh29iN1g) and the demo starts at [0:50](https://youtu.be/16slh29iN1g?t=50) (the slides are [here](/Slides/reInvent2019_BPF_Performance_Analysis) or as a [PDF](/Slides/reInvent2019_BPF_Performance_Analysis.pdf)):

I'm demonstrating a tool I developed to turn my laptop's wifi signal strength into audio (someone described this as a BPF [theremin](https://en.wikipedia.org/wiki/Theremin).) I first developed it as this [bpftrace](https://github.com/iovisor/bpftrace) one-liner:

```
bpftrace -e 'k:__iwl_dbg /str(arg4) == "Rssi %d, TSF %llu\n"/ { printf("strength: %d\n", arg5);  }'
Attaching 1 probe...
strength: -46
strength: -46
strength: -45
strength: -45
[...]

```

This only works for the iwl driver. In the video I explained how I arrived at tracing \_\_iwl\_dbg() in this way, and how you can follow a similar approach for tracing unfamiliar code.

Since I wanted to emit audio, I then switched to [BCC](https://github.com/iovisor/bcc) and rewrote it in Python so I could use an audio library. This is not my best code, since I hacked it in a hurry, but here it is:

```
#!/usr/bin/python
#
# iwlstrength.py    iwl wifi stregth as audio
#           For Linux, uses BCC, eBPF. Embedded C.
#
# Tone generation from: https://gist.github.com/nekozing/5774628
#
# 29-Apr-2019   Brendan Gregg   Created this.

from __future__ import print_function
from bcc import BPF
import pygame, numpy, pygame.sndarray

# Sound setup
sampleRate = 44100
pygame.mixer.pre_init(sampleRate, -16, 1)
pygame.init()
note = {}
for f in range(-100, 20):
    # pregenerate lookup table of frequency to note
    # this avoids the CPU cost at runtime
    # please, we need a better tone generation library (if it exists I didn't find it)
    freq = 1700 + (f * 15)
    peakvol = 4096
    note[f] = numpy.array([peakvol * numpy.sin(2.0 * numpy.pi * freq * x / sampleRate) for x in range(0, sampleRate / 5)]).astype(numpy.int16)
sound = pygame.sndarray.make_sound(note[0])

# BPF
b = BPF(text="""
#include

int kprobe____iwl_dbg(struct pt_regs *ctx, int a, int b, int c, int d, const char *fmt, int val0) {
    bpf_trace_printk("%d %s\\n", val0, fmt);
    return 0;
}
""")

last=0
while 1:
    try:
        (task, pid, cpu, flags, ts, msg) = b.trace_fields()
        (val0, fmt) = msg.split(' ', 1)
        if (fmt == "Rssi %d, TSF %llu"):
            signal = int(val0);
            for c in range(0, (100 + signal) / 2):
                print("*", end="")
            print("")
            if (last != signal):
                sound.stop()
                sound = pygame.sndarray.make_sound(note[signal])
                sound.play(-1)
            last = signal
    except KeyboardInterrupt:
        exit()
    except ValueError:
        next

sound.stop()

```

Remember how this was a one-liner in bpftrace?

If you wish to develop your own BPF observability tools, start with bpftrace and only use BCC when needed. My [BPF Performance Tools](/bpf-performance-tools-book.html) book has plenty of examples. This is the culmination of five years of work: the BPF kernel runtime, C support, LLVM and Clang support, the BCC front-end, and finally the bpftrace language. Starting with other interfaces is like writing your first Java program in [JVM bytecode](https://medium.com/@davethomas_9528/writing-hello-world-in-java-byte-code-34f75428e0ad). You can...but if you're looking for an educational exercise, I'd recommend using BPF tools to find performance wins.

bpftrace became even more powerful on Linux 5.3, which raised the BPF instruction limit from four thousand instructions to effectively unlimited (one million instructions.) Masanori Misono has already ported [Tetris to bpftrace](https://github.com/mmisono/bpftrace-tetris) (originally by Tomoki Sekiyama):

![](/blog/images/2019/bpftrace-tetris.png)

It reads keystrokes by tracing the pty\_write() kernel function.

On the topic of tracing keystrokes, David S. Miller (networking maintainer) recently asked if I still used my "noisy typewriter," which I had turned on a little too loud during a [LISA 2016](https://www.youtube.com/watch?v=GsMs3n8CB6g) demo. Since I switched laptops it no longer works, so I just wrote it using BPF: [bpf-typewriter](https://github.com/brendangregg/bpf-typewriter). The source is:

```
#!/usr/local/bin/bpftrace --unsafe
#include

kprobe:kbd_event
/arg1 == EV_KEY && arg3 == 1/
{
    if (arg2 == KEY_ENTER) {
        system("aplay -q clink.au >/dev/null 2>&1 &");
    } else {
        system("aplay -q click.au >/dev/null 2>&1 &");
    }
}

```

You can check it out on [github](https://github.com/brendangregg/bpf-typewriter). If desired, it could be rewritten in BCC to avoid calling aplay(1).

I hope you enjoy these tools, and I look forward to seeing what other people create, either practical or for fun!

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
