---
title: Compilers and Termination Revisited
url: https://blog.regehr.org/archives/161
published: "2010-06-07T17:31:35Z"
feed: regehr
guid: http://blog.regehr.org/?p=161
---

# Compilers and Termination Revisited

My earlier post [C compilers Disprove Fermat’s Last Theorem](https://blog.regehr.org/archives/140) generated a good amount of discussion both here and on Reddit.  Unfortunately, the discussion was riddled with misunderstandings. Some of this was because the topic is subtle, but some was my fault: the post was intended to be lightweight and I failed to explain the underlying issues well at all.  This post gives a more complete picture.

Our goal is to determine the meaning of this C program:

```
#include <stdio.h>

int fermat (void) {
  const int MAX = 1000;
  int a=1,b=1,c=1;
  while (1) {
    if (((a*a*a) == ((b*b*b)+(c*c*c)))) return 1;
    a++;
    if (a>MAX) {
      a=1;
      b++;
    }
    if (b>MAX) {
      b=1;
      c++;
    }
    if (c>MAX) {
      c=1;
    }
  }
  return 0;
}
```

```
int main (void) {
  if (fermat()) {
    printf ("Fermat's Last Theorem has been disproved.\n");
  } else {
    printf ("Fermat's Last Theorem has not been disproved.\n");
  }
  return 0;
}
```

This program is a simple counterexample search; it terminates if it is able to disprove a special case of Fermat’s Last Theorem.  Since this theorem is generally believed to be true, we would expect a counterexample search to run forever.  On the other hand, commonly available C compilers emit terminating code:

```
regehr@john-home:~$ icc fermat2.c -o fermat2
regehr@john-home:~$ ./fermat2
Fermat's Last Theorem has been disproved.

regehr@john-home:~$ suncc -O fermat2.c -o fermat2
"fermat2.c", line 20: warning: statement not reached
regehr@john-home:~$ ./fermat2
Fermat's Last Theorem has been disproved.
```

Before proceeding, let’s clarify a few things.  First, I am not asking this question:

> How could I change this code so that the compiler will respect its termination characteristics?

This is easy and there are several reliable ways to do it, for example using volatile variables or inline assembly.

Second, I am not interested in “answers” like this:

> It is obvious what is going on here: the compiler sees that one path out of the function is dead, and then deduces that the only remaining path must be live.

This observation is not untrue, but it’s a little like explaining that World War II happened because people couldn’t all just get along.  It completely fails to get at the heart of the matter.

Third, there are no integer overflow games going on here, as long as the code is compiled for a platform where an int is at least 32 bits. This is easy to see by inspecting the program. The termination problems are totally unrelated to integer overflow.

# Program Semantics

A program’s meaning is determined by the semantics of the language in which it is written.  The semantics tells us how to interpret constructs in the language (how wide is an integer? how does the “if” operator work?)  and how to put the results of operations together into an overall result.  Some computer languages have a formal mathematical semantics, some have a standards document, and some simply have a reference implementation.

Let’s look at a few examples.  I’ll continue to use C, and will be quite informal (for a formal take on the meaning of C programs, see [Michael Norrish’s PhD thesis](http://www.cl.cam.ac.uk/TechReports/UCAM-CL-TR-453.pdf)).  To keep things simple, we’ll look only at “self-contained” programs that take no inputs.  Consider this program:

```
int main (void) {
  return 3;
}
```

it means {{3,””}}.  The notation is slightly cluttered but can be read as “the program has a unique interpretation which is to return 3 and perform no side effects.”  To keep things simple, I’m representing side effects as a string printed to stdout.  Of course, in the general case there are other kinds of side effects.

Here’s a slightly more complex program:

```
int main (void) {
  int a = 1;
  return 2 + a;
}
```

it also means {{3,””}} since there is no possibility of integer overflow.

Not all programs have a unique meaning.  Consider:

```
int main (void) {
  unsigned short a = 65535;
  return a + 1;
}
```

The meaning of this program is {{65536,””}, {0,””}}.  In other words, it has two meanings: it may return 65536 or 0 (in both cases performing no side-effecting operations) depending on whether the particular C implementation being used has defined the size of an unsigned short to be 16 bits or to be larger than 16 bits.

Another way that a C program can gain multiple meanings is by performing operations with unspecified behavior.  Unlike implementation defined behavior, where the implementation is forced to document its choice of behavior and use it consistently, unspecified behavior can change even within execution of a single program.  For example:

```
int a;
```

```
int assign_a (int val) {
  a = val;
  return val;
}
```

```
int main (void) {
  assign_a (0) + assign_a (1);
  return a;
}
```

Because the order of evaluation of the subexpressions in C is unspecified, this program means {{0,””}, {1,””}}.  That is, it may return either 0 or 1.

This C program:

```
#include <stdio.h>
```

```
int main (void) {
  return printf ("hi\n");
}
```

means {{0,””}, {1,”h”}, {2,”hi”}, {3,”hi\\n”}, {-1,””}, {-2,””}, {-3,””}, …}.  The 4th element of this set, with return value 3, is the one we expect to see.  The 1st through 3rd elements indicate cases where the I/O subsystem truncated the string.  The 5th and subsequent elements indicate cases where the printf() call failed; the standard mandates that a negative value is returned in this case, but does not say which one.  Here it starts to become apparent why reasoning about real C programs is not so easy.  In subsequent examples we’ll ignore program behaviors where printf() has something other than the expected result.

Some programs, such as this one, don’t mean anything:

```
#include <limits.h>
```

```
int main (void) {
  return INT_MAX+1;
}
```

In C, overflowing a signed integer has undefined behavior, and a program that does this has no meaning at all.  It is ill-formed. We’ll denote the meaning of this program as {{UNDEF}}.

It’s important to realize that performing an undefined operation has unbounded consequences on the program semantics.  For example, this program:

```
#include <limits.h>
```

```
int main (void) {
  INT_MAX+1;
  return 0;
}
```

also means {{UNDEF}}. The fact that the result of the addition is not used is irrelevant: operations with undefined behavior are program cancer and poison the entire execution. Many real programs are undefined only sometimes.  For example we can slightly modify an earlier example like this:

```
int a;
```

```
int assign_a (int val) {
  a = val;
  return val;
}
```

```
int main (void) {
  assign_a (0) + assign_a (1);
  return 2/a;
}
```

This program means {{UNDEF}, {2,””}}.  Showing that a real C program has well-defined behavior in all possible executions is very difficult.  This, combined with the fact that undefined behavior often goes unnoticed for some time, explains why so many C programs contain security vulnerabilities such as buffer overflows, integer overflows, etc.

One might ask: Is a C program that executes an operation with undefined behavior guaranteed to perform any side effects which precede the undefined operation?  That is, if we access some device registers and then divide by zero, will the accesses happen?  I believe the answer is that the entire execution is poisoned, not just the parts of the execution that follow the undefined operation.  Certainly this is the observed behavior of C implementations (for example, content buffered to stdout is not generally printed when the program segfaults).

Finally we’re ready to talk about termination.  All examples shown so far have been terminating programs.  In contrast, this example does not terminate:

```
#include <stdio.h>
```

```
int main (void) {
  printf ("Hello\n");
  while (1) { }
  printf ("World\n");
  return 0;
}
```

Clearly we cannot find an integer return value for this program since its return statement is unreachable.  The C “abstract machine,” the notional C interpreter defined in the standard, has an unambiguous behavior when running this program: it prints Hello and then hangs forever.  When a program behaves like this we’ll say that its meaning is {{âŠ¥,”Hello\\n”}}.  Here âŠ¥ (pronounced “bottom”) is simply a value outside the set of integers that we can read as indicating a non-terminating execution.

Assuming that signed integers can encode values up to two billion (this is true on all C implementations for 32- and 64-bit platforms that I know of), the semantics that the abstract C machine gives to the Fermat program at the top of this post is {{âŠ¥,””}}.  As we have seen, a number of production-quality C compilers have a different interpretation.  We’re almost ready to get to the bottom of the mystery but first let’s look at how some other programming languages handle non-terminating executions.

# Java

[Section 17.4.9 of the Java Language Specification (3rd edition)](http://java.sun.com/docs/books/jls/third_edition/html/memory.html#17.4.9) specifically addresses the question of non-terminating executions, assigning the expected {{âŠ¥,””}} semantics to a straightforward Java translation of the Fermat code. Perhaps the most interesting thing about this part of the Java Language Specification is the amount of text it requires to explain the desired behavior.  First, a special “hang” behavior is defined for the specific case where code executes forever without performing observable operations.  Second, care is taken to ensure that an optimizing compiler does not move observable behaviors around a hang behavior.

# C++

C++0x, like Java, singles out the case where code executes indefinitely without performing any side effecting operations.  However, the interpretation of this code is totally different: it is an undefined behavior.  Thus, the semantics of the Fermat code above in C++0x is {{UNDEF}}.  In other words, from the point of view of the language semantics, a loop of this form is no better than an out-of-bounds array access or use-after-free of a heap cell. This somewhat amazing fact can be seen in the following text from Section 6.5.0 of the draft standard (I’m using [N3090](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2010/n3090.pdf)):

> A loop that, outside of the for-init-statement in the case of a for statement,
> - makes no calls to library I/O functions, and
> - does not access or modify volatile objects, and
> - performs no synchronization operations (1.10) or atomic operations (Clause 29)
>
> may be assumed by the implementation to terminate.
>
> \[ Note: This is intended to allow compiler transformations, such as removal of empty loops, even when termination cannot be proven. –end note \]

Unfortunately, the words “undefined behavior” are not used.  However, anytime the standard says “the compiler may assume P,” it is implied that a program which has the property not-P has undefined semantics.

Notice that in C++, modifying a global (or local) variable is not a side-effecting operation.  Only actions in the list above count.  Thus, there would seem to be a strong possibility that real programmers are going to get burned by this problem.  A corollary is that it is completely clear that a C++ implementation may claim to have disproved Fermat’s Last Theorem when it executes my code.

We can ask ourselves: Do we want a programming language that has these semantics?  I don’t, and I’ll tell you what: if you are a C++ user and you think this behavior is wrong, leave a comment at the bottom of this post or send me an email.  If I get 50 such responses, I’ll formally request that the C++ Standard committee revisit this issue.  I haven’t done this before, but in an email conversation Hans Boehm (who is on the C++ committee) told me:

> If you want the committee to revisit this, all you have to do is to find someone to add it as a national body comment.  That’s probably quite easy.  But I’m not sure enough has changed since the original discussion that it would be useful.

Anyway, let me know.

# Haskell

Haskell has a bottom type that is a subtype of every other type.  Bottom is a type for functions which do not return a value; it corresponds to an error condition or non-termination.  Interestingly, Haskell fails to distinguish between the error and non-terminating cases: this can be seen as trading diagnostic power for speed.  That is, because errors and infinite loops are equivalent, the compiler is free to perform various transformations that, for example, print a different error message than one might have expected.  Haskell users (I’m not one) appear to be happy with this and in practice Haskell implementations appear to produce perfectly good error messages.

# Other Languages

Most programming languages have no explicit discussion of termination and non-termination in their standards / definitions.  In general, we can probably read into this that a language implementation can be expected to preserve the apparent termination characteristics of its inputs. Rupak Majumdar pointed me to [this nice writeup](http://pho.ucsd.edu/rjhala/koenig.pdf) about an interesting interaction between a non-terminating loop and the SML type system.

# C

Ok, let’s talk about termination in C.  I’ve saved this for last not so much to build dramatic tension as because the situation is murky.  As we saw above, the reality is that many compilers will go ahead and generate terminating object code for C source code that is non-terminating at the level of the abstract machine.  We also already saw that this is OK in C++0x and not OK in Java.

The relevant part of the C standard (I’m using [N1124](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1124.pdf)) is found in 5.1.2.3:

> The least requirements on a conforming implementation are:
> - At sequence points, volatile objects are stable in the sense that previous accesses are complete and subsequent accesses have not yet occurred.
> - At program termination, all data written into files shall be identical to the result that execution of the program according to the abstract semantics would have produced.
> - The input and output dynamics of interactive devices shall take place as specified in 7.19.3. The intent of these requirements is that unbuffered or line-buffered output appear as soon as possible, to ensure that prompting messages actually appear prior to a program waiting for input.

Now we ask: Given the Fermat program at the top of this post, is icc or suncc meeting these least requirements?  The first requirement is trivially met since the program contains no volatile objects.  The third requirement is met; nothing surprising relating to termination is found in 7.19.3.  The second requirement is the tricky one.  If it is talking about termination of the program running on the abstract machine, then it is vacuously met because our program does not terminate.  If it is talking about termination of the actual program generated by the compiler, then the C implementation is buggy because the data written into files (stdout is a file) differs from the data written by the abstract machine.  (This reading is due to Hans Boehm; I had failed to tease this subtlety out of the standard.)

So there you have it: the compiler vendors are reading the standard one way, and others (like me) read it the other way.  It’s pretty clear that the standard is flawed: it should, like C++ or Java, be unambiguous about whether this behavior is permitted.

# Does It Matter if the Compiler Terminates an Infinite Loop?

Yes, it matters, but only in fairly specialized circumstances.  Here are a few examples.

The Fermat program is a simple counterexample search.  A more realistic example would test a more interesting conjecture, such as whether a program contains a bug or whether a possibly-prime number has a factorization.  If I happen to write a counterexample search that fails to contain side-effecting operations, a C++0x implementation can do anything it chooses with my code.

Linux 2.6.0 contains this code:

```
NORET_TYPE void panic(const char * fmt, ...)  {
  ... do stuff ...
  for (;;)
    ;
}
```

If the compiler optimizes this function so that it returns, some random code will get executed.  Luckily, gcc is not one of the compilers that is known to terminate infinite loops.  (Michal Nazarewicz found this example.)

In embedded software I’ll sometimes write a deliberate infinite loop.  For example to hang up the CPU if main() returns.  A group using LLVM for compiling embedded code ran into exactly that problem, causing random code to run.

When re-flashing an embedded system with a new code image, it would not be uncommon to hang the processor in an infinite loop waiting for a watchdog timer to reboot the processor into the new code.

Another plausible bit of code from an embedded system is:

```
while (1) {
#ifdef FEATURE_ONE
  do_feature_one();
#endif
#ifdef FEATURE_TWO
  do_feature_two();
#endif
#ifdef FEATURE_THREE
  do_feature_three();
#endif
}
fputs("Internal error\n", stderr);
```

If you compile this code for a product that contains none of the three optional features, the compiler might terminate my loop and cause the error code to run. (This code is from Keith Thompson.)

Finally, if I accidentally write an infinite loop, I’d prefer my program to hang so I can use a debugger to find the problem.  If the compiler deletes the loop and also computes a nonsensical result, as in the Fermat example, I have no easy way to find the latent error in my system.

# Are Termination-Preserving Compilers Uneconomical?

The C and C++ languages have undefined behavior when a signed integer overflows.  Java mandates two’s complement behavior.  Java’s stronger semantics have a real cost for certain kinds of tight loops such as those found in digital signal processing, where undefined integer overflow can buy (I have heard) 50% speedup on some real codes.

Similarly, Java’s termination semantics are stronger than C++0x’s and perhaps stronger than C’s.  The stronger semantics have a cost: the optimizer is no longer free to, for example, move side effecting operations before or after a potentially non-terminating loop.  So Java will either generate slower code, or else the C/C++ optimizer must become more sophisticated in order to generate the same code that Java does.  Does this really matter?  Is is a major handicap for compiler vendors?  I don’t know, but I doubt that the effect would be measurable for most real codes.

# Worse is Better

Richard Gabriel’s classic [Worse is Better](http://www.dreamsongs.com/WorseIsBetter.html) essay gives the example where UNIX has worse semantics than Multics: it permits system calls to fail, forcing users to put them in retry loops.  By pushing complexity onto the user (which is worse), UNIX gains implementation simplicity, and perhaps thereby wins in the marketplace (which is better).  Pushing nonintuitive termination behavior onto the user, as C++0x does, is a pretty classic example of worse is better.

# Hall of Shame

These C compilers known to not preserve termination properties of code: Sun CC 5.10, Intel CC 11.1, LLVM 2.7, Open64 4.2.3, and Microsoft Visual C 2008 and 2010.  The LLVM developers consider this behavior a bug and have since fixed it. As far as I know, the other compiler vendors have no plans to change the behavior.

These C compilers, as far as I know, do not change the termination behavior of their inputs: GCC 3.x, GCC 4.x, and the WindRiver Diab compiler.

# Acknowledgments

My understanding of these issues benefited from conversations with Hans Boehm and Alastair Reid.  This post does not represent their views and all mistakes are mine.
