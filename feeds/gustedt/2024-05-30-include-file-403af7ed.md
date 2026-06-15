---
title: '#include __FILE__'
url: https://gustedt.wordpress.com/2024/05/30/include-__file__/
published: "2024-05-30T14:01:22Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=4107
---

# #include **FILE**

Recursive inclusion is possible in C, and more surprisingly it even makes sense in some cases. In fact it provides an easy way to do code unrolling of small code snippets:

```
#if !defined(DO_I) || (DO_I < DO_BOUND)
# include "do-incr.c"
DO_BODY
# include __FILE__
#endif
#undef DO_I

```

Here this uses two macros that have to be defined beforehand, `DO_BOUND ` and `DO_BODY`, and one macro `DO_I` for a hexadecimal loop counter that is maintained by the include file `"do-incr.c"`. If we hold the above recursive code in some file `"doit.c"` (plus some extras, see below) we can use this as simply as in

```
#define DO_BODY [DO_I] = DO_I,
#define DO_BOUND 0x17
unsigned array[] = {
# include "doit.c"
};
#undef DO_BODY
#undef DO_BOUND

```

do generate the intialization of an array,

```
unsigned array[] = {
[0x0000] = 0x0000,
[0x0001] = 0x0001,
...
[0x0016] = 0x0016,
[0x0017] = 0x0017,
};

```

or with an additional macro `DO_NAME` ( `DO_I` without a hex prefix) as

```
#define ENAME_(X) enum_ ## X
#define ENAME(X) ENAME_(X)
#define DO_BODY ENAME(DO_NAME) = DO_I,
#define DO_BOUND 197
enum {
# include "doit.c"
};
#undef DO_BODY
#undef DO_BOUND

```

for the declaration of an enumeration type

```
enum fun {
enum_0000 = 0x0000,
enum_0001 = 0x0001,
...
enum_00C4 = 0x00C4,
enum_00C5 = 0x00C5,
};

```

Here the given limit of `197` is in fact a limit that is imposed by gcc for which I was testing this. They restrict the depth of inclusion to 200, unless you raise that limit with the command line option `-fmax-include-depth`.

Note also, that the recursive line `#include __FILE__` uses a macro to specify the file name because `#include` lines are indeed evaluated by the preprocessor if necessary.

Recursion is only reasonable if we have a stop condition, so to handle this the include file `"do-incr.c"` is crucial. Since in standard C we have no way to evaluate the line of a `#define` while processing it, we have to ensure that we can change the value of `DO_I`. That file looks something like

```
#ifndef DO_I
# undef  DO_I0
# define DO_I0 0
# undef  DO_I1
# define DO_I1 0
# undef  DO_I2
# define DO_I2 0
# undef  DO_I3
# define DO_I3 0
# define DO_I DO_HEX(DO_NAME)
#elif DO_HEX(DO_I0) == 0
# undef DO_I0
# define DO_I0 1
#elif DO_HEX(DO_I0) == 1
# undef DO_I0
# define DO_I0 2
...
#elif DO_HEX(DO_I0) == 0xF
# undef DO_I0
# define DO_I0 0
# include "do-incr1.c"
#endif

```

As you may have guessed, this uses macros `DO_I0`, `DO_I1`, `DO_I2`, and `DO_I3` as hexdigits of a 16 bit number, and includes another file `"do-incr1.c"` if the digit `DO_I0` overflows.

I hope that if you are interested in this technique, you will be able to join the dots in the above code. To be complete, here are the macros that join the digits to make a hexnumber ( `DO_HEX`) and to make a name component ( `DO_NAME`):

```
#ifndef DO_HEX
# define DO_HEX_(X) 0x ## X
# define DO_HEX(X) DO_HEX_(X)
# define DO_NAME_(A, B, C, D) A ## B ## C ## D
# define DO_NAME_IIII(A, B, C, D) DO_NAME_(D, C, B, A)
# define DO_NAME DO_NAME_IIII(DO_I0, DO_I1, DO_I2, DO_I3)
#endif

```
