---
title: Why Not Mix Signed and Unsigned Values in C/C++?
url: https://blog.regehr.org/archives/268
published: "2010-09-15T15:35:00Z"
feed: regehr
guid: http://blog.regehr.org/?p=268
---

# Why Not Mix Signed and Unsigned Values in C/C++?

Most C/C++ programmers have been told to avoid mixing signed and unsigned values in expressions. However — at least in part because we usually follow this advice — many of us are not totally on top of the underlying issues. This program illustrates what can go wrong:

> ```
> #include <stdio.h>
> int main (void)
> {
>   long a = -1;
>   unsigned b = 1;
>   printf ("%d\n", a > b);
>   return 0;
> }
> ```

Here’s what happens on an x86-64 Linux box:

> ```
> [regehr@gamow ~]$ gcc compare.c -o compare
> [regehr@gamow ~]$ ./compare
> 0
> [regehr@gamow ~]$ gcc -m32 compare.c -o compare
> [regehr@gamow ~]$ ./compare
> 1
> ```

In other words, the inequality is false on x64 and true on x86. If this doesn’t give you at least one brief moment of “WTF?” then you’re doing a lot better than I did the first time I saw something like this happen.

The underlying issue is a feature interaction. The first feature is C’s integer promotion strategy, which preserves values but often does not preserve signedness. Usually, the arguments to any arithmetic operator are promoted to signed int — or to a larger signed type, if necessary, to make the operands have the same size. However, if the type contains values not representable in the signed promoted type, the promoted type is instead unsigned. For example, unsigned char and unsigned short can both be promoted to int because all of their values can be represented in an int. On the other hand, unsigned int cannot be promoted to int because (assuming ints are 32 bits) values like 2147483648 are not representable.

The second feature is C’s method for choosing which version of an operator to use. Although the greater-than operator in C always looks like “>”, behind the scenes there are quite a few different operators: signed integer >, unsigned integer >, signed long >, unsigned long >, etc. If either operand to “>” is unsigned, then an unsigned comparison is used, otherwise the comparison is signed.

Now, to return to the example: on a 64-bit platform, b is promoted to signed long and the signed “>” is used. On a 32-bit platform, because “int” and “long” are the same size, b remains unsigned, forcing the unsigned “>” to be used. This explains the reversal of the sense of the comparison.

Sign problems can sneak into code in two additional ways. First, it’s easy to forget that constants are signed by default, even when they are used in a context that seems like it should be unsigned. Second, the result of a comparison operator is always an int: a signed type.

Once we’re aware of these issues, it’s not hard to think through a puzzle like the one above. On the other hand, it can be very difficult to debug this kind of problem in a large piece of software, especially since sign problems are probably not the first item on our list of suspected root causes for a bug.

When writing new software, it’s definitely prudent to turn on compiler warnings about sign problems. Unfortunately, GCC 4.4 doesn’t warn about the program above even when given the -Wall option. The -Wsign-compare option does give a warning, but only when generating 32-bit code. When generating 64-bit code, there’s no warning since b is promoted to a signed type before being exposed to the “>” operator. So if we’re primarily developing on the 64-bit platform, the problem may remain latent for a while.

Just to make things extra confusing, [one time I tracked down a problem](https://bugs.launchpad.net/ubuntu/hardy/+source/gcc-4.2/+bug/256797) where the version of GCC that was released as part of Ubuntu Hardy for x86 miscompiled a function very similar to the one above. It took this program and compiled it to return 1:

> ```
> int foo (void) {
>   signed char a = 1;
>   unsigned char b = -1;
>   return a > b;
> }
> ```

Here both values should be promoted to signed int and the comparison is then (1 > 255). Obviously, in this case the compiler was buggy. The base version of GCC, 4.2.2, does not have this bug. However, the Ubuntu people applied about 5 MB of patches to this compiler before packaging it up and somehow broke it. A few years ago I found similar bugs in [CIL](http://www.cs.berkeley.edu/~necula/cil/) and in an early version of Clang. Apparently even compiler developers are not immune to signed/unsigned confusion.
