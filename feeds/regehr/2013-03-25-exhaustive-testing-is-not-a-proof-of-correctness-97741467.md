---
title: Exhaustive Testing is Not a Proof of Correctness
url: https://blog.regehr.org/archives/917
published: "2013-03-25T19:12:07Z"
feed: regehr
guid: http://blog.regehr.org/?p=917
---

# Exhaustive Testing is Not a Proof of Correctness

It is often said that exhaustively testing a piece of software is equivalent to performing a proof of correctness. Although this idea is intuitively appealing—and I’ve said it myself a few times—it is incorrect in a technical sense and also in practice.

What’s wrong with exhaustive testing in practice? The problem comes from the question: What is it that we are testing? One possible answer would be “We’re testing the program we wrote.” For example, I might write this program containing an exhaustive test harness and an ostensible identity function which is defined to return the value of its argument for any value in the range -5..5:

```
#include <stdio.h>

int identity (int x)
{
}

int main (void)
{
  int i;
  for (i=-5; i<=5; i++) {
    printf ("identity(%d) = %d\n", i, identity(i));
  }
  return 0;
}

```

Let’s run the exhaustive tester:

```
$ gcc ident_undef.c ; ./a.out
identity(-5) = -5
identity(-4) = -4
identity(-3) = -3
identity(-2) = -2
identity(-1) = -1
identity(0) = 0
identity(1) = 1
identity(2) = 2
identity(3) = 3
identity(4) = 4
identity(5) = 5

```

Nice! We have used exhaustive testing to show that our program is correct. But actually not:

```
$ gcc -O ident_undef.c ; ./a.out
identity(-5) = 1322972520
identity(-4) = 26
identity(-3) = 18
identity(-2) = 18
identity(-1) = 18
identity(0) = 18
identity(1) = 17
identity(2) = 17
identity(3) = 17
identity(4) = 17
identity(5) = 17

```

The identity function is, of course, woefully wrong even if it happened to work at -O0. So clearly we did not prove the C program to be correct. On the other hand, perhaps we proved the compiled program to be correct? In this case (I looked at the object code) the compiled program is correct.

The thing is, it is not difficult to create a different compiled program that passes exhaustive testing, while being demonstrably incorrect. For example, a program that relies upon buggy operating system functionality or buggy hardware (such as a Pentium with the fdiv bug) could pass exhaustive testing while being totally wrong.

So maybe the thing that we’re proving correct isn’t just the compiled program. Maybe it’s the entire computer system: the hardware platform, the operating system, and also our compiled program. Now, a program that gives the correct result by relying on the fdiv bug is perfectly fine.

There are two problems with saying that we have proved our computer system correct (with respect to its ability to execute a program we have chosen). First, the computer system as a whole is of course incorrect: any real hardware and operating system will have some known bugs and we have failed to argue that none of these bugs can affect the execution of the identity function (to do that, we would have to test every input to our program in every possible state that the hardware and OS could be in—a tough job). Second, the computer is a real physical object and it follows the laws of physics. If we wait long enough, a disturbance will occur (it could be external, like some alpha particles, or internal, like a quantum fluctuation) that causes the computer to fail to give the right result for our program.

The fundamental problem here is that our “proof” was not in terms of an abstraction, it was in terms of a physical object. Physical objects live in the real world where all guarantees are probabilistic.

The second problem with saying that exhaustive testing constitues a proof (actually, the second aspect of the only problem) is that a proof of correctness is a mathematical proof, whereas a collection of successful test cases is not a mathematical proof. A collection of successful test cases, even if it is exhaustive, may form a very compelling argument, but that doesn’t make it a proof.

Although I do not know of anyone who has done this, it should be perfectly feasible to automatically turn an exhaustive collection of test cases into a proof of correctness. To do this you would first need to borrow (or create) a formal semantics for the computing platform. This could be a semantics for C, it could be a semantics for x86-64, or it could be something different. The formal semantics can be used to evaluate the behavior of the computer program for every input; if the behavior is correct for all inputs, then we can finally construct a proof of correctness. The point of using a formal semantics is that it provides a mathematical interpretation, as opposed to a physical one, for the computing platform.
