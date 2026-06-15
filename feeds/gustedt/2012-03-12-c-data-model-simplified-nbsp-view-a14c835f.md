---
title: C data model, simplified&nbsp;view
url: https://gustedt.wordpress.com/2012/03/12/c-data-model-simplified-view/
published: "2012-03-12T23:55:34Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1496
---

# C data model, simplified&nbsp;view

As we have seen in the [earlier post](https://gustedt.wordpress.com/2012/03/02/the-c-data-model/), if you dig deeper into it the C data model is quite complex and has corner cases that are a bit bizarre. This might make C weirder than it need to be at a first approach.

As I said, the two most important properties of objects (or values) in C are if they are mutable (or not) and if they are addressable (or not), leaving us with 4 principal categories of objects that we have to deal with.

## immutable objects or values

Can best be though of as objects that or program has not the right to change. They may be defined through different constructs, but for all of these constructs we can add a \`const\` qualification to the type without changing the semantics of the program. An easy rule to make that clear is

> Add a `const` qualifier for every type specification that involves an immutable object or value

A typical example where you really should do this are string literals, this will avoid a lot of trouble

```
char const* hello = "hello";

```

But also other immutable values can take no harm if you think them as `const` qualified. In particular the so-called *rvalues*, the results of function calls, casts or other expressions are best imagined as bearing an implicit `const`.

```
const struct toto func(void);
unsigned a = (unsigned const)M_PI;

```

are equivalent to the counterparts without `const`.

## Unaddressable objects or values

An object that has an address can potentially be modified outside the visible control flow. A pointer containing this address can be passed to another function or to another thread. Objects that are unaddressable are safe with that respect. No hidden changes of value can happen. The optimizing compiler exactly knows the control flow and may optimize any form of assignment.

> Think of all *objects* that you don’t have to pass around to functions as if they were declared with `register`.

Again a typical example for an object that can be imagined as if it were implicitly declared with a `register` storage class are *rvalues*, see above.

## The four classes of objects and values

This classification gives us four classes of objects that we manipulate in C. Let us give nicknames to these beasts, that correspond to the most common usage:

propertiesnicknameC jargonmutable and addressableplain objectmutable and unaddressableregisterobject of `register` storage classimmutable and addressableread-only objectobject of `const` qualified typeimmutable and unaddressableconstantrvalue

Note that a *register* does not necessarily correspond to a register in the CPU. A program may have much more such registers than the can be realized in the CPU, for example. Important is the concept, an object that is kept track entirely by the “local” program code, no surprises from other parts of the program, other threads or signal handlers.

## Caveats

- A `const` qualified `register` variable must be initialized. There is no way to assign a value to such a variable or that it receives a value from “outside”. If it would not be initialized, it could never be used at all.
- A `register` variable shouldn’t be `volatile`. There is no way modify such a variable outside the normal control flow. Such a variable, though formally possible, makes no sense.
- Arrays and unaddressable objects don’t play well together. This is because of a restriction and inconsistencies in the treatment of arrays in C. In most contexts an array (even if compound of a `struct`) is converted to a pointer to its first element.

  Generally array elements inside a register can not be accessed directly for evaluation or modification. There are ways around this, but these are a real code bloat, not well-suited to be used by non-experienced programmers.

- C doesn’t have the possibility to effectively declare globally named constants of other than `int` type. For other types ( `double`, pointers, compound types) we have to resort to preprocessor macros such as `M_PI`.
- Some of the operations that lead to immutable objects do not prevent these objects to be used in assignments (or other modifying operations), there is no constraint violation but the behavior is undefined. This means in particular that the compiler is not forced to issue a diagnostic about the error. This concerns in particular string literals and array components of rvalues.

## Possible C extensions in view of this classification

- **Let array designators be applied to arrays just as field designators are applied to `struct`**

Array designators already appear in initializers and allow e.g the initialization of a `register` array variable. But when we use something like `a[3]` in an expression this is always rewritten as `*(a + 3)` and in particular involves pointer arithmetic.

Such a bypass through addresses is completely useless, there is no technical reason that this must be so. In fact if an array is declared as part of a `struct` any of its compounds can be sucked out of it and any that is addressed with an *integer constant expression* can be set. \[Homework 🙂 Hint: use intermediate compound literals\]

- **Allow the `register` storage class in file scope for `const` qualified variables.**

The main reason why “global” `const` qualified objects cannot act as real *constants* in C as they do in C++ is that there is no real concept in C where such an object would reside if somebody would take the address of such a variable. If we’d make such an object a `register` this problem disappears completely.

```
register float _Complex const I = ... something ...;

```

would be much more helpful for debugging than doing the same with a macro. And one day we could perhaps simply have

```
register void *const nullpointer = 0;

```

- **Allow the initialization of `extern` `const` qualified objects.**

This could be regulated in the same way as `inline` functions since C99.

An `extern` `const` qualified variable with initializer could be used anywhere this seems suitable. A `extern` definition would then look like this:

```
extern long const version_number = 201304L;

```

In one translation unit we would then place an “instantiation”:

```
long const version_number;

```

This would allow the use of this sort of objects to be optimized where this is possible. Currently this is only possible by declaring the object `static`. Unfortunately as soon as we take an address of such a beast, `static` variables have a different address per compilation unit. Not only that this multiplies the storage that this needs, also comparison of addresses will not always give what a user might expect. With this proposed extension we would avoid replication of data and always have the same address for the “same” object.

- **Allow arrays with known size to be assigned.**

This stems from the old tradition that absolutely wants to view an array to decay to a pointer to the first element under most circumstances. I don’t think that valid code would break if we’d introduce such a feature.

For backwards compatibility we still would have that an array function parameter is replaced by a pointer in the first array dimension.
