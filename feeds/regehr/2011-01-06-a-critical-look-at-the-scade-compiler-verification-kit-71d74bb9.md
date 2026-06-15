---
title: A Critical Look at the SCADE Compiler Verification Kit
url: https://blog.regehr.org/archives/337
published: "2011-01-06T21:54:02Z"
feed: regehr
guid: http://blog.regehr.org/?p=337
---

# A Critical Look at the SCADE Compiler Verification Kit

While searching for related work on compiler testing and certification, I ran across the [SCADE Compiler Verification Kit](http://www.esterel-technologies.com/products/compiler-verification-kit/): a collection of SCADE-generated C code along with some test vectors. The idea is to spot compiler problems early and in a controlled fashion by testing the compiler using the kinds of C code that SCADE generates.

SCADE is a suite of tools for building safety critical embedded systems; it generates C code that is then cross-compiled for the target platform using a convenient compiler. Using C as a portable assembly language is a fairly common strategy due to the large variety of embedded platforms out there.

Given that there is a lot of variation in the quality of embedded C compilers (in other words, there are some really buggy compilers out there), something like the Compiler Verification Kit (CVK) probably had to be created. It’s a great idea and I’m sure it serves its purpose. Unfortunately, the developers over-state their claims. The CVK web page says:

> Once this verification of the compiler is complete, **no further low-level testing is needed** on the SCADE-generated object code. \[emphasis is theirs\]

This is saying that since the CVK tests all language features and combinations of language features used by SCADE, the CVK will reveal all compiler bugs that matter. This claim is fundamentally wrong. Just as a random example, we’ve found compiler bugs that depend on the choice of a constant used in code. Is the CVK testing all possible choices of constants? Of course not.

The web page also states that that CVK has:

> …the test vectors needed to achieve 100% MC/DC of structural code coverage.

No doubt this is true, but it is misleading. 100% coverage of the generated code is not what is needed here. 100% coverage of the compiler under test would be better, but even that would be insufficient to guarantee the absence of translation bugs. Creating safety-critical systems that work is a serious business, and it would be nice if the tool vendors in this sector took a little more trouble to provide accurate technical claims.
