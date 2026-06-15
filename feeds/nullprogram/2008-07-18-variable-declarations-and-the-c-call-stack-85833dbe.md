---
title: Variable Declarations and the C Call Stack
url: https://nullprogram.com/blog/2008/07/18/
published: "2008-07-18T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2008/07/18/
---

# Variable Declarations and the C Call Stack

## [Variable Declarations and the C Call Stack](/blog/2008/07/18/)

July 18, 2008

nullprogram.com/blog/2008/07/18/

A co-worker asked me a question today about C/C++ pointers,

> If a pointer is declared inside a function with no explicit initialization, can I assume that the pointer is initialized to `NULL`?

We were down in the lab and, therefore, he had no Internet access to look it up himself, which is why he asked. When I code C, it is just a sort of mental habit to not use a non-static function variable without first initializing it, but is this accurate? I *knew* the answer was "no", but I wanted to be able to explain the "why".

Anyway, I quickly recalled some of my experimental C programs and thought carefully about the mechanics of what is going on behind-the-scenes, allowing me to confidently give him a "no" answer. I then threw this together in a few seconds to prove it,

```c
#include

void a ()
{
  int   * x = (int *) 0x012345FF;
  double  y = -63454;
  (void) x;
  (void) y;
}

void b ()
{
  int   * x;
  double  y;
  printf ("%p, %f\n", x, y);
}

int main ()
{
  a ();
  b ();
  return 0;
}
```

When you compile it, make sure you don't use the optimization options ( `-O`, `-O2`, or `-O3` for `gcc`) because they change the inner-workings of the program. It might do things like make those functions inline (so they won't be on the stack as I am intending), or even toss out `a()`, as it appears to do nothing. The compiler sees that, even though I "used" variables in `a()` by casting them to `void`, nothing is really happening, so `a()` can be ignored. We can probably get around this with a tacked on ` volatile` declaration, which you might see a lot of in a micro-controller program. In a micro-controller, some memory addresses are mapped to registers external to the software, so, from the compiler's point of view, access to these locations may look like nothing is really happening. Optimizing away variables that point to these memory locations will lead to an incorrect binary, so your robot or laser guided shark or whatever won't work.

Anyway, compiling with optimization will break my example! So don't do it here.

When compiling, you should get some warnings about using uninitialized variables, which is kind of the point of my example. Ignore it. That warning alone gives away the answer to the main question, really, but this example is a bit more fun!

Before you run it, study it and think about what the output should look like. When `a()` is called, its stack frame goes into the call stack, which contains the two declared variables. These variables are then assigned as part of the function execution. When `a()` returns, the frame is popped off the stack. Then `b()` is called, and, as the variable declarations are exactly the same, it will fit right over top of `a()`'s old stack frame, and its variables will line up. `x` and `y` are not assigned any value, so they pick up whatever junk was lying around, which happens to be the values assigned in `a()`.

When you run the program, this is the output,

```
0x12345ff, -63454.000000

```

The pointer is *not* initialized to `NULL`. If `x` is passed back uninitialized under the assumption that a `NULL` is being passed, some other poor function that handles the return value may dereference it, resulting in possibly some [nasal demons](http://www.catb.org/jargon/html/N/nasal-demons.html), but most likely an annoying segmentation fault. Worse, this error may occur far, far away from where the actual problem is, and even worse than that, only sometimes (depending on the state of the call stack at just the right moment).

Note here that I am talking about non-static function variable declarations. Global variables and static function variables will not be on the stack. They are placed in a fixed location (in the data segment), and their values are implicitly initialized to 0 at *compile time*.

- [c](/tags/c/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Variable%20Declarations%20and%20the%20C%20Call%20Stack) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Variable+Declarations+and+the+C+Call+Stack).

« [Up is Down](/blog/2008/06/26/)

» [Sudoku Solver](/blog/2008/07/20/)
