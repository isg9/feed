---
title: It&#8217;s Time for a Modern Synthesis Kernel
url: https://blog.regehr.org/archives/1676
published: "2019-07-02T17:13:00Z"
feed: regehr
guid: http://blog.regehr.org/?p=1676
---

# It&#8217;s Time for a Modern Synthesis Kernel

[Alexia Massalin’s 1992 PhD thesis](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.29.4871&rep=rep1&type=pdf) has long been one of my favorites. It promotes the view that operating systems can be much more efficient than then-current operating systems via runtime code generation, lock-free synchronization, and fine-grained scheduling. In this piece we’ll only look at runtime code generation, which can be cleanly separated from the other aspects of this work.

## Runtime Code Generation in Ring 0

The promise of kernel-mode runtime code generation is that we can have very fast, feature-rich operating systems by, for example, not including code implementing generic read() and write() system calls, but rather synthesizing code for these operations each time a file is opened. The idea is that at file open time, the OS has a lot of information that it can use to generate highly specialized code, eliding code paths that are provably not going to execute.

Runtime code generation was a well-known idea in 1992, but it wasn’t used nearly as widely as it is today. In 2019, of course, just-in-time compilers are ubiquitous. However, operating system kernels still do not use runtime code generation very much, with a few exceptions such as:

- several OS kernels, including Linux, have a simple JIT compiler in their BPF implementation

- VMware used to use dynamic code generation to performantly virtualize OS kernels on x86 chips that lacked hardware virtualization extensions; I doubt that this is commonly used any longer

