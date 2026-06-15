---
title: Find the Integer Bug
url: https://blog.regehr.org/archives/1136
published: "2014-04-21T03:42:46Z"
feed: regehr
guid: http://blog.regehr.org/?p=1136
---

# Find the Integer Bug

Not all of the functions below are correct. The first person to leave a comment containing a minimal fix to a legitimate bug will get a small prize. I’m posting this not because the bug is particularly subtle or interesting but rather because I wrote this code for a piece about integer overflow and thought — wrongly, as usual — that I could get this stuff right the first time.

By “legitimate bug” I mean a bug that would cause the function to execute undefined behavior or return an incorrect result using GCC or Clang on Linux on x86 or x64 — I’m not interested in unexpected implementation-defined integer truncation behaviors, patches for failures under K&R C on a VAX-11, or style problems. For convenience, here are [GCC’s implementation-defined behaviors for integers](http://gcc.gnu.org/onlinedocs/gcc/Integers-implementation.html#Integers-implementation). I don’t know that Clang has a section in the manual about this but in general we expect its integers to behave like GCC’s.

```
#include <limits.h>
#include <stdint.h>

/*
 * specification:
 *
 * perform 32-bit two's complement addition on arguments a and b
 * store the result into *rp
 * return 1 if overflow occurred, 0 otherwise
 */

int32_t checked_add_1(int32_t a, int32_t b, int32_t *rp) {
  int64_t lr = (int64_t)a + (int64_t)b;
  *rp = lr;
  return lr > INT32_MAX || lr < INT32_MIN;
}

int32_t checked_add_2(int32_t a, int32_t b, int32_t *rp) {
  int32_t r = (uint32_t)a + (uint32_t)b;
  *rp = r;
  return
    (a < 0 && b < 0 && r > 0) ||
    (a > 0 && b > 0 && r < 0);
}

int32_t checked_add_3(int32_t a, int32_t b, int32_t *rp) {
  if (b > 0 && a > INT32_MAX - b) {
    *rp =  (a + INT32_MIN) + (b + INT32_MIN);
    return 1;
  }
  if (b < 0 && a < INT32_MIN - b) {
    *rp =  (a - INT32_MIN) + (b - INT32_MIN);
    return 1;
  }
  *rp = a + b;
  return 0;
}

```

**UPDATE:** The prize has been awarded, see [comment 5](https://blog.regehr.org/archives/1136#comment-15397). The prize is an Amazon gift certificate for $27.49 — the cost of the kindle edition of [Hacker’s Delight](http://www.amazon.com/Hackers-Delight-Edition-Henry-Warren-ebook/dp/B009GMUMTM).
