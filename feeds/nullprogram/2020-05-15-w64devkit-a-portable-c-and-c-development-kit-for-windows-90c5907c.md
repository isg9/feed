---
title: 'w64devkit: a Portable C and C++ Development Kit for Windows'
url: https://nullprogram.com/blog/2020/05/15/
published: "2020-05-15T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2020/05/15/
---

# w64devkit: a Portable C and C++ Development Kit for Windows

## [w64devkit: a Portable C and C++ Development Kit for Windows](/blog/2020/05/15/)

May 15, 2020

nullprogram.com/blog/2020/05/15/

*This article was discussed [on Hacker News](https://news.ycombinator.com/item?id=23292161).*

As a computer engineer, my job is to use computers to solve important problems. Ideally my solutions will be efficient, and typically that means making the best use of the resources at hand. Quite often these resources are machines running Windows and, despite my misgivings about the platform, there is much to be gained by properly and effectively leveraging it.

Sometimes [targeting Windows while working from another platform](/blog/2018/11/15/) is sufficient, but other times I must work on the platform itself. There [are various options available](/blog/2016/06/13/) for C development, and I’ve finally formalized my own development kit: [**w64devkit**](https://github.com/skeeto/w64devkit).

For most users, the value is in the **78MiB .zip** available in the “Releases” on GitHub. This (relatively) small package includes a state-of-the-art C and C++ compiler ( [latest GCC](http://mingw-w64.org/)), a [powerful
text editor](https://www.vim.org/), [debugger](https://www.gnu.org/software/gdb/), a [complete x86 assembler](https://www.nasm.us/), and [miniature unix environment](https://frippery.org/busybox/). It’s “portable” in that there’s no installation. Just unzip it and start using it in place. With w64devkit, it literally takes a few seconds on any Windows to get up and running with a fully-featured, fully-equipped, first-class [development
environment](https://sanctum.geek.nz/arabesque/unix-as-ide-introduction/).

The development kit is cross-compiled entirely from source using Docker, though Docker is not needed to actually use it. The repository is just a Dockerfile and some documentation. The only build dependency is Docker itself. It’s also easy to customize it for your own personal use, or to audit and build your own if, for whatever reason, you didn’t trust my distribution. This is in stark contrast to Windows builds of most open source software where the build process is typically undocumented, under-documented, obtuse, or very complicated.

### From script to Docker

Publishing this is not necessarily a commitment to always keep w64devkit up to date, but this Dockerfile *is* derived from (and replaces) a shell script I’ve been using continuously [for over two years now](/blog/2018/04/13/#a-better-alternative). In this period, every time GCC has made a release, I’ve built myself a new development kit, so I’m already in the habit.

I’ve been using Docker on and off for about 18 months now. It’s an oddball in that it’s something I learned on the job rather than my own time. I formed an early impression that still basically holds: **The** **main purpose of Docker is to contain and isolate misbehaved software to** **improve its reliability**. Well-behaved, well-designed software benefits little from containers.

My unusual application of Docker here is no exception. [Most software
builds are needlessly complicated and fragile](/blog/2017/03/30/), especially Autoconf-based builds. Ironically, the worst configure scripts I’ve dealt with come from GNU projects. They waste time on superfluous checks (“Does your compiler define `size_t`?”) then produce a build that doesn’t work anyway because you’re doing something slightly unusual. Worst of all, despite my best efforts, the build will be contaminated by the state of the system doing the build.

My original build script was fragile by extension. It would work on one system, but not another due to some subtle environment change — a slightly different system header that reveals a build system bug ( [example in GCC](https://gcc.gnu.org/legacy-ml/gcc/2017-05/msg00219.html)), or the system doesn’t have a file at a certain hard-coded absolute path that shouldn’t be hard-coded. Converting my script to a Dockerfile locks these problems in place and makes builds much more reliable and repeatable. The misbehavior is contained and isolated by Docker.

Unfortunately it’s not *completely* contained. In each case I use make’s `-j` option to parallelize the build since otherwise it would take hours. Some of the builds have subtle race conditions, and some bad luck in timing can cause a build to fail. Docker is good about picking up where it left off, so it’s just a matter of trying again.

In one case a build failed because Bison and flex were not installed even though they’re not normally needed. Some dependency isn’t expressed correctly, and unlucky ordering leads to an unused `.y` file having the wrong timestamp. Ugh. I’ve had this happen a lot more in Docker than out, probably because file system operations are slow inside Docker and it creates greater timing variance.

### Other tools

The README explains some of my decisions, but I’ll summarize a few here:

- Git. Important and useful, so I’d love to have it. But it has a weird installation (many [.zip-unfriendly symlinks](https://github.com/skeeto/w64devkit/issues/1)) tightly-coupled with msys2, and its build system does not support cross-compilation. I’d love to see a clean, straightforward rewrite of Git in a single, appropriate implementation language. Imagine installing the latest Git with `go get git-scm.com/git`. ( *Update*: [libgit2 is working on
  it](https://github.com/libgit2/libgit2/pull/5507)!)

- Bash. It’s a much nicer interactive shell than BusyBox-w32 `ash`. But the build system doesn’t support cross-compilation, and I’m not sure it supports Windows without some sort of compatibility layer anyway.

- Emacs. Another powerful editor. But the build system doesn’t support cross-compilation. It’s also *way* too big.

- Go. Tempting to toss it in, but [Go already does this all correctly
  and effectively](/blog/2020/01/21/). It simply doesn’t require a specialized distribution. It’s trivial to manage a complete Go toolchain with nothing but Go itself on any system. People may say its language design comes from the 1970s, but the tooling is decades ahead of everyone else.

### Alternatives

For a long, long time Cygwin filled this role for me. However, I never liked its bulky nature, the complete opposite of portable. Cygwin processes always felt second-class on Windows, particularly in that it has its own view of the file system compared to other Windows processes. They could never fully cooperate. I also don’t like that there’s no toolchain for cross-compiling with Cygwin as a target — e.g. compile Cygwin binaries from Linux. Finally [it’s been essentially obsoleted by
WSL](/blog/2017/11/30/) which matches or surpasses it on every front.

There’s msys and [msys2](https://www.msys2.org/), which are a bit lighter. However, I’m still in an isolated, second-class environment with weird path translation issues. These tools *do* have important uses, and it’s the only way to compile most open source software natively on Windows. For those builds that don’t support cross-compilation, it’s *the* only path for producing Windows builds. It’s just not what I’m looking for when developing my own software.

*Update*: [llvm-mingw](https://github.com/mstorsjo/llvm-mingw) is an eerily similar project using Docker the same way, but instead builds LLVM.

### Using Docker for other builds

I also [converted my GnuPG build script](https://github.com/skeeto/gnupg-windows-build) to a Dockerfile. Of course I don’t plan to actually *use* GnuPG on Windows. I just need it [for passphrase2pgp](/blog/2019/07/10/), which I test against GnuPG. This tests the Windows build.

In the future I may extend this idea to a few other tools I don’t intend to include with w64devkit. If you have something in mind, you could use my Dockerfiles as a kind of starter template.

- [c](/tags/c/)
- [cpp](/tags/cpp/)
- [win32](/tags/win32/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20w64devkit:%20a%20Portable%20C%20and%20C++%20Development%20Kit%20for%20Windows) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=w64devkit%3A+a+Portable+C+and+C%2B%2B+Development+Kit+for+Windows).

« [How to Read UTF-8 Passwords on the Windows Console](/blog/2020/05/04/)

» [Latency in Asynchronous Python](/blog/2020/05/24/)
