---
title: Some Goals for High-impact Verified Compiler Research
url: https://blog.regehr.org/archives/1565
published: "2018-01-11T07:04:12Z"
feed: regehr
guid: http://blog.regehr.org/?p=1565
---

# Some Goals for High-impact Verified Compiler Research

I believe that translation validation, a branch of formal methods, is just about ready for widespread use. Translation validation means proving that a particular execution of a compiler did the right thing, as opposed to proving once and for all that every execution of a compiler will do the right thing. These are very different. Consider some obstacles to once-and-for-all verification of a tool like GCC or LLVM where:

- Most of execution is spent processing pointer soup where correctness depends on poorly documented and incredibly detailed properties of the soup.

- Hundreds or thousands of analyses and transformations are performed, and due to performance constraints the compiler implementation entangles them in a way that is nearly impossible to disentangle.

- The implementation language is usually unsafe, forcing any formal verification effort to spend an outrageous amount of effort proving properties of the compiler code, such as memory safety, that are incidental to the task of interest. A convincing formal verifier for the subset of C++ that LLVM is written in doesn’t even exist.

- For some compiler algorithms like register allocation, it appears to be fundamentally easier to check the result than it is to prove that the right result is always computed. For example, CompCert uses this approach (or did the last time I looked).

- The compiler is under active, rapid development. Any proof would have to be redone, likely incurring significant effort, for every release.

So it’s clear that once-and-for-all formal verification of LLVM or GCC is never going to happen, the costs ludicrously outweigh the benefits. Translation validation, on the other hand, is already to some extent practical, see for example [this effort to prove that the seL4 object code refines its C source code](https://www.cl.cam.ac.uk/~mom22/pldi13.pdf). (Refinement just means that C gives a typical program many different meanings and we need to prove that the compiler has picked one of them.)

Other recent, LLVM-based work in this area includes [Program Analysis for Compiler Validation](https://llvm.org/pubs/2008-11-PASTE-CompilerValidation.pdf) (2008), [Evaluating Value-Graph Translation Validation for LLVM](https://dash.harvard.edu/bitstream/handle/1/4762396/pldi84-tristan.pdf) (2011), [Equality-Based Translation Validator for LLVM](https://cseweb.ucsd.edu/~lerner/papers/cav2011.pdf) (2011), [Formal Verification of SSA-Based Optimizations for LLVM](http://www.cis.upenn.edu/~stevez/papers/ZNMZ13.pdf) (2013), [An Extensible Verified Validator For Simple Optimizations in LLVM](http://sf.snu.ac.kr/gil.hur/publications/ROSAEC_2014_002.pdf) (2014), and [Black-box equivalence checking across compiler optimizations](http://www.cse.iitd.ac.in/~sbansal/pubs/aplas17.pdf) (2017).

This work is awesome but research tools don’t, by themselves, stop people from being burned by compiler bugs. One way to make things better is to combine translation validation with aggressive testing, [like we did here](https://blog.regehr.org/archives/1510), and then make sure any resulting bugs get fixed. Better yet, we can try to push a translation validation out into the world so that anyone can use it. It’s time for this to happen. The rest of this piece is some thoughts about how that should work.

## Goal 1: Ease of Use

The only thing an application developer should need to do is add a compiler flag like this:

```
clang++ -O -tv file.cpp

```

or:

```
rustc -O -tv file.rs

```

and then the compiler either validates, or fails to validate, its translation. It has to be this easy.

## Goal 2: Near zero overhead for compiler developers

Translation validation can’t get in the way of normal development for a production compiler: it has to be almost entirely on the side. This doesn’t mean, however, that the compiler can’t help out the validator, but rather that this has to happen in non-invasive ways. For example, certain optimizations on nested loops that are hard to validate might need to emit a bit of extra debug info or optimization remarks or whatever, to help the validator piece together what happened.

## Goal 3: Performance

Since translation validation will result in a lot of solver calls, it is going to be somewhat slow, probably well over an order of magnitude slower than regular compilation. A fairly easy way to speed it up would be to add a (persistent, networked) caching layer to exploit the fact that most parts of most code bases don’t change very often. We’ve had good luck using this kind of a cache for Souper, which is also slow due to making many solver calls.

## Goal 4: Multiple Validators

Research tends to move rapidly when there is a level playing field and a clearly-defined goal, allowing different groups to compete or cooperate, as they see fit. Competition can be particularly motivating, see for example [SMT-COMP](http://smtcomp.sourceforge.net/).

The primary metric for choosing a winner in a translation validation competition is the number of functions validated for compilation of a given benchmark using a particular LLVM version and optimization level. Verification time would be a good secondary metric.

To ensure a fair competition, it would be best for all validators to be using the same semantics for the source and target languages. This isn’t so straightforward: all too often these mathematical artifacts end up not being readable or reusable since they are deeply embedded in the implementation of a formal methods tool (this is unfortunately the case for [Alive](https://github.com/nunoplopes/alive), for example). A canonical, readable, writable, and reusable semantics for each of C, C++, Rust, Swift, LLVM IR, x86-64, etc. is something we should be spending significant resources on. [This sort of thing](https://alastairreid.github.io/alastairreid.github.io/ARM-v8a-xml-release/) is what I’m talking about.

## Conclusions

Just to be clear, beyond the Alive-based work referenced above, I’m not working on, nor do I have any plans to work on, translation validation. Rather, it is clearly the right way to gain confidence that a production-grade compiler has done its job. The technologies are in reach and we should be working to deploy them widely.
