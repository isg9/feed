---
title: A Fun Zig Program
url: https://matklad.github.io/2025/04/21/fun-zig-program.html
published: "2025-04-21T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2025/04/21/fun-zig-program.html
---

# A Fun Zig Program

Apr 21, 2025

This short example is worth pondering a bit if you are learning Zig:

```
fn f(comptime x: bool) if (x) u32 else bool {
    return if (x) 0 else false;
}

const print = @import("std").debug.print;
pub fn main() void {
    print("{} {}", .{f(false), f(true)});
}
```

It is curious in three ways:

- You can write arbitrary expressions in type position.

- In functions, type expressions can use preceding `comptime` parameters.

- Functions *must* specify the signature, there’s no type inference, no `-> auto`. When a generic function is called, Zig first computes the signature. While doing so, it ignores function’s body. Afterwards, the compiler separately checks that the signature is correct for the call site, and that the body is correct for the signature.
