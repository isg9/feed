---
title: Burning in a Module with Random Unit Testing
url: https://blog.regehr.org/archives/737
published: "2012-06-24T19:23:00Z"
feed: regehr
guid: http://blog.regehr.org/?p=737
---

# Burning in a Module with Random Unit Testing

Sometimes a class or subsystem makes us uneasy; when something goes wrong in our software, we’ll immediately suspect the shady module is somehow involved. Often this code needs to be scrapped or at least refactored, but other times it’s just immature and needs to be burned in. Randomized unit testing can help with this burn-in process, increasing our confidence in a module.

This post isn’t about creating a random unit tester. Sometimes you can reuse a tester such as [Quickcheck](http://www.haskell.org/haskellwiki/Introduction_to_QuickCheck2) or [Randoop](http://code.google.com/p/randoop/). If not, it’s easy to create one by hand; I’ll cover that in a separate post.

The point is to increase our confidence in a module. But what does that mean? I’ll try to clarify with an example. Let’s say I implement a new data structure, perhaps a B-tree. Even after doing some unit testing, I probably won’t be particularly confident in its correctness. The question is: Under what conditions can I use random testing to gain as much confidence in my new B-tree as I would have in a balanced search tree from the C++ STL? For that matter, how confident am I in a random data structure from the STL in the first place? I would say “moderately confident.” That is, in general I would be happy to just use STL code in my program without specifically unit-testing it first. On the other hand, I would not just reuse the STL if I were developing safety-critical software.

Almost every time I write a software module that has a clean API and that might be reused, I write a random tester for it. Over the years I’ve developed a sort of informal procedure that, if successful, results in a burned-in module that I’m fairly confident about. Here are the necessary conditions.

- Understandable code, clean API — I have to be able to understand the entire module at once. If not, it needs to be broken into units that are tested separately. The module can’t contain spaghetti logic or make use of extraneous state. If it does, it needs to be refactored or rewritten before burning in.
- Heavy use of assertions — Every precondition and postcondition that can be checked, is. A repOk() / [checkRep()](http://www.pgbovine.net/programming-with-rep-invariants.htm) method exists that does strong invariant checking. During fuzzing it is invoked after every regular API call.
- Mature fuzzer for the module’s API — The random unit tester for the module is strong: it has been iteratively improved to the point where its tests reach into all parts of the module’s logic.
- Fault injection — APIs used (as opposed to provided) by the module, such as system calls, have been tested using mocks that inject all error conditions that might happen in practice.
- Good coverage — The maturity of the fuzzer is demonstrated by 100% coverage of branches that I believe should be covered. This includes error checking branches, but of course does not include assertion failure branches. At the system testing level, 100% coverage is generally impossible, but at the unit level I consider it to be mandatory. Coverage failures indicate bad code, a bad API, or a bad random tester.
- Separate validation of code that is sneaky with respect to coverage — Branch coverage is, in some cases, a very weak criterion. Examples include complex conditionals and code that uses lookup tables. These have to be separately validated.
- Checkers are happy — Valgrind, IOC, `gcc -Wall`, pylint, or whatever tools apply to the code in question are happy, at least up to the point of diminishing returns.
- Oracles are happy — If a strong oracle is available, such as an alternative implementation of the same API, then it has been used during random testing and no important differences in output have been found.

You could argue that this list has little to do with random testing, but I’d disagree. I seldom if ever trust a software module unless it has been subjected to a broad variety of inputs, and it can be very hard to get these diverse inputs without using random numbers somehow. A haphazard (or even highly systematic) collection of unit tests written by myself or some other developer does not accomplish this, for code of any complexity.

A lot of real-world software, particularly in web-land, seems to be burned in by deploying it and watching the error logs and mailing lists. This development style has its place, especially since there’s a lot of software that’s just not easy to unit test. Test via deployment is how most of my group’s open-source projects work, in fact. But that doesn’t mean that it’s not satisfying and useful to be able to produce a piece of high-quality software the first time.

Might it be possible to bypass the burn-in process, for example using formal verification? Absolutely not, though we would hope that verified software contains fewer errors. Also, the errors found will tend to have a different character. The relationship between software testing and verification is a tricky issue that will increase in importance over the next few decades.

This post would be incomplete without mentioning that people with a background in academic software engineering sometimes claim that random testing can improve our confidence in software in a totally different sense from what I’m talking about here. In that line of thinking, you create an operational profile for what real inputs look like, you generate random test cases that “look like” inputs in the profile, and then finally you devise a statistical argument that gives a lower bound for the reliability of the software. I don’t happen to believe that this kind of argument is useful very often.
