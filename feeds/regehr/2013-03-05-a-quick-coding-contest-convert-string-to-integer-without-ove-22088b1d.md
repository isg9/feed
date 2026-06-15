---
title: 'A Quick Coding Contest: Convert String to Integer Without Overflow'
url: https://blog.regehr.org/archives/909
published: "2013-03-05T22:40:09Z"
feed: regehr
guid: http://blog.regehr.org/?p=909
---

# A Quick Coding Contest: Convert String to Integer Without Overflow

**UPDATE:** As of Saturday March 9, the contest is closed. Results will be posted in a few days. I’ve created a [Github repository containing all submissions](https://github.com/regehr/str2long_contest) and my test harness.

Regular readers will know that I’m obsessed with integer overflows. One apparently simple piece of code that many programmers get wrong in this respect is converting strings to integers. It is slightly tricky. The goal of this contest is to submit the nicest (judged by me) implementation in C for this specification:

```
/*
 * if input matches ^-?[0-9]+\0$ and the resulting integer is
 * representable as a long, return the integer; otherwise if
 * the input is a null-terminated string, set error to 1 and
 * return any value; otherwise behavior is undefined
 */
extern int error;
long str2long (const char *);

```

I deliberately didn’t name the function atol or strtol since its semantics are slightly nonstandard. The regular expression above probably isn’t standard either; it is simply trying to specify a null-terminated string containing only a signed decimal integer of arbitrary size. No other characters (even whitespace) are permitted.

A few extra things:

- Don’t call library functions; your code should perform the conversion itself
- It is fine to use constants from `limits.h`
- You should write conforming C code that works on x86 and x86-64. I’ll probably only test entries using GCC and Clang but even so you are not allowed to use extensions such as GCC’s 128-bit integers
- You may assume that implementation-defined behaviors are the standard ones for the x86 and x86-64 platforms on Linux machines

- Your code must not execute undefined behaviors (use a [recent Clang to check for this](https://blog.regehr.org/archives/905))

Please mail entries to me, not only to avoid spoiling the fun but also because WordPress loves to mangle C code left in the comments. Feel free to leave comments that don’t include code, of course. I’ll announce the winner in a few days, and will send the winner a small prize. By submitting an entry into this contest you are agreeing to let me use your entry in a subsequent blog post, either as a positive or negative example.

**NOTE:** The best Clang flags to use are: `-fsanitize=integer -fno-sanitize=unsigned-integer-overflow`

**UPDATE:** As of Saturday March 9, the contest is closed. Results will be posted in a few days. I’ve created a [Github repository containing all submissions](https://github.com/regehr/str2long_contest) and my test harness.
