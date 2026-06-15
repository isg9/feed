---
title: Assertions Are Pessimistic, Assumptions Are Optimistic
url: https://blog.regehr.org/archives/1096
published: "2014-02-05T19:55:04Z"
feed: regehr
guid: http://blog.regehr.org/?p=1096
---

# Assertions Are Pessimistic, Assumptions Are Optimistic

We *[assert](https://blog.regehr.org/archives/1091)* a condition when we believe it to be true in every non-buggy execution of our program, but we want to be notified if this isn’t the case. In contrast, we *assume* a condition when our belief in its truth is so strong that we don’t care what happens if it is ever false. In other words, while assertions are fundamentally pessimistic, assumptions are optimistic.

C and C++ compilers have a large number of built-in assumptions. For example, they assume that every pointer that is dereferenced refers to valid storage and every signed math operation fails to overflow. In fact, they make one such assumption for each of the hundreds of [undefined behaviors](https://blog.regehr.org/archives/213) in the languages. Assumptions permit the compiler to generate fast code without working too hard on static analysis.

This post explores the idea of a general-purpose assume mechanism. Although this isn’t found (as far as I know — please leave a comment if you know of something) in any commonly-used programming language, it might be useful when we require maximum performance. Although an assume mechanism seems inherently unsafe, we could use it in a safe fashion if we used a formal methods tool to prove that our assumptions hold. The role of the assumption mechanism, then, is to put high-level knowledge about program properties — whether from a human or a formal methods tool — into a form that the compiler can exploit when generating code.

Standard C has the “restrict” type qualifier that lets the compiler assume that the pointed-to object is not aliased by other pointers. Let’s consider this (deliberately silly) function for summing up the elements of an array:

```
void sum (int *array, int len, int *res)
{
  *res = 0;
  for (int i=0; i<len; i++) {
    *res += array[i];
  }
}

```

The problem that the compiler faces when translating this code is that res might point into the array. Thus, \*res has to be updated in every loop iteration. GCC and Clang generate much the same code:

**```** **sum: movl $0, (%rdx)** **xorl %eax, %eax** **.L2: cmpl %eax, %esi** **jle .L5** **movl (%rdi,%rax,4), %ecx** **incq %rax** **addl %ecx, (%rdx)** **jmp .L2** **.L5: ret**

**```**

Both compilers are able to generate code that is five times faster — using vector instructions — if we change the definition of sum() to permit the compiler to assume that res is not an alias for any part of array:

```
void sum (int *restrict array, int len, int *restrict res)

```

Another form of assumption is the \_\_builtin\_unreachable() primitive supported by GCC and Clang, which permits the compiler to assume that a particular program point cannot be reached. In principle \_\_builtin\_unreachable() is trivial to implement: any unconditionally undefined construct will suffice:

```
void unreachable_1 (void)
{
  int x;
  x++;
}

void unreachable_2 (void)
{
  1/0;
}

void unreachable_3 (void)
{
  int *p = 0;
  *p;
}

void unreachable_4 (void)
{
  int x = INT_MAX;
  x++;
}

void unreachable_5 (void)
{
  __builtin_unreachable ();
}

```

But in practice the compiler’s interpretation of undefined behavior is not sufficiently hostile — bet you never thought I’d say that! — to drive the desired optimizations. Out of the five functions above, only unreachable\_5() results in no code at all being generated (not even a return instruction) when a modern version of GCC or Clang is used.

The intended use of \_\_builtin\_unreachable() is in situations where the compiler cannot infer that code is dead, for example following a block of inline assembly that has no outgoing control flow. Similarly, if we compile this code we will get a return instruction at the bottom of the function even when x is guaranteed to be in the range 0-2.

```
int foo (int x)
{
  switch (x) {
  case 0: return bar(x);
  case 1: return baz(x);
  case 2: return buz(x);
  }
  return x; // gotta return something and x is already in a register...
}

```

If we add a \_\_builtin\_unreachable() at the bottom of the function, or in the default part of the switch, then the compiler drops the return instruction. If the assumption is violated, for example by calling foo(7), execution will fall into whatever code happens to be next — undefined behavior at its finest.

But what about the general-purpose assume()? This turns out to be easy to implement:

```
void assume (int expr)
{
  if (!expr) __builtin_unreachable();
}

```

So assume() is using \_\_builtin\_unreachable() to kill off the collection of program paths in which expr fails to be true. The interesting question is: Can our compilers make use of assumptions to generate better code? The results are a bit of a mixed bag. The role of assume() is to generate dataflow facts and, unfortunately, the current crop of compilers can only be trusted to learn very basic kinds of assumptions. Let’s look at some examples.

First, we might find it annoying that integer division in C by a power of 2 cannot be implemented using a single shift instruction. For example, this code:

```
int div32 (int x)
{
  return x/32;
}

```

Results in this assembly from GCC:

**```** **div32: leal 31(%rdi), %eax** **testl %edi, %edi** **cmovns %edi, %eax** **sarl $5, %eax** **ret**

**```**

And this from Clang:

**```** **div32: movl %edi, %eax** **sarl $31, %eax** **shrl $27, %eax** **addl %edi, %eax** **sarl $5, %eax** **ret**

**```**

The possibility of a negative dividend is causing the ugly code. Assuming that we know that the argument will be non-negative, we try to fix the problem like this:

```
int div32_nonneg (int x)
{
  assume (x >= 0);
  return div32 (x);
}

```

Now GCC nails it:

**```** **div32_nonneg:** **movl %edi, %eax** **sarl $5, %eax** **ret**

**```**

Clang, unfortunately, generates the same code from div32\_nonneg() as it does from div32(). Perhaps it lacks the proper value range analysis. I’m using Clang 3.4 and GCC 4.8.2 for this work, by the way. **UPDATE:** In a comment Chris Lattner provides a link to [this known issue in LLVM](http://llvm.org/bugs/show_bug.cgi?id=810).

Next we’re going to increment the value of each element of an array:

```
void inc_array (int *x, int len)
{
  int i;
  for (i=0; i<len; i++) {
    x[i]++;
  }
}

```

The code generated by GCC -O2 is not too bad:

**```** **inc_array:** **testl %esi, %esi** **jle .L18** **subl $1, %esi** **leaq 4(%rdi,%rsi,4), %rax** **.L21: addl $1, (%rdi)** **addq $4, %rdi** **cmpq %rax, %rdi** **jne .L21** **.L18: rep ret**

**```**

However, is that test at the beginning really necessary? Aren’t we always going to be incrementing an array of length at least one? If so, let’s try this:

```
void inc_nonzero_array (int *x, int len)
{
  assume (len > 0);
  inc_array (x, len);
}

```

Now the output is cleaner:

**```** **inc_nonzero_array:** **subl $1, %esi** **leaq 4(%rdi,%rsi,4), %rax** **.L24: addl $1, (%rdi)** **addq $4, %rdi** **cmpq %rax, %rdi** **jne .L24** **rep ret**

**```**

The preceding example was suggested by Arthur O’Dwyer in a [comment](https://blog.regehr.org/archives/1091#comment-13972) on my assertion post.

If you readers have any good examples where non-trivial assumptions result in improved code, I’d be interested to see them.

Next, let’s look at the effect of assumptions in a larger setting. I compiled LLVM/Clang in four different modes. First, in its default “Release+Assertions” configuration. Second, in a “minimal assert” configuration where assert() was defined like this:

**```** **#define assert(expr) ((expr) ? static_cast<void>(0) : _Exit(-1))**

**```**

This simply throws away the verbose failure information. Third, in a “no assert” configuration where assert() was defined like this:

**```** **#define assert(expr) (static_cast<void>(0))**

**```**

Fourth, I turned each assertion in LLVM/Clang into an assumption like this:

**```** **#define assert(expr) ((expr) ? static_cast<void>(0) : __builtin_unreachable())**

**```**

Then I built each of the configurations using GCC 4.8.2 and Clang 3.4, giving a total of eight Clang binaries as a result. Here’s the code size:

![](https://blog.regehr.org/extra_files/clang-size.png)

Bear in mind that the y-axis doesn’t start at zero, I wanted to make the differences stand out. As expected, default assertions have a heavy code size penalty. Perhaps interestingly, GCC was able to exploit assumptions to the point where there was a small overall code size win. LLVM did not manage to profit from assumptions. Why would assumptions have a code size penalty over no assertions? My guess is that some assumptions end up making function calls that cannot be optimized away, despite their lack of side effects.

Now let’s look at the performance of the eight clangs. The benchmark here was optimized compilation of a collection of C++ files. Each number in the graph is the median value of 11 reps, but really this precaution was not necessary since there was very little variance between reps. Note again that the y-axis doesn’t start at zero.

![](https://blog.regehr.org/extra_files/clang-speed.png)

Both compilers are slowed down by assumes. Again, I would guess this is because sometimes the compiler cannot elide function calls that are made during the computation of a expression that is being assumed.

One surprise from these graphs is that Clang is generating code that is both smaller and faster than GCC’s. My view (formed several years ago) has been that Clang usually produces slightly slower code, but requires less compile time to do it. This view could easily have become invalidated by rapid progress in the compiler. On the other hand, since the code being compiled is LLVM/Clang itself, perhaps we should expect that Clang has been extensively tuned to generate good object code in this case.

A not-surprise from these graphs is that we generally cannot just take a big collection of assertions, turn them into assumptions, and expect good results. Getting good results has two parts: an easy one and a hard one. The easy part is selectively removing assumptions that are hurting more than they help. These would tend to be complex conditions that are slow to compute and that have no chance of generating dataflow facts that the compiler can make use of. This picking and choosing could even be done automatically.

The second part of getting good results out of assumptions is creating compilers that are more generally capable of learning and exploiting arbitrary new dataflow facts provided by users. In the short run, this is a matter of testing and fixing and tuning things up. In the long run, my belief is that compilers are unnecessarily hampered by performance constraints — they have to be pretty fast, even when compiling large codes. An alternative is to support a “-Osearch” mode where the compiler spends perhaps a lot of time looking for better code sequences; see this comment from [bcs](https://blog.regehr.org/archives/1091#comment-13991). The compiler’s search could be [randomized](http://cs.stanford.edu/people/eschkufz/research/asplos291-schkufza.pdf) or it could involve integration with an SMT solver. Companies that burn a lot of CPU cycles in server farms should be highly motivated to optimize, for example, the 500 functions that use the most power every day. I’m assuming that some sort of systematic cluster-wide profiling facility exists, we don’t want to waste time optimizing code unless it’s causing pain.

The postcondition of any assumption is the same as the postcondition of asserting the same expression. Thus, we can see that asserts can also make our code faster — but this requires the speedup from the extra dataflow facts to outweigh the slowdown from the assertion check. I don’t know that anyone has ever looked into the issue of when and where assertions lead to faster code. We would not expect this to happen too often, but it would be cute if compiling some big program with -DNDEBUG made it slower.

Although they would normally be used to support optimization, I believe that assumptions can also directly support bug detection. A tool such as [Stack](http://css.csail.mit.edu/stack/) could be modified to analyze a program with the goal of finding code that becomes dead due to an assumption. The presence of such code is likely to signal a bug.

In summary, assume() is a mechanism for introducing new undefined behaviors into programs that we write. Although this mechanism would be easy to misuse, there’s no reason why it cannot be used safely and effectively when backed up by good programming practices and formal methods.
