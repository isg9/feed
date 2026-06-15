---
title: Linux eBPF Stack Trace Hack
url: https://www.brendangregg.com/blog/2016-01-18/ebpf-stack-trace-hack.html
published: "2016-01-18T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-01-18/ebpf-stack-trace-hack.html
---

# Linux eBPF Stack Trace Hack

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

## Linux eBPF Stack Trace Hack

18 Jan 2016

Stack trace support by Linux [eBPF](http://www.brendangregg.com/blog/2015-05-15/ebpf-one-small-step.html) will make many new and awesome things possible, however, it didn't make it into the just-released Linux 4.4, which added [other eBPF features](http://kernelnewbies.org/Linux_4.4#head-20c20e63018e8fb916fd26476eda2512e2d96632). Envisaging some time on older kernels that have eBPF but not stack tracing, I've developed a hacky workaround for doing awesome things now.

I'll show my new [bcc tools](https://github.com/iovisor/bcc#tracing) (eBPF front-end) that do this, then explain how it works.

## stackcount: Frequency Counting Kernel Stack Traces

The [stackcount](https://github.com/iovisor/bcc/blob/master/tools/stackcount_example.txt) tool frequency counts kernel stacks for a given function. This is performed in kernel for efficiency using an eBPF map. Only unique stacks and their counts are copied to user-level for printing.

For example, frequency counting kernel stack traces that led to submit\_bio():

```
# ./stackcount submit_bio
Tracing 1 functions for "submit_bio"... Hit Ctrl-C to end.
^C
  submit_bio
  submit_bh
  journal_submit_commit_record.isra.13
  jbd2_journal_commit_transaction
  kjournald2
  kthread
  ret_from_fork
  mb_cache_list
    1

[...truncated...]

  submit_bio
  submit_bh
  jbd2_journal_commit_transaction
  kjournald2
  kthread
  ret_from_fork
  mb_cache_list
    38

  submit_bio
  ext4_writepages
  do_writepages
  __filemap_fdatawrite_range
  filemap_flush
  ext4_alloc_da_blocks
  ext4_rename
  ext4_rename2
  vfs_rename
  sys_rename
  entry_SYSCALL_64_fastpath
    79

```

The order of printed stack traces is from least to most frequent. The most frequent in this example, printed last, was taken 79 times during tracing.

The last stack trace shows syscall handling, ext4\_rename(), and filemap\_flush(): looks like an application issued file rename has caused back end disk I/O due to ext4 block allocation and a filemap\_flush().

This tool should be very useful for exploring and studying kernel behavior, quickly answering how a given function is being called.

## stacksnoop: Printing Kernel Stack Traces

The [stacksnoop](https://github.com/iovisor/bcc/blob/master/tools/stacksnoop_example.txt) tool prints kernel stack traces for each event. For example, for ext4\_sync\_fs():

```
# ./stacksnoop -v ext4_sync_fs
TIME(s)            COMM     PID    CPU STACK
42005557.056332998 sync     22352  1
42005557.056336999 sync     22352  1   ip: ffffffff81280461 ext4_sync_fs
42005557.056339003 sync     22352  1   r0: ffffffff811ed7f9 iterate_supers
42005557.056340002 sync     22352  1   r1: ffffffff8121ba25 sys_sync
42005557.056340002 sync     22352  1   r2: ffffffff81775cb6 entry_SYSCALL_64_fastpath
42005557.056358002 sync     22352  1
42005557.056358002 sync     22352  1   ip: ffffffff81280461 ext4_sync_fs
42005557.056359001 sync     22352  1   r0: ffffffff811ed7f9 iterate_supers
42005557.056359999 sync     22352  1   r1: ffffffff8121ba35 sys_sync
42005557.056359999 sync     22352  1   r2: ffffffff81775cb6 entry_SYSCALL_64_fastpath

```

Since the output is verbose, this isn't suitable for high frequency calls (eg, over 1,000 per second). You can use [funccount](https://github.com/iovisor/bcc/blob/master/tools/funccount_example.txt) from bcc tools to measure the rate of a function call, and if it is high, try stackcount instead.

## How It Works: Crazy Stuff

eBPF is an in-kernel virtual machine that can do all sorts of things, including " [crazy stuff](http://www.slideshare.net/AlexeiStarovoitov/bpf-inkernel-virtual-machine/5)". So I wrote a user-defined stack walker in eBPF, which the kernel can run. Here is the relevant code from stackcount (you are not expected to understand this):

```
#define MAXDEPTH    10

struct key_t {
    u64 ip;
    u64 ret[MAXDEPTH];
};
BPF_HASH(counts, struct key_t);

static u64 get_frame(u64 *bp) {
    if (*bp) {
        // The following stack walker is x86_64 specific
        u64 ret = 0;
        if (bpf_probe_read(&ret, sizeof(ret), (void *)(*bp+8)))
            return 0;
        if (bpf_probe_read(bp, sizeof(*bp), (void *)*bp))
            *bp = 0;
        if (ret < __START_KERNEL_map)
            return 0;
        return ret;
    }
    return 0;
}

int trace_count(struct pt_regs *ctx) {
    FILTER
    struct key_t key = {};
    u64 zero = 0, *val, bp = 0;
    int depth = 0;

    key.ip = ctx->ip;
    bp = ctx->bp;

    // unrolled loop, 10 (MAXDEPTH) frames deep:
    if (!(key.ret[depth++] = get_frame(&bp))) goto out;
    if (!(key.ret[depth++] = get_frame(&bp))) goto out;
    if (!(key.ret[depth++] = get_frame(&bp))) goto out;
    if (!(key.ret[depth++] = get_frame(&bp))) goto out;
    if (!(key.ret[depth++] = get_frame(&bp))) goto out;
    if (!(key.ret[depth++] = get_frame(&bp))) goto out;
    if (!(key.ret[depth++] = get_frame(&bp))) goto out;
    if (!(key.ret[depth++] = get_frame(&bp))) goto out;
    if (!(key.ret[depth++] = get_frame(&bp))) goto out;
    if (!(key.ret[depth++] = get_frame(&bp))) goto out;

out:
    val = counts.lookup_or_init(&key, &zero);
    (*val)++;
    return 0;
}

```

Once eBPF supports this properly, much of the above code will become a single function call.

If you are curious: I've used an unrolled loop to walk each frame (eBPF doesn't do backwards jumps), with a maximum of ten frames in this case. It walks the RBP register (base pointer) and saves the return instruction pointer for each frame into an array. I've had to use explicit bpf\_probe\_read()s to dereference pointers (bcc can automatically do this in some cases). I've also left the unrolled loop in the code (Python could have generated it) to keep it simple, and to help illustrate overhead.

This hack (so far) only works for x86\_64, kernel-mode, and to a limited stack depth. If I (or you) really need more, keep hacking, although bear in mind that this is just a workaround until proper stack walking exists.

## Other Solutions

stackcount implements an important new capability for the core Linux kernel: frequency counting stack traces. Just printing stack traces, like what stacksnoop does, has been possible for a long time: ftrace can do this, which I use in my [kprobe](https://github.com/brendangregg/perf-tools/blob/master/kernel/kprobe_example.txt) tool from [perf-tools](https://github.com/brendangregg/perf-tools). [perf\_events](http://www.brendangregg.com/perf.html) can also dump stack traces and has a reporting mode that will print unique paths and percentages (although it is performed less efficiently in user mode).

SystemTap has long had the capability to frequency count kernel- and user-mode stack traces, also in kernel for efficiency, although it is an add-on and not part of the mainline kernel.

## Future Readers

If you're on Linux 4.5 or later, then eBPF may officially support stack walking. To check, look for something like a BPF\_FUNC\_get\_stack in [bpf\_func\_id](https://github.com/torvalds/linux/blob/master/include/uapi/linux/bpf.h). Or check the latest source code to tools like [stackcount](https://github.com/iovisor/bcc/blob/master/tools/stackcount) – the tool should still exist, but the above stack walker hack may be replaced with a simple call.

Thanks to Brenden Blanco (PLUMgrid) for help with this hack. If you're at [SCaLE14x](https://www.socallinuxexpo.org/scale/14x) you can catch his [IO Visor eBPF talk](https://www.socallinuxexpo.org/scale/14x/presentations/kernel-low-latency-tracing-and-networking) on Saturday, and my [Broken Linux Performance Tools](https://www.socallinuxexpo.org/scale/14x/presentations/broken-linux-performance-tools) talk on Sunday!

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
