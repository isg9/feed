---
title: Uninitialized Variables
url: https://blog.regehr.org/archives/519
published: "2011-04-25T07:11:24Z"
feed: regehr
guid: http://blog.regehr.org/?p=519
---

# Uninitialized Variables

I’ve been tempted, a couple of times, to try to discover how much performance realistic C/C++ programs gain through the languages’ failure to automatically initialize function-scoped storage. It would be easy to take a source-to-source transformer like [CIL](http://www.eecs.berkeley.edu/~necula/cil/) and use it to add an explicit initializer to every variable that lacks one. Then, presumably, a modern optimizing compiler would eliminate the large fraction of initializers that are dominated by subsequent stores. If compilers are not smart enough to do this, and overhead remains high, a more sophisticated approach would be to:

- Only initialize a variable when some powerful (but conservative) interprocedural static analyzer thinks it may be needed
- Initialize a variable close to its first use, to avoid problems with locality and with artificially lengthening the live range

My guess is that many programs would not be slowed down noticeably by automatic initialization, but that it would not be too hard to find codes that slow down 5%-20%.

Lacking automatic initialization, most of us get by using compiler warnings and dynamic tools like Valgrind. I was recently surprised to learn that warnings+Valgrind are not as reliable as I’d have hoped. First, the compiler warnings are fundamentally best-effort in the sense that common compilers more or less give up on code using arrays, pointers, function calls, and some kinds of loops. For example:

> ```
> int foo1 (void) {
>   int y,z;
>   for (y=0; y<5; y++)
>     z++;
>   return z;
> }
>
> int foo2 (void) {
>   int a[15];
>   return a[10];
> }
>
> void bar (int *p) {
> }
>
> int foo3 (void) {
>   int x;
>   bar (&x);
>   return x;
> }
> ```

Each of foo1(), foo2(), and foo3() returns a value that depends on a read from uninitialized storage, but recent versions of GCC and Clang fail to give a warning about this, at least for the command line options I could think to try. Intel CC 12.0 finds the first two and Clang’s static analyzer finds the problem with foo2(). However, both ICC and Clang’s analyzer are fairly easy to fool. For example, neither gives any warning about this code:

> ```
> int foo2b (void) {
>   int a[6], i;
>   for (i=0; i<5; i++) {
>     a[i] = 1;
>   }
>   return a[5];
> }
> ```

Next let’s talk about Valgrind. It is greatly handicapped by the fact that optimizing compilers seek out and destroy code relying on undefined behaviors. This means that Valgrind never gets a chance to see the problems, preventing it from reporting errors. We can turn the functions above into a complete program by adding:

> ```
> int main (void) {
>   return foo1() + foo2() + foo3();
> }
> ```

When compiled by GCC or Clang at -O2, the resulting executable passes through Valgrind with zero errors found. But let’s be clear: the problem still exists, it is just hidden from Valgrind. Each function still has to return something, and the programmer has failed to specify what it is. Basically the compiler will fabricate a value out of thin air. Or, as I like to say in class, it will somehow manage to generate the worst possible value — whatever it is. On the other hand, when optimizations are turned off, the output of either GCC or Clang is properly flagged as returning a value depending on uninitialized storage.

Is Valgrind reliable when it monitors code produced at -O0? Unfortunately not: the following functions pass through without error, regardless of optimization level:

> ```
> void foo5 (void) {
>   int x,y;
>   for (x = 0; x < 5; x++)
>     y;
> }
>
> void foo6 (void) {
>   int a[1];
>   a[0];
> }
> ```

Moreover, neither GCC nor Clang warns about these. On the other hand, these functions are perhaps harmless since the uninitialized data aren’t actually used for anything. (Keep in mind, however, that according to the C standard, these functions are unambiguously wrong, and executing either of them destroys the meaning of the entire program.)

Failure to initialize function-scoped variables is one of the many holes in C/C++ that were introduced under the philosophies “trust the programmer” and “performance above all.” The resulting problems have been patched in a not-totally-satisfactory way using a collection of tools.

Is there a lesson here? Perhaps. I think it would be reasonable for tool developers to ensure that the following invariant holds:

> For all C/C++ programs, any values read from uninitialized storage either:
> - result in a compiler warning,
> - result in a Valgrind error,
> - fail to propagate (via data flow or control flow) to any system call.

I believe (but am not 100% sure) that Valgrind does not need to be modified to make this happen. Compilers definitely need to be changed. Basically, the compiler has to (1) without optimizations, generate code that permits Valgrind to detect all uses of uninitialized storage and (2) when optimizing, restrain itself from performing any transformation that conceals a problem from Valgrind, unless it emits a warning corresponding to the runtime behavior that has disappeared. Condition 1 is already the case for GCC and Clang (as far as I know) but a fair amount of work would probably be required to guarantee that condition 2 holds.

The other solution — automatic initialization — is of dubious value to C/C++ programmers because the resulting code would not be portable to other compilers. The standard would have to mandate initialization before this became generally useful, and there’s no way that’s going to happen. On the other hand, if I were an operating system vendor, and I cared at all about reliability and security, I’d probably hack the bundled compiler to do automatic initialization. Programmers writing conforming code would never notice, and sloppy code would be silently fixed instead of being perhaps exploitable.

Finally, I modified the six foo\* functions above to include explicit initializers. When compiled using GCC, the code size overhead due to initialization is 0 bytes. Using Clang, 15 bytes. Using ICC, 40 bytes. This is all at -Os on x86-64. The ICC overhead all comes from foo2() — the compiler emits code initializing all 15 array elements even though 14 of these are obviously useless. Perhaps GCC does such a good job because its middle-end has already been tuned for ahead-of-time compilation of Java code.
