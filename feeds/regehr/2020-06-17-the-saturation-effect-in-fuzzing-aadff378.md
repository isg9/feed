---
title: The Saturation Effect in Fuzzing
url: https://blog.regehr.org/archives/1796
published: "2020-06-17T16:17:48Z"
feed: regehr
guid: https://blog.regehr.org/?p=1796
---

# The Saturation Effect in Fuzzing

*This piece is co-authored by [Alex Groce](https://agroce.github.io/) and John Regehr.*

Here’s something we’ve seen happen many times:

- We apply a fuzzer to some non-trivial system under test (SUT), and initially it finds a lot of bugs.
- As these bugs are fixed, the SUT sort of becomes immune to this fuzzer: the number of new bugs found by the fuzzer drops off, eventually approaching zero.
- Subsequently, a different fuzzer, applied to the same system, finds a lot of bugs.

What is going on with the first fuzzer? Two things are happening. First, the frequency at which the bugs in a system are triggered by any given source of stochastic inputs follows a [power law distribution](https://en.wikipedia.org/wiki/Power_law). As the often-triggered bugs get discovered and fixed, we’re eventually left with only bugs that are triggered at such low probability that they are effectively not findable within a reasonable compute budget. Second, some bugs are out of reach for a given fuzzer even given an infinite amount of CPU time. For example, if we fuzz GCC using Csmith, we will never find a bug in GCC’s optimization of string compares because Csmith is incapable of generating code that compares strings. Similarly, a fuzzer for JPEG compressors that only constructs images that are 1000×1000 pixels, all the same random color, is unlikely to be capable of discovering most bugs. These examples are basic, but almost every fuzzer, no matter how sophisticated, has analogous limitations.

The second problem is usually the preferable one to have: we can fix it by adding new features to a generation-based fuzzer, or by finding a better set of seeds for a mutation-based fuzzer. Even if we don’t know anything about the as-yet-undiscovered bugs, we can often infer that they might exist by noticing a gap in coverage induced by the fuzzer.

The first problem is often much harder to work around because we typically do not have any real understanding of how the space of test cases that triggers a particular bug intersects with the probability distribution that is implicit in our fuzzing tool. We can tweak and tune the probabilities in the fuzzer in order to change the distribution, but without solid guidance this is often no better than guesswork.

## Detecting Saturation

There isn’t much research on fuzzer saturation. In principle, coverage-based stopping criteria aim at this, but most of these simply set some moderately-difficult target and assume that if you reach it, you are done. Since most fuzzing efforts will never hit an arbitrary, high code coverage target for any complex SUT, this is not useful. The one effort we are aware of to propose something more relevant for typical saturation is by [Tramontana et al](https://dl.acm.org/doi/10.1145/3339836). They propose splitting a testing effort into k subsets, each with its own coverage measure, and stopping when all k subsets have identical coverage. The larger k is, the longer it will take to detect saturation, but the more confident you can be that testing is “done.” The stopping point does not need to be in terms of coverage; it could instead use automated bug triage to see if all runs have found identical sets of bugs.

It is possible to have two different kinds of saturation interacting. The most important kind of saturation is saturation in terms of bug-finding, of course: a particular way of generating test cases has hit the point at which producing new bugs is highly unlikely. However, we don’t just look at bug-finding in testing; we also look at coverage, and use coverage as a signal of progress in testing. Critically, it’s not just humans who do this; fuzzers use novel coverage of some kind (CFG edges, paths, or even dataflow) to guide their behavior. AFL attempts to cover new paths as its primary goal, with new crashes as a kind of side-effect. This means, however, that there can be four cases for saturation, in practice:

1. *Coverage is not saturated, and bugs are not saturated.* This is the case where you don’t need to think about saturation. Your testing process is generating new coverage and new bugs frequently; just continue fuzzing.
2. *Coverage is saturated, and bugs are saturated.* The opposite case is also straightforward: if neither new coverage nor new bugs are being generated, it’s time for a change, such as improving the fuzzer or perhaps finding a different SUT to target.
3. *Coverage is saturated, but bugs are not saturated.* This is an interesting case, where the coverage metric being used is insufficiently expressive. In other words, coverage stops increasing not because no further interesting exploration of the SUT is possible, but rather because coverage is poorly correlated with the kind of execution space exploration that leads to new bugs. It is pretty easy to get into this situation, for example if the notion of coverage is weak (statement coverage) and bugs tend to depend on hitting particular paths or states, rather than just hitting bad code locations. This can be a problem for automated termination of fuzzing efforts, either because the fuzzer appears to be “done” but is in fact still generating good results, or (more commonly) when AFL/libFuzzer is stuck due to the coverage signal not helping it reach another part of the input space (where coverage might no longer be saturated). The path-dependency of AFL/libFuzzer means that just starting again with the same corpus, or throwing in some new seeds from another fuzzer, might help escape this particular trap.
4. *Coverage is not saturated, but bugs are saturated*. This is a real possibility and a real pain in coverage-driven fuzzing. When using AFL to fuzz a compiler, it is not uncommon to see a run get stuck generating huge numbers of uninteresting paths, presumably exploring things like “ways to have the parser fail,” that seldom produce interesting bugs. Recent research on specifying parts of code that aren’t to be counted for coverage can help with this problem. In our experience fuzzing real-world compilers with AFL, the “pending:” field often never goes to zero, even if nothing interesting is happening, and fuzzing has been running for days or weeks.

## Saturation in Practice: Fuzzing a Smart Contract Compiler

Saturation isn’t a one-time thing in a real fuzzing effort, but an ebb-and-flow responding to either new code in the system under test or changes in the fuzzing regime. For example, here is a chart showing the number of GitHub issues (almost all actual bugs, with a small number of duplicates of already-known bugs) we submitted for the solc Solidity smart contract compiler, as part of [an ongoing research effort to make fuzzing compilers with AFL easier](https://github.com/agroce/afl-compiler-fuzzer). Here’s some [more detail about this fuzzing effort](https://blog.trailofbits.com/2020/06/05/breaking-the-solidity-compiler-with-a-fuzzer/).

![](https://lh4.googleusercontent.com/B6BAnjEuD29tI8jpJ16v-6WPt3JhGC6SUNODFSt8NVFA5nSIYRPEPvYeMteeXQEYj7VSxy8L4oqwa1YaDPJRS_mDFNTcRQJ8bCCdZFJaS-kNVIsSunMmlLbFomZJ3H8D30pmJlKE)

We started fuzzing the compiler in early February, and immediately found a few bugs, using both our new technique and plain-old-AFL.  Then we continued running the fuzzer, but found nothing novel for a few days. On February 21, we added several new mutators to our fuzzing engine (see e.g. [this commit](https://github.com/agroce/afl-compiler-fuzzer/commit/98393a86eaef32fc0b5e14b9ce9daebb4d885f76)), and “broke” the saturation.  Then there was a long gap, with a few more bugs finally found after running the fuzzer for over a month, performing well over 1 billion compilations, on a corpus consisting of *every interesting path* found by any previous run.

Just recently, however, we made another change.  Previously, we had based our corpus only on the Solidity compiler’s syntax tests; we switched to using *all* the Solidity files in the entire test subtree of the repository on May 14th. This turned up a number of new bugs, for example in the SMTChecker (which previously wasn’t being fuzzed at all). We’ll probably return to saturation soon, though! Note that during this time, no fuzzer instance (and we’ve run plenty of them) has ever saturated path coverage in AFL.

A [just-accepted FSE paper by Bohme and Falk](https://mboehme.github.io/paper/FSE20.EmpiricalLaw.pdf) gives much more explicit empirical data on the nature of saturation barriers: finding linearly more instances of a given set of bugs requires linearly more computation power directed towards fuzzing a target; finding linearly more distinct bugs requires exponentially more fuzzing compute power.

## Mitigating Saturation

Beyond understanding and detecting saturation, of course we would like to be able to delay its onset. That is, we want to find more bugs before a given fuzzer saturates. This is a relatively natural and easy activity for mutation-based fuzzers because we can not only cast a wider net when finding seeds, but we can also add new mutation operators in a relatively painless way because mutations should be independent of each other.

Fuzzers that generate code from scratch are a trickier case. Perhaps the closest thing to a free lunch that we are aware of is [swarm testing](http://www.cs.utah.edu/~regehr/papers/swarm12.pdf), which randomly reconfigures the fuzzer prior to generating a new test case. The goal is to avoid the problem where the products of a large number of random choices all end up looking more or less the same. (A low-dimensional example is a collection of images, each of which is created by randomly assigning each pixel to be black or white with 50% probability. These are not easy to tell apart.)

Our other options are more work:

- Improve the stuck fuzzer, for example by teaching it to generate more features supported by the SUT.
- Run a collection of fuzzers, if these are available.
- Do a better job at integrating feedback from the SUT.

This last option is probably the most promising one: the state of the art in merging generative fuzzing with feedback-driven fuzzing is, at present, not very sophisticated. We believe there is room for major improvements here.
