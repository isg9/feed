---
title: Enforced bounds checking for frozen function&nbsp;interfaces
url: https://gustedt.wordpress.com/2023/06/10/enforced-bounds-checking-for-frozen-function-interfaces/
published: "2023-06-10T13:20:51Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3921
---

# Enforced bounds checking for frozen function&nbsp;interfaces

Many functions in the C library and other legacy codes have interfaces that don’t permit the current bounds checking syntax for array parameters, but that are frozen in time such that nobody will ever be able to change and improve them. In this example we go with

```
void* memcpy(void*restrict s1, void const*restrict s2, size_t n);

```

but in fact any existing interface that has size parameters after array or pointer parameters that depend on them has similar problems. The intent of this post is to show how the corresponding header can be easily tuned, such that modern compilers are able to provide static analysis that diagnoses calls that receive buffers that are too small compared to `n`. The trick to do so is just a two-level wrapper that interchanges call arguments, nothing very deep, and not something that compilers wouldn’t be able to perform if we just had means to specify this property in the language.

Here, the available interface features allow to encode important information about the function arguments of `memcpy`:

- Both arguments may be pointers or arrays of arbitrary object types (because the types are `void`)
- The buffer pointed to by the first argument might be modified by the function (because there is no qualifier on the base type).
- The buffer pointed to by the second argument will not be modified by the function (because there is a `const` qualifier on the base type).
- The pointed-to buffers must not overlap (because both are qualified with `restrict`)

Not visible in that syntax is the imperative that both buffers have to be at least `n` bytes wide.

If we just reorder the parameters and transform them into arrays we can give an alternative interface that expresses this requirement as well:

```
[[__maybe_unused__]]
static inline
void* memcpy_swpd(size_t n,
                  unsigned char s1[restrict static n],
                  unsigned char const s2[restrict static n])
                  [[__unsequenced__]]
{
  // This captures a possible pre-existing macro for memcpy
  return memcpy(s1, s2, n);
}

```

Unfortunately, we have to use the type `unsigned char`, which is C’s native type for bytes, instead of `void`, so the function cannot be called as easily as `memcpy`. Also this interface cannot directly be used instead of the C library call, and so we cannot use it to identify bounds errors in existing code.

With a macro definition for `memcpy` we can change that

```
#define memcpy(S1, S2, N)                              \
memcpy_swpd((N),                                       \
            /* Make sure no qualification gets lost */ \
            (void*){ (S1), },                          \
            /* Make sure no volatile gets lost */      \
            (void const*){ (S2), })

```

So this takes the arguments in the order in which `memcpy` expects them and dispatches them to a call to `memcyp_swpd`

There is a complication, because the original functions has `void` pointers and we want the macro to accept the same arguments as the original `memcpy`. So here we have to convert the arguments to `void` pointers such that this function can be called without troubles.

We don’t want to use simple casts to `void*`, for example, because that would cast away all other type checks that we still want to maintain. For example with casts we would not be able to detect if the arguments were non-zero integers, errors that are easily detected by the function interface. Therefore we use compound literals where the arguments `S1` and `S2` are initializers. By that, only an argument type that has an implicit conversion to the corresponding `void` pointer type can be used, all others will be diagnosed.

Now whether or not such an approach detects bound errors depends a lot on the compiler that we will actually use to compile our code. Let’s look at four trivial examples where `memcpy` is used to copy a string literal into a compound literal.

```
  puts(memcpy((char[6]){ 0 }, "hello", 6));
  // Erroneous target buffer, should not be qualified. diagnostic?
  puts(memcpy((char volatile[6]){ 0 }, "he1lo", 6));
  // Erroneous use of target buffer, using 6 where there are only 5. diagnostic?
  puts(memcpy((char[5]){ 0 }, "he2lo", 6));
  // Erroneous use of source buffer, using 7 where there are only 6. diagnostic?
  puts(memcpy((char[7]){ 0 }, "he3lo", 7));

```

Here, all the arguments to the calls have a fixed array size and type, and so errors can in principle be detected at compile time. If you test this code with your favorite compiler you should at least see a diagnostic for the second call, indicating that the `volatile` qualification is not appropriate.

The other two errors are not so easily detected, although for this special case of a C library function the compiler could have enough knowledge about the expected buffer sizes. In fact, for the third call the compound literal is too small, for the fourth it is the string literal.

Indeed, a recent clang 17 compiler detects the error in the third call, but not the fourth:

```
generic.c:41:8: warning: 'memcpy' will always overflow; destination buffer has size 5, but size argument is 6 [-Wfortify-source]
   41 |   puts(memcpy((char[5]){ 0 }, "he2lo", 6));
      |        ^

```

Gcc 13 is a bit better, here, and detects both:

```
generic.c:41:8: warning: ‘memcpy’ forming offset 5 is out of the bounds [0, 5] of object ‘({anonymous})’ with type ‘char[5]’ [-Warray-bounds=]
   41 |   puts(memcpy((char[5]){ 0 }, "he2lo", 6));
      |        ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
generic.c:41:26: note: ‘({anonymous})’ declared here
   41 |   puts(memcpy((char[5]){ 0 }, "he2lo", 6));
      |                          ^
generic.c:43:8: warning: ‘memcpy’ forming offset 6 is out of the bounds [0, 6] [-Warray-bounds=]
   43 |   puts(memcpy((char[7]){ 0 }, "he3lo", 7));
      |        ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

```

