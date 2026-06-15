---
title: Default arguments for&nbsp;C99
url: https://gustedt.wordpress.com/2010/06/03/default-arguments-for-c99/
published: "2010-06-03T21:49:08Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=33
---

# Default arguments for&nbsp;C99

From C++ we know the concept of default arguments for functions. Briefly stated, this concept allows to call a function with less arguments than it is specified and provide default values for the missing ones. As an example lets take the following prototype:

```
      void one_or_two(int a, int b = 5);

```

Here `one_or_two` may be called with one or two arguments and in the case of one, the value `5` is provided.

The goal here is to have the same effect in C, C99 to be precise, by using the macro preprocessor and inline functions. To do that will use the following elements/features

1. Macros that hide a function
2. Counting macro arguments
3. Choose the right version of a function call
4. `inline` functions for default arguments

**Macros that hide a function**

An important property of the C preprocessor is its lack of recursion. At a first glance it looks that this is merely a restriction and not an advantage. But here the fact that using a macro name inside the expansion of that same macro is just left \`as is’ is quite helpful. But even more than that, a macro that is defined to receive arguments, but that is found without following parenthesis, is also left alone. The global view of our macro will be something like the following:

```
#define one_or_two(...) ONE_OR_TWO_ARGS(one_or_two, __VA_ARGS__)

```

So here, when calling the macro in a program the macro definition will be inserted and the token `one_or_two` which is found there will be left as such. Then, the remaining part of the arguments is expanded and if `ONE_OR_TWO_ARGS` is itself a macro, it will be expanded.

Here we also use a feature that is only normalized since C99, variable macro arguments. This is indicated by the `...` in the definition of `one_or_two`. By that, `one_or_two` may be called with any number of arguments and the token `__VA_ARGS__` in the definition is replaced by the arguments, including the commas between them.

**Counting macro arguments**

To implement the macro `ONE_OR_TWO_ARGS` we will need to determine how many arguments it receives. This can be achieved with something like the following:

```
#define _ARG2(_0, _1, _2, ...) _2
#define NARG2(...) _ARG2(__VA_ARGS__, 2, 1, 0)

```

If `NARG2` is called with two arguments the `2` of its expansion is in third position and we will see this `2`. If it is called with just one argument the `1` will be in that place and thus be the result of the expansion. You probably easily imagine an extension of that macro to treat say 64 or 128 arguments.

**Choose the right version of a function call**

Finally we want two different versions of `ONE_OR_TWO_ARGS`, one if it is called with one argument and one for two:

```
#define _ONE_OR_TWO_ARGS_1(NAME, a) a, NAME ## _default_arg_1()
#define _ONE_OR_TWO_ARGS_2(NAME, a, b) a, b

```

Both macros receive in `NAME` the name of the function that this all about. But only `_ONE_OR_TWO_ARGS_1` uses it to produce the name of a default function to call. If `NAME` would be `one_or_two` the function call produced by the preprocessor would be `one_or_two_default_arg_1()`. This is achieved by the `##` operator of the preprocessor that glues two adjacent tokens into one.

A mechanism to choose between the two now could look like this

```
#define __ONE_OR_TWO_ARGS(NAME, N, ...) _ONE_OR_TWO_ARGS_ ## N (NAME, __VA_ARGS__)
#define _ONE_OR_TWO_ARGS(NAME, N, ...) __ONE_OR_TWO_ARGS(NAME, N, __VA_ARGS__)
#define ONE_OR_TWO_ARGS(NAME, ...) NAME(_ONE_OR_TWO_ARGS(NAME, NARG2(__VA_ARGS__), __VA_ARGS__))

```

`__ONE_OR_TWO_ARGS` is supposed to receive `N` the actual number of arguments in `__VA_ARGS__` as it second argument and constructs a call to either `_ONE_OR_TWO_ARGS_1` or `_ONE_OR_TWO_ARGS_2`.

`_ONE_OR_TWO_ARGS` is just an intermediate step that ensures that all arguments are evaluated sufficiently often such that really the value of the call to `NARG2` is put in place. Otherwise `_ONE_OR_TWO_ARGS_` and `NARG2` would be glued into a single (sensless) token `_ONE_OR_TWO_ARGS_NARG2`.

Finally, `ONE_OR_TWO_ARGS` places the name of the function, determines the number of arguments it has received and calls `_ONE_OR_TWO_ARGS`.

**`inline` functions for default arguments**

Remains to define the function for the default argument. It might look like the following:

```
static inline
int one_or_two_default_arg_1(void) {  return 5; }

```

Here the `inline` keyword (stolen from C++ and new in C99) suggests that the function body should be \`substituted’ in place where a call to it appears. So at a first glance it looks that this defines something similar to a macro, but actually an important difference is the point where evaluation takes place; an `inline` function itself is evaluated in the context in which it is defined. Only its arguments are evaluated at the place it is called. On the other hand, a macro is **not** evaluated at the point of its definition but at the point of its call.

To see that take the case that a default function is not just evaluating a constant expression but doing some real work.

```
static inline
int inline_counter_default_arg_1(void) { ++myCounter, return myCounter; }
#define macro_counter_default_arg_1(void) (++myCounter)

```

Here `inline_counter_default_arg_1` uses **the** global counter variable `myCounter` that is visible at the point of its definition. If there is none, this results in an error. `macro_counter_default_arg_1` evaluates `myCounter` in the context of the caller, and this might actually refer to different variables at different places. The first results in the rule which C++ implements for default arguments: they are evaluated in the scope of definition. The second one is a different model for which C++ has no equivalent.

Finally a complete example.

```
#include <stdio.h>
#define _ARG2(_0, _1, _2, ...) _2
#define NARG2(...) _ARG2(__VA_ARGS__, 2, 1, 0)
#define _ONE_OR_TWO_ARGS_1(NAME, a) a, NAME ## _default_arg_1()
#define _ONE_OR_TWO_ARGS_2(NAME, a, b) a, b
#define __ONE_OR_TWO_ARGS(NAME, N, ...) _ONE_OR_TWO_ARGS_ ## N (NAME, __VA_ARGS__)
#define _ONE_OR_TWO_ARGS(NAME, N, ...) __ONE_OR_TWO_ARGS(NAME, N, __VA_ARGS__)
#define ONE_OR_TWO_ARGS(NAME, ...) NAME(_ONE_OR_TWO_ARGS(NAME, NARG2(__VA_ARGS__), __VA_ARGS__))
#define one_or_two(...) ONE_OR_TWO_ARGS(one_or_two, __VA_ARGS__)

// function definition, also calls the macro, but you wouldn't notice
void one_or_two(int a, int b) { printf("%s seeing a=%d and b=%d\n", __func__, a, b); }

static inline int one_or_two_default_arg_1(void) {  return 5; }

int main (void) {
  // call with default argument
  one_or_two(6);
  // call with default argument
  one_or_two(6, 0);
  // taking a function pointer still works
  void (*func_pointer)(int, int) = one_or_two;
  // But this pointer may only be called with the complete set of
  // arguments
  func_pointer(3, 4);
}

```
