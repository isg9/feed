---
title: surprising occurrence of identifiers in header&nbsp;files
url: https://gustedt.wordpress.com/2012/02/15/surprising-occurrence-of-identifiers-in-header-files/
published: "2012-02-15T17:35:40Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1320
---

# surprising occurrence of identifiers in header&nbsp;files

I remember being stuck sometime ago because a system header at the time on the platform that I was using defined the undocumented identifier `barrier`. IIRC this even was a macro, which made the bug really hard to track, seemingly harmless code simply exploded.

Hopefully nowadays platform implementors are a bit more careful in not polluting the namespace, but still avoiding naming conflicts is not so easy. E.g `inline` functions are a useful tool when you want to expose small functions to all compilation units of a program. There is one pitfall, though, when it comes to naming conventions for their parameter names and local variables. If you get the name wrong, as in this simple example

```
inline double my_sin(double PHI) { return sinf(PHI); }

```

other users of your code might encounter random problems if they define a macro `PHI`.

This example is doubly bad, not only because it exposes an undocumented identifier but also because that identifier is all uppercase, against usual conventions that reserves such identifiers for preprocessor macros. But in any case, even if we change that to lowercase `phi` we don’t know in what a context the code will be included, so the error just becomes less likely, but it may occur nevertheless. And if it occurs, an error message can be really bizarre, incomprehensible for the poor colleague that tries to use your code.

Standard prototype declarations in header files don’t have that problem. For a good reason C allows you to omit the parameter names for declarations:

```
double my_sin(double);

```

So this then is completely harmless and only exposes the identifier that is declared.

But already the declaration of `struct` types in headers may have the same problem

```
struct myString {
  size_t n;
  char* c;
};

```

has the same problem as the inline function example. An application that `#define` s `n` or `c` will see weird errors.

This is probably the reason why in POSIX all structure fields have additional tags in their names. This makes it even more unlikely that the names collide with a user-defined one.

```
struct in_addr {
  in_addr_t s_addr;
};

```

Here the `in_` and `s_` prefixes make name clashes with user code unlikely.

Some people may say that macros are evil, users that use macros should not be supported. But the funny thing is that macros themselves get the thing right, concerning the identifiers for their parameters:

```
#define MYVAL(X) (double)(X)

```

this only defines `MYVAL` in global scope, `X` is only visible within the macro expansion.

So not the macros are the problem, but the pollution of the namespace with identifiers that are not from a reserved set and that perhaps are not even documented.

The problem occurs for

- parameter names of `inline` functions
- field names of `struct` or `union`
- local variables inside `inline` functions or macros

If you are an implementor of a compiler or C library, e.g, you can get away from this problem by using reserved identifiers. Usually identifiers with two leading underscores are taken for that, e.g such as the gcc extension `__COUNTER__`. We mere mortals don’t have that possibility.

The only choice I see is to fix a (or several) prefixes that you will use for all such identifiers. A lot of such prefixes (or postfixes) are all already used by others, e.g `pthread_`, `_t`, `str`, but you can certainly find some that are still short enough and have a low probability to be used by others.

In P99 we have the luxurious waste of four prefixes `P99_`, `P00_`, `p99_` and `p00_`, where the ’99’ identifiers are those that are supposed to be used by our users and the ’00’ prefixes are those that mark names (parameters, macros, …) that are only of internal use.
