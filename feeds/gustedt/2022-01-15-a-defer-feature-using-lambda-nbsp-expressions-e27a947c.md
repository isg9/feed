---
title: A defer feature using lambda&nbsp;expressions
url: https://gustedt.wordpress.com/2022/01/15/a-defer-feature-using-lambda-expressions/
published: "2022-01-15T18:25:00Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3814
---

# A defer feature using lambda&nbsp;expressions

In the previous post [https://gustedt.wordpress.com/2020/12/14/a-defer-mechanism-for-c/](https://gustedt.wordpress.com/2020/12/14/a-defer-mechanism-for-c/) I already presented an idea for a defer feature for C, namely a feature that would provide a simple mechanism to cleanup at the end of a block, regardless how that block is left. There are many different extension out there that provide such a feature, for example POSIX’ `pthread_cleanup_push` and `pthread_cleanup_pop`, Microsoft’s `__try` and `__finally` and gcc’s `cleanup` attribute. Although they are used quite often, none of these extension has yet been adopted widely enough to prefer it over the others for standardization.

In a new paper

> [A simple defer feature for C](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2895.htm)

we present a much simpler version of a unifying framework for such cleanup features which is conditionned to the acceptance of lambdas into C23:

**defer** *lambda-expression;*

Using lambda expressions for this feature helps to clarify which evaluation stratgy is applied for the code inside the depending block. In fact, it has been much debated if variables used in the depending block should be evaluated at the point of the **defer** itself, or at the end when it is executed. This debate has been as fruiteless as for lambdas, compound expressions or nested functions: there is not one “natural” approach for that in C. For lambdas WG21 (C++) has been well advised to leave this entirely to the user of the feature, namely to the specification of the capture clause of the lambda. So using a lambda expression for **defer** just uses that same strategy, the variables from the surrounding scope that are used by a **defer** will simply depend on the capture clause of the lambda expression. (For the terminlogy for lambdas and their captures that I apply see [the latest lambda proposal for C23](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2892.pdf).)

In the following example, the values of `p`, code>q and `r` are used as arguments to `free` at the end of the execution of `main`. Because the corresponding capture is a shadow capture, for `p` the initial value is used as argument to the call; for `q` it is an identifier capture and thus the value is used that was stored last before a `return` statement is met or the execution of the function body ends. Similarly, for `r` the value capture `rp` has the address of `r` and frees the last allocation for which the address was stored in `r`. The four `return` statements are all valid and according to the control flow that is taken the function executes `0`, `1`, `2`, or `3` defer callbacks. If at least the first three allocations are successful, the storage is freed in the order `r`, `q` and `p`. If the call to `realloc` fails, the initial value of `q` is passed as argument to `free`.

```
#include <stdlib.h>
int main(void) {
   double*const p = malloc(sizeof(double[23]));
   if (!p) return EXIT_FAILURE;
   defer [p]{ free(p); };

   double* q = malloc(sizeof(double[23]));
   if (!q) return EXIT_FAILURE;
   defer [&q]{ free(q); };

   double* r = malloc(sizeof(double[23]));
   if (!r) return EXIT_FAILURE;
   defer [rp = &r]{ free(*rp); };
   {
       double* s = realloc(q, sizeof(double[32]));
       if (s) q = s;
       else return EXIT_FAILURE;
   }
}

```

Because lambda expressions are “just” values, it is much more natural to model this version of the **defer** feature as a *defer declaration*. If you have a background in C++ you can simply think of it as a declaration of an unamend object of lambda type for which the destructor does nothing else than to call the lambda with no parameters. In fact, it is a nice exercise to implement such a feature in C++ by declaring a hidden variable of a specific class type that has a constructor (from a lambda or so) and a destructor as only members, and then wrap such a thing in a macro (yes!) named **defer**.

Other properties of defer declarations are left open and are specified to be implementation defined:

- Is **defer** allowed in any block or just for the function body itself?
- What happens for preliminary exits of the thread or of the whole execution?
- What happens with signals?

For example, for the first an imlementation that would solely build on gcc’s `cleanup` attribute could choose to restrict **defer** to the toplevel block of the function.

Also this proposal puts asside other related features that were discussed in the orginal paper such as `panic` and `recover`. Such extensions, that not yet have so much implementation experience in C, may be left to a new technical specification (TS) that WG14 prospects for this whole feature.
