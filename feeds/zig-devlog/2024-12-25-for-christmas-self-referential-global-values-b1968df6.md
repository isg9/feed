---
title: 'For Christmas: Self-Referential Global Values'
url: https://ziglang.org/devlog/2024/#2024-12-25
published: "2024-12-25T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2024/#2024-12-25
---

# For Christmas: Self-Referential Global Values

December 25, 2024

# [For Christmas: Self-Referential Global Values](\#2024-12-25)

Author: Matthew Lugg

I recently took it upon myself to tackle [issue #131](https://github.com/ziglang/zig/issues/131). This has been a long-standing bug in the compiler, dating way back to the earliest days of the C++ bootstrap compiler: it was the 9th oldest open issue in the Zig repository! Essentially, fixing it allows you to write code like this:

```zig
const S = struct { ptr: *const S, x: u32 };

// `foo` contains a pointer to itself
const foo: S = .{ .ptr = &foo, .x = 123 };

// Or:

// `a` contains a pointer to `b`, and...
// `b` contains a pointer to `a`
const a: S = .{ .ptr = &b, .x = 123 };
const b: S = .{ .ptr = &a, .x = 456 };

```

Previously, both of the examples above would have failed with a “dependency loop detected” error; now, they compile just as you expect (and the same works if they’re declared `var`, of course).

This issue had been open for so long because it actually requires a non-trivial extension to the compiler: separating semantic analysis for the *type* and *value* of container-level declarations. This is complicated further by the fact that some global declarations might not have a type annotation, and that some (such as `extern` declarations) have a *type* but no *value*. It also interacts with the work-in-progress incremental compilation feature in non-trivial ways, so we had to make sure we got that right. All in all, this resulted in a [pretty bulky diff](https://github.com/ziglang/zig/pull/22303) to the compiler.

Regardless, the point is, this now works on Zig `master`! If you have a use case for it, feel free to give it a try – and please do open an issue if you encounter any problems along the way.

Merry Christmas to those who celebrate!
