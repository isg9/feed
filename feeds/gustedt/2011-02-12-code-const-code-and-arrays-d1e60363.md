---
title: <code>const</code> and arrays
url: https://gustedt.wordpress.com/2011/02/12/const-and-arrays/
published: "2011-02-12T09:29:47Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=935
---

# <code>const</code> and arrays

Qualifying a type with `const` may have some surprises. Consider the following

```
toto A = { 0 };
toto const* p = &A;

```

where `toto` is an opaque `typedef` that we don’t control. The question would be if the initialization of `p` is valid or not? From the way I pose that question, you have probably guessed the answer: it depends.

If `toto` is an elementary type, a pointer type, a `struct` or a `union`, all works as you would hope. The const qualifier restricts the what you will be able to do with `A` through `p` so C allows this initialization and analogous assignments as well.

For arrays all is different.

```
typedef double dArr[1];
dArr a0 = { 0 };
dArr const* a = &a0;         // incompatible types

typedef double const cArr[1];
cArr c0 = { 0 };
cArr const* c = &c0;         // multiple const, ok
cArr* d = &c0;               // compatible types!
double const (*e)[1] = &c0;  // compatible types!

a = c;                       // ok
a = d;                       // ok
a = e;                       // ok

```

Running this through `clang` gives me the following warning

```
test-const.c:28:15: warning: incompatible pointer types initializing 'dArr const *'
      (aka 'double const (*)[1]') with an expression of type 'dArr *' (aka 'double (*)[1]')
  dArr const* a = &a0;
              ^   ~~~

```

So what clang is telling us here is that the type of `a` is not exactly what we would have expected, namely a pointer to a `const` qualified array type. In fact `a` is of the same type as `c`, `d` and `e`, a pointer to an array of `const double`.

The simple wording of the standard for that property is the following:

> If the specification of an array type includes any type qualifiers, the element type is so-qualified, not the array type.

I could only speculate what historic reasons this special rules for arrays has. For `struct`, e.g, the same problem occurs (what qualifiers have the individual fields?) and a more natural solution has been chosen.

Consequences of this rule are nasty for function calls. Because of the other rule about the first array dimension being equivalent to a pointer declaration, declaring a matrix parameter `const` is very tedious. Let’s look at some innocent interfaces:

```
typedef double matrix[1][1];
void doit(matrix const A) ;
.
matrix B;
doit(B);                     // incompatible types
matrix const C;
doit(C);                     // multiple const, ok
double const D[1];
doit(&D);                    // ok

```

These difficulties are really pointless and due to syntax alone, a typical case of a mis-design. Qualifying matrices with `const` makes perfect sense for functions, perhaps even more than for primitive types. In code that uses large matrices we may want to map matrices of coefficients in read-only segments. An interface should be able to reflect that property, protect the `const` qualification, but must also be able to accept other matrices that are by coincidence not `const` qualified.

A typical repair work to evade that problem is to do a cast:

```
doit((double const(*)[1])B);

```

This is not very practicable: who ever wants to write casts such as `(double const(*)[1])`?

But this is also dangerous. Such a cast would happily take **any** pointer and interpret it as a `double` matrix. So before casting in that way, we should at least make sure that the base type of the argument matrix is assignment compatible with the base type that is expected by the function. An expression that would check for this could look as follows

```
(double const*const){ &B[0][0] }

```

That is we declare a compound literal of type *pointer to `const` qualified base type* that is initialized with the address of the first element of the matrix. Now our function call could look like this:

```
doit((double const(*)[1])(double const*const){ &B[0][0] });

```

Such a strategy can only work because we have guarantees from the C standard:

- The address of a compound object ( `struct`, `union` or array) coincides with the address of its first member. Or stated otherwise, no objects starts with padding bytes.
- The alignment of a compound object is compatible with the alignment of each of its components.

So we improve security by obfuscating things even further. Clearly such expressions as the one above can’t be written on a daily base to earn a living, we need macros that encapsulate such a mess. A generic macro to do such a thing would still be a bit complicated to use, because we need the base type and the dimension of the matrix. This would be tedious to maintain at the place where we are calling the function. So instead of providing such a macro that just does that cast, in P99 I decided augment the macro [P99\_ACALL](http://p99.gforge.inria.fr/p99-html/p99__for_8h.html) that we have seen in [a recent post](https://gustedt.wordpress.com/2011/01/13/vla-as-function-arguments/).

```
/* header file */
inline
double dotproductFunc(P99_AARG(double const, A),
                      P99_AARG(double const, B)) {
  register double ret = 0.0;
  P99_DO(size_t, i, 0, P99_ALEN(A, 1))
    ret += (*A)[i] * (*B)[i];
  return ret;
}

#define dotproduct(VA, VB) dotproductFunc(P99_ACALL(VA, 1, double const), P99_ACALL(VB, 1, double const))

/* in just one compilation unit */
extern inline
double dotproductFunc(P99_AARG(double const, A),
                      P99_AARG(double const, B));

```

The difference is just that we add `const` qualifiers to the arguments of `doproductFunc` and make the dimension and the target base type explicit in the macro declaration. Now, `dotproduct` may be called with vectors that are `const` qualified or not but type checking for the base type of the matrix is still in place.
