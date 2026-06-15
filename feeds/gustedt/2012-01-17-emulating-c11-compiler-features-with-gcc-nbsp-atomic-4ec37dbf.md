---
title: Emulating C11 compiler features with gcc:&nbsp;_Atomic
url: https://gustedt.wordpress.com/2012/01/17/emulating-c11-compiler-features-with-gcc-_atomic/
published: "2012-01-17T14:35:01Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1258
---

# Emulating C11 compiler features with gcc:&nbsp;_Atomic

The new support for atomic operations in C11 is probably the most useful one. Support for atomic instructions is present in all commodity processors since at least 20 years, but a standardized interface in one of the main programming languages was really missing. Up to now you always had to implement stubs for these operations in assembler. This by itself was not as difficult for a given platform, but writing platform independent code quickly became tedious.

P99 now has an emulation of parts of these features that allow you to use them in a preview for C11. This implementation mainly uses (again) extensions of the gcc family. It should even work for older versions of gcc that don’t implement their `__sync_..` built-in.

In C11, `_Atomic` comes in two flavors, as a type *qualifier* (like `const` or `volatile`) or as type *specifier* (like in `struct` or array declarations). Because with its function-like syntax it is much simpler, P99 only implements the latter.

```
_Atomic(int) hui = ATOMIC_VAR_INIT(42);
atomic_fetch_add(&hui, 1);
fprintf(stderr, "the value is %d\n", atomic_load(&hui));

P99_DECLARE_STRUCT(test);
struct test { /* put something here */ };
P99_DECLARE_ATOMIC(test);

_Atomic(test) myTester = ATOMIC_VAR_INIT( { /* an init expression for test */ } );

```

I hope you can guess a pattern.

- the implementation is limited to types that can be named through a single identifier
- types that are not predefined such as structure, union or pointer types must have a declaration by `P99_DECLARE_ATOMIC` before the first use of `_Atomic`.
- For the predefined integer types ( `unsigned`, `int`, `long`, `_Bool`, …) the resulting atomic type is independent of the particular alias that you use for that type. E.g if on a platform `size_t` and `ulong` are aliases for `unsigned long`, `_Atomic(size_t)` and `_Atomic(ulong)` will refer to the same atomic type, namely `atomic_ulong`.
- For other types ( `struct`, `union` or pointers) the resulting type is bound to the particular alias ( `typedef`) for which you call the macro. In the example above the name would be `atomic_test`.
- The atomic operations are only provided through function like interfaces such as `atomic_load`, `atomic_store`, `atomic_fetch_add` etc. Using operators `=`, `+=` or evaluation in expressions doesn’t work.

For the moment all of this is modestly tested in a local project of mine, but which uses these atomic features a lot. So hopefully the main initial bugs are tracked down.

In addition to the `_Atomic` types themselves, another important atomic type that is specified by the standard, namely `atomic_flag`, is fully implemented in P99. This type is thought as *the* basic primitive for the implementation of atomic operations. But actually it can serve other interesting purposes. I’ll probably write an entire blog entry on that, soon.

For the moment the implementation heavily uses gcc-promoted extensions:

- everything that we already have seen for `_Generic`
- `__typeof__` even if the compiler would provide direct support for `_Generic` (such as clang)
- statement expressions via the `({ ... })` construct
- `__sync_...` built-in for the atomic primitives, or, if not present `__asm__` for stub implementations (x86 and arm, only).

There would probably be a possibility to do something even more restricted without using `_Generic` or similar by just using the standard type names such as `atomic_int`, `atomic_ulong` etc., but for the moment the code is not factorized enough.

For the declaration of atomic pointer types you’d have to use `P99_DECLARE_ATOMIC` the same way as you have seen above for a structure type: first `typedef` the pointer type so you may access it through a single identifier and then use that identifier to declare the atomic type. Then the underlying `_Generic` mechanism will choose the appropriate atomic operations directly on the pointer type, if they are available. That is, e.g `atomic_add` on an atomic pointer type would use an atomic assembler instruction if such a beast is available. E.g

```
typedef size_t* stackp;
P99_DECLARE_ATOMIC(stackp);

for (_Atomic(stackp) p = ATOMIC_VAR_INIT(arr);
     atomic_load(p) < arr + 46;
     ) {
  ... do something ...
  if (some condition) atomic_add(p, 1);
}

```
