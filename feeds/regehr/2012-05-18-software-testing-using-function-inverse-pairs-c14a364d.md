---
title: Software Testing Using Function/Inverse Pairs
url: https://blog.regehr.org/archives/708
published: "2012-05-18T01:34:49Z"
feed: regehr
guid: http://blog.regehr.org/?p=708
---

# Software Testing Using Function/Inverse Pairs

One of the hardest problems in software testing is finding convenient, strong oracles—programs that can tell us whether the software under test worked correctly. Weak oracles, such as checking to make sure a program doesn’t crash, are always available, but not always of much use. An example of a strong oracle would be a reference implementation.

Sometimes software comes in function/inverse pairs, meaning that we get a program implementing some function f and another that implements f-1. When this happens we can (ideally) make up an input x and then check to make sure that f-1(f(x))=x. Often this works only in one direction. For example, let’s consider an assembler/disassembler pair. If I take some arbitrary assembly language, assemble it, and then disassemble the resulting object code, I’m unlikely to get exactly what I started with. On the other hand, if I take some arbitrary object code, disassemble it, and then assemble the resulting assembly code, I should get back the same object code. The question is whether or not f or f-1 is prepared to accept inputs in non-canonical forms. This problem can usually be solved by running a second cycle through the system. For example, Jesse Ruderman of [jsfunfuzz](http://www.squarefree.com/2007/08/02/introducing-jsfunfuzz/) fame reports taking JavaScript code in non-canonical form, compiling and decompiling it, [and then doing the same thing again](http://www.squarefree.com/2007/08/02/fuzzing-for-correctness/). If the second and third versions of the program differ, a bug has been found. I don’t know how many of the impressive [1500+ bugs](https://bugzilla.mozilla.org/show_bug.cgi?id=349611) found by jsfunfuzz come from this method.

Here’s a list of common function/inverse pairs that I came up with:

- pickle/unpickle, or save/load, of in-memory data
- checkpoint/restore of process or virtual machine state
- assemble/disassemble and compile/decompile
- encode/decode of data for transmission
- encrypt/decrypt
- compress/decompress, either lossless or lossy

The thing I would like to learn from you readers is what other function/inverse pairs you can think of. I feel like I must be missing many.

This topic was suggested by Alastair Reid who additionally suggested that an assembler/disassembler pair for an architecture such as ARM could be exhaustively tested in this fashion since there are only 232 possible instruction words. Very cool!
