---
title: Better Testing With Undefined Behavior Coverage
url: https://blog.regehr.org/archives/388
published: "2011-03-03T04:32:31Z"
feed: regehr
guid: http://blog.regehr.org/?p=388
---

# Better Testing With Undefined Behavior Coverage

*\[The bit puzzle results are based on data from Chad Brubaker and the saturating operation results are based on data from Peng Li. They are respectively an undergrad and a grad student in Utah’s CS program.\]*

[Klee](http://klee.llvm.org/) is a tool that attempts to generate a collection of test cases inducing *path coverage* on a system under test. Path coverage means that all feasible control flow paths are executed. It is a strong kind of coverage, but still [misses bugs](https://blog.regehr.org/archives/386). One way to improve Klee would be to add support for different kinds of coverage metrics: weaker ones like statement coverage would scale to larger programs, and stronger ones such as boundary-value coverage would find more bugs in small codes.

A different way to improve Klee is to continue to target path coverage, but alter the definition of “path.” For example:

- When testing an x86-64 binary containing a cmov instruction, we could make sure to execute both its condition-true path and condition-false path.
- When testing the C expression **`foo(bar(),baz())`**, we could make sure to test evaluating foo() and bar() in both orders, instead of just letting the compiler pick one.

This piece proposes *undefined behavior coverage*, which simply means that for any operation that has conditionally-defined behavior, the well-defined and the undefined behaviors are considered to be separate paths. For example, the C expression `3/y` has two paths: one where `y` is zero and the other where `y` is non-zero.

Obviously, undefined behavior coverage only makes sense for languages such as C and C++ that admit operations with undefined behavior. An undefined behavior, as defined by the C and C++ standards, is one where the language implementation can do anything it likes. The point is to make the compiler developers’ job easier — they may simply assume that undefined behavior never happens. The tradeoff is that the burden of verification is shifted onto language users.

Undefined behavior coverage makes sense for what I call [type 2 functions](https://blog.regehr.org/archives/213): those whose behavior is conditionally well-defined.

# An Example

Here’s a simple C function:

> ```
> int add_and_shift (int x, int y, int z) {
>   return (x+y)<<z;
> }
>
> ```

Due to C’s undefined behaviors, this function has a non-trivial precondition:

> **0 â‰¤ z < sizeof(int)\*CHAR\_BIT**
>
> **INT\_MIN â‰¤ x+y â‰¤ INT\_MAX**

(This is for ANSI C; in C99 the precondition is stronger and quite a bit more complicated, but we won’t worry about that.) If the precondition is not satisfied, the function’s return value is unpredictable. In fact, it’s a bit worse than that: as soon as the program executes an undefined behavior the C implementation is permitted to send email to the developer’s mother (though this hardly ever happens).

The point is that although shift\_and\_add() seems to admit a single path, it really has a number of additional paths corresponding to failed preconditions for its math operators. If we fail to test these paths, we can miss bugs. Since the precondition checks for math operators in C/C++ are pretty simple, we can just add them in an early phase of the compiler, and that’s exactly what Peng’s hacked version of Clang does.

Without undefined behavior checks, LLVM code for add\_and\_shift() looks like this:

> ```
> define i32 @add_and_shift(i32 %x, i32 %y, i32 %z) nounwind readnone {
> entry:
>   %add = add i32 %y, %x
>   %shl = shl i32 %add, %z
>   ret i32 %shl
> }
> ```

Obviously there’s just one path, and the test case that Klee picks to exercise this path is:

- x = 0, y = 0, z = 0

Next, we compile the same function with undefined behavior checks and run Klee again. This time we get four test cases:

- x = 0, y = 0, z = 0
- x = 0, y = 0, z = 64
- x = -2, y = INT\_MIN, z = 0
- x = 2, y = 1, z = 0

The first three tests are exactly the kind of inputs we’d hope to see after looking at the precondition. The 4th input appears to follow the same path as the first. I don’t know what’s going on — perhaps it emerges from some idiosyncrasy of the checked code or maybe Klee simply throws in an extra test case for its own reasons.

Combining Klee with an undefined behavior checker causes Klee to generate additional test cases that — by invoking operations with undefined behavior — should shine some light into dark corners of the system under test. A potential drawback is that all the extra paths are going to cause the path explosion problem to happen sooner than it otherwise would have. However, this should not be serious since we can just run Klee on both versions of the code.

But this is all just talk. The real question is: does this method find more bugs?

# Bit Puzzle Results

The first collection of code is several years’ worth of solutions to an early assignment in Utah’s CS 4400. I [already discussed these](https://blog.regehr.org/archives/385), so I won’t repeat myself. For each bit puzzle, students receive a reference implementation (which they cannot simply copy since it doesn’t follow the rules for student solutions) and a simple test harness that runs their code against the reference implementation on some inputs, compares the results, and complains about any differences. For each of 10 bit puzzles we have 105 solutions written by students. The automated test suite determines that 84 of these 1050 solutions are faulty. In other words, they return incorrect output for at least one input. [Differential testing with Klee](https://blog.regehr.org/archives/467) finds seven additional buggy functions, for a total of 91.

When the students’ codes are augmented with checks for integer undefined behaviors, Klee finds more paths to explore. The test cases that it generates find the 91 incorrect functions that are already known plus 11 more, for a total of 102 buggy functions. Just to be perfectly clear: a buggy function is one that (after being compiled by GCC) returns the wrong output for an input in a test suite. We are not counting instances of undefined behavior as bugs, we are simply using Klee and the undefined behavior checker to generate a better test suite.

We were able to exhaustively test some of the bit puzzles. In these cases, exhaustive testing failed to find any bugs not found by differential Klee with undefined behavior coverage.

# Saturating Operation Results

The second collection of code is 336 [saturating math operations](https://blog.regehr.org/archives/390). In this case, the additional tests generated by Klee to satisfy undefined behavior coverage found no additional buggy functions beyond those found using differential whitebox testing. My hypothesis is that:

1. The shift-related undefined behaviors in these functions always involved constant arguments, since shifts were used only to compute values like the maximum and minimum representable integer of a certain width. Since the arguments were constant, Klee had no opportunity to generate additional test cases.
2. The addition and subtraction overflow undefined behaviors were compiled by GCC into modular arithmetic, despite the fact that this behavior is not guaranteed by the standard. This is a natural consequence of generating code using the x86 add and sub instructions. Modular arithmetic was the behavior that people (including me, as described in the previous post) wanted and expected. Therefore, undefined behavior coverage exposed no bugs. Modern C compilers sometimes compile math overflows in a non-modular way (for example, evaluating (x+1)>x to 1), but the saturating arithmetic functions — by chance — do not use code like that.

We were able to exhaustively test saturating operations that take chars (for 16 total bits of input) and short ints (for 32 total bits of input). In these cases, exhaustive testing failed to find any bugs not already found by differential Klee.

# Conclusion

Undefined behavior coverage is a special case of a more interesting code coverage metric that I’ll describe in a subsequent post. We need to try Klee + undefined behavior coverage on some real applications and see what happens; I’m cautiously optimistic that it will be useful.
