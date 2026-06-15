---
title: Overflows in SafeInt
url: https://blog.regehr.org/archives/593
published: "2011-09-22T15:17:00Z"
feed: regehr
guid: http://blog.regehr.org/?p=593
---

# Overflows in SafeInt

**Update from Friday 9/23:** The SafeInt developers have already uploaded a new version that fixes the problems described in this post. Nice!

I have a minor obsession with [undefined behaviors in C and C++](https://blog.regehr.org/archives/213). Lately I was tracking down some integer overflows in Firefox — of which there are quite a few — and some of them seemed to originate in the well-known [SafeInt](http://safeint.codeplex.com/) library that it uses to avoid performing unsafe integer operations. Next, I poked around in the latest version of SafeInt and found that while executing its (quite good) test suite there are 43 different places in the code where an undefined integer operation is performed. A substantial number of them stem from code like this:

> ```
> bool negative = false;
> if (a<0) {
>   a = -a;
>   negative = true;
> }
>
> ... assume a is positive ...
>
> if (negative) {
>   return -result;
> } else {
>   return result;
> }
>
> ```

To see a real example of this, take a look at the code starting at line 2100 of [SafeInt3.hpp](http://safeint.codeplex.com/releases/view/71431). The problem here, of course, is that for the most common choices of implementation-defined behaviors in C++, negating INT\_MIN is an undefined behavior.

Now we have to ask a couple of questions. First, does this code operate properly if the compiler happens to implement -INT\_MIN using 2’s complement arithmetic? I’m not sure about all of the overflows in SafeInt, but I believe this function actually does work in that case. The second question is, do compilers (despite not being required to do so) give 2’s complement semantics to -INT\_MIN? Every compiler I tried has this behavior when optimizations are disabled. On the other hand, not all compilers have this behavior when optimizations are turned on. A simple test you can do is to compile this function with maximum optimization:

> ```
> void bar (void);
>
> void foo (int x) {
>  if (x<0) x = -x;
>  if (x<0) bar();
> }
> ```

If the resulting assembly code contains no call to bar(), then the compiler has (correctly) observed that every path through this function either does not call bar() or else relies on undefined behavior. Once the compiler sees this, it is free to eliminate the call to bar() as dead code. Most of the compilers I tried — even ones that are known to exploit other kinds of integer undefined behavior — don’t perform this optimization. However, here’s what I get from a recent GCC snapshot:

> ```
> [regehr@gamow ~]$ current-gcc -c -O2 overflow2.c
> [regehr@gamow ~]$ objdump -d overflow2.o
> 0000000000000000 <foo>:
>  0:    f3 c3                    repz retq
> ```

Now, does this same optimization ever fire when compiling SafeInt code, causing it to return a wrong result? Here’s a bit of test code:

> ```
> #include <cstdio>
> #include <climits>
> #include "SafeInt3.hpp"
>
> void test (__int32 a, __int64 b) {
>   __int32 ret;
>   bool res = SafeMultiply (a, b, ret);
>   if (res) {
>     printf ("%d * %lld = %d\n", a, b, ret);
>   } else {
>     printf ("%d * %lld = INVALID\n", a, b);
>   }
> }
>
> int main (void) {
>   test (INT_MIN, -2);
>   test (INT_MIN, -1);
>   test (INT_MIN, 0);
>   test (INT_MIN, 1);
>   test (INT_MIN, 2);
>   return 0;
> }
> ```

Next we compile it with the recent g++ at a couple of different optimization levels and run the resulting executables:

> ```
> [regehr@gamow safeint]$ current-g++ -O1 -Wall safeint_test.cpp
> [regehr@gamow safeint]$ ./a.out
> -2147483648 * -2 = INVALID
> -2147483648 * -1 = INVALID
> -2147483648 * 0 = 0
> -2147483648 * 1 = -2147483648
> -2147483648 * 2 = INVALID
> [regehr@gamow safeint]$ current-g++ -O2 -Wall safeint_test.cpp
> [regehr@gamow safeint]$ ./a.out
> -2147483648 * -2 = INVALID
> -2147483648 * -1 = INVALID
> -2147483648 * 0 = 0
> -2147483648 * 1 = INVALID
> -2147483648 * 2 = INVALID
> ```

The first set of results is correct. The second set is wrong for the INT\_MIN \* 1 case, which should not overflow. Basically, at -O2 and higher gcc and g++ turn on optimization passes that try to generate better code by taking integer undefined behaviors into account. Let’s be clear: this is not a compiler bug; g++ is simply exploiting a bit of leeway given to it by the C++ standards.

What can we take away from this example?

- It’s a little ironic that the SafeInt library (a widely used piece of software, and not a new one) is itself performing operations with undefined behavior. This is not the only safe math library I’ve seen that does this — it is simply very hard to avoid running afoul of C/C++’s integer rules, particularly without good tool support.
- It’s impressive that G++ was able to exploit this undefined behavior. GCC is getting to be a very strongly optimizing compiler and also SafeInt was carefully designed to not get in the way of compiler optimizations.

If you have security-critical code that uses SafeInt to manipulate untrusted data, should you be worried? Hard to say. I was able to get SafeInt to malfunction, but only in a conservative direction (rejecting a valid operation as invalid, instead of the reverse) and only using a recent G++ snapshot (note that I didn’t try a lot of compilers — there could easily be others that do the same thing). Also, there are 42 more integer overflow sites that I didn’t look at in detail. One way to be safe would be to use GCC’s -fwrapv option, which forces 2’s complement semantics for signed integer overflows. Clang also supports this option, but most other compilers don’t have an equivalent one. Also unfortunately, SafeInt is structured as a header file instead of a true library, so it’s not like just this one file can be recompiled separately.

This issue has been reported [here](http://safeint.codeplex.com/workitem/14278).

**Update:** I checked [CERT’s IntegerLib](https://www.securecoding.cert.org/confluence/display/cplusplus/INT03-CPP.+Use+a+secure+integer+library) (download [here](http://www.cert.org/secure-coding/IntegerLib.zip)) and it is seriously broken. Its own test suite changes behavior across -O0 … -O3 for each of Clang, GCC, and Intel CC. The undefined behaviors are:

> **UNDEFINED at <add.c, (24:12)> : Op: +, Reason : Signed Addition Overflow**
>
> **UNDEFINED at <add.c, (24:28)> : Op: +, Reason : Signed Addition Overflow**
>
> **UNDEFINED at <add.c, (29:12)> : Op: +, Reason : Signed Addition Overflow**
>
> **UNDEFINED at <add.c, (43:14)> : Op: +, Reason : Signed Addition Overflow**
>
> **UNDEFINED at <add.c, (43:30)> : Op: +, Reason : Signed Addition Overflow**
>
> **UNDEFINED at <add.c, (47:14)> : Op: +, Reason : Signed Addition Overflow**
>
> **UNDEFINED at <add.c, (61:12)> : Op: +, Reason : Signed Addition Overflow**
>
> **UNDEFINED at <add.c, (61:28)> : Op: +, Reason : Signed Addition Overflow**
>
> **UNDEFINED at <add.c, (65:15)> : Op: +, Reason : Signed Addition Overflow**
>
> **UNDEFINED at <mult.c, (52:13)> : Op: \*, Reason : Signed Multiplication Overflow**
>
> **UNDEFINED at <mult.c, (69:47)> : Op: \*, Reason : Signed Multiplication Overflow**
>
> **UNDEFINED at <sub.c, (24:23)> : Op: -, Reason : Signed Subtraction Overflow**
>
> **UNDEFINED at <sub.c, (28:12)> : Op: -, Reason : Signed Subtraction Overflow**
>
> **UNDEFINED at <sub.c, (42:23)> : Op: -, Reason : Signed Subtraction Overflow**
>
> **UNDEFINED at <sub.c, (46:12)> : Op: -, Reason : Signed Subtraction Overflow**
>
> **UNDEFINED at <sub.c, (60:23)> : Op: -, Reason : Signed Subtraction Overflow**
>
> **UNDEFINED at <sub.c, (64:12)> : Op: -, Reason : Signed Subtraction Overflow**
>
> **UNDEFINED at <unary.c, (28:9)> : Op: -, Reason : Signed Subtraction Overflow**
>
> **UNDEFINED at <unary.c, (37:9)> : Op: -, Reason : Signed Subtraction Overflow**
>
> **UNDEFINED at <unary.c, (46:9)> : Op: -, Reason : Signed Subtraction Overflow**

I cannot imagine this library is suitable for any purpose if you are using an optimizing compiler.

**Update from Friday 9/23:** I looked into the problems in IntegerLib in more detail. The different output across different compilers and optimization levels isn’t even due to the integer problems — it’s due to the fact that a couple of functions have no “return” statement, and therefore return garbage. Do not use IntegerLib! Use SafeInt.
