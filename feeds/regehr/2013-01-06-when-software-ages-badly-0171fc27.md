---
title: When Software Ages Badly
url: https://blog.regehr.org/archives/866
published: "2013-01-06T17:21:46Z"
feed: regehr
guid: http://blog.regehr.org/?p=866
---

# When Software Ages Badly

In some respects, software ages gracefully: it generally starts out working poorly but gets better over time as bugs are fixed. Unlike hardware, there’s no physical wearing out of parts. This post is about a few ways in which software doesn’t get better with age.

In *The Mythical Man Month* Brooks observes that any sufficiently complex software system will tend to have an “irreducible number of errors”—a reference to the fact that in a crufty old software base, any bug fix is likely to break something else. It is possible to fight this phenomenon through aggressive modularization and refactoring but these are expensive.

The second way for old software to become less reliable is when the meaning of its code changes. Obviously this can happen when the software is ported to a new architecture where, for example, pointers or integers have a different size. Also, a library or operating system update—even an obvious fix for a simple bug—can break an application that relied on the previous semantics. A compiler upgrade can break working code in any number of insidious ways. For example, an improved optimizer can make code run faster, exposing previously latent race conditions. A C/C++ compiler’s [exploitation of undefined behavior](https://blog.regehr.org/archives/761) can be ratcheted up a notch, breaking code that previously worked. This has been continuously happening for at least a decade and it’s not going to stop in the foreseeable future.

The inputs that are provided to a software system can change in harmful ways. The best example of this that I can think of is when a new class of attack becomes well understood, such as stack buffer overflows, heap overflows, and integer overflows. When a nasty new fuzzing tool appears, a previously solid-seeming piece of code can be shown to be not correct at all. The input domain can also change shape when a piece of software is reused in a new environment; [Ariane 5 flight 501](http://www.di.unito.it/~damiani/ariane5rep.html) is an excellent example.

Finally, in the long run, good old software rot can happen. Programming idioms, programming languages, software architectures, and other aspects of code are always evolving and eventually progress gets the better of almost any piece of software.

We’ve been writing software for hardly more than half a century, and I suspect that the vast majority of lines of code in existence were written since 2000. What will the world look like when we’ve had software for as long as we’ve had the printing press? What about when we’ve had computer languages for as long as we’ve had spoken languages? How are we going to perform refactoring on a code base of a hundred billion lines? Vernor Vinge [envisions professional programmer-archaeologists](http://www.amazon.com/Deepness-Sky-Vernor-Vinge/dp/0812536355/) whose job it is to deal with these systems; I’m not sure we’re that far off from needing these people now.
