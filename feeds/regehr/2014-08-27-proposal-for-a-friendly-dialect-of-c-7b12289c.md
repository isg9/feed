---
title: Proposal for a Friendly Dialect of C
url: https://blog.regehr.org/archives/1180
published: "2014-08-27T17:01:08Z"
feed: regehr
guid: http://blog.regehr.org/?p=1180
---

# Proposal for a Friendly Dialect of C

*\[This post is jointly authored by Pascal Cuoq, Matthew Flatt, and John Regehr.\]*

In this post, we will assume that you are comfortable with the material in all three parts of [John’s undefined behavior writeup](https://blog.regehr.org/archives/213) and also with all three parts of [Chris Lattner’s writeup about undefined behavior](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know.html). Additionally, [this paper](http://pdos.csail.mit.edu/papers/stack:sosp13.pdf) is excellent background reading.

C compilers generate faster and smaller code by assuming that the compiled program will never execute an undefined behavior (UB). Each time a compiler exploits a new kind of UB or increases the optimizer’s reach in other ways, some code that previously worked will break. Thus, compiler upgrades cause consistent low-grade headaches for developers and maintainers of large bodies of C code. It’s not hard to find [reasonable objections](https://gcc.gnu.org/ml/gcc-help/2011-07/msg00219.html) to this kind of thing as well as [irate bug reports](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=30475#c36). The code that gets broken by the UB-aware compiler is incorrect according to the standard, but following all of the rules in the C standard in a large code base is brutally difficult and, practically speaking, few programmers are capable of it. For example, a sufficiently advanced compiler can break six of the nine well-worn C programs in SPEC CINT 2006 by only exploiting integer undefined behaviors. The problem is that the ostensible user base for C — people implementing low-level systems code — is not necessarily well served by creeping undefined-behavior exploitation. In short, modern C is not a friendly programming language.

When developers are not 100% certain that their code is free of undefined behaviors, one thing they do is add compiler-specific flags that disable certain UB-based optimizations. For example, PostgreSQL uses the `-fwrapv` option, which tells GCC to implement two’s complement wrapping behavior for signed integer overflows. For analogous reasons, the Linux kernel uses `-fno-strict-aliasing` and `-fno-delete-null-pointer-checks`. The problem with these sorts of flags is that they are compiler-specific, the flags don’t necessarily mean the same thing across compiler versions, the flags individually don’t provide much protection against UB exploitation, and developers must watch out for new kinds of breakage and new flags to add to configuration scripts.

As Chris Lattner says at the end of his [third post](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know_21.html) on this topic, using various `-f` flags amounts to selecting a different dialect of C. Instead of having programmers learn, mix, match, and track various `-f` flags, we propose defining a *friendly* dialect of C that trades some optimization capability for ease of reasoning. This friendly dialect might be supported through a `-std=friendly-c` flag (if you’ll indulge the idea that the friendly dialect could be a standard) that merely implies a group of `-f` flags for a given version of GCC or LLVM. The flag would be otherwise orthogonal to code generation options, such as `-O2`. Our goal is to combine

- minimal additional effort for compiler developers by — as much as possible — simply requiring that they provide behaviors that are already present or are at least easily available; with

- minimal slowdown when compared to maximum UB-aware compiler optimizations by (1) not requiring any [UBSan-like dynamic checks](http://clang.llvm.org/docs/UsersManual.html#controlling-code-generation) to be added and (2) disabling only optimizations that provide a bad value proposition in terms of performance vs. friendliness.

As a starting point, we imagine that friendly C is like the current C standard, but replacing many occurrences of “X has undefined behavior” with “X results in an unspecified value”. That adjustment alone can produce a much friendlier language. In other cases, we may be forced to refer to machine-specific details that are not features of the C abstract machine, and we are OK with that.

Here are some features we propose for friendly C:

1. The value of a pointer to an object whose lifetime has ended remains the same as it was when the object was alive.

2. Signed integer overflow results in two’s complement wrapping behavior at the bitwidth of the promoted type.

3. Shift by negative or shift-past-bitwidth produces an unspecified result.

4. Reading from an invalid pointer either traps or produces an unspecified value. In particular, all but the most arcane hardware platforms can produce a trap when dereferencing a null pointer, and the compiler should preserve this behavior.

5. Division-related overflows either produce an unspecified result or else a machine-specific trap occurs.

6. If possible, we want math- and memory-related traps to be treated as externally visible side-effects that must not be reordered with respect to other externally visible side-effects (much less be assumed to be impossible), but we recognize this may result in significant runtime overhead in some cases.

7. The result of any signed left-shift is the same as if the left-hand shift argument was cast to unsigned, the shift performed, and the result cast back to the signed type.

8. A read from uninitialized storage returns an unspecified value.

9. It is permissible to compute out-of-bounds pointer values including performing pointer arithmetic on the null pointer. This works as if the pointers had been cast to `uintptr_t`. However, the translation from pointer math to integer math is not completely straightforward since incrementing a pointer by one is equivalent to incrementing the integer-typed variable by the size of the pointed-to type.

10. The strict aliasing rules simply do not exist: the representations of integers, floating-point values and pointers can be accessed with different types.

11. A data race results in unspecified behavior. Informally, we expect that the result of a data race is the same as in C99: threads are compiled independently and then data races have a result that is dictated by the details of the underlying scheduler and memory system. Sequentially consistent behavior may not be assumed when data races occur.

12. memcpy() is implemented by memmove(). Additionally, both functions are no-ops when asked to copy zero bytes, regardless of the validity of their pointer arguments.

13. The compiler is granted no additional optimization power when it is able to infer that a pointer is invalid. In other words, the compiler is obligated to assume that any pointer might be valid at any time, and to generate code accordingly. The compiler retains the ability to optimize away pointer dereferences that it can prove are redundant or otherwise useless.

14. When a non-void function returns without returning a value, an unspecified result is returned to the caller.

In the interest of creating a readable blog post, we have kept this discussion informal and have not made any attempt to be comprehensive. If this proposal gains traction, we will work towards an implementable specification that addresses all 203 items listed in Annex J of the [C11 standard](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf). A friendly C++ could also be defined but that is a bigger job.

We are not trying to fix the deficiencies of the C language nor making an argument for or against C. Rather, we are trying rescue the predictable little language that we all know is hiding within the C standard. This language generates tight code and doesn’t make you feel like the compiler is your enemy. We want to decrease the rate of bit rot in existing C code and also to reduce the auditing overhead for safety-critical and security-critical C code. The intended audience for `-std=friendly-c` is people writing low-level systems such as operating systems, embedded systems, and programming language runtimes. These people typically have a good guess about what instructions the compiler will emit for each line of C code they write, and they simply do not want the compiler silently throwing out code. If they need code to be faster, they’ll change how it is written.

Related reading:

- [Changing some undefined behavior into ill-formed](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2605.pdf)
- [Distinguishing criticality of undefined behavior](http://grouper.ieee.org/groups/plv/DocLog/100-199/100-thru-129/22-OWGV-N-0104/n0104.html)
- LLVM makes a distinction between undefined behavior and [undefined values](http://llvm.org/docs/LangRef.html#undefined-values)

We appreciate feedback.
