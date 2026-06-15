---
title: Rust is mostly safety
url: https://graydon2.dreamwidth.org/247406.html
published: "2016-12-29T08:16:45Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:247406
---

# Rust is mostly safety

Steve Klabnik recently writes that ["Rust is more than safety"](http://words.steveklabnik.com/rust-is-more-than-safety); in this post he argues that Rust's "marketing pitch" focuses too much on safety, and that that topic doesn't really resonate with everyone.

I'll grant that if one's goal is strictly to expand the market for Rust, it might be worthwhile looking at other strengths that might appeal to other users who don't care about "systems languages" + "safety". And there is, of course, a lot to love in Rust besides its safety features. We actively reproduced that which was tasteful and powerful from other languages. The traits system of bounded polymorphism and existentials is a delight. The classical ML-style algebraic types and patterns are as wonderful as ever. The expressive and flexible compilation model, along with a first class package manager, is a breath of fresh air in systems work.

And yet. And yet.

There is part of me that winces when someone goes looking for a justification for Rust above and beyond safety. Safety in the systems space is Rust's raison d'être. Especially safe *concurrency* (or as Aaron put it, [fearless concurrency](https://blog.rust-lang.org/2015/04/10/Fearless-Concurrency.html)). I do not know how else to put it. When someone says they "don't have safety problems" in C++, I am astonished: a statement that must be made in ignorance, if not outright negligence. The fact of the matter is that the further down the software stack one goes, the *worse the safety situation gets*. Our engineering discipline has this dirty secret, but it is not so secret anymore: every day the world stumbles forward on creaky, malfunctioning, vulnerable, error-prone systems software and every day the toll in human misery increases. Billions of dollars, countless lives lost. Can one really say "I don't have safety problems" in systems programming?

I do not mean to pick on C++: the same problems plague C, Ada, Alef, Pascal, Mesa, PL/I, Algol, Forth, Fortran ... show me a language with manual memory management and threading, and I will show you an engineering tragedy waiting to happen. I do not think it is too unfair to say that if Rust provided *nothing but* safety upgrades to the systems space, it would have been worth all the time and energy. And it *has* been the overriding design priority from the very beginning: no wild pointers, no null pointers, no shared mutable state. Take a look at the [very first slide deck](http://venge.net/graydon/talks/intro-talk-2.pdf). That's what Rust has always been about: bringing safety features to the systems niche, a niche that has stubbornly resisted them for decades while higher-level languages have moved ahead in safety.

A few valiant attempts at bringing GC into systems programming -- Modula-3, Eiffel, Sather, D, Go -- have typically cut themselves off from too many tasks due to tracing GC overhead and runtime-system incompatibility, and *still* failed to provide a safe concurrency model.

Rust's success in raising the bar for safe, concurrent systems programming -- in ways that the niche finds usable, performant and compatible -- cannot (imo) be overstated. That is its purpose, and that has been its great success.

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=247406) comments
