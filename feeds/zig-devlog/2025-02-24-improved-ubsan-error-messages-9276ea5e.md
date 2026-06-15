---
title: Improved UBSan Error Messages
url: https://ziglang.org/devlog/2025/#2025-02-24
published: "2025-02-24T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2025/#2025-02-24
---

# Improved UBSan Error Messages

February 24, 2025

# [Improved UBSan Error Messages](\#2025-02-24)

Author: David Rubin

Lately, I’ve been extensively working with C interop, and one thing that’s been sorely missing is clear error messages from UBSan. When compiling C with `zig cc`, Zig provides better defaults, including implicitly enabling `-fsanitize=undefined`. This has been great for catching subtle bugs and makes working with C more bearable. However, due to the lack of a UBSan runtime, all undefined behavior was previously caught with a `trap` instruction.

For example, consider this example C program:

```c
#include <stdio.h>

int foo(int x, int y) {
    return x + y;
}

int main() {
    int result = foo(0x7fffffff, 0x7fffffff);
    printf("%d\n", result);
}

```

Running this with `zig cc` used to result in an unhelpful error:

```
$ zig run test.c -lc
fish: Job 1, 'zig run empty.c -lc' terminated by signal SIGILL (Illegal instruction)

```

Not exactly informative! To understand what went wrong, you’d have to run the executable in a debugger. Even then, tracking down the root cause could be daunting. Many newcomers ran into this `Illegal instruction` error without realizing that UBSan was enabled by default, leading to confusion. This issue was common enough to warrant a dedicated [Wiki page](https://github.com/ziglang/zig/wiki/zig-cc-compatibility-with-clang#ubsan-and-sigill-illegal-instruction).

With the new [UBSan runtime merged](https://github.com/ziglang/zig/pull/22488), the experience has completely changed. Now instead of an obscure `SIGILL`, you get a much more helpful error message:

```
$ zig run test.c -lc
thread 208135 panic: signed integer overflow: 2147483647 + 2147483647 cannot be represented in type 'int'
/home/david/Code/zig/build/test.c:4:14: 0x1013e41 in foo (test.c)
    return x + y;
             ^
/home/david/Code/zig/build/test.c:8:18: 0x1013e63 in main (test.c)
    int result = foo(0x7fffffff, 0x7fffffff);
                 ^
../sysdeps/nptl/libc_start_call_main.h:58:16: 0x7fca4c42e1c9 in __libc_start_call_main (../sysdeps/x86/libc-start.c)
../csu/libc-start.c:360:3: 0x7fca4c42e28a in __libc_start_main_impl (../sysdeps/x86/libc-start.c)
???:?:?: 0x1013de4 in ??? (???)
???:?:?: 0x0 in ??? (???)
fish: Job 1, 'zig run test.c -lc' terminated by signal SIGABRT (Abort)

```

Now, not only do we see *what* went wrong (signed integer overflow), but we also see *where* it happened – two critical pieces of information that were previously missing.

## Remaining Limitations

While the new runtime vastly improves debugging, there are still two features that LLVM’s UBSan runtime provides which ours doesn’t support yet:

1. In C++, UBSan can detect when an object’s vptr indicates the wrong dynamic type or when its lifetime hasn’t started. Supporting this would require replicating the Itanium C++ ABI, which isn’t worth the extreme complexity.
2. Currently, the runtime doesn’t show the exact locations of attributes like `assume_aligned` and `__nonnull`. This should be relatively straightforward to add, and contributions are welcome!

If you’ve ever been frustrated by cryptic `SIGILL` errors while trying out Zig, this update should make debugging undefined behavior a lot easier!
