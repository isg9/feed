---
title: initialization of dynamically allocated&nbsp;struct
url: https://gustedt.wordpress.com/2010/06/22/initialization-of-dynamically-allocated-struct/
published: "2010-06-22T15:03:40Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=72
---

# initialization of dynamically allocated&nbsp;struct

Other than `static` initialization, for stack or global variables C does not foresee much for storage that is allocated through `malloc`. I have set up an easy convention for myself to deal with this. It resembles much constructors as we know them from C++.

```
typedef struct {
 int a;
 double b;
} A;
#define A_INITIALIZER { .a = 1, .b = 0.5 }

static inline
A* A_init(A *t) {
 if (t) *t = (A)A_INITIALIZER;
 return t;
}

```

That is a function that receives a pointer of my `struct` as first argument, initializes the storage it points to, and then returns the same pointer it received as an argument. In this example the initialization is done by assigning a compound literal, but we could in fact place any code we need to initialize a variable of type `A` here.

Depending on the type, such an \`init’ function may receive more arguments

```
typedef struct {
 int a;
 double b;
} B;
#define B_INITIALIZER(F) { .a = 1, .b = F }

static inline
B* B_init(B *t, double f) {
  if (t) *t = (B)B_INITIALIZER(f);
  return t;
}

```

The return value comes handy when I want to allocate and initialize with just one expression. With the above types we can do

```
A* anA = A_init(malloc(sizeof(A)));
B* aB = B_init(malloc(sizeof(B)), 0.7);

```

Observe that there is no cast of the return from `malloc`. Such a cast could hide a missing prototype for `malloc` and as a result expose you to weird portability bugs. Also observe that I have made the “init” function robust against null pointers, so if the `malloc` doesn’t work, you’d still have to check `anA` or `aB` for null.

We may then use all of that to construct a C++ like `NEW` macro that allocates and initializes.

```
#define _NEW_ARGS(T, ...) T ## _init(malloc(sizeof(T)), __VA_ARGS__)
#define _NEW(T) T ## _init(malloc(sizeof(T)))
#define NEW(...) IF_1_ELSE(__VA_ARGS__)(_NEW(__VA_ARGS__))(_NEW_ARGS(__VA_ARGS__))

A* anotherA = NEW(A);
B* anotherB = NEW(B, 99.0);

```

The inner part of the macro comes in two flavors, one for the case that `NEW` is called with just the type as an argument, the other one if there are other arguments. These then are just passed through to the corresponding init function.

Note that all this only works by *naming convention* and ` typedef`. A `typedef` is necessary to have the name of the type in one token. Something like `struct B` or `unsigned long` would make it impossible to define the init functions as we do. If we have several ` typedef` for the same type, we have to declare an init function for each of these names.

**Edit:** In P99, now there is [`P99_NEW`, a macro that may receive a variable argument list](http://p99.gforge.inria.fr/p99-html/group__preprocessor__allocation.html), and thus is able to handle default arguments to the “init” function.
