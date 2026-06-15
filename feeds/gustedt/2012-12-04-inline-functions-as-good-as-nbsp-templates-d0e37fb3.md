---
title: inline functions as good as&nbsp;templates
url: https://gustedt.wordpress.com/2012/12/04/inline-functions-as-good-as-templates/
published: "2012-12-04T23:24:33Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1869
---

# inline functions as good as&nbsp;templates

I recently started to implement parts of the “Bounds checking interfaces” of C11 (Annex K) for P99 and observed a nice property of my implementation of `qsort_s`. Since for P99 basically all functions are inlined, my compilers (gcc and clang) are able to integrate the comparison functions completely into the sorting code, just as an equivalent implementation in C++ would achieve with `template` code.

Probably nobody has noticed the function `qsort_s` from that obscure Annex K, yet, so I note its interface here:

```
errno_t qsort_s(void *base,
                rsize_t nmemb,
                rsize_t size,
                int (*compar)(const void *x, const void *y, void *context),
                void *context);

```

As you can see it is a bit different from the usual `qsort`, since it allows to provide `context` to the comparison function `compar`. So other than for `qsort`, for `qsort_s` there is no need to control a complicated comparison function through global variables, all information that it needs can be passed through the `context` pointer. Good.

I thought at a first that such a function with a `void*` argument without any type information would be even harder to optimize than `qsort`. When looking closer I was really positively surprised. As an example I used the following comparison functions:

```
inline
int uComp0(void const*a, void const*b) {
  register uint64_t const*const A = a;
  register uint64_t const*const B = b;
  {
    register uint64_t const a = *A;
    register uint64_t const b = *B;
    return (a < b) ? -1 : (a > b);
  }
}

inline
int uComp(void const*a, void const*b, void*c) {
  if (c) {
    register uint64_t const*const A = a;
    register uint64_t const*const B = b;
    register uint64_t const*const C = c;
    {
      register uint64_t const c = *C;
      register uint64_t const a = *A % c;
      register uint64_t const b = *B % c;
      return (a < b) ? -1 : (a > b);
    }
  } else
    return uComp0(a, b);
}

```

Here the function `uComp0` would be a comparison function on 64 bit integers as we could use it for `qsort`. `uComp` artificially adds the difficulty that if the third (=context) parameter is not null, it is interpreted as pointing to a modulus for the integers. Although this function looks a bit complicated, a decent compiler should be able to produce code of about 20 to 25 assembler statements for it; most of the C is just clutter, re-typing of pointers and such things. But, as the original C code, such an assembler always has to test the condition on `c`.

Now look at different calls to the `qsort_s` function

```
uint64_t array[n];
uint64_t modulus = argc;
.
.
qsort_s(array, n, sizeof *array, uComp, 0);
qsort_s(array, n, sizeof *array, uComp, &modulus);

```

For the first call the compilers are capable of inlining all calls to `uComp` and the to cut off all the “if” branches of that inlined code. Since the context is `0` the branch cannot be taken. For the second call, the compilers note that only the “else” path can be taken. Also they deduce that the value of `modulus` cannot change inside the expanded call and load that modulus only at the beginning and reuse its value afterwards.

So, a simple interface with function pointers nowadays is as capable as a C++ `template` declaration, but it is even easier to use. No need to declare an artificial data type that would encapsulate an `uint64_t` just to attach this special comparison, readable (well, as readable as other C) and easy to maintain.

———

Hm, to nice to be as true as that. The real implementation of `qsort_s` is a bit more complicated than that, because of another functionality that it needs and that is only implicit, not present in its interface, namely a copy function for the elements. Without an explicit specification of such a function, the compiler has difficulties to know about the alignment requirements for the copy operation and only plain calls to `memcpy` could be used.

This is the point where C++ wins a bit, there an encapsulating class would implicitly also define a copy operator. Such an operator would provide the alignment information to the compiler.

The new C11 feature `_Alignof` helps to work around this difficulty, in that case, but makes the implementation of `qsort_s` a bit more clumsy than I would wish.

———

Another thing that could perhaps observed from all this, is that the “Bounds checking interfaces” of C11 in Annex K brings in interesting things. Unfortunately people are quite reluctant in implementing or using these interfaces. Seemingly without looking at the contents, but just because it has been proposed by the big software company that still has a very bad reputation in the C community, Microsoft. I probably will present these interfaces in more detail in a future post, the interfaces themselves are not as obscure, as their presentation in the C11 standard might seem, and because they transform a lot of undefined behavior into a clean concept of “runtime constraint violation”.
