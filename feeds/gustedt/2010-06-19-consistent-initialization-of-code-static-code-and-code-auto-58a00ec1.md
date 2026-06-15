---
title: Consistent initialization of <code>static</code> and <code>auto</code> variables
url: https://gustedt.wordpress.com/2010/06/19/consistent-initialization-of-static-and-auto-variables/
published: "2010-06-19T10:03:02Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=65
---

# Consistent initialization of <code>static</code> and <code>auto</code> variables

Lack of initialization, especially of pointer fields, is one of the most sources of bugs in C programs that easily lead to crashes and other undefined behavior. C99 give us a feature that helps at least a little bit with this, which are named initializers.

```
    typdedef struct {
      int a;
      double d;
    } A;
    typedef struct {
      A b;
      char* name;
    } B;

```

With C89 you would do something like this to initialize variables of these types:

```
    A anA = { -1, 1.0 };
    B aB = { { -1, 1.0 }, NULL };

```

This has several down points. For `anA` you have to know that field `a` is actually the first in type `A`. Whenever you add a field to the definition of `A`, you will have to check all initializations of all variables of that type if they are still doing the right thing. This might be just infeasible on a large code base and difficult to handle if you have fields of the same type but that should get different values.

In C99 you may write

```
    A anA = { .a = -1, .d = 1.0 };

```

which is much clearer, and stable with respect to ordering of the fields or addition of other fields. In the same spirit for `aB` we should do

```
    B aB = { .b = { .a = -1, .d = 1.0 }, .name = NULL };

```

In many cases this is quite inconvenient to type and often repetitive (thus error prone). Many structures have some \`default’ or \`standard’ way how they are to be initialized. For myself I have a simple naming convention for that where I would do

```
#define A_INITIALIZER  { .a = -1, .d = 1.0 }
#define B_INITIALIZER  { .b = A_INITIALIZER, .name = NULL }

    A anA = A_INITIALIZER;
    B aB = B_INITIALIZER;

```

If this makes sense for the type under consideration you might even give arguments to these defines. We could e.g make the following change for both types such that the field name in both always reflects how the corresponding variable had been named.

```
    typdedef struct {
      int a;
      double d;
      char const*const name;
    } A;
    typedef struct {
      A b;
      char const*const name;
    } B;

#define A_INITIALIZER(NAME)  { .a = -1, .d = 1.0, .name = #NAME }
#define B_INITIALIZER(NAME)  { .b = A_INITIALIZER(NAME.b), .name = #NAME }

    A anA = A_INITIALIZER(anA);
    B aB = B_INITIALIZER(aB);

```
