---
title: Pointer Overflow Checking is in LLVM
url: https://blog.regehr.org/archives/1518
published: "2017-06-02T04:29:44Z"
feed: regehr
guid: http://blog.regehr.org/?p=1518
---

# Pointer Overflow Checking is in LLVM

Production-grade memory safety for legacy C and C++ code has proven to be a frustratingly elusive goal: plenty of research solutions exist but none of them appear to be deployable as-is. So instead, we have a patchwork of partial solutions such as [CFI](https://clang.llvm.org/docs/ControlFlowIntegrity.html), [ASLR](https://en.wikipedia.org/wiki/Address_space_layout_randomization), stack canaries, hardened allocators, and [NX](https://en.wikipedia.org/wiki/NX_bit).

Today’s quick post is about another piece of the puzzle that very recently landed in LLVM: pointer overflow checking. At the machine level a pointer overflow looks just like an unsigned integer overflow, but of course at the language level the overflowing operation is pointer arithmetic, not unsigned integer arithmetic. Keep in mind that in these languages, unsigned overflow is defined but signed overflow is undefined. Pointer overflow is a weak indicator of undefined behavior (UB): the stricter rule is that it is UB to create a pointer that lies more than one element outside of an allocated object. It is UB merely to create such a pointer, it does not need to be dereferenced. Also, it is still UB even if the overflowed pointer happens to refer to some other allocated object.

[Here is the patch](https://reviews.llvm.org/D33305), it was originally developed by Will Dietz (who is doing his PhD at UIUC under Vikram Adve) and then pushed into the tree by Vedant Kumar (a compiler hacker at Apple). In 2013, Will wrote a [great blog post](https://wdtz.org/catching-pointer-overflow-bugs.html) about the patch. He showed lots of examples of pointer overflows in open source programs. Also see [an earlier post of mine](https://blog.regehr.org/archives/1395).

To see pointer overflow checking in action you’ll need to build a very recent Clang/LLVM (r304461 or later) from source, and then you can try out this stupid little program:

```
$ cat pointer-overflow.c
#include  <stdio.h>
#include  <stdint.h>

int main(void) {
  for (int i, *p = &i; ; p += 1000)
    printf("%p\n", p);
}
$ clang -O3 pointer-overflow.c -Wall -fsanitize=pointer-overflow -fsanitize-trap=pointer-overflow -m32
$ ./a.out 0xff8623c4
0xff863364
0xff864304
0xff8652a4
...
0xffffd804
0xffffe7a4
0xfffff744
Illegal instruction
$

```

Of course the result is much the same if the pointer is decremented in the loop, instead of incremented; it just takes longer to hit the overflow.

The transformation implemented by the compiler here is pretty straightforward. Here’s IR for the uninstrumented program (I cleaned it up a bit):

```
define i32 @main() {
entry:
  %i = alloca i32, align 4
  br label %for.cond

for.cond:
  %p.0 = phi i32* [ %i, %entry ], [ %add.ptr, %for.cond ]
  %call = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([4 x i8], [4 x i8]* @.str, i32 0, i32 0), i32* %p.0)
  %add.ptr = getelementptr inbounds i32, i32* %p.0, i32 1000
  br label %for.cond
}

```

To instrument the program, the last two instructions are changed into these three instructions (and also a trap basic block is added, which simply calls the LLVM trap intrinsic):

```
  %1 = icmp ult i32* %p.0, inttoptr (i32 -4000 to i32*)
  %add.ptr = getelementptr inbounds i32, i32* %p.0, i32 1000
  br i1 %1, label %for.cond, label %trap

```

The icmp checks whether the not-yet-incremented pointer is below 0xfffff060, in which case it can be incremented without overflowing.

Can pointer overflow checking by used as a mitigation in production code? This should be fine if you (as I did above) use the `-fsanitize-trap=pointer-overflow` flag to avoid dragging in any of the UBSan runtime library. But how efficient is it? I ran SPEC INT 2006 with and without pointer overflow checking. 400.perlbench actually contains pointer overflows so we’ll leave it out. Here are the raw scores [with](../extra_files/spec-pointer-overflow-checking.pdf) and [without](../extra_files/spec-no-pointer-overflow-checking.pdf) pointer overflow checking, and here are the increases in runtime due to pointer overflow checking, sorted from best to worst:

462.libquantum-1%429.mcf5%471.omnetpp5%403.gcc9%483.xalancbmk12%473.astar27%401.bzip234%445.gobmk50%458.sjeng79%464.h264ref113%456.hmmer119%

Keep in mind that this implementation is totally untuned (the patch landed just today). No doubt these scores could be improved by teaching LLVM to eliminate unnecessary overflow checks and, when that doesn’t work, to hoist checks out of inner loops.

Although, in the example above, I enabled pointer overflow checking using an explicit flag, these checks are now part of UBSan and `-fsanitize=undefined` will enable them.
