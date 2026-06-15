---
title: Zig Bootstrap Porting Progress
url: https://ziglang.org/devlog/2024/#2024-11-19
published: "2024-11-19T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2024/#2024-11-19
---

# Zig Bootstrap Porting Progress

November 19, 2024

# [Zig Bootstrap Porting Progress](\#2024-11-19)

Author: Alex Rønne Petersen

The [zig-bootstrap](https://github.com/ziglang/zig-bootstrap) project makes it possible to produce a Zig compiler to run on (almost) any target with minimal system dependencies on the build machine. As part of my recent port work on Zig, I made it a goal to get zig-bootstrap working on as many of our supported cross-compilation target triples ( `zig targets | jq -r .libc[]`) as reasonably possible. I’m happy to report that this effort is now complete.

At time of writing, the list of target triples supported by zig-bootstrap is as follows:

- `aarch64-linux-(gnu,musl)`
- `aarch64-macos-none`
- `aarch64-windows-gnu`
- `aarch64_be-linux-(gnu,musl)`
- `arm[eb]-linux-(gnu,musl)eabi[hf]`
- `loongarch64-linux-(gnu,gnusf,musl)`
- `mips[el]-linux-(gnu,musl)eabi[hf]`
- `mips64[el]-linux-(gnu,musl)abi(64,n32)`
- `powerpc-linux-(gnu,musl)eabi[hf]`
- `powerpc64-linux-musl`
- `powerpc64le-linux-(gnu,musl)`
- `riscv(32,64)-linux-(gnu,musl)`
- `s390x-linux-(gnu,musl)`
- `thumb[eb]-linux-musleabi[hf]`
- `thumb-windows-gnu`
- `x86-linux-(gnu,musl)`
- `x86-windows-gnu`
- `x86_64-linux-(gnu,musl)[x32]`
- `x86_64-macos-none`
- `x86_64-windows-gnu`

[The latest list can be found here.](https://github.com/ziglang/zig-bootstrap/blob/master/README.md#supported-triples)

If you’ve been itching to run Zig on some unusual or obscure target, please take zig-bootstrap for a spin and see how it works for you. If you run into any build issues, please report those on the [zig-bootstrap](https://github.com/ziglang/zig-bootstrap) repository; issues with running the built compiler should be reported on [ziglang/zig](https://github.com/ziglang/zig).
