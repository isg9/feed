---
title: Handling control flow inside&nbsp;macros
url: https://gustedt.wordpress.com/2011/02/02/handling-control-flow-inside-macros/
published: "2011-02-02T20:35:27Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=883
---

# Handling control flow inside&nbsp;macros

When people program macros that they want to be executed anywhere where a statement could be, they often use the

```
do {...} while(0)

```

construct. This construct has a big disadvantage in that it may change control flow in an unexpected way when you’d use it as a generic macro tool to collect statements:

```
#define PXX_BLOCK(...) do { __VA_ARG__ } while(0)
.
#define PXX_HYPOTHETIC_BREAK(X)           \
PXX_BLOCK(                                \
if ((X)%2) {                              \
     printf("some thing is wrong:\n");    \
     break;                               \
})
.
for (size_t i = 0; i < n; i+=2) {
    call_a_function(&i);
    PXX_HYPOTHETIC_BREAK(i);
}

```

In this example the `break` statement of the macro will not do what the user of `PXX_HYPOTHETIC_BREAK` might expect, namely to break out of the `for`-loop, but just break out of the `do-while` and continue executing the `for`:

```
for (size_t i = 0; i < n; i+=2) {
    call_a_function(&i);
    do {
      if ((X)%2) {
         printf("some thing is wrong:\n");
         break;
      } while (0);
   }
}

```

P99 uses another approach to do such statement grouping inside macros that does not present surprises concerning the control flow:

```
#define P99_NOP ((void)0)
#define P99_PREFER(...) if (1) { __VA_ARGS__ } else
#define P99_BLOCK(...) P99_PREFER(__VA_ARGS__) P99_NOP

```

and with this replacement of the block macro we obtain as expansion for `PXX_HYPOTHETIC_BREAK(i)`:

```
if (1) {
  if (i%2) {
    printf("some thing is wrong:\n");
    break;
  }
} else ((void)0)

```

- This is placed perfectly fine where C expects a single statement and not a block.
- The `break` statement in there is not hidden by a loop statement and has its effect on the surrounding `for` loop.
- The `if` statement has a corresponding `else`, so an eventual other `else` of the user code will not bind to this one here.
- The else and its depending statement ((void)0) is never executed. But even if it where a constant that is cast to void is an expression without side effects that is eliminated by the compiler.

Other macros that P99 derives from this:

```
#define P99_AVOID P99_PREFER(/* NOP */)
#define P99_XCASE P99_AVOID case
#define P99_XDEFAULT P99_AVOID default

```

The first may at first serve as some sort of comment marker.

```
P99_AVOID dont_do_that_thing(x, y, z);

```

Will never call the function `dont_do_that_thing` since this statement is unreachable. But other than a comment marker `//` this will always check if the call is syntactically correct. If other things change in your code that would make this statement invalid, you’d still notice and be forced to keep your code up-to-date.

For the two other macros look at a hypothetical usage of these “X” cases, X for e **X** clusive:

```
switch (c) {
  P99_XCASE 5: print("take 5\n");
  P99_XCASE 7: return;
  P99_XDEFAULT: ++i;
}

```

The idea is here that these are all *exclusive* cases, no need to even put `break` after the cases to stop the `switch`.

```
switch (c) {
  if (1) { } else case 5: print("take 5\n");
  if (1) { } else case 7: return;
  if (1) { } else default: ++i;
}

```

Here all the `if (1)` have the effect than the statement that is depending on the `else` clause will never be executed unless we jump directly into it via the `case` label. (For those that have doubts, the resulting code is valid C. The switch statement is much more generic than you think.)

Another application is to hide some variables. Your remember perhaps my entry about [*scope bound resource management with `for` scopes*](https://gustedt.wordpress.com/2010/08/14/scope-bound-resource-management-with-for-scopes/). The idea there is to use the variable of a `for`-scope as a local variable for a block or statement but that is still only executed once. One drawback of using a plain `for`-variable is that C99 doesn’t allow these to be static.

```
for (int p00 = 1; p00; p00 = 0)
  for (pthread_mutex_t* point = 0; p00; p00 = 0)
    P99_PREFER(
              static pthread_mutex_t crit = PTHREAD_MUTEX_INITIALIZER;
              crit.lock();
              point = &crit;
              goto JUMPLABEL;
             )
       JUMPLABEL: for (; p00; (point->unlock(), p00 = 0))

```

This has a mutex “crit” that is locked on entry and unlocked when leaving. P99 gives some more programming tools at hand to write such things in macros using `P99_GUARDED_BLOCK` for the `lock/unlock` calls and `P00_BLK_DECL_STATIC` to encapsulate this kind of `static` declaration as we have seen it here. A simple to use macro `CRITICAL_SECTION` can be defined like this:

```
#define MUTUAL_EXCLUDE(MUT)                                    \
P99_GUARDED_BLOCK(pthread_mutex_t*,                            \
      P99_FILEID(mut),                                         \
      &(MUT),                                                  \
      pthread_mutex_lock(P99_FILEID(mut)),                     \
      pthread_mutex_unlock(P99_FILEID(mut)))

#define CRITICAL_SECTION                                                    \
P00_BLK_START                                                               \
P00_BLK_DECL_STATIC(pthread_mutex_t, orwl__crit, PTHREAD_MUTEX_INITIALIZER) \
MUTUAL_EXCLUDE((*orwl__crit))
.
.
CRITICAL_SECTION {
  /* do something critical and important here */
  a += 42;
}

```
