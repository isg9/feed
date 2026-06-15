---
title: constant expressions
url: https://gustedt.wordpress.com/2010/12/27/constant-expressions/
published: "2010-12-27T09:25:51Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=688
---

# constant expressions

In C, constant expressions come in two different flavors:

1. *integer constant expressions*
2. *initializer constant expressions*.

The naming of (1.) is (again!) sort of unfortunate, because as we will see below there are *constant expressions of integer type* that are not *integer constant expressions* in the sense of the C standard. Better think of (1.) of as *compile time constant integer expressions*.

Constant expressions are composed by applying different types of operators and macros to constants.

## Constants and parameters

Before we want to compose more complicated expressions we will have a look what constants are there in C:

- *Integer constants* are anything that is composed of digits (in a broad sense) and that can be interpreted at compile time, even by the preprocessor. They come in three different flavors, hexadecimal, octal and decimal constants.

  - Hexadecimal constants start with the two characters `0x` that are followed by a non-empty sequence of characters ` 0 ... 9, a .. f, A .. F`.
  - Octal constants start by `0` and have digits `0 ... 7`. (yes, `0` is an octal constant.)
  - Decimal constants have digits `0 ... 9` and start by anything that is not `0`.

Integer constants may have a suffix composed of `u` or `U` to have them of unsigned and of `l` or `L` to have them `long` or `long long`.

- *Integer character constants* are commonly referred to just as character constants. The are denoted by enclosing single quotes and are a portable way to specify an integer that corresponds to a certain character in the actual encoding. E. g. `'a'` is a constant of type `int` that corresponds to the code of the character *a*. These constants can be prefixed by the character *L* to denote a *wide character* of type `wchar_t`. Future versions of C will use *u* and *U* for Unicode characters.
- *Enumeration constants* are symbolic constants that are declared in the program. They are declared as in the examples below and are always of type `int`. These are the only named constants that may enter into a *integer constant expressions*.
- Floating point constants, such as `0.0` or `1.0E2`.
- *Address constants* are the results of

  - the unary `&` operator applied to a variable of static storage
  - implicit conversion of an array with static storage duration
  - implicit conversion of a function designator
  - an explicit cast of an integer constant to a pointer type, e.g `(int*)0` or `(unsigned*)0xbeaf`.

Often confounded with constants by beginners of C are *`const` qualified variables*, better called *unmutable variables* or *parameters*:

```
int const one = 1;
static double const pi = 3.14;
double const*const piP = π

```

All these are not constants in the understanding of C. Note in particular that `&pi` is an address constant as specified above, but `piP` isn’t. Also note that `one` would be considered an integer constant in C++.

## Integer constant expressions

… can be used

- as values for enumeration constants:

  ```
  enum { red = 0xF00, green = 0x0F0, blue = 0x00F };
  enum { nl = '\n' };
  enum {
    size1 = sizeof(char),
    size2 = sizeof(short),
    size3 = sizeof(int),
    size4 = sizeof(long),
    size5 = sizeof(long long),
  };

  ```

- length parameters for `static` arrays and `typedef`‘s

  ```
  static unsigned char doubleBytes[sizeof(double)];
  typeof unsigned uInUlong[sizeof(unsigned long)/sizeof(unsigned)];

  ```

Those are all expressions that we may obtain from composing

- integer constants, integer character constants, enumeration constants
- constant `sizeof` or `offsetof` expressions
- casts of constant floating point expressions to integer types such as `(unsigned)(red / 2.0)`

with the usual operators such as

- unary `!`, `~`, `+` and –
- binary `+`, -, `*`, `/`, `&`, `|`, `^`, `&&`, and `||`
- ternary `? :`
- the comma operator, but this is probably not very useful.

Beware that not all `sizeof` expressions are constant since the sizeof operator may be applied also to *variable length arrays*. But `sizeof` expressions of all other variables or types will be constant.

Such expressions may then safely be used to declare enumeration constants or to specify the length of a `static` array or inside a `typedef`. The important aspect is here that all of this can be done at compile time. The compiler doesn’t need to know anything about the execution environment to compute these expressions.

## Initializer constant expressions

… additionally have floating point constants, address constants and non-integer casts that compose them. In particular something like the following is valid:

```
struct toto { int a; double b; };
static struct toto A;
&A.b ==  (double*)((char*)&A + offsetof(struct toto, b))

```

Important is that a variable is only used to get its address but that it is not evaluated (its value is not used).

An interesting particular case is that it is allowed to take the address of a variable with static storage inside its initializer:

```
struct tutu { double f; double*const fp; };
static struct toto A = { .f = 0.5, .fp = &A.f, };

```

Here the field `.fp` because it is *`const` qualified* must be specified during initialization. Otherwise it would contain a garbage value that could never be changed.

Actually, not all kind of expressions are allowed that use address constants. We may only add or subtract an integer constant expression to such an address. Other types of expressions (e.g comparing two different addresses) would need to be evaluated at startup time, something that is not foreseen in C.

Initializer constant expressions can be used anywhere where we want to initialize a variable with static storage duration. Because they may involve addresses of objects, the compiler might not be able to compute such expression completely at compile time. If your code is e.g compile as *position independent code*, PIC, the address of a variable or function is only determined at program startup. It may be different in each invocation of your program. It is the task of the runtime system, usually the so-called dynamic loader, to provide your program with the value of all these expressions. But there is a guarantee that this is done before any program code is entered.

Finally, initializer constant expressions may well be of an integer type without being integer constant expressions as understood by the C standard. E.g

```
struct tata { double f; uintptr_t const self; };
static struct tata A = { .f = 0.5, .self = (uintptr_t)&A };

```

is a valid initialization of `A`. But although it is of integer type, the expression `(uintptr_t)&A` from the initializer is not valid to initialize an enumeration constant for example.
