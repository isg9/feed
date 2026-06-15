---
title: Anonymous Struct Types Removed
url: https://ziglang.org/devlog/2024/#2024-11-01
published: "2024-11-01T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2024/#2024-11-01
---

# Anonymous Struct Types Removed

November 01, 2024

# [Anonymous Struct Types Removed](\#2024-11-01)

Author: Matthew Lugg

Rejoice, ye faithful; the days of undocumented structural aggregate types are over!

Until recently, Zig had the notion of an “anonymous struct type”. When you wrote `.{ .x = 123 }` with no [result type](https://ziglang.org/documentation/master/#Result-Types), it would give this value an “anonymous struct” type, which is a special kind of struct which can coerce to other types based on *structure* rather than identity. In other words, this worked:

```zig
test "coerce anonymous struct" {
    const foo = .{ .x = 123 };
    const S = struct { x: ?u32 };
    // It's true that `@TypeOf(foo) != S`, and yet...
    const bar: S = foo; // ...this works!
    try std.testing.expect(bar.x.? == 123);
}
const std = @import("std");

```

A little bit of trivia: this existed because a long time ago, Zig didn’t actually have the concept of a “result type”, so *all* type annotations worked via coercions like that!

I [proposed removing this system](https://github.com/ziglang/zig/issues/16865) last year, and finally got around to [actually doing it](https://github.com/ziglang/zig/pull/21817) last week. You can still write `.{ .x = 123 }` without a result type, but it’s given a “normal” struct type, with no magical coercions allowed. So now, the above code example fails just like you’d expect:

```
example.zig:5:20: error: expected type 'example.test.coerce anonymous struct.S', found 'example.test.coerce anonymous struct__struct_90'
    const bar: S = foo; // ...this works!
                   ^~~
example.zig:2:18: note: struct declared here
    const foo = .{ .x = 123 };
                ~^~~~~~~~~~~~
example.zig:3:15: note: struct declared here
    const S = struct { x: ?u32 };
              ^~~~~~~~~~~~~~~~~~

```

Good riddance!
