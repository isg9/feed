---
title: Two Rules for Random Testing
url: https://blog.regehr.org/archives/381
published: "2011-02-12T00:00:40Z"
feed: regehr
guid: http://blog.regehr.org/?p=381
---

# Two Rules for Random Testing

When you run a test case on a piece of software, you’re conducting an experiment where the null hypothesis is “the system under test correctly executes this test.” The problem is that the value of each such experiment is small because there are so many possible inputs. Random testing has a stronger null hypothesis: “the system under test correctly executes as many generated test cases as we have time to run.” The idea is to explore parts of the space of possible system inputs that a human wouldn’t think to test. This leads us to:

> **Random Testing Rule 1:** Err on the side of expressiveness by prohibiting the generation of as few (legal) inputs as possible.

In other words, a better random tester is one that will — given time — explore a larger part of the input space. Expressiveness, however, can be difficult to achieve:

1. It is easy to fail to notice parts of the input space that should be explored. How many of us know all of Linux’s system calls? All of C’s trigraphs? All of GNU C’s inline assembly directives?
2. Some valid inputs are very hard to distinguish from invalid inputs. For example, assume that we’re attempting to generate random JavaScript programs that terminate. See the problem? On the other hand, often there are shortcuts — maybe we can just kill the apparently non-terminating programs after a timeout. In other cases, the system under test can itself tell us whether an input is valid.

Of course, expressiveness isn’t the whole story for random testing. For example, we can generate all possible JavaScript programs by generating random bit strings. However, if we actually do this we’ll waste almost all of our testing time generating bit strings and getting them rejected by the same tiny part of the JS lexer. This observation leads us to:

> **Random Testing Rule 2:** Restrict the expressiveness of generated programs as needed to achieve effective testing.

Trivially, the random bit string generator was too expressive to be an effective tester for a JavaScript implementation. Similarly, if I’m testing a new, buggy floating point optimization pass for a compiler, simply turning off integer variables in the test case generator may be the way to proceed. In general we must shape the inputs so that they exercise the right parts of the system under test.

Good taste is required to distinguish between parts of the input space that are germane vs. those that are irrelevant. For example, there is a large family of JavaScript programs differing only in choice of identifier names, and another large family differing only in choice of floating point constants. These sub-spaces of the input space probably do not need to be explored deeply because they are not tightly connected to the difficult logic in a JS VM. On the other hand, [see here](http://www.exploringbinary.com/php-hangs-on-numeric-value-2-2250738585072011e-308/) for a story about a magic FP constant that hangs PHP on certain platforms.

**Summary:** Good random testing is achieved by balancing the tension between too much and too little expressiveness in the test case generator.
