---
title: What Other Dynamic Checkers for C/C++ are Needed?
url: https://blog.regehr.org/archives/966
published: "2013-07-09T22:05:32Z"
feed: regehr
guid: http://blog.regehr.org/?p=966
---

# What Other Dynamic Checkers for C/C++ are Needed?

Detectors for memory-related undefined behaviors in C/C++, while being imperfect, are at least [something that smart people have spent a lot of time thinking about](https://blog.regehr.org/archives/939). Integer-related undefined behaviors have received much less attention, but then again they are a lot simpler than memory unsafety.

Today’s question is: What other checkers should we create? Here are a few. I’m not sure that any of these are really pressing problems in practice, but I think it would be far from useless to test important C/C++ codes under these tools. I’d be happy to hear suggestions for other tools that might be useful.

This discussion of [undefined behavior checkers for Clang](https://docs.google.com/document/d/1o7Xw6dohIuHLOve3hxtTjCrrUbd2NHUgmq_WGrao6js/edit#heading=h.pgx8h2ru49tm) is useful.

# Unspecified order of evaluation of arguments to a function

When you write something like `f(g(),h())`, the arguments can be evaluated in any order. This matters when the arguments perform I/O or when they modify overlapping state.

Writing a rigorous tool for dynamically finding programs whose behavior is sensitive to argument evaluation order would be difficult and its performance would likely be bad. In contrast, it would be simple to hack a compiler to randomize (either at compile time or at run time) the order in which arguments are evaluated, at which point we would simply look for test cases to fail when the option to do this is turned on.

In a sensible world, left-to-right evaluation would be specified by the standards. Failing that, individual compilers should specify this. Sometimes people say that the unspecified order of evaluation permits the generation of more efficient code but it is very hard to believe that this effect is ever usefully exploited. A recent version of gcc that I checked consistently evaluates arguments right-to-left; a recent Clang consistently evaluates them left-to-right. Older versions of gcc would switch orders depending on optimization option.

# Unsequenced modifications to lvalues

Modifying an object multiple times within a C/C++ expression, or accessing an lvalue “other than to determine the new value” is undefined. Here’s a program with unsequenced modifications to `x`:

```
#include <stdio.h>
#include <limits.h>

int main (void)
{
  unsigned x;
  unsigned *p = &x;
  x = 0;
  printf ("%u\n", x++ + --x);
  return 0;
}

```

And the result:

> **```** **$ gcc -w side_effect.c ; ./a.out** **4294967295** **$ gcc -w -O side_effect.c ; ./a.out** **4294967294** **$ clang -w side_effect.c ; ./a.out** **0**
>
> **```**

For some examples (such as this program) GCC and Clang provide good warnings, but minor modifications such as accessing a variable through a pointer will suppress the warnings without making the underlying problem go away.

I think a dynamic checker for this kind of undefined behavior could be pretty efficient because the total number of objects touched while evaluating an expression is usually pretty small.

# Strict aliasing violations

The strict aliasing rules in C/C++ require basically that if you cast a pointer into any kind of pointer besides `char *`, you don’t dereference the new pointer. Unfortunately I’m not sure how to check for violations short of a full type safety implementation, which is no fun for anyone.

Compiler warnings for strict aliasing violations exist, but seem to be unpredictable. For example, neither GCC nor Clang warns about either `c1()` or `check()` [in this post](https://blog.regehr.org/archives/959). These are both extremely simple and obvious violations of strict aliasing. So it’s hard to see how these compilers could be providing robust warnings about more difficult examples found in real code.
