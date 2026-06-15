---
title: type-safe parametric polymorphism
url: https://gustedt.wordpress.com/2021/10/18/type-safe-parametric-polymorphism/
published: "2021-10-18T05:30:00Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3782
---

# type-safe parametric polymorphism

Among the other new proposals for C23 that WG14 received are two brilliant gems by Alex Gilding

- [The `constexpr` specifier](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2851.pdf)
- [The `void`– *which-binds*: typesafe parametric polymorphism](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2853.pdf)

The first is probably what you’d expect if you know the C++ feature and how it started (I think in C++11), namely a possibility to specify constants for any type (if used to declare an object) or to have simple function calls that can be forced to be evaluated at compile time. There are still some rough edges to cut (storage class, linkage) but basically, if Wg14 weren’t so predictably unpredictable, this should be a homerun.

The other one is on a whole different level, eye opening and very, very promissing. Its main idea is to add annotations to APIs to avoid breaking ABIs and code duplication at the same time.

Let’s take a first example what the features that are proposed can do. For the remainder of this post I will be assuming that we have a C23 compiler that implements the features.

```
void * memcpy(void * restrict s1, const void *restrict s2, size_t n);

```

This function uses the catch-all type `void` to indicate that the pointers that it receives as arguments may point to any bytes, as long as for both there are at least `n` bytes available, the two byte arrays don’t overlap, and as long as the object pointed-to by `s1` is mutable.

Now in 99% of the cases the types of the objects that are pointed-to are probably the same and the byte values that are copied from `s2` into `s1` will combine to a sensible object in the target. Problems occur if the pointed to objects have different size or if they even have different type. Even if `float` has 32 bit the following code is erroneous, because it overwrites an object with floating point type by a representation of an integer type, and the resulting representation might not be valid.

```
uint32_t a = 4288912;
float x;
memcpy(&x, &a, sizeof(x)); // semantic error, don't do this!
printf("an unsigned as a float: %g", x); // may trap

```

The remedy using features from the above papers looks as follows

```
constexpr void [[bind_var(A)]]* (*cpy_typed)(void [[bind_var(A)]]* restrict s1, const void [[bind_var(A)]]*restrict s2, size_t n) = memcpy;
...
uint32_t a;
float x = 45.9;
cpy_typed(&a, &x, sizeof(a)); // Contraint violation, diagnosis required!

```

So what had been undefined behavior above, now becomes a hard error (constraint violation as we call it in C), but the function that is used underneath, `memcpy`, is still exactly the same.

The trick is to annotate the `void*` pointers of the orginal interface with *attributes* a new C23 feature (borrowed from C++) that allows to annotate precice spots of the syntax. Here it is three times the same attribute `[[bind_var(A)]]`, indicating that the code that uses this interface is expected to pass in two pointers that have the same base type named `A` (which ever that is) and also to return a pointer to an object of that same base type `A`.

Now all this specification is put on a function pointer `cpy_typed` that is `constexpr` and initialized to `memcpy` (scroll to the end of the line to see this.)

No extra code is generated, but type safety is improved!

In C23 with type-generic lambdas, if that is accepted, you could do something like this

```
#define cpy_typed(S1, S2, N)                            \
[](auto* s1, typeof(*s1) const* s2, size_t n) {         \
  return (typeof(*s1)*)memcpy(s1, s2, n); }(S1, S2, N); \
}(S1, S2, N)

```

Using such a lambda would provide the same type checks, but at the potential cost of efficiency and code size. We would be depending on the optimization capacities of the compiler, the call to `memcpy` would often have an additional indirection and converting the lambda to a function pointer could imply the generation of an additional stub each time.

The new feature becomes even more interesting when we look into functions that receive a callback to perform a task that depends on such a function. Again, let’s look at a function of the C library

```
void qsort(void *base, size_t nmemb, size_t size, int (*compar)(const void *, const void *));

int comp_double(void const* x, void const* y) {
    double const* X = x; double const* Y;
    return (*X < *Y) ? -1 : (*X == *Y ? 0 : +1);
}
...
double   dArr[5] = { 4.0, 2.2, 5.1, 6.3, 9.4, };
unsigned uArr[5] = { 4, 2, 5, 6, 9, };

qsort(dArr, 5, sizeof dArr[0], comp_double); // ok
qsort(uArr, 5, sizeof uArr[0], comp_double); // undefined

```

If you are not used to this kind of syntax, this is a function that sorts the array to which `base` points. That array is supposed to be composed of `nmemb` elements of size `size` and `compar` is a pointer to a comparison function that receives two pointers, compares the objects and returns values less, equal or greater then `0` if the first is smaller, equal or greater. The underlying type for `base` and the sort order that `compar` describes can be arbitrary.

Now, again C just puts the responsibility that all of this is consistent on your (the programmers) shoulders. For now there is no mechanism in C that could provide even the simplest consistency checks. On the other hand we gain one function, one single external symbol in the C library that is capable of doing a generic sort. Usually it is relatively efficient and small.

How this changes with the proposed papers? First this could be augmented in a similar way by adding attributes

```
constexpr void (*sort_typed)(void [[bind_var(A)]] *base, size_t nmemb, size_t size, int (*compar)(const void [[bind_var(A)]]*, const void [[bind_var(A)]]*)) = qsort;

```

So now this is a function pointer that points to the `qsort` function, but that ensures that it can only be called with consistent types.

```
int comp_double(void const [[bind_type(double)]]* x, void const [[bind_type(double)]]* y) {
    double const* X = x; double const* Y;
    return (*X < *Y) ? -1 : (*X == *Y ? 0 : +1);
}
...
double   dArr[5] = { 4.0, 2.2, 5.1, 6.3, 9.4, };
unsigned uArr[5] = { 4, 2, 5, 6, 9, };

sort_typed(dArr, 5, sizeof dArr[0], comp_double); // ok
sort_typed(uArr, 5, sizeof uArr[0], comp_double); // hard error

```

So now the compiler knows that a function is expected that receives pointer the same pointer target type than `dArr` and `uArr`. For `dArr` this base type is `double` so everything works out fine. For `uArr` it would be expecting `unsigned` and so it is able to alert the programmer that the specific call is inconsistent.

Here we have the second attribute syntax `[[bind_type(double)]]` that is introduced by the paper. It ensures that `comp_double` can only be called with pointers coming from `double`. For the function definition it also guarantees that the parameters can only be silently converted to pointers to `double` and nothing else.

So instead of blowing up the C library by an unbounded number of functions (as for example `templates` in C++ or type-specific functions and type-generic interfaces to them in C) Alex’ trick helps us to maintain exactly the one function that has been in the C library since the epoch (no ABI change), and to augment the capacity of the compiler to detect inconsistencies (a change in the API).
