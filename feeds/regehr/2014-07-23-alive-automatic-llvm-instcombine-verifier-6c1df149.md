---
title: 'ALIVe: Automatic LLVM InstCombine Verifier'
url: https://blog.regehr.org/archives/1170
published: "2014-07-23T22:32:35Z"
feed: regehr
guid: http://blog.regehr.org/?p=1170
---

# ALIVe: Automatic LLVM InstCombine Verifier

**\[This post was jointly written by [Nuno Lopes](http://web.ist.utl.pt/nuno.lopes/), David Menendez, [Santosh Nagarakatte](http://www.cs.rutgers.edu/~santosh.nagarakatte/), and [John Regehr](http://www.cs.utah.edu/~regehr/).\]**

A modern compiler is a big, complex machine that contains a lot of moving parts, including many different kinds of optimizations. One important class of optimization is *peephole optimizations*, each of which translates a short sequence of instructions into a more desirable sequence of instructions. For example, consider this LLVM code that first shifts an unsigned 32-bit value 29 bits to the left, then 29 bits to the right:

```
%1 = shl i32 %x, 29
%2 = lshr i32 %1, 29

```

As long as %1 is not used anywhere else, this computation would be better written as:

```
%2 = and i32 %x, 7

```

Unsurprisingly, LLVM already knows how to perform this peephole optimization; the code implementing it can be [found here](http://llvm.org/docs/doxygen/html/InstCombineShifts_8cpp_source.html#l00233).

Although most peephole transformations are pretty straightforward, the problem is that there are a lot of them, creating a lot of opportunities to make mistakes. Many of LLVM’s peephole optimizations can be found in its [instruction combiner](http://llvm.org/svn/llvm-project/llvm/trunk/lib/Transforms/InstCombine/). According to Csmith, the instruction combiner was the single buggiest file in LLVM (it used to all be a single file) with 21 bugs found using random testing.

Wouldn’t it be nice if, instead of embedding peephole optimizations inside C++ code, they could be specified in a clearer fashion, and if bugs in them could be found automatically? These are some of the goals of a new project that we have been working on. So far, we have produced an early prototype of a [tool called ALIVe](https://github.com/nunoplopes/alive) that reads in the specification for one or more optimizations and then, for each one, either proves that it is correct or else provides a counterexample illustrating why it is wrong. For example, the optimization above can be written in ALIVe like this:

```
%1 = shl i32 %x, 29
%2 = lshr i32 %1, 29
  =>
%2 = and i32 %x, 7

```

Each optimization that is fed to ALIVe has an input or left-hand side (LHS), before the =>, that specifies a pattern to look for in LLVM code. Each optimization also has an output or right-hand side (RHS) after the => that specifies some new LLVM code that has to refine the original code. Refinement happens when the new code produces the same effect as the old code for all inputs that do not trigger undefined behavior.

Here is what happens when the code above is provided to ALIVe:

```
$ alive.py < example1.opt
----------------------------------------
Optimization: 1
Precondition: true
%1 = shl i32 %x, 29
%2 = lshr i32 %1, 29
  =>
%2 = and i32 %x, 7

Done: 1
Optimization is correct!

```

(All of the example ALIVe files from this post [can be found here](https://github.com/nunoplopes/alive/tree/master/tests/announcement_examples).)

The proof is accomplished by encoding the meaning of the LHS and RHS in an SMT query and then asking a solver whether the resulting formula is satisfiable. The “Done: 1” in the output means that ALIVe’s case splitter, which deals with instantiations of type variables, only had one case to deal with.

Of course, the optimization that we just specified is not a very good one: it handles only a single register width (32 bits) and a single shift amount (29 bits). The general form is a bit more interesting since the optimized code contains a constant not found on the LHS:

```
%1 = shl %x, C
%2 = lshr %1, C
  =>
%2 = and %x, (1<<(width(C)-C))-1

```

This also verifies as correct. To finish specifying this optimization, we also would want to support the case where the right shift comes before the left shift. At present, to do this in ALIVe you need to specify a second optimization rule, but in the future we will support transformations that are parameterized by lists of instructions.

## Undefined Behavior

Undefined behavior makes optimizations much harder to think about. For example, let's look at [PR20186](http://llvm.org/PR20186), an LLVM wrong-code bug that we found while translating optimizations from the instruction combiner into ALIVe, where the optimization looks like this:

```
%a = sdiv %X, C
%r = sub 0, %a
  =>
%r = sdiv %X, -C

```

In other words, dividing an integer by a constant, and then negating the result, can be optimized into dividing the integer by the negated constant. The optimization is attractive since it reduces the number of instructions. It also looks reasonable at first glance since we all learned in school that -(X/C) is equal to X/(-C).

However, ALIVe is not happy with this optimization:

```
$ alive.py < example2.opt
----------------------------------------
Optimization: 1
Precondition: true
%a = sdiv %X, C
%r = sub 0, %a
  =>
%r = sdiv %X, -C

Done: 1
ERROR: Domain of definedness of Target is smaller than Source's for i2 %r

Example:
%X i2 = 2 (0x2)
C i2 = 1 (0x1)
%a i2 = 2 (0x2)
Source value: 2 (0x2)
Target value: undef

```

What's the problem? If C=1, then -C=-1. When %X=-2, %X/(-C) overflows, since 2 is not representable in a 2-bit signed representation (ALIVe gives counterexamples at the smallest bitwidth that is required to trigger an optimization bug). [LLVM's language reference](http://llvm.org/docs/LangRef.html#sdiv-instruction) states that overflow in `sdiv` leads to undefined behavior. In summary, in the original code there was no undefined behavior when C=1, but the optimized version may be undefined in that case.

ALIVe is able to prove that a fixed version of this optimization is correct (but note that we've restricted the bitwidth since division is difficult for SMT solvers):

```
----------------------------------------
Optimization: 1
Precondition: ((C != 1) && !isSignBit(C))
%a = sdiv i16 %X, C
%r = sub 0, %a
  =>
%r = sdiv %X, -C

Done: 1
Optimization is correct!

```

And, in fact, the fix that was committed to LLVM's instruction combiner is equivalent to the one we've shown here.

A precondition specifies additional conditions beyond the occurrence of a pattern of instructions that must hold before the optimization is allowed to fire. The isSignBit() predicate is an LLVM function that tests whether a value is INT\_MIN, where INT\_MIN is the minimum integer value for a given bit-width. In general, it is not possible to call arbitrary LLVM code from ALIVe code, but we have reimplemented some commonly used functions, and will continue to add more of these as needed.

Another kind of undefined behavior bug that ALIVe can help find is incorrect propagation of the nsw/nuw flags which inform LLVM that an arithmetic operation is guaranteed to not overflow (nsw stands for "no signed wrap" and nuw is "no unsigned wrap"). For example, while translating instruction combiner patterns into ALIVe, we ran across [PR20189](http://llvm.org/PR20189) (which was independently discovered and then reported by another developer). This transformation wanted to convert x -nsw (-C) into x +nsw C, which is invalid; the optimization should only fire when both subtractions on the LHS are nsw.

So far, we have translated some of LLVM's instruction combiner (88 transformations, in total) into ALIVe; you can [look at the results of that effort here](https://github.com/nunoplopes/alive/tree/master/tests/instcombine). Each file in this directory corresponds to a file in LLVM's instruction combiner.

## Design Goals

Tools based on formal methods are not always easy to use. One of our major design goals is that regular LLVM developers, and other interested people, can use ALIVe to explore new transformations and to evaluate the correctness of existing transformations. To meet this goal, we have designed ALIVe to look and act like the textual representation of LLVM IR. The major departure from LLVM is support for abstraction. As we saw above, optimizations must contain elements of abstraction in order to be effective in as wide a range of situations as possible. Syntactically, ALIVe supports abstraction via omission and via variables. For example, if you fail to specify the bit-width of a register, then ALIVe will assume that the optimization is intended to apply to all bit-widths. If an optimizations doesn't care about bitwidth but requires that two registers have the same width, then they can be given symbolic widths. Another aspect of usability is that ALIVe should clearly explain its reasoning not only in the case where something is wrong with an optimization, but also when an optimization is correct. We are still working out the details of this, but probably ALIVe will need to share some details about the results of its type inference. The reason that ALIVe must explain itself in the "correct optimization" case is to help avoid vacuously correct specifications, which are undesirable although not nearly as harmful as incorrect specifications.

The second major goal of ALIVe is to automate as many hard and tedious tasks as possible. Obviously, reasoning about the correctness of optimizations is one of these tasks. Another is inferring the placement of LLVM's undefined behavior qualifiers: nsw, nuw, and exact. Above, we saw an example where LLVM was erroneously optimistic about where such a flag needed to be placed. We believe that the instruction combiner contains many instances of the opposite error: undefined behavior flags are left out in cases where they could have been added. ALIVe will help find these, leading to more effective optimizations. Going forward, we also want to compile ALIVe specifications into efficient C++ code that could be included in LLVM. This has multiple benefits for LLVM developers: eliminating bugs in the instruction combiner and reducing the time to develop new instruction combiner patterns. Code review can then focus on whether the pattern is profitable, rather than worrying that it may be wrong.

Our third goal is to separate correctness from profitability. These are orthogonal concerns and neither is easy to deal with. Currently, this separation is accomplished by not reasoning about profitability at all. In some cases, the profitability of peephole optimizations is straightforward: if instructions can be removed, they should be. But in many other cases, transformations are desirable for a more subtle reason: they create conditions that are conducive to further optimization. Some of this is captured by [canonicalization rules](http://llvm.org/docs/doxygen/html/InstructionCombining_8cpp_source.html#l00022).

## The Language

ALIVe abstracts away types. For example, the icmp instruction can either take a pair of integers of arbitrary bit-width or a pair of pointers. If you write an optimization that matches an icmp instruction, then ALIVe will try to prove the optimization correct for all possible integer bit-widths and pointer types, since an optimization may be correct for, say, i1 but not i8. Currently, however, ALIVe limits its proofs to bit-widths from 1 to 64 bits, and pointer sizes of 32 and 64 bits. Types may be named to express the constraint that a type is the same in different parts of an optimization.

ALIVe supports the specification of optimizations over several instruction types. Generally, anything that is left unspecified means that the optimization is intended to apply to all possible instantiations. For example, the following optimization encodes all possible choices of icmp comparisons and zero extensions:

```
%1 = icmp %n, %a
%2 = zext %1
%3 = icmp %n, %b
%4 = zext %3
%5 = or %2, %4
%6 = icmp eq %5, 0
  =>
%1 = icmp %n, %a
%3 = icmp %n, %b
%. = or %1, %3
%6 = icmp eq %., 0

```

ALIVe assumes that identifiers matching the regex 'C\\d\*' are constants. This is not important for correctness, but it will be important once we start generating C++ code from the specifications.

Like LLVM, identifiers starting with % are unaliased temporaries that are likely to be mapped to registers by a backend. They hold arbitrary values (subject to preconditions) on the input side. If a temporary appears on both the RHS and the LHS, then the RHS must end up with the same value for the temporary as the LHS, for all valid inputs. A valid input is one that does not trigger undefined behavior in any instruction on the LHS. If a register appears only on the LHS or only on the RHS and it is not an input, then it is an intermediate value that imposes no correctness requirement. In practice, that value will be removed from the output if the number of users reaches zero.

## What Next?

There's much left to do! We are still working on improving and extending the ALIVe language, as well as improving the performance of the tool and the quality of the diagnostics.

We welcome users and contributions from the LLVM community. We are looking for feedback regarding the specification language, the tool itself, how to best integrate ALIVe within the LLVM development process, and so on.

If you find a bug in ALIVe or find an LLVM bug using ALIVe or if you manage to verify an awesome optimization, please get in touch.

## Licensing

Although ALIVe is open source (Apache 2 license), it relies on the [Z3 SMT solver](http://z3.codeplex.com/). Z3 is licensed under the Microsoft Research License Agreement (MSR-LA), which forbids commercial usage. However -- after discussions with people at Microsoft -- it is our and Microsoft's understanding that using Z3 with ALIVe for the development of LLVM does not constitute commercial usage for the following reasons:

1. LLVM is not a commercial product of any particular company.
2. ALIVe is free.

Questions regarding Z3's license should be directed to Behrooz Chitsaz (behroozc@microsoft.com), the Director of IP at Microsoft Research, who kindly offered to answer any question regarding the usage of Z3 within ALIVe. This statement does not constitute legal advice and it is not legally binding. Interested parties should seek professional consultation with an attorney.
