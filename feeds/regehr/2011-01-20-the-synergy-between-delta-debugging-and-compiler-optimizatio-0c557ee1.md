---
title: The Synergy Between Delta Debugging and Compiler Optimization
url: https://blog.regehr.org/archives/356
published: "2011-01-20T22:12:51Z"
feed: regehr
guid: http://blog.regehr.org/?p=356
---

# The Synergy Between Delta Debugging and Compiler Optimization

Before reporting a compiler bug, it’s best to reduce the size of the failure-inducing input. For example, this morning I [reported an LLVM bug](http://llvm.org/bugs/show_bug.cgi?id=9013) where the compiler enters an infinite loop when compiling this C code:

> ```
> static int foo (int si1, int si2) {
>   return si1 - si2;
> }
> void bar (void) {
>   unsigned char l_197;
>   for (;; l_197 = foo (l_197, 1))
>     ;
> }
>
> ```

My bug report was a good one because the test program looks pretty close to minimal; it was reduced manually and automatically from a code that was originally 25 KB. Testcase reduction is so important that the GCC folks have an [entire document devoted to the topic](http://gcc.gnu.org/wiki/A_guide_to_testcase_reduction), including instructions for how to use the [Berkeley Delta implementation](http://delta.tigris.org/).

The Berkeley Delta is pretty awesome except when it is not. It often gets stuck at local minima that cannot be avoided except by making coordinated changes to multiple parts of a program. For example, it will never remove an argument from a function definition and also from all calls to the function. Therefore, a person works with delta like so:

1. Run delta
2. Do more reduction by hand
3. Goto step 1

After spending more hours on this kind of activity than I’d care to admit, I realized that most of the stuff I was doing resembled compiler optimizations:

- inlining a function
- deleting a dead function argument
- propagating a constant
- reducing the dimension of an array
- removing a level of indirection from a pointer

I think the similarity is a useful observation, but there are important differences between an optimizer and a testcase reducer:

1. An optimizer must be semantics-preserving, whereas a testcase reducer doesn’t need to be
2. An optimization is desirable when it speeds up the code, whereas a reduction step is desirable when it both simplifies the test case and also doesn’t make the bug disappear

Thus, to reuse an optimizer as a reducer, we would simply need to drive it in a typical Delta loop, being ready to back out any transformation which makes the bug go away or fails to make the test case simpler. I haven’t had time to do this; hopefully someone else will.
