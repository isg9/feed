---
title: no-matrix build option
url: https://ziglang.org/devlog/2025/#2025-10-09
published: "2025-10-09T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2025/#2025-10-09
---

# no-matrix build option

October 09, 2025

# [no-matrix build option](\#2025-10-09)

Author: Andrew Kelley

I added a new build option, `-Dno-matrix`, which makes it so that using `zig build test-std` or `zig build test-behavior` it only runs the default, native target tests. This is handy when you already know it’s going to be in a failing state for a while and you’re working on a big change.

In particular, it combines with `--watch` and `-fincremental` to make the development loop instant. This is huge for me since I work on the standard library often. Previously, I would run `zig test lib/std.zig` in order to get this effect, but since it takes a few seconds to compile, I would try to fix all the errors before rerunning the command. This worked okay to amortize the cost, but now I just press save in vim and the new errors are literally on my screen in the blink of an eye.

So here’s the full build command I’ve been enjoying:

```
zig build test-std -Dno-matrix --watch -fincremental

```
