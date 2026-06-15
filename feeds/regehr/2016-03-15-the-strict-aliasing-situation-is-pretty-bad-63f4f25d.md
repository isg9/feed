---
title: The Strict Aliasing Situation is Pretty Bad
url: https://blog.regehr.org/archives/1307
published: "2016-03-15T10:27:33Z"
feed: regehr
guid: http://blog.regehr.org/?p=1307
---

# The Strict Aliasing Situation is Pretty Bad

I’ll start with a quick review of the strict aliasing rules in C and C++ and then present some less well-known material.

## Strict Aliasing

Compiler optimizations are often shot down by the potential for pointers to be aliases. For example, although we might naively expect a compiler to optimize this function to return zero, that cannot happen because x and y might refer to the same location:

```
int foo(int *x, int *y) {
  *x = 0;
  *y = 1;
  return *x;
}

```

Generated code typically looks like this:

```
foo:    movl    $0, (%rdi)
        movl    $1, (%rsi)
        movl    (%rdi), %eax
        ret

```

Failure to optimize is particularly frustrating when we know for sure that x and y are not aliases. The “restrict” keyword in C was introduced to help solve this sort of problem but we’re not going to talk about that today. Rather, we’re going to talk about an orthogonal solution, the aliasing rules in C and C++ that permit the compiler to assume that an object will not be aliased by a pointer to a different type. Often this is called “strict aliasing” although that term does not appear in the standards. Consider, for example, this variant of the program above:

```
int foo(int *x, long *y) {
  *x = 0;
  *y = 1;
  return *x;
}

```

Since a pointer-to-int and a pointer-to-long may be assumed to not alias each other, the function can be compiled to return zero:

```
foo2:   movl    $0, (%rdi)
        xorl    %eax, %eax
        movq    $1, (%rsi)
        ret

```

As we see here, the aliasing rules give C and C++ compilers leverage that they can use to generate better code. On the other hand, since C is a low-level language and C++ can be used as a low-level language, both permit casting between pointer types, which can end up creating aliases that violate the compiler’s assumptions. For example, we might naively write code like this to access the representation of a floating point value:

```
unsigned long bogus_conversion(double d) {
  unsigned long *lp = (unsigned long *)&d;
  return *lp;
}

```

