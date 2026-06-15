---
title: a praise of size_t and other unsigned&nbsp;types
url: https://gustedt.wordpress.com/2013/07/15/a-praise-of-size_t-and-other-unsigned-types/
published: "2013-07-15T16:17:13Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1954
---

# a praise of size_t and other unsigned&nbsp;types

Again I had a discussion with someone from a C++ background who claimed that one should use signed integer types where possible, and who also claimed that the unsignedness of `size_t` is merely a historical accident and would never be defined as such nowadays. I strongly disagree with that, so I decided to write this up, for once.

What I write here will only work with C, and can possibly extended to C++ and other languages that implement unsigned integer types, e.g good old Pascal had a `cardinal` type.

Let’s get practical and look into what type to use for array indexing, let’s call it `T`. A typical use of such an index type is something like

```
for (T i = 0; i < array_size; ++i) {
   A[i] = something_nice;
}

```

Here the first “problem” might the comparision `i < array_size`. Your compiler might already wine here, if `i` and `array_size` are of different signedness (and he is right). Everybody knows that index type and bounds should be of the same type, and if it were a variable, `array_size` should have been declared as

```
T array_size = something;

```

What if it is not a variable, but an expression? Well that expression should be of the same type. Now the difficulty arises, the type of expressions in C is a subtle subject. In general, you can only find out of the concrete type of an expression by looking up the types of all of its components. If we suppose that they are all of the same width and

- all signed, the result is signed
- all unsigned, the result is unsigned
- mixed, the result is unsigned

(might be a bit different on some historical platforms, but should be a safe bet for the last 20 years.)

Now bound expressions often contain operands that are unsigned, for the simple reason that `size_t` is a unsigned type: a typical “trick” to compute the number of elements in an array is

```
sizeof A / sizeof A[0]

```

which is of type `size_t`. E.g for the above simple loop we have

```
for (T i = 0; i < sizeof A / sizeof A[0]; ++i) {
   A[i] = something_nice;
}

```

and so `T` should be an unsigned type.

Another typical thing to do with indices is to check if they are in bounds. Usually the lower bound is 0 and the upper bound is some value. Typical code for a more complicated iteration would be

```
for (T i = start_value; in_bounds(i); i = update(i)) {
   A[i] = something_nice;
}

```

How would you then implement `in_bounds`? If `T` is signed you’d do

```
bool in_bounds(T i) {
   return i >= 0 && i < some_value;
}

```

whereas if it is unsigned you’d have something simpler:

```
bool in_bounds(T i) {
   return i < some_value;
}

```

This works independently of `update(i)`, even if that would decrement `i`. Provided that our array A has at least 42 elements the following works like a charm with `size_t`.

```
for (size_t i = 41; i < sizeof A / sizeof A[0]; --i) {
   A[i] = something_nice;
}

```

Such computations always have well defined behavior, unsigned types just wrap around below 0 or above their maximal value. No traps, no signals, no exceptions.

In general index calculations with unsigned types just work, whereas for signed types you’d often have to think of casting some of the boundary expressions and treat special cases. So using an unsigned type for `size_t` is not a historical accident, it is a necessity if you want to have things simple. And other than most other modern programming languages, C is made to be simple and straight.

Now why use `size_t` and not `unsigned`? Because it has the width that you’d need to address any kind of object on your platform. E.g doing complicated index computations for matrices easily overflows if you use a narrow unsigned type.
