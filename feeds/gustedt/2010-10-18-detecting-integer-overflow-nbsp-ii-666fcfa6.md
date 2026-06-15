---
title: Detecting integer overflow&nbsp;II
url: https://gustedt.wordpress.com/2010/10/18/detecting-integer-overflow-ii/
published: "2010-10-18T22:21:11Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=531
---

# Detecting integer overflow&nbsp;II

In [an earlier post](https://gustedt.wordpress.com/2010/10/16/detecting-integer-overflow/) we came up with a general solution to check for potential under- or overflow in an integer addition. On most modern architectures this can be done more efficiently, even when assuming that there are no special instructions that capture overflow bits or such.

The key observation is that in the generic version we are doing two additions (resp. subtractions) instead of one. On most modern architectures there is hope that we may be able to avoid that. In particular, this should be possible if the unsigned type has one value bit more than the unsigned type. So let us assume just that for the moment and see what we can gain.

First of all there is a simple way to detect this situation in a constant expression, here again with `intmax_t` and `uintmax_t` as integer types:

```
(INTMAX_MAX < UINTMAX_MAX)

```

this will be true if and only if there is at least one more value bit in `uintmax_t`.

To be able to use the result of an addition that we perform first to check for validity and then for the result, we need an important feature namely to convert an unsigned value into a signed one in a meaning full way, without producing a range error. The easiest way is to mimic what is done if two’s complement is the target integer representation:

```
inline
intmax_t
p99_twosj(uintmax_t a) {
  uintmax_t const type_max = INTMAX_MAX;
  uintmax_t const type_max1 = type_max + 1;
  /* the unsigned max, as if it had just one value bit more */
  uintmax_t const utype_max = (2 * type_max) + 1;
  return
    /* for positive values there is nothing to do, this includes the
       case where the unsigned type has the same number of value bits
       as the signed type */
    (a <= type_max)
    ? a
    /* Capture the special case where type_max1 is a trap
       representation for the signed type */
    : (((INTMAX_MIN == -INTMAX_MAX) && (a == type_max1))
       ? -0
       /* otherwise compute the negative modulo utype_max + 1. for
          the case that the unsigned type is much wider than the
          signed type we mask the higher order bits away. */
       : (-(intmax_t)(utype_max - (utype_max & a))) - 1);
}

```

The comments in the code should speak for themselves, only the last line needs some more explanation I think. The “natural” way to write this would be

```
-(intmax_t)(utype_max - (utype_max & a) + 1))

```

that is to add the `1` before the negation. But for a corner case this could lead to a value of `type_max1` before the cast into `intmax_t`, by definition an overflow.

So with this function `p99_twosj` we now have one that does the reasonable translation from the unsigned type to the signed. This will work regardless of the sign presentation, with the only exception when the result might be `type_max1` which might not be transformed to a valid negative value. So we don’t have to forget to cover that case later when doing the real “add” function.

Now OK, it seems to work, but is it efficient? The answer is “yes”, at least on the limited architecture that I tried (64bit intel) with three different compilers: `p99_twosj` translates to the identity function on them, that is the argument register is just passed through to the output register, without any change.

To do the inverse, we just have to take care of the fact that the unsigned type might even have more than one value bit more

```
inline
uintmax_t
p99_unsigj(intmax_t a) {
  uintmax_t const type_max = INTMAX_MAX;
  uintmax_t const type_max1 = type_max + 1;
  /* the unsigned max, as if it had just one value bit more */
  uintmax_t const utype_max = (2 * type_max) + 1;
  return
    (a >= 0)
    ? a
    /* Capture the special case where -INTMAX_MIN can not represented
       in the signed type */
    : (((INTMAX_MIN < -INTMAX_MAX) && (a == INTMAX_MIN))
       ? type_max1
       /* otherwise compute the negative modulo utype_max + 1. */
       : (utype_max - (uintmax_t)-a) + 1);
}

```

Now let’s come to the adding function itself. We split it in two, one for the unsigned addition and one for the tests and conversions:

```
inline
uintmax_t
p00_add0j(intmax_t a, intmax_t b) {
  register uintmax_t ua = p99_unsigj(a);
  register uintmax_t ub = p99_unsigj(b);
  register uintmax_t res = ua + ub;
  return res;
}
p99_inline
intmax_t
p00_addj(intmax_t a, intmax_t b, int* err) {
  _Bool a0 = (a < 0);
  _Bool b0 = (b < 0);
  register uintmax_t uc = p00_add0j(a, b);
  register intmax_t c = p99_twosj(uc);
  _Bool c0 = (c < 0);
  if ((a0 == b0) && P99_EXPECT((a0 != c0), 0))
    if (err) *err = ERANGE;
  return c;
}

```

`p00_add0j` does the conversion into unsigned values and the addition. `p00_addj` now computes three checks of signs, for the two input values and the return value. If the first two have the same sign and the third differs from that one, then an underflow ( `a` and `b` are negative) or overflow ( `a` and `b` are positive) occurred.

For the efficiency, `gcc` gets along quite well and produces compact and readable assembler. `clang` produces essentially the same, it is spilling one register too much. But it does this by requiring substantially more compile time and memory. opencc is not fit enough to tackle this code satisfiable: the assembler is lengthy and has remaining traces of the handling of special cases in the code.
