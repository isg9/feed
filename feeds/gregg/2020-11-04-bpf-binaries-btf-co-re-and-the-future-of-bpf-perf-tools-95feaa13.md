---
title: 'BPF binaries: BTF, CO-RE, and the future of BPF perf tools'
url: https://www.brendangregg.com/blog/2020-11-04/bpf-co-re-btf-libbpf.html
published: "2020-11-04T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2020-11-04/bpf-co-re-btf-libbpf.html
---

# BPF binaries: BTF, CO-RE, and the future of BPF perf tools

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

## BPF binaries: BTF, CO-RE, and the future of BPF perf tools

04 Nov 2020

Two new technologies, BTF and CO-RE, are paving the way for BPF to become a billion-dollar industry. Right now there are many BPF (eBPF) startups building networking, security, and performance products (and more in stealth). However, requiring customers to install the LLVM, Clang, and kernel header dependencies – which can consume over 100 Mbytes of storage – is an adoption drag. BTF and CO-RE eliminate these dependencies at runtime, not only making BPF more practical for embedded Linux environments, but for adoption everywhere.

These technologies are:

- **BTF**: BPF Type Format, which provides struct information to avoid needing Clang and kernel headers.
- **CO-RE**: BPF Compile-Once Run-Everywhere, which allows compiled BPF bytecode to be relocatable, avoiding the need for recompilation by LLVM.

Clang and LLVM are still required for compilation, but the result is a lightweight ELF binary that includes the precompiled BPF bytecode and can be run everywhere. The BCC project has a collection of these, called [libbpf tools](https://github.com/iovisor/bcc/tree/master/libbpf-tools). As an example, I ported over my opensnoop(8) tool:

```
# ./opensnoop
PID    COMM              FD ERR PATH
27974  opensnoop         28   0 /etc/localtime
1482   redis-server       7   0 /proc/1482/stat
1657   atlas-system-ag    3   0 /proc/stat
[…]

```

This opensnoop(8) is an ELF binary that doesn't use libLLVM or libclang:

```
# file opensnoop
opensnoop: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/l, for GNU/Linux 3.2.0, BuildID[sha1]=b4b5320c39e5ad2313e8a371baf5e8241bb4e4ed, with debug_info, not stripped

# ldd opensnoop
    linux-vdso.so.1 (0x00007ffddf3f1000)
    libelf.so.1 => /usr/lib/x86_64-linux-gnu/libelf.so.1 (0x00007f9fb7836000)
    libz.so.1 => /lib/x86_64-linux-gnu/libz.so.1 (0x00007f9fb7619000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f9fb7228000)
    /lib64/ld-linux-x86-64.so.2 (0x00007f9fb7c76000)

# ls -lh opensnoop opensnoop.stripped
-rwxr-xr-x 1 root root 645K Feb 28 23:18 opensnoop
-rwxr-xr-x 1 root root 151K Feb 28 23:33 opensnoop.stripped

```

... and stripped is only 151 Kbytes.

Now imagine a BPF product: instead of requiring customers install various heavyweight (and brittle) dependencies, a BPF agent may now be a single tiny binary that works on any kernel that has BTF.

## How this works

It's not just a matter of saving the BPF bytecode in ELF and then sending it to any other kernel. Many BPF programs walk kernel structs that can change from one kernel version to another. Your BPF bytecode may still execute on different kernels, but it may be reading the wrong struct offsets and printing garbage output! opensnoop(8) doesn't walk kernel structs since it instruments stable tracepoints and their arguments, but many other tools do.

This is an issue of *relocation*, and both BTF and CO-RE solve this for BPF binaries. BTF provides type information so that struct offsets and other details can be queried as needed, and CO-RE records which parts of a BPF program need to be rewritten, and how. CO-RE developer Andrii Nakryiko has written long posts explaining this in more depth: [BPF Portability and CO-RE](https://facebookmicrosites.github.io/bpf/blog/2020/02/19/bpf-portability-and-co-re.html) and [BTF Type Information](https://facebookmicrosites.github.io/bpf/blog/2018/11/14/btf-enhancement.html).

## CONFIG\_DEBUG\_INFO\_BTF=y

These new BPF binaries are only possible if this kernel config option is set. It adds about 1.5 Mbytes to the kernel image (this is tiny in comparison to DWARF debuginfo, which can be hundreds of Mbytes). Ubuntu 20.10 has already made this config option the default, and all other distros should follow. Note to distro maintainers: it requires pahole >= 1.16.

## The future of BPF performance tools, BCC Python, and bpftrace

For BPF performance tools, you should start with running [BCC](https://github.com/iovisor/bcc) and [bpftrace](https://github.com/iovisor/bpftrace) tools, and then coding in bpftrace. The BCC tools should eventually be switched from Python to libbpf C under the hood, but will work the same. **Coding performance tools in BCC Python is now considered deprecated** as we move to libbpf C with BTF and CO-RE (although we still have library work to do, such as for USDT support, so the Python versions will be needed for a while). Note that there are other use cases of BCC that may continue to use the Python interface; BPF co-maintainer Alexei Starovoitov and myself briefly discussed this on [iovisor-dev](https://lists.iovisor.org/g/iovisor-dev/topic/future_of_bcc_python_tools/77827559?p=,,,20,0,0,0::recentpostdate%2Fsticky,,,20,2,0,77827559).

My [BPF Performance Tools](/bpf-performance-tools-book.html) book focused on running BCC tools and coding in bpftrace, and that doesn't change. However, **Appendix C's Python programming examples are now considered deprecated.** Apologies for the inconvenience. Fortunately it's only 15 pages of appendix material out of the 880-page book.

What about bpftrace? It does support BTF, and in the future we're looking at reducing its installation footprint as well (it can currently get to [29 Mbytes](https://github.com/iovisor/bpftrace/issues/342), and we think it can go a lot smaller). Given an average libbpf program size of 229 Kbytes (based on the current libbpf tools, stripped), and an average bpftrace program size of 1 Kbyte (my book tools), a large collection of bpftrace tools plus the bpftrace binary may become a smaller installation footprint than the equivalent in libbpf. Plus the bpftrace versions can be modified on the fly. libbpf is better suited for more complex and mature tools that needs custom arguments and libraries.

As screenshots, the future of BPF performance tools is this:

```
# ls /usr/share/bcc/tools /usr/sbin/*.bt
argdist       drsnoop         mdflush         pythongc     tclobjnew
bashreadline  execsnoop       memleak         pythonstat   tclstat
[...]
/usr/sbin/bashreadline.bt    /usr/sbin/mdflush.bt    /usr/sbin/tcpaccept.bt
/usr/sbin/biolatency.bt      /usr/sbin/naptime.bt    /usr/sbin/tcpconnect.bt
[...]

```

... and this:

```
# bpftrace -e 'BEGIN { printf("Hello, World!\n"); }'
Attaching 1 probe...
Hello, World!
^C

```

... and **not** this:

```
#!/usr/bin/python

from bcc import BPF
from bcc.utils import printb

prog = """
int hello(void *ctx) {
    bpf_trace_printk("Hello, World!\\n");
    return 0;
}
"""
[...]

```

Thanks to Yonghong Song (Facebook) for leading development of BTF, Andrii Nakryiko (Facebook) for leading development of CO-RE, and everyone else involved in making this happen.

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