Now if we compile the same test with the macro replacement as described above, we expose the errors in the third and fourth usage. Gcc 13 is indeed able to issue diagnostics, here for the third call:

```
generic.h:394:1: warning: ‘memcpy_swpd’ accessing 6 bytes in a region of size 5 [-Wstringop-overflow=]
  394 | memcpy_swpd((N),                                    \
      | ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  395 |         /* Make sure no qualification gets lost */  \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  396 |         (void*){ (S1), },                           \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  397 |         /* Make sure no volatile gets lost */       \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  398 |         (void const*){ (S2), })
      |         ~~~~~~~~~~~~~~~~~~~~~~~
generic.c:48:8: note: in expansion of macro ‘memcpy’
   48 |   puts(memcpy((char[5]){ 0 }, "he2lo", 6));
      |        ^~~~~~
generic.h:394:1: note: referencing argument 2 of type ‘unsigned char[]’
  394 | memcpy_swpd((N),                                    \
      | ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  395 |         /* Make sure no qualification gets lost */  \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  396 |         (void*){ (S1), },                           \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  397 |         /* Make sure no volatile gets lost */       \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  398 |         (void const*){ (S2), })
      |         ~~~~~~~~~~~~~~~~~~~~~~~
generic.c:48:8: note: in expansion of macro ‘memcpy’
   48 |   puts(memcpy((char[5]){ 0 }, "he2lo", 6));
      |        ^~~~~~
generic.h:394:1: note: referencing argument 3 of type ‘const unsigned char[]’
  394 | memcpy_swpd((N),                                    \
      | ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  395 |         /* Make sure no qualification gets lost */  \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  396 |         (void*){ (S1), },                           \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  397 |         /* Make sure no volatile gets lost */       \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  398 |         (void const*){ (S2), })
      |         ~~~~~~~~~~~~~~~~~~~~~~~
generic.c:48:8: note: in expansion of macro ‘memcpy’
   48 |   puts(memcpy((char[5]){ 0 }, "he2lo", 6));
      |        ^~~~~~
generic.h:368:7: note: in a call to function ‘memcpy_swpd’
  368 | void* memcpy_swpd(size_t n,
      |       ^~~~~~~~~~~

```

This might be a bit verbose, but it clearly identifies the problem. For the fourth call we get:

```
generic.h:394:1: warning: ‘memcpy_swpd’ reading 7 bytes from a region of size 6 [-Wstringop-overread]
  394 | memcpy_swpd((N),                                    \
      | ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  395 |         /* Make sure no qualification gets lost */  \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  396 |         (void*){ (S1), },                           \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  397 |         /* Make sure no volatile gets lost */       \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  398 |         (void const*){ (S2), })
      |         ~~~~~~~~~~~~~~~~~~~~~~~
generic.c:50:8: note: in expansion of macro ‘memcpy’
   50 |   puts(memcpy((char[7]){ 0 }, "he3lo", 7));
      |        ^~~~~~
generic.h:394:1: note: referencing argument 3 of type ‘const unsigned char[]’
  394 | memcpy_swpd((N),                                    \
      | ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  395 |         /* Make sure no qualification gets lost */  \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  396 |         (void*){ (S1), },                           \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  397 |         /* Make sure no volatile gets lost */       \
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  398 |         (void const*){ (S2), })
      |         ~~~~~~~~~~~~~~~~~~~~~~~
generic.c:50:8: note: in expansion of macro ‘memcpy’
   50 |   puts(memcpy((char[7]){ 0 }, "he3lo", 7));
      |        ^~~~~~
generic.h:368:7: note: in a call to function ‘memcpy_swpd’
  368 | void* memcpy_swpd(size_t n,
      |       ^~~~~~~~~~~

```

So the compiler is in fact capable of telling us that here a read access (other then for the previous that was a potential write access) moves beyond the object.

Unfortunately clang 17 is not able to detect these errors and gives no diagnostic.

You may also have been wondering what the weird additions enclosed in `[[ ]]` to the definition of `memcpy_swpd` are about. These are so-called attributes, a new feature in C23. The first **`maybe_unused`** applies to the function definition as a whole to inform the compiler that indeed this function might never be used in the current translation unit. This avoids diagnostics for `static` identifiers in header files.

The second, **`unsequenced`** is more involved. It attributes a property to the function type, namely that this function only accesses

- its arguments, and
- any objects that are reachable through pointer arguments.

No other state of the program, such as global variables, is affected. This is an extension of what in CS usually would be called a *pure* function. As a consequence, calls to this function may be moved around, e.g hoisted out of a loop, as long as the data dependencies for the call arguments and the return value are not changed.

Both attributes are specified with leading pairs of underscores, to avoid that they macro-expand in case that the user had a macro with the same name.
