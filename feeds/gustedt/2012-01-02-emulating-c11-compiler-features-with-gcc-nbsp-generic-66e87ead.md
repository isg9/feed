---
title: Emulating C11 compiler features with gcc:&nbsp;_Generic
url: https://gustedt.wordpress.com/2012/01/02/emulating-c11-compiler-features-with-gcc-_generic/
published: "2012-01-02T16:58:45Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1224
---

# Emulating C11 compiler features with gcc:&nbsp;_Generic

Besides the new interfaces for threads and atomic operations that I already mentioned earlier, others of the new features that come with C11 are in reality already present in many compilers. Only that not all of them might agree upon the syntax, and especially not with the new syntax of C11. So actually emulating some these features is already possible and [I implemented some of them](http://p99.gforge.inria.fr/p99-source/p99-html/group__C11__keywords.html) in [P99](http://p99.gforge.inria.fr) on the base of what gcc provides: these are

- `static_assert` (or `_Static_assert`) to make compile time assertions (a misnomer, again!)
- `alignof` (or `_Alignof`) to get the alignment constraint for a type
- `alignas` (of `_Alignas`) to constraint the alignment of objects or `struct` members. Only the variant that receives a constant expression is directly supported. The variant with a type argument can be obtained simply by `alignas(alignof(T))`.
- `noreturn` (or `_Noreturn`) to specify that a function is not expected to return to the caller
- `thread_local` (or `_Thread_local`) for thread local storage
- `_Generic` for type generic expression.

The most interesting among them is probably the latter feature, type generic expressions.

Gcc already has three builtins that can be used to something like `_Generic`:

- `__typeof__(EXP)` gives the type of the expression `EXP`
- `__builtin_types_compatible_p(T1, T2)` is true if the two types are compatible
- `__builtin_choose_expr(CNTRL, EXP1, EXP2)` choose between the two expressions at compile time.

This only gives binary decisions and not multiple ones like `_Generic` but with a good dose of `P99_FOR` we can come to something that resembles a lot. (I spare you the details of the implementation.) The syntax that we support is as follows

```
#include "p99_generic.h"

#define P99_GENERIC(EXP, DEF, ...)  <some nasty use of P99 and gcc builtins>

```

For example as in the following

```
P99_GENERIC(a,
             , // empty default expression
             (int*, a),
             (double*, x));

```

That is an expression `EXP`, followed by a default value `DEF`, followed by a list of type value pairs. So here this is an expression that depending on the type of a will have a type of `int*` or `double*` that will be set to `a` or `x`, respectively.

In C11 syntax, the above would be coded with some kind of “label” syntax:

```
 _Generic(a,
          int*: a,
          double*: x);

```

As you can see above, the default value can be omitted. If so, it is replaced with some appropriate expression that should usually give you a syntax error.

Here is an example with a default expression that will be used when none of the types matches:

```
uintmax_t max_uintmax(uintmax_t, uintmax_t);
int       max_int(int, int);
long      max_long(long, long);
long long max_llong(long long, long long);
float     max_float(float, float);
double    max_double(double, double);

 a = P99_GENERIC(a + b,
                 max_uintmax,
                 (int, max_int),
                 (long, max_long),
                 (long long, max_llong),
                 (float, max_float),
                 (double, max_double))(a, b);

```

In C11 syntax

```
 a = _Generic(a + b,
              default: max_uintmax,
              int: max_int,
              long: max_long,
              long long: max_llong,
              float: max_float,
              double: max_double)(a, b);

```

Here all the expressions evaluate to a function specifier. If `a + b` is `int`, … or `double` the appropriate maximum function is chosen for that type. If none of these matches, the one for `uintmax_t` is chosen. The corresponding function is then evaluated with `a` and `b` as arguments.

- Because the choice expression is `a + b` its type is the promoted common type of `a` and `b`. E.g for all types that are narrower than `int`, e.g `short`, normally int will be the type of the expression and `max_int` will be the function. If `a` would be `unsigned` and `b` would be `double` the result would be `double` as well.

- The return type of the `_Generic` expression is a function to two arguments. If it would be for `int`, e.g, the type would be `int ()(int, int)`. So the return type of the function call would be `int` in that case.

- The arguments are promoted and converted to the expected type of the chosen function.

NB: if the compiler is already C11 complying, the `P99_GENERIC` expression will just be translated to the corresponding `_Generic` expression.

Otherwise only gcc and compatible compilers are supported.

*Addendum:* Today I checked out the new version of *clang* and discovered that their new version 3.1 that came out in December already supports `_Generic` directly. Well done.
