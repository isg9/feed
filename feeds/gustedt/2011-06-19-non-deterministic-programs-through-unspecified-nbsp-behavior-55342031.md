---
title: non-deterministic programs through unspecified&nbsp;behavior
url: https://gustedt.wordpress.com/2011/06/19/non-deterministic-programs-through-unspecified-behavior/
published: "2011-06-19T14:07:06Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1038
---

# non-deterministic programs through unspecified&nbsp;behavior

C defines different types of behavior for certain features:

1. deterministic behavior, most features such as the `!` operator
2. unspecified behavior, where the compiler has a choice on how to implement a feature, e.g the evaluation order of subexpressions. Here, I also include “needs not to” behavior, where it seems that it has been forgotten to mention this explicitly, e.g if two `const` qualified compound literals of the same scope with the same value are folded into one object
3. implementation defined behavior, behavior unspecified by the standard but for which an implementation has to document its choices, e.g the greatest value of an `int`.
4. undefined behavior, the compiler is allowed to eat your hard disk for breakfast, such as when accessing an array element out of bounds

Towards its end, the C standard has long list for the later three types.

Where most C programmers will know about the possible pitfalls of undefined behavior, it seems that unspecified behavior has a lot less attention although it might have severe consequences, too.

Unspecified behavior can make a program behave non-deterministically, it may have different results if different optimization options are turned on or (in theory) may even have different outcome even with the same executable and the same input. So it makes a program non-portable, we should never rely on such a feature.

Let us see this with an example. As mentioned above `const` qualified literals such as `(int const){ 0 }` can be folded if they have the same value:

```
void const*a = &(int const){ 0 };
void const*b = &(int const){ 0 };
if (a == b) // do something
else        // do something else

```

Here a compiler can freely choose which part of the `if` statement it wants to execute. It can decide that at compile time and optimize the equality test away. It might have a complex strategy that decides from the depth of the call stack (or whatever) if the two are the same at run time.

Clearly this is an artificial, made-up, example. But decisions on whether two pointers have equal values are done in code and in such cases we’d have to guarantee ourselves that the two branches that the code may take have the same observable behavior at the end.

Another important example is the evaluation order for function calls. An expression like `f(a, b)` has **three** different subexpressions evaluated before a function is called. In view of parallel execution, future compilers could even decide at run time to run subexpression (calls to functions e.g) in different threads, who knows.

Let us have a look at a simple (again made-up) case:

```
 printf(" is the result printing %d and %d characters in function arguments\n",
         printf("A"),
         printf("B"));

```

Here the two calls to `printf` in the argument list of the other one are executed before the call to the principal `printf`. But in which order?

I made some short experiments with the compilers that I have available for my machine, clang, gcc, icc, opencc, pcc, tcc:

- `int const` compound literal folded: none
- string literal folded: all but tcc
- evaluation order of printf example: all “AB”, only gcc has “BA”
- function call `f(a,b)`: before an execution of the call as such, clang, icc, pcc, tcc have `f|a|b`, gcc has `b|a|f`, and opencc has `a|b|f`.

We can see from this that depending on evaluation order is certainly not a good idea. Also note that even if icc and opencc claim to be “mostly” compatible with gcc, this may mean that certain observable properties of the program might be different.

To finish, here is a C fragment that allows to test for evaluation order. But as said, there is no guarantee that this has even consistent behavior between runs of the same executable:

```
static unsigned execorder = 0;
printfunc printer(unsigned occurence) {
  printf("(%u,%u)", occurence, execorder);
  ++execorder;
  return printf;
}
...
printer(0)(" is the result printing %d and %d characters in function arguments\n",
             printer(1)("A"),
             printer(2)("B"));

```
