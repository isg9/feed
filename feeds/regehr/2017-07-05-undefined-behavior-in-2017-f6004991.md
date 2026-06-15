---
title: Undefined Behavior in 2017
url: https://blog.regehr.org/archives/1520
published: "2017-07-05T05:58:27Z"
feed: regehr
guid: http://blog.regehr.org/?p=1520
---

# Undefined Behavior in 2017

*This post is jointly authored by Pascal Cuoq and John Regehr.*

Recently we’ve heard a few people imply that problems stemming from undefined behaviors (UB) in C and C++ are largely solved due to ubiquitous availability of dynamic checking tools such as ASan, UBSan, MSan, and TSan. We are here to state the obvious — that, despite the many excellent advances in tooling over the last few years, UB-related problems are far from solved — and to look at the current situation in detail.

Valgrind and most of the sanitizers are intended for debugging: emitting friendly diagnostics regarding undefined behaviors that are executed during testing. Tools like this are exceptionally useful and they have helped us progress from a world where almost every nontrivial C and C++ program executed a continuous stream of UB to a world where quite a few important programs seem to be largely UB-free in their most common configurations and use cases.

The problem with dynamic debugging tools is that they don’t do anything to help us to cope with the worst UBs: the ones that we didn’t know how to trigger during testing, but that someone else has figured out how to trigger in deployed software — while exploiting it. The problem reduces to doing good testing, which is hard. Tools like afl-fuzz are great but they barely begin to scratch the surface of large programs that process highly structured inputs.

One way to sidestep problems in testing is to use static UB-detection tools. These are steadily improving, but sound and precise static analysis is not necessarily any easier than achieving good test coverage. Of course the two techniques are attacking the same problem — identifying feasible paths in software — from opposite sides. This problem has always been extremely hard and probably always will be. We’ve written a lot elsewhere about finding UBs via static analysis; in this piece our focus is on dynamic tools.

The other way to work around problems in testing is to use UB mitigation tools: these turn UB into defined behavior in production C and C++, effectively gaining some of the benefits of a safe programming language. The challenge is in engineering mitigation tools that:

- don’t break our code in any corner cases,

- have very low overhead,

- don’t add effective attack surfaces, for example by requiring programs to be linked against a non-hardened runtime library,

- raise the bar for determined attackers (in contrast, debugging tools can afford to use heuristics that aren’t resistant to adversaries),

- compose with each other (in contrast, some debugging tools such as ASan and TSan are not compatible, necessitating two runs of the test suite for any project that wants to use both).

Before looking at some individual kinds of UB, let’s review the our goals here. These apply to every C and C++ compiler.

**Goal 1: Every UB (yes, all ~200 of them, we’ll give the list towards the end of this post) must either be documented as having some defined behavior, be diagnosed with a fatal compiler error, or else — as a last resort — have a sanitizer that detects that UB at runtime.** This should not be controversial, it’s sort of a minimal requirement for developing C and C++ in the modern world where network packets and compiler optimizations are effectively hostile.

**Goal 2: Every UB must either be documented as having some defined behavior, be diagnosed with a fatal compiler error, or else have an optional mitigation mechanism that meets the requirements above.** This is more difficult; it necessitates, for example, production-grade memory safety. We like to think that this can be achieved in many execution environments. OS kernels and other maximally performance-critical code will need to resort to more difficult technologies such as formal methods.

The rest of this piece will look at the current situation for various classes of undefined behaviors. We’ll start with the big ones.

## Spatial Memory Safety Violations

**Background:** Accessing out-of-bounds storage and even creating pointers to that storage are UB in C and C++. The [1988 Morris Worm](../extra_files/IWorm-paper.pdf) gave us an early hint of what the next N years would be like. So far we know that N >= 29, and probably N will end up being about 75.

**Debugging:** Valgrind and ASan are both excellent debugging tools. For many use cases ASan is the better choice because it has much less overhead. Both tools retain the representation of addresses as 32- or 64-bit values, and reserve forbidden red zones around valid blocks. This is a robust and compatible approach: it interoperates seamlessly with non-instrumented binary libraries and also supports existing code that relies on pointers being convertible to integers.

Valgrind, working from executable code, cannot insert red zones between stack variables because stack layout is implicitly hard-coded in the offsets of instructions that access the stack, and it would be an impossibly ambitious project to remap stack addresses on the fly. As a result, Valgrind has only limited support for detecting errors in manipulating storage on the stack. ASan works during compilation and inserts red zones around stack variables. Stack variables are small and numerous, so address space and locality considerations prevent the use of very large red zones. With default settings, the addresses of two adjacent local int variables x and y end up separated by 16 bytes. In other words, the verifications done by ASan and Valgrind are only for one memory layout, and the memory layout for which the verifications are done is different from the memory layout of the uninstrumented execution.

