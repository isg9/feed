---
title: 'C17: rvalues and function types drop&nbsp;qualifiers'
url: https://gustedt.wordpress.com/2018/06/15/c17-rvalues-and-function-types-drop-qualifiers/
published: "2018-06-15T21:15:42Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3657
---

# C17: rvalues and function types drop&nbsp;qualifiers

At least since C99 the C standard has:

> The properties associated with qualified types are meaningful only for expressions that are lvalues.

This phrase expresses the intent that type qualifiers are to be ignored in “some sense” if they appear in other contexts, namely for the *values of expressions*, *i.e.* rvalues.

Until the invention of `_Generic` in C11, nobody really cared what happened to qualifiers of expressions: only the value and size, not the type itself of an expression was observable from inside a C program.

This changed with C11, `_Generic` makes types observable. Unfortunately the formulation of the `_Generic` feature itself did not make it clear if it could be used to also detect the qualifications. C11 leaves the following questions unresolved

1. Are there contexts where qualified rvalues can pop up?
2. Can `_Generic` distinguish a qualified argument from an unqualified one?

In particular the lack of clarification of the second question lead to divergence in the implementation of the `_Generic` feature in mainline compilers.

In our work for C17, we tackled these questions in two “defect reports”, [DR 423](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2257.htm#dr_423) (for a.) and [DR 481](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2257.htm#dr_481) (for b.).

### DR 423

For the first, there are two context that seemed to allow values of expressions to have qualifiers: casts and function returns. For both it was decided that the intent of the standard never had been to have qualifications here, and that the standard should be fixed to clearly state that intent. Therefore DR 423 proposes two modifications. For casts, it changes the first phrase of 6.5.4.p5 (change in red)

> Preceding an expression by a parenthesized type name converts the value of the expression to *the unqualified version of* the named type.

For function returns, the situation is a bit more complicated, because the current syntax allows return types of functions to be qualified. *E.g.*

```
double const foo(void) {
    return 0;
}

```

Is a valid definition of a function. Whenever it is called, the return value is an rvalue, so is this value a `double` or a `double const`?

The answer that was found is to treat the above function as if the `const` where not present. That is, the return type of the function is adjusted to the unqualified type. The proposed change for 6.7.6.3 p5 is

> If, in the declaration “ **T** **D1**“, **D1** has the form
>
> **D** ( parameter-type-list )
>
> or
>
> **D** ( identifier-list opt )
>
> and the type specified for ident in the declaration “ **T** **D**” is “ *derived-declarator-type-list* **T**“, then the type specified for ident is “ *derived-declarator-type-list* function returning *the unqualified version of* **T**“.

Beware that this has more effects than just changing the qualification of value expressions. Compilers that will claim to be compliant to C17 must accept the following:

```
double (*foop)(void) = foo;
double const (*foopc)(void) = foop;

```

That is any function type or type derived from that behaves as if the qualifier were not present. Most current compilers will probably see these as a constraint violations. Also, since now types that may have been considered different are now clearly specified to be the same, the following expression is a constraint violation:

```
_Generic(X,
    double (*)(void): 1,
    double const (*)(void): 2
)

```

because a generic expression is not supposed to have two entries with compatible types. Most current compilers probably would accept that construct, so they’d have to be fixed.

### DR 481

For `_Generic` it was decided that the intent of the construct was to implement some kind of function overloading for C and that it was meant to have a behavior that is the closest possible to that intended feature. So it was decided to force that the type of a controlling expression is seen *as if* it would undergo the same conversions as arguments to function calls. The proposed text for 6.5.1.1 reads

> The type of the controlling expression is the type of the expression as if it had undergone an lvalue conversionnew), array to pointer conversion, or function to pointer conversion. That type shall be compatible with at most one of the types named in the generic association list.
>
> new) lvalue conversion drops type qualifiers.

As a consequence the following code is still valid

```
_Generic(X,
    double: 1,
    double const: 2
)

```

only that the second association can never trigger. Therefore it would be reasonable for compilers to give a warning diagnostic for such a generic expression.

This does not mean, though, that qualifiers are not observable at all. You just have to put a little bit more effort into it:

```
_Generic(&X,
    double*: 1,
    double const*: 2
)

```

This should still work and both associations may trigger depending on whether or not you pass an lvalue that is `const` qualified or not.
