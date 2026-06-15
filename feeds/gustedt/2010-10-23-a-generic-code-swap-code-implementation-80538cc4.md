---
title: A generic <code>swap</code> implementation
url: https://gustedt.wordpress.com/2010/10/23/a-generic-swap-implementation/
published: "2010-10-23T11:13:51Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=556
---

# A generic <code>swap</code> implementation

Swapping the contents of two variable is an elementary task that is often met in daily programming. There are two generic strategies to do that for general types.

The first, most commonly used, creates a temporary variable and assigns the values cyclically:

```
T a = ...;
T b = ...;
.
T tmp = a; a = b; a = tmp;

```

We will call this strategy `P99_SWAP1`. Here the compiler must realize the three assignments strictly in order, otherwise the resulting program would not be correct.

The second one, call it `P99_SWAP2`, tries to do something similar, but to relax the ordering constraints a bit:

```
T a = ...;
T b = ...;
.
T tmpa = a; T tmpb = b;
a = tmpb; b = tmpa;

```

For the tradeof of using more resources (stack space or registers) this can result in more efficient code. The two objects can be loaded and stored in parallel. But the gain will probably only be visible for small objects. So a possible attempt to combine the two could be

```
#define P99_SWAP(A, B) (sizeof(A) > sizeof(uintmax_t) ? P99_SWAP1(A, B) : P99_SWAP2(A, B))

```

But then, how to implement the two “submacros” `P99_SWAP1` and `P99_SWAP2(A, B)`? A particular difficulty if we want to do this generically as a C macro or function is that we don’t have access to the type of the expressions that are passed as arguments `A` and `B`. So let us first write “crossfingers” function and macro that just forget about the type issues:

```
inline
void p00_swap2(void* a, void* b, void* tmpa, void* tmpb, size_t len) {
   memcpy(tmpa, a, len);
   memcpy(tmpb, b, len);
   memcpy(b, tmpa, len);
   memcpy(a, tmpb, len);
}
#define P00X_SWAP2(A, B) p00_swap2(&(A), &(B), (char[sizeof(A)]){ [0] = 0 }, (char[sizeof(A)]){ [0] = 0 }, sizeof(A))

```

Here the weirdo expressions `(char[sizeof(A)]){ [0] = 0 }` are so-called compound literals that provide temporary objects for the copy operations.

This has several drawbacks. First we don’t even check if `A` and `B` correspond to objects of the same size, but we happily copy into them. So first we have to assert that they at least agree on their sizes such that we don’t provoke undefined behavior. This can be achieved by some expression magic for the two compound literals:

```
(char[sizeof(A)]){ [(intmax_t)sizeof(A) - sizeof(B)] = 0 }

```

What is happening here? The right part inside the `[]`, a designated initializer, is used to initialize one element of the `char[]`. Now here we take a the difference of the two sizes: this will be the element at position `0` if both sizes are the same. If `sizeof(A)` is smaller than `sizeof(B)` because of the cast to `intmax_t` this will produce a negative number at compile time.

If now we apply the symmetric strategy to the second compound literal we get a macro that compiles well when it is called with two objects of the same size, and that produces a compile time error when the sizes disagree:

```
#define P00_SWAP2(A, B)
p00_swap2(                                                      \
     &(A),                                                       \
     &(B),                                                       \
     (char[sizeof(A)]){ [(intmax_t)sizeof(A) - sizeof(B)] = 0 }, \
     (char[sizeof(B)]){ [(intmax_t)sizeof(B) - sizeof(A)] = 0 }, \
     sizeof(A))

```

This now already is much safer, but perhaps not safe enough, since the two objects might have the same size but still not be of the same type. What we can do is an extra check if the two types are assignment compatible. This can be done by something like the following which might look a bit hacky a first glance

```
(1 ? &(A) : ((A = B), NULL))

```

This is a conditional that is always true, so it always evaluates to \`&(A)\`. The second “false” part is never evaluated at run time but only checked if it is correct C code. And it wouldn’t be if `A` and `B` would not be assignment compatible.
