---
title: How to Read and Write Other Process Memory
url: https://nullprogram.com/blog/2016/09/03/
published: "2016-09-03T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2016/09/03/
---

# How to Read and Write Other Process Memory

## [How to Read and Write Other Process Memory](/blog/2016/09/03/)

September 03, 2016

nullprogram.com/blog/2016/09/03/

I recently put together a little game memory cheat tool called [MemDig](https://github.com/skeeto/memdig). It can find the address of a particular game value (score, lives, gold, etc.) after being given that value at different points in time. With the address, it can then modify that value to whatever is desired.

I’ve been using tools like this going back 20 years, but I never tried to write one myself until now. There are many memory cheat tools to pick from these days, the most prominent being [Cheat Engine](http://www.cheatengine.org/). These tools use the platform’s debugging API, so of course any good debugger could do the same thing, though a debugger won’t be specialized appropriately (e.g. locating the particular address and locking its value).

My motivation was bypassing an in-app purchase in a single player Windows game. I wanted to convince the game I had made the purchase when, in fact, I hadn’t. Once I had it working successfully, I ported MemDig to Linux since I thought it would be interesting to compare. I’ll start with Windows for this article.

### Windows

Only three Win32 functions are needed, and you could almost guess at how it works.

- [OpenProcess()](https://msdn.microsoft.com/en-us/library/windows/desktop/ms684320)
- [ReadProcessMemory()](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680553)
- [WriteProcessMemory()](https://msdn.microsoft.com/en-us/library/windows/desktop/ms681674)

It’s very straightforward and, for this purpose, is probably the simplest API for any platform (see update).

As you probably guessed, you first need to open the process, given its process ID (integer). You’ll need to select the *desired access* bit a bit set. To read memory, you need the `PROCESS_VM_READ` and `PROCESS_QUERY_INFORMATION` rights. To write memory, you need the `PROCESS_VM_WRITE` and `PROCESS_VM_OPERATION` rights. Alternatively you could just ask for all rights with `PROCESS_ALL_ACCESS`, but I prefer to be precise.

```
DWORD access = PROCESS_VM_READ |
               PROCESS_QUERY_INFORMATION |
               PROCESS_VM_WRITE |
               PROCESS_VM_OPERATION;
HANDLE proc = OpenProcess(access, FALSE, pid);

```

And then to read or write:

```
void *addr; // target process address
SIZE_T written;
ReadProcessMemory(proc, addr, &value, sizeof(value), &written);
// or
WriteProcessMemory(proc, addr, &value, sizeof(value), &written);

```

Don’t forget to check the return value and verify `written`. Finally, don’t forget to [close it](https://msdn.microsoft.com/en-us/library/windows/desktop/ms724211) when you’re done.

```
CloseHandle(proc);

```

That’s all there is to it. For the full cheat tool you’d need to find the mapped regions of memory, via [VirtualQueryEx](https://msdn.microsoft.com/en-us/library/windows/desktop/aa366907). It’s not as simple, but I’ll leave that for another article.

### Linux

Unfortunately there’s no standard, cross-platform debugging API for unix-like systems. Most have a ptrace() system call, though each works a little differently. Note that ptrace() is not part of POSIX, but appeared in System V Release 4 (SVr4) and BSD, then copied elsewhere. The following will all be specific to Linux, though the procedure is similar on other unix-likes.

In typical Linux fashion, if it involves other processes, you use the standard file API on the /proc filesystem. Each process has a directory under /proc named as its process ID. In this directory is a virtual file called “mem”, which is a file view of that process’ entire address space, including unmapped regions.

```
char file[64];
sprintf(file, "/proc/%ld/mem", (long)pid);
int fd = open(file, O_RDWR);

```

The catch is that while you can open this file, you can’t actually read or write on that file without attaching to the process as a debugger. You’ll just get EIO errors. To attach, use ptrace() with `PTRACE_ATTACH`. This asynchronously delivers a `SIGSTOP` signal to the target, which has to be waited on with waitpid().

You could select the target address with [lseek()](http://pubs.opengroup.org/onlinepubs/9699919799/functions/lseek.html), but it’s cleaner and more efficient just to do it all in one system call with [pread()](http://pubs.opengroup.org/onlinepubs/9699919799/functions/read.html) and [pwrite()](http://pubs.opengroup.org/onlinepubs/9699919799/functions/write.html). I’ve left out the error checking, but the return value of each function should be checked:

```
ptrace(PTRACE_ATTACH, pid, 0, 0);
waitpid(pid, NULL, 0);

off_t addr = ...; // target process address
pread(fd, &value, sizeof(value), addr);
// or
pwrite(fd, &value, sizeof(value), addr);

ptrace(PTRACE_DETACH, pid, 0, 0);

```

The process will (and must) be stopped during this procedure, so do your reads/writes quickly and get out. The kernel will deliver the writes to the other process’ virtual memory.

Like before, don’t forget to close.

```
close(fd);

```

To find the mapped regions in the real cheat tool, you would read and parse the virtual text file /proc/ *pid*/maps. I don’t know if I’d call this stringly-typed method elegant — the kernel converts the data into string form and the caller immediately converts it right back — but that’s the official API.

Update: Konstantin Khlebnikov has pointed out the [process\_vm\_readv()](http://man7.org/linux/man-pages/man2/process_vm_readv.2.html) and [process\_vm\_writev()](http://man7.org/linux/man-pages/man2/process_vm_writev.2.html) system calls, available since Linux 3.2 (January 2012) and glibc 2.15 (March 2012). These system calls do not require ptrace(), nor does the remote process need to be stopped. They’re equivalent to ReadProcessMemory() and WriteProcessMemory(), except there’s no requirement to first “open” the process.

- [win32](/tags/win32/)
- [linux](/tags/linux/)
- [c](/tags/c/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20How%20to%20Read%20and%20Write%20Other%20Process%20Memory) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=How+to+Read+and+Write+Other+Process+Memory).

« [Two Years as a High School Mentor](/blog/2016/09/02/)

» [Inspecting C's qsort Through Animation](/blog/2016/09/05/)
