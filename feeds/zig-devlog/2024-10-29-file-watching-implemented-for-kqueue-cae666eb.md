---
title: File Watching Implemented for kqueue
url: https://ziglang.org/devlog/2024/#2024-10-29
published: "2024-10-29T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2024/#2024-10-29
---

# File Watching Implemented for kqueue

October 29, 2024

# [File Watching Implemented for kqueue](\#2024-10-29)

Author: Andrew Kelley

I knocked out [#20599](https://github.com/ziglang/zig/issues/20599) on a plane ride last week (coincidentally on the way to a [KQ tournament](https://bumblebeargames.com/pages/bumblebash-5)), so if you’re tracking master branch, you can now use the `zig build --watch` feature on macOS, FreeBSD, OpenBSD, NetBSD, DragonFlyBSD, and Haiku.

This feature tracks all file system inputs, automatically repeating invalidated build steps. It can be quite handy when refactoring a codebase using a terminal-based workflow since it means seeing instant feedback when any build inputs - source code or otherwise - are updated.

KQueue allows placing a watch on an open directory handle, but the event does not indicate the name of the changed file within any given directory. This is no problem for the Zig Build System. If any directory that contains watched files are triggered, the respective steps are invalidated, and then those steps use fstat to determine if the cached artifacts are still valid.
