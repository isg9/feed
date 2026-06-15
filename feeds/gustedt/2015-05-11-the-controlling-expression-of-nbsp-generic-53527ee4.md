---
title: The controlling expression of&nbsp;_Generic
url: https://gustedt.wordpress.com/2015/05/11/the-controlling-expression-of-_generic/
published: "2015-05-11T22:06:22Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=2256
---

# The controlling expression of&nbsp;_Generic

Recently it showed that the C standard seems to be ambiguous on how to interpret the controlling expression of `_Generic`, the one that determines the choice. Compiler implementors have given different answers to this question; we will see below that there is code that is interpreted quite differently by different existing compilers. None of them is “wrong” at a first view, so this tells us that we must be careful when we use `_Generic`. In this post I will try to explain the problem and to give you some work around for common cases.

## The Symptoms

The symptoms of the problem are that there is code that is interpreted differently by different compilers. On one side there is gcc that follows a certain interpretation and on the other there are clang and IBM. Rember that a `_Generic` expression makes a choice according to the *type* of first expression:

```
char const* a = _Generic("bla", char*: "blu");                 // clang error
char const* b = _Generic("bla", char[4]: "blu");               // gcc error
char const* c = _Generic((int const){ 0 }, int: "blu");        // clang error
char const* d = _Generic((int const){ 0 }, int const: "blu");  // gcc error
char const* e = _Generic(+(int const){ 0 }, int: "blu");       // both ok
char const* f = _Generic(+(int const){ 0 }, int const: "blu"); // both error

```

Here for example, in the first two lines clang sees `"bla"` as an array of 4 `char while gcc applies "array to pointer conversion" and sees it as a pointer to char.`

## The Problem

The real problem that is behind that interpretation is that the compilers give different answers to the following question:

> Does the controlling expression of a \_Generic primary expression undergo any type of conversion to calculate the type that is used to do the selection?

Gcc applies conversions as they would be done in most other expressions (choice 1), clang and IBM don’t (choice 2). And unfortunately both sides have their argument why the C standard should be interpreted as they are doing it. I will not go into the details, here, how the standard is unclear about this, if you are more interesting in details you’ll find a link to follow below.

## Working around it

Since today C implementations have already taken different paths for this feature, you should be careful when using `_Generic` to remain in the intersection of these two interpretations. A certain number of design questions should be answered when implementing a type generic macro:

- Do I want to differentiate the outcome according to the qualification of the argument?
- Do I want to distinguish arrays from pointer arguments?
- Do I want to distinguish narrow types?
- Do I want to apply it to composite types, namely `struct` types?

The following lists different strategies for common scenarios, that can be used to code type generic macros that will work with both of the choices 1 or 2.

## Wide integers and floating point types

This is e.g the case of the C library interfaces in `<tgmath.h>`. If we know that the possible type of the argument is restricted in such a way, the easiest is to apply the unary plus operator `+`, as in

```
  #define F(X) _Generic(+(X),             \
    default: doubleFunc,                  \
    int: intFunc,                         \
    ...                                   \
    _Complex long double: cldoubleFunc)(X)

  #define fabs(X) _Generic(+(X),          \
    default: fabs,                        \
    float: fabsf,                         \
    long double: fabsl)(X)

```

This `+` sign ensures an lvalue to rvalue conversion, and, that it will error out at compilation time for pointer types or arrays. It also forcibly promotes narrow integer types, usually to `int`. For the later case of `fabs` all integer types will map to the `double` version of the function, and the argument will eventually be converted to `double` before the call is made.

## Adding pointer types and converting arrays

If we also want to capture pointer types *and* convert arrays  to pointers, we should use `+0`.

```
  #define F(X) _Generic((X)+0),           \
    default: doubleFunc,                  \
    char*: stringFunc,                    \
    char const*: stringFunc,              \
    int: intFunc,                         \
    ...                                   \
    _Complex long double: cldoubleFunc)(X)

```

This binary `+` ensures that any array is first converted to a pointer; the properties of `0` ensure that this constant works well with all the types that are to be captured, here. It also forcibly promotes narrow integer types, usually to `int`.

## Converting arrays, only

If we k now that a macro will only be used for array and pointer types, we can use the `[]` operator:

```
  #define F(X) _Generic(&((X)[0]),        \
    char*: stringFunc,                    \
    char const*: stringFunc,              \
    wchar_t*: wcsFunc,                    \
    ...                                   \
    )(X)

```

This operator only applies to array or to pointer types and would error if present with any integer type.

## Using qualifiers of types or arrays

If we want a macro that selects differently according to type qualification or according to different array size, we can use the `&` operator:

```
  #define F(X) _Generic(&(X),        \
    char**: stringFunc,              \
    char(*)[4]: string4Func,         \
    char const**: stringFunc,        \
    char const(*)[4]: string4Func,   \
    wchar_t**: wcsFunc,              \
    ...                              \
    )(X)

```

## Further reading

If you want to know more, details on the story can be read in a [document](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1930.htm "Controlling expression of _Generic primary expression") of the C committee’s list.
