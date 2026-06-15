---
title: Right shift on signed types is not well&nbsp;defined
url: https://gustedt.wordpress.com/2010/06/02/right-shift-on-signed-types-is-not-well-defined/
published: "2010-06-02T06:15:55Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=20
---

# Right shift on signed types is not well&nbsp;defined

The shift operators ( `<<` and `>>`) shift the bits in a word to the left or the right. From such an explanation it doesn’t follow directly what should happen with the bits at the word boundaries. There are several commonly used strategies

- *logical*:

  Bits that go beyond the word boundary are dropped and the new positions are filled with zeroes.
- *ones*:

  Bits that go beyond the word boundary are dropped and the new positions are filled with ones.
- *arithmetic*:

  1. Shift is \`logical’ for positive values.
  2. For negative values right shift is \`ones’ and
  3. left shift is \`logical’ but always sets the highest order bit (sign bit) to 1.
- *circular*: Bits that go beyond the word boundary are reinserted at the other end.

\`Arithmetic’ shift has its name from the fact that it implements an integer multiplication or division by a power of two.

For `unsigned` integer types C prescribes that the shift operators are \`logical’ . So *e.g* `(~0U >> 1)` results in a word of all ones but for the highest order bit which is 0. The picture darkens when it comes to `signed` types. Here the compiler implementor may choose between a \`logical’ and an \`arithmetic’ shift. Basically this means that the use of the right shift operator on `signed` values is not portable unless very special care is taken. We can detect which shift is implemented by the simple expression `((~0 >> 1) < 0)`

- If the shift is \`logical’ the highest order bit of the left side of the comparison is 0 so the result is positive.
- If the shift is \`arithmetic’ the highest order bit of the left side is 1 so the result is negative.

Observe in particular that in case of an arithmetic shift `(~0 >> 1) == ~0`. So this operator has two fixed points in that case, 0 and -1. If we want a portable shift we may choose the following operations

```
#define LOGSHIFTR(x,c) (((x) >> (c)) &amp; ~(~0 << (sizeof(int)*CHAR_BIT - (c)))

```

This produces a mask with the correct number of 1’s in the low order bits and performs a bitwise and with the result of the compiler shift. Observe

- This supposes that x is of type `int`, a type independent definition would be much more complicated.
- `c` is evaluated twice so don’t use side effects here.

Here is a C99 program to test your compiler.

```
#include <limits.h>
#include <stdio.h>

extern
int logshiftr(int x, unsigned c);

extern
int arishiftr(int x, unsigned c);

#define HIGHONES(c) ((signed)(~(unsigned)0 << (sizeof(signed)*CHAR_BIT - (c))))
#define HIGHZEROS(c) (~HIGHONES(c))

inline
int logshiftr(int x, unsigned c) {
  return (x >> c) &amp; HIGHZEROS(c);
}

inline
int arishiftr(int x, unsigned c) {
  return logshiftr(x, c) ^ (x < 0 ? HIGHONES(c) : 0);
}

int main(int argc) {
  int b = argc > 1 ? argc : 0;
  int val[11u] = { b, b + 1, b - 1, b + 2, b - 2, b + 3, b - 3, b + 4, b - 4, b + 5, b - 5};
  printf("shift\tvalue\t>>\tlogical\tarith\n");
  for (unsigned sh = 1; sh < 3; ++sh)
    for (unsigned i = 0; i < 11u; ++i)
      printf("%+d\t%+d\t%+d\t%+d\t%+d\n",
             sh,
             val[i],
             (val[i] >> sh),
             logshiftr(val[i], sh),
             arishiftr(val[i], sh));
}

```
