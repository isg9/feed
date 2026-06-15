---
title: Integer Overflow Paper
url: https://blog.regehr.org/archives/691
published: "2012-03-28T15:35:55Z"
feed: regehr
guid: http://blog.regehr.org/?p=691
---

# Integer Overflow Paper

My coauthors and I just finished the final version of our [paper about integer overflows in C/C++ programs](http://www.cs.utah.edu/~regehr/papers/overflow12.pdf) that’s going to appear at [ICSE 2012](http://www.ifi.uzh.ch/icse2012/), a software engineering conference. Basically we made [a tool for dynamically finding integer overflows](http://embed.cs.utah.edu/ioc/) (and related integer undefined behaviors) and used it to look at a lot of software. As you might expect, lots of overflows occur.

Our analysis is based on dividing overflows into four kinds:

1. Intentional, well-defined overflows, such as letting an unsigned integer wrap around in a PRNG. These are not a problem.
2. Unintentional, well-defined overflows, such as an unsigned multiplication wrapping around when this was not expected to happen. These are logic errors.
3. Intentional, undefined overflows, such as computing INT\_MAX using (1<<31)-1. These are often “time bombs” — behaviors that (may) currently work but are waiting to be broken by improvements in compiler optimization.
4. Unintentional, undefined overflows, such as letting a signed multiplication overflow when this was not expected to happen. These are logic errors.

The conclusion is that people should at least test for undefined behaviors using a tool like IOC, and probably should also check for well-defined but unexpected overflows.
