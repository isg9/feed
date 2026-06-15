---
title: Checking Up on Dataflow Analyses
url: https://blog.regehr.org/archives/1388
published: "2016-04-18T19:15:13Z"
feed: regehr
guid: http://blog.regehr.org/?p=1388
---

# Checking Up on Dataflow Analyses

An important tool for reasoning about programs without running them is dataflow analysis, which can be used to compute facts such as:

- an integer-valued variable is non-negative at a particular program point

- a conditional branch always goes one way

- an indirect write cannot modify a particular array

Dataflow facts drive optimizations such as constant propagation and dead code elimination. Outside of compilers, dataflow analyses are important in tools that prove safety properties, such as absence of undefined behaviors in C or C++ programs. Dataflow analysis is backed by a rich academic literature and tool writers have decades of experience implementing it. [This set of lecture notes](http://www.itu.dk/people/brabrand/UFPE/Data-Flow-Analysis/static.pdf) is my favorite introduction to dataflow analysis.

Despite all of the nice theorems, dataflow analyses are hard to get right, in part because people are just not very good at reasoning about properties that hold over sets of program executions. As a simple exercise in this kind of thinking, try this: you have a variable A whose value is known to be in the range \[0..1000\] and a variable B that is known to be in the range \[10000..10050\]. The program computes the bitwise xor of A and B and assigns the result into C. Provide tight bounds on the values that C can take. If you find this to be easy, that’s great — but consider that you then have to make this work for all intervals and after that there are about 9,999 similarly fiddly things left to implement and any mistakes you make are likely to result in miscompilations.

So now let’s finally get to the point of this post. The point is that since dataflow is hard, if you implement a dataflow analysis, you should also implement a dynamic checker that looks for cases where the analysis has come to the wrong conclusion. To keep going with the running example, the program being analyzed contains this line of code:

```
// A in [0..1000] and B in [10000..10050]
C = A ^ B;

```

If our interval analysis says that C has to be in the range \[9225..12255\], we would rewrite the code to add an assertion:

```
// A in [0..1000] and B in [10000..10050]
C = A ^ B;
assert(C >= 9225 && C <= 12255);

```

Now we can run the instrumented program and, if we’re very lucky, it will execute this code with values such as A = 830 and B = 10041, making C = 9223, triggering an assertion violation telling us that our analysis is unsound and giving us something to debug. The other way to debug this sort of problem is to backtrack from a miscompilation — an experience enjoyed by basically nobody.

Every dataflow analysis that I’ve implemented or supervised the implementation of has gotten a dynamic checker. This is a great testing technique that has numerous advantages. It finds bugs in practice. It can be automated, either using a program generator like Csmith or else using a regular test suite. The annotated programs serve as a useful way to look at the precision of an analysis: if not very many assertions show up, or if the bounds are very loose, then we probably need to do some tuning for precision. Finally, if we don’t find any bugs, then the dataflow analysis is probably (please excuse me, formal methods people) correct enough for practical purposes.

Of course, not everything that can be computed by a dataflow analysis can be expressed in code. For example, in C and C++ there’s no lightweight way to assert that a pointer refers to valid storage or that a variable is uninitialized. [E-ACSL](http://frama-c.com/eacsl.html) is a neat piece of related work in this general area.

Just recently I’ve been looking at the [efficiency of integer overflow checking using LLVM](https://blog.regehr.org/archives/1384) and since then I wrote some code that uses integer range dataflow facts to remove overflow checks that can be proved to not fire. [Here’s the under-review patch](http://reviews.llvm.org/D19110). The ranges are computed by an LLVM pass called LazyValueInfo (LVI), which better be correct if we’re going to rely on it, so I wrote a [little LLVM pass](https://github.com/regehr/llvm-test-lvi) that does the necessary checking. It processes each function in between the optimization and code generation phases by:

- [Making a list of instructions](https://github.com/regehr/llvm-test-lvi/blob/master/TestLVI.cpp#L69-L82) that LVI knows something about.

- Creating a new basic block [that executes a trap instruction](https://github.com/regehr/llvm-test-lvi/blob/master/TestLVI.cpp#L23-L32).

- [Conditionally jumping to the trap block](https://github.com/regehr/llvm-test-lvi/blob/master/TestLVI.cpp#L115-L130) if the actual value produced by any instruction is out of bounds with respect to the predicted value range.

(If you build this pass, you’ll need a slightly hacked LLVM, see the README.) Although it might seem tempting to run the optimizers again on the instrumented code in order to clean it up a bit, this would be a very bad idea: the very dataflow analyses that we’re trying to check up on would be used to drive optimizations, which could easily end up incorrectly deleting our checks.

So far, some testing using Csmith hasn’t turned up any bugs in LVI, which is great. Less great is the fact that it drops precision all over the place: a lot of tuning up is needed before it is the basis for a really strong redundant overflow check remover.

The technique I’m describing is not as widely known or as widely used as it should be, considering how easy and effective it is. The best answer is that C = \[9216..10239\].

**UPDATE:** The [latest version](https://github.com/regehr/llvm-test-lvi) of my LLVM dataflow checker also validates the results of computeKnownBits(), which tries to figure out which bits of a value must be either zero or one.

[Here’s a bit of further reading](http://blog.frama-c.com/index.php?pages/Csmith-testing) about how these techniques (and others) were applied to the Frama-C static analyzer.