This function is undefined under the aliasing rules and while it happens to be [compiled into the same code that would be emitted without the strict aliasing rules](https://goo.gl/a6M2NV), it is easy to write incorrect code that looks like it is getting broken by the optimizer:

```
#include <stdio.h>

long foo(int *x, long *y) {
  *x = 0;
  *y = 1;
  return *x;
}

int main(void) {
  long l;
  printf("%ld\n", foo((int *)&l, &l));
}

```

```
$ gcc-5 strict.c ; ./a.out
1
$ gcc-5 -O2 strict.c ; ./a.out
0
$ clang strict.c ; ./a.out
1
$ clang -O2 strict.c ; ./a.out
0

```

An exception to the strict aliasing rules is made for pointers to character types, so it is always OK to inspect an object’s representation via an array of chars. This is necessary to make memcpy-like functions work properly.

So far, this is very well known. Now let’s look at a few consequences of strict aliasing that are perhaps not as widely known.

## Physical Subtyping is Broken

An [old paper that I like](http://research.cs.wisc.edu/wpis/papers/fse99.pdf) uses the term “physical subtyping” to refer to the struct-based implementation of inheritance in C. Searching for “ [object oriented C](https://www.google.com/search?ie=UTF-8&q=%22object%20oriented%20c%22&oq=%22object%20oriented%20c%22)” returns quite a few links. Additionally, many large C systems (the Linux kernel for example) implement OO-like idioms. Any time this kind of code casts between pointer types and dereferences the resulting pointers, it violates the aliasing rules. Many aliasing rule violations can be found in [this book about object oriented C](https://www.cs.rit.edu/~ats/books/ooc.pdf). Some build systems, such as Linux’s, invoke GCC with its -fno-strict-aliasing flag to avoid problems.

**Update based on some comments from Josh Haberman and Sam Tobin-Hochstadt:** It looks like the specific case where the struct representing the derived type includes its parent as its first member should not trigger UB. The language in this part of the standard is very hard to parse out.

This program [from the Cerberus project](https://www.cl.cam.ac.uk/~pes20/cerberus/) illustrates the problem with changing the type of a pointer to struct:

```
#include <stdio.h>

typedef struct { int i1; } s1;
typedef struct { int i2; } s2;

void f(s1 *s1p, s2 *s2p) {
  s1p->i1 = 2;
  s2p->i2 = 3;
  printf("%i\n", s1p->i1);
}

int main() {
  s1 s = {.i1 = 1};
  f(&s, (s2 *)&s);
}

```

```
$ gcc-5 effective.c ; ./a.out
3
$ gcc-5 -O2 effective.c ; ./a.out
2
$ clang-3.8 effective.c ; ./a.out
3
$ clang-3.8 -O2 effective.c ; ./a.out
3

```

## Chunking Optimizations Are Broken

Code that processes bytes one at a time tends to be slow. While an optimizing compiler can sometimes make a naive character-processing loop much faster, in practice we often need to help the compiler out by explicitly processing word-sized chunks of bytes at a time. Since the data reinterpretation is generally done by casting to a non-character-typed pointer, the resulting accesses are undefined. Search the web for “fast memcpy” or “fast memset”: many of the hits will return erroneous code. [Example 1](http://codereview.stackexchange.com/questions/41094/memcpy-implementation), [example 2](http://www.programering.com/a/MTMxEjMwATM.html), [example 3](http://www.xs-labs.com/en/blog/2013/08/06/optimising-memset/). Although I have no evidence that it is being miscompiled, [OpenSSL’s AES implementation uses chunking](https://github.com/openssl/openssl/blob/master/crypto/aes/aes_ige.c#L69) and is undefined.

One way to get chunking optimizations without UB is to use GCC’s may\_alias attribute, as [seen here in Musl](http://git.musl-libc.org/cgit/musl/tree/src/string/memset.c#n33). This isn’t supported even by Clang, as far as I know.

## Offset Overlap is Bad

Here is a devilish little program [by Richard Biener and Robbert Krebbers](https://gcc.gnu.org/ml/gcc/2015-03/msg00083.html) that I found via the Cerberus report:

```
#include <stdio.h>
#include <stdlib.h>

struct X {
  int i;
  int j;
};

int foo(struct X *p, struct X *q) {
  q->j = 1;
  p->i = 0;
  return q->j;
}

int main() {
  unsigned char *p = malloc(3 * sizeof(int));
  printf("%i\n", foo((struct X *)(p + sizeof(int)),
                     (struct X *)p));
}

```

It is ill-formed according to LLVM and GCC:

```
$ clang-3.8 krebbers.c ; ./a.out
0
$ clang-3.8 -O2 krebbers.c ; ./a.out
1
$ gcc-5 krebbers.c ; ./a.out
0
$ gcc-5 -O2 krebbers.c ; ./a.out
1

```

## int8\_t and uint8\_t Are Not Necessarily Character Types

[This bug](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=66110) (and some linked discussions) indicate that compiler developers don’t necessarily consider int8\_t and uint8\_t to be character types for aliasing purposes. Wholesale replacement of character types with standard integer types — [as advocated here](https://matt.sh/howto-c), for example — would almost certainly lead to interesting strict aliasing violations when the resulting code was run through a compiler that doesn’t think int8\_t and uint8\_t are character types. Happily, no compiler has done this yet (that I know of).

## Summary

A lot of C code is broken under strict aliasing. Separate compilation is probably what protects us from broader compiler exploitation of the brokenness, but it is a very poor kind of protection. Static and dynamic checking tools are needed. If I were writing correctness-oriented C that relied on these casts I wouldn’t even consider building it without -fno-strict-aliasing.

Pascal Cuoq provided feedback on a draft of this piece.
