---
title: Non-deterministic Compilers?
url: https://blog.regehr.org/archives/294
published: "2010-11-05T03:48:23Z"
feed: regehr
guid: http://blog.regehr.org/?p=294
---

# Non-deterministic Compilers?

Compilers are supposed to be deterministic, and they generally are. However, when a compiler has memory safety bugs (use of free memory, usually) and runs on an OS with ASLR ( [address space layout randomization](http://en.wikipedia.org/wiki/Address_space_layout_randomization)) enabled, it can behave non-deterministically. Compile a file one time and the result is correct, compile the exact same file again and now the executable crashes or generates the wrong answer. This may sound like a silly case that happens only in theory, but while hunting for compiler bugs I’ve seen it happen a number of times.

The scary scenario is this:

1. Safety critical embedded code is compiled for testing, and happens to be compiled correctly.
2. Testing proceeds and finds no problems.
3. For whatever reason, the system is compiled again and this time the wrong code is generated.
4. Wrong code is deployed.

Unlikely? Sure! But so are a lot of other things people developing safety critical systems have to worry about. If I were developing safety critical code I’d turn off ASLR for development tools, it’s cheap insurance.
