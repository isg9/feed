---
title: 'research!rsc: How To Build a User-Level CPU Profiler'
url: https://research.swtch.com/pprof
published: "2013-08-08T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/pprof
---

# research!rsc: How To Build a User-Level CPU Profiler

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# How To Build a User-Level CPU Profiler Russ Cox August 8, 2013 *research.swtch.com/pprof* Posted on Thursday, August 8, 2013.

When I spent a summer as a Google intern in 2006, one of the many pleasant surprises was Google’s pprof tool, which makes profiling a C++ program’s CPU and memory usage incredibly easy. It had already been open sourced, and when I returned to grad school, I incorporated pprof into my standard development toolbox when writing C programs. Later, when I was back at Google working on Go, implementing support for pprof was a must. Now it’s part of the standard development toolbox for any Go programmer.

I’ve written on the Go blog about [what it’s like to use pprof
to profile Go programs](http://blog.golang.org/profiling-go-programs). This post is about how pprof gathers the CPU profile, with the help of hardware timers and the operating system.

### Hardware Timers

One of the key jobs of an operating system is to allow multiple programs to run on a computer at the same time, each under the illusion that they have continuous access to one or more CPUs, even though in practice each CPU can only be running one program at a time. On most systems, what happens is that the operating system asks the computer’s timer chip to interrupt normal execution every so often (every 10 milliseconds is common) and run a small piece of the operating system called, appropriately enough, the timer interrupt handler. The timer interrupt handler checks to see if other programs are waiting to run. If so, and if the current program has used up its turn on the CPU, the handler saves the CPU registers of the current program and loads the CPU registers for another program that has been waiting. Shuffling the programs on and off the CPUs is called multitasking, and doing it based on hardware interrupts like this is called preemptive multitasking: one running program is preempted to make room for another.

Not all operating systems provide preemptive multitasking. Some, like MS-DOS or iOS 3, have no multitasking, so only one program can run at a time. Others, like Windows 3.1 and Mac OS 9, use cooperative multitasking, in which programs voluntarily relinquish the CPU at certain points. Cooperative multitasking avoids the complexity of using the timer, but it means that one badly written program can hang a machine.

Another useful thing you can do with a hardware interrupt is gather a profile of a running program. Every time the hardware interrupt happens, the operating system can record what the program was doing. A collection of these samples makes up a (hopefully representative) profile of where the program spends its time.

### A Simple Profiler

Since the operating system moderates access to the hardware, it must be involved to use the hardware timer to implement a profiler. The simplest approach is for the operating system to collect the profile. To illustrate this approach, I’ll describe what the Plan 9 kernel does for user-level profiling. This is little changed from the original Unix *profil*(2) scheme.

When a program is running, writing to a control file in the /proc file system enables profiling. At that point, the operating system allocates an array of counts with one count for every 8-byte section of the program code, and attaches it to the program. Then, each time a timer interrupt happens, the handler uses the program counter—the address of the instruction in the code that the program is currently executing—divided by 8 as an index into that array and increments that entry. Later, another program (on Plan 9, called *tprof*) can be run to read the profile from the kernel and determine the function and specific line in the original source code corresponding to each counter and summarize the results.

This design keeps the kernel very simple: the timer handler only adds an increment instruction. However, it is also inflexible: that’s all the kernel will do for a program. And there are potential improvements that could be made with more flexibility. For example, in programs that use libraries, the profiles may not have enough context to be useful. If you find your program is spending 30% of its time in `memset` but you don’t know what is calling `memset`, you can’t fix the problem.

The next step forward is to move profiling logic out of the kernel, so that it is easier to customize.

### User-Level Timers

Modern Unix systems move profiling logic out of the kernel by providing a timer abstraction to user programs. The programs can then build whatever they want, including profiling. (Another use might be to build preemptive scheduling in a user-level lightweight thread scheduler.)

On these systems, *setitimer*(2) provides three timers, for “real time,’’ “virtual time,’’ and “profiling time.’’ The “real time’’ timer counts down in real (wall clock) time. If it is set for 5 seconds, it fires 5 seconds from now. The “virtual time’’ timer counts down in program execution time. If it is set for 5 seconds, it fires after the program has executed for 5 seconds. The “profiling time’’ timer is similar except that it also counts down when the kernel is executing a system call on behalf of the program. When any of these timers fires, the kernel stops what the program is doing and invokes a special routine in the program called a signal handler. The signal handler in a program is analogous to the timer interrupt handler in the kernel. The signal being delivered depends on the kind of timer: `SIGALRM`, `SIGVTALRM`, or `SIGPROF` for the three kinds.

A program that wants to profile its own execution can set a profiling timer and then arrange to record a profiling sample in its signal handler. Because the profiling is being done by the program, not by the kernel, it can be replaced without kernel changes and reboots, by changing the program and recompiling. This allows experimentation and in turn the construction of richer profiles. In particular, it allows gathering more information in each profiling sample.

### Profiling with pprof

In pprof, each profiling sample records not just the current program counter but also the program counter for each frame in the current call stack.

Obtaining the current call stack is not always trivial: typically it requires arranging for the code being executed to maintain a certain form (for example, to use a frame pointer in all functions) or arranging for additional metadata to be available during the trace. But modern computers are incredibly fast: even walking the call stack using metadata lookups can be done in well under 100 microseconds, which would correspond to only a 1% slowdown for a 10 millisecond profiling period.

Recording a sampled stack is a little tricky. The signal handler, because it interrupts normal program execution, is limited in what it can do. In particular, the signal handler cannot even allocate memory or acquire locks, so the stored profile cannot grow dynamically.

In pprof, the profile is maintained in a hash table in which each entry records a distinct stack. Specifically, in the Go pprof, each stack hashes to a specific hash value denoting one of 1024 buckets in a hash table, and in that bucket are four entries for stacks with that hash. If the stack being recorded is already present, the existing entry’s count is simply incremented. Otherwise, the stack must replace one of the four entries. If one or more is unused, the decision is easy. If all four entries are in use, pprof evicts the entry with the smallest count. The evicted entry is appended to a fixed-size log with space for 218 entries, and then its space is reused for the stack we want to record. A background goroutine, not running as a signal handler, copies entries out of the log and into the saved profile data. If the goroutine cannot keep up with the rate at which the log is filling, the counts for the entries that are lost are assigned to a special “lost profile’’ function. In practice, most programs spend their time doing the same thing over and over again, so the active work of the program fits in the hash table, the log grows slowly, and the log draining goroutine has no problem keeping up. In C++ pprof, there is no log-draining goroutine. Instead, the signal handler writes the log to an already open file when it fills.

### Interpreting the data

A pprof profile, then, is a set of stack traces with each mapped to the count of the number of times that trace was seen. The usual way this profile is presented is to create a graph in which each function is a node and an edge between nodes denotes a call. Each function node has two counts: first the sum of the counts of all the profiles in which it was the final entry and second the sum of the counts of all the profiles in which it appeared. These correspond to time executing that function and cumulative time executing that function and the things it called. The edges are given counts corresponding to the number of samples in which that call was observed somewhere on the stack. Sizing the nodes and edges based on their counts makes the expensive parts of the program stand out. [This example](http://benchgraffiti.googlecode.com/hg/havlak/havlak1.svg) should view nicely in most web browsers; use the scroll wheel to zoom and click and drag to pan. In that profile, it is apparent that `hash_lookup` called from `mapaccess1` is taking the most time, and we can use the call context to see that about 40% of the calls are from code in `main.FindLoops` while 60% of the calls are from code in `main.DFS`.

[![](pprof.png)](http://benchgraffiti.googlecode.com/hg/havlak/havlak1.svg)

Of course, there is also a command-line interface, tabular results, and annotated source code. See the [Profiling Go Programs](http://blog.golang.org/profiling-go-programs) post.

### Further Reading

Timer-based profiling is very old. On Unix, the heritage dates back at least to the Seventh Edition. The [Seventh Edition Unix manual](https://9p.io/7thEdMan/v7vol1.pdf) contains documentation for *prof*(1), *profil*(2), and *monitor*(3), and you can even read the source for [monitor](http://www.tuhs.org/Archive/PDP-11/Trees/V7/usr/src/libc/gen/mon.c). However, pprof is the first profiler I’ve seen that records and presents stack traces well. If you’d like to read more about pprof, it is part of the C++ [gperftools](https://code.google.com/p/gperftools/) open source project and has good documentation. The Go implementation of the CPU profile collection is in [runtime/cpuprof.c](http://golang.org/src/pkg/runtime/cpuprof.c).

It is of course also possible to collect profiles without timers, such as by rewriting the program code. In general the overhead of these tends to be larger than timer-based sampling, and it can skew the results so that what looks expensive with profiling enabled is not expensive normally.

Modern CPUs have introduced all kinds of other profiling timers, based on instructions executed, cache misses, and so on. These are very powerful, but so far the interfaces to them are operating system-specific. One nice thing about pprof is that, because it’s using an old, widely supported Unix feature, it’s fairly portable. I can use a single tool on FreeBSD, Linux, and OS X.

*Setitimer*(2) was introduced by 4.2BSD; that indirection makes excellent tools like pprof possible. Of course, it has to work correctly. In my next post I’ll look at one system where it doesn’t.
