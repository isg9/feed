---
title: A Variable Argument Hazard
url: https://blog.regehr.org/archives/1303
published: "2016-03-09T09:56:35Z"
feed: regehr
guid: http://blog.regehr.org/?p=1303
---

# A Variable Argument Hazard

Variable argument functions in C and C++ can be tricky to use correctly, and they typically only get compiler-based type checking for special cases such as printf(). Recently I ran across an entertaining variable argument bug using [tis-interpreter](http://trust-in-soft.com/tis-interpreter/).

Although I will use code from SQLite to illustrate this bug, this bug has not been found in SQLite itself. Rather, it was in some code used to test that database.

SQLite’s API has a configuration function that takes variable arguments:

```
int sqlite3_config(int op, ...);

```

Several of its configuration commands, such as this one, expect pointer-typed arguments:

```
case SQLITE_CONFIG_LOG: {
  typedef void(*LOGFUNC_t)(void*,int,const char*);
  sqlite3GlobalConfig.xLog = va_arg(ap, LOGFUNC_t);
  sqlite3GlobalConfig.pLogArg = va_arg(ap, void*);
  break;
}

```

We’re only allowed to execute this code when the first two variable arguments are respectively compatible with LOGFUNC\_t and void \*, otherwise the behavior is undefined.

The test code contains calls that look like this:

```
sqlite3_config(SQLITE_CONFIG_LOG, 0, pLog);

```

At first glance there’s no problem — normally it is fine to pass a 0 to a function that expects a pointer argument: the 0 is turned into an appropriately-typed null pointer. On the other hand, when a function takes variable arguments (or when it lacks a prototype) the compiler cannot do this kind of caller-side type conversion and instead a different set of rules, the “default argument promotions” are applied, resulting in a zero-valued int being passed to sqlite3\_config() in the first variable argument position. Another fun aspect of the default argument promotions is that they force all float arguments to be promoted to double. These rules seem to be the same in C++ though they would fire less often due to C++’s stricter rules about function prototypes.

This problem was not noticed because the code happens to be compiled benignly on both x86 and x64 platforms. On x64 Linux, pointers and ints are not the same size, but the calling convention says that the first six integer and pointer arguments are passed in 64-bit registers — so the problem gets covered up by an implicit promotion to 64 bits. [The ABI](http://www.x86-64.org/documentation/abi.pdf) does not guarantee that the higher bits are zeroed in this case, but in this example they happen to be cleared by GCC and LLVM.

On x86 Linux, all arguments are passed in memory but since pointers and integers are the same size, the problem is again covered up. An ABI that represents a null pointer differently than a zero-valued long integer would expose this bug right away, as would an x64 program that passes at least six variable arguments. In any case, the fix is easy:

```
sqlite3_config(SQLITE_CONFIG_LOG, (LOGFUNC_t)0, pLog);

```

In summary, be careful with the default argument promotions that are done when calling variable argument functions.

Thanks to Pascal Cuoq who provided comments on this piece.

Also see [this article by Rich Felker](http://ewontfix.com/11/) and [this one by Jens Gustedt](https://gustedt.wordpress.com/2010/11/07/dont-use-null/).
