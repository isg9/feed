---
title: opensnoop For Linux
url: https://www.brendangregg.com/blog/2014-07-25/opensnoop-for-linux.html
published: "2014-07-25T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-07-25/opensnoop-for-linux.html
---

# opensnoop For Linux

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

## opensnoop For Linux

25 Jul 2014

Here's another tool I didn't think would be possible: what files are being opened on my Linux system?

```
# ./opensnoop
Tracing open()s. Ctrl-C to end.
COMM             PID      FD FILE
[...]
sshd             6503     -1 /usr/share/ssh/blacklist.RSA-2048
sshd             6503     -1 /etc/ssh/blacklist.RSA-2048
sshd             6503    0x4 /root/.ssh/authorized_keys
sshd             6503     -1 /var/run/nologin
sshd             6503     -1 /etc/nologin
sshd             6503    0x4 /etc/passwd
sshd             6503    0x4 /etc/shadow
sshd             6503    0x4 /etc/localtime
sshd             6503    0x4 /etc/security/capability.conf
sshd             6503    0x4 /etc/login.defs
[...]

```

This is tracing all processes system-wide, and using a more efficient tracer than [strace](/blog/2014-05-11/strace-wow-much-syscall.html).

How about just files containing "conf", to see what config files are actively opened?

```
# ./opensnoop conf
Tracing open()s for filenames containing "conf". Ctrl-C to end.
COMM             PID      FD FILE
run              19107   0x3 /etc/nsswitch.conf
stat             19111   0x3 /etc/nsswitch.conf
webapp           19119   0x4 /home/webapp/3.7/rel/conf/logging.conf
webapp           19119   0x7 /etc/nsswitch.conf
webapp           19119   0x7 /etc/host.conf
webapp           19119   0x7 /etc/resolv.conf
webapp           19119   0x8 /home/webapp/3.7/rel/conf/application.properties
catalina.sh      19107   0x3 /etc/nsswitch.conf
run              19122   0x3 /etc/nsswitch.conf
[...]

```

Nice! So that's where my webapp config files are...

How about files ending in "log"?

```
# ./opensnoop 'log$'
Tracing open()s for filenames containing "log$". Ctrl-C to end.
COMM             PID      FD FILE
webapp           21144   0x4 /var/tmp/webapp/3.7/stash/webapp.log
webapp           21159   0x4 /var/tmp/webapp/3.7/stash/webapp.log
webapp           21174   0x4 /var/tmp/webapp/3.7/stash/webapp.log
bash             21301   0x4 /var/log/lastlog
[...]

```

Oh, no wonder I couldn't find them, I wasn't looking in /var/tmp!...

And open()s that returned an error (eg, file not found)?

```
# ./opensnoop -x
Tracing open()s. Ctrl-C to end.
COMM             PID      FD FILE
smtp             2690     -1 /usr/lib/tls/x86_64/libnss_db.so.2
smtp             2690     -1 /usr/lib/tls/libnss_db.so.2
smtp             2690     -1 /usr/lib/x86_64/libnss_db.so.2
smtp             2690     -1 /usr/lib/libnss_db.so.2
java             4680     -1 /home/tomcat/conf/Standalone/localhost
java             4680     -1 /home/tomcat/conf/Standalone/localhost
java             4680     -1 /home/tomcat/conf/Standalone/localhost
[...]

```

Catching a process hunt around its library path is normal, like we see here for smtp. But I wonder if that java app was supposed to be finding that file...

## Options

