---
title: Counting Compiler Crashes
url: https://blog.regehr.org/archives/295
published: "2010-11-05T17:50:53Z"
feed: regehr
guid: http://blog.regehr.org/?p=295
---

# Counting Compiler Crashes

This is a bit of material from my GCC Summit talk that I thought was worth bringing out in a separate post.

As an experiment, we compiled 1,000,000 randomly generated C programs using GCC 3.\[0-4\].0, GCC 4.\[0-5\].0, LLVM-GCC 1.9, LLVM-GCC 2.\[0-8\], and Clang 2.\[6-8\]. All compilers targeted x86 and were given the -O0, -O1, -O2, -Os, and -O3 options. We then asked the question: How many ways is each compiler crashed? We count crash bugs by looking for unique “assertion failure” strings in LLVM output and “internal compiler error” strings in GCC output. This is conservative because typically a compiler will also have a number of crashes due to null pointer dereferences and other memory safety violations, and we don’t try to tell these apart. Here are the three crash messages we got from Clang 2.8:

> Statement.cpp:944: void Statement::post\_creation\_analysis(std::vector<const Fact\*, std::allocator<const Fact\*> >&, CGContext&) const: Assertion \`0′ failed.
>
> StatementIf.cpp:81: static StatementIf\* StatementIf::make\_random(CGContext&): Assertion \`ok’ failed.
>
> Block.cpp:512: bool Block::find\_fixed\_point(std::vector<const Fact\*, std::allocator<const Fact\*> >, std::vector<const Fact\*, std::allocator<const Fact\*> >&, CGContext&, int&, bool) const: Assertion \`0′ failed.

Here are the GCC results:

![](https://blog.regehr.org/wp-content/uploads/2010/11/gcc_unique_crash.png)

And here are the LLVM-GCC results:

![](https://blog.regehr.org/wp-content/uploads/2010/11/llvm_unique_crash.png)

The thing I really like about these results is the way it shows that the latest versions of GCC and LLVM are really solid. Both teams have put a tremendous amount of work into their tools.

The “bugs fixed” annotations in the graphs refer to fixes to bugs that we found (using random testing) and reported. We had hoped to establish some sort of nice causal link between fixing these bugs and improving the resistance of compilers to crashing, but these graphs are a long way from showing anything like that. There is just a lot going on in each of these projects — the experiment is too uncontrolled to give us any kind of solid evidence.

Are compiler crashes good or bad? Certainly they’re a pain when you’re just trying to get work done, but overall they’re good. First, they can almost always be worked around by changing the optimization level. Second, a crash is the compiler’s way of failing fast, which is good. If that assertion hadn’t been there, it’s entirely possible the compiler would have run to completion, generating incorrect code.

**Update from 11/6/2010:** Here are the graphs showing rate of crashing, as opposed to number of distinct crash bugs, that Chris asked about in a comment.

![](https://blog.regehr.org/wp-content/uploads/2010/11/gcc_crash_log.png)

![](https://blog.regehr.org/wp-content/uploads/2010/11/llvm_crash_log.png)

Although it’s not totally clear what to read into these numbers, they do seem to tell a good story. GCC steadily improves during the 3.x series, regresses following the major changes that went into 4.0.0, then (more or less) steadily improves again. Similarly, LLVM starts off with a rather high rate of crashes and improves over time. Both compilers are exceptionally solid in their latest releases (note the log scale: both compilers have reduced their crash rates by at least 3 orders of magnitude).