- pre-NT Windows kernels would dynamically generate bitblit code. I learned this in a talk by a VMware employee; this code generation was apparently a debugging issue for VMware since it would fight with their own runtime code generator. Some details can be found [in this post](https://blogs.msdn.microsoft.com/oldnewthing/?p=97995). The [old paper](https://pdos.csail.mit.edu/~rsc/pike84bitblt.pdf) about the origins of this technique in the Xerox Alto is a classic.

- TempleOS, as explained in this [nice writeup](http://www.codersnotes.com/notes/a-constructive-look-at-templeos/), made heavy use of dynamic code generation

Anyway, back to Synthesis. The OS and code generators were all written, from scratch, in 68020 assembly language. How do we translate Massalin’s ideas to 2019? Most likely by reusing an existing code generator and OS. For most of this piece I’ll assume that that’s what we want to do, but we’ll also briefly touch on customized alternatives.

## Code Generator Requirements

The particular technology that we use for runtime code generation isn’t that important, but for now let’s imagine using LLVM. This means that the pieces of the kernel that we wish to specialize will need to be shipped as bitcode, and then we’ll ask LLVM to turn it into object code as needed. LLVM has lots of great optimization passes, from which we could pick a useful subset, and it is [not hard to use in JIT mode](https://www.youtube.com/watch?v=hILdR8XRvdQ). On the other hand, LLVM isn’t as fast as we’d like and also it has a large footprint. In production we’d need to think carefully whether we wanted to include a big chunk of non-hardened code in the kernel.

What optimizations are we expecting the code generator to perform? Mostly just the basic ones: function inlining, constant propagation, and dead code elimination, followed by high-quality instruction selection and register allocation. The hard part, as we’re going to see, is convincing LLVM that it is OK to perform these optimizations as aggressively as we want. This is an issue that Massalin did not need to confront: her kernel was designed in such a way that she knew exactly what could be specialized and when. Linux, on the other hand, was obviously not created with staged compilation in mind, and we’re going to have to improvise somewhat if we want this to work well.

My guess is that while LLVM would be great for prototyping purposes, for deployment we’d probably end up either reusing a lighter-weight code generator or else creating a new one that is smaller, faster, and more suitable for inclusion in the OS. Performance of runtime code generation isn’t just a throughput issue, there’ll also be latency problems if we’re not careful. We need to think about the impact on security, too.

## Example: Specializing write() in Linux

Let’s assume that we’ve created a version of Linux that is capable of generating a specialized version of the write() system call for a pipe. This OS needs — but we won’t discuss — a system call dispatch mechanism to rapidly call the specialized code when it is available. In Synthesis this was done by giving each process its own trap vector.

Before we dive into the code, let’s be clear about what we’re doing here: we are pretending to be the code generator that is invoked to create a specialized write() method. Probably this is done lazily at the time the system call is first invoked using the new file descriptor. The specialized code can be viewed as a cached computation, and as a bonus this cache is self-invalidating: it should be valid as long as the file descriptor itself is valid. (But later we’ll see that we can do a better job specializing the kernel if we support explicit invalidation of runtime-generated code.)

If you want to follow along at home, I’m running Linux 5.1.14 under QEMU, using [these instructions](https://www.anmolsarma.in/post/single-step-kernel/) to single-step through kernel code, and driving the pipe logic using [this silly program](https://gist.github.com/regehr/02b48aa3cf96df423401b09878bc03b6).

Skipping over the trap handler and such, [ksys\_write()](https://github.com/torvalds/linux/blob/e93c9c99a629c61837d5a7fc2120cd2b6c70dbdd/fs/read_write.c#L592-L606) is where things start to happen for real:

```
ssize_t ksys_write(unsigned int fd, const char __user *buf, size_t count)
{
	struct fd f = fdget_pos(fd);
	ssize_t ret = -EBADF;

	if (f.file) {
		loff_t pos = file_pos_read(f.file);
		ret = vfs_write(f.file, buf, count, &pos);
		if (ret >= 0)
			file_pos_write(f.file, pos);
		fdput_pos(f);
	}

	return ret;
}

```

At this point the “fd” parameter can be treated as a compile-time constant, but of course “buf” and “count” cannot. If we turn “fd” into a constant, will LLVM be able to propagate it through the remaining code? It will, as long as:

1. We inline all function calls.

2. Nobody takes the address of “fd”.

It’s not that calls and pointers will always block the optimizer, but they complicate things by bringing interprocedural analysis and pointer analysis into the picture.

Our goal is going to be to see whether the code generator can infer the contents of the struct returned from fdget\_pos(). (You might wonder why performance-sensitive code is returning a “struct fd” by value. Turns out this struct only has two members: a pointer and an integer.)

The call to fdget\_pos() goes to [this code](https://github.com/torvalds/linux/blob/e93c9c99a629c61837d5a7fc2120cd2b6c70dbdd/include/linux/file.h#L70-L73):

```
static inline struct fd fdget_pos(int fd)
{
	return __to_fd(__fdget_pos(fd));
}

```

and then [here](https://github.com/torvalds/linux/blob/e93c9c99a629c61837d5a7fc2120cd2b6c70dbdd/fs/file.c#L793-L805):

```
unsigned long __fdget_pos(unsigned int fd)
{
	unsigned long v = __fdget(fd);
	struct file *file = (struct file *)(v & ~3);

	if (file && (file->f_mode & FMODE_ATOMIC_POS)) {
		if (file_count(file) > 1) {
			v |= FDPUT_POS_UNLOCK;
			mutex_lock(&file->f_pos_lock);
		}
	}
	return v;
}

```

and then (via a trivial helper that I’m not showing) [here](https://github.com/torvalds/linux/blob/e93c9c99a629c61837d5a7fc2120cd2b6c70dbdd/fs/file.c#L765-L781):

```
static unsigned long __fget_light(unsigned int fd, fmode_t mask)
{
	struct files_struct *files = current->files;
	struct file *file;

	if (atomic_read(&files->count) == 1) {
		file = __fcheck_files(files, fd);
		if (!file || unlikely(file->f_mode & mask))
			return 0;
		return (unsigned long)file;
	} else {
		file = __fget(fd, mask, 1);
		if (!file)
			return 0;
		return FDPUT_FPUT | (unsigned long)file;
	}
}

```

Keep in mind that up to here, we haven’t seen any optimization blockers. In \_\_fdget\_light(), we run into our first interesting challenge: “current” is a macro that returns a pointer to the running process’s PCB (in Linux the PCB, or process control block, is a “task\_struct” but I’ll continue using the generic term). The current macro ends up being [a tiny bit magical](https://kernelnewbies.org/FAQ/current), but its end result can be treated as a constant within the context of a given process. There is no way a code generator like LLVM will be able to reach this conclusion, so we’ll need to give it some help, perhaps by annotating certain functions, macros, and struct fields as returning values that are constant over a given scope. This is displeasing but it isn’t clear there’s any easier or better way to achieve our goal here. The best we can hope for is that the annotation burden is close to proportional to the number of data types in the kernel; if it ends up being proportional to the total amount of code then our engineering effort goes way up.

Now, assuming that we can treat “current” as a compile-time constant, we’re immediately faced with a similar question: is the “files” field of the PCB constant? It is (once the process is initialized) but again there’s not going to be any easy way for our code generator to figure this out; we’ll need to rely on another annotation.

Continuing, the “count” field of files is definitely not a constant: this is a reference count on the process’s file descriptor table. A single-threaded Linux process will never see count > 1, but a multi-threaded process will. (Here we need to make the distinction between open file instances, which are shared following a fork, and the file descriptor table, which is not.) The fast path here is exploiting the insight that if our process is single-threaded we don’t need to worry about locking the file descriptor table, and moreover the process is not going to stop being single-threaded during the period where we rely on that invariant, because we trust the currently running code to not do the wrong thing.

Here our specializing compiler has a fun policy choice to make: should it specialize for the single threaded case? This will streamline the code a bit, but it requires the generated code to be invalidated later on if the process does end up becoming multithreaded — we’d need some collection of invalidation hooks to make that happen.

Anyhow, let’s continue into [\_\_fcheck\_files()](https://github.com/torvalds/linux/blob/e93c9c99a629c61837d5a7fc2120cd2b6c70dbdd/include/linux/fdtable.h#L82-L91):

```
static inline struct file *__fcheck_files(struct files_struct *files, unsigned int fd)
{
	struct fdtable *fdt = rcu_dereference_raw(files->fdt);

	if (fd < fdt->max_fds) {
		fd = array_index_nospec(fd, fdt->max_fds);
		return rcu_dereference_raw(fdt->fd[fd]);
	}
	return NULL;
}

```

At this point we’re in deep “I know what I’m doing” RCU territory and I’m going to just assume we can figure out a way for the code generator to do what we want, which is to infer that this function returns a compile-time-constant value. I think this’ll work out in practice, since even if the open file instance is shared across processes, the file cannot be truly closed until its reference count goes to zero. Anyway, let’s move forward.

Next, we’re back in \_\_fget\_light() and then \_\_fdget\_pos(): our code generator should be able to easily fold away the remaining branches in these functions. Finally, we return to line 4 of ksys\_write() and we know what the struct fd contains, making it possible to continue specializing aggressively. I don’t think making this example any longer will be helpful; hopefully the character of the problems we’re trying to solve are now apparent.

In summary, we saw four kinds of variables in this exercise:

1. Those such as the “fd” parameter to write() that the code generator can see are constant at code generation time.

2. Those such as the “current” pointer that are constant, but where the code generator cannot see this fact for one reason or another. To specialize these, we’ll have to give the compiler extra information, for example using annotations.

3. Those such as the “count” field of the “files\_struct” that are not actually constant, but that seem likely enough to remain constant that we may want to create a specialized version treating them as constants, and then be ready to invalidate this code if the situation changes.

4. Those that are almost certainly not worth trying to specialize. For example, the “count” parameter to write() is not likely to remain constant over a number of calls.

Writing one byte to a pipe from a single-threaded process executes about 3900 instructions on Linux 5.1.14 (this is just in ksys\_write(), I didn’t measure the trapping and untrapping code). The Synthesis thesis promises an order of magnitude performance improvement. Can specialization reduce the fast path on this system call to 390 instructions? It would be fun to find out.

I’ll finish up this example by observing that even though I chose to present code from the filesystem, I think it’s network stack code that will benefit from specialization the most.

## Discussion

I have some experience with OS kernels other than Linux, and my belief is that attempting to dynamically specialize any mainstream, production-grade OS other than Linux would run into the same issues we just saw above. At the level the code generator cares about, there just isn’t much effective difference between these OSes: they’re all big giant blobs of C with plentiful indirection and domain-specific hacks.

If our goal is only to create a research-grade prototype, it would be better to start with something smaller than Linux/Windows/Darwin so that we can refactor specialization-unfriendly parts of the OS in a reasonable amount of time. xv6 is at the other extreme: it is super easy to hack on, but it is so incredibly over-simplified that it could not be used to test the hypothesis “a realistic OS can be made much faster using specialization.” Hilariously, an xv6+LLVM system would be about 0.15% OS code and 99.85% compiler code. Perhaps there’s a middle ground that would be a better choice, Minix or OpenBSD or whatever.

Given two developers, one who knows LLVM’s JIT interfaces and one who’s a good Linux kernel hacker, how long would it take to bring up a minimally ambitious dynamically specializing version of Linux? I would guess this could be done in a week or two, there’s not really anything too difficult about it (it’s easy to say this while blogging, of course). The problem is that this would not give good results: only the very easiest specialization opportunities will get spotted by the runtime code generator. But perhaps this would generate enough interest that people would keep building on it.

Do we want to do specialization work on C code? No, not really, it’s just that every one of our production-grade kernels is already written in it. A fun but engineering-intensive alternative would be to create a new, specialization-friendly kernel in whatever programming language looks most suitable. Functional languages should offer real advantages here, but of course there are issues in using these languages to create a performant OS kernel. Perhaps [Mirage](https://mirage.io/) is a good starting point here, it is already all about specialization — but at system build time, not at runtime.

An ideal programming environment for a modern Synthesis kernel would provide tool and/or language support for engineering specialization-friendly kernel code. For example, we would identify a potential specialization point and then the tools would use all of our old friends — static analysis, dynamic analysis, symbolic execution, etc. — to show us what data items fall into each of the four categories listed in the last section, and provide us with help in refactoring the system so that specialization can work better. A tricky thing here is taking into account the different kinds of concurrency and synchronization that happen in a sophisticated OS.

Some useful questions to ask (and of course we’re always asking these same things when doing OS and compiler research) are: How are we supposed to think about a dynamically specializing OS kernel? What are the new abstractions, if any? Specialization could really benefit from some sort of first-class “code region over which these values are effectively constant” and then also “but the constant-ness is invalidated by this set of events.”

## Why Now?

The literature on dynamic specialization of OS code is interesting: it looks like there was a flurry of interest inspired by Synthesis in the mid/late 90s. [Many of these papers](https://www.cc.gatech.edu/~calton/publication.html) had Calton Pu, Massalin’s thesis supervisor, on the author list. Not a whole lot has happened in this area since then, as far as I know. The only paper I can think of about optimistic OS specialization is [this one](https://www.cc.gatech.edu/~calton/publications/sosp95.pdf); it’s a nice paper, I recommend it. Static OS specialization, on the other hand, is what unikernels are all about, so there’s been quite a bit of work done on this.

It seems like time to revive interest in dynamic OS specialization because:

- Most of the processor speed wins lately are application specific; the cores that execute OS code are not getting noticeably faster each year, nor do they seem likely to. In fact, way back in 1989 John Ousterhout argued that increases in processor speed [weren’t benefiting OS code as much as other kinds of code](https://www.hpl.hp.com/techreports/Compaq-DEC/WRL-TN-11.pdf).

- OSes have slowed down recently to mitigate side channel attacks. Maybe we can get some of that speed back using dynamic specialization.

- OSes are way bloatier than they were in the 90s, increasing the potential benefits due to specialization.

- Compiler technology is far ahead of where it was in the 90s, with off-the-shelf toolkits like LLVM providing high-quality solutions to many of the problems we’d run into while prototyping this work.

*I’d like to thank Perry Metzger who suggested this piece and also provided feedback on a draft of it. Perry worked with Alexia back in the day and hopefully he’ll also write about this topic.*

*Finally, I don’t want to give the impression that I’m summarizing a research proposal or an in-progress project. This is the kind of thing I love to think about, is all.*
