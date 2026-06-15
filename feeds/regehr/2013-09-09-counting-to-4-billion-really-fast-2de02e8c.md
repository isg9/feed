---
title: Counting to 4 Billion Really Fast
url: https://blog.regehr.org/archives/1033
published: "2013-09-09T23:10:39Z"
feed: regehr
guid: http://blog.regehr.org/?p=1033
---

# Counting to 4 Billion Really Fast

Before teaching today I wrote a silly program to make sure that the bitwise complement of any 32-bit value is equal to one less than the two’s complement negation of that value:

```
#include <stdio.h>
#include <limits.h>
#include <assert.h>

void foo (unsigned x)
{
  unsigned x1 = ~x;
  unsigned x2 = -x - 1;
  assert (x1 == x2);
}

int main (void)
{
  unsigned x = 0;
  long checked = 0;
  while (1) {
    foo (x);
    checked++;
    if (x==UINT_MAX) break;
    x++;
  }
  printf ("checked %ld values\n", checked);
  return 0;
}

```

When the program terminated without any perceptible delay, I figured there was a bug, but nope: the code is good. It turns out that both GCC and Clang optimize the program into effectively this:

```
int main (void)
{
  printf ("checked 4294967296 values\n");
  return 0;
}

```

The surprise (for me) was that at `-O1` — which traditionally does not enable interprocedural optimizations or aggressive loop transformations — both compilers looked inside the function `foo()` closely enough to figure out that it is a nop, and also that both compilers were able to predict that a not-traditionally-structured loop executes 2^32 times. I do so many posts about compiler bugs here that I figured a bit of antidote would be nice.

This is gcc 4.2.1 and Apple LLVM version 4.2.
