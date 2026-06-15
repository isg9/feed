---
title: Software Testing Using Easy Cases
url: https://blog.regehr.org/archives/858
published: "2012-12-18T17:14:05Z"
feed: regehr
guid: http://blog.regehr.org/?p=858
---

# Software Testing Using Easy Cases

Chapter 2 of [Street Fighting Mathematics](http://mitpress.mit.edu/sites/default/files/titles/content/9780262514293_Creative_Commons_Edition.pdf) is called Easy Cases. The idea is very simple:

> A correct solution works in all cases, including the easy ones.

Solution 2 in [this writeup of the napkin ring problem](http://www.puzzles.com/puzzleplayground/holeinthesphere/holeinthesphereprintplay.pdf) is a fun example of applying this principle.

Easy cases can be exploited for software testing as well as in mathematics. Many instances of this are trivial in the sense that any test case written by a human is likely to be an easy case when compared to the universe of all possible inputs. What I’m interested in here are clever and non-obvious applications of easy cases.

Let’s take the example of an [FIR filter](http://en.wikipedia.org/wiki/Finite_impulse_response). Since these eat up a lot of CPU cycles in some systems, people create fiendishly optimized FIRs in assembly that are not so easy to verify by inspection. Chapter 8 of the [ARM System Developer’s Guide](http://www.amazon.com/ARM-System-Developers-Guide-Architecture/dp/1558608745) has many great examples ( [code here](http://www.elsevierdirect.com/companions/9781558608740/software/files_by_chapter/ch08_dsp.zip)). An FIR filter implementation can be tested using the following easy cases:

- Feed the filter an impulse signal (…,0,0,0,1,0,0,0,…) and it should output its coefficients in order.
- Feed the filter a step signal (…,0,0,0,1,1,1,…) and its output should settle to the sum of its coefficients.
- Feed the filter a sine wave and its output should be a sine wave with the expected amplitude given the filter’s design.

Can you write an FIR that is wrong, but that passes these tests? Sure, but these will weed out a lot of wrong implementations. I learned about these tests [here](http://www.dspguru.com/dsp/faqs/fir/implementation). In general, a lot of codes, particularly mathy ones, have these easy cases that can and should be exploited for simple sanity checks.
