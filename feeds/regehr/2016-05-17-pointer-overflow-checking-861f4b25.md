---
title: Pointer Overflow Checking
url: https://blog.regehr.org/archives/1395
published: "2016-05-17T14:06:14Z"
feed: regehr
guid: http://blog.regehr.org/?p=1395
---

# Pointer Overflow Checking

Most programming languages have a lot of restrictions on the kinds of pointers that programs can create. C and C++ are unusually permissive in this respect: pointers to arbitrary objects and subobjects, usually all the way down to bytes, can be constructed. Consequently, most address computations can be expressed either in terms of integer arithmetic or pointer arithmetic. For example, a function based on array lookup:

```
void *memcpy(void *dst, const void *src, size_t n) {
  const char *s = src;
  char *d = dst;
  for (int i = 0; i < n; ++i)
    d[i] = s[i];
  return dst;
}

```

can just as easily be expressed in terms of pointers:

```
void *memcpy(void *dst, const void *src, size_t n) {
  const char *s = src;
  char *d = dst;
  while (n--)
    *d++ = *s++;
  return dst;
}

```

Idiomatic C tends to favor pointer-based code. For one thing, pointers are more expressive: a pointer can name any memory location while an integer index only makes sense when combined with a base address. Also, developers have a sense that the lower-level code will execute faster since it is closer to how the machine thinks. This may or may not be true: the tradeoffs are complex due to details of the semantics of pointers and integers, and also because different compiler optimizations will tend to fire for pointer code and integer code. Modern compilers can be pretty bright, at least for very simple codes: the version of GCC that I happen to be using for testing (GCC 5.3.0 at -O2) turns both functions above into [exactly the same object code](https://godbolt.org/g/vLuRTU).

It is undefined behavior to perform pointer arithmetic where the result is outside of an object, with the exception that it is permissible to point one element past the end of an array:

```
int a[10];
int *p1 = a - 1; // UB
int *p2 = a; // ok
int *p3 = a + 9; // ok
int *p4 = a + 10; // ok, but can't be dereferenced
int *p5 = a + 11; // UB

```

Valgrind and ASan are intended to catch dereferences of invalid pointers, but not their creation or use in comparisons. UBSan catches the creation of invalid pointers in simple cases where bounds information is available, but not in the general case. [tis-interpreter](http://trust-in-soft.com/tis-interpreter/) can reliably detect creation of invalid pointers.

A lot of C programs (I think it’s safe to say almost all non-trivial ones) create and use invalid pointers, and often they get away with it in the sense that C compilers usually give sensible results when illegal pointers are compared (but not, of course, dereferenced). On the other hand, when pointer arithmetic overflows, the resulting pointers can break assumptions being made in the code.

For the next part of this piece I’ll borrow some examples from [a LWN article from 2008](https://lwn.net/Articles/278137/). We’ll start with a buffer length check implemented like this:

```
void awesome_length_check(unsigned len) {
  char *buffer_end = buffer + BUFLEN;
  if (buffer + len >= buffer_end)
    die_a_gory_death();
}

```

Here the arithmetic for computing `buffer_end` is totally fine (assuming the buffer actually contains BUFLEN elements) but the subsequent `buffer + len` risks UB. Let’s look at the generated code for a 32-bit platform:

**```** **awesome_length_check:** **cmpl $100, 4(%esp)** **jl .LBB0_1** **jmp die_a_gory_death** **.LBB0_1:** **retl**

**```**

In general, pointer arithmetic risks overflowing when either the base address lies near the end of the address space or when the offset is really big. Here the compiler has factored the pointer out of the computation, making overflow more difficult, but let’s assume that the offset is controlled by an attacker. We’ll need a bit of a driver to see what happens:

```
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>

#define BUFLEN 100
char buffer[BUFLEN];

void die_a_gory_death(void) { abort(); }

void awesome_length_check(unsigned len) {
  char *buffer_end = buffer + BUFLEN;
  if (buffer + len >= buffer_end)
    die_a_gory_death();
}

int main(void) {
  // volatile to suppress constant propagation
  volatile unsigned len = UINT_MAX;
  awesome_length_check(len);
  printf("length check went well\n");
  return 0;
}

```

And then:

**```** **$ clang -O -m32 -Wall ptr-overflow5.c** **$ ./a.out** **length check went well** **$ gcc-5 -O -m32 -Wall ptr-overflow5.c** **$ ./a.out** **length check went well**

**```**

The problem is that once the length check succeeds, subsequent code is going to feel free to process up to `UINT_MAX` bytes of untrusted input data, potentially causing massive buffer overruns.

One thing we can do is explicitly check for a wrapped pointer:

```
void awesome_length_check(unsigned len) {
  char *buffer_end = buffer + BUFLEN;
  if (buffer + len >= buffer_end ||
      buffer + len < buffer)
    die_a_gory_death();
}

```

But this is just adding further reliance on undefined behavior and the LWN article mentions that compilers have been observed to eliminate the second part of the check. As the article points out, a better answer is to just avoid pointer arithmetic and do the length check on unsigned integers:

```
void awesome_length_check(unsigned len) {
  if (len >= BUFLEN)
    die_a_gory_death();
}

```

The problem is that we can’t very well go and retrofit all the C code to use integer checks instead of pointer checks. We can, on the other hand, use compiler support to catch pointer overflows as they happen: they are always UB and never a good idea.

Will Dietz, one of my collaborators on the integer overflow checking work we did a few years ago, extended UBSan to catch pointer overflows and wrote [a great blog post](http://wdtz.org/catching-pointer-overflow-bugs.html) showing some bugs that it caught. Unfortunately, for whatever reason, these patches didn’t make it into Clang. The other day Will reminded me that they exist; I dusted them off and [submitted them for review](http://reviews.llvm.org/D20322) — hopefully they will get in this time around.

Recently I’ve been thinking about using UBSan for hardening instead of just bug finding. [Android is doing this](http://android-developers.blogspot.fr/2016/05/hardening-media-stack.html?m=1), for example. Should we use pointer overflow checking in production? I believe that after the checker has been thoroughly tested, this makes sense. Consider the trapping mode of this sanitizer:

```
clang -O3 -fsanitize=pointer-overflow -fsanitize-trap=pointer-overflow

```

The runtime overhead on SPEC CINT 2006 is about 5%, so probably acceptable for code that is security-critical but not performance-critical. I’m sure we can reduce this overhead with some tuning of the optimizers. The 400.perlbench component of SPEC 2006 contained two pointer overflows that I had to fix.

Pointer overflow isn’t one of the UBs that we can finesse away by adjusting the standard: it’s a real machine-level phenomenon that is hard to prevent without runtime checks.

There’s plenty more work we could do in this sanitizer, such as catching arithmetic involving null pointers.

**Update:** I built the latest releases of PHP and FFmpeg using the pointer overflow sanitizer and both of them execute pointer overflows while running their own test suites.
