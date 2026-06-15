---
title: Avoid writing <code>va_arg</code> functions
url: https://gustedt.wordpress.com/2011/07/10/avoid-writing-va_arg-functions/
published: "2011-07-10T23:03:03Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1074
---

# Avoid writing <code>va_arg</code> functions

As we have seen in [another post](https://gustedt.wordpress.com/2010/08/04/va_arg-functions-and-macros/), functions that receive a variable argument list but which are all the same type are better replaced by a function and variadic macro. P99 has easy means to transform an argument list `x0, x2, .., xN` into two parameters the first being just the length of the list, here `N+1` and an array that has the values. These types of functions can usually be much better optimized in place by modern compilers.

So `va_arg` functions should first of all restricted to the case that the function may receive incompatible types as its argument (such as floating point and integers). If used for that the default argument promotion rules may be really [harmful](https://gustedt.wordpress.com/2010/11/07/dont-use-null/).

This is particularly nasty if your function expects some wide type (e.g `size_t`) and from time to time you call the function with a small constant, say `0` or `1`. This constant is them only promoted to `int`, and your program may crash weirdly.

The usage of `va_arg` functions can become quite unintuitive and ugly, `printf` is a particular example. As soon as you have a `typedef` to an integer type we are in troubled water.

Some simple rules in a future revision of C could easily help us out:

- promote all integer argument to `[u]intmax_t`
- promote all data pointer arguments to `void*`
- promote all floating point arguments to `long double`

That’s it. All those problems would disappear. No trap calling a `va_arg` function with a small constant. `printf` would just need some format specifications but no length specification. No need for `PRIxx` macros.

Code that uses `va_arg` macros correctly wouldn’t even break, because converting a value to the maximal integer width and back should result in exactly the same value.

Well, most probably, this will never happen. First, let’s face it, C people are conservative. They will bring forward all kinds of arguments, and when all else fail they will say it gives a performance hit. (Who is implementing a `va_arg` function and expects it to be performance optimal?)

The only incompatibility would be between object code that is compiled with the old and the new model. They wouldn’t link well, respectively crash in production. So everything would have to be recompiled.

As long as such a change will not happen, better avoid `va_arg` functions completely. They are not worth the trouble.
