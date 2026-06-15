---
title: Undefined Integer Behaviors in Student Code, Part 1
url: https://blog.regehr.org/archives/385
published: "2011-02-17T22:20:33Z"
feed: regehr
guid: http://blog.regehr.org/?p=385
---

# Undefined Integer Behaviors in Student Code, Part 1

*\[This post is based on material from Chad Brubaker, a really smart CS undergrad at Utah who did all the work getting these data. The integer undefined behavior checker was created by my student Peng Li.\]*

[Integer undefined behaviors in C/C++](https://blog.regehr.org/archives/226), such as INT\_MAX+1 or 1<<-1, create interesting opportunities for compiler optimizations and they also make programming in C/C++ not unlike crossing a minefield while blindfolded. This piece describes the results of running a checker for integer undefined behaviors on code submitted by several years’ worth of students in Utah’s [CS 4400](http://www.eng.utah.edu/~cs4400/). The checker simply inserts checking logic in front of any potentially-undefined operation. Then, we ran each student’s code on the same inputs used by the test harness the students were given as part of the assignment. CS 4400 is based on [Computer Systems: A Programmer’s Perspective, by Bryant and O’Hallaron](http://csapp.cs.cmu.edu/). It’s great textbook and I think students get a lot out of the course. I taught it three times in the early/mid 2000s and others have taught it since then.

The assignment is to write C code solving various small “bit puzzles.” Each solution must be straight-line code and can only use C’s bitwise operators: `! ~ & ^ | + << >>`. There are a few additional restrictions that are specific to individual problems, such as limiting the number of operations and further restrictions on operator choice.

Students are given reference solutions for the bit puzzles (written in C, but using loops and otherwise not following the rules) and also they are given a harness that runs their code against the reference solutions on some fixed test vectors.

The results below divide students’ solutions into four bins:

- correct results and no undefined behavior
- wrong results (for at least one input) and no undefined behavior
- correct results but contains undefined behavior (for at least one input)
- wrong results and contains undefined behavior

The most interesting case is where code gives the right answers despite executing undefined behaviors. These solutions are basically just lucky. In other words, the version of GCC installed on our lab machines, at the optimization level specified by the makefile, happens to have a particular behavior. But even the same code, on the same compiler, using the same optimization options, could break if called from a different context.

# get\_least\_significant\_byte

Extract the least significant byte from an int.

correctwrongno undefined1050undefined00

# is\_equal

Return 1 if two ints are equal, and zero otherwise (note that == is not an allowed operator).

correctwrongno undefined970undefined80

All of the undefined behaviors were overflow of signed addition.

# any\_even\_one

Return 1 if any even bit of an int is set, otherwise return 0.

correctwrongno undefined1023undefined00

# clear\_all\_but\_lower\_bits

Clear all but the N least significant bits of an int.

correctwrongno undefined671undefined343

- 16 solutions contained signed addition overflows
- 4 contained a signed right-shift by negative or past bitwidth
- 17 contained a signed left-shift by negative or past bitwidth

# rotate\_right

Rotate the bits in an int to the right.

correctwrongno undefined73undefined878

- 2 overflowed a signed addition
- 8 contained an unsigned left-shift by negative or past bitwidth
- 7 contained an unsigned right-shift by negative or past bitwidth
- 8 contained a signed right-shift by negative or past bitwidth
- 70 contained a signed left-shift by negative or past bitwidth

# indicate\_rightmost\_one

Generate an int with a single set bit corresponding to the rightmost set bit in another int.

correctwrongno undefined111undefined930

93 solutions overflowed a signed addition — only a single student managed to avoid this problem and also return the right answers

# divide\_by\_four

Divide an int by four, rounding towards zero.

correctwrongno undefined948undefined03

3 solutions overflowed a signed addition

# fits\_n\_bits

Return 1 if a value can be represented as an N-bit 2’s complement integer.

correctwrongno undefined7912undefined104

- 9 solutions overflowed a signed addition
- 2 solutions contained a signed right-shift by negative or past bitwidth
- 3 solutions contained a signed left-shift by negative or past bitwidth

This puzzle was interesting because it was the only one where we found undefined behavior in the reference solution, which executes this code to compute the largest representable integer:

> ```
> tmax_n = (1<<(n-1)) - 1;
> ```

1<<31 evaluates to 0x80000000, which is INT\_MIN on a machine where int is 32 bits. Then, evaluating INT\_MIN-1 is an operation with undefined behavior.

Moreover, in C99, evaluating 1<<31 is undefined.

# is\_greater

Return 1 if one int is larger than another, and zero otherwise.

correctwrongno undefined411undefined846

90 solutions overflowed a signed addition

# subtract\_overflows

Return 1 if subtracting one int from another overflows, and zero otherwise.

correctwrongno undefined013undefined893

92 solutions overflowed a signed addition — nobody avoided this hazard

# What’s the Problem?

The problem is that a large fraction of our junior-level CS undergraduates are producing code that executes integer undefined behaviors. At best, this kind of code is not portable across compilers, platforms, or even changes in compiler version, optimization options, or calling context. At worst, undefined behaviors are time bombs that will lead to exploitable vulnerabilities or other serious malfunctions in software systems. I want our graduates to understand what undefined behavior means and how to avoid it, not because C/C++ programming is so great, but because the concepts are much more broadly applicable.

# What’s the Solution?

Fixing the problem in the context of Utah’s CS 4400 course should be straightforward:

- Include some lecture material on undefined behaviors
- Make Peng’s undefined behavior checker part of the test suite that students are given as part of the assignment; since it gives nice clear error messages, the output should be directly actionable

I’ll see if Erin, the current instructor, is interested in working with me to implement this for Fall 2011. We’ll see how it goes.

*\[Note: I’ve deliberately left code solving the bit puzzles out of this post, and I will reject any comments containing solutions.\]*
