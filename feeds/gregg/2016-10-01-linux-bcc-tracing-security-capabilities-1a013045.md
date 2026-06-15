---
title: Linux bcc Tracing Security Capabilities
url: https://www.brendangregg.com/blog/2016-10-01/linux-bcc-security-capabilities.html
published: "2016-10-01T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-10-01/linux-bcc-security-capabilities.html
---

# Linux bcc Tracing Security Capabilities

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

## Linux bcc Tracing Security Capabilities

01 Oct 2016

Which Linux security capabilities are your applications using? I recently developed a new tool, capable, to print out capability checks live:

```
# capable
TIME      UID    PID    COMM             CAP  NAME                 AUDIT
22:11:23  114    2676   snmpd            12   CAP_NET_ADMIN        1
22:11:23  0      6990   run              24   CAP_SYS_RESOURCE     1
22:11:23  0      7003   chmod            3    CAP_FOWNER           1
22:11:23  0      7003   chmod            4    CAP_FSETID           1
22:11:23  0      7005   chmod            4    CAP_FSETID           1
22:11:23  0      7005   chmod            4    CAP_FSETID           1
22:11:23  0      7006   chown            4    CAP_FSETID           1
22:11:23  0      7006   chown            4    CAP_FSETID           1
22:11:23  0      6990   setuidgid        6    CAP_SETGID           1
22:11:23  0      6990   setuidgid        6    CAP_SETGID           1
22:11:23  0      6990   setuidgid        7    CAP_SETUID           1
22:11:24  0      7013   run              24   CAP_SYS_RESOURCE     1
22:11:24  0      7026   chmod            3    CAP_FOWNER           1
[...]

```

