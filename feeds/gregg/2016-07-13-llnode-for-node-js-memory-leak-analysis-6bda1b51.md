---
title: llnode for Node.js Memory Leak Analysis
url: https://www.brendangregg.com/blog/2016-07-13/llnode-nodejs-memory-leak-analysis.html
published: "2016-07-13T00:00:00Z"
feed: gregg
guid: https://www.brendangregg.com/blog/2016-07-13/llnode-nodejs-memory-leak-analysis.html
---

# llnode for Node.js Memory Leak Analysis

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

## llnode for Node.js Memory Leak Analysis

13 Jul 2016

The memory of your Node.js process is growing endlessly, what do you do?

I start with a page fault flame graph using Linux [perf](http://www.brendangregg.com/perf.html), which can be generated on the live running process with low overhead. That only solves some issues, though. Other approaches include analyzing heap snapshots with [heapdump](https://strongloop.com/strongblog/how-to-heap-snapshots/), or taking a core dump and analyzing it with mdb findjsobjects on an old Solaris image.

Fortunately, findjsobjects has just been made available for Linux in [llnode](https://github.com/nodejs/llnode), an lldb plugin, which I'll write about here. Thanks to Fedor Indutny for creating llnode, Howard Hellyer for [contributing](https://github.com/indutny/llnode/commit/c517e0ca09b714a487850ca74730d89f4a1c321d) findjsobjects support, and thanks to Dave Pacheco for first developing this [kind of analysis](http://dtrace.org/blogs/dap/2011/10/31/nodejs-v8-postmortem-debugging/).

llnode is not yet well known or documented. To help out, I'll share the commands and screenshots I used for taking a fresh Ubuntu Xenial server with Node v4.4.7 and running some memory object analysis. NOTE: It's 13-Jul-2016, and I'd expect this to be simplified in future versions. See the [llnode](https://github.com/nodejs/llnode) repository for the latest.

## 1\. Installing llnode

Just the commands:

```
sudo bash
apt-get update
apt-get install -y lldb-3.8 lldb-3.8-dev gcc g++ make gdb lldb
apt-get install -y python-pip; pip install six
git clone https://github.com/nodejs/llnode
cd llnode
git clone https://chromium.googlesource.com/external/gyp.git tools/gyp
./gyp_llnode -Dlldb_dir=/usr/lib/llvm-3.8/
make -C out/ -j2
make install-linux

```

(Update: this used to be [https://github.com/indutny/llnode](https://github.com/indutny/llnode) .)

Perhaps is the future this will just be "apt-get install -y llnode" (update: there's now "npm install llnode"). llnode works for MacOS as well, and can be installed via brew.

## 2\. Taking a Core Dump

You can use gcore, noting that it may pause node for some period (short or long). My Node.js PID is 30833.

```
# ulimit -c unlimited
# gcore 30833
[New LWP 30834]
[New LWP 30835]
[New LWP 30836]
[New LWP 30837]
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
syscall () at ../sysdeps/unix/sysv/linux/x86_64/syscall.S:38
38  ../sysdeps/unix/sysv/linux/x86_64/syscall.S: No such file or directory.
warning: target file /proc/30833/cmdline contained unexpected null characters
Saved corefile core.30833
# ls -lh core.30833
-rw-r--r-- 1 root root 855M Jul 12 20:56 core.30833

```

This has hit a warning. It's likely that the gdb (which includes gcore) installed for Xenial isn't matching this kernel version. I thought I'd better note this in case you hit it. If you can't get gcore to work at all, maybe lldb and "process save-core"?

Another problem you might hit is that the core file hangs when lldb tries to read it:

```
# lldb -f /usr/bin/node -c /var/cores/core.node.30833
(lldb) target create "/usr/bin/node" --core "/var/cores/core.node.30833"
[...hang...]

```

This time I suspect it's the known lldb bug [26322 Core load hangs](https://bugs.llvm.org//show_bug.cgi?id=26322), which was [fixed](https://reviews.llvm.org/rL287858) in late 2016, but you might have an older lldb version that misses the fix. This only affects core files taken from live processes, rather than those from a crash.

Since I was hitting that lldb bug, I tried crashing the process instead. WARNING, this will crash the process! To start with I disabled Ubuntu's appreport, and setup traditional core dumps, then sent a SIGBUS (why not):

```
# mkdir /var/cores
# echo "/var/cores/core.%e.%p.%t" > /proc/sys/kernel/core_pattern
# kill -BUS 30833
# ls -lh /var/cores/core.node.30833.1468367170
-rw------- 1 root root 65M Jul 12 23:46 /var/cores/core.node.30833.1468367170

```

For production use I'll go fix gcore (although, we are in a fault-tolerant environment).

## 3\. Memory Ranges File

On lldb 3.8 and earlier, an extra step was needed for findjsobjects to work (this was [fixed lldb 3.9](https://github.com/nodejs/llnode/issues/91)). This is where a memory ranges file must be created and environment variable must be set. There are scripts to generate this file for Mac OS and Linux in llnode. While in the llnode directory:

```
# ./scripts/readelf2segments.py /var/cores/core.node.30833.1468367170 > core.30833.ranges
# export LLNODE_RANGESFILE=core.30833.ranges

```

Again, this step vanishes on [lldb 3.9](https://github.com/nodejs/llnode/issues/91).

## 4\. Starting llnode

Starting up lldb (as root):

```
# lldb -f /usr/bin/node -c /var/cores/core.node.30833.1468367170
(lldb) target create "/usr/bin/node" --core "/var/cores/core.node.30833.1468367170"
Core file '/var/cores/core.node.30833.1468367170' (x86_64) was loaded.

```

Your /usr/bin/node should exist, and match the node version that the core dump is from. If you are debugging multiple node versions, you can keep the binaries under different paths and specify them in `-f` as needed. You can also omit the `-f /usr/bin/node` option.

The help message:

```
(lldb) v8
The following subcommands are supported:

      bt              -- Show a backtrace with node.js JavaScript functions and their args. An optional argument is accepted; if that argument is a number, it specifies
                         the number of frames to display. Otherwise all frames will be dumped.

                         Syntax: v8 bt [number]
      findjsinstances -- List all objects which share the specified map.
                         Accepts the same options as `v8 inspect`
      findjsobjects   -- List all object types and instance counts grouped by map and sorted by instance count.
                         Requires `LLNODE_RANGESFILE` environment variable to be set to a file containing memory ranges for the core file being debugged.
                         There are scripts for generating this file on Linux and Mac in the scripts directory of the llnode repository.
      inspect         -- Print detailed description and contents of the JavaScript value.

                         Possible flags (all optional):

                          * -F, --full-string    - print whole string without adding ellipsis
                          * -m, --print-map      - print object's map address
                          * --string-length num  - print maximum of `num` characters in string

                         Syntax: v8 inspect [flags] expr
      print           -- Print short description of the JavaScript value.

                         Syntax: v8 print expr
      source          -- Source code information

For more help on any particular subcommand, type 'help  '.

```

Run this yourself to see the latest.

## 5\. Stack Trace

The bt command will print the stack backtrace (not interesting in this case, just showing it works):

```
(lldb) v8 bt
 * thread #1: tid = 0, 0x00007f75719a7c19 libc.so.6`syscall + 25 at syscall.S:38, name = 'node', stop reason = signal SIGBUS
  * frame #0: 0x00007f75719a7c19 libc.so.6`syscall + 25 at syscall.S:38
    frame #1: 0x0000000000fc375a node`uv__epoll_wait(epfd=, events=, nevents=, timeout=) + 26 at linux-syscalls.c:321
    frame #2: 0x0000000000fc1838 node`uv__io_poll(loop=0x00000000019aa080, timeout=-1) + 424 at linux-core.c:243
    frame #3: 0x0000000000fb2506 node`uv_run(loop=0x00000000019aa080, mode=UV_RUN_ONCE) + 342 at core.c:351
    frame #4: 0x0000000000dfedf8 node`node::Start(int, char**) + 1080
    frame #5: 0x00007f75718c7830 libc.so.6`__libc_start_main(main=(node`main), argc=2, argv=0x00007ffc531bb6b8, init=, fini=, rtld_fini=, stack_end=0x00007ffc531bb6a8) + 240 at libc-start.c:291
    frame #6: 0x0000000000722f1d node`_start + 41

```

Remember to run `v8 bt` and not just `bt`, which I do out of habit.

## 6\. llnode Object Analysis

Finding all JavaScript objects (reminds me of Java's jmap -histo):

```
(lldb) v8 findjsobjects
 Instances  Total Size Name
 ---------- ---------- ----
          1         24 (anonymous)
          1         24 JSON
          1         24 process
          1         32 Arguments
          1         32 HTTPParser
          1         32 Signal
          1         32 Timer
          1         32 WriteWrap
          1         56 DefineError.aV
          1         56 MathConstructor
          1         56 RangeError
          1         96 Console
          1        104 Agent
          1        112 Server
          1        120 exports.FreeList
          1        136 Module
          2         64 TTY
          2        208 EventEmitter
          2        208 WriteStream
          6        336 Error
          7        560 (ArrayBufferView)
         29        704 (Object)
         42       2352 NativeModule
       1219     146280 ServerResponse
       1219     263304 IncomingMessage
       1220      39040 TCP
       1859     416416 Socket
       1860     386880 WritableState
       2435     136360 WriteReq
       3080     591360 ReadableState
       3718     178528 CorkedRequest
       9708     543000 Object
       9747     467856 TickObject
      32585    1042720 (Array)

```

This is a simple app without a real issue, I'm just showing the commands and screenshots. At this point you'd be looking for large counts of an unexpected object to study further. You could also take two core dumps at different times, and then look at the difference between the findjsobject counts, to see which objects were growing.

Listing all (2) instances of WriteStream:

```
(lldb) v8 findjsinstances WriteStream
0x0000360583199f89:
0x0000360583199ff1:

```

Now inspecting one of those instances of WriteStream:

```
(lldb) v8 i 0x0000360583199f89
0x0000360583199f89:,
    ._hadError=0x00001c604c904251:,
    ._handle=0x0000360583119091:,
    ._parent=0x00001c604c904101:,
    ._host=0x00001c604c904101:,
    ._readableState=0x00003605831a8dd1:,
    .readable=0x00001c604c904251:,
    .domain=0x00001c604c904101:,
    ._events=0x00003605831a8e91:,
    ._eventsCount=,
    ._maxListeners=0x00001c604c9041b9:,
    ._writableState=0x00003605831a2609:,
    .writable=0x00001c604c904211:,
    .allowHalfOpen=0x00001c604c904251:,
    .destroyed=0x00001c604c904251:,
    ._bytesDispatched=,
    ._sockname=0x00001c604c904101:,
    ._writev=0x00001c604c904101:,
    ._pendingData=0x00001c604c904101:,
    ._pendingEncoding=0x00001c604c904291:,
    .server=0x00001c604c904101:,
    ._server=0x00001c604c904101:,
    .=,
    .columns=,
    .rows=,
    ._type=0x00001c604c9bd571:,
    .fd=,
    ._isStdio=0x00001c604c904211:,
    .destroySoon=0x00001002183b4821:,
    .destroy=0x00001002183b4821:}>

```

Listing all instances of Socket:

```
(lldb) v8 findjsinstances Socket
0x00000c41530496f9:
0x00000c4153049909:
0x00000c4153049b19:
0x00000c4153049d29:
0x00000c4153049f39:
0x00000c415304a149:
0x00000c415304a359:
[...]

```

Inspecting an instance of Socket:

```
(lldb) v8 i 0x00000c41530496f9
0x00000c41530496f9:,
    ._hadError=0x00001c604c904251:,
    ._handle=0x00001c604c904101:,
    ._parent=0x00001c604c904101:,
    ._host=0x00001c604c904101:,
    ._readableState=0x00000c4153087361:,
    .readable=0x00001c604c904251:,
    .domain=0x00001c604c904101:,
    ._events=0x00000c4153087421:,
    ._eventsCount=,
    ._maxListeners=0x00001c604c9041b9:,
    ._writableState=0x00000c41530497d9:,
    .writable=0x00001c604c904251:,
    .allowHalfOpen=0x00001c604c904211:,
    .destroyed=0x00001c604c904211:,
    ._bytesDispatched=,
    ._sockname=0x00001c604c904101:,
    ._pendingData=0x00001c604c904101:,
    ._pendingEncoding=0x00001c604c904291:,
    .server=0x0000360583196da1:,
    ._server=0x0000360583196da1:,
    .=,
    ._idleTimeout=,
    ._idleNext=0x00001c604c904101:,
    ._idlePrev=0x00001c604c904101:,
    ._idleStart=,
    .parser=0x00001c604c904101:,
    .on=0x0000360583196801:,
    ._paused=0x00001c604c904251:,
    .read=0x00001002183260c9:,
    ._consuming=0x00001c604c904211:,
    ._httpMessage=0x00001c604c904101:}>

```

One of the largest Object counts for my process was TickObject:

```
(lldb) v8 findjsinstances TickObject
0x00000c41535a9221:
0x00000c41535a9301:
0x00000c41535a93e1:
0x00000c41535a94c1:
0x00000c41535aa241:
0x00000c41535ab5a9:
0x00000c41535ab7b1:
[...]
(lldb) v8 i 0x00000c41535a9221
0x00000c41535a9221:,
    .domain=0x00001c604c904101:,
    .args=0x00000c41535a9171:}>

```

Fewer details for this object, but hopefully still enough to continue investigating. It's given me a callback function, plus I know the object name.

That's all for now. Hopefully these steps and screenshots are helpful, although do note that the steps are likely to be improved in future versions. Follow [llnode](https://github.com/nodejs/llnode) for the latest.

For more reading:

- [findrefs command for llnode](https://developer.ibm.com/node/2016/10/13/findrefs-command-for-llnode/).
- [Advances in core-dump debugging for Node.js](https://developer.ibm.com/node/2016/09/27/advances-in-core-dump-debugging-for-node-js/), including an mdb v8 vs llnode comparison.

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
