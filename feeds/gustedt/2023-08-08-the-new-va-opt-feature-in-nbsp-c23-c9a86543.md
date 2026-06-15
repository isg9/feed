---
title: The new __VA_OPT__ feature in&nbsp;C23
url: https://gustedt.wordpress.com/2023/08/08/the-new-__va_opt__-feature-in-c23/
published: "2023-08-08T14:46:36Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3978
---

# The new **VA_OPT** feature in&nbsp;C23

C23 brings a new macro feature that eases programming with variadic macros a lot. Before it has been a challenge to detect if a `...` argument was actually empty or contained at least one real token. This feature was introduced into C23 by a paper by Thomas Köppe [“Comma omission and comma deletion”](https://open-std.org/JTC1/SC22/WG14/www/docs/n3033.htm) A similar feature had already been integrated to C++20 before.

As an example for the feature, suppose we want to augment calls to `fprintf` such that two types of problems are detected:

1. If there is only a format argument and no others, we want to use `fputs` to avoid scanning the format at execution time.
2. If there are more than one arguments, it should be assured that the format is a string literal, such that the contents can be parsed at compile time.

The principle to do this with the new `__VA_OPT__` feature is relatively simple:

```
#define fprintf(STREAM, FORMAT, ...)            \
  FPRINTF_II ## __VA_OPT__(Iplus)               \
  (STREAM, FORMAT __VA_OPT__(,) __VA_ARGS__)

```

Note the special feature, which is also new with C23, that

```
fprintf(STREAM, FORMAT)

```

and

```
fprintf(STREAM, FORMAT,)

```

are equivalent: an empty variadic macro argument is the same as having only the named arguments and omitting the comma and other trailing arguments completely.

All that remains is to give definitions for the two macros to which we dispatch.

```
#define FPRINTF_II(STREAM, FORMAT)              \
  fputs(FORMAT, STREAM)
#define FPRINTF_IIIplus(STREAM, FORMAT, ...)    \
  fprintf(STREAM, "" FORMAT "", __VA_ARGS__)

```

When used for typical `fprintf` code as this one,

```
int s1 = fprintf(stderr, "some message\n");
int s2 = fprintf(stderr, "some message %d\n", 2);
int s3 = fprintf(stderr, "some message %d %d\n", 3, 4);

```

after preprocessing the replaced code looks as follows.

```
int s1 = fputs("some message\n", stderr);
int s2 = fprintf(stderr, "" "some message %d\n" "", 2);
int s3 = fprintf(stderr, "" "some message %d %d\n" "", 3, 4);

```

That is, the call without extra arguments is replaced by a call to `fputs` (with inverted arguments) and the others have gained two extra `""` around the format string. The latter is only valid syntax if that format string is itself a string literal, and so with these simple tricks we can avoid suspicious format strings without that the compiler sees them.

Obviously, this particular macro is only an example of what you can do with the `__VA_OPT__` feature. For `fprintf` itself probably nothing is necessary, compilers nowadays are quite good to detect problems, here, but I hope you get the picture what this new feature does.

The feature officially comes with C23, but chances are that your compiler already implements this out of the box. I tested this with recent versions of `gcc` and `clang` without any problems.