capable uses [bcc](https://github.com/iovisor/bcc), a front-end and a collection of tools that use new Linux enhanced BPF tracing capabilities. capable works by using BPF with kprobes to dynamically trace the kernel cap\_capable() function, and then uses a table to map the capability index to the name seen in the output. Here's the [source code](https://github.com/iovisor/bcc/blob/master/tools/capable.py): it's pretty straightforward.

I wrote it as a colleague, Michael Wardrop, asked what security capabilities our applications were actually using. Given a list, we could use setcap(8) (or other software) to improve the security of applications by only allowing the necessary capabilities.

## Non-audit Checks

The cap\_capable() function has an audit argument, which directs whether the capability check should write an audit message or not, if that's configured. By default, I only print capability checks where this is true, but capable can also trace all checks with the -v option:

```
# capable -h
usage: capable [-h] [-v] [-p PID]

Trace security capability checks

optional arguments:
  -h, --help         show this help message and exit
  -v, --verbose      include non-audit checks
  -p PID, --pid PID  trace this PID only

examples:
    ./capable             # trace capability checks
    ./capable -v          # verbose: include non-audit checks
    ./capable -p 181      # only trace PID 181

```

Here's some non-audit events:

```
# capable -v
TIME      UID    PID    COMM             CAP  NAME                 AUDIT
20:53:45  60004  22061  lsb_release      21   CAP_SYS_ADMIN        0
20:53:45  60004  22061  lsb_release      21   CAP_SYS_ADMIN        0
20:53:45  60004  22061  lsb_release      21   CAP_SYS_ADMIN        0
20:53:45  60004  22061  lsb_release      21   CAP_SYS_ADMIN        0
20:53:45  60004  22061  lsb_release      21   CAP_SYS_ADMIN        0
20:53:45  60004  22061  lsb_release      21   CAP_SYS_ADMIN        0
[...]

```

What are all those?

I'll start by showing the cap\_capable() function prototype, from security/commoncap.c:

```
int cap_capable(const struct cred *cred, struct user_namespace *targ_ns,
                int cap, int audit)

```

Now I can use bcc's trace program to inspect these calls (bear with me), given that cap will be arg3, and audit arg4 (from the above prototype):

```
# trace 'cap_capable "cap: %d, audit: %d", arg3, arg4'
TIME     PID    COMM         FUNC             -
20:56:18 25535  lsb_release  cap_capable      cap: 21, audit: 0
20:56:18 25535  lsb_release  cap_capable      cap: 21, audit: 0
20:56:18 25535  lsb_release  cap_capable      cap: 21, audit: 0
20:56:18 25535  lsb_release  cap_capable      cap: 21, audit: 0
20:56:18 25535  lsb_release  cap_capable      cap: 21, audit: 0
[...]

```

That one-liner is pretty similar to my capable program, except it lacks the "NAME" column with human readable translations.

I'm really doing this so I can add the (newly added) -K and -U options, which print kernel and user-level stack traces. I'll just use -K:

```
# trace -K 'cap_capable "cap: %d, audit: %d", arg3, arg4'
TIME     PID    COMM         FUNC             -
[...]
20:59:58 30607  lsb_release  cap_capable      cap: 21, audit: 0
    Kernel Stack Trace:
        ffffffff813659d1 cap_capable
        ffffffff813684bb security_vm_enough_memory_mm
        ffffffff811deda6 expand_downwards
        ffffffff811def64 expand_stack
        ffffffff81234321 setup_arg_pages
        ffffffff8128c10b load_elf_binary
        ffffffff81234cee search_binary_handler
        ffffffff8128b7ff load_script
        ffffffff81234cee search_binary_handler
        ffffffff8123635a do_execveat_common.isra.35
        ffffffff812367da sys_execve
        ffffffff81003bae do_syscall_64
        ffffffff81861ca5 return_from_SYSCALL_64

20:59:58 30607  lsb_release  cap_capable      cap: 21, audit: 0
    Kernel Stack Trace:
        ffffffff813659d1 cap_capable
        ffffffff813684bb security_vm_enough_memory_mm
        ffffffff811df623 mmap_region
        ffffffff811dff4b do_mmap
        ffffffff811c122a vm_mmap_pgoff
        ffffffff811c1295 vm_mmap
        ffffffff8128bb93 elf_map
        ffffffff8128c271 load_elf_binary
        ffffffff81234cee search_binary_handler
        ffffffff8128b7ff load_script
        ffffffff81234cee search_binary_handler
        ffffffff8123635a do_execveat_common.isra.35
        ffffffff812367da sys_execve
        ffffffff81003bae do_syscall_64
        ffffffff81861ca5 return_from_SYSCALL_64
[...]

```

Awesome. So these are coming from security\_vm\_enough\_memory\_mm(). By reading the source, I see it's used to reserve some memory for root. It's not a hard failure if the capability is missing. And it's not really a security check, hence why it disabled audit.

I should add a -K option to the capable tool, so it can print stack traces too.

## Older Kernels

To use capable, you'll need a 4.4 kernel. To use the -K option, 4.6.

Here's a version using my older [perf-tools](https://github.com/brendangregg/perf-tools) collection, which uses ftrace and should work on much older kernels including the 3.x series:

```
# ./perf-tools/bin/kprobe -s 'p:cap_capable cap=%dx audit=%cx' 'audit != 0'
Tracing kprobe cap_capable. Ctrl-C to end.
             run-4440  [003] d... 6417394.367486: cap_capable: (cap_capable+0x0/0x70) cap=0x18 audit=0x1
             run-4440  [003] d... 6417394.367492:
 => ns_capable_common
 => capable
 => do_prlimit
 => SyS_setrlimit
 => entry_SYSCALL_64_fastpath
           chmod-4453  [006] d... 6417394.399020: cap_capable: (cap_capable+0x0/0x70) cap=0x3 audit=0x1
           chmod-4453  [006] d... 6417394.399027:
 => ns_capable_common
 => ns_capable
 => inode_owner_or_capable
 => inode_change_ok
 => xfs_setattr_nonsize
 => xfs_vn_setattr
 => notify_change
 => chmod_common
 => SyS_fchmodat
 => entry_SYSCALL_64_fastpath
           chmod-4453  [006] d... 6417394.399035: cap_capable: (cap_capable+0x0/0x70) cap=0x4 audit=0x1
           chmod-4453  [006] d... 6417394.399037:
 => ns_capable_common
 => capable_wrt_inode_uidgid
 => inode_change_ok
 => xfs_setattr_nonsize
 => xfs_vn_setattr
 => notify_change
 => chmod_common
 => SyS_fchmodat
 => entry_SYSCALL_64_fastpath
           chmod-4455  [007] d... 6417394.402524: cap_capable: (cap_capable+0x0/0x70) cap=0x4 audit=0x1
           chmod-4455  [007] d... 6417394.402529:
 => ns_capable_common
 => capable_wrt_inode_uidgid
 => inode_change_ok
 => xfs_setattr_nonsize
 => xfs_vn_setattr
 => notify_change
 => chmod_common
 => SyS_fchmodat
 => entry_SYSCALL_64_fastpath
[...]

```

It's a one-liner using my kprobe tool. It's also (currently) a bit harder to use: I need to know which registers those arguments will be in: the example above is for x86\_64 only.

That's all for now. Happy hacking.

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
