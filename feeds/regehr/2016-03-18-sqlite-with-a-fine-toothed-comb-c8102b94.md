---
title: SQLite with a Fine-Toothed Comb
url: https://blog.regehr.org/archives/1292
published: "2016-03-18T16:12:40Z"
feed: regehr
guid: http://blog.regehr.org/?p=1292
---

# SQLite with a Fine-Toothed Comb

One of the things I’ve been doing at Trust-in-Soft is looking for defects in open-source software. The program I’ve spent the most time with is SQLite: an extremely widely deployed lightweight database. At ~113 KSLOC of pointer-intensive code, SQLite is too big for easy static verification. On the other hand, it has an [incredibly impressive test suite](https://www.sqlite.org/testing.html) that has already been used to drive dynamic tools such as Valgrind, ASan, and UBSan. I’ve been using these tests with [tis-interpreter](http://trust-in-soft.com/tis-interpreter/).

Here I try to depict the various categories of undefined behaviors (UBs) where this post is mainly about the blue-shaded area:

![](https://blog.regehr.org/extra_files/tis-interpreter.png)

This figure leaves out many UBs (there are a few hundred in all) and is not to scale.

For honesty’s sake I should add that using tis-interpreter isn’t painless (it wasn’t designed as a from-scratch C interpreter, but rather is an adaptation of a sound formal methods tool). It is slower even than Valgrind. It has trouble with separately compiled libraries and with code that interacts with the system, such as mmap() calls. As I tried to show in the figure above, when run on code that is clean with respect to ASan and UBSan, it tends to find fiddly problems that a lot of people aren’t going to want to hear about. In any case, one of the reasons that I’m pushing SQLite through tis-interpreter is to help us find and fix the pain points.

Next let’s look at some bugs. I’ve been reporting issues as I go, and a number of things that I’ve reported have already been fixed in SQLite. I’ll also discuss how these bugs relate to the idea of a [Friendly C dialect](https://blog.regehr.org/archives/1180).

## Values of Dangling Pointers

SQLite likes to use — but not dereference — pointers to heap blocks that have been freed. It did this at quite a few locations. For example, at a time when zHdr is dangling:

```
if( (zHdr>=zEndHdr && (zHdr>zEndHdr
  || offset64!=pC->payloadSize))
 || (offset64 > pC->payloadSize)
){
  rc = SQLITE_CORRUPT_BKPT;
  goto abort_due_to_error;
}

```

These uses are undefined behavior, but are they compiled harmfully by current C compilers? I have no evidence that they are, but in other situations the compiler will take advantage of the fact that a pointer is dangling; see [this post](http://trust-in-soft.com/dangling-pointer-indeterminate/) and also [Winner #2 here](https://blog.regehr.org/archives/767). You play with dangling pointers at your own risk. Valgrind and ASan make no attempt to catch these uses, as far as I know.

Using the value of a dangling pointer is a nettlesome UB, causing inconvenience while — as far as I can tell — giving the optimizer almost no added power in realistic situations. Eliminating it is a no-brainer for Friendly C.

## Uses of Uninitialized Storage

I found several reads of uninitialized storage. This is [somewhere between unspecified and undefined behavior](http://blog.frama-c.com/index.php?post/2013/03/13/indeterminate-undefined).

One idiom was something like this:

```
int dummy;
some sort of loop {
  ...
  // we don't care about function()'s return value
  // (but its other callers might)
  dummy += function();
  ...
}
// dummy is not used again

```

Here the intent is to avoid a compiler warning about an ignored return value. Of course a better alternative is to initialize dummy; the compiler can still optimize away the unwanted bookkeeping if function() is inlined or otherwise specialized.

At least one uninitialized read that we found was potentially harmful, though we couldn’t make it behave unpredictably. Also, it had not been found by Valgrind. Both of these facts — the predictability and the lack of an alarm — might be explained by the compiler reusing a stack slot that had previously contained a different local variable. Of course we must not count on such coincidences working out well.

A Friendly C could ignore reads of uninitialized storage based on the idea that tool support for detecting this class of error is good enough. This is the solution I would advocate. A heavier-handed alternative would be compiler-enforced zeroing of heap blocks and automatic variables.

## Out-of-Bounds Pointers

In C you are not allowed to compute — much less use or dereference — a pointer that isn’t inside an object or one element past its end.

SQLite’s vdbe struct has a member called aMem that uses 1-based array indexing. To avoid wasting an element, this array is initialized like this:

```
p->aMem = allocSpace(...);
p->aMem--;

```

I’ve elided a bunch of code, the full version is in sqlite3VdbeMakeReady() in [this file](http://www.sqlite.org/cgi/src/artifact/2c15cf88de4df974). The real situation is more complicated since allocSpace() isn’t just a wrapper for malloc(): UB only happens when aMem points to the beginning of a block returned by malloc(). This could be fixed by avoiding the 1-based addressing, by allocating a zero element that would never be used, or by reordering the struct fields.

Other out-of-bounds pointers in SQLite were computed when pointers into character arrays went past the end. This particular class of UB is commonly seen even in hardened C code. It is probably only a problem when either the pointer goes far out of bounds or else when the object in question is allocated near the end of the address space. In these cases, the OOB pointer can wrap, causing a bounds check to fail to trigger, potentially causing a security problem. The full situation [involves an undesirable UB-based optimization and is pretty interesting](https://lwn.net/Articles/278137/). A good solution, as the LWN article suggests, is to move the bounds checks into the integer domain. The Friendly C solution would be to legitimize creation of, and comparison between, OOB pointers. Alternately, we could beef up the sanitizers to complain about these things.

## Illegal Arguments

It is UB to call memset(), memcpy(), and other library functions with invalid or null pointer arguments. [GCC actively exploits this UB to optimize code](https://goo.gl/h7K89h). SQLite had a place where it called memset() with an invalid pointer and another calling memcpy() with a null pointer. In both cases the length argument was zero, so the calls were otherwise harmless. A Friendly C dialect would only require each pointer to refer to as much storage as the length argument implies is necessary, including none at all.

## Comparisons of Pointers to Unrelated Objects

When the relational operators >, >=, <, <= are used to compare two pointers, the program has undefined behavior unless both pointers refer to the same object, or to the location one past its end. SQLite, like many other programs, wants to do these kinds of comparisons. The solution is to reduce undefined behavior to implementation defined behavior by casting to uintptr\_t and comparing the resulting integers. For example, SQLite now has this method for checking if a pointer (that might be to another object) lies within a memory range:

```
# define SQLITE_WITHIN(P,S,E) \
    ((uintptr_t)(P)>=(uintptr_t)(S) && \
     (uintptr_t)(P)<(uintptr_t)(E))

```

Uses of this macro can be found in [btree.c](http://www.sqlite.org/cgi/src/artifact/6eee126fe9d1f571).

With respect to pointer comparisons, tis-interpreter’s (and Frama-C’s) intentions are stronger than just detecting undefined behavior: we want to ensure that execution is deterministic. Comparisons between unrelated objects destroy determinism because the allocator makes no guarantees about their relative locations. On the other hand, if the pair of comparisons in a SQLITE\_WITHIN call is treated atomically, and if S and E point into the same object, there is no determinism violation. We added a “within” builtin to Frama-C that can be used without violating determinism and also a bare pointer comparison that — if used — breaks the guarantee that Frama-C’s results hold for all possible allocation orders. Sound analysis of programs that depend on the relative locations of different allocated objects is a research problem and something like a model checker would be required.

In Friendly C, comparisons of pointers to unrelated objects would act as if they had already been cast to uintptr\_t. I don’t think this ties the hands of the optimizer at all.

## Summary

SQLite is a carefully engineered and thoroughly tested piece of software. Even so, it contains undefined behaviors because, until recently, no good checker for these behaviors existed. If anything is going to save us from UB hell, it’s tools combined with developers who care to listen to them. Richard Hipp is a programming hero who always responds quickly and has been receptive to the fiddly kinds of problems I’m talking about here.

## What to Do Now

Although the situation is much, much better than it was five years ago, C and C++ developers will not be on solid ground until every undefined behavior falls into one of these two categories:

- Erroneous UBs for which reliable and ubiquitous (but perhaps optional and inefficient) detectors exist. These can work either at compile time or run time.

- Benign behaviors for which compiler developers have provided a documented semantics.
