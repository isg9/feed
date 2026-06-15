---
title: Early Superoptimizer Results
url: https://blog.regehr.org/archives/1146
published: "2014-05-11T16:06:10Z"
feed: regehr
guid: http://blog.regehr.org/?p=1146
---

# Early Superoptimizer Results

\[ [Here’s a shortcut to the results](http://www.cs.utah.edu/~regehr/souper/). But it would be best to read the post first.\]

Following my [previous superoptimizer post](https://blog.regehr.org/archives/1109), my student Jubi and I were getting up to speed on the prerequisites — SMT solvers, LLVM internals, etc. — when Googler [Peter Collingbourne](http://www.pcc.me.uk/~peter/) contacted me saying that he had recently gotten a superoptimizer up and running and might I be interested in working with him on it? I read his code and found it to be charmingly clear and simple. Also, one of my basic principles in doing research is to avoid competing, since competition wastes resources and burns students because the resulting race to publication effectively has an arbitrary winner. So I immediately started feeding bug reports to Peter.

The new superoptimizer, [Souper](https://github.com/google/souper/), makes a few simplifying assumptions:

- The only optimization candidates that it considers are the true and false values. Therefore, at present Souper only harvests expressions that compute an i1: a one-bit integer, which is how Booleans are represented in LLVM. Thus, the result of a Souper run is a collection of expressions that LLVM could have — but did not — evaluate to either true or false.

- It doesn’t yet have models for all instructions or for all undefined behaviors for the instructions it does support.

These assumptions need to be relaxed. One generalization that should be pretty easy is to harvest expressions that end up as integers of arbitrary width. The interesting thing about this is that we cannot take time to check if every harvested expression evaluates to, for example, every possible value that an i32 can take. What we will do instead is to ask the SMT solver to synthesize the equivalent constant. The problem is that by default, when we make an equivalence query to an SMT solver, it is an unsat result that signals equivalence, and unsat doesn’t come with a model — it indicates failure to find a model. It turns out there’s a cute trick (which I learned from Nuno Lopes) involving a quantifier which flips a query around such that an equivalence results in sat, and therefore a model, from which we can pluck the synthesized constant. Consider this Z3/Python code where we’re asking, for a variety of constants c, how to express i\*c (where i is an integer variable) in the form i<<x + i<<y + i<<z:

```
from z3 import *

s = Solver()

def checkit (c):
    s.push()
    i, x, y, z = BitVecs('i x y z',32)
    q = ForAll (i, i*c == ((i<<x) + (i<<y) + (i<<z)))
    s.add(q)
    s.add(x>=0, x<32)
    s.add(y>=0, y<32)
    s.add(z>=0, z<32)
    if s.check() == sat:
        m = s.model()
        print ("i * " + str(c) +
               " == i<<" + str(m.evaluate(x)) +
                " + i<<" + str(m.evaluate(y)) +
                " + i<<" + str(m.evaluate(z)))
    else:
        print "i * " + str(c) + " has no model"
    s.pop()

for m in range(100):
    checkit(m)

```

This is just an example but it’s the kind of thing that might make sense on a small embedded processor where the integer multiply instruction is expensive or doesn’t exist. The results include:

**```** **i * 28 == i<<4 + i<<3 + i<<2** **i * 29 has no model** **i * 30 has no model** **i * 31 has no model** **i * 32 == i<<4 + i<<3 + i<<3** **i * 33 == i<<4 + i<<4 + i<<0**

**```**

The [full set of results is here](https://blog.regehr.org/extra_files/z3_shift_mul.txt). I particularly enjoyed the solver's solutions for the first three cases. So we know that the synthesis part of a superoptimizer is possible and in fact probably not all that difficult. But that's a digression that we'll return to in a later post; let's get back to the main topic.

Now I'll show you how to read Souper's output. You may find it useful to keep the [LLVM instruction set reference](http://llvm.org/docs/LangRef.html) handy. Here's an optimization report:

**```** **%0:i32 = var** **%1:i32 = mul 4294967294:i32, %0** **%2:i1 = eq 1:i32, %1** **cand %2 0:i1**

**```**

The first line tells us that %0 has type i32 -- a 32-bit integer -- corresponding to a signed or unsigned int in C/C++, and that it is a variable: an input to the superoptimized code that may hold any value. Reasoning about any-valued variables is hard but solvers are good at it and that is the entire point of the superoptimizer.

The second line tells us that %1 is a new i32 computed by multiplying %0 by -2. The third line tells us that %2 is a new i1 -- a Boolean or 1-bit integer -- computed by seeing if %1 is equal to 1. The last line, starting with "cand", is Souper telling us that it believes %2 will always take the value 0. If Souper tells us this when running on optimized code, it has found a missed optimization. In this case LLVM has missed the fact that multiplying an arbitrary value by an even number can never result in an odd number. Is this a useful optimization to implement in LLVM? I don't know, but GCC does it, see the [bottom of this page](http://www.cs.utah.edu/~regehr/souper/output_6.html).

Souper finds many missed optimizations that fit this general pattern:

**```** **%0:i32 = var** **%1:i64 = sext %0** **%2:i64 = sdiv 2036854775807:i64, %1** **%3:i1 = ne 0:i64, %2** **cand %3 1:i1**

**```**

Here the observation is that if we divide a large constant by an arbitrary 32-bit value, the result cannot be zero. GCC does not find this one.

Some Souper output contains path constraints:

**```** **%0:i32 = var** **%1:i1 = eq 0:i32, %0** **pc %1 1:i1** **%2:i32 = addnsw 1:i32, %0** **%3:i1 = slt %2, 2:i32** **cand %3 1:i1**

**```**

Here, at line 3, we learn that %1 must take the value 1 in the remaining code due to a path constraint. In the original LLVM code there was a conditional branch exiting if %1 had the value 0. Since %1 has the value 1, we can infer, in the remaining code, that %0 contains 0. Thus, %2 contains 1 and the expression %2 < 2 must evaluate to true.

Finally, this charming example exploits the fact that if the product of two numbers is not zero, then neither of the numbers could have been zero:

**```** **%0:i32 = var** **%1:i32 = var** **%2:i32 = mul %0, %1** **%3:i1 = eq 0:i32, %2** **pc %3 0:i1** **%4:i1 = eq 0:i32, %0** **cand %4 0:i1**

**```**

One more thing that you might see in the full set of results is an entry like this: `%0 = block`. This means (more or less) that %0 is a value that is going to pass through the code to a phi node without being otherwise used, this is useful for increasing Souper's precision.

I think that's about all you need to know in order to read [the full set of results](http://www.cs.utah.edu/~regehr/souper/) from a couple days of running Csmith, Souper, and C-Reduce in a loop. First, we wait for Souper to find a missed optimization and then second, we find a minimal C program that exhibits the missed optimization. The results have been ranked in a way that attempts to push more similar results (that are more likely to be duplicates) lower in the list.

So far, the most common pattern that comes out of Souper's findings is that LLVM needs an integer range analysis. Such an analysis would also help eliminate integer overflow checks, one of my hobby horses. LLVM also doesn't always propagate information that would be best represented at the bit level, such as the even/odd distinction required for the first optimization that I discussed. Finally, LLVM does not always learn from branches. My not-necessarily-educated guess is that all of this is a symptom of LLVM's heavy reliance on the [instruction combiner](http://llvm.org/svn/llvm-project/llvm/trunk/lib/Transforms/InstCombine/), which is not so much an optimization pass as a loose federation of a few hundred peephole passes.

Some of the missing LLVM optimizations won't be hard to implement for people have passable C++ and who have spent some time becoming familiar with the instruction combiner. But here are a few things we need to keep in mind:

- One might ask: Does it make sense to harvest missed optimizations from randomly generated code? My initial idea was that since Csmith's programs are free from undefined behaviors, the resulting optimizations would be less likely to be evil exploitation of undefined behaviors. But also I did it because it was easy and I was curious what the results would look like. My judgement is the the results are interesting enough to deserve a blog post. Perhaps an easier way to avoid exploiting undefined behavior would be to add a command line option telling Souper to avoid exploiting undefined behaviors.

- For each missed optimization we should do a cost/benefit analysis. The cost of implementing a new optimization is making LLVM a bit bigger and a bit more likely to contain a bug. The benefit is potential speedup of code that contains the idioms.

- Although the reduced C programs can be useful, you should look at Souper output first and the C code second. For one thing, the Boolean that Souper finds is sometimes a bit obscured in the C code. For another, the C-Reduce output is somewhat under-parenthesized -- it will test your knowledge of C's operator precedence rules. Finally C-Reduce has missed some opportunities to fold constants, so for example we see ~1 instead of -2 in the 2nd example from the top.

- Each missed optimization found by Souper should be seen as a member of a class of missed optimizations. So the goal is obviously not to teach LLVM to recognize the specific cases found by Souper, but rather to teach it to be smart about some entire class of optimizations. My belief is that this generalization step can be somewhat automated, but that is a research problem.

- Although all of the optimizations that I've looked at are correct, there's always the possibility that some of them are wrong, for example due to a bug in Souper or STP.

This article presents some very early results. I hope that it is the beginning of a virtuous circle where Souper and LLVM can both be strengthened over time. It will be particularly interesting to see what kinds of optimizations are missing in LLVM code emitted by [rustc](https://github.com/mozilla/rust), GHC, or [llgo](https://github.com/go-llvm/llgo).

**UPDATE:** Here are some bugs that people have filed or fixed in response to these results:

- [Optimize signed icmp of -(zext V)](http://llvm.org/viewvc/llvm-project?view=revision&revision=208809)
- [Optimize integral reciprocal (udiv 1, x and sdiv 1, x) to not use division](http://llvm.org/viewvc/llvm-project?view=revision&revision=208750)
- [ComputeMaskedBits & friends should know that multiplying by a power of two leaves low bits clear](http://llvm.org/bugs/show_bug.cgi?id=19711)

It's very cool that people are acting on this! Please let me know if you know of more results than are listed here.
