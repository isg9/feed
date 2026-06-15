---
title: 'research!rsc: Hacking the OS X Kernel for Fun and Profiles'
url: https://research.swtch.com/macpprof
published: "2013-08-13T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/macpprof
---

# research!rsc: Hacking the OS X Kernel for Fun and Profiles

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Hacking the OS X Kernel for Fun and Profiles Russ Cox August 13, 2013 *research.swtch.com/macpprof* Posted on Tuesday, August 13, 2013.

My last post described how user-level CPU profilers work, and specifically how Google’s pprof profiler gathers its CPU profiles with the help of the operating system. The specific feature needed from the operating system is the profiling timer provided by *setitimer*(2) and the `SIGPROF` signals that it delivers.

If the operating system’s implementation of that feature doesn’t work, then the profiler doesn’t work. This post looks at a common bug in Unix implementations of profiling signals and the fix for OS X, applied by editing the OS X kernel binary.

If you haven’t read “ [How to Build a User-Level CPU Profiler](pprof),’’ you might want to start there.

### Unix and Signals and Threads

My earlier post referred to profiling *programs*, without mention of processes or threads. Unix in general and SIGPROF in particular predate the idea of threads. SIGPROF originated in the 4.2BSD release of Berkeley Unix, published in 1983. In Unix at the time, a process was a single thread of execution.

Threads did not come easily to Unix. Early implementations were slow and buggy and best avoided. Each of the popular Unix variants added thread support independently, with many shared mistakes.

Even before we get to implementation, many of the original Unix APIs are incompatible with the idea of threads. Multithreaded processes allow multiple threads of execution in a single process address space. Unix maintains much per-process state, and the kernel authors must decide whether each piece of state should remain per-process or change to be per-thread.

For example, the single process stack must be split into per-thread stacks: it is impossible for independently executing threads to be running on a single stack. Because there are many threads, thread stacks tend to be smaller than the one big process stack that non-threaded Unix programs had. As a result, it can be important to define a separate stack for running signal handlers. That setting is per-thread, for the same reason that ordinary stacks are per-thread. But the choice of handler is per-process.

File descriptors are per-process, but then one thread might open a file moments before another thread forks and execs a new program. In order for the open file not to be inherited by the new program, we must introduce a new variant of *open*(2) that can open a file descriptor atomically marked “close on exec.’’ And not just open: every system call that creates a new file descriptor needs a variant that creates the file descriptor “close on exec.’’

Memory is per-process, so malloc must use a lock to serialize access by independent threads. But again, one thread might acquire the malloc lock moments before another thread forks and execs a new program. The fork makes a new copy of the current process memory, including the locked malloc lock, and that copy will never see the unlock by the thread in the original program. So the child of fork can no longer use malloc without occasional deadlocks.

That’s just the tip of the iceberg. There are a lot of changes to make, and it’s easy to miss one.

### Profiling Signals

Here’s a thread-related change that is easy to miss.

The goal of the profiling signal is to enable user-level profiling. The signal is sent in response to a program using up a certain amount of CPU time. More specifically, in a multithreaded kernel, the profiling signal is sent when the hardware timer interrupts a thread and the timer interrupt handler finds that the execution of that thread has caused the thread’s process’s profiling timer to expire. In order to profile the code whose execution triggered the timer, the profiling signal must be sent to the thread that is running. If the signal is sent to a thread that is not running, the profile will record idleness such as being blocked on I/O or sleeping as execution and will be neither accurate nor useful.

Modern Unix kernels support sending a signal to a process, in which case it can be delivered to an arbitrary thread, or to a specific thread. *Kill*(2) sends a signal to a process, and *pthread\_kill*(2) sends a signal to a specific thread within a process.

Before Unix had threads, the code that delivered a profiling signal looked like `psignal(p,` `SIGPROF)`, where `psignal` is a clearer name for the implementation of the *kill*(2) system call and `p` is the process with the timer that just expired. If there is just one thread per process, delivering the signal to the process cannot possibly deliver it to the wrong thread.

In multithreaded programs, the `SIGPROF` must be delivered to the running thread: the kernel must call the internal equivalent of *pthread\_kill*(2), not *kill*(2).

FreeBSD and Linux deliver profiling signals correctly. Empirically, NetBSD, OpenBSD, and OS X do not. (Here is a [simple C test program](sigtest.c).) Without correct delivery of profiling signals, it is impossible to build a correct profiler.

### OS X Signal Delivery

To Apple’s credit, the OS X kernel sources are published and open source, so we can look more closely at the buggy OS X implementation.