A minor weakness of ASan and Valgrind is that they can miss undefined behaviors that get optimized away before the instrumentation has a chance to run, [as in this example](https://godbolt.org/g/1c6xK6).

**Mitigation:** We’ve long had partial mitigation mechanisms for memory unsafety, including ASLR, stack canaries, hardened allocators, and NX. More recently, [production-grade CFI](https://clang.llvm.org/docs/ControlFlowIntegrity.html) (control flow integrity) has become available. Another interesting recent development is [pointer authentication in ARMv8.3](https://www.qualcomm.com/documents/whitepaper-pointer-authentication-armv83). [This paper](https://people.eecs.berkeley.edu/~dawnsong/papers/Oakland13-SoK-CR.pdf) has a good overview of memory safety mitigations.

A serious drawback of ASan as a mitigation tool is illustrated here:

```
$ cat asan-defeat.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

char a[128];
char b[128];

int main(int argc, char *argv[]) {
  strcpy(a + atoi(argv[1]), "owned.");
  printf("%s\n", b);
  return 0;
}
$ clang-4.0 -O asan-defeat.c
$ ./a.out 128
owned.
$ clang-4.0 -O -fsanitize=address -fno-common asan-defeat.c
$ ./a.out 160
owned.
$

```

In other words, ASan simply forces an attacker to compute a different offset in order to corrupt a target memory region. (Thanks to Yury Gribov for pointing out that we should be using the -fno-common flag to ASan.)

To mitigate this kind of undefined behavior, real bounds checking must be performed, as opposed to only verifying that each memory access lands in some valid region. Memory safety is the gold standard here. Although there is much academic work on memory safety, some showing apparently reasonable overheads and good compatibility with existing software, it has not yet seen widespread adoption. [Checked C](https://www.microsoft.com/en-us/research/project/checked-c/) is a very cool project to keep an eye on in this space.

**Summary:** Debugging tools for this class of error are very good. Good mitigations are available but this class of bug can only be reliably stopped by full memory/type safety.

## Temporal Memory Safety Violations

**Background:** A “temporal memory safety violation” is any use of a memory location after its lifetime has ended. This includes addresses of automatic variables outliving these variables; use-after-free, where a dangling pointer is accessed for reading or writing; and, double free, which can be just as harmful in practice, since free() modifies metadata that is usually adjacent to the block being freed. If the block has already been freed, these writes can fall on memory used for any other purpose and, in principle, can have as much consequence as any other invalid write.

**Debugging:** ASan is designed to detect use-after-free bugs, which often lead to hard-to-reproduce, erratic behavior. It does so by placing freed memory blocks in a quarantine, preventing their immediate reuse. For some programs and inputs, this can increase memory consumption and decrease locality. The user can configure the size of the quarantine in order to trade false positives for resource usage.

ASan can also detect addresses of automatic variables surviving the scope of these variables. The idea is to turn automatic variables into heap-allocated blocks, that the compiler automatically allocates when execution enters the block, and frees (while retaining them in a quarantine) when execution leaves the block. This option is turned off by default, because it makes programs even more memory-hungry.

The temporal memory safety violation in the program below causes it to behave differently at the default optimization level and at -O2. ASan can detect a problem in the program below with no optimization, but only if the option detect\_stack\_use\_after\_return is set, and only if the program was not compiled with optimization.

```
$ cat temporal.c
#include <stdio.h>

int *G;

int f(void) {
  int l = 1;
  int res = *G;
  G = &l;
  return res;
}

int main(void) {
  int x = 2;
  G = &x;
  f();
  printf("%d\n", f());
}
$ clang -Wall -fsanitize=address temporal.c
$ ./a.out
1
$ ASAN_OPTIONS=detect_stack_use_after_return=1 ./a.out
=================================================================
==5425==ERROR: AddressSanitizer: stack-use-after-return ...
READ of size 4 at 0x0001035b6060 thread T0
^C
$ clang -Wall -fsanitize=address -O2 temporal.c
$ ./a.out
32767
$ ASAN_OPTIONS=detect_stack_use_after_return=1 ./a.out
32767
$ clang -v
Apple LLVM version 8.0.0 (clang-800.0.42.1)
...

```

In some other examples, the sanitizer’s failure to detect UB that has been “optimized out” can be argued to be harmless, since the optimized-out UB has no consequence. This is not the case here! The program is meaningless in any case, but the unoptimized program behaves deterministically and works as if the variable x had been declared static, whereas the optimized program, in which ASan does not detect any foul play, does not behave deterministically and reveals an internal state that is not supposed to be seen:

```
$ clang -Wall -O2 temporal.c
$ ./a.out
1620344886
$ ./a.out
1734516790
$ ./a.out
1777709110

```

**Mitigation:** As discussed above, ASan is not intended for hardening, but various hardened allocators are available; they use the same quarantining strategy to render use-after-free bugs unexploitable.

**Summary:** Use ASan (together with “ASAN\_OPTIONS=detect\_stack\_use\_after\_return=1” for the test cases that are small enough to allow it). Vary optimization levels in case some compilations catch errors that others don’t.

## Integer Overflow

**Background:** Integers cannot underflow, but they can overflow in both directions. Signed integer overflow is UB; this includes INT\_MIN / -1, INT\_MIN % -1, negating INT\_MIN, shift with negative exponent, left-shifting a one past the sign bit, and (sometimes) left-shifting a one into the sign bit. Division by zero and shift by >= bitwidth are UB in both the signed and unsigned flavors. [Read more here](http://www.cs.utah.edu/~regehr/papers/tosem15.pdf).

**Debugging:** LLVM’s UBSan is very good for debugging integer-related undefined behaviors. Because UBSan works near the source level, it is highly reliable. There are some quirks relating to compile-time math; for example, [this program traps as C++11 but not as C11](https://godbolt.org/g/BFHKg9); we believe this follows the standards but haven’t looked into it closely. GCC has its own version of UBSan but it [isn’t 100% trustworthy](https://godbolt.org/g/2spSWN); here it looks like constants are being folded before the instrumentation pass gets to run.

**Mitigation:** UBSan in trapping mode (on hitting UB, process aborts w/o printing a diagnostic) can be used for mitigation. It is usually reasonably efficient and it doesn’t add attack surface. [Parts of Android use UBSan to mitigate integer overflows](https://android-developers.googleblog.com/2016/05/hardening-media-stack.html) (including unsigned overflows, which of course are not undefined). Although integer overflows are generic logic errors, in C and C++ they are particularly harmful because they often lead to memory safety violations. In a memory-safe language they tend to do much less damage.

**Summary:** Integer undefined behaviors are not very difficult to catch; UBSan is the only debugging tool you’re likely to ever need. An issue with mitigating integer UBs is the overhead. For example, they cause SPEC CPU 2006 to run about 30% slower. There is plenty of room for improvement, both in eliminating overflow checks that cannot fire and in making the remaining checks less obstructive to the loop optimizers. Someone with resources should push on this.

## Strict Aliasing Violations

**Background:** The “strict aliasing rules” in the C and C++ standards allow the compiler to assume that if two pointers refer to different types, they cannot point to the same storage. This enables nice optimizations but risks breaking programs that take a flexible view of types (roughly 100% of large C and C++ programs take a flexible view of types somewhere). For a thorough overview see [Sections 1-3 of this paper](http://trust-in-soft.com/wp-content/uploads/2017/01/vmcai.pdf).

**Debugging:** The state of the art in debugging tools for strict aliasing violations is weak. Compilers warn about some easy cases, but these warnings are extremely fragile. [libcrunch](https://github.com/stephenrkell/libcrunch) warns that a pointer is being converted to a type “pointer to thing” when the pointed object is not, in fact, a “thing.” This allows polymorphism though void pointers, but catches misuses of pointer conversions that are also strict aliasing violations. With respect to the C standard and C compilers’ interpretation of what it allows them to optimize in their type-based alias analyses, however, libcrunch is neither sound (it does not detect some violations that happen during the instrumented execution) nor complete (it warns about pointer conversions that smell bad but do not violate the standard).

**Mitigation:** This is easy: pass the compiler a flag (-fno-strict-aliasing) that disables optimizations based on strict aliasing. The result is a C/C++ compiler that has an old-school memory model where more or less arbitrary casts between pointer types can be performed, with the resulting code behaving as expected. Of the big three compilers, it is only LLVM and GCC that are affected, MSVC doesn’t implement this class of optimization in the first place.

**Summary:** Correctness-sensitive code bases need significant auditing: it is always suspicious and dangerous to cast a pointer to any type other than a char \*. Alternatively, just turn off strict-aliasing-based optimizations using a flag and make sure that nobody ever builds the code without using this flag.

## Alignment Violations

**Background:** RISC-style processors have tended to disallow memory accesses where the address is not a multiple of the size of the object being accessed. On the other hand, C and C++ programs that use unaligned pointers are undefined regardless of the target architecture. Historically we have been complacent about this, first because x86/x64 support unaligned accesses and second because compilers have so far not done much to exploit this UB.

Even so, here is an [excellent blog post](https://pzemtsov.github.io/2016/11/06/bug-story-alignment-on-x86.html) explaining how the compiler can break code that does unaligned accesses when targeting x64. The code in the post violates strict aliasing in addition to violating the alignment rules, but the crash (we verified it under GCC 7.1.0 on OS X) occurs even when the `-fno-strict-aliasing` flag is passed to the compiler.

**Debugging:** UBSan can detect misaligned memory accesses.

**Mitigation:** None known.

**Summary:** Use UBSan.

## Loops that Neither Perform I/O nor Terminate

**Background:** A loop in C or C++ code that neither performs I/O nor terminates is undefined and can be terminated arbitrarily by the compiler. See [this post](https://blog.regehr.org/archives/140) and [this note](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1528.htm).

**Debugging:** No tools exist.

**Mitigation:** None, besides avoiding heavily-optimizing compilers.

**Summary:** This UB is probably not a problem in practice (even if it is moderately displeasing to some of us).

## Data Races

**Background:** A data race occurs when a piece of memory is accessed by more than one thread, at least one of the accesses is a store, and the accesses are not synchronized using a mechanism such as a lock. Data races are UB in modern flavors of C and C++ (they do not have a semantics in older versions since those standards do not address multithreaded code).

**Debugging:** [TSan](https://clang.llvm.org/docs/ThreadSanitizer.html) is an excellent dynamic data race detector. Other similar tools exist, such as the Helgrind plugin for Valgrind, but we have not used these lately. The use of dynamic race detectors is complicated by the fact that races can be very difficult to trigger, and worse this difficulty depends on variables such as the number of cores, the thread scheduling algorithm, whatever else is going on on the test machine, and on the moon’s phase.

**Mitigation:** Don’t create threads.

**Summary:** This particular UB is probably a good idea: it clearly communicates the idea that developers should not count on racy code doing anything in particular, but should rather use atomics (that cannot race by definition) if they don’t enjoy locking.

## Unsequenced Modifications

**Background:** In C, “sequence points” constrain how early or late a side-effecting expression such as x++ can take effect. C++ has a different but more-or-less-equivalent formulation of these rules. In either language, unsequenced modifications of the same value, or an unsequenced modification and use of the same value, results in UB.

**Debugging:** Some compilers emit warnings for obvious violations of the sequencing rules:

```
$ cat unsequenced2.c
int a;

int foo(void) {
  return a++ - a++;
}
$ clang -c unsequenced2.c
unsequenced2.c:4:11: warning: multiple unsequenced modifications to 'a' [-Wunsequenced]
  return a++ - a++;
          ^     ~~
1 warning generated.
$ gcc-7 -c unsequenced2.c -Wall
unsequenced2.c: In function 'foo':
unsequenced2.c:4:11: warning: operation on 'a' may be undefined [-Wsequence-point]
   return a++ - a++;
          ~^~

```

However, a bit of indirection defeats these warnings:

```
$ cat unsequenced.c
#include <stdio.h>

int main(void) {
  int z = 0, *p = &z;
  *p += z++;
  printf("%d\n", z);
  return 0;
}
$ gcc-4.8 -Wall unsequenced.c ; ./a.out
0
$ gcc-7 -Wall unsequenced.c ; ./a.out
1
$ clang -Wall unsequenced.c ; ./a.out
1

```

**Mitigation:** None known, though it would be almost trivial to define the order in which side effects take place. The Java Language Definition provides an example of how to do this. We have a hard time believing that this kind of constraint would meaningfully handicap any modern optimizing compiler. If the standards committees can’t find it within their hearts to make this happen, the compiler implementors should do it anyway. Ideally, all major compilers would make the same choice.

**Summary:** With a bit of practice, it is not too difficult to spot the potential for unsequenced accesses during code reviews. We should be wary of any overly-complex expression that has many side effects. This leaves us without a good story for legacy code, but hey it has worked until now, so perhaps there’s no problem. But really, this should be fixed in the compilers.

A non-UB relative of unsequenced is “indeterminately sequenced” where operations may happen in an order chosen by the compiler. An example is the order of the first two function calls while evaluating f(a(), b()). This order should be specified too. Left-to-right would work. Again, there will be no performance loss in non-insane circumstances.

## TIS Interpreter

We now change gears and take a look at the approach taken by [TIS Interpreter](http://trust-in-soft.com/tis-interpreter/), a debugging tool that looks for undefined behavior in C programs as it executes them line by line. TIS Interpreter runs programs much more slowly than the LLVM-based sanitizers, and even much more slowly than Valgrind. However, TIS Interpreter can usefully be compared to these sanitizers: it works from the source code, leaves the problem of coverage to test suites and fuzzing tools, and identifies problems along the execution paths that it has been provided inputs for.

A fundamental difference between TIS Interpreter and any single sanitizer is that TIS Interpreter’s goal is, along the execution paths it explores, to be exhaustive: to find all the problems that ASan, MSan, and UBSan are designed to find some of (give or take a couple of minor exceptions that we would be delighted to discuss at great length if provoked). For example, TIS Interpreter identifies unsequenced changes to overlapping memory zones within an expression, such as (\*p)++ + (\*q)++ when the pointers p and q alias. The problem of the unspecified order of function calls in a same expression, that TIS Interpreter orders without warning when a different order could produce a different result, is a known limitation that will eventually be fixed.

TIS Interpreter’s approach to detecting memory safety errors differs sharply from ASan’s and Valgrind’s in that it doesn’t find errors for a specific heap layout, but rather treats as an error any construct that could lead the execution to behave differently depending on memory layout choices. In other words, TIS Interpreter has a symbolic view of addresses, as opposed to the concrete view taken by Valgrind and ASan. This design choice eliminates the “only the instrumented version of the program is safe, and the instrumented version behaves differently from the deployed version” problem. The occasional C program is written to behave differently depending on the memory layout (for instance if addresses are fed to hash functions or used to provide a total ordering between allocated values). TIS Analyzer warns that these programs are doing this (which is always good to know); sometimes, tweaks make it possible to analyze them in TIS Interpreter anyway, but the resulting guarantees will be weaker.

It is sometimes useful, for debugging purposes, to see the first UB that occurs in an execution. Consider a loop in which MSan warns that uninitialized memory is being used, and in which ASan warns about an out-of-bounds read. Is the out-of-bounds read caused by the incorporation of uninitialized memory in the computation of the index, or is the use of uninitialized memory caused by the index being computed wrongly? One cannot use both ASan and MSan at the same time, so this is a mystery that developers need to solve for themselves. The value of looking for all undefined behaviors at the same time is in this case the confidence that the first undefined behavior seen is not a symptom of a previous undefined behavior. Another advantage is [finding undefined behavior that one was not looking for](https://trust-in-soft.com/an-old-quirky-libksba-bug/).

Detection of strict aliasing violations in TIS Interpreter is being worked on, following as much as possible the C standard and the interpretation of C compiler designers (which can be observed in each compiler’s translation of well-chosen examples).

## But What About the Rest of the Undefined Behaviors?

Let’s take a quick look at the contents of [Appendix J.2: a non-normative, non-exhaustive list of undefined behaviors in C](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1548.pdf#page=571). Keep in mind that no equivalent list has ever been created for C++, as far as we know.

First, we’ll list the UBs that we’ve discussed explicitly in this post:

- The execution of a program contains a data race (5.1.2.4).

- An object is referred to outside of its lifetime (6.2.4).

- The value of a pointer to an object whose lifetime has ended is used (6.2.4).

- The value of an object with automatic storage duration is used while it is indeterminate (6.2.4, 6.7.9, 6.8).

- Conversion to or from an integer type produces a value outside the range that can be represented (6.3.1.4).

- An lvalue does not designate an object when evaluated (6.3.2.1).

- Conversion between two pointer types produces a result that is incorrectly aligned (6.3.2.3).

- A side effect on a scalar object is unsequenced relative to either a different side effect on the same scalar object or a value computation using the value of the same scalar object (6.5).

- An exceptional condition occurs during the evaluation of an expression (6.5).

- An object has its stored value accessed other than by an lvalue of an allowable type (6.5).

- The operand of the unary \* operator has an invalid value (6.5.3.2).

- The value of the second operand of the / or % operator is zero (6.5.5).

- Addition or subtraction of a pointer into, or just beyond, an array object and an integer type produces a result that does not point into, or just beyond, the same array object (6.5.6).

- Addition or subtraction of a pointer into, or just beyond, an array object and an integer type produces a result that points just beyond the array object and is used as the operand of a unary \* operator that is evaluated (6.5.6).

- Pointers that do not point into, or just beyond, the same array object are subtracted (6.5.6).

- An array subscript is out of range, even if an object is apparently accessible with the given subscript (as in the lvalue expression a\[1\]\[7\] given the declaration int a\[4\]\[5\]) (6.5.6).

- The result of subtracting two pointers is not representable in an object of type ptrdiff\_t (6.5.6).

- An expression is shifted by a negative number or by an amount greater than or equal to the width of the promoted expression (6.5.7).

- An expression having signed promoted type is left-shifted and either the value of the expression is negative or the result of shifting would be not be representable in the promoted type (6.5.7).

- Pointers that do not point to the same aggregate or union (nor just beyond the same array object) are compared using relational operators (6.5.8).

- An object is assigned to an inexactly overlapping object or to an exactly overlapping object with incompatible type (6.5.16.1).

And second, those that we have not addressed:

- A “shall” or “shall not” requirement that appears outside of a constraint is violated (clause 4).

- A nonempty source file does not end in a new-line character which is not immediately preceded by a backslash character or ends in a partial preprocessing token or comment (5.1.1.2).

- Token concatenation produces a character sequence matching the syntax of a universal character name (5.1.1.2).

- A program in a hosted environment does not define a function named main using one of the specified forms (5.1.2.2.1).

- A character not in the basic source character set is encountered in a source file, except in an identifier, a character constant, a string literal, a header name, a comment, or a preprocessing token that is never converted to a token (5.2.1).

- An identifier, comment, string literal, character constant, or header name contains an invalid multibyte character or does not begin and end in the initial shift state (5.2.1.2).

- The same identifier has both internal and external linkage in the same translation unit (6.2.2).

- A trap representation is read by an lvalue expression that does not have character type (6.2.6.1).

- A trap representation is produced by a side effect that modifies any part of the object using an lvalue expression that does not have character type (6.2.6.1).

- The operands to certain operators are such that they could produce a negative zero result, but the implementation does not support negative zeros (6.2.6.2).

- Two declarations of the same object or function specify types that are not compatible (6.2.7).

- A program requires the formation of a composite type from a variable length array type whose size is specified by an expression that is not evaluated (6.2.7).

- Demotion of one real floating type to another produces a value outside the range that can be represented (6.3.1.5).

- A non-array lvalue with an incomplete type is used in a context that requires the value of the designated object (6.3.2.1).

- An lvalue designating an object of automatic storage duration that could have been declared with the register storage class is used in a context that requires the value of the designated object, but the object is uninitialized. (6.3.2.1).

- An lvalue having array type is converted to a pointer to the initial element of the array, and the array object has register storage class (6.3.2.1).

- An attempt is made to use the value of a void expression, or an implicit or explicit conversion (except to void) is applied to a void expression (6.3.2.2).

- Conversion of a pointer to an integer type produces a value outside the range that can be represented (6.3.2.3).

- A pointer is used to call a function whose type is not compatible with the referenced type (6.3.2.3).

- An unmatched ‘ or ” character is encountered on a logical source line during tokenization (6.4).

- A reserved keyword token is used in translation phase 7 or 8 for some purpose other than as a keyword (6.4.1).

- A universal character name in an identifier does not designate a character whose encoding falls into one of the specified ranges (6.4.2.1).

- The initial character of an identifier is a universal character name designating a digit (6.4.2.1).

- Two identifiers differ only in nonsignificant characters (6.4.2.1).

- The identifier \_\_func\_\_ is explicitly declared (6.4.2.2).

- The program attempts to modify a string literal (6.4.5).

- The characters ‘, \\, “, //, or /\* occur in the sequence between the < and > delimiters, or the characters ‘, \\, //, or /\* occur in the sequence between the ” delimiters, in a header name preprocessing token (6.4.7).

- For a call to a function without a function prototype in scope, the number of âˆ— arguments does not equal the number of parameters (6.5.2.2).

- For call to a function without a function prototype in scope where the function is defined with a function prototype, either the prototype ends with an ellipsis or the types of the arguments after promotion are not compatible with the types of the parameters (6.5.2.2).

- For a call to a function without a function prototype in scope where the function is not defined with a function prototype, the types of the arguments after promotion are not compatible with those of the parameters after promotion (with certain exceptions) (6.5.2.2).

- A function is defined with a type that is not compatible with the type (of the expression) pointed to by the expression that denotes the called function (6.5.2.2).

- A member of an atomic structure or union is accessed (6.5.2.3).

- A pointer is converted to other than an integer or pointer type (6.5.4).

- An expression that is required to be an integer constant expression does not have an integer type; has operands that are not integer constants, enumeration constants, character constants, sizeof expressions whose results are integer constants, or immediately-cast floating constants; or contains casts (outside operands to sizeof operators) other than conversions of arithmetic types to integer types (6.6).

- A constant expression in an initializer is not, or does not evaluate to, one of the following: an arithmetic constant expression, a null pointer constant, an address constant, or an address constant for a complete object type plus or minus an integer constant expression (6.6).

- An arithmetic constant expression does not have arithmetic type; has operands that are not integer constants, floating constants, enumeration constants, character constants, or sizeof expressions; or contains casts (outside operands to size operators) other than conversions of arithmetic types to arithmetic types (6.6).

- The value of an object is accessed by an array-subscript \[\], member-access . or âˆ’>, address &, or indirection \* operator or a pointer cast in creating an address constant (6.6).

- An identifier for an object is declared with no linkage and the type of the object is incomplete after its declarator, or after its init-declarator if it has an initializer (6.7).

- A function is declared at block scope with an explicit storage-class specifier other than extern (6.7.1).

- A structure or union is defined as containing no named members, no anonymous structures, and no anonymous unions (6.7.2.1).

- An attempt is made to access, or generate a pointer to just past, a flexible array member of a structure when the referenced object provides no elements for that array (6.7.2.1).

- When the complete type is needed, an incomplete structure or union type is not completed in the same scope by another declaration of the tag that defines the content (6.7.2.3).

- An attempt is made to modify an object defined with a const-qualified type through use of an lvalue with non-const-qualified type (6.7.3).

- An attempt is made to refer to an object defined with a volatile-qualified type through use of an lvalue with non-volatile-qualified type (6.7.3).

- The specification of a function type includes any type qualifiers (6.7.3).

- Two qualified types that are required to be compatible do not have the identically qualified version of a compatible type (6.7.3).

- An object which has been modified is accessed through a restrict-qualified pointer to a const-qualified type, or through a restrict-qualified pointer and another pointer that are not both based on the same object (6.7.3.1).

- A restrict-qualified pointer is assigned a value based on another restricted pointer whose associated block neither began execution before the block associated with this pointer, nor ended before the assignment (6.7.3.1).

- A function with external linkage is declared with an inline function specifier, but is not also defined in the same translation unit (6.7.4).

- A function declared with a \_Noreturn function specifier returns to its caller (6.7.4).

- The definition of an object has an alignment specifier and another declaration of that object has a different alignment specifier (6.7.5).

- Declarations of an object in different translation units have different alignment specifiers (6.7.5).

- Two pointer types that are required to be compatible are not identically qualified, or are not pointers to compatible types (6.7.6.1).

- The size expression in an array declaration is not a constant expression and evaluates at program execution time to a nonpositive value (6.7.6.2).

- In a context requiring two array types to be compatible, they do not have compatible element types, or their size specifiers evaluate to unequal values (6.7.6.2).

- A declaration of an array parameter includes the keyword static within the \[ and \] and the corresponding argument does not provide access to the first element of an array with at least the specified number of elements (6.7.6.3).

- A storage-class specifier or type qualifier modifies the keyword void as a function parameter type list (6.7.6.3).

- In a context requiring two function types to be compatible, they do not have compatible return types, or their parameters disagree in use of the ellipsis terminator or the number and type of parameters (after default argument promotion, when there is no parameter type list or when one type is specified by a function definition with an identifier list) (6.7.6.3).

- The value of an unnamed member of a structure or union is used (6.7.9).

- The initializer for a scalar is neither a single expression nor a single expression enclosed in braces (6.7.9).

- The initializer for a structure or union object that has automatic storage duration is neither an initializer list nor a single expression that has compatible structure or union type (6.7.9).

- The initializer for an aggregate or union, other than an array initialized by a string literal, is not a brace-enclosed list of initializers for its elements or members (6.7.9).

- An identifier with external linkage is used, but in the program there does not exist exactly one external definition for the identifier, or the identifier is not used and there exist multiple external definitions for the identifier (6.9).

- A function definition includes an identifier list, but the types of the parameters are not declared in a following declaration list (6.9.1).

- An adjusted parameter type in a function definition is not a complete object type (6.9.1).

- A function that accepts a variable number of arguments is defined without a parameter type list that ends with the ellipsis notation (6.9.1).

- The } that terminates a function is reached, and the value of the function call is used by the caller (6.9.1).

- An identifier for an object with internal linkage and an incomplete type is declared with a tentative definition (6.9.2).

- The token defined is generated during the expansion of a #if or #elif preprocessing directive, or the use of the defined unary operator does not match one of the two specified forms prior to macro replacement (6.10.1).

- The #include preprocessing directive that results after expansion does not match one of the two header name forms (6.10.2).

- The character sequence in an #include preprocessing directive does not start with a letter (6.10.2).

- There are sequences of preprocessing tokens within the list of macro arguments that would otherwise act as preprocessing directives (6.10.3).

- The result of the preprocessing operator # is not a valid character string literal (6.10.3.2).

- The result of the preprocessing operator ## is not a valid preprocessing token (6.10.3.3).

- The #line preprocessing directive that results after expansion does not match one of the two well-defined forms, or its digit sequence specifies zero or a number greater than 2147483647 (6.10.4).

- A non-STDC #pragma preprocessing directive that is documented as causing translation failure or some other form of undefined behavior is encountered (6.10.6).

- A #pragma STDC preprocessing directive does not match one of the well-defined forms (6.10.6).

- The name of a predefined macro, or the identifier defined, is the subject of a #define or #undef preprocessing directive (6.10.8).

- An attempt is made to copy an object to an overlapping object by use of a library function, other than as explicitly allowed (e.g., memmove) (clause 7).

- A file with the same name as one of the standard headers, not provided as part of the implementation, is placed in any of the standard places that are searched for included source files (7.1.2).

- A header is included within an external declaration or definition (7.1.2).

- A function, object, type, or macro that is specified as being declared or defined by some standard header is used before any header that declares or defines it is included (7.1.2).

- A standard header is included while a macro is defined with the same name as a keyword (7.1.2).

- The program attempts to declare a library function itself, rather than via a standard header, but the declaration does not have external linkage (7.1.2).

- The program declares or defines a reserved identifier, other than as allowed by 7.1.4 (7.1.3).

- The program removes the definition of a macro whose name begins with an underscore and either an uppercase letter or another underscore (7.1.3).

- An argument to a library function has an invalid value or a type not expected by a function with variable number of arguments (7.1.4).

- The pointer passed to a library function array parameter does not have a value such that all address computations and object accesses are valid (7.1.4).

- The macro definition of assert is suppressed in order to access an actual function (7.2).

- The argument to the assert macro does not have a scalar type (7.2).

- The CX\_LIMITED\_RANGE, FENV\_ACCESS, or FP\_CONTRACT pragma is used in any context other than outside all external declarations or preceding all explicit declarations and statements inside a compound statement (7.3.4, 7.6.1, 7.12.2).

- The value of an argument to a character handling function is neither equal to the value of EOF nor representable as an unsigned char (7.4).

- A macro definition of errno is suppressed in order to access an actual object, or the program defines an identifier with the name errno (7.5).

- Part of the program tests floating-point status flags, sets floating-point control modes, or runs under non-default mode settings, but was translated with the state for the FENV\_ACCESS pragma “off” (7.6.1).

- The exception-mask argument for one of the functions that provide access to the floating-point status flags has a nonzero value not obtained by bitwise OR of the floating-point exception macros (7.6.2).

- The fesetexceptflag function is used to set floating-point status flags that were not specified in the call to the fegetexceptflag function that provided the value of the corresponding fexcept\_t object (7.6.2.4).

- The argument to fesetenv or feupdateenv is neither an object set by a call to fegetenv or feholdexcept, nor is it an environment macro (7.6.4.3, 7.6.4.4).

- The value of the result of an integer arithmetic or conversion function cannot be represented (7.8.2.1, 7.8.2.2, 7.8.2.3, 7.8.2.4, 7.22.6.1, 7.22.6.2, 7.22.1).

- The program modifies the string pointed to by the value returned by the setlocale function (7.11.1.1).

- The program modifies the structure pointed to by the value returned by the localeconv function (7.11.2.1).

- A macro definition of math\_errhandling is suppressed or the program defines an identifier with the name math\_errhandling (7.12).

- An argument to a floating-point classification or comparison macro is not of real floating type (7.12.3, 7.12.14).

- A macro definition of setjmp is suppressed in order to access an actual function, or the program defines an external identifier with the name setjmp (7.13).

- An invocation of the setjmp macro occurs other than in an allowed context (7.13.2.1).

- The longjmp function is invoked to restore a nonexistent environment (7.13.2.1).

- After a longjmp, there is an attempt to access the value of an object of automatic storage duration that does not have volatile-qualified type, local to the function containing the invocation of the corresponding setjmp macro, that was changed between the setjmp invocation and longjmp call (7.13.2.1).

- The program specifies an invalid pointer to a signal handler function (7.14.1.1).

- A signal handler returns when the signal corresponded to a computational exception (7.14.1.1).

- A signal occurs as the result of calling the abort or raise function, and the signal handler calls the raise function (7.14.1.1).

- A signal occurs other than as the result of calling the abort or raise function, and the signal handler refers to an object with static or thread storage duration that is not a lock-free atomic object other than by assigning a value to an object declared as volatile sig\_atomic\_t, or calls any function in the standard library other than the abort function, the \_Exit function, the quick\_exit function, or the signal function (for the same signal number) (7.14.1.1).

- The value of errno is referred to after a signal occurred other than as the result of calling the abort or raise function and the corresponding signal handler obtained a SIG\_ERR return from a call to the signal function (7.14.1.1).

- A signal is generated by an asynchronous signal handler (7.14.1.1).

- A function with a variable number of arguments attempts to access its varying arguments other than through a properly declared and initialized va\_list object, or before the va\_start macro is invoked (7.16, 7.16.1.1, 7.16.1.4).

- The macro va\_arg is invoked using the parameter ap that was passed to a function that invoked the macro va\_arg with the same parameter (7.16).

- A macro definition of va\_start, va\_arg, va\_copy, or va\_end is suppressed in order to access an actual function, or the program defines an external identifier with the name va\_copy or va\_end (7.16.1).

- The va\_start or va\_copy macro is invoked without a corresponding invocation of the va\_end macro in the same function, or vice versa (7.16.1, 7.16.1.2, 7.16.1.3, 7.16.1.4).

- The type parameter to the va\_arg macro is not such that a pointer to an object of that type can be obtained simply by postfixing a \* (7.16.1.1).

- The va\_arg macro is invoked when there is no actual next argument, or with a specified type that is not compatible with the promoted type of the actual next argument, with certain exceptions (7.16.1.1).

- The va\_copy or va\_start macro is called to initialize a va\_list that was previously initialized by either macro without an intervening invocation of the va\_end macro for the same va\_list (7.16.1.2, 7.16.1.4).

- The parameter parmN of a va\_start macro is declared with the register storage class, with a function or array type, or with a type that is not compatible with the type that results after application of the default argument promotions (7.16.1.4).

- The member designator parameter of an offsetof macro is an invalid right operand of the . operator for the type parameter, or designates a bit-field (7.19).

- The argument in an instance of one of the integer-constant macros is not a decimal, octal, or hexadecimal constant, or it has a value that exceeds the limits for the corresponding type (7.20.4).

- A byte input/output function is applied to a wide-oriented stream, or a wide character input/output function is applied to a byte-oriented stream (7.21.2).

- Use is made of any portion of a file beyond the most recent wide character written to a wide-oriented stream (7.21.2).

- The value of a pointer to a FILE object is used after the associated file is closed (7.21.3).

- The stream for the fflush function points to an input stream or to an update stream in which the most recent operation was input (7.21.5.2).

- The string pointed to by the mode argument in a call to the fopen function does not exactly match one of the specified character sequences (7.21.5.3).

- An output operation on an update stream is followed by an input operation without an intervening call to the fflush function or a file positioning function, or an input operation on an update stream is followed by an output operation with an intervening call to a file positioning function (7.21.5.3).

- An attempt is made to use the contents of the array that was supplied in a call to the setvbuf function (7.21.5.6).

- There are insufficient arguments for the format in a call to one of the formatted input/output functions, or an argument does not have an appropriate type (7.21.6.1, 7.21.6.2, 7.28.2.1, 7.28.2.2).

- The format in a call to one of the formatted input/output functions or to the strftime or wcsftime function is not a valid multibyte character sequence that begins and ends in its initial shift state (7.21.6.1, 7.21.6.2, 7.26.3.5, 7.28.2.1, 7.28.2.2, 7.28.5.1).

- In a call to one of the formatted output functions, a precision appears with a conversion specifier other than those described (7.21.6.1, 7.28.2.1).

- A conversion specification for a formatted output function uses an asterisk to denote an argument-supplied field width or precision, but the corresponding argument is not provided (7.21.6.1, 7.28.2.1).

- A conversion specification for a formatted output function uses a # or 0 flag with a conversion specifier other than those described (7.21.6.1, 7.28.2.1).

- A conversion specification for one of the formatted input/output functions uses a length modifier with a conversion specifier other than those described (7.21.6.1, 7.21.6.2, 7.28.2.1, 7.28.2.2).

- An s conversion specifier is encountered by one of the formatted output functions, and the argument is missing the null terminator (unless a precision is specified that does not require null termination) (7.21.6.1, 7.28.2.1).

- An n conversion specification for one of the formatted input/output functions includes any flags, an assignment-suppressing character, a field width, or a precision (7.21.6.1, 7.21.6.2, 7.28.2.1, 7.28.2.2).

- A % conversion specifier is encountered by one of the formatted input/output functions, but the complete conversion specification is not exactly %% (7.21.6.1, 7.21.6.2, 7.28.2.1, 7.28.2.2).

- An inv alid conversion specification is found in the format for one of the formatted input/output functions, or the strftime or wcsftime function (7.21.6.1, 7.21.6.2, 7.26.3.5, 7.28.2.1, 7.28.2.2, 7.28.5.1).

- The number of characters transmitted by a formatted output function is greater than INT\_MAX (7.21.6.1, 7.21.6.3, 7.21.6.8, 7.21.6.10).

- The result of a conversion by one of the formatted input functions cannot be represented in the corresponding object, or the receiving object does not have an appropriate type (7.21.6.2, 7.28.2.2).

- A c, s, or \[ conversion specifier is encountered by one of the formatted input functions, and the array pointed to by the corresponding argument is not large enough to accept the input sequence (and a null terminator if the conversion specifier is s or \[) (7.21.6.2, 7.28.2.2).

- A c, s, or \[ conversion specifier with an l qualifier is encountered by one of the formatted input functions, but the input is not a valid multibyte character sequence that begins in the initial shift state (7.21.6.2, 7.28.2.2).

- The input item for a %p conversion by one of the formatted input functions is not a value converted earlier during the same program execution (7.21.6.2, 7.28.2.2).

- The vfprintf, vfscanf, vprintf, vscanf, vsnprintf, vsprintf, vsscanf, vfwprintf, vfwscanf, vswprintf, vswscanf, vwprintf, or vwscanf function is called with an improperly initialized va\_list argument, or the argument is used (other than in an invocation of va\_end) after the function returns (7.21.6.8, 7.21.6.9, 7.21.6.10, 7.21.6.11, 7.21.6.12, 7.21.6.13, 7.21.6.14, 7.28.2.5, 7.28.2.6, 7.28.2.7, 7.28.2.8, 7.28.2.9, 7.28.2.10).

- The contents of the array supplied in a call to the fgets or fgetws function are used after a read error occurred (7.21.7.2, 7.28.3.2).

- The file position indicator for a binary stream is used after a call to the ungetc function where its value was zero before the call (7.21.7.10).

- The file position indicator for a stream is used after an error occurred during a call to the fread or fwrite function (7.21.8.1, 7.21.8.2).

- A partial element read by a call to the fread function is used (7.21.8.1).

- The fseek function is called for a text stream with a nonzero offset and either the offset was not returned by a previous successful call to the ftell function on a stream associated with the same file or whence is not SEEK\_SET (7.21.9.2).

- The fsetpos function is called to set a position that was not returned by a previous successful call to the fgetpos function on a stream associated with the same file (7.21.9.3).

- A non-null pointer returned by a call to the calloc, malloc, or realloc function with a zero requested size is used to access an object (7.22.3).

- The value of a pointer that refers to space deallocated by a call to the free or realloc function is used (7.22.3).

- The alignment requested of the aligned\_alloc function is not valid or not supported by the implementation, or the size requested is not an integral multiple of the alignment (7.22.3.1).

- The pointer argument to the free or realloc function does not match a pointer earlier returned by a memory management function, or the space has been deallocated by a call to free or realloc (7.22.3.3, 7.22.3.5).

- The value of the object allocated by the malloc function is used (7.22.3.4).

- The value of any bytes in a new object allocated by the realloc function beyond the size of the old object are used (7.22.3.5).

- The program calls the exit or quick\_exit function more than once, or calls both functions (7.22.4.4, 7.22.4.7).

- During the call to a function registered with the atexit or at\_quick\_exit function, a call is made to the longjmp function that would terminate the call to the registered function (7.22.4.4, 7.22.4.7).

- The string set up by the getenv or strerror function is modified by the program (7.22.4.6, 7.23.6.2).

- A command is executed through the system function in a way that is documented as causing termination or some other form of undefined behavior (7.22.4.8).

- A searching or sorting utility function is called with an invalid pointer argument, even if the number of elements is zero (7.22.5).

- The comparison function called by a searching or sorting utility function alters the contents of the array being searched or sorted, or returns ordering values inconsistently (7.22.5).

- The array being searched by the bsearch function does not have its elements in proper order (7.22.5.1).

- The current conversion state is used by a multibyte/wide character conversion function after changing the LC\_CTYPE category (7.22.7).

- A string or wide string utility function is instructed to access an array beyond the end of an object (7.23.1, 7.28.4).

- A string or wide string utility function is called with an invalid pointer argument, even if the length is zero (7.23.1, 7.28.4).

- The contents of the destination array are used after a call to the strxfrm, strftime, wcsxfrm, or wcsftime function in which the specified length was too small to hold the entire null-terminated result (7.23.4.5, 7.26.3.5, 7.28.4.4.4, 7.28.5.1).

- The first argument in the very first call to the strtok or wcstok is a null pointer (7.23.5.8, 7.28.4.5.7).

- The type of an argument to a type-generic macro is not compatible with the type of the corresponding parameter of the selected function (7.24).

- A complex argument is supplied for a generic parameter of a type-generic macro that has no corresponding complex function (7.24).

- At least one field of the broken-down time passed to asctime contains a value outside its normal range, or the calculated year exceeds four digits or is less than the year 1000 (7.26.3.1).

- The argument corresponding to an s specifier without an l qualifier in a call to the fwprintf function does not point to a valid multibyte character sequence that begins in the initial shift state (7.28.2.11).

- In a call to the wcstok function, the object pointed to by ptr does not have the value stored by the previous call for the same wide string (7.28.4.5.7).

- An mbstate\_t object is used inappropriately (7.28.6).

- The value of an argument of type wint\_t to a wide character classification or case mapping function is neither equal to the value of WEOF nor representable as a wchar\_t (7.29.1).

- The iswctype function is called using a different LC\_CTYPE category from the one in effect for the call to the wctype function that returned the description (7.29.2.2.1).

- The towctrans function is called using a different LC\_CTYPE category from the one in effect for the call to the wctrans function that returned the description (7.29.3.2.1).

Most of these items are already detected, could be detected easily, or would be detected as a side effect of solving UBs that we discussed in detail. In other words, a few basic technologies, such as shadow memory and run-time type information, provide the infrastructure needed to detect a large fraction of the hard-to-detect UBs. Alas it is difficult to make shadow memory and run-time type information fast.

## Summary

What is the modern C or C++ developer to do?

- Be comfortable with the “easy” UB tools — the ones that can usually be enabled just by adjusting a makefile, such as compiler warnings and ASan and UBSan. Use these early and often, and (crucially) act upon their findings.

- Be familiar with the “hard” UB tools — those such as TIS Interpreter that typically require more effort to run — and use them when appropriate.

- Invest in broad-based testing (track code coverage, use fuzzers, etc.) in order to get maximum benefit out of dynamic UB detection tools.

- Perform UB-aware code reviews: build a culture where we collectively diagnose potentially dangerous patches and get them fixed before they land.

- Be knowledgeable about what’s actually in the C and C++ standards since these are what compiler writers are going by. Avoid repeating tired maxims like “C is a portable assembly language” and “trust the programmer.”

Unfortunately, C and C++ are mostly taught the old way, as if programming in them isn’t like walking in a minefield. Nor have the books about C and C++ caught up with the current reality. These things must change.

Good luck, everyone.

*We’d like to thank various people, especially @CopperheadOS on Twitter, for discussing these issues with us.*
