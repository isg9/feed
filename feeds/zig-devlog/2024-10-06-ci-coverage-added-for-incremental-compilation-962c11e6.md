---
title: CI Coverage Added for Incremental Compilation
url: https://ziglang.org/devlog/2024/#2024-10-06
published: "2024-10-06T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2024/#2024-10-06
---

# CI Coverage Added for Incremental Compilation

October 06, 2024

# [CI Coverage Added for Incremental Compilation](\#2024-10-06)

Author: Matthew Lugg

Over the last few months, myself and Jakub have been working hard on incremental compilation, where the compiler can “remember” parts of a previous build and only re-compile the code that changed. We just hit an important milestone here: **our CI test suite [now includes](https://github.com/ziglang/zig/pull/21518) our “incremental compilation” tests!** This set of tests is small right now, but will grow rapidly as we fix bugs and, eventually, fuzz test the implementation.

You can find all our incremental compilation test cases [here](https://github.com/ziglang/zig/tree/148b5b4c7806a694084f1e6f9514b0333cb75c6a/test/incremental). The CI runs all of them on `x86_64-linux` with the self-hosted x86\_64 backend, as well as on a few targets with the C backend ( `-ofmt=c`). It’s approaching the point where you *can* start to use incremental compilation in some very basic cases; for instance, I’ve recently managed to perform an incremental update on [Andrew’s Tetris clone](https://github.com/andrewrk/tetris).

If you’re using a `master` build of Zig on Linux on x86\_64, you can try playing with incremental compilation **right now** with this command:

```
zig build -fincremental --watch

```

(Make sure your `Step.Compile` has `.use_llvm = false, .use_lld = false`, so Zig doesn’t try to use LLVM!)

Beware, though, that you’ll run into bugs pretty quickly, including false positive compile errors and even miscompilations; use at your own risk.
