---
title: str2long Contest Results Part 1
url: https://blog.regehr.org/archives/914
published: "2013-03-20T05:03:11Z"
feed: regehr
guid: http://blog.regehr.org/?p=914
---

# str2long Contest Results Part 1

\[ **NOTE:** Reading this post only makes sense if you read and cared about [this previous post](https://blog.regehr.org/archives/909).\]

Ok, evaluating the submissions has been more work than I anticipated, and also things have gotten pretty busy at work, so I’m going to split the evaluation of the submissions into two parts. This first part will discuss objective metrics while the second will be subjective.

I received approximately 78 implementations of `str2long()`, including multiple submissions from a number of people. I say “approximately” since I deleted some submissions where the sender noticed it was buggy before I did. All submissions can be found [on Github here](https://github.com/regehr/str2long_contest). A total of 35 submissions pass all of my tests, which include many functional tests in addition to integer overflow checks using Clang’s `-fsanitize=integer`. My tests are reasonably good (I hope) but I still could have missed things, let me know if this is the case. I tested all submissions on an x86-64 Linux machine, targeting both 64-bit and 32-bit modes. The nastiest test case that I used is one that allocates a 9GB string that is mostly leading zeros; this caused a few submissions that passed all other tests to fail, of course because they used a 32-bit index somewhere. One might reasonably argue that this is a vanishingly unlikely use case, but it’s not prohibited by the specification so I tested for it.

Besides correctness, I also looked at code size and performance. Neither were explicit criteria for this contest, but all other things being equal faster code is better. Code size is mainly interesting because it serves as a very crude way to measure algorithmic complexity. In other words, code size answers the question: How much complexity remains after an optimizing compiler has looked at the function?

Since compilers are notoriously finicky about what they will and won’t optimize, I used a variety of compilers and report only the best result. The compilers are:

- GCC 4.4.7, 4.5.4, 4.6.3, 4.7.2, and a pre-4.8 snapshot (r196703, compiled from SVN a few days ago)

- Clang 3.0, 3.1, 3.2, and a pre-3.3 snapshot (r177225, compiled from SVN a few days ago)

- Intel CC 12.0.5 and 13.1.0

Each compiler targeted x86-64 and was invoked at `-O0`, `-O1`, `-O2`, `-Os`, and `-O3`. I’m aware there are other flags controlling code generation but I don’t have time to play those games. Code size was measured using the `size` command after passing the compiler flags `-fno-exceptions -fno-unwind-tables -fno-asynchronous-unwind-tables` which are necessary to prevent extraneous code from being generated and counted. With these preliminaries aside, here are the size results:

bytescompilersubmission 96 icc-13.0.1 -O1 libc109 gcc-snap -Os pascal110 gcc-4.6 -Os peter110 gcc-snap -Os ryan112 icc-13.1.0 -O1 bernd\_2115 gcc-snap -Os tordek\_2116 gcc-snap -Os davidl116 gcc-snap -Os davidl\_2122 clang-3.0 -Os yolanpa129 gcc-4.6 -Os daniel\_2129 gcc-snap -Os francois\_2131 gcc-snap -Os jeffrey136 gcc-4.4 -Os yang\_2137 gcc-snap -Os davide138 gcc-4.4 -Os david\_2139 gcc-snap -Os thomas145 gcc-snap -Os matthewf\_2145 clang-snap -Os chucky\_2148 gcc-4.4 -Os mikael\_2159 gcc-snap -Os greg160 gcc-snap -Os phil165 gcc-snap -Os mattias172 icc-13.1.0 -O1 stefan173 gcc-4.6 -Os matthew\_3175 gcc-4.4 -Os sidney176 icc-13.1.0 -O1 patrick182 gcc-snap -Os bastian185 gcc-4.7 -Os andrew203 clang-3.2 -Os till232 clang-3.1 -Os olivier251 gcc-snap -Os kevin275 gcc-snap -Os renaud276 gcc-4.6 -Os ben283 gcc-snap -Os ethan\_2312 clang-snap -Os mats

The `libc` entry doesn’t count, it’s just a call to `strtol` that has been wrapped with a bit of logic to make it conform to the `str2long` specification. Perhaps interestingly, no compiler consistently wins. Happily, `-Os` is the most popular compiler flag.

Performance was measured by taking the best out of 10 runs on an otherwise idle Core i7-2600. The metric is the average number of nanoseconds required to convert a string including whatever overhead is incurred by the test harness. As [Robert Graham observed](http://erratasec.blogspot.com/2013/03/code-philosophy-integer-overflows.html), the test cases are perhaps skewed too heavily towards invalid inputs, which would tend to penalize code optimized for the valid case.

Here are the results:

ns compiler submission21.4 icc-13.1.0 -O2 bernd\_2 21.6 icc-13.1.0 -Os peter 22.4 gcc-4.7 -O2 yang\_2 22.5 clang-3.0 -O1 ryan 22.6 gcc-4.7 -O2 till 23.8 icc-13.1.0 -O1 yolanpa 24.7 gcc-4.7 -O2 tordek\_2 25.3 clang-3.2 -Os andrew 25.6 gcc-4.4 -O3 davidl\_2 25.7 gcc-4.7 -O2 pascal 25.7 gcc-4.5 -O3 renaud 26.0 gcc-4.7 -O2 davidl 26.2 icc-12.0.5 -O3 stefan 26.8 gcc-snap -O3 bastian 27.1 gcc-4.7 -O2 chucky\_2 27.2 clang-3.1 -O3 olivier 27.5 gcc-snap -O3 mikael\_2 27.8 gcc-4.7 -O2 kevin 27.9 clang-3.2 -O2 davide 28.3 gcc-snap -O3 david\_2 28.5 clang-3.2 -O1 patrick 28.7 gcc-snap -O3 sidney 29.6 gcc-4.7 -O3 phil 32.1 gcc-snap -O3 greg 33.1 clang-3.1 -O2 ben 33.1 icc-12.0.5 -Os jeffrey 33.8 icc-13.1.0 -O2 mattias 34.5 gcc-snap -O2 thomas 34.6 clang-snap -O3 matthewf\_2 37.5 gcc-4.7 -O3 matthew\_3 49.0 clang-3.1 -O1 francois\_2 52.8 icc-12.0.5 -O2 libc 96.4 gcc-4.7 -Os daniel\_2103.3 gcc-4.4 -O3 mats128.3 gcc-4.4 -O2 ethan\_2

Again, no particular compiler wins, and this time no particular optimization flag consistently wins either. The `strtol` code from libc performs surprisingly well considering that it is implementing a slightly more generic specification than `str2long`.

These results would perhaps be improved by repeating them on a modern ARM platform and some good ARM compilers, but I don’t have those available right now.

This concludes the objective portion of the contest results. The followup will be subjective and will look at the strategies and idioms that people used to implement the `str2long` specification without overflowing integers.

Thanks to everyone who submitted code! This has been fun.
