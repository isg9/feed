---
title: We Need Hardware Traps for Integer Overflow
url: https://blog.regehr.org/archives/1154
published: "2014-05-28T16:31:01Z"
feed: regehr
guid: http://blog.regehr.org/?p=1154
---

# We Need Hardware Traps for Integer Overflow

Processors should support integer math instructions that optionally trap on overflow. Because popular architectures lack this feature, otherwise excellent modern systems programming languages, such as Rust, Go, and D, have default integer types that wrap. This is bad because unexpected wrapping causes programs to produce incorrect results, although of course integer overflow in a safe language is not the utter disaster that it is in C/C++, where integer overflow and type unsafety cooperate to create the worst kinds of bugs. The reason that Rust provides wrapping integers is that so far, the costs of a better semantics — both in terms of runtime overhead and in terms of implementation effort for the Rust team — exceed the perceived benefits. My belief is that hardware traps could change this inequation in a favorable way.

The lack of trapping math instructions doesn’t only hurt low-level languages. In JavaScript, where all numbers are semantically double-precision floats, replacing doubles with integer types in the runtime requires expensive software overflow checks. I’ve heard that this results in a 5-10% slowdown for basically all JavaScript code. In languages such as Python and Racket the default integer type overflows into bignums instead of doubles; Matthew Flatt tells me that Racket’s performance penalty due to software overflow checks probably exceeds 100% for jitted tight loops that do lots of integer math.

You might be saying to yourself: But I require wrapping integers to implement crypto codes and PRNGs and hash functions and stuff like that. The answer is easy: we provide wrapping operators that can be used in these specialized situations. One choice would be `+.`, `-.`, etc. In a unicode language we might use ⊞, ⊟, etc. (Jesse Ruderman suggested this, and I like the idea).

Architectures such as MIPS and Alpha support integer overflow traps. However, to a good approximation, the only architectures that matter right now are ARM’s and Intel’s. There are two issues in adding integer overflow traps to these ISAs. First, where do we get opcode space for the new trapping instructions? For x86 and x86-64, which support an elaborate system of instruction prefixes, it may make sense to use that mechanism, although this comes with a code size penalty. ARM has fixed-size instructions so finding space may be trickier there. A mail to a friend at ARM on this topic has so far gone unanswered, but I am guessing that this could be shoehorned into ARMv9 somehow. The second issue is the effect of integer overflow traps on chip area and critical path length. An experienced architect who I talked to doesn’t think these are serious problems, and in any case the complexity is dramatically lower than the complexity of implementing something like [hardware support for array bounds checking](http://en.wikipedia.org/wiki/Intel_MPX).

This post isn’t as much of an opinion piece as a plea to the folks at ARM, Intel, and AMD: Please provide this feature. It is needed in order to make high-level languages faster and low-level languages saner.

**UPDATE:** Some data about the overhead of software integer overflow checking in C/C++ codes can be found in [section IIID of this paper](http://www.cs.utah.edu/~regehr/papers/overflow12.pdf). Please keep in mind, however, that this implementation is tuned for debugging not performance. A highly tuned software integer undefined behavior checker for C/C++ could probably have overhead in the 5% range.