[opensnoop](https://github.com/brendangregg/perf-tools/blob/master/opensnoop) options are summarized by the USAGE message (there's also a [man page](https://github.com/brendangregg/perf-tools/blob/master/man/man8/opensnoop.8) and [examples file](https://github.com/brendangregg/perf-tools/blob/master/examples/opensnoop_example.txt)):

```
# ./opensnoop -h
USAGE: opensnoop [-htx] [-d secs] [-p PID] [-n name] [filename]
                 -d seconds      # trace duration, and use buffers
                 -n name         # process name to match on I/O issue
                 -p PID          # PID to match on I/O issue
                 -t              # include time (seconds)
                 -x              # only show failed opens
                 -h              # this usage message
                 filename        # match filename (partials, REs, ok)
  eg,
       opensnoop                 # watch open()s live (unbuffered)
       opensnoop -d 1            # trace 1 sec (buffered)
       opensnoop -p 181          # trace I/O issued by PID 181 only
       opensnoop conf            # trace filenames containing "conf"
       opensnoop 'log$'          # filenames ending in "log"

```

The -p option employs an in-kernel filter for efficiency.

opensnoop traces events as they happen, which for very frequent open()s can begin to cost measurable overhead. The "-d" mode buffers and prints the buffer at the end, reducing overheads if needed.

## What, Why, and How

This is another ftrace-based hack for my [perf-tools](https://github.com/brendangregg/perf-tools) collection.

There are some great ways to implement opensnoop on Linux using tracers such as perf\_events (the "perf" command) with kernel debuginfo, or using SystemTap or ktap. But my aim is to use this now on a fleet of AWS EC2 cloud instances running Linux 3.2, and *without* kernel debuginfo (which is impractical to install everywhere in this dynamic environment, since it can be 100s of Mbytes).

The without-debuginfo requirement makes this especially tricky. One problem was that I didn't think ftrace (or perf\_events) were able to dereference strings, having only seen examples with hex addresses. It turns out they do have this capability, and in fact, opensnoop can be a few one-liners:

```
# perf probe --add 'do_sys_open filename:string'
[...]
# perf record --no-buffering -e probe:do_sys_open -o - -a | PAGER=cat perf script -i -
 multilog 19075 [001] 5586.323974: probe:do_sys_open: (ffffffff811af8b0) filename_string="."
 multilog 19075 [001] 5586.324013: probe:do_sys_open: (ffffffff811af8b0) filename_string="./main"
 multilog 19075 [001] 5586.324028: probe:do_sys_open: (ffffffff811af8b0) filename_string="lock"
    snmpd  1255 [000] 5586.576142: probe:do_sys_open: (ffffffff811af8b0) filename_string="/proc/net/dev"
    snmpd  1255 [000] 5586.576278: probe:do_sys_open: (ffffffff811af8b0) filename_string="/proc/net/if_inet6"
[...]
# perf probe --del do_sys_open

```

That's an example of dynamic tracing of the kernel function do\_sys\_open(), and examining an argument (filename) as a string. Except this needs kernel debuginfo to recognize the "filename" symbol.

Note that on older systems (including Linux 3.2), instead of --no-buffering it is -D.

Without kernel debuginfo, I can fashion a similar one-liner using CPU registers instead of symbols. I was struggling to get strings to work with this, and started thinking it wouldn't be possible, then asked for help on the [perf-users mailing list](http://thread.gmane.org/gmane.linux.kernel.perf.user/1663/focus=1668). Fortunately, it is possible!

Instead of using registers on the entry of do\_sys\_open(), I ended up tracing the return value of getname() (which do\_sys\_open() calls), and processing that as either a char \* or struct filename \* depending on the kernel version. It's a brittle hack, but I thought this was a bit better than guessing the register for do\_sys\_open() on different platforms.

## Conclusion

Tracing open()s can tell you a lot about running applications: the locations of their config, log, and data files, and file open errors. This is a Linux port of my popular opensnoop tool, which I've used for many years to help with performance analysis and troubleshooting.

This is also another proof of concept for older Linux kernels, like [iosnoop](/blog/2014-07-16/iosnoop-for-linux.html), using the existing ftrace and kprobes tracing frameworks. If you happen to have kernel debuginfo, you can also just use the one-liners I included above. Warnings apply: opensnoop and those one-liners use dynamic tracing on Linux, which has had kernel panic bugs in the past, so know what you are doing, test first, and use at your own risk.

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
