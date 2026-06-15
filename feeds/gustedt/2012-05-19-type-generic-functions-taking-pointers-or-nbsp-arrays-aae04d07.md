---
title: type generic functions taking pointers or&nbsp;arrays
url: https://gustedt.wordpress.com/2012/05/19/type-generic-functions-taking-pointers-or-arrays/
published: "2012-05-19T14:26:11Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1640
---

# type generic functions taking pointers or&nbsp;arrays

Every C programmer knows about the implicit conversion an array object `A` undergoes in most contexts: it is converted to a pointer to its first element, as if the programmer had written \`&A\[0\]\`. The technical term of the standard for that is “lvalue conversion”, and the corresponding section describes what precisely is going on when an object (an lvalue) is evaluated for its contents (the rvalue).

One particular place where this conversion takes place is when an array is passed as an argument to a function. So in that case all additional information about the array, in particular its length, is not made available to the function. One could always pass the size of the array ( `sizeof A`) as additional argument to the function, but this then would always require to pass this information also in case the function is called with what its interface suggest, just a pointer. In this post I will describe a new possibility that is provided by the `_Generic` keyword in C11 that allows us to create type generic interfaces that knows how to distinguish between the case being called with an array or with a pointer as an argument.

Type generic expressions that are constructed with `_Generic` have the particularity that none of their subexpressions undergo lvalue conversion. Let us have a look at an example:

```
_Generic(A, double*: A[0], default: _Generic(&A[0], double*: A))

```

This has four different expressions `A`, `A[0]`, `&A[0]` and `A`. All of them are valid syntactical expressions if `A` corresponds to a pointer or to an array. (To me more precise a `register` array would not be allowed, though). So in these cases the above code provides a valid type generic expression. For any other type of object, e.g an `double` variable or expression or for an object of `struct` type it would be a syntax error.

Now, this expression does an extra check, namely for the base type of `A`: it is only valid if the base type is `double`. Any other type, even `float` or `double const`, would be an error. To summarize the properties of this expression

- it accepts pointers to `double` or arrays of `double` as “argument” `A`
- it rejects all other types of arguments
- it returns an lvalue expression consisting either of a single `double` object or of a `double` array

Now let us add some `()` to force the evaluation order and put this in a macro

```
#define DOUBLE_LENGTH(A) (sizeof _Generic((A), double*: (A)[0], default: _Generic(&(A)[0], double*: (A))))/sizeof(double)

```

The prefix `sizeof` will give us with the size of the lvalue that we discussed above, and the whole macro evaluation will

- return 1 when called with a `double*`
- return the length of the array when called with a `double[]`
- issue a syntax error otherwise

Now if we have a function

```
double fmax(size_t n, double a[n])

```

we could wrap this in a macro

```
#define fmax(A) fmax(DOUBLE_LENGTH(A), (A))

```

that could be equally called with a pointer to `double` (supposing that this corresponds to an array of length 1) or an array of `double`.

With P99 we even could make the wrapper more convenient to provide the length parameter only when need, that is when the macro is called with a single argument instead of two. (If you like to program with P99, this is a good exercise.)

If you want to check for more than one type, type generic expressions as given above become a bit clumsy. [P99](http://p99.inria.gforge.frp99-html/p99__generic_8h.html "P99") now has four interfaces that deal with this. They all accept an expression as a first argument and then a list of types that are to be checked for the base type of the pointer or array.

- `P99_OVALUE` gives the object as in the example above.
- `P99_AVALUE` returns the argument array or an array of one element at the same address as the pointer target.
- `P99_OBJSIZE` returns the size of the object.
- `P99_OBJLEN` returns the length of the array or one if the argument was a pointer.

With that, the above definition could be written as

```
#define DOUBLE_LENGTH(A) P99_OBJLEN(A, double)

```

and if we would like to accept qualified pointers or arrays as well

```
#define DOUBLE_QUALIFIED_LENGTH(A) P99_OBJLEN(A, double, double const, double volatile, double const volatile)

```
