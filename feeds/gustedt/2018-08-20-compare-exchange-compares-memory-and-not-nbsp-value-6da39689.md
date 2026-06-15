---
title: compare exchange compares memory and not&nbsp;value
url: https://gustedt.wordpress.com/2018/08/20/compare-exchange-compares-memory-and-not-value/
published: "2018-08-20T20:44:42Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3674
---

# compare exchange compares memory and not&nbsp;value

In C11, 7.17.7.4, introduces `atomic_compare_exchange` generic functions. These are precious tools when using atomics: they allow to conditionally store new data in an atomic variable and to retrieve a previous value of it, eventually. You can see that as a generalization of `atomic_fetch_and_add` where we are also able to retrieve a counter value and change it at the same time.

C11 stated that the *value* would be taken into account for the conditional part, that is that the existing value would be compared to a desired value. This works well for arithmetic types, where value and object representation are mostly the same. It works less well if the atomic type is a structure because `struct` types simply have no equality operator.

So to implement these generic functions, implementations basically had to use functionality that is equivalent to `memcmp`, byte to byte comparison of the underlying object representation.

But structures may have padding between the members, so comparison that would be done by using `memcmp` could, at least in theory, never be consistent with copying by value, because padding bytes could end up being different.

So it was decided that the only reasonable choice was to use byte-to-byte comparison and copying, and to change the first sentence of paragraph 3 of 7.17.7.4 such that it now reads:

> Atomically, compares the *contents of the memory* pointed to by `object` for equality with that *pointed to by* `expected`, and if true, replaces the *contents of the memory pointed to by* `object` with `desired`, and if false, updates the *contents of the memory pointed to by `expected` with that* pointed to by `object`.
