---
title: My new debugbreak command
url: https://nullprogram.com/blog/2022/07/31/
published: "2022-07-31T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2022/07/31/
---

# My new debugbreak command

## [My new debugbreak command](/blog/2022/07/31/)

July 31, 2022

nullprogram.com/blog/2022/07/31/

I [previously mentioned](/blog/2022/06/26/) the Windows feature where [pressing
F12](https://docs.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-registerhotkey) in a debuggee window causes it to break in the debugger. It works with any debugger — GDB, RemedyBG, Visual Studio, etc. — since the hotkey simply raises a breakpoint [structured exception](https://docs.microsoft.com/en-us/cpp/cpp/structured-exception-handling-c-cpp). It’s been surprisingly useful, and I’ve wanted it available in more contexts, such as console programs or even on Linux. The result is a new [`debugbreak`
command](https://github.com/skeeto/w64devkit/blob/4282797/src/debugbreak.c), now included in [w64devkit](/blog/2020/05/15/). Though, of course, you already have [everything you need](/blog/2020/09/25/) to build it and try it out right now. I’ve also worked out a Linux implementation.

It’s named after an [MSVC intrinsic and Win32 function](https://docs.microsoft.com/en-us/visualstudio/debugger/debugbreak-and-debugbreak). It takes no arguments, and its operation is indiscriminate: It raises a breakpoint exception in *all* debuggee processes system-wide. Reckless? Perhaps, but certainly convenient. You don’t need to tell it which process you want to pause. It just works, and a good debugging experience is one of ease and convenience.

The linchpin is [DebugBreakProcess](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-debugbreakprocess). The command walks the process list and fires this function at each process. Nothing happens for programs without a debugger attached, so it doesn’t even bother checking if it’s a debuggee. It couldn’t be simpler. I’ve used it on everything from Windows XP to Windows 11, and it’s worked flawlessly.

```
HANDLE s = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
PROCESSENTRY32W p = {sizeof(p)};
for (BOOL r = Process32FirstW(s, &p); r; r = Process32NextW(s, &p)) {
    HANDLE h = OpenProcess(PROCESS_ALL_ACCESS, 0, p.th32ProcessID);
    if (h) {
        DebugBreakProcess(h);
        CloseHandle(h);
    }
}

```

I use it almost exclusively from Vim, where I’ve given it a [leader
mapping](https://learnvimscriptthehardway.stevelosh.com/chapters/06.html). With the editor focused, I can type backslash then `d` to pause the debuggee.

```
map <leader>d :call system("debugbreak")<cr>

```

With the debuggee paused, I’m free to add new breakpoints or watchpoints, or print the call stack to see what the heck it’s busy doing. The mechanism behind DebugBreakProcess is to create a new thread in the target, with that thread raising the breakpoint exception. The debugger will be stopped in this new thread. In GDB you can use the `thread` command to switch over to the thread that actually matters, usually `thr 1`.

### debugbreak on Linux

On unix-like systems the equivalent of a breakpoint exception is a `SIGTRAP`. There’s already a standard command for sending signals, [`kill`](https://man7.org/linux/man-pages/man1/kill.1.html), so a `debugbreak` command can be built using nothing more than a few lines of shell script. However, unlike DebugBreakProcess, signaling every process with `SIGTRAP` will only end in tears. The script will need a way to determine which processes are debuggees.

Linux exposes processes in the file system as virtual files under `/proc`, where each process appears as a directory. Its `status` file includes a `TracerPid` field, which will be non-zero for debuggees. The script inspects this field, and if non-zero sends a `SIGTRAP`.

```
#!/bin/sh
set -e
for pid in $(find /proc -maxdepth 1 -printf '%f\n' | grep '^[0-9]\+$'); do
    grep -q '^TracerPid:\s[^0]' /proc/$pid/status 2>/dev/null &&
        kill -TRAP $pid
done

```

This script, now part of [my dotfiles](/blog/2012/06/23/), has worked very well so far, and effectively smoothes over some debugging differences between Windows and Linux, reducing my context switching mental load. There’s probably a better way to express this script, but that’s the best I could do so far. On the BSDs you’d need to parse the output of `ps`, though each system seems to do its own thing for distinguishing debuggees.

### A missing feature

I had originally planned for one flag, `-k`. Rather than breakpoint debugees, it would terminate all debuggee processes. This is especially important on Windows where debuggee processes block builds due to file locking shenanigans. I’d just run `debugbreak -k` as part of the build. However, it’s not possible to terminate debuggees paused in the debugger — the common situation. I’ve given up on this for now.

- [c](/tags/c/)
- [cpp](/tags/cpp/)
- [win32](/tags/win32/)
- [linux](/tags/linux/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20My%20new%20debugbreak%20command) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=My+new+debugbreak+command).

« [Assertions should be more debugger-oriented](/blog/2022/06/26/)

» [The quick and practical "MSI" hash table](/blog/2022/08/08/)
