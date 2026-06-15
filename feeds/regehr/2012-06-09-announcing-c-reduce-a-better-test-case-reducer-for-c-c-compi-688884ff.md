---
title: 'Announcing C-Reduce: A Better Test-Case Reducer for C/C++ Compiler Debugging'
url: https://blog.regehr.org/archives/697
published: "2012-06-09T16:42:40Z"
feed: regehr
guid: http://blog.regehr.org/?p=697
---

# Announcing C-Reduce: A Better Test-Case Reducer for C/C++ Compiler Debugging

*Test-case reduction* means taking a large input to a computer program (for compiler debugging, the input is itself a program) and turning it into a much smaller input that still triggers the bug. It is a very important part of the debugging process.

[Delta](http://delta.tigris.org/), an excellent open-source implementation of the [delta debugging](http://www.st.cs.uni-saarland.de/dd/) algorithm *ddmin*, has been the test-case reduction tool of choice for compiler developers. We’ve just released C-Reduce: a test case reducer that often gives output 25 times smaller than Delta’s. The contribution of C-Reduce is adding a lot of domain specificity. For example, it knows how to resolve a typedef, flatten a namespace, inline a function call, remove a dead argument from a function, and rename a variable. These (and dozens of other transformations) use Clang as a source-to-source transformation engine. Although domain specificity is displeasing in a reduction tool, C/C++ programs cannot be fully reduced without it. [This earlier blog post](https://blog.regehr.org/archives/696) contains many examples of C-Reduce’s output.

Resources:

- Download source from the [C-Reduce home page](http://embed.cs.utah.edu/creduce/).
- Learn how to use C-Reduce and see some examples [here](http://embed.cs.utah.edu/creduce/using/).
- See our [PLDI 2012 paper](http://www.cs.utah.edu/~regehr/papers/pldi12-preprint.pdf) for a more thorough explanation and evaluation of C-Reduce.

We’d be happy to hear feedback.
