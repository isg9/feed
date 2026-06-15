---
title: Checked integer arithmetic in the prospect of&nbsp;C23
url: https://gustedt.wordpress.com/2022/12/18/checked-integer-arithmetic-in-the-prospect-of-c23/
published: "2022-12-18T14:03:08Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3855
---

# Checked integer arithmetic in the prospect of&nbsp;C23

As you might have noticed, C23 is scheduled to come out in November 2023 and will have a lot of improvements and new features, in particular for integers. One of the most controversial properties of integer arithmetic in C is the overflow behavior. C23 will have a new header `<stdckdint.h>` for “checked integer operations”, that helps to deal with overflow and puts the responsibility in your hands, the programmer. In addition to the result of an arithmetic operation, the interfaces provide an extra `bool` value that tells if the operation has been erroneous or not.

The addition that has been integrated into C23 is the [core proposal](https://open-std.org/JTC1/SC22/WG14/www/docs/n2683.pdf) and

[some amendments](https://open-std.org/JTC1/SC22/WG14/www/docs/n2867.pdf). The history of this looks a bit confusing because later editions of the paper have removed parts that had already been adopted.

Anyhow, for many of you it is even possible to use these features in a C23 compatible way just now, because they are closely modeled after similar gcc and clang features. Since overflow still is an important source of bugs and security issues, you should just

**start using checked integer operations, now!**

There are three new type-generic interfaces for addition, subtraction and multiplication:

```
#include <stdckdint.h>
bool ckd_add(type1 *result, type2 a, type3 b);
bool ckd_sub(type1 *result, type2 a, type3 b);
bool ckd_mul(type1 *result, type2 a, type3 b);

```

Their working is quite simple: the arithmetic is as if performed in the set of mathematical integers and then the value is written to `*result`. If it fits, the return value is `false`. If it doesn’t fit, the return value is `true`, and the stored result are the lower bits of the mathematical value, just as much bits as as do fit into the target type. For unsigned types that is exactly the same value as would have given an operation with that target type; the additional `bool` value just tells you if there has been a wrap around. For signed types, this corresponds to the bit pattern of the corresponding unsigned value, and is exactly the value that you’d expect, knowing that now two’s complement is the only sign representation that remains valid for C.

Now how can we use this already with gcc and similar compilers, if we don’t even yet have C23 compilers? To see that let’s look an example code:

```
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <stdatomic.h>

/* This is in the public domain, so you can copy this around freely.
But I would be nice if you could just link to this blog post at
https://gustedt.wordpress.com/ such that users see an explanation. */

/* C23 has three new tg interfaces in the new <stdkdint.h> header.
They are modeled after similar gcc features. They are meant to do
arithmetic with overflow check by using everything the compiler can
get from instruction flags that already exist on most CPU.

If we don't have the header, yet, we may easily emulate it if we are
on a compiler claiming compatibility with gcc. */

#if __has_include(<stdckdint.h>)
# include <stdckdint.h>
#else
# ifdef __GNUC__
#  define ckd_add(R, A, B) __builtin_add_overflow ((A), (B), (R))
#  define ckd_sub(R, A, B) __builtin_sub_overflow ((A), (B), (R))
#  define ckd_mul(R, A, B) __builtin_mul_overflow ((A), (B), (R))
# else
#  error "we need a compiler extension for this"
# endif
#endif

#if __STC_VERSION__ < 202300L
# define nullptr ((void*)0)
#endif

#ifndef TYPE
# define TYPE unsigned long long
#endif

typedef unsigned long long ullong;

// Use a fence such that the assembler is not reordered too wildly
#define fence atomic_signal_fence(memory_order_seq_cst)

int main(int argc, char* argv[argc + 1]) {
  if (argc != 2) {
    fprintf(stderr, "we need exactly 1 arguments, found %d\n", argc-1);
  } else {
    TYPE a = strtoull(argv[1], nullptr, 0);
    TYPE result_add;
    TYPE result_sub;
    TYPE result_mul;

    for (TYPE b = 1; b < 4; ++b) {
      bool add_invalid = ckd_add(&result_add, a, b);
      fence;
      bool sub_invalid = ckd_sub(&result_sub, a, b);
      fence;
      bool mul_invalid = ckd_mul(&result_mul, a, b);
      fence;

      printf("operation add for %#llX and %#llX is %s, result is %#llX\n",
             (ullong)a, (ullong)b, (add_invalid ? "invalid" : "valid"), (ullong)result_add);
      printf("operation sub for %#llX and %#llX is %s, result is %#llX\n",
             (ullong)a, (ullong)b, (sub_invalid ? "invalid" : "valid"), (ullong)result_sub);
      printf("operation mul for %#llX and %#llX is %s, result is %#llX\n",
             (ullong)a, (ullong)b, (mul_invalid ? "invalid" : "valid"), (ullong)result_mul);
    }
  }
}

```

Here in line `18` we use a new feature `__has_include` that also comes with C23 and which gcc and clang implement since a long time. So if the header exists we will be using it, if not we define three fallback macros that do the job.

If you successfully compile this into an executable, you may test the feature with some big number as the first and only command-line argument. On my machine this gives for example the following:

```
<517>$ ./test-ckd 0xFFFFFFFFFFFFFFFD
operation add for 0XFFFFFFFFFFFFFFFD and 0X1 is valid, result is 0XFFFFFFFFFFFFFFFE
operation sub for 0XFFFFFFFFFFFFFFFD and 0X1 is valid, result is 0XFFFFFFFFFFFFFFFC
operation mul for 0XFFFFFFFFFFFFFFFD and 0X1 is valid, result is 0XFFFFFFFFFFFFFFFD
operation add for 0XFFFFFFFFFFFFFFFD and 0X2 is valid, result is 0XFFFFFFFFFFFFFFFF
operation sub for 0XFFFFFFFFFFFFFFFD and 0X2 is valid, result is 0XFFFFFFFFFFFFFFFB
operation mul for 0XFFFFFFFFFFFFFFFD and 0X2 is invalid, result is 0XFFFFFFFFFFFFFFFA
operation add for 0XFFFFFFFFFFFFFFFD and 0X3 is invalid, result is 0
operation sub for 0XFFFFFFFFFFFFFFFD and 0X3 is valid, result is 0XFFFFFFFFFFFFFFFA
operation mul for 0XFFFFFFFFFFFFFFFD and 0X3 is invalid, result is 0XFFFFFFFFFFFFFFF7

```

Here you see that for the input value multiplication overflowed for values `0x2` and `0x3`, whereas for addition that happens only for value `0x3`. The object code that the compiler produces for this should be close to optimal: it should use processor flags to track the validity of the operation and optimize out stores to `*result` if only the value (and not the pointer) is used later on.

One valid use of these functions is to use them with signed integer types and to just ignore the `bool` return. This then leads to the value as it wraps around, but the behavior is always well-defined, if that is want you want or need.

As you can see, there are no interfaces for division or modulo. This is so, because these are quite different from the other arithmetic. For unsigned types there is exactly one input value that is invalid, namely `0` as the divisor. When we have that, there is no clear value or bit-pattern that would be reasonable to return as `*result`. Division by zero is a logical error that indeed should never occur, and if it does, where a real error handling path is in order.

For all other value we are fine; valid division and modulo operations always have results that are smaller than the input. (There could be a problem if we choose a result type that is too narrow, though.)

For signed values there is an additional difficulty. There is exactly one value for which negation leads to an error, namely the most negative values of the type. So for example

```
INT_MIN / -1

```

has the mathematical value `INT_MAX + 1` which obviously does not fit into an `int`.
