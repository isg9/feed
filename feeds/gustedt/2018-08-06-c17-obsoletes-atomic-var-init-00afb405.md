---
title: C17 obsoletes ATOMIC_VAR_INIT
url: https://gustedt.wordpress.com/2018/08/06/c17-obsoletes-atomic_var_init/
published: "2018-08-06T16:19:34Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3661
---

# C17 obsoletes ATOMIC_VAR_INIT

[C17](https://www.iso.org/standard/74528.html) is only a “bugfix” release of the C standard, with one exception, the changes concerning `ATOMIC_VAR_INIT` are a normative.

In C11, `ATOMIC_VAR_INIT` had to be used as an initializer for an atomic object. Several defect reports remarked that the working of this was not clear at all.

1. The default initialization of atomic objects, such as static variables was not clear.
2. `ATOMIC_VAR_INIT` was specified to receive a *value* as its argument. But not for `struct` there are no constant values that could be uniformly used in file scope.

When discussing these problems, the committee noted that all known implementations of C11 atomics did not need the macro for an initialization. Standard initializers just as for any other data type could be used. It was thus decided to remove the obligation to use the macro for an initialization. Section 7.17.2 p2, second sentence now reads:

> An atomic object with automatic storage duration that is not explicitly initialized using ATOMIC\_VAR\_INIT is initially in an indeterminate state; however, the default (zero) initialization for objects with static or thread-local storage duration is guaranteed to produce a valid state.

By that small change atomic variables are not treated any different from other variables anymore. If they are explicitly or implicitly initialized they have a well defined state. Also since now atomic variables are like anybody else, the initial *value* in case of default initialization is `0`.

Because now the macro is basically useless, a note was added to 7.31.8, future library directions for `stdatomic.h`:

> The macro `ATOMIC_VAR_INIT` is an obsolescent feature.

---

- [A list of the changes in C17.](https://gustedt.wordpress.com/2018/04/17/c17/)
