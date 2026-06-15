---
title: Differential Whitebox Testing Is Good
url: https://blog.regehr.org/archives/467
published: "2011-02-26T05:58:37Z"
feed: regehr
guid: http://blog.regehr.org/?p=467
---

# Differential Whitebox Testing Is Good

*\[This post is based on data gathered by Chad Brubaker and Peng Li, respectively an undergrad and grad student in CS at Utah.\]*

Two courses I’ve taught at Utah, [CS 4400](http://www.eng.utah.edu/~cs4400/) and [CS 5785](http://www.eng.utah.edu/~cs5785/), have assignments where students write short integer functions for which we — the instructors — have automatic graders. In 4400 the students get access to the test framework while they work, in 5785 they do not.

[Klee](http://klee.llvm.org/) is a whitebox test case generation tool: it looks at a piece of code and attempts to generate a collection of inputs that cover all possible execution paths. I was curious how Klee would fare when compared to the hand-written testers, both of which have been in use for more than five years.

One way to use Klee is like this:

1. Klee generates test cases for a piece of code written by student
2. We run the test cases through the student code and the reference code, looking for inequal output

ï»¿This works poorly: Klee fails to find lots of bugs found by the hand-written testers. Klee can also be used to perform differential whitebox test case generation (which I already briefly discussed [here](https://blog.regehr.org/archives/386)):

1. Klee generates test cases for the code `assert(student_code(input)==reference_code(input))`
2. We run the test cases through the student code and the reference code, looking for inequal output

This works very well: Klee’s tests not only find all bugs found by the hand-written testers, but also some new bugs are identified. What appears to be going on is that the hand-written testers use values that the instructors’ intuitions indicate would be good for triggering bugs. But students do not always write code corresponding to the instructors’ intuitions; in that case, paths are missed by the hand-written tester whereas Klee goes ahead and thoroughly tests them.

# Results

From CS 4400 we have 1050 bit puzzle solutions from students. 84 buggy functions are identified by the hand-written test suite. Differential test case generation using Klee finds all of these bugs plus seven more, for a total of 91.

From CS 5785 we have 336 saturating operations from students. 115 buggy functions are identified by the hand-written test suite. Differential test case generation using Klee finds all of these bugs plus six more, for a total of 121.

# Differential Klee vs. Exhaustive Testing

Klee can find more bugs, but can it find all of them? To explore this, we exhaustively tested each function that has â‰¤32 bits of input.

For the bit puzzles, exhaustive testing revealed some bugs not found by Klee. I’ll provide details in a subsequent post.

For the saturating operations, exhaustive testing revealed a single bug not found be Klee. The problem turned out to be code that computed the maximum and minimum integer values like this:

> ```
> int type = 8*sizeof(signed char);
> min = -(pow(2,type)/2);
> max = min+1;
> max = 0-max;
> ```

Clang/LLVM fails to optimize away the pow() call, which remains in the bitcode where (for reasons I haven’t looked into) it prevents Klee from generating good test cases. Klee properly found the bug in this function when the code was changed to this:

> ```
> min = CHAR_MIN;
> max = CHAR_MAX;
>
> ```

The student couldn’t have written the code this way, however, since their math operations had to work for four different integer widths.

# Conclusion

A 5%-8% increase in the number of buggy functions identified is pretty significant. Also, by using Klee we no longer need to write a test case generator by hand (although I still would, since I wouldn’t necessarily trust Klee to do a good job without some supporting evidence).
