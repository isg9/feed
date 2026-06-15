---
title: C Compilers Disprove Fermat&#8217;s Last Theorem
url: https://blog.regehr.org/archives/140
published: "2010-04-29T03:19:13Z"
feed: regehr
guid: http://blog.regehr.org/?p=140
---

# C Compilers Disprove Fermat&#8217;s Last Theorem

*\[Update: [I wrote another post on this topic that may explain the underlying issues more clearly](https://blog.regehr.org/archives/161).\]*

Obviously I’m not serious: compilers are bad at solving high-level math problems and also there is good reason to believe this theorem cannot be disproved. But I’m getting ahead of myself. Recently — for reasons that do not matter here — I wanted to write C code for an infinite loop, but where the compiler was not to understand that the loop was infinite.  In contrast, if I had merely written

```
  while (1) { }
```

or

```
  for (;;) { }
```

most optimizing compilers would see the loop’s inability to exit and generate code accordingly. For example, given this C code:

```
  void foo (void)
  {
    for (;;) { }
    open_pod_bay_doors();
  }

```

Most compilers will emit something like this:

```
  foo:
    L2: jmp L2
```

In this case the compiler emits neither the call to open\_pod\_bay\_doors() nor the final return instruction because both are provably not executed.

Perhaps interestingly, LLVM/Clang recognizes that this slightly obfuscated infinite loop never exits:

```
  unsigned int i = 0;
  do {
    i+=2;
  } while (0==(i&1));

```

Faced with a loop optimizer that has some brains, I decided to stop messing around and wrote a loop that should thwart any compiler’s termination analysis:

```
  const int MAX = 1000;
  int a=1,b=1,c=1;
  while ((c*c*c) != ((a*a*a)+(b*b*b))) {
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

```

This loop only terminates if it finds a counterexample to a special case of Fermat’s Last Theorem.  Fermat’s Last Theorem, of course, states that no solution exists for the equation an \+ bn = cn for positive integers a, b, and c and for integer n>2. Here n=3 and a,b,c are in the range \[1..1000\]. On a platform with 32-bit integers 1000 is a reasonable maximum because 2\*10003 is not much less than 231.

It turns out that when optimizations are enabled, several compilers (LLVM/Clang 2.7, Open64-x86 4.2.3, Sun CC 5.10, and Intel CC 11.1.072) go ahead and permit this loop to terminate.  Specifically, when the loop is enclosed in a function, the compilers emit x86 assembly which looks something like this:

```
  fermat:
    ret

```

The implication, of course, is that the compiler has disproved Fermat’s Last Theorem.  Faced with this incredible mathematical discovery, I held my breath and added a line of code at the end of the function to print the counterexample: the values of a, b, and c.  Unfortunately, with their bluffs called in this fashion, all of the compilers emitted code that actually performed the requested computation, which of course does not terminate.  I got the feeling that these tools — like Fermat himself — had not enough room in the margin to explain their reasoning.

What is really going on here is that compiler optimizations and termination analysis have long been at odds.  In other words, if compilers were obligated to preserve the termination properties of the code they translate, they would be unable to perform (or would have more difficulty performing) some of the optimizations that they use to create efficient code.  A choice is being made by compiler developers — probably consciously, though it’s hard to be sure — to prefer speed over correctness. The news, however, is not all bad: Microsoft’s C compiler, the Wind River Diab C compiler, and several versions of GCC all did the right thing, changing the termination properties of none of the examples I tried.

**Update from Sat 5/1:** It turns out the LLVM folks have been working on this problem lately and their latest SVN now does not contain this bug.  Very nice!

**Update from Sat 5/1:** Someone on Reddit noticed (and one of my students confirmed) that the Microsoft compilers do have termination bugs.  The compilers in both Visual Studio 2008 and 2010 generate code for the Fermat function, but then calls to this function are dropped because it is believed to be free of side effects (this was exactly what LLVM did before they fixed the problem).

# Update from Friday 4/30

I’ll try to clarify a few of the questions that have come up on Reddit and in the comments here.  Also I fixed a mistake in the statement of Fermat’s Last Theorem that someone on Reddit pointed out.  Thanks!

**Q: Does this actually matter at all?**

**A:** Yes, but in very specialized situations that usually only come up when developing embedded software.  One example is described [here](http://llvm.org/bugs/show_bug.cgi?id=965#c10): the poster wants the program being simulated to hang when main() exits, but LLVM deletes the loop that was intended to hang up the processor.  The workaround was to compile the code with optimizations turned off.  Another example happens when an embedded system has updated its firmware and wants to do nothing until the watchdog timer reboots the processor into the new version.  It’s no coincidence that gcc and the Wind River C compiler — both of which are heavily used in the embedded world — get termination right.

**Q: Since infinite loops are bad style, isn’t it OK for the compiler to terminate them?  Shouldn’t people be putting the CPU to sleep, blocking the running thread, or whatever?**

**A:** First, not all programs have an operating system or even a threading system to call out to. Embedded software commonly runs on the bare metal.  Second, the meaning of a program is defined by the language standard and style has nothing to do with it.  See my earlier post [The Compiler Doesn’t Care About Your Intent](https://blog.regehr.org/archives/47).

**Q: Does the C standard permit/forbid the compiler to terminate infinite loops?**

**A:** The compiler is given considerable freedom in how it implements the C program, but its output must have the same externally visible behavior that the program would have when interpreted by the “C abstract machine” that is described in [the standard](http://www.open-std.org/JTC1/SC22/wg14/www/docs/n1124.pdf).  Many knowledgeable people (including me) read this as saying that the termination behavior of a program must not be changed.  Obviously some compiler writers disagree, or else don’t believe that it matters.  The fact that reasonable people disagree on the interpretation would seem to indicate that the C standard is flawed.  In contrast, [the Java language definition is quite clear that infinite loops may not be terminated by the JVM](http://java.sun.com/docs/books/jls/third_edition/html/memory.html#17.4.9).

**Q: Are you saying the compiler should do termination analysis?  That’s impossible by trivial reduction to the halting problem.**

**A:** Termination analysis does not need to be part of the compiler at all. However, I (and others) would claim that the compiler should perform a termination analysis of any useless loop before deleting it.  Although the general problem is not computable, many specific instances can be easily solved.

**Q: Does the Fermat code in this post execute any signed integer overflows or other undefined behaviors?**

**A:** I don’t believe so.

# Update from Saturday 5/1

**Q: Didn’t you know Fermat’s Last Theorem was proved in 1995?**

**A:** I did know that.  Since I got my math degree in 1995, it would have been very difficult for me to miss this event :). I was making a weak joke and also referring to the fact that proofs, especially complicated ones, can contain errors. In fact, as someone noted in the comments, Wiles’ initial proof was wrong. Also note that the n=3 special case was proved much earlier, in 1770.

**Q: What’s the best workaround if you really want an infinite loop in C?**

**A:** As several people have pointed out, looping on a volatile-qualified variable is probably the best choice. But keep in mind that [compilers don’t always respect volatile](http://www.cs.utah.edu/~regehr/papers/emsoft08-preprint.pdf)….

# One more update from Saturday 5/1

Here’s a fun complete program that is more compelling than the code above because it explicitly uses a return value from the “theorem disproved” branch of the code:

```
int fermat (void)
{
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

#include <stdio.h>

int main (void)
{
  if (fermat()) {
    printf ("Fermat's Last Theorem has been disproved.\n");
  } else {
    printf ("Fermat's Last Theorem has not been disproved.\n");
  }
  return 0;
}

```

Here’s what the Intel and Sun compilers have to say:

```
regehr@john-home:~$ icc fermat2.c -o fermat2
regehr@john-home:~$ ./fermat2
Fermat's Last Theorem has been disproved.
regehr@john-home:~$ suncc -O fermat2.c -o fermat2
"fermat2.c", line 20: warning: statement not reached
regehr@john-home:~$ ./fermat2
Fermat's Last Theorem has been disproved.
```

Open64-x86 and LLVM/Clang 2.7 have the same behavior.  Although plenty of folks in the peanut gallery disagree, it seems perfectly clear to me that this is a serious error in these compilers.  I mean, why return 1?  Is that worse or better than returning 0?  Neither result makes any sense.
