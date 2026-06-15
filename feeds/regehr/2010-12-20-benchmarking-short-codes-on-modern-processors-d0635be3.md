---
title: Benchmarking Short Codes on Modern Processors
url: https://blog.regehr.org/archives/330
published: "2010-12-20T23:45:56Z"
feed: regehr
guid: http://blog.regehr.org/?p=330
---

# Benchmarking Short Codes on Modern Processors

Around 15 years ago, as a newish graduate student, I got access to a Pentium-based Linux machine. One of the coolest things about this machine was the new RDTSC instruction that measured the number of clock ticks since the processor had been reset. This could be used to directly observe cache misses, branch mispredictions, and other low-level performance phenomena; it was a little like looking through a microscope for the first time. I ended up using the timestamp counter for a lot of things during the next few years, including in my [Hourglass](http://www.cs.utah.edu/flux/papers/hourglass-usenix02/) tool.

Lately my student Yang Chen has been using RDTSC to [measure the execution times of function calls](https://blog.regehr.org/archives/320) that take on the order of 10 to 1000 cycles to execute. Unfortunately, this activity is no longer nearly as straightforward as it once was. This post lists some things we learned. It’s somewhat specific to GCC and Linux.

# Enabling Basic Processor-Level Timing Stability

In the BIOS, turn off hyperthreading and anything having to do with turbo mode and frequency scaling. Turn off all OS-level frequency scaling as well; this is of course specific to the OS. [These instructions](http://superuser.com/questions/179329/disable-frequency-scaling-ondemand-daemon-on-ubuntu-10-04) worked fine for us.

# Avoiding Scheduling and Multicore Problems

The scheduler can mess up benchmarking in two main ways. First, it can simply take the processor away from the code being benchmarked. Second, it can migrate code to a different processor, slowing it down and also causing problems because the timestamp counters are typically not closely synchronized across CPUs. There seem to be three approaches to dealing with context switches messing up timing:

1. Ignore them. This is reasonable when timing events that are very short because context switches aren’t that fast; they’ll show up as massive outliers that are easy to discard.
2. Minimize their impact by raising the priority of running code using sched\_setscheduler() and by pinning code to a single processor using the taskset command. Both of these are easy in Linux, although priority elevation requires root privileges.
3. Lock out the scheduler by disabling interrupts. This can be done from user-mode using the iopl(3) call, or can it be done by running code in a kernel module. Either way, root privileges are required. Incautious disabling of interrupts is a good way to necessitate hard reboots of a machine.

# Dealing with Out-of-Order Execution

The RDTSC instruction is not serializing: it can be reordered significantly with respect to other in-flight instructions, causing serious (apparent) timing anomalies. Recently [this fantastic white paper](http://edc.intel.com/Link.aspx?id=3954) addressing this issue was released by Intel. It would have saved Yang and me a ton of time if it had existed six months earlier. The short answer is that if you’re running a newish processor in 64-bit mode, just use these functions to start and stop the timer:

> ```
> typedef unsigned long long ticks;
>
> static __inline__ ticks start (void) {
>   unsigned cycles_low, cycles_high;
>   asm volatile ("CPUID\n\t"
> 		"RDTSC\n\t"
> 		"mov %%edx, %0\n\t"
> 		"mov %%eax, %1\n\t": "=r" (cycles_high), "=r" (cycles_low)::
> 		"%rax", "%rbx", "%rcx", "%rdx");
>   return ((ticks)cycles_high << 32) | cycles_low;
> }
>
> static __inline__ ticks stop (void) {
>   unsigned cycles_low, cycles_high;
>   asm volatile("RDTSCP\n\t"
> 	       "mov %%edx, %0\n\t"
> 	       "mov %%eax, %1\n\t"
> 	       "CPUID\n\t": "=r" (cycles_high), "=r" (cycles_low):: "%rax",
> 	       "%rbx", "%rcx", "%rdx");
>   return ((ticks)cycles_high << 32) | cycles_low;
> }
>
> ```

Why use RDTSC to start the timer and RDTSCP to stop it? The white paper explains this and contains a lot of other good details.

# Other Precautions

- Code and data should be cache-line aligned when possible.
- The machine under test should be quiescent. It could also be booted in single-user mode, but we don’t bother.
- [Turning off ASLR](http://superuser.com/questions/127238/how-to-turn-off-aslr-in-ubuntu-9-10) seems like a good idea, though we have not done the measurements to see how much it affects repeatability.
- It used to be the case that paging would affect timing results, so we would use the mlockall() call to pin a process’s pages into RAM. Machines have so much memory these days that this hardly seems necessary.
- It’s possible to boot a multicore with one or more processors disabled; this will reduce or eliminate TLB shootdowns, other cache invalidation traffic, and memory system contention. Again, we don’t bother.

# Analyzing Data

The last important piece of the puzzle is to use robust statistical methods. Never take the average of a series of timing values: their distribution is almost always skewed by massive outliers. The minimum value may or may not be safe to use — there could be outliers on the low end due to migrations between unsynchronized processors. A good starting point would be to look at data points at the 25th, 50th, and 75th percentiles; if these are not very close together, something is probably amiss.

Any time you get timing data from a real processor, it must be scrutinized carefully to see if it makes sense. A good understanding of modern processor architectures really helps. We’ve seen odd things happen such as a machine using frequency scaling when we thought it had been permanently disabled. Either a kernel upgrade or similar disturbed our configuration or we simply messed up.

# Summary

Modern computers create a fairly hostile environment for accurate benchmarking of short-duration events. To do it right, you want to use a dedicated machine, carefully navigate the issues I’ve outlined (and whatever others I’ve missed), and develop a benchmarking procedure that includes plenty of sanity checks.
