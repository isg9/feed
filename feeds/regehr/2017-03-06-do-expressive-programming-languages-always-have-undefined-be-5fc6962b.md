---
title: Do Expressive Programming Languages Always Have Undefined Behavior?
url: https://blog.regehr.org/archives/1476
published: "2017-03-06T22:21:37Z"
feed: regehr
guid: http://blog.regehr.org/?p=1476
---

# Do Expressive Programming Languages Always Have Undefined Behavior?

In the [Hacker News comments](https://news.ycombinator.com/item?id=13648333) on [one of my previous posts about undefined behavior](https://blog.regehr.org/archives/1467), someone said this:

> AFAIK Gödel’s incompleteness theorems imply that \_any\_ language will have at least some undefined behaviour.

Let’s take a quick look at this statement, keeping in mind that incompleteness and undecidability can be remarkably tricky topics. Some years ago I read and enjoyed [Gödel’s Theorem: An Incomplete Guide to Its Use and Abuse](https://www.amazon.com/G%C3%B6dels-Theorem-Incomplete-Guide-Abuse/dp/1568812388/) (and no doubt need to reread it).

First off, it is clear that there exist programming languages that are free of UB, such as one where the semantics of every program is to print “7”. Whatever it is that UB means (we have not formally defined it), it seems clear that the language that always prints “7” does not have it.

There are also useful languages that are obviously UB-free, such as an expression language that evaluates elementary functions over IEEE floats. These languages are particularly easy to reason about because they are not Turing-complete: all computations terminate and we simply have to ensure that they terminate with a defined result.

In contrast, the HN commenter may have intended to invoke [Rice’s Theorem](https://en.wikipedia.org/wiki/Rice's_theorem): “Any nontrivial property about the language recognized by a Turing machine is undecidable.” A consequence is that when f() is some arbitrary computation we cannot in general hope to decide whether a program like this invokes undefined behavior:

```
main() {
  f();
  return 1 / 0; // UB!
}

```

But this is a red herring. Rice’s Theorem only applies to non-trivial properties: “properties that apply neither to no programs nor all programs in the language.” To sidestep it, we only need to define a programming language where UB-freedom is a trivial property. This is done by ensuring that every operation that a program can perform is a total function: it is defined in all circumstances. Thus, programs in this language will either terminate in some defined state or else fail to terminate. This kind of extremely tight specification is not typically done for realistic programming languages because it is a lot of work, particularly if the language has open-ended interactions with other levels of the system, such as inline assembly. But the problem is only a practical one; there is no problem in principle. It is not too difficult to write an UB-free specification for a (Turing-complete) toy language or subset of a real language.

Now let’s return to the original HN comment: it was about the incompleteness theorems, not about the halting problem. I’m not sure what to do with that, as I don’t see that Gödel’s theorems have any direct bearing on undefined behavior in programming languages.
