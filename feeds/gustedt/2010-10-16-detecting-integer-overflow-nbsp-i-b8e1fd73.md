---
title: Detecting integer overflow&nbsp;I
url: https://gustedt.wordpress.com/2010/10/16/detecting-integer-overflow/
published: "2010-10-16T10:15:09Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=502
---

# Detecting integer overflow&nbsp;I

A recent [discussion on stackoverflow](http://stackoverflow.com/questions/3944505/detecting-signed-overflow-in-c-c) has shown that detecting integer overflow without provoking undefined behavior need some reflection, and that the quick answers are not necessarily the best ones.

After having looked into the [anatomy of integer types](https://gustedt.wordpress.com/2010/09/21/anatomy-of-integer-types-in-c/) we are armed to tackle this question rigorously. We will use `intmax_t` as the base type for which we want to detect overflow or underflow of a simple addition. This is to make sure that this is more fun and such that nobody will have the simple excuse to just use a wider signed type for the arithmetic. You will easily be able to come up with equivalent code for other signed integer types at the end, I hope.

```
inline
intmax_t add(intmax_t lhs, intmax_t rhs) {
  if (p00_overflow(lhs, rhs))
    abort();
  else
    return lhs + rhs;
}

```

Let us look into a generic code that will detect overflow, regardless of the architecture:

```
inline
bool p00_overflow(intmax_t lhs, intmax_t rhs) {
  return (lhs >= 0)
    ? (INTMAX_MAX - lhs < rhs)
    : (INTMAX_MIN - lhs > rhs);
}

```

This works first of all because if you reformulate the two inner conditionals (as mathematical expressions) you get:

```
INTMAX_MAX < lhs + rhs
INTMAX_MIN > lhs + rhs

```

which are just the right terms that we want to test. Our addition may only overflow if `lhs` is positive, and it may only underflow if it is negative, so we are doing the correct comparison for the correct case.

Then we have to check that we don’t have any other unwanted overflow in there by itself. But this is simple to see, `INTMAX_MAX` minus a positive value will always be between `0` and `INTMAX_MAX` (including), and `INTMAX_MIN` minus a negative value will always be between `INTMAX_MIN` and `0`.

In total, our function add from above will do 2 additions (or subtractions) and check at two conditions. In a [follow up](https://gustedt.wordpress.com/2010/10/18/detecting-integer-overflow-ii/) we discuss a variant that on most modern architectures will only do 1 addition and one or two condition checks.
