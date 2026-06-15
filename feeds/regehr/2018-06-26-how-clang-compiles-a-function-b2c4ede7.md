---
title: How Clang Compiles a Function
url: https://blog.regehr.org/archives/1605
published: "2018-06-26T22:41:08Z"
feed: regehr
guid: http://blog.regehr.org/?p=1605
---

# How Clang Compiles a Function

I’ve been planning on writing a post about how LLVM optimizes a function, but it seems necessary to first write about how Clang translates C or C++ into LLVM. This is going to be fairly high-level:

- rather than looking at Clang’s internals, I’m going to focus on how Clang’s output relates to its input

- we’re not going to look at any non-trivial C++ features

We’ll use this small function that I borrowed from [these excellent lecture notes on loop optimizations](https://www.cs.cmu.edu/~fp/courses/15411-f13/lectures/17-loopopt.pdf):

```
bool is_sorted(int *a, int n) {
  for (int i = 0; i < n - 1; i++)
    if (a[i] > a[i + 1])
      return false;
  return true;
}

```

Since Clang doesn’t do any optimization, and since LLVM IR was initially designed as a target for C and C++, the translation is going to be relatively easy. I’ll use Clang 6.0.1 (or something near it, since it hasn’t quite been released yet) on x86-64.

The command line is:

```
clang++ is_sorted.cpp -O0 -S -emit-llvm

```

In other words: compile the file is\_sorted.cpp as C++ and then tell the rest of the LLVM toolchain: don’t optimize, emit assembly, but no actually emit textual LLVM IR instead. Textual IR is bulky and not particularly fast to print or parse; the binary “bitcode” format is always preferred when a human isn’t in the loop. The [full IR file is here](https://blog.regehr.org/extra_files/is_sorted.ll.txt), we’ll be looking at it in parts.

Starting at the top of the file we have:

```
; ModuleID = 'is_sorted.cpp'
source_filename = "is_sorted.cpp"
target datalayout = "e-m:e-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-unknown-linux-gnu"

```

Any text between a semicolon and the end of a line is a comment, so the first line does nothing, but if you care, an LLVM “module” is basically a compilation unit: a container for code and data. The second line doesn’t concern us. The third line describes some choices and assumptions made by the compiler; they don’t matter much for this post but you can [read more here](https://llvm.org/docs/LangRef.html#data-layout). The [target triple](https://wiki.osdev.org/Target_Triplet) is a gcc-ism that doesn’t concern us here either.

An LLVM function optionally has attributes:

```
; Function Attrs: noinline nounwind optnone uwtable

```

Some of them (like these) are supplied by the front end, others get added later by optimization passes. This is just a comment, the actual attributes are specified towards the end of the file. These attributes don’t have anything to do with the meaning of the code, so I’m not going to discuss them, but you can read [read about them here](https://llvm.org/docs/LangRef.html#function-attributes) if you’re curious.

Ok, finally on to the function itself:

```
define zeroext i1 @_Z9is_sortedPii(i32* %a, i32 %n) #0 {

```

“zeroext” means that the return value of the function (i1, a one-bit integer) should be zero-extended, by the backend, to whatever width the ABI requires. Next comes the mangled name of the function, then its parameter list, which is basically the same as in the C++ code, except that the “int” type has been refined to 32 bits. The #0 connects this function to an [attribute group](https://llvm.org/docs/LangRef.html#attribute-groups) near the bottom of the file.

Now on to the first basic block:

```
entry:
  %retval = alloca i1, align 1
  %a.addr = alloca i32*, align 8
  %n.addr = alloca i32, align 4
  %i = alloca i32, align 4
  store i32* %a, i32** %a.addr, align 8
  store i32 %n, i32* %n.addr, align 4
  store i32 0, i32* %i, align 4
  br label %for.cond

```

Every LLVM instruction has to live inside a basic block: a collection of instructions that is only entered at the top and only exited at the bottom. The last instruction of a basic block must be a [terminator instruction](https://llvm.org/docs/LangRef.html#terminators): fallthrough is not allowed. Every function must have an entry block that has no predecessors (places to come from) other than the function entry itself. [These properties and others](https://llvm.org/docs/LangRef.html#well-formedness) are checked when an IR file is parsed, and can optionally be checked more times during compilation by the “module verifier.” The verifier is useful for debugging situations where a pass emits illegal IR.

The first four instructions in the entry basic block are “alloca”s: stack memory allocations. The first three are for implicit variables created during the compilation, the fourth is for the loop induction variable. Storage allocated like this can only be accessed using load and store instructions. The next three instructions initialize three of the stack slots: a.addr and n.addr are initialized using the values passed into the function as parameters and i is initialized to zero. The return value does not need to be initialized: this will be taken care of later by any code that wasn’t undefined at the C or C++ level. The last instruction is an unconditional branch to the subsequent basic block (we’re not going to worry about it here, but most of these unnecessary jumps can be elided by an LLVM backend).

You might ask: why does Clang allocate stack slots for a and n? Why not just use those values directly? In this function, since a and n are never modified, this strategy would work, but noticing this fact is considered an optimization and thus outside of the job Clang was designed to do. In the general case where a and n might be modified, they must live in memory as opposed to being SSA values, which are — by definition — given a value at only one point in a program. Memory cells live outside of the SSA world and can be modified freely. This may all seem confusing and arbitrary but these design decisions have been found to allow many parts of a compiler to be expressed in natural and efficient ways.

I think of Clang as emitting degenerate SSA code: it meets all the requirements for SSA, but only because basic blocks communicate through memory. Emitting non-degenerate SSA requires some care and some analysis and Clang’s refusal to do these leads to a pleasing separation of concerns. I haven’t seen the measurements but my understanding is that generating a lot of memory operations and then almost immediately optimizing many of them away isn’t a major source of compile-time overhead.

Next let’s look at how to translate a for loop. The general form is:

```
for (initializer; condition; modifier) {
  body
}

```

This is translated roughly as:

```
  initializer
  goto COND
COND:
  if (condition)
    goto BODY
  else
    goto EXIT
BODY:
  body
  modifier
  goto COND
EXIT:

```

Of course this translation isn’t specific to Clang: any C or C++ compiler would do the same.

In our example, the loop initializer has been folded into the entry basic block. The next basic block is the loop condition test:

```
for.cond:                                         ; preds = %for.inc, %entry
  %0 = load i32, i32* %i, align 4
  %1 = load i32, i32* %n.addr, align 4
  %sub = sub nsw i32 %1, 1
  %cmp = icmp slt i32 %0, %sub
  br i1 %cmp, label %for.body, label %for.end

```

As a helpful comment, Clang is telling us that this basic block can be reached either from the for.inc or the entry basic block. This block loads i and n from memory, decrements n (the “nsw” flag preserves the C++-level fact that signed overflow is undefined; without this flag an LLVM subtract would have two’s complement semantics), compares the decremented value against i using “signed less than”, and then finally branches either to the for.body basic block or the for.end basic block.

The body of the for loop can only be entered from the for.cond block:

```
for.body:
  %2 = load i32*, i32** %a.addr, align 8
  %3 = load i32, i32* %i, align 4
  %idxprom = sext i32 %3 to i64
  %arrayidx = getelementptr inbounds i32, i32* %2, i64 %idxprom
  %4 = load i32, i32* %arrayidx, align 4
  %5 = load i32*, i32** %a.addr, align 8
  %6 = load i32, i32* %i, align 4
  %add = add nsw i32 %6, 1
  %idxprom1 = sext i32 %add to i64
  %arrayidx2 = getelementptr inbounds i32, i32* %5, i64 %idxprom1
  %7 = load i32, i32* %arrayidx2, align 4
  %cmp3 = icmp sgt i32 %4, %7
  br i1 %cmp3, label %if.then, label %if.end

```

The first two lines load a and i into SSA registers; i is then widened to 64 bits so it can participate in an address computation. [getelementptr](https://llvm.org/docs/LangRef.html#getelementptr-instruction) (gep for short) is LLVM’s famously baroque pointer computation instruction, it even has [its own faq](https://llvm.org/docs/GetElementPtr.html). Unlike a machine language, LLVM does not treat pointers and integers the same way. This facilitates alias analysis and other memory optimizations. The code then goes on to load a\[i\] and a\[i + 1\], compare them, and branch based on the result.

The if.then block stores 0 into the stack slot for the function return value and branches unconditionally to the function exit block:

```
if.then:
  store i1 false, i1* %retval, align 1
  br label %return

```

The else block is trivial:

```
if.end:
  br label %for.inc

```

And the block for adding one to the loop induction variable is easy as well:

```
for.inc:
  %8 = load i32, i32* %i, align 4
  %inc = add nsw i32 %8, 1
  store i32 %inc, i32* %i, align 4
  br label %for.cond

```

This code branches back up to the loop condition test.

If the loop terminates normally, we want to return true:

```
for.end:
  store i1 true, i1* %retval, align 1
  br label %return

```

And finally whatever got stored into the return value stack slot is loaded and returned:

```
return:
  %9 = load i1, i1* %retval, align 1
  ret i1 %9

```

There’s a bit more at the bottom of the function but nothing consequential. Ok, this post became longer than I’d intended, the next one will look at what the IR-level optimizations do to this function.

(Thanks to Xi Wang and Alex Rosenberg for sending corrections.)
