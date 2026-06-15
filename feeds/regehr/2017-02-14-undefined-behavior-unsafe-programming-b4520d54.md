---
title: Undefined Behavior != Unsafe Programming
url: https://blog.regehr.org/archives/1467
published: "2017-02-14T23:07:46Z"
feed: regehr
guid: http://blog.regehr.org/?p=1467
---

# Undefined Behavior != Unsafe Programming

Undefined behavior (UB) in C and C++ is a clear and present danger to developers, especially when they are writing code that will execute near a trust boundary. A less well-known kind of undefined behavior exists in the intermediate representation (IR) for most optimizing, ahead-of-time compilers. For example, LLVM IR has [undef](http://llvm.org/docs/LangRef.html#undefined-values) and [poison](http://llvm.org/docs/LangRef.html#poison-values) in addition to true explodes-in-your-face C-style UB. When people become aware of this, a typical reaction is: “Ugh, why? LLVM IR is just as bad as C!” This piece explains why that is not the correct reaction.

Undefined behavior is the result of a design decision: the refusal to systematically trap program errors at one particular level of a system. The responsibility for avoiding these errors is delegated to a higher level of abstraction. For example, it is obvious that a safe programming language can be compiled to machine code, and it is also obvious that the unsafety of machine code in no way compromises the high-level guarantees made by the language implementation. Swift and Rust are compiled to LLVM IR; some of their safety guarantees are enforced by dynamic checks in the emitted code, other guarantees are made through type checking and have no representation at the LLVM level. Either way, UB at the LLVM level is not a problem for, and cannot be detected by, code in the safe subsets of Swift and Rust. Even C can be used safely if some tool in the development environment ensures that it will not execute UB. The [L4.verified](https://ts.data61.csiro.au/projects/TS/l4.verified/) project does exactly this.

**The essence of undefined behavior is the freedom to avoid a forced coupling between error checks and unsafe operations.** The checks, once decoupled, can be optimized, for example by being hoisted out of loops or eliminated outright. The remaining unsafe operations can be — in a well-designed IR — mapped onto basic processor operations with little or no overhead. As a concrete example, consider this Swift code:

```
func add(a : Int, b : Int)->Int {
  return (a & 0xffff) + (b & 0xffff);
}

```

Although a Swift implementation must trap on integer overflow, the compiler observes that overflow is impossible and emits this LLVM IR:

```
define i64 @add(i64 %a, i64 %b) {
  %0 = and i64 %a, 65535
  %1 = and i64 %b, 65535
  %2 = add nuw nsw i64 %0, %1
  ret i64 %2
}

```

Not only has the checked addition operation been lowered to an unchecked one, but also the `add` instruction has been marked with LLVM’s `nsw` and `nuw` attributes, indicating that both signed and unsigned overflow are undefined. In isolation these attributes provide no benefit, but they may enable additional optimizations after this function is inlined. When the [Swift benchmark suite](https://swift.org/blog/swift-benchmark-suite/) is compiled to LLVM, about one in eight addition instructions has an attribute indicating that integer overflow is undefined.

In this particular example the `nsw` and `nuw` attributes are redundant since an optimization pass could re-derive the fact that the `add` cannot overflow. However, in general these attributes and others like them add real value by avoiding the need for potentially expensive static analyses to rediscover known program facts. Also, some facts cannot be rediscovered later, even in principle, since information is lost at some compilation steps.

In summary, undefined behavior in programmer-visible abstractions represents an aggressive and dangerous tradeoff: it sacrifices program correctness in favor of performance and compiler simplicity. On the other hand, UB at lower levels of the system, such as machine code or a compiler IR, is an internal design choice that needn’t have any effect on the programmer-facing parts of the system. This kind of UB simply requires us to accept that safety checks can be usefully factored out of their corresponding unsafe operations to afford efficient execution.
