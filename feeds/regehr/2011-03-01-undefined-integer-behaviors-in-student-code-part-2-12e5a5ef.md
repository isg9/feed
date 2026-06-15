---
title: Undefined Integer Behaviors in Student Code, Part 2
url: https://blog.regehr.org/archives/390
published: "2011-03-01T20:43:15Z"
feed: regehr
guid: http://blog.regehr.org/?p=390
---

# Undefined Integer Behaviors in Student Code, Part 2

*\[This post is based on data gathered by my student Peng Li. He also wrote the undefined behavior checker.\]*

The other day I posted about [undefined integer behaviors in code written by students](https://blog.regehr.org/archives/385) in a class I used to teach. This post is more of the same, this time from [CS 5785](http://www.eng.utah.edu/%7Ecs5785/), my advanced embedded systems course. Early in the course I often ask the students to implement saturating versions of signed and unsigned addition and subtraction. Their solutions are required to work regardless of whether an integer datatype is defined to be 8, 16, 32, or 64 bits long. Students are not given any kind of test suite, but I repeatedly emphasize that they should write lots of test cases and inspect the output. The assignment is graded by a tester I wrote that uses a combination of a few hundred values that lie near interesting boundaries, and a few thousand random values. The main reason I give this assignment is to get students warmed up a bit and to motivate some material later on in the course. The assignment is [here](http://www.eng.utah.edu/~cs5785/hw2/) (see part 2).

# Unsigned Saturating Operations

These functions are easy to get right, and also it’s not hard to avoid undefined behavior since the math is unsigned.

ADDcorrectwrongno undefined6614undefined13SUBTRACTcorrectwrongno undefined7410undefined00

# Signed Saturating Operations

Although the amount of code required to implement these is small (my solutions are 5 lines each), they’re not completely straightforward.

ADDcorrectwrongno undefined1510undefined2831SUBTRACTcorrectwrongno undefined114undefined2049

The undefined behaviors were a mix of shift errors and overflowing addition and subtraction. Many students used some sort of shift as part of computing the maximum / minimum representable integer.

# My Code Was Wrong Too

It turns out that my reference solutions (written years ago, before I understood the horror that is undefined behavior in C) contained two signed overflows. Ouch.

# Can We Avoid Undefined Behavior?

90% of the signed operations written by students prior to 2010 executed an undefined integer behavior. In Fall 2010 I included some [lecture material on undefined behavior](http://www.eng.utah.edu/~cs5785/slides/06-6up.pdf) and I also gave students access to Peng’s undefined behavior checker and told them to use it. The result? 46% of the 52 signed functions (written by the 26 students) executed an undefined integer behavior. This is a significant improvement, but still not very good. The real problem — judging from the large number of incorrect solutions — is that many students didn’t test their code very well. Next time I’ll encourage students to [graph the functions](https://blog.regehr.org/archives/392) implemented by their code; this makes some kinds of bugs obvious.
