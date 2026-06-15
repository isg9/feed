---
title: Linux System Calls, Error Numbers, and In-Band Signaling
url: https://nullprogram.com/blog/2016/09/23/
published: "2016-09-23T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2016/09/23/
---

# Linux System Calls, Error Numbers, and In-Band Signaling

## [Linux System Calls, Error Numbers, and In-Band Signaling](/blog/2016/09/23/)

September 23, 2016

nullprogram.com/blog/2016/09/23/

Today I got an e-mail asking about a previous article on [creating
threads on Linux using raw system calls](/blog/2015/05/15/) (specifically x86-64). The questioner was looking to use threads in a program without any libc dependency. However, he was concerned about checking for mmap(2) errors when allocating the thread’s stack. The [mmap(2) man
page](http://man7.org/linux/man-pages/man2/mmap.2.html) says it returns -1 (a.k.a. `MAP_FAILED`) on error and sets errno. But how do you check errno without libc?

As a reminder here’s what the (unoptimized) assembly looks like.

```
stack_create:
    mov rdi, 0
    mov rsi, STACK_SIZE
    mov rdx, PROT_WRITE | PROT_READ
    mov r10, MAP_ANONYMOUS | MAP_PRIVATE | MAP_GROWSDOWN
    mov rax, SYS_mmap
    syscall
    ret

```

As usual, the system call return value is in `rax`, which becomes the return value for `stack_create()`. Again, its C prototype would look like this:

```
void *stack_create(void);

```

If you were to, say, intentionally botch the arguments to force an error, you might notice that the system call isn’t returning -1, but other negative values. What gives?

The trick is that **errno is a C concept**. That’s why it’s documented as [errno(3)](http://man7.org/linux/man-pages/man3/errno.3.html) — the 3 means it belongs to C. Just think about how messy this thing is: it’s a thread-local value living in the application’s address space. The kernel rightfully has nothing to do with it. Instead, the mmap(2) wrapper in libc assigns errno (if needed) after the system call returns. This is how [*all* system calls
through libc work](http://man7.org/linux/man-pages/man2/intro.2.html), even with the [syscall(2)
wrapper](http://man7.org/linux/man-pages/man2/syscall.2.html).

So how does the kernel report the error? It’s an old-fashioned return value. If you have any doubts, take it straight from the horse’s mouth: [mm/mmap.c:do\_mmap()](http://lxr.free-electrons.com/source/mm/mmap.c?v=4.6#L1143). Here’s a sample of return statements.

```
if (!len)
        return -EINVAL;

/* Careful about overflows.. */
len = PAGE_ALIGN(len);
if (!len)
        return -ENOMEM;

/* offset overflow? */
if ((pgoff + (len >> PAGE_SHIFT)) < pgoff)
        return -EOVERFLOW;

/* Too many mappings? */
if (mm->map_count > sysctl_max_map_count)
        return -ENOMEM;

```

It’s returning the negated error number. Simple enough.

If you think about it a moment, you might notice a complication: This is a form of in-band signaling. On success, mmap(2) returns a memory address. All those negative error numbers are potentially addresses that a caller might want to map. How can we tell the difference?

1) None of the possible error numbers align on a page boundary, so they’re not actually valid return values. NULL *does* lie on a page boundary, which is one reason why it’s not used as an error return value for mmap(2). The other is that you might actually want to map NULL, for better [or worse](https://blogs.oracle.com/ksplice/entry/much_ado_about_null_exploiting1).

2) Those low negative values lie in a region of virtual memory reserved exclusively for the kernel (sometimes called “ [low
   memory](https://linux-mm.org/HighMemory)”). On x86-64, any address with the most significant bit set (i.e. the sign bit of a signed integer) is one of these addresses. Processes aren’t allowed to map these addresses, and so mmap(2) will never return such a value on success.

So what’s a clean, safe way to go about checking for error values? It’s a lot easier to read [musl](https://www.musl-libc.org/) than glibc, so let’s take a peek at how musl does it in its own mmap: [src/mman/mmap.c](https://git.musl-libc.org/cgit/musl/tree/src/mman/mmap.c?h=v1.1.15).

```
if (off & OFF_MASK) {
    errno = EINVAL;
    return MAP_FAILED;
}
if (len >= PTRDIFF_MAX) {
    errno = ENOMEM;
    return MAP_FAILED;
}
if (flags & MAP_FIXED) {
    __vm_wait();
}
return (void *)syscall(SYS_mmap, start, len, prot, flags, fd, off);

```

Hmm, it looks like its returning the result directly. What happened to setting errno? Well, syscall() is actually a macro that runs the result through \_\_syscall\_ret().

```
#define syscall(...) __syscall_ret(__syscall(__VA_ARGS__))

```

Looking a little deeper: [src/internal/syscall\_ret.c](https://git.musl-libc.org/cgit/musl/tree/src/internal/syscall_ret.c?h=v1.1.15).

```
long __syscall_ret(unsigned long r)
{
    if (r > -4096UL) {
        errno = -r;
        return -1;
    }
    return r;
}

```

Bingo. As documented, if the value falls within that “high” (unsigned) range of negative values for *any* system call, it’s an error number.

Getting back to the original question, we could employ this same check in the assembly code. However, since this is a anonymous memory map with a kernel-selected address, **there’s only one possible error:** **ENOMEM** (12). This error happens if the maximum number of memory maps has been reached, or if there’s no contiguous region available for the 4MB stack. The check will only need to test the result against -12.

- [linux](/tags/linux/)
- [x86](/tags/x86/)
- [c](/tags/c/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Linux%20System%20Calls,%20Error%20Numbers,%20and%20In-Band%20Signaling) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Linux+System+Calls%2C+Error+Numbers%2C+and+In-Band+Signaling).

« [Render the Mandelbrot Set with jq](/blog/2016/09/15/)

» [The Vulgarness of Abbreviated Function Templates](/blog/2016/10/02/)
