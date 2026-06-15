---
title: Parallelizing Delta Debugging
url: https://blog.regehr.org/archives/749
published: "2012-07-11T13:12:05Z"
feed: regehr
guid: http://blog.regehr.org/?p=749
---

# Parallelizing Delta Debugging

Delta debugging is a technique for taking an input to a computer program that causes the program to display a certain behavior (such as crashing) and automatically creating a smaller input that triggers the same behavior. For complex programs that typically process large inputs (compilers, web browsers, etc.) delta debugging is an important part of the overall debugging workflow.

A key invariant in a delta debugger is that the current version of the input is always “interesting”—it triggers the bug or whatever. Delta speculatively transforms this input into a smaller input and then tests if this new input is still interesting. If so, it becomes the current input. Otherwise, it is discarded and a different transformation is tried. Termination occurs when no transformation can make the current input smaller. The greedy search looks something like this:

![](https://blog.regehr.org/wp-content/uploads/2012/07/Slide1b.png)

Here the original input (version 1) was transformed four times. Three of the resulting inputs failed to have the property of interest but the fourth was interesting, giving version 2. Version 2 was successfully transformed the first time, giving version 3 which, since it cannot be transformed any more, constitutes the delta debugger’s output.

While the [original formulation](http://www.st.cs.uni-saarland.de/papers/tse2002/) of delta debugging uses only a single kind of transformation (removing a contiguous chunk of the input), we found that this didn’t produce nicely reduced C programs. My group’s tool, [C-Reduce](http://embed.cs.utah.edu/creduce/), is parameterized by a collection of pluggable transformations that, together, work better for this specific task.

In many cases, a run of a delta debugger takes only a few minutes, which is fast enough that we probably have no motivation to make it faster. On the other hand, we have found that reducing multi-megabyte C++ programs that trigger compiler bugs can be quite slow, on the order of a couple of days in some of the worst cases we’ve seen. Delta debugging is slow when the test for interestingness is slow and also when the greedy search fails to make rapid progress during its initial phases. Both of these tend to be the case for large C++ programs; rapid initial progress appears to be difficult due to the complex and dense dependency web in typical C++ code.

The other day I decided to parallelize C-Reduce for shared-memory machines. My first idea was to relax the delta debugging invariant so that each processor could have its own current version of the input. This requires a way to merge results so that processors can take advantage of each others’ progress. It would work something like this:

![](https://blog.regehr.org/wp-content/uploads/2012/07/Slide2b.png)

Here the code running on Core 2 has decided that instead of performing a regular C-Reduce transformation in version 3, it is rather going to merge version 3 with Core 1’s current version, which happens to be 2. Of course the result may not be interesting, but this diagram assumes that it is.

A problem is that existing utilities like “merge” are quite conservative; they aggressively signal conflicts. Also, it seems hard to know how often to merge. After a bit of experimentation I decided that this wasn’t the way to go. My next idea was to parallelize only the part of C-Reduce that performs line-based deletion of chunks of a file. If we rig C-Reduce so that instead of deleting lines it replaces them with blank lines, then merging becomes trivial. I think this was a reasonable idea but I stopped working on it after thinking of a simpler way.

The strategy used by the current (development) version of C-Reduce maintains the delta debugging invariant that we always have a global best result. Parallelization happens by speculatively transforming the current input and then forking off subprocesses to test these inputs for interestingness. The strategy for speculation is to assume that no transformed input is interesting; I chose this heuristic because, in practice, transformations produce uninteresting results more often than not. In this illustration, two speculative processes have been forked but neither has completed:

![](https://blog.regehr.org/wp-content/uploads/2012/07/Slide3.png)

As interestingness tests terminate, more are forked:

![](https://blog.regehr.org/wp-content/uploads/2012/07/Slide4.png)

As soon as any transformation succeeds in producing an interesting result, all remaining speculative processes are killed off and a new batch is forked:

![](https://blog.regehr.org/wp-content/uploads/2012/07/Slide5.png)

Since this kind of parallelization does not break the delta debugging invariant, not much of C-Reduce needed to be modified to support it. Although there are many improvements still to implement, performance seems decent. Here’s a graph showing reduction time vs. degree of parallelism on a Core i7-2600 with hyperthreading disabled for a 165 KB Csmith program that triggers [this LLVM bug](http://llvm.org/bugs/show_bug.cgi?id=13326):

![](https://blog.regehr.org/wp-content/uploads/2012/07/foo.png)

Why does performance get worse when C-Reduce uses all four cores? My guess is that some resource such as L3 cache or main memory bandwidth becomes over-committed at that point. It could also be the case that the extra speculation is simply not worth it.

Here’s what happens when we reduce a big C++ program that crashes a commercial compiler using an i7-2600 with hyperthreading turnred on:

![](https://blog.regehr.org/wp-content/uploads/2012/07/foo1.png)

When the degree of parallelism exceeds five, performance falls even more off a cliff. For example, when C-Reduce tries to use six CPUs the reduction takes more than seven hours. This graph seems to tell the same story as the previous one: parallelism can help but some resource saturates well before all (virtual) processors are used.

I don’t yet know how C-Reduce will perform on a serious machine, but we have some new 32-core boxes that we’ll experiment with as we get time.

Perhaps my favorite thing about this little experiment with parallelization is that it leverages the purely functional nature of C-Reduce’s API for pluggable transformations. Previous versions of this API had been stateful, which would have made it hard to deal with speculation failures. I’m not particularly a fan of functional programming languages, but it’s pretty obvious that a functional programming style is one of the keys to making parallel programming work.

I decided to write this up since as far as I know, nobody has written a parallelized test case reducer before.
