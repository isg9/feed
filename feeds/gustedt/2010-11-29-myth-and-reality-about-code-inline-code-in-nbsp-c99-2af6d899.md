---
title: Myth and reality about <code>inline</code> in&nbsp;C99
url: https://gustedt.wordpress.com/2010/11/29/myth-and-reality-about-inline-in-c99/
published: "2010-11-29T17:32:37Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=616
---

# Myth and reality about <code>inline</code> in&nbsp;C99

There is a very common misunderstanding about the `inline` keyword in C99. Rumors go that this would just be a useless hint to the compiler, that generation of external symbols or not is left to the compiler implementor, that this is some bizarre take over from C++ or whatever.

None of that is completely correct.

So let us first dress a list of corrections:

- Yes, inline is a [misnomer](https://gustedt.wordpress.com/2010/08/18/misnomers-in-c/).
- Yes, `inline` comes from C++. But this is not a bad thing as such.
- No, the use of `inline` does change the emitted code, so it is not useless.
- Yes, the wording for that feature in the standard is one of the obscure, dark corners of the language. In fact, it is merely a question of mastering the language in which the standard itself is written, English.
- No, C99 does not leave the decision of emitting an external symbol of an `inline` function to the compiler implementor.
- Yes, C99 got right what was wrong in C++. It makes no assumption on where and if an external symbol for an `inline` function is emitted and if gives you a syntax to explicitly require emission in a compilation unit of your liking.

So let’s go through some of these points to see what `inline` is good for and how we can use it with a C99 conforming compiler.

`Inline`, from its naming, is thought to be a hint to the compiler that it might be a good idea to integrate a function into the place where it is called. Often, if the compiler succeeds in doing that this can result in code that is better optimized: the overhead of a call is avoided, register spill is reduced and generally register usage can be streamlined.

So generally it is a good idea to let the compiler decide whether or not to inline a function. To do so, the compiler must at least have seen a definition of the function. But if you have a classical big C project with functions that are implemented in different compilation units the compiler can’t do that integration for you. So you’d have to put the implementation of such candidate functions in header files.

Historical C, aka C89, only had one way to do this and such that you don’t create conflicts between the different compilation units: `static` functions. `Static` functions are functions that are only visible inside the corresponding compilation unit. If you declare a function static in a header file, each compilation unit might have its own copy of the function without creating a conflict between the different copies. Thus the first goal is achieved, the compiler sees the definition of the function and may (or may not) place it where the function is actually called.

The disadvantages of this approach are twofold:

- In large projects, there may be many copies of the same function that blow up the object size. The compiler is not forced to implement such a function in each unit, but it might, e.g when debugging options are switch on.
- It is impossible to compare pointers to such a function since you will never be sure which of the different version you will get.

Came the idea of C++ with the extra keyword `inline` to replace `static` for that case. The idea is to

- have the definition of a function in a header file, visible to the whole project
- have only one external symbol generated, if the function can’t be inlined at some place

How to achieve this was left to the implementation of the compiler. Early version of C++ followed different, incompatible models on when and where to emit the function for the external symbol, if any.

Historically, many people probably got wrong ideas about `inline` because one important compiler, namely gcc, got it wrong when first proposing it as an extension to C89. In their version, the emitted symbol was generated from an independent, second, definition. This was perhaps thought to be more flexible, but it unwillingly had really bad impact on code where the two definitions were not consistent. If you think of having two different implementations of a function in two different files you can easily imagine that this was just difficult to handle and much error prone.

C99 adopted this new keyword and its purpose, but changed this ambiguity about the place where the external symbol should live and what definition should be taken for the external. The simplest version for C99 is to do the following in a header (.h) file

```
inline
double dabs(double x) {
  return x < 0.0 ? -x : x;
}

```

And then in exactly one compilation unit (.c file)

```
extern inline double dabs(double x);

```

The later just ensures that an external symbol `dabs` is emitted in that unit and that the definition that was seen in the header file is used. Simple and clean as that.

Nowadays, where the C99 standard still isn’t completely implemented in many compilers, many of them still get it wrong. Gcc corrected its first approach and now (since version 4.2.1) works with the method above. Others that mimic gcc like opencc or icc are still stuck on the old model that gcc had once invented. The include file [p99\_compiler.h](http://p99.gforge.inria.fr/p99-html/p99__compiler_8h.html) of [P99](http://p99.gforge.inria.fr/) helps you make sure that your code will work even with these compilers.
