---
title: Finding Compiler Bugs by Removing Dead Code
url: https://blog.regehr.org/archives/1161
published: "2014-06-20T19:36:46Z"
feed: regehr
guid: http://blog.regehr.org/?p=1161
---

# Finding Compiler Bugs by Removing Dead Code

I was pretty bummed to miss [PLDI](http://conferences.inf.ed.ac.uk/pldi2014/program.html) this year, it has been my favorite conference recently. One of the talks I was most interested in seeing was [Compiler Validation via Equivalence Modulo Inputs](http://www.cs.ucdavis.edu/~su/publications/emi.pdf) by some folks at UC Davis. Although I had been aware of this paper (which I’ll call “the EMI paper” from now on) for a while, I was hesitant to write this post — the work is so close to my work that I can’t avoid having a biased view. So anyway, keep that in mind.

One of the things that makes testing hard is the necessity of oracles. An oracle tells us if a run of the software being tested was buggy or not. A cute idea that no doubt has been around for a long time, but which has more recently been called [metamorphic testing](http://en.wikipedia.org/wiki/Metamorphic_testing), is to work around the oracle problem by taking an existing test case and changing it into a new test case whose output can be predicted. For example, if I have a test case (and expected answer) for a program that classifies a triangle as isosceles/scalene/right/etc., I can scale, translate, and rotate the triangle in my test case in order to create many new test cases that all have the same answer as the original one.

So how should one apply metamorphic testing to compilers? It’s not very hard to come up with bad ideas such as adding layers of parens, rewriting (x+y) to be (y+x), rewriting x to be (x+0), etc. The reason that these are bad ideas is that the changes will be trivially undone by the optimizer, resulting in poor testing of the optimizer logic. Some better ideas can be found in [this paper on metamorphic compiler testing](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=5693203) (IEEE paywall, sorry) which found a few GCC bugs.

The EMI paper is based on a particularly clever application of metamorphic testing where the program transformation is removal of dead code. Of course compilers know how to remove dead code, so how is this transformation any better than the bad ones I already mentioned? The idea is to remove dynamically dead code: code that is dead in some particular run. This kind of code is easy to detect using a code coverage tool. Of course this code may not be dead in other executions of the program, but this is fine: we’ll just need to be careful not to test the compiled program on anything other than the input that was originally used to do dead code discovery. So basically EMI works like this:

1. Run a C program on whatever input we have sitting around, using compiler instrumentation to check which lines are executed.

2. Create a new C program lacking randomly chosen pieces of code that did not execute in Step 1.

3. Run the new program on the same input. Report a compiler bug if its output has changed.

Notice that the original C program had better not execute undefined behavior or rely on unspecified behavior or else we’ll get false positives.

The cleverness of EMI is not abstract or conceptual. Rather, EMI is clever because it works: at the time the paper was finalized the authors had reported 147 compiler bugs that were confirmed by developers, 110 of which have been fixed. This last number — fixed bugs — is the impressive one, since finding bugs that people care about enough to fix is generally a lot harder than just finding bugs.

The great thing about EMI is that it is a simple and extensible process. For example, it would not be hard to adapt the idea to C++. In contrast, random generation of a meaningful subset of C++11 is a project that we have been reluctant to start because we don’t yet know how to build this generator at an acceptable level of effort. Another easy extension to EMI would be adding or modifying dead code rather than simply deleting it. More generally, metamorphic compiler testing is probably an underused idea.

I was interested to read that the vast majority (all but four, it looks like) of the bugs discovered by EMI were triggered by mutated versions of Csmith programs. One reason that this is interesting is that since Csmith programs are “closed” — they take no inputs — the statically and dynamically dead code in such a program is precisely the same code. Therefore, an apparent advantage of using dynamic information — that it can remove code that is not dead in all executions — turns out to be a bit of a red herring. EMI works in this situation because the dead code elimination passes in compilers are not very precise.

An interesting question is: Why is Csmith+EMI so effective? One reason is that Csmith programs tend to contain a large amount of dead code, giving EMI a lot of room to play. It is just hard to generate expressive random code (containing lots of conditionals, etc.) that isn’t mostly dead, as far as we can tell. We’ve known this for a while and basically we don’t care — but we never guessed that it would turn out to be a hidden advantage.

Another problem with using EMI to mutate non-Csmith programs is that many real C programs execute undefined behaviors and even when they do not, it is generally difficult to verify that fact. Csmith, in contrast, has been somewhat co-designed with [Frama-C](http://frama-c.com/) such that the two tools work together with no additional effort. Automated undefined behavior detection is a crucial part of doing automated test-case reduction using C-Reduce.

One might ask: How useful is EMI when applied to C++ given that there is no C++smith? I look forward to learning the answer. The lack of a robust undefined behavior checker for C++ is another problem, although projects like LLVM’s UBsan are slowly chipping away at this.

The EMI authors say “… the majority of \[Csmith’s\] reported bugs were compiler crashes as it is difficult to steer its random program generation to specifically exercise a compiler’s most critical components—its optimization phases.” This doesn’t follow. The actual situation is subtle, but keep in mind that the entire purpose of Csmith is to exercise the compiler’s optimization phases. We spent years working on making Csmith good at this exact thing. We did in fact report more crash bugs than wrong code bugs but the real reasons are (1) we aggressively avoided duplicate bug reports by reporting only one wrong code bug at a time, and (2) wrong code bugs tend to be fixed much more slowly than crash bugs. In essence, the reasons that we reported fewer wrong code bugs than crash bugs are complex ones having more to do with social factors (and perhaps our own laziness) than to do with weaknesses of Csmith. Of course it might still be the case that EMI is better than Csmith at discovering middle-end optimizer bugs, but the EMI authors have not yet shown any evidence backing up that sort of claim. Finally, it is not necessarily useful to think of compiler crash bugs and wrong code bugs as being different things. The underlying bugs look much the same, the difference is often that in one case someone put the right assertion into the compiler (causing the inconsistency to be detected, leading to crash via assertion violation) and in the other case the inconsistency was undetected.

On a closely related note, after finishing this paper I was left asking: Given a limited testing budget, would it be better to run Csmith or to run Csmith+EMI? In other words, which method discovers either more bugs or higher-quality bugs? This would be a fun experiment to run, although there are some subtleties such as the fact that GCC and LLVM have (effectively) evolved to be somewhat resistant to Csmith, giving any new testing method an implicit advantage.

One thing about the EMI work that makes me selfishly happy is that over the last couple of years I’ve slacked off on reporting compiler bugs. This makes me feel guilty since Csmith continues to be capable of finding bugs in any given version of GCC or LLVM. Anyway, I feel like I’ve done my time here, so have fun with the bug reporting, guys!

In summary, EMI is very cool and anyone interested in compiler correctness should read the paper. It’s also worth thinking about how to apply its ideas (and metamorphic testing in general) to other application domains.
