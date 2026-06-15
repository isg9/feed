---
title: testing compile time constness and null pointers with C11&#8217;s&nbsp;_Generic
url: https://gustedt.wordpress.com/2013/08/22/testing-compile-time-constness-and-null-pointers-with-c11s-_generic/
published: "2013-08-22T13:23:27Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1973
---

# testing compile time constness and null pointers with C11&#8217;s&nbsp;_Generic

Sometimes in C it is useful to distinguish if an expression is an “integral constant expression” or a “null pointer constant”. E.g for an object that is allocated statically, only such expressions are valid initializers. Usually we are able to determine that directly when writing an initializer, but if we want to initialize a more complicated `struct` with a function like initializer macro, with earlier versions of C we have the choice:

- Use a compiler extension such as gcc’s `__builtin_constant_p`
- We’d have to write two different versions of such a macro, one for static allocation and one for automatic.

In the following I will explain how to achieve such a goal with C11’s `_Generic` feature. I am not aware of a C++ feature that provides the same possibilities. Also, this uses the ternary operator (notably different in C and C++), so readers that merely come from that community should read the following with precaution.

With a dummy type like this:

```
typedef struct p00_nullptr_test p00_nullptr_test;

```

an expression like the following will evaluate to a compile time constant of value `0` or `1`, according to whether `Y` is a nullpointer constant:

```
_Generic((1 ? (p00_nullptr_test*)0 : (void*)(Y)),
         p00_nullptr_test*: true,
         default: false)

```

Or this will evaluate `myFunc(Y)` iff `Y` is a constant integer expression:

```
_Generic((1 ? (p00_nullptr_test*)0 : (void*)((Y)-(Y))),
         p00_nullptr_test*: 0,
         default: myFunc(Y))

```

To explain how such a thing can work, we first have to do a little digression to the mentioned ternary operator. Take the expression in the following as an exercise to test your C skills.

```
(1 ? (p00_nullptr_test*)X : (void*)Y)

```

The value of the expression is clearly the value of `X`, but what is its type?

Probably you figured it out: it depends, more precisely it depends on `Y`, but not on `X`. But what is even more curious it doesn’t only depend on the type of `Y`, which is cast away anyhow, but on its value: if Y is a nullpointer constant, and only then, the return type of the ternary expression is `p00_nullptr_test*`, otherwise it is `void*`.

```
(void*)Y

```

may or may not be a null pointer constant in the sense of the C standard. If `Y` is an integer constant of value `0` it is one, if `Y` is of non-integer and non- `void*` type or its value is not `0` it is not a null pointer constant.

Permitted for the first case is an integer literal (e.g `0`, `0x0`, `'` `\` `0` `'`), an enumeration constant of value `0` ( `enum { zero }`), or a constant expression composed of literals, enumeration constants and `sizeof` expressions that evaluates to `0`.

If `Y` is any of the following, the result is not a nullpointer constant:

- `1` (value is not `0`)
- `a` (for any variable `a`, even `const`-qualified, of value `0`)
- `(int*)0` (because the pointer that is converted is not `void*`)
- `(void const*)0` (for the same reason, qualified versions of `void` are not allowed)

Now let’s go back to our first type generic expression

```
_Generic((1 ? (p00_nullptr_test*)0 : (void*)(Y)),
         p00_nullptr_test*: true,
         default: false)

```

and look how it works. As said if `Y` is a nullpointer constant, the ternary expression has type `p00_nullptr_test*` and the first choice triggers. In all other cases the ternary has type `void*` and the default is chosen.

For the second type generic expression an additional reasoning applies. For any arithmetic expression `(Y)-(Y)` always has value `0`, but it only is a nullpointer constant if `Y` is a constant integral expression.

For [P99](http://p99.gforge.inria.fr "P99") I wrap some of this into macros that are ready to use:

```
P99_GENERIC_INTEGRAL_CONSTANT(EXP, XTRUE, XFALSE)  // is EXP a compile time integer constant
P99_GENERIC_NULLPTR_CONSTANT(PEXP, XTRUE, XFALSE)  // is PEXP 0 or (void*)0 at compile time
P99_GENERIC_PCONST(PEXP, NCONST, CONST)            // is *PEXP const qualified or not

```

all these accept arbitrary (but valid) expression as their 2nd and 3rd arguments.

The second macro can for example well be used for initialization of a pointer type.

```
#define TRUC_INITIALIZER(PEXP) {                                                    \
   .someField = 32,                                                                 \
   .pointerField = P99_GENERIC_NULLPTR_CONSTANT(PEXP, (truc*)0, truc_account(PEXP)),\
}

static truc the_static_truc = TRUC_INITIALIZER(0);
auto truc the_auto_truc = TRUC_INITIALIZER(42);

```

Here the initialization of the `static` variable will only see constant expressions. The `auto` in turn will have the function `truc_account()` called each time the execution reaches that initialization.

If you have other use cases of this idea that are not yet covered by existing macros in P99, please drop me a line.
