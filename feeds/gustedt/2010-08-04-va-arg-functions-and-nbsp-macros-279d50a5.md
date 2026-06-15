---
title: va_arg functions and&nbsp;macros
url: https://gustedt.wordpress.com/2010/08/04/va_arg-functions-and-macros/
published: "2010-08-04T20:50:22Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=201
---

# va_arg functions and&nbsp;macros

Traditionally C has functions with a variable length argument list, so-called *variadic functions*. The handling of such arguments is done with the `va_list` data type from `stdarg.h` and the corresponding macros. I see two pitfalls with this type of approach that usually make it relatively difficult to use, even in cases where the arguments are supposed to be all the same type `T`.

- There is no indication by these macros how long the list that is passed as argument is.
- There is an implicit conversion of small integer arguments to `signed` or `unsigned` `int` according to the integer promotion rules. These types only have an implementation defined width.

Then first pitfalls requires that usually we need to apply one of the following techniques to handle the list:

- [Terminate the list at each call by a special value](https://kernelbob.wordpress.com/2010/04/05/variadic-functions-in-c-a-new-idiom/). This convention has the disadvantage that each caller has to follow this rule and that failing to do so might produce errors that are hard to track.
- Provide a count of the arguments as an extra parameter that precedes the list. Whereas here also the calling side must do something for each call, at least the convention can be determined from the prototype of the function.
- As a variation of this gives a format string of how the arguments are to be interpreted. The `printf` family of functions uses this approach,

Since C99 we now have *macros* with variable length argument lists. These can be used to interface functions that obtain a length parameter and an array of type `T` and that then are much easier to use on the calling side. Suppose that we have a function `varArrFunc` and a macro `varListMacro` as follows (for and explanation of the implementation see below)

```
   #define P99_CALL_VA_ARG(NAME, TYPE, ...)  (NAME(P99_NARG(__VA_ARG__), (TYPE[]){ __VA_ARG__ }))

   void varArrFunc(size_t len, T* A);
   #define varListMacro(...)  P99_CALL_VA_ARG(varArrFunc, T,  __VA_ARG__ )

```

Such a macro/function pair may then just be called as `varListMacro(78, 7, 9, 99)` or `varListMacro("a", "toto")`, if for the first example we assume that `T` is compatible with `int` or for the second that it is with `char*`. As we can see this avoids both pitfalls

- There is no need to have a calling side convention to handle the length of the argument list.
- All argument conversion is to a type `T` that we specify clearly in the definition of `varListMacro`. If e.g we specify `T` to be `uint64_t` we will always know which value the function `varArrFunc` will see if we feed in `(signed char)-1` as an argument.

How does this work? First we need a macro `P99_NARG(...)` that provides us with the number of arguments that it receives. We showed how to implement such a macro in a [earlier post](https://gustedt.wordpress.com/2010/06/03/default-arguments-for-c99/). Then in its second part the macro `P99_CALL_VA_ARG` uses a *compound literal* to pass an array of base type `T` with our arguments as initial values to the function `varArrFunc`.

Such an implementation is at least as efficient as would be an implementation of `varArrFunc` itself as a variadic function.

- The length of the array is computed at compile time. It is known there, so the information should not get lost.
- As for the variadic function approach at run time each individual argument is only evaluated once.
- Where the variadic function approach would implement the argument list on the stack of the callee, here the array is implemented on the stack of the caller. In any case it is on the stack. For any of the calling conventions that we mentioned above we would either need an extra terminating argument or an extra parameter, so our use of a length parameter to `varArrFunc` is as efficient as that.

- As an extra bonus, the call to `varArrFunc` may even be inlined, if we specify it with `inline`. This then may lead to optimizations that generally are more difficult to achieve for the variadic function approach:

  - The handling the array `A` of parameters inside `varArrFunc` will usually be done with a simple `for`-loop.
  - This loop then has known bounds for each call and the compiler may do loop unrolling
  - Once unrolled, the compiler might even avoid the whole generation of the array and use the parameter expressions directly.
