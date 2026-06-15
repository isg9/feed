---
title: 'C Puzzle: Double Trouble'
url: https://blog.regehr.org/archives/683
published: "2012-02-09T17:16:12Z"
feed: regehr
guid: http://blog.regehr.org/?p=683
---

# C Puzzle: Double Trouble

I ran into an interesting C program that both of my C oracles ( [KCC](http://code.google.com/p/c-semantics/) and [Frama-C](http://frama-c.com/)) consider to be well-defined, but that are inconsistently compiled by the current development versions of GCC and Clang on x86-64.

```
int printf (const char *, ...);

void fn2 (int p1)
{
  printf ("%d\n", p1);
}

union U0 {
  short f0;
  int f1;
};

union U0 b;
int *c = &b.f1;

int main ()
{
  short *d = &b.f0;
  *d = 0;
  *c = 1;
  fn2 (b.f0);
  return 0;
}

```

The results:

> ```
> [regehr@gamow 1]$ current-gcc -O1 small.c ; ./a.out
> 1
> [regehr@gamow 1]$ current-gcc -O2 small.c ; ./a.out
> 0
> [regehr@gamow 1]$ clang -O1 small.c ; ./a.out
> 1
> [regehr@gamow 1]$ clang -O2 small.c ; ./a.out
> 0
> ```

GCC is revision 183992 and Clang is 150180.

The question is: Is this program well-defined (in which case I get to report two compiler bugs) or is it undefined (in which case I get to report two bugs in C analysis tools)?

**UPDATE:**

Just to confuse things a bit more, I’ll add this program. GCC prints 1 at all optimization levels but Clang changes its answer to 0 at higher levels.

```
int printf (const char *, ...);

union U0 {
  short f0;
  int f1;
} b;

union U0 *c = &b;

int main ()
{
  b.f0 = 0;
  c->f1 = 1;
  printf ("%d\n", b.f0);
  return 0;
}

```

I decided to report this as an [LLVM bug](http://llvm.org/bugs/show_bug.cgi?id=11966) just to make sure that the folks there have thought this kind of example over. I suspect it’ll get closed as a WONTFIX, which is OK — real compiler design involves a lot of hard tradeoffs.
