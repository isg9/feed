---
title: Dynamic types in LLDB
url: https://matklad.github.io/2017/10/21/lldb-dynamic-type.html
published: "2017-10-21T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2017/10/21/lldb-dynamic-type.html
---

# Dynamic types in LLDB

Oct 21, 2017

If you are wondering how debuggers work, I suggest reading Eli Bendersky’s [eli-on-debuggers](https://eli.thegreenplace.net/tag/debuggers). However after having read these notes myself, I still had one question unanswered. Namely, how can debugger show fields of a class, if the type of the class is known only at runtime?

## [Example](\#Example)

Consider this situation: you have a pointer of type `A*`, which at runtime holds a value of some subtype of `A`. Could the debugger display the fields of the actual type? Turns out, it can handle cases like the one below just fine!

```
struct Base { ... };

struct Derived: Base { ... };

void foo(Base& x) {
    // `x` can be `Derived` or `Base` here.
    // How can debugger show fields of `Derived` then?
}
```

## [DWARF](\#DWARF)

Could it be possible that information about dynamic types is present in DWARF? If we look at the DWARF, we’ll see that there’s layout information for both `Base` and `Derive` types, as well as a entry for `x` parameter, which says that it has type `Base`. And this makes sense: we don’t know that `x` is `Derived` until runtime! So debugger must somehow figure the type of the variable dynamically.

## [No Magic](\#No-Magic)

As usual, there’s no magic. For example, LLDB has a hard-coded knowledge of C++ programming language, which allows debugger to inspect types at runtime. Specifically, this is handled by `LanguageRuntime` LLDB **plugin**, which has a curious function [`GetDynamicTypeAndAddress`](https://github.com/llvm-mirror/lldb/blob/bc19e289f759c26e4840aab450443d4a85071139/include/lldb/Target/LanguageRuntime.h#L82), whose job is to poke the representation of value to get its real type and adjust pointer, if necessary (remember, with multiple inheritance, casts may change the value of the pointer).

The implementation of this function for C++ language lives in [ItaniumABILanguageRuntime.cpp](https://github.com/llvm-mirror/lldb/blob/bc19e289f759c26e4840aab450443d4a85071139/source/Plugins/LanguageRuntime/CPlusPlus/ItaniumABI/ItaniumABILanguageRuntime.cpp#L185). Although, unlike C, C++ lacks a standardized ABI, almost all compilers on all non-Windows platforms use a [specific ABI](http://refspecs.linuxbase.org/cxxabi-1.83.html), confusingly called Itanium (after a now effectively dead 64-bit CPU architecture).
