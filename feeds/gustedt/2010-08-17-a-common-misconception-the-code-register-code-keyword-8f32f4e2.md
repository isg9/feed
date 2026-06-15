---
title: 'A common misconception: the <code>register</code> keyword'
url: https://gustedt.wordpress.com/2010/08/17/a-common-misconsception-the-register-keyword/
published: "2010-08-17T09:11:20Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=272
---

# A common misconception: the <code>register</code> keyword

Since probably its beginnings C has the `register` keyword and I recently read several times that *it has no effect* and would be ignored by the compiler. This is a misconception of what this keyword does and what it is meant to do. Unfortunately, this is just due to the misnoming of the feature as *register*. One of the [misnomers that we encounter in C](https://gustedt.wordpress.com/2010/08/18/misnomers-in-c/), others being `static`, `inline` and `const`.

The only purpose that `register` nowadays (and since long) has is that inhibits the taking of the address of a variable. I makes *e.g* the following code *invalid*:

```
register double a;
double *ap = &a;

```

The principal use of this is to serve as an optimization hint to the compiler:

> I know, I wouldn’t need an address of that variable, do all you can to optimize the access to it

In that sense it is of the same importance as the `restrict` keyword.

This can be particularly useful for array variables. An array variable is easily confounded with a pointer variable. Unless it is followed by a `[expr]` or with a `sizeof` it *evaluates* to the address of the first element. If you declare the array `register` all these uses are forbidden; we only access individual elements or ask for the total size. Such an `register`-array then may be much easier used as if it just were a set of variable by the optimizer. No aliasing (accessing the same variable through different pointers) may occur.

Another use for declaring a variable as `register` and `const` is to inhibit any non-local change of that variable, even trough taking its address and then casting the pointer. Even if you think that you yourself would never do this, once you pass a pointer (even with a `const` attribute) to some other function, you can never be sure that this might be malicious and change the variable under your feet. So in a setting with exposure to risks, you could declare all your variables as being `register` and then carefully inspect all the remaining places where you take addresses and pass stack pointers to other functions.

As said, unfortunately this purpose is not so easily deducible from its name \`register\`. Holding the variable in a register of the CPU is just one possible optimization that the compiler might find for such a variable, another one would be to just realize it as an immediate assembler operator. Maybe one day a compiler could enforce to hold such a variable in cache, and not to spill it out to higher levels of the memory hierarchy. Or maybe in many cases it just can’t do anything for you, perhaps it was worth to try?

*Edit:* `register` variables are also the technical base of a proposal for generic [named constants](https://gustedt.wordpress.com/2012/08/17/named-constants/) that I recently elaborated.
