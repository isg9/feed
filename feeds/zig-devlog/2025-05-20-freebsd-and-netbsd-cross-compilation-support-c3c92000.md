---
title: FreeBSD and NetBSD Cross-Compilation Support
url: https://ziglang.org/devlog/2025/#2025-05-20
published: "2025-05-20T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2025/#2025-05-20
---

# FreeBSD and NetBSD Cross-Compilation Support

May 20, 2025

# [FreeBSD and NetBSD Cross-Compilation Support](\#2025-05-20)

Author: Alex Rønne Petersen

Pull requests [#23835](https://github.com/ziglang/zig/pull/23835) and [#23913](https://github.com/ziglang/zig/pull/23913) have now been merged. This means that, using `zig cc` or `zig build`, you can now build binaries targeting FreeBSD 14.0.0+ and NetBSD 10.1+ from any machine, just as you’ve been able to for Linux, macOS, and Windows for a long time now.

This builds on the [strategy](https://github.com/ziglang/libc-abi-tools) we were already using for glibc and will soon be using for other targets as well. For any given FreeBSD/NetBSD release, we build libc and related libraries for every supported target, and then extract public symbol information from the resulting ELF files. We then combine all that information into a very compact `abilists` file that gets shipped with Zig. Finally, when the user asks to link libc while cross-compiling, we load the `abilists` file and build a stub library for each constituent libc library ( `libc.so`, `libm.so`, etc), making sure that it accurately reflects the symbols provided by libc for the target architecture and OS version, and has the expected [soname](https://en.wikipedia.org/wiki/Soname). This is all quite similar to how the [llvm-ifs tool](https://llvm.org/docs/CommandGuide/llvm-ifs.html) works.

We currently import [crt0](https://en.wikipedia.org/wiki/Crt0) code from the latest known FreeBSD/NetBSD release and manually apply any patches needed to make it work with any OS version that we support cross-compilation to. This is necessary because the OS sometimes changes the crt0 ABI. We’d like to eventually [reimplement the crt0 code in Zig](https://github.com/ziglang/zig/issues/23875).

We also ship FreeBSD/NetBSD system and libc headers with the Zig compiler. Unlike the stub libraries we produce, however, we always import headers from the latest version of the OS. This is because it would be far too space-inefficient to ship separate headers for every OS version, and we realistically don’t have the time to audit the headers on every import and add appropriate version guards to all new declarations. The good news, though, is that we do accept patches to add version guards when necessary; we’ve already had many contributions of this sort in our imported glibc headers.

Please take this for a spin and report any bugs you find!

We would like to also add support for [OpenBSD libc](https://github.com/ziglang/zig/issues/2878) and [Dragonfly BSD libc](https://github.com/ziglang/zig/issues/23880), but because these BSDs cannot be conveniently cross-compiled from Linux, we need motivated users of them to chip in. Besides those, we are also looking into [SerenityOS](https://github.com/ziglang/zig/issues/23879), [Android](https://github.com/ziglang/zig/issues/23906), and [Fuchsia](https://github.com/ziglang/zig/issues/23877) libc support.