The profiling signals are delivered by the function `bsd_ast` in the file [kern\_sig.c](http://www.opensource.apple.com/source/xnu/xnu-2050.22.13/bsd/kern/kern_sig.c). Here is the relevant bit of code:

```
void
bsd_ast(thread_t thread)
{
    proc_t p = current_proc();
    ...
    if (timerisset(&p->p_vtimer_prof.it_value)) {
        uint32_t    microsecs;

        task_vtimer_update(p->task, TASK_VTIMER_PROF, µsecs);

        if (!itimerdecr(p, &p->p_vtimer_prof, microsecs)) {
            if (timerisset(&p->p_vtimer_prof.it_value))
                task_vtimer_set(p->task, TASK_VTIMER_PROF);
            else
                task_vtimer_clear(p->task, TASK_VTIMER_PROF);

            psignal(p, SIGPROF);
        }
    }
    ...
}

```

The `bsd_ast` function is the BSD half of the OS X timer interrupt handler. If profiling is enabled, `bsd_ast` decrements the timer and sends the signal if the timer expires. The innermost if statement is resetting the the timer state, because *setitimer*(2) allows both one-shot and periodic timers.

As predicted, the code is sending the profiling signal to the process, not to the current thread. There is a function `psignal_uthread` defined in the same source file that sends a signal instead to a specific thread. One possible fix is very simple: change `psignal` to `psignal_uthread`.

I filed a report about this bug as [Apple Bug Report #9177434](http://golang.org/change/35b716c94225) in March 2011, but the bug has persisted in subsequent releases of OS X. In my report, I suggested a different fix, inside the implementation of `psignal`, but changing `psignal` to `psignal_uthread` is even simpler. Let’s do that.

### Patching the Kernel

It should be possible to rebuild the OS X kernel from the released sources. However, I do not know whether the sources are complete, and I do not know what configuration I need to use to recreate the kernel on my machine. I have no confidence that I’d end up with a kernel appropriate for my computer. Since the fix is so simple, it should be possible to just modify the standard OS X kernel binary directly. That binary lives in `/mach_kernel` on OS X computers.

If we run `gdb` on `/mach_kernel` we can see the compiled machine code for `bsd_ast` and find the section we care about.

```
$ gdb /mach_kernel
(gdb) disas bsd_ast
Dump of assembler code for function bsd_ast:
0xffffff8000568a50 : push   %rbp
0xffffff8000568a51 : mov    %rsp,%rbp
...
if (timerisset(&p->p_vtimer_prof.it_value))
0xffffff8000568b7b :       cmpq   $0x0,0x1e0(%r15)
0xffffff8000568b83 :       jne    0xffffff8000568b8f
0xffffff8000568b85 :       cmpl   $0x0,0x1e8(%r15)
0xffffff8000568b8d :       je     0xffffff8000568b9f
task_vtimer_set(p->task, TASK_VTIMER_PROF);
0xffffff8000568b8f :       mov    0x18(%r15),%rdi
0xffffff8000568b93 :       mov    $0x2,%esi
0xffffff8000568b98 :       callq  0xffffff80002374f0
0xffffff8000568b9d :       jmp    0xffffff8000568bad
task_vtimer_clear(p->task, TASK_VTIMER_PROF);
0xffffff8000568b9f :       mov    0x18(%r15),%rdi
0xffffff8000568ba3 :       mov    $0x2,%esi
0xffffff8000568ba8 :       callq  0xffffff8000237660
psignal(p, SIGPROF);
0xffffff8000568bad :       mov    %r15,%rdi
0xffffff8000568bb0 :       xor    %esi,%esi
0xffffff8000568bb2 :       xor    %edx,%edx
0xffffff8000568bb4 :       xor    %ecx,%ecx
0xffffff8000568bb6 :       mov    $0x1b,%r8d
0xffffff8000568bbc :       callq  0xffffff8000567340
...

```

I’ve annotated the assembly with the corresponding C code in italics. The final sequence is odd. It should be a call to `psignal` but instead it is a call to code 224 bytes beyond the start of the `threadsignal` function. What’s going on is that `psignal` is a thin wrapper around `psignal_internal`, and that wrapper has been inlined. Since `psignal_internal` is a static function, it does not appear in the kernel symbol table, and so `gdb` doesn’t know its name.

The definitions of `psignal` and `psignal_uthread` are:

```
void
psignal(proc_t p, int signum)
{
    psignal_internal(p, NULL, NULL, 0, signum);
}

static void
psignal_uthread(thread_t thread, int signum)
{
    psignal_internal(PROC_NULL, TASK_NULL, thread, PSIG_THREAD, signum);
}

```

With the constants expanded, the call we’re seeing is `psignal_internal(p,` `0,` `0,` `0,` `0x1b)` and the call we want to turn it into is `psignal_internal(0,` `0,` `thread,` `4,` `0x1b)`. All we need to do is prepare the different argument list.

Unfortunately, the `thread` variable was passed to `bsd_ast` in a register, and since it is no longer needed where we are in the function, the register has been reused for other purposes: `thread` is gone.

Fortunately, `bsd_ast`’s one and only invocation in the kernel is `bsd_ast(current_thread())`, so we can reconstruct the value by calling `current_thread` ourselves.

Unfortunately, there is no room in the 15 bytes from `bsd_ast+349` to `bsd_ast+364` to insert such a call and still prepare the other arguments.

Fortunately, we can optimize a bit of the preceding code to make room. Notice that the calls to `task_vtimer_set` and `task_vtimer_clear` are passing the same argument list, and that argument list is prepared in both sides of the conditional:

```
...
if (timerisset(&p->p_vtimer_prof.it_value))
0xffffff8000568b7b :       cmpq   $0x0,0x1e0(%r15)
0xffffff8000568b83 :       jne    0xffffff8000568b8f
0xffffff8000568b85 :       cmpl   $0x0,0x1e8(%r15)
0xffffff8000568b8d :       je     0xffffff8000568b9f
task_vtimer_set(p->task, TASK_VTIMER_PROF);
0xffffff8000568b8f :       mov    0x18(%r15),%rdi
0xffffff8000568b93 :       mov    $0x2,%esi
0xffffff8000568b98 :       callq  0xffffff80002374f0
0xffffff8000568b9d :       jmp    0xffffff8000568bad
task_vtimer_clear(p->task, TASK_VTIMER_PROF);
0xffffff8000568b9f :       mov    0x18(%r15),%rdi
0xffffff8000568ba3 :       mov    $0x2,%esi
0xffffff8000568ba8 :       callq  0xffffff8000237660
psignal(p, SIGPROF);
0xffffff8000568bad :       mov    %r15,%rdi
0xffffff8000568bb0 :       xor    %esi,%esi
0xffffff8000568bb2 :       xor    %edx,%edx
0xffffff8000568bb4 :       xor    %ecx,%ecx
0xffffff8000568bb6 :       mov    $0x1b,%r8d
0xffffff8000568bbc :       callq  0xffffff8000567340
...

```

We can pull that call setup above the conditional, eliminating one copy and giving ourselves nine bytes to use for delivering the signal. A call to `current_thread` would take five bytes, and then moving the result into an appropriate register would take two more, so nine is plenty. In fact, since we have nine bytes, we can inline the body of `current_thread`—a single nine-byte `mov` instruction—and change it to store the result to the correct register directly. That avoids needing to prepare a position-dependent call instruction.

The final version is:

```
...
0xffffff8000568b7b :       mov    0x18(%r15),%rdi
0xffffff8000568b7f :       mov    $0x2,%esi
0xffffff8000568b84 :       cmpq   $0x0,0x1e0(%r15)
0xffffff8000568b8c :       jne    0xffffff8000568b98
0xffffff8000568b8e :       cmpl   $0x0,0x1e8(%r15)
0xffffff8000568b96 :       je     0xffffff8000568b9f
0xffffff8000568b98 :       callq  0xffffff80002374f0
0xffffff8000568b9d :       jmp    0xffffff8000568ba4
0xffffff8000568b9f :       callq  0xffffff8000237660
0xffffff8000568ba4 :       xor    %edi,%edi
0xffffff8000568ba6 :       xor    %esi,%esi
0xffffff8000568ba8 :       mov    %gs:0x8,%rdx
0xffffff8000568bb1 :       mov    $0x4,%ecx
0xffffff8000568bb6 :       mov    $0x1b,%r8d
0xffffff8000568bbc :       callq  0xffffff8000567340
...

```

If we hadn’t found the duplicate call setup to factor out, another possible approach would have been to factor the two very similar code blocks handling `SIGVTALRM` and `SIGPROF` into a single subroutine, sitting in the middle of the `bsd_ast` function code, and to call it twice. Removing the second copy of the code would leave plenty of space for the longer `psignal_uthread` call setup.

The code we’ve been using is from OS X Mountain Lion, but all versions of OS X have this bug, and the relevant bits of `bsd_ast` haven’t changed from version to version, although the compiler and therefore the generated code do change. Even so, all have the basic pattern and all can be fixed with the same kind of rewrite.

### Using the Patch

If you use the Go or the C++ gperftools and want accurate CPU profiles on OS X, I’ve packaged up the binary patcher as [code.google.com/p/rsc/cmd/pprof\_mac\_fix](http://godoc.org/code.google.com/p/rsc/cmd/pprof_mac_fix). It can handle OS X Snow Leopard, Lion, and Mountain Lion. Will OS X Mavericks need a fix too? We’ll see.

### Further Reading

Binary patching is an old, venerable technique. This is just a simple instance of it. If you liked reading about this, you may also like to read Jeff Arnold’s paper “ [Ksplice: Automatic Rebootless Kernel Updates](http://www.ksplice.com/doc/ksplice.pdf).’’ Ksplice can construct binary patches for Linux security vulnerabilities and apply them on the fly to a running system.
