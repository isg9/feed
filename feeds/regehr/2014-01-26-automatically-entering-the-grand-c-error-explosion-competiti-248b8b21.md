---
title: Automatically Entering the Grand C++ Error Explosion Competition
url: https://blog.regehr.org/archives/1088
published: "2014-01-26T17:21:19Z"
feed: regehr
guid: http://blog.regehr.org/?p=1088
---

# Automatically Entering the Grand C++ Error Explosion Competition

G++ can be comically verbose; developers sometimes like to wallpaper their cubes with choice error messages from Boost or STL programs. [The Grand C++ Error Explosion Competition](http://tgceec.tumblr.com/post/70134083864/introducing-the-grand-c-error-explosion-competition) asks the question: how large can we make the ratio between error output and compiler input?

I’m not much of a C++ person but when the contest was announced I was doing some experiments in using [C-Reduce](http://embed.cs.utah.edu/creduce/) as way to search for C++ programs that have interesting properties. Of course, we usually use C-Reduce to search for small programs, but [Alex](http://eecs.oregonstate.edu/people/groce) and I have been using it (and other reducers) to find, for example, programs that cause interesting parts of the compiler to execute. It only took a minute or two to setup C-Reduce so that its goal was to maximize the GCEEC’s fitness function. I started it running on four C++ files; after a few days three of the reductions didn’t show signs of terminating but the fourth one — some random part of the LLVM backend — reduced to this:

```
struct x0 struct A<x0(x0(x0(x0(x0(x0(x0(x0(x0(x0(_T1,x0 (_T1> <_T1*, x0(_T1*_T2>
binary_function<_T1*, _T2, x0{ }

```

Somewhat surprisingly, there aren’t any templates here. When compiled using G++ 4.8.1 (I’m using the one that comes with Ubuntu 13.10 on x86-64) we get 5 MB of output. It wasn’t too hard to (1) clean up this output a bit and (2) recognize that the repeated `(x0` substring is important. Thus, my entry to the GCEEC was:

```
struct x struct z<x(x(x(x(x(x(x(x(x(x(x(x(x(x(x(x(x(x(x(x(x(x(y,x(y><y*,x(y*w>v<y*,w,x{}

```

Every added `(x` approximately doubles the size of the error output. It was tricky to choose the right number of these substrings to include since I wanted to bump up against the timeout without pushing past it. But really, at this point the competition became a lot less interesting because we can pick a target ratio of output to input and trivially craft an input that reaches the target (assuming we don’t run into implementation limits). So the contest is basically a bandwidth contest where the question is: How many bytes can we get out of G++ on the specified platform within the 5 minute timeout? At this point the winner depends on how many cores are available, the throughput of Linux pipes, etc., which isn’t too satisfying.

I was a little bummed because I didn’t need to use a trick I had been saving up, which was to give the C++ file a name that is 255 characters long — this is useful because the name of the source file is repeated many times in the error output (and the length of the source file name is not part of the fitness function). However, it was delightful to read the [other contest entries](http://tgceec.tumblr.com/post/74534916370/results-of-the-grand-c-error-explosion-competition) which used some nice tricks I wouldn’t have thought of.

Would it be fun to repeat this contest for Clang++ or MSVC++? Also, why is G++ so verbose? My guess is that its error reporting logic should be running (to whatever extent this is possible) much earlier in the compilation process, before templates and other things have been expanded. Also, it would probably be useful to enforce a limit on the length of any given error message printed by the compiler on the basis that nobody is interested in anything past the first 10 KB or whatever.
