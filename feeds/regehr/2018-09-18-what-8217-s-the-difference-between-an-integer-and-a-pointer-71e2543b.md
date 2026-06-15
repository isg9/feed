---
title: What&#8217;s the difference between an integer and a pointer?
url: https://blog.regehr.org/archives/1621
published: "2018-09-18T14:10:55Z"
feed: regehr
guid: http://blog.regehr.org/?p=1621
---

# What&#8217;s the difference between an integer and a pointer?

(This piece is an alternate introduction and advertisement for [a soon-to-be-published research paper](http://www.cs.utah.edu/~regehr/oopsla18.pdf).)

In an assembly language we typically don’t have to worry very much about the distinction between pointers and integers. Some instructions happen to generate addresses whereas others behave arithmetically, but underneath there’s a single data type: bitvectors. At the opposite end of the PL spectrum, a high-level language won’t offer opportunities for pointer/integer confusion because the abstractions are completely firewalled off from each other. Also, of course, a high-level language may choose not to expose anything that resembles a pointer.

The problematic case, then, is the performance-oriented systems programming language: C, C++, Rust, and a few others. The issue is that:

1. When possible, they pretend to be high-level, in order to justify optimizations that would be unsound at the assembly level. For example, they assume that a pointer derived from one heap allocation cannot alias a pointer derived from a different heap allocation.

2. When necessary, they allow the programmer to manipulate pointers in low-level ways, such as inspecting the representation of a pointer, creating a new pointer at some offset from an existing one, storing unrelated data in unused parts of a pointer, and even fabricating a pointer out of whole cloth.

These goals conflict: the better a language is for justifying high-level optimizations, the worse it seems to be for implementing an OS kernel, and vice versa. There are a few ways we might deal with this:

- We could require the high-level language to call out to low-level code written in a different language, as Java does with its JNI.

- We could create a special low-level mode for a safe language where it can break the normal rules, as Rust does with its unsafe code.

- We could go ahead and freely mix high-level and low-level code, hoping that the compiler can sort it all out, as C and C++ do.

Let’s look at an example:

```
#include <stdio.h>
#include <stdlib.h>

int main(void) {
  int *x = malloc(4);
  *x = 3;
  int *y = malloc(4);
  *y = 5;
  uintptr_t xi = (uintptr_t)x;
  uintptr_t yi = (uintptr_t)y;
  uintptr_t diff = xi - yi;
  printf("diff = %ld\n", (long)diff);
  int *p = (int *)(yi + diff);
  *p = 7;
  printf("x = %p, *x = %d\n", x, *x);
  printf("y = %p, *y = %d\n", y, *y);
  printf("p = %p, *p = %d\n", p, *p);
}

```

When run (on my Mac) it gives:

```
$ clang-6.0 -O3 mem1d.c ; ./a.out
diff = -96
x = 0x7fcb0ec00300, *x = 7
y = 0x7fcb0ec00360, *y = 5
p = 0x7fcb0ec00300, *p = 7
$ gcc-8 -O3 mem1d.c ; ./a.out
diff = -96
x = 0x7f93b6400300, *x = 7
y = 0x7f93b6400360, *y = 5
p = 0x7f93b6400300, *p = 7

```

What happened here? First, we grabbed a couple of heap cells and initialized their contents. Second, we used integer math to fabricate a pointer p that has the same value as x, and stored through that pointer. Third, we asked the compiler what contents it thinks are in the locations pointed to by x, y, and p. Both GCC and Clang/LLVM are of the opinion that the store through p changed the value referred to by x. This is probably the result that we had hoped for, but let’s look into it a bit more deeply.

First, does this C program execute undefined behavior? It does not! Computing out-of-bounds pointers is UB but we are free to do whatever we like with integer-typed values that were derived from pointers. Second, does the C standard mandate the behavior that we saw here? It does not! In fact it gives only very sparse guidance about the behavior of integers derived from pointers and vice versa. [You can read about that here](https://nullprogram.com/blog/2016/05/30/).

Now let’s modify the program a tiny bit:

```
#include <stdio.h>
#include <stdlib.h>

int main(void) {
  int *x = malloc(4);
  *x = 3;
  int *y = malloc(4);
  *y = 5;
  uintptr_t xi = (uintptr_t)x;
  uintptr_t yi = (uintptr_t)y;
  uintptr_t diff = xi - yi;
  printf("diff = %ld\n", (long)diff);
  int *p = (int *)(yi - 96); // <--- CHANGED!
  *p = 7;
  printf("x = %p, *x = %d\n", x, *x);
  printf("y = %p, *y = %d\n", y, *y);
  printf("p = %p, *p = %d\n", p, *p);
}

```

The only difference is at line 13: instead of adding `diff` to yi to compute p, we added the observed value of the difference: -96. Now we’ll run it:

```
$ clang-6.0 -O3 mem1d2.c ; ./a.out
diff = -96
x = 0x7ff21ac00300, *x = 7
y = 0x7ff21ac00360, *y = 5
p = 0x7ff21ac00300, *p = 7
$ gcc-8 -O3 mem1d2.c ; ./a.out
diff = -96
x = 0x7fce5e400300, *x = 3
y = 0x7fce5e400360, *y = 5
p = 0x7fce5e400300, *p = 7

```

Clang behaves the same as before, but GCC no longer believes that storing through p results in x being overwritten, despite the fact that x and p have the same value. What is going on is that as far as GCC is concerned, since we computed the address of x through guesswork instead of through an actual observation, our guess is effectively illegitimate. (If this guessing game is too simple for you, there are entertaining variations that avoid the need for information from a previous run of the program, such as guessing addresses using an exhaustive loop or making huge allocations so that the resulting addresses are constrained by the pigeonhole principle.) Here again we are operating well outside of the C standard and are rather in a zone where compiler writers need to make decisions that balance optimization power against developers’ expectations. I like this example because it is weird and perhaps surprising.

What should we take away from this? First, it is a big mistake to try to understand pointers in a programming language as if they follow the same rules as pointer-sized integer values. Even people who have come to terms with UB and its consequences are often surprised by this. Second, these issues are not specific to C and C++, but rather are going to create problems in any language that wants highly optimizing compilers but also permits low-level pointer manipulation.

So what are the actual rules that Clang and GCC are following when compiling these examples? The short answer is that it’s complicated and not well documented. Some useful discussion can be found in [this piece about pointer provenance](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2263.htm). For a longer answer that is specific to LLVM, see [this paper that will get presented at OOPSLA in November](http://www.cs.utah.edu/~regehr/oopsla18.pdf). Also, one of my coauthors [wrote a blog post about this material](https://www.ralfj.de/blog/2018/07/24/pointers-and-bytes.html).
