---
title: When is Undefined Behavior OK?
url: https://blog.regehr.org/archives/748
published: "2012-07-09T17:48:18Z"
feed: regehr
guid: http://blog.regehr.org/?p=748
---

# When is Undefined Behavior OK?

Under what circumstances is it acceptable for a programming language to admit undefined behaviors? Here I mean [undefined behavior in the C/C++ sense](https://blog.regehr.org/archives/213) where, for example, “anything can happen” when you use an uninitialized variable. In my opinion, five conditions need to be fulfilled.

First, the undefined behavior must provide a significant, robust performance win. By “significant” I mean well into double-digit percent speedup and by “robust” I mean that a lot of programs should receive the benefit. The undefinedness of out-of-bounds array accesses in C/C++ clearly satisfies this criterion. The undefinedness of signed overflow does not (I think) because you’ll need to look pretty hard before you can find an important application that gets double-digit slowdown if you compile it using GCC or Clang’s `-fwrapv` flag. People I trust have told me that these programs do exist, but I doubt that most of us use them on a daily basis.

Second, behavior should only be undefined as a last resort—when it cannot be reasonably phrased as an unspecified or implementation-defined behavior. In C/C++, an *unspecified behavior* is one where the implementation chooses from a set of well-defined behaviors, but there is no requirement that the choice is documented or consistent. An *implementation-defined behavior* is one that must be documented and consistent. As a random example, in C99 a program’s behavior is undefined if “the specification of a function type includes any type qualifiers.” Making this implementation-defined would not seem to lose anything. Many of C99’s 191 undefined behaviors have a similar character.

Third, undefined behavior is only acceptable when a sound, complete, efficient, and widely available dynamic checker exists (of course static checkers are good too). Preferably these checkers should be accessible using commonly-available compiler flags. For example, `gcc -Wall` serves as a reliable static checker for many undefined behaviors in C99 (but not for the any of the worst ones, of course). To meet this criterion, C/C++ would have to be modified to make memory safety easier to enforce, particularly in the presence of separate compilation. [IOC](http://embed.cs.utah.edu/ioc/) is dynamically sound and complete with respect to some of C/C++’s integer undefined behaviors. More such tools are needed.

Fourth, the undefined behavior needs to be comprehensible and local. By “local” I mean that we need to be able to blame a very small amount of code for any bug relating to undefined behavior. Most of C/C++’s undefined behaviors (even the ones related to integer overflow and array accesses) are perfectly understandable and local. On the other hand, C99’s strict aliasing rules are an abysmal failure in this regard—as far as I can tell, not even compiler implementors fully understand how these interact with rules for accessing union fields. The undefinedness of non-terminating loops in C++11 is another nasty failure to meet this criterion because the undefined behavior is a property of all code that is reachable from the loop.

Finally, the undefined behavior should only punish programs that engage in risky behavior. For example, making data races undefined is OK if we consider multithreading to be risky (as I do). On the other hand, undefined integer overflows are not OK because few people would consider `i++` to be a risky behavior. The best way to satisfy this criterion is by making the behavior defined by default, but then providing a greppable loophole. For example, the `reinterpret_cast` in C++ seems like a nice move in this direction. A special no-overflow qualifier for induction variables would probably give all of the performance benefits that we currently get from undefined signed overflows without any of the global punishment.

One might interpret these guidelines as saying “undefined behavior is never OK.” However, that is not what I intend. Niche languages for low-level, performance-critical applications—embedded systems and OS kernels, mainly—will have undefined behaviors for the foreseeable future and I think that is fine. On the other hand, the current situation in C and C++ is not fine and we are paying for it every day.
