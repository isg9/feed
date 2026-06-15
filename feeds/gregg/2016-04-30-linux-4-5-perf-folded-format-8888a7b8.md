---
title: Linux 4.5 perf folded format
url: https://www.brendangregg.com/blog/2016-04-30/linux-perf-folded.html
published: "2016-04-30T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-04-30/linux-perf-folded.html
---

# Linux 4.5 perf folded format

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

## Linux 4.5 perf folded format

30 Apr 2016

In Linux 4.5 there was an enhancement to Linux perf\_events (aka "perf") that reduces the CPU cost of flame graph generation. It's "-g folded". If you're automating flame graph generation, you might want to consider using this post Linux 4.5 (at least, until BPF improves this even further post 4.6).

[![](/blog/images/2016/perf_events_flow.png)](http://www.slideshare.net/brendangregg/scale2015-linux-perfprofiling/22)

Flame graphs are a hierarchal visualization for profiled stack traces, and I described them in the recent ACMQ article [The Flame Graph](http://queue.acm.org/detail.cfm?id=2927301). The original implementation (on [github](https://github.com/brendangregg/FlameGraph)) has a Perl program that accepts a "folded" format of stack trace profiles, which has a stack trace on a single line separated by semicolons, and then a value (usually frequency count of the stack trace). There are converters for different profilers, including stackcollapse-perf.pl for Linux perf\_events.

I summarized this flame graph generation sequence in my SCALE13x presentation, shown on the right. Since the "perf" command can output profile data in different ways, can't it just emit folded format directly, eliminating the need for reprocessing in stackcollapse-perf.pl? It's not ideal, since we still must dump each event via perf.data and reprocess in user space (instead of aggregating in kernel space, which BPF will do later on), but it would be a big improvement.

Namhyung Kim has implemented a "folded" output mode for `perf report` for this purpose, and it has been integrated in [Linux 4.5](http://kernelnewbies.org/Linux_4.5#head-91786676f2fa527d183aef79c68445ad19eb85bc). I'll show you how it works.

Here's the usual output of `perf report -n --stdio`:

```
# perf report -n --stdio
[...]
# Children      Self       Samples  Command        Shared Object               Symbol
# ........  ........  ............  .............  ..........................  .............................................
#
    29.39%     0.00%             0  bash           [kernel.vmlinux]            [k] return_from_SYSCALL_64
            |
            ---return_from_SYSCALL_64
               |
                --29.38%--do_syscall_64
                          |
                          |--18.60%--sys_execve
                          |          |
                          |           --18.54%--do_execveat_common.isra.36
                          |                     |
                          |                     |--15.90%--search_binary_handler
                          |                     |          |
                          |                     |           --15.84%--load_elf_binary
                          |                     |                     |
[...]

```

As an aside: if you're looking at this and scratching your head, you should know about another recent change to perf: switching from callee to caller order by default. Here's the old style (specified using `-g callee`):

```
# perf report -n --stdio -g callee
[...]
    15.85%     0.03%            32  bash           [kernel.vmlinux]            [k] load_elf_binary
            |
            ---load_elf_binary
               |
                --15.84%--search_binary_handler
                          do_execveat_common.isra.36
                          sys_execve
                          do_syscall_64
                          return_from_SYSCALL_64
                          __execve

    15.29%     0.03%            32  bash           [kernel.vmlinux]            [k] flush_old_exec
            |
            ---flush_old_exec
               |
                --15.28%--load_elf_binary
                          search_binary_handler
                          do_execveat_common.isra.36
                          sys_execve
                          do_syscall_64
                          return_from_SYSCALL_64

```

I digress. (We did discuss this change on [lkml](https://lkml.org/lkml/2015/10/5/697).)

With the new output mode, folded, we can now do this:

```
# perf report --stdio --no-children -n -g folded,0,caller,count -s comm
[...]
# Overhead       Samples  Command
# ........  ............  .............
#
    60.43%         72372  bash
10282 0x436fd
6378 make_child;__libc_fork;return_from_SYSCALL_64;do_syscall_64;sys_clone;_do_fork;copy_process.part.30;copy_page_range
5944 __execve;return_from_SYSCALL_64;do_syscall_64;sys_execve;do_execveat_common.isra.36;search_binary_handler;load_elf_binary;flush_old_exec;mmput;exit_mmap;unmap_vmas;unma
p_single_vma;unmap_page_range
3207 __execve;return_from_SYSCALL_64;do_syscall_64;sys_execve;do_execveat_common.isra.36;search_binary_handler;load_elf_binary;flush_old_exec;mmput;exit_mmap;unmap_vmas;unma
p_single_vma;unmap_page_range;page_remove_rmap
2746 __execve;return_from_SYSCALL_64;do_syscall_64;sys_execve;do_execveat_common.isra.36;search_binary_handler;load_elf_binary;flush_old_exec;mmput;exit_mmap;tlb_finish_mmu;
tlb_flush_mmu_free;free_pages_and_swap_cache;release_pages
1059 __execve;return_from_SYSCALL_64;do_syscall_64;sys_execve;do_execveat_common.isra.36;copy_strings.isra.26;strnlen_user
[...]
    37.44%         44842  date
2462 0x401f0fc3f30678;_dl_addr
1639 entry_SYSCALL_64_fastpath;0x27e154;do_group_exit;do_exit;mmput;exit_mmap;unmap_vmas;unmap_single_vma;unmap_page_range
1153 do_lookup_x
1032 entry_SYSCALL_64_fastpath;0x27e154;do_group_exit;do_exit;mmput;exit_mmap;unmap_vmas;unmap_single_vma;unmap_page_range;page_remove_rmap
796 _dl_sysdep_start;dl_main;_dl_relocate_object
646 entry_SYSCALL_64_fastpath;0x27e154;do_group_exit;do_exit;mmput;exit_mmap;tlb_finish_mmu;tlb_flush_mmu_free;free_pages_and_swap_cache;release_pages
481 entry_SYSCALL_64_fastpath;0x27e154;do_group_exit;do_exit

```

And with a touch of awk:

```
# perf report --stdio --no-children -n -g folded,0,caller,count -s comm | \
    awk '/^ / { comm = $3 } /^[0-9]/ { print comm ";" $2, $1 }' | more
bash;0x436fd 10282
bash;make_child;__libc_fork;return_from_SYSCALL_64;do_syscall_64;sys_clone;_do_fork;copy_process.part.30;copy_page_range 6378
bash;__execve;return_from_SYSCALL_64;do_syscall_64;sys_execve;do_execveat_common.isra.36;search_binary_handler;load_elf_binary;flush_old_exec;mmput;exit_mmap;unmap_vmas;unma
p_single_vma;unmap_page_range 5944
bash;__execve;return_from_SYSCALL_64;do_syscall_64;sys_execve;do_execveat_common.isra.36;search_binary_handler;load_elf_binary;flush_old_exec;mmput;exit_mmap;unmap_vmas;unma
p_single_vma;unmap_page_range;page_remove_rmap 3207
bash;__execve;return_from_SYSCALL_64;do_syscall_64;sys_execve;do_execveat_common.isra.36;search_binary_handler;load_elf_binary;flush_old_exec;mmput;exit_mmap;tlb_finish_mmu;
tlb_flush_mmu_free;free_pages_and_swap_cache;release_pages 2746
[...]

```

This is the folded format that flamegraph.pl wants, and can be piped directly into flamegraph.pl.

As for CPU cost, here's the old way:

```
# time (perf script | ./FlameGraph/stackcollapse-perf.pl > /dev/null)

real    0m4.659s
user    0m7.644s
sys     0m1.355s

```

Versus the new:

```
# time (perf report --stdio --no-children -n -g folded,0,caller,count -s comm | awk '/^ / { comm = $3 } /^[0-9]/ { print comm ";" $2, $1 }' > /dev/null)

real    0m4.235s
user    0m3.006s
sys     0m1.269s

```

The run time didn't change much, but the CPU cost (user + sys) did, from 9.0 seconds to 4.3, for this example. (The run time is lower than CPU time because the first example ran multi-threaded.) Thanks Namhyung!

Linux 4.6 should improve this a lot more, as BPF included support for stack traces (BPF\_MAP\_TYPE\_STACK\_TRACE), allowing them to be frequency counted in kernel context.

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
