---
title: Proofs from Tests
url: https://blog.regehr.org/archives/913
published: "2013-03-21T17:52:40Z"
feed: regehr
guid: http://blog.regehr.org/?p=913
---

# Proofs from Tests

An idea that I’ve been interested in for a while is that a good test suite should be able to directly support formal verification. How would this work, given that testing usually misses bugs? The basic insight is that a test case is usually telling us about more than just one execution: it’s telling us about a class of similar executions that will all give the correct result if the test case succeeds. [Partition testing](http://en.wikipedia.org/wiki/Equivalence_partitioning) is based on this same idea.

To get started, let’s do a little thought experiment where our test suite is exhaustive: it tests every possible input value. Although this is obviously very restrictive, exhaustive testing is sometimes feasible for non-trivial examples such as a function that decodes a binary ARM instruction.

What does it take to make a proof out of an exhaustive collection of test cases? We need some sort of a mathematical model for how the system works—an executable formal semantics—and also a mathematical model for how the system is intended to work: a formal specification. Then, we basically unroll the execution of the system on each test case and show that the result is equivalent to the specified result. The conjunction of this collection of statements about the system is our proof. The problem is that it’s going to be a very large, unsatisfying proof.

So the next problem is turning the cumbersome proof into a prettier one. The basic strategy is to look for a collection of “similar” conjuncts: these should all have the same control flow and, for all operations performed (addition, subtraction, etc.) the inputs and outputs should live in the same continuous region. Two’s complement addition, for example, has a discontinuity where it wraps around; the 8-bit signed addition operations 50+50 and 100+100 therefore do not live in the same continuous region. I’m guessing that for a suitable definition of “continuous” (which may not be totally straightforward for bitwise operations) it should be possible to automatically collapse a collection of cases into a single case that covers all of them. In terms of partition testing, our final proof will have one conjunct for each partition.

But we still haven’t accomplished anything particularly useful, since we can’t exhaustively test most of the codes that we care about. What I hope is that if we actually went through this exercise, we’d learn something about how to attack the more general case where we have a non-exhaustive collection of test cases and we want to automatically expand each of them to reason about an entire partition of the input space. If we could do that, a collection of test cases achieving 100% partition coverage would lead to a proof of correctness.

Consider a 2-bit saturating adder where the result of the addition sticks at the minimum or maximum representable value instead of overflowing. This function takes a pair of values in the range -2..1 and returns a value in the same range. A C implementation would be:

```
int twobit_sat_add (int a, int b)
{
  int result = a+b;
  if (result>1) return 1;
  if (result<-2) return -2;
  return result;
}

```

Here is an exhaustive collection of test cases with results:

```
twobit_sat_add (-2, -2) = -2
twobit_sat_add (-2, -1) = -2
twobit_sat_add (-2,  0) = -2
twobit_sat_add (-2,  1) = -1
twobit_sat_add (-1, -2) = -2
twobit_sat_add (-1, -1) = -2
twobit_sat_add (-1,  0) = -1
twobit_sat_add (-1,  1) =  0
twobit_sat_add ( 0, -2) = -2
twobit_sat_add ( 0, -1) = -1
twobit_sat_add ( 0,  0) =  0
twobit_sat_add ( 0,  1) =  1
twobit_sat_add ( 1, -2) = -1
twobit_sat_add ( 1, -1) =  0
twobit_sat_add ( 1,  0) =  1
twobit_sat_add ( 1,  1) =  1

```

The proof of correctness requires an executable semantics for C; these are available. The resulting proof will be large, even for this simple example, so let’s look at the partitions.

Only one test case takes the first return path:

```
twobit_sat_add (1, 1) = 1

```

Three test cases take the second return path:

```
twobit_sat_add (-2, -2) = -2
twobit_sat_add (-2, -1) = -2
twobit_sat_add (-1, -2) = -2

```

And these test cases take the third return path:

```
twobit_sat_add (-2,  0) = -2
twobit_sat_add (-2,  1) = -1
twobit_sat_add (-1, -1) = -2
twobit_sat_add (-1,  0) = -1
twobit_sat_add (-1,  1) =  0
twobit_sat_add ( 0, -2) = -2
twobit_sat_add ( 0, -1) = -1
twobit_sat_add ( 0,  0) =  0
twobit_sat_add ( 0,  1) =  1
twobit_sat_add ( 1, -2) = -1
twobit_sat_add ( 1, -1) =  0
twobit_sat_add ( 1,  0) =  1

```

Within each collection of test cases, no discontinuities in C’s addition operator are triggered, so our final proof should contain three top-level conjuncts. Is it possible to reconstruct this proof given only, for example, these three representative test cases?

```
twobit_sat_add ( 1,  1) =  1
twobit_sat_add (-2, -1) = -2
twobit_sat_add ( 0, -1) = -1

```

I don’t know, but I hope so. Perhaps techniques such as abstract interpretation can be used to reason effectively about the behavior of entire partitions. If so, then this proof based on three test cases would also work for saturating additions of any width (assuming that the implementation had available to it an addition operation at least one bit wider—if not, a bit more work is required).

To summarize, here’s the work to be done:

- For some simple programming language where an executable formal semantics already exists, show that an exhaustive collection of test cases for a function can be automatically turned into a proof of correctness.

- Define partition coverage in an appropriate fashion.

- Show that an exhaustive proof of correctness can be turned into a proof by partitions.

- Show that a single test case per partition is sufficient to generate an overall proof of correctness.

- For extra credit, automatically detect the case where the test suite does not achieve 100% partition coverage.

I suspect that a lot of this hinges upon finding an appropriate way to define partitions. A conservative definition for what constitutes a partition will make the research easier (because there will be less going on inside each partition) but also less useful because more test cases will be required to achieve 100% partition coverage. A liberal definition of partition makes the testing part easier and the proofs harder. Clearly, in practice, it is not OK to insist (as I did above) that every test case in a partition has identical control flow: we’ll need to be able to abstract over loop counts and recursion depth.

The basic insight here is that some of the work of formal verification can be borrowed from the testing phase of software development. We can’t avoid testing even when we’re doing formal verification, so we might as well try to get more benefit out of the sunk costs. My hope is that for a smallish language subset, this project is within the scope of a PhD thesis. If anyone knows of work that already accomplishes some or all of the goals I’ve outlined here, please let me know.
