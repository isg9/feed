---
title: Anatomy of integer types in&nbsp;C
url: https://gustedt.wordpress.com/2010/09/21/anatomy-of-integer-types-in-c/
published: "2010-09-21T04:06:31Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=366
---

# Anatomy of integer types in&nbsp;C

Integer types in C may present subtle traps (sic!) that many people are not aware of when doing seemingly simple things like `~0` or

```
(1 << (sizeof(int)*CHAR_BIT - 1))

```

Most times, on almost all processors these will produce the desired effects, but sometimes such a code will fail, crash, spit a lot of warnings. I will try to analyze this a bit, to show what may go wrong, here, and how you can get around a lot of possible problems.

This is a discussion that is a bit involved, you should know a bit of things about [integer types](https://gustedt.wordpress.com/2010/06/24/integer-type-confusion/?preview=true&preview_id=104&preview_nonce=05c3e98612) and how to use them.

## The simple case: `unsigned char`

The first and only case that is guaranteed to be simple are `unsigned char`. On most commodity architectures this will correspond to `uint8_t`, but to be most general let us assume here that it has `CHAR_BIT >= 8` bits. The C99 standard enforces that this type takes values from `0` to `UCHAR_MAX = M - 1` where *M*, the modulus, is given exactly by the formula `2CHAR_BIT`.

All computation for `unsigned char` is computation modulo *M*: e.g for `CHAR_BIT == 8` the result of `128 + 127` when we assign it to an `unsigned char` is `255`, `128 + 128` is `0` and the result of `128 + 129` is `1`.

There are no hidden bits in an `unsigned char`. The ith bit (counting from 0) is simply set by something like:

```
(((unsigned char)1) << i)

```

Simple and clean.

## Other unsigned types: the first traps appear

In most aspects other unsigned types, say *T*, that are not `_Bool` are like `unsigned char`. They have a modulus *MT* (generally different from the one for `unsigned char`), *MT = 2w,*, the maximum value is *MT-1*, computation is done modulo *MT*, and the bits can be accessed in a similar way as before. Good.

( `_Bool` is special since converting any value that is not `0` to `_Bool` results in `1`.)

The only thing is, that *MT* has not much to do with the *size* of the type: `sizeof(T)*CHAR_BIT` is only an upper bound for the width *w*, but it mustn’t forcibly be that value. There can be bits that don’t contribute to the value of the variable, so-called *padding* bits. The system may e.g use such padding bits for

1. nothing, i.e never access these bits
2. garbage, access them occasionally, e.g in case of “overflow”, but never clear them
3. error correction, make sure that one bit error in the variable can be repaired
4. the sign bit of the corresponding signed type

The first two cases are relatively harmless, although for 2 we already may have a hard time to distinguish it from 3 or 4. Either of the two cases appears on every C99 platform: `_Bool` is unsigned (it has no negative values) and it only has 1 value bit. Since it must at least have a width of `char`, it has at least 7 padding bits. A platform may use the padding bits or not, just storing `1` for a true value or `UCHAR_MAX`.

The case 3 is a hard one already, since it may lead to a so-called *trap representation*, i.e an invalid bit pattern that is stored in an object. If we suppose that we manipulate the bit pattern directly, placing a wrong bit in the padding bits can mix up the error correction and signal an error to the application. On a POSIX system, accessing such a mixed up object might signal an arithmetic exception ( `SIGFPE`) that leads to the termination of the program.

The case 4 is really nasty. Below when discussing signed types we will see more on that.

## Determine the width of an unsigned type

If we suppose that for our unsigned type *T* we want to find the width. From what we know this is now easy: `(T)-1`, minus one transformed into a value of type *T* will always be *M-1*. Since *M* is forcibly a power of two, we just have to check for the values w that are less or equal to `sizeof(T)*CHAR_BIT`, easy. In the following we will assume that we have a macro `P99_TWIDTH` that does that work for us.

## Signed integer types: at a first glance

Signed integer types only add a little tiny bit to all that, namely the *sign bit*. This tells when a value is positive (sign bit 0) or negative (sign bit 1). Now there are several possibilities on how to interpret the bit pattern of a signed integer as a whole. C allows the compiler implementor to choose between three different representations:

**sign and magnitude**:this is probably the most natural one for people not too involved in bit manipulation. The value bits determine the magnitude of the value and the sign bit, well, the sign.**two’s complement**:the sign bit has the symbolical value *-2N***one’s complement**:the sign bit has the symbolical value *-2N-1*

Thus for the later two, we take the value that is given by the value bits and then subtract a large value if the sign bit is set, that makes it negative.

So why are there three different such representations? Because each has a specific problem. For the first and the third we find two representations with value 0. For *sign and magnitude* this is obvious; the value 0 can have the sign bit switched on or off. For *one’s complement* the largest value for the value bits *is* *-2N-1*, so subtracting that same value leads to 0. The encodings of 0 with sign bit on in these representations are called *negative zero* s. Whether or not these are trap representations (crash your application, raise a signal, do something weird…) is left to the compiler implementor.

So when manipulating bits we must be careful not to run in such a trap. But this is exactly what we do when we complement all the bits of `0`: `~0` can lead to a trap representation when we have to deal with one’s complement.

The other representation, *two’s complement*, does not have that problem, and this one is the chosen one for most modern platforms. Unfortunately it has another problem for the minimal allowed value. The minimal *possible* value is simply *MIN0 = -2N*, namely if the sign bit is on and all other bits are off. But this value has a bizarre property: the positive value *-MIN0* has no representation in the type. We may choose among to evil

1. We allow a value of *MIN0*, but then the prefix operation “-” can raise a range condition since *-MIN0* is not representable.
2. We forbid *MIN0* i.e mark it as a trap representation, potentially signaling an error if we do some bit manipulations.

So if MIN and MAX are the smallest and greatest value of the type besides the usual arithmetic under- or overflow we have to avoid the following computations

- *~0*, may lead to trap for one’s complement.
- *~MAX*, may lead to trap for sign and magnitude.
- *-MIN*, may lead to range error for two’s complement.

## Fitting unsigned and signed types together

To (almost) each unsigned integer type there is a signed one of the same size, the only exception being `_Bool` for which it makes not much sense to have a signed counterpart. Even if the sizes of \`unsigned int\` and \`signed int\` are the same, their widths *Nu* and *Ns* are not necessarily so. The standard only imposes that the number of value bits (the so-called precision) of the signed type is no larger than of the unsigned type. Expressed in terms of the width: *Ns ≤ Nu+1*.

The most common way the mapping between the bits of the unsigned and signed is as follows:

**unsigned**value**signed**signvalue

That is the sign bit of the signed is the highest order bit of the unsigned. Easy.

Things get a little more nasty if there is common padding:

**unsigned**paddingvalue**signed**paddingsignvalue

This can be handled, but you’d have to avoid the common error to take `sizeof(T)*CHAR_BIT` for the width of the type.

To have fewer bits in the signed than in the unsigned is also possible:

**unsigned**paddingvalue**signed**paddingsignvalue

But such a thing is not very likely, technically it makes not much sense.

Comes the last one, the really nasty:

**unsigned**paddingvalue**signed**paddingsignvalue

Here the sign bit is hidden in the padding bit(s) of the unsigned.

- We have no legal way to manipulate the sign bit through operations on the unsigned.
- If the representation is two’s complement and *MIN0* is allowed, *-MIN0* is not even representable as an unsigned value.
- A simple function like the following `abs` might signal an error on *MIN0*.

  ```
  unsigned abs(int a);

  ```

Fortunately platforms with this representations of `signed` are exotic. Historically, the reason for implementing such a design had been processors that did only implement signed arithmetic. There the easiest way for a compiler implementor is just to take the sign bit as an \`overflow’ indicator for the unsigned types. Whether or not on such platforms the implementors had at least the pity to drop *MIN0* from view I don’t know. Also I hope that nobody will ever port a C99 compiler to such a beast.

## Getting access to all the bits

Fortunately, for `unsigned char` and `signed char` the later scenario can’t happen since `unsigned char` is not allowed to have padding bits. A union such as

```
union overlay {
   T x;
   unsigned char X[sizeof(T)];
};

```

will always allow you to access each individual bit of `x`, regardless what type `T` represents. But beware, no other type than `unsigned char` should be used for such a trick: `signed char` may having padding bits, and `char` you don’t even know the signdness of that beast, don’t you.

Edit: Since C11 `signed char` can’t have padding bits, either, but it still could have a trap representation.
