---
title: Linux MySQL Slow Query Tracing with bcc/BPF
url: https://www.brendangregg.com/blog/2016-10-04/linux-bcc-mysqld-qslower.html
published: "2016-10-04T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-10-04/linux-bcc-mysqld-qslower.html
---

# Linux MySQL Slow Query Tracing with bcc/BPF

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

## Linux MySQL Slow Query Tracing with bcc/BPF

04 Oct 2016

My mysqld\_qslower tool prints MySQL queries slower than a given threshold, and is run on the MySQL server. By default, it prints queries slower than 1 millisecond:

```
# mysqld_qslower `pgrep -n mysqld`
Tracing MySQL server queries for PID 14371 slower than 1 ms...
TIME(s)        PID          MS QUERY
0.000000       18608   130.751 SELECT * FROM words WHERE word REGEXP '^bre.*n$'
2.921535       18608   130.590 SELECT * FROM words WHERE word REGEXP '^alex.*$'
4.603549       18608    24.164 SELECT COUNT(*) FROM words
9.733847       18608   130.936 SELECT count(*) AS count FROM words WHERE word REGEXP '^bre.*n$'
17.864776      18608   130.298 SELECT * FROM words WHERE word REGEXP '^bre.*n$' ORDER BY word

```

This is a bit like having a custom slow queries log, where the threshold can be picked on the fly.

It is a [bcc](https://github.com/iovisor/bcc) tool that uses the MySQL USDT probes (user statically defined tracing) that were introduced for DTrace. bcc is a front-end and a collection of tools that use new Linux enhanced BPF tracing capabilities.

USDT support in bcc/BPF is new, and involves allowing BPF code to be attached to USDT probes, eg, from mysqld\_qslower:

```
# enable USDT probe from given PID
u = USDT(pid=pid)
u.enable_probe(probe="query__start", fn_name="do_start")
u.enable_probe(probe="query__done", fn_name="do_done")

```

... and then fetching arguments to those USDT probes. This BPF code hashes the timestamp and the query string pointer (from arg1) to the current thread (pid) for later lookup:

```
struct start_t {
    u64 ts;
    char *query;
};
BPF_HASH(start_tmp, u32, struct start_t);

int do_start(struct pt_regs *ctx) {
    u32 pid = bpf_get_current_pid_tgid();
    struct start_t start = {};
    start.ts = bpf_ktime_get_ns();
    bpf_usdt_readarg(1, ctx, &start.query);
    start_tmp.update(&pid, &start);
    return 0;
};

```

The full source to mysqld\_qslower is [here](https://github.com/iovisor/bcc/blob/master/tools/mysqld_qslower.py), and more [example output](https://github.com/iovisor/bcc/blob/master/tools/mysqld_qslower_example.txt).

The tplist tool from bcc can be used to list USDT probes from a pid or binary. Eg:

```
# tplist -l /usr/local/mysql/bin/mysqld
/usr/local/mysql/bin/mysqld mysql:filesort__start
/usr/local/mysql/bin/mysqld mysql:filesort__done
/usr/local/mysql/bin/mysqld mysql:handler__rdlock__start
/usr/local/mysql/bin/mysqld mysql:handler__rdlock__done
/usr/local/mysql/bin/mysqld mysql:handler__unlock__done
/usr/local/mysql/bin/mysqld mysql:handler__unlock__start
/usr/local/mysql/bin/mysqld mysql:handler__wrlock__start
/usr/local/mysql/bin/mysqld mysql:handler__wrlock__done
/usr/local/mysql/bin/mysqld mysql:insert__row__start
/usr/local/mysql/bin/mysqld mysql:insert__row__done
/usr/local/mysql/bin/mysqld mysql:update__row__start
/usr/local/mysql/bin/mysqld mysql:update__row__done
/usr/local/mysql/bin/mysqld mysql:delete__row__start
/usr/local/mysql/bin/mysqld mysql:delete__row__done
/usr/local/mysql/bin/mysqld mysql:net__write__start
/usr/local/mysql/bin/mysqld mysql:net__write__done
/usr/local/mysql/bin/mysqld mysql:net__read__start
/usr/local/mysql/bin/mysqld mysql:net__read__done
/usr/local/mysql/bin/mysqld mysql:query__exec__start
/usr/local/mysql/bin/mysqld mysql:query__exec__done
/usr/local/mysql/bin/mysqld mysql:query__cache__miss
/usr/local/mysql/bin/mysqld mysql:query__cache__hit
/usr/local/mysql/bin/mysqld mysql:connection__start
/usr/local/mysql/bin/mysqld mysql:connection__done
/usr/local/mysql/bin/mysqld mysql:select__start
/usr/local/mysql/bin/mysqld mysql:select__done
/usr/local/mysql/bin/mysqld mysql:query__parse__start
/usr/local/mysql/bin/mysqld mysql:query__parse__done
/usr/local/mysql/bin/mysqld mysql:command__start
/usr/local/mysql/bin/mysqld mysql:command__done
/usr/local/mysql/bin/mysqld mysql:query__start
/usr/local/mysql/bin/mysqld mysql:query__done
/usr/local/mysql/bin/mysqld mysql:update__start
/usr/local/mysql/bin/mysqld mysql:update__done
/usr/local/mysql/bin/mysqld mysql:multi__update__start
/usr/local/mysql/bin/mysqld mysql:multi__update__done
/usr/local/mysql/bin/mysqld mysql:delete__start
/usr/local/mysql/bin/mysqld mysql:delete__done
/usr/local/mysql/bin/mysqld mysql:multi__delete__start
/usr/local/mysql/bin/mysqld mysql:multi__delete__done
/usr/local/mysql/bin/mysqld mysql:insert__start
/usr/local/mysql/bin/mysqld mysql:insert__done
/usr/local/mysql/bin/mysqld mysql:insert__select__start
/usr/local/mysql/bin/mysqld mysql:insert__select__done
/usr/local/mysql/bin/mysqld mysql:keycache__read__block
/usr/local/mysql/bin/mysqld mysql:keycache__read__miss
/usr/local/mysql/bin/mysqld mysql:keycache__read__done
/usr/local/mysql/bin/mysqld mysql:keycache__read__hit
/usr/local/mysql/bin/mysqld mysql:keycache__read__start
/usr/local/mysql/bin/mysqld mysql:keycache__write__block
/usr/local/mysql/bin/mysqld mysql:keycache__write__done
/usr/local/mysql/bin/mysqld mysql:keycache__write__start
/usr/local/mysql/bin/mysqld mysql:index__read__row__start
/usr/local/mysql/bin/mysqld mysql:index__read__row__done
/usr/local/mysql/bin/mysqld mysql:read__row__start
/usr/local/mysql/bin/mysqld mysql:read__row__done

```

You can also use "readelf -n .../mysqld" to double check. Applications that have these probes typically need to be compiled with --with-dtrace or --enable-dtrace on Linux for them to be included in the binary.

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
