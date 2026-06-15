---
title: Principles for Undefined Behavior in Programming Language Design
url: https://blog.regehr.org/archives/1436
published: "2016-10-10T20:09:15Z"
feed: regehr
guid: http://blog.regehr.org/?p=1436
---

# Principles for Undefined Behavior in Programming Language Design

I’ve had a post with this title on the back burner for years but I was never quite convinced that it would say anything I haven’t said before. Last night I watched [Chandler Carruth’s talk about undefined behavior at CppCon 2016](https://www.youtube.com/watch?v=yG1OZ69H_-o) and it is good material and he says it better than I think I would have, and I wanted to chat about it a bit.

First off, this is a compiler implementor’s point of view. Other smart people, such as Dan Bernstein, have a [very different point of view](https://groups.google.com/forum/m/#!msg/boring-crypto/48qa1kWignU/o8GGp2K1DAAJ) (but also keep in mind that Dan [doesn’t believe compiler optimization is useful](https://cr.yp.to/talks/2015.04.16/slides-djb-20150416-a4.pdf)).

Chandler is not a fan of the term [nasal demons](https://groups.google.com/forum/?hl=en#!msg/comp.std.c/ycpVKxTZkgw/S2hHdTbv4d8J), which he says is misleadingly hyperbolic, since the compiler isn’t going to maliciously turn undefined behavior (UB) into code for erasing your files or whatever. This is true, but Chandler leaves out the fact that our 28-year-long computer security train wreck (the [Morris Worm](https://en.wikipedia.org/wiki/Morris_worm) seems like as good a starting point as any) has been fueled to a large extent by undefined behavior in C and (later) C++ code. In other words, while the compiler won’t emit system calls for erasing your files, a memory-related UB in your program will permit a random person on the Internet to insert instructions into your process that issue system calls doing precisely that. From this slightly broader point of view, nasal demons are less of a caricature.

The first main idea in Chandler’s talk is that we should view UB at the PL level as being analogous to narrow contracts on APIs. Let’s look at this in more detail. An API with a wide contract is one where you can issue calls in any order, and you can pass any arguments to API calls, and expect predictable behavior. One simple way that an API can have a wider contract is by quietly initializing library state upon the first call into the library, as opposed to requiring an explicit call to an init() function. Some libraries do this, but many libraries don’t. For example, an OpenSSL man page says “SSL\_library\_init() must be called before any other action takes place.” This kind of wording indicates that a severe obligation is being placed on users of the OpenSSL API, and failing to respect it would generally be expected to result in unpredictable behavior. Chandler’s goal in this first part of the talk is to establish the analogy between UB and narrow API contracts and convince us that not all APIs want to be maximally wide. In other words, narrow APIs may be acceptable when their risks are offset by, for example, performance advantages.

Coming back to programming languages (PL), we can look at something like the signed left shift operator as exposing an API. The signed left shift API in C and C++ is particularly narrow and while many people have by now internalized that it can trigger UB based on the shift exponent (e.g., 1 << -1 is undefined), fewer developers have come to terms with restrictions on the left hand argument (e.g., 0 << 31 is defined but 1 << 31 is not). Can we design a wide API for signed left shift? Of course! We might specify, for example, that the result is zero when the shift exponent is too large or is negative, and that otherwise the result is the same as if the signed left-hand argument was interpreted as unsigned, shifted in the obvious way, and then reinterpreted as signed.

At this point in the talk, we should understand that “UB is bad” is an oversimplification, that there is a large design space relating to narrow vs. wide APIs for libraries and programming language features, and that finding the best point in this design space is not straightforward since it depends on performance requirements, on the target platform, on developers’ expectations, and more. C and C++, as low-level, performance-oriented languages, are famously narrow in their choice of contracts for core language features such as pointer and integer operations. The particular choices made by these languages have caused enormous problems and reevaluation is necessary and ongoing. The next part of Chandler’s talk provides a framework for deciding whether a particular narrow contract is a good idea or not.

Chandler provides these four principles for narrow language contracts:

1. Checkable (probabilistically) at runtime

2. Provide significant value: bug finding, simplification, and/or optimization

3. Easily explained and taught to programmers

4. Not widely violated by existing code that works correctly and as intended

The first criterion, runtime checkability, is crucial and unarguable: without it, we get latent errors of the kind that continue to contribute to insecurity and that have been subject to creeping exploitation by compiler optimizations. Checking tools such as ASan, UBSan, and tis-interpreter reduce the problem of finding these errors to the problem of software testing, which is very difficult, but which we need to deal with anyhow since there’s more to programming than eliminating undefined behaviors. Of course, any property that can be checked at runtime can also be checked without running the code. Sound static analysis avoids the need for test inputs but is otherwise much more difficult to usefully implement than runtime checking.

Principle 2 tends to cause energetic discussions, with (typically) compiler developers strongly arguing that UB is crucial for high-quality code generation and compiler users equally strongly arguing for defined semantics. I find the bug-finding arguments to be the most interesting ones: do we prefer Java-style two’s complement integers or would we rather retain maximum performance as in C and C++ or mandatory traps as in Swift or a hybrid model as in Rust? Discussions of this principle tend to center around examples, which is mostly good, but is bad in that any particular example excludes a lot of other use cases and other compilers and other targets that are also important.

Principle 3 is an important one that tends to get neglected in discussions of UB. The intersection of HCI and PL is not incredibly crowded with results, as far as I know, though many of us have some informal experience with this topic because we teach people to program. Chandler’s talk contains a section on explaining signed left shift that’s quite nice.

Finally, Principle 4 seems pretty obvious.

One small problem you might have noticed is that there are undefined behaviors that fail one or more of Chandler’s criteria, that many C and C++ compiler developers will defend to their dying breath. I’m talking about things like strict aliasing and [termination of infinite loops](https://blog.regehr.org/archives/140) that violate (at least) principles 1 and 3.

In summary, the list of principles proposed by Chandler is excellent and, looking forward, it would be great to use it as a standard set of questions to ask about any narrow contract, preferably before deploying it. Even if we disagree about the details, framing the discussion is super helpful.
