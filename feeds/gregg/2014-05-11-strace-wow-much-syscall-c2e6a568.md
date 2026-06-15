---
title: strace Wow Much Syscall
url: https://www.brendangregg.com/blog/2014-05-11/strace-wow-much-syscall.html
published: "2014-05-11T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2014-05-11/strace-wow-much-syscall.html
---

# strace Wow Much Syscall

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

## strace Wow Much Syscall

11 May 2014

[![](/blog/images/2014/doge-strace.jpg)](http://en.wikipedia.org/wiki/Doge_%28meme%29)

I wouldn't dare run strace(1) in production without seriously considering the consequences, and first trying the alternates. While it's widely known (and continually rediscovered) that strace is an amazing tool, it's much less known that it currently is – and always has been – dangerous.

strace is the system call tracer for Linux. It currently uses the arcane ptrace() (process trace) debugging interface, which operates in a violent manner: pausing the target process for each syscall so that the debugger can read state. And doing this twice: when the syscall begins, and when it ends.

This means strace pauses your application twice for each syscall, and context-switches each time between the application and strace. It's like putting traffic metering lights on your application.

> BUGS: A traced process runs slowly.

– *strace(1) man page*

In some cases this doesn't matter. Your application might be completely broken to start with, or you could be running it in a test environment where speed is not a concern. In production, you'll want to be really careful, and think though the consequences (e.g., could this cause the target to fail-over due to exceeding latency thresholds?). Better, look for alternates to try first, like Linux [perf\_events](http://www.brendangregg.com/perf.html).

## Overhead

The performance overhead of strace is relative to the system call rate it is instrumenting. To put something of an upper bound on it, here is a simple worst-case test:

```
$ dd if=/dev/zero of=/dev/null bs=1 count=500k
512000+0 records in
512000+0 records out
512000 bytes (512 kB) copied, 0.103851 s, 4.9 MB/s

```

That finished in 0.103851 seconds.

Now with strace (here I'm tracing a syscall that is never called, accept(), but we pay the overhead anyway):

```
$ strace -eaccept dd if=/dev/zero of=/dev/null bs=1 count=500k
512000+0 records in
512000+0 records out
512000 bytes (512 kB) copied, 45.9599 s, 11.1 kB/s

```

That's **442x slower**. It is worst-case, since dd(1) is performing syscalls as fast as it can.

## Handy strace One-Liners

```
# Slow the target command and print details for each syscall:
strace command

# Slow the target PID and print details for each syscall:
strace -p PID

# Slow the target PID and any newly created child process, printing syscall details:
strace -fp PID

# Slow the target PID and record syscalls, printing a summary:
strace -cp PID

# Slow the target PID and print open() syscalls only:
strace -eopen -p PID

# Slow the target PID and print open() and stat() syscalls only:
strace -eopen,stat -p PID

# Slow the target PID and print connect() and accept() syscalls only:
strace -econnect,accept -p PID

# Slow the target command and see what other programs it launches (slow them too!):
strace -qfeexecve command

# Slow the target PID and print time-since-epoch with (distorted) microsecond resolution:
strace -ttt -p PID

# Slow the target PID and print syscall durations with (distorted) microsecond resolution:
strace -T -p PID

```

## How To Learn strace

There's much you can learn about interpreting strace output. The following two steps should get you started with key system calls.

## 1\. Learn Key Syscalls

You should know what the following 12 key system calls do, which you'll see regularly in strace output. Test your knowledge! How many of these do you know? Mouse-over for answers.

syscallwhat it does`read`

read bytes from a file descriptor (file, socket)

`write`

write bytes from a file descriptor (file, socket)

`open`

open a file (returns a file descriptor)

`close`

close a file descriptor

`fork`

create a new process (current process is forked)

`exec`

execute a new program

`connect`

connect to a network host

`accept`

accept a network connection

`stat`

read file statistics

`ioctl`

set I/O properties, or other miscellaneous functions

`mmap`

map a file to the process memory address space

`brk`

extend the heap pointer

Click here to reveal all. Each syscall has a man page, so if you are at the command line, it should only take a few seconds to jog your memory.

There are variants of these syscalls, so you may see "execve" for "exec", and "pread" as well as "read". There should be man pages for these, too.

## 2\. Simple strace Example

You can practice and learn strace by aiming it at some basic commands: ls(1), sleep(1), ssh(1), etc.

Lets strace the venerable `ls -l` as our first example. We'll list /etc so that there are plenty of files to examine, and the output is mostly what you expect (listing files) and not program startup and initialization.

The output starts by showing program initialization:

```
$ strace ls -l /etc
execve("/bin/ls", ["ls", "-l", "/etc"], [/* 35 vars */]) = 0
brk(0)                                  = 0x9a5000
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f9336828000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=19218, ...}) = 0
mmap(NULL, 19218, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f9336823000
close(3)                                = 0
[...]

```

The execve() variant is used to run /bin/ls, then libraries are pulled in. There'll be multiple groups beginning with an open() of a /lib\* location and ending in close().

As an example of reading a single line: take a look at the open() line, and also the open(2) man page. The man page summarizes this syscall as:

```
       int open(const char *pathname, int flags);
       int open(const char *pathname, int flags, mode_t mode);

```

It also explains these arguments in detail, and the return value. So, our "pathname" was "/etc/ld.so.cache", and the flags were O\_RDONLY\|O\_CLOEXEC. It returned 3, a file descriptor for later use with other syscalls.

Later in the output the real action begins:

```
openat(AT_FDCWD, "/etc", O_RDONLY|O_NONBLOCK|O_DIRECTORY|O_CLOEXEC) = 3
getdents(3, /* 169 entries */, 32768)   = 5280
lstat("/etc/motd", {st_mode=S_IFLNK|0777, st_size=25, ...}) = 0
lgetxattr("/etc/motd", "security.selinux", 0x9b58f0, 255) = -1 ENODATA (No data available)
readlink("/etc/motd", "/var/lib/update-motd/motd", 26) = 25
lstat("/etc/gdbinit.d", {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
lgetxattr("/etc/gdbinit.d", "security.selinux", 0x9b5950, 255) = -1 ENODATA (No data available)
getxattr("/etc/gdbinit.d", "system.posix_acl_access", 0x0, 0) = -1 ENODATA (No data available)
getxattr("/etc/gdbinit.d", "system.posix_acl_default", 0x0, 0) = -1 ENODATA (No data available)
[...]

```

This opens /etc and reads the directory entries (getdents()), then for each file it calls lstat(), a stat() variant, and two extended attribute varients: lgetxattr() and getxattr(). getxattr() is called twice, once for each namespace: see its second argument.

After collecting all this file information, ls(1) begins printing it out, sorted:

```
clock_gettime(CLOCK_REALTIME, {1399855888, 841769117}) = 0
write(1, "drwxr-xr-x  4 root root    4096 "..., 50drwxr-xr-x  4 root root    4096 Dec 10 23:47 acpi
) = 50
stat("/etc/localtime", {st_mode=S_IFREG|0644, st_size=118, ...}) = 0
write(1, "-rw-r--r--  1 root root      16 "..., 53-rw-r--r--  1 root root      16 Dec 10 23:48 adjtime
) = 53
stat("/etc/localtime", {st_mode=S_IFREG|0644, st_size=118, ...}) = 0
write(1, "-rw-r--r--  1 root root    1512 "..., 53-rw-r--r--  1 root root    1512 Jan 12  2010 aliases
) = 53
stat("/etc/localtime", {st_mode=S_IFREG|0644, st_size=118, ...}) = 0
write(1, "-rw-r-----  1 root smmsp  12288 "..., 56-rw-r-----  1 root smmsp  12288 Dec 10 23:47 aliases.db
) = 56
stat("/etc/localtime", {st_mode=S_IFREG|0644, st_size=118, ...}) = 0
write(1, "drwxr-xr-x  2 root root    4096 "..., 58drwxr-xr-x  2 root root    4096 Apr 25 20:47 alternatives
) = 58
[...]

```

The output is a little messed up since `ls -l` and `strace` are writing to the same terminal. The write() syscalls are printing a line of output for each file, and ...

WTF?? Why is ls(1) running stat() on /etc/localtime for *every line of output?*

## Performance Tuning ls(1)

A quick search online (I would have hit up the ls(1) source next) explained the [problem](http://stackoverflow.com/questions/4554271/how-to-avoid-excessive-stat-etc-localtime-calls-in-strftime-on-linux) seen here: my TZ environment variable is not set. Setting it eliminated those stat() /etc/localtime calls.

So I just found a performance win with this simple example. Oh, um, I meant it this way, of course!

Coaxing ls(1) to run long enough to take a performance measurement:

```
$ time ls -l `perl -e 'print "/etc " x 1000'` >/dev/null

real    0m3.562s
user    0m1.004s
sys 0m2.464s

```

And with the fix:

```
$ time TZ=US/Pacific ls -l `perl -e 'print "/etc " x 1000'` >/dev/null

real    0m2.753s
user    0m0.804s
sys 0m1.868s

```

I just **improved performance by 23%** (2.753s vs 3.562s). Not bad. (I wouldn't get too excited about finding and eliminating stat()s, though, as they are a fast syscall, and wins are not usually this high.)

This is a good example of the type of performance issues that strace is good at identifying: problems of the load applied. While we discovered it by fishing around with strace, this would also be found by following a performance methodology: [workload characterization](http://www.slideshare.net/brendangregg/velocity-stoptheguessing2013/32), for system calls.

Since it can distort time measurements so severely (although it does have a "-O overhead" option, and heuristic, to try to compensate), strace is not so good at determining performance issues involving latency. It also is of much less use outside of the syscall interface, for example if the performance issue was deeper in the kernel or application.

## Versus Advanced Tracers

Comparing the current version of strace (e.g., ver 4.8) vs current versions of perf\_events/ktap/SystemTap/LTTng/dtrace4linux (as per 2014).

Pros:

- strace is simple. Syscalls only. POSIX command line interface.
- Automatic meaningful output for each syscall type. Don't have to code this yourself.
- Widely available and mature.

Cons:

- WARNING: Can cause significant and sometimes *massive* performance overhead, in the worst case, slowing the target application by over 100x. This may not only make it unsuitable for production use, but any timing information may also be so distorted as to be misleading.
- Can't trace multiple processes simultaneously (with the exception of followed children).
- Visibility is limited to the system call interface.

There is a possible con: in the past, strace has had bugs which can leave the target process, or its followed children, in the STOP state (e.g., [here](https://bugzilla.redhat.com/show_bug.cgi?id=590172), [here](http://lethargy.org/%7Ejesus/writes/beware-of-strace)). This could cause a serious production outage, as the application is now frozen mid-flight. If you realize this immediately and can fix it (kill strace, then `kill -CONT` the process), then you may avoid a serious outage. However, you may still have caused a burst of application requests with multi-second latency (outliers), depending on how quickly you typed in the kill command.

If you'd like to write a blog post about how awesome strace is and show a case study of its use, then great, but please copy the above WARNING bullet point.

## Did you know?

- strace is banned in France, where it is classified as a cracking tool (it can trace plain-text I/O).
- strace is designed to operate at the system call layer, also called its "test depth".
- In the latest development version (12.1), strace has an easter egg: with "-VV" it begins the trace by printing "DIVE! DIVE!", and playing an aah-WOO-gah alarm on /dev/audio.
- The first strace is on display at the Computer History Museum in San Jose. It was made in 1961 using wire wrapped circuitry and core memory, and could trace up to 12 system calls per second. Within its sheet metal enclosure are four seats: for the surgeon, co-pilot, and their two secretaries. It could stay submerged at syscall depth for up to 9 hours.
- In 1963, Digital Electronics released a cheaper 2-seater version of strace, which was enormously popular.
- The largest strace is the Soviet "Tacti-call" class, which can remain at syscall depth for over 120 days, and has a maximum crew of 160. It is nuclear powered, and has 4 forward and 2 aft signal tubes, each typically armed with type 9 signals.
- On the bottom of San Francisco bay are several thousand unused straces, which were intended for Y2K issues that never arose, and so were scuttled.
- The rarely seen presidential "strace one" is an advanced model that is rumored to operate below syscall depth; however, its specifications are classified.
- In the worst Twitter outage of 2013, engineers were forced to take their strace below crush depth (also called "collapse depth"). They survived the depth and located the root cause, but were then eaten by the fail whale.

## More About strace

Since the following appear to lack overhead warnings, I added a confirmation box on the links:

- [The Magic of Strace](http://chadfowler.com/blog/2014/01/26/the-magic-of-strace/) (2014)
- [Debugging obscure Postgres problems with strace](http://blog.endpoint.com/2013/06/debugging-obscure-postgres-problems.html) (2013)
- [Using strace to Debug Stuck Celery Tasks](http://www.caktusgroup.com/blog/2013/10/30/using-strace-debug-stuck-celery-tasks/) (2013)
- [Diagnosing Magento speed issues with strace](https://www.iweb-hosting.co.uk/blog/diagnosing-magento-speed-issues-with-strace.html) (2012)
- [Strace -- The Sysadmin's Microscope](https://blogs.oracle.com/ksplice/entry/strace_the_sysadmin_s_microscope) (2010)
- [strace: for fun, profit, and debugging.](http://timetobleed.com/hello-world/) (2008)
- [Introducing strace - a System call tracing and Signal reporting tool](http://linuxgazette.net/148/saha.html) (2008)
- [5 simple ways to troubleshoot using Strace](http://www.hokstad.com/5-simple-ways-to-troubleshoot-using-strace) (2008)

These can be useful as they show case studies for real issues, and also provide different perspectives based on each author's area of interest and expertise for applying strace.

If you'd like to learn more about strace internals and ptrace(), I recommend:

- [Write Yourself an Strace in 70 Lines of Code](https://blog.nelhage.com/2010/08/write-yourself-an-strace-in-70-lines-of-code/)

## Beyond strace

[![](/blog/images/2014/linuxperftools-600.png)](http://www.brendangregg.com/linuxperf.html)

There are many articles and posts on strace(1), as well as tcpdump(8) and top(1), but few on harder Linux observability tools and frameworks, like ltrace (library trace), perf\_events, tracepoints, kprobes, uprobes, etc. I joked about this in my [SCaLE12x keynote](http://www.youtube.com/watch?v=6TYC5h4yz1o), showing a [simplified](http://www.slideshare.net/brendangregg/what-linux-can-learn-from-solaris-performance-and-viceversa/101) Linux internals diagram composed of only top, strace, and tcpdump!

strace has limited visibility: (an instrumentable version of) the syscall interface. My Linux performance observability tools diagram on the right shows the scope of strace vs other tools. Now, much can be inferred about kernel activity from the syscall requests, but there are plenty of areas for which you have no direct visibility. Browse the diagram to see the other tools you can use to go beyond strace.

I usually skip strace and go straight to these other tools, like perf ( [perf\_events](http://www.brendangregg.com/perf.html)) or one of the other in-development Linux dynamic tracers. These give me deeper visibility, lower overhead, and customizable output. Although, sometimes the quickest route is just firing up strace, and I will, if the overhead is acceptable.

## strace Future

strace(1) is a great tool, but the current version (using ptrace()) can slow down the target severely. Be aware of the overhead strace causes, and consider other alternates for Linux that use lower-cost buffered tracing. These include perf\_events, sysdig, ktap, and SystemTap. Many of these also allow custom kernel summaries, reducing overheads further.

Linux 3.7 introduced [perf trace](http://www.brendangregg.com/perf.html#More), a buffered strace-like subcommand for perf\_events, which was introduced as an " [alternate to strace](http://kernelnewbies.org/Linux_3.7#head-5f16e5d3f07c2a1601c98f4b81e522c12f6cae7b)". There is also [sysdig](http://www.sysdig.org/), which allows custom filters and output. It's quite possible that in the future strace will become a wrapper to one of these, and all my warnings about the overheads will be out of date (it won't mean that overheads can be ignored, though!).

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
