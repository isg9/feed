---
title: inline cache applications in scheme — wingolog
url: https://wingolog.org/archives/2012/05/29/inline-cache-applications-in-scheme
published: "2012-05-29T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2012/05/29/inline-cache-applications-in-scheme
---

# inline cache applications in scheme — wingolog

## [inline cache applications in scheme](/archives/2012/05/29/inline-cache-applications-in-scheme)

29 May 2012 8:07 AM

- [scheme](/tags/scheme)
- [javascript](/tags/javascript)
- [v8](/tags/v8)
- [guile](/tags/guile)
- [plt](/tags/plt)
- [dispatch](/tags/dispatch)
- [goops](/tags/goops)
- [pic](/tags/pic)
- [self](/tags/self)

The [inline cache](http://en.wikipedia.org/wiki/Inline_caching) is a dynamic language implementation technique that originated in Smalltalk 80 and Self, and made well-known by JavaScript implementations. It is fundamental for getting good JavaScript performance.

**a cure for acute dynamic dispatch**

A short summary of the way inline caches work is that when you see an operation, like `x + y`, you don't compile in a procedure call to a generic addition subroutine. Instead, you compile a call to a procedure stub: the inline cache (IC). When the IC is first called, it will generate a new procedure specialized to the particular types that flow through that particular call site. On the next call, if the types are the same, control flows directly to the previously computed implementation. Otherwise the process repeats, potentially resulting in a [*polymorphic* inline cache](http://www.cs.ucsb.edu/~urs/oocsb/papers/ecoop91.pdf) (one with entries for more than one set of types).

An inline cache is called "inline" because it is specific to a particular call site, not to the operation. Also, adaptive optimization can later inline the stub in place of the call site, if that is considered worthwhile.

Inline caches are a win wherever you have dynamic dispatch: named field access in JavaScript, virtual method dispatch in Java, or generic arithmetic -- and here we get to Scheme.

**the skeptical schemer**

What is the applicability of inline caches to Scheme? The only places you have dynamic dispatch in Scheme are in arithmetic and in ports.

Let's take arithmetic first. Arithmetic operations in Scheme can operate on number of a wide array of types: fixnums, bignums, single-, double-, or multi-precision floating point numbers, complex numbers, rational numbers, etc. Scheme systems are typically compiled ahead-of-time, so in the absence of type information, you always want to inline the fixnum case and call out \[of line\] for other cases. (Which line is this? The line of flow control: the path traced by a program counter.) But if you end up doing a lot of floating-point math, this decision can cost you. So inline caches can be useful here.

Similarly, port operations like `read-char` and `write` can operate on any kind of port. If you are always writing UTF-8 data to a file port, you might want to be able to inline `write` for UTF-8 strings and file ports, possibly inlining directly to a syscall. It's probably a very small win in most cases, but a win nonetheless.

These little wins did not convince me that it was worthwhile to use ICs in a Scheme implementation, though. In the context of Guile, they're even less applicable than usual, because Guile is a bytecode-interpreted implementation with a self-hosted compiler. ICs work best when implemented as runtime-generated native code. Although it probably will by the end of the year, Guile doesn't generate native code yet. So I was skeptical.

**occam's elf**

Somehow, through all of this JavaScript implementation work, I managed to forget the biggest use of inline caches in GNU systems. Can you guess?

The [PLT](http://www.airs.com/blog/archives/41)!

You may have heard how this works, but if you haven't, you're in for a treat. When you compile a shared library that has a reference to `printf`, from the C library, the compiler doesn't know where `printf` will be at runtime. So even in C, that most static of languages, we have a form of dynamic dispatch: a call to an unknown callee.

When the dynamic linker loads a library at runtime, it could resolve all the dynamic references, but instead of doing that, it does something more clever: it doesn't. Instead, the compiler and linker collude to make the call to `printf` call a stub -- an inline cache. The first time that stub is called, it will resolve the dynamic reference to `printf`, and replace the stub with an indirect call to the procedure. In this way we trade off a faster loading time for dynamic libraries at the cost of one indirection per call site, for the inline cache. This stub, this inline cache, is sometimes called the PLT entry. You might have seen it in a debugger or a disassembler or something.

I found this when I was writing an [ELF](http://en.wikipedia.org/wiki/Executable_and_Linkable_Format) linker for Guile's new virtual machine. More on that at some point in the future. ELF is interesting: I find that if I can't generate good code in the ELF format, I'm generating the wrong kind of code. Its idiosyncrasies remind me of what happens at runtime.

**lambda: the ultimate inline cache**

So, back to Scheme. Good Scheme implementations are careful to have only one way of calling a procedure. Since the only kind of callable object in the Scheme language is generated by the `lambda` abstraction, Scheme implementations typically produce uniform code for procedure application: load the procedure, prepare the arguments, and go to the procedure's entry point.

However, if you're already eating the cost of dynamic linking -- perhaps via separately compiled Scheme modules -- you might as well join the operations of "load a dynamically-linked procedure" and "go to the procedure's entry point" into a call to an inline cache, as in C shared libraries. In the cold case, the inline cache resolves the dynamic reference, updates the cache, and proceeds with the call. In the hot case, the cache directly dispatches to the call.

One benefit of this approach is that it now becomes cheap to support other kinds of applicable objects. One can make hash tables applicable, if that makes sense. (Clojure folk seem to think that it does.) Another example would be to more efficiently support dynamic programming idioms, like [generic functions](//wingolog.org/archives/2008/10/17/dispatch-strategies-in-dynamic-languages). Inline caches in Scheme would allow generic functions to have per-call-site caches instead of per-operation caches, which could be a big win.

It seems to me that this dynamic language implementation technique could allow Guile programmers to write different kinds of programs. The code to generate an inline cache could even itself be controlled by a meta-object protocol, so that the user could precisely control application of her objects. The mind boggles, but pleasantly so!

Thanks to Erik Corry for provoking this thought, via a conversation at JSConf EU last year. All blame to me, of course.

**as [PLT\_HULK](http://twitter.com/PLT_HULK) would say**

NOW THAT'S AN APPLICATION OF AN INLINE CACHE! HA! HA HA!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [dynamic dispatch: a followup](/archives/2008/10/19/dynamic-dispatch-a-followup)
- [on generators](/archives/2013/02/25/on-generators)
- [JavaScriptCore, the WebKit JS implementation](/archives/2011/10/28/javascriptcore-the-webkit-js-implementation)
- [v8: a tale of two compilers](/archives/2011/07/05/v8-a-tale-of-two-compilers)
- [on-stack replacement in v8](/archives/2011/06/20/on-stack-replacement-in-v8)

### 8 responses

1. Will says:[29 May 2012 10:25 AM](#b763aca847ce35baf169a74163d3a9c10b83ddbb)

   a win for "virtual method dispatch in Java"? I'd have imagined vtables win out because Java is a statically typed language?

2. [wingo](http://wingolog.org/) says:[29 May 2012 11:11 AM](#47d2aa5af81ddc0d698bcd1aad5a573b560e9eea)

   You can indeed do better than vtables; see [Keith Rarick](http://wingolog.org/archives/2008/10/17/dispatch-strategies-in-dynamic-languages#80963571c8979abaf6b085a32e737c9dd73ffd90)'s comment here, some 5 years ago.

3. [Manuel Simoni](http://axisofeval.blogspot.com) says:[29 May 2012 12:04 PM](#2aa032739afb113be7fbec35fb6964cb433689d1)

   Reminds me of [Modern dynamic linking infrastructure for PLT](http://lambda-the-ultimate.org/node/3474).

4. [Jeff Walden](http://whereswalden.com/) says:[29 May 2012 3:55 PM](#d781087a0023235992952736693fab9cc0118766)

   I guess I'm way too close to this stuff now, because you say, "The only places" and my first thought is, "What about map/apply/reduce/for-each?!?!" :-)

5. John Cowan says:[29 May 2012 4:40 PM](#a11d0186a9a13eed08b94b3ff972e35fa80b2eeb)

   "Scheme implementations typically produce uniform code for procedure application: load the procedure, prepare the arguments, and go to the procedure's entry point."

   Say what? One of the classical Scheme compiler optimizations, going right back to the Rabbit compiler, is to handle different kinds of procedures differently. Are all the variables local? Don't bother to create a closure. Is the continuation always known (usually the result of LET, AND, OR)? Just have the code jump to its continuation. And so on.

   It's true that some fast compilers, like Chicken, treat all procedures uniformlyl, but Chicken pays a price for that: everything is pretty durn fast, but nothing is ultra-fast. For example, even programs that don't use call/cc pay the price for it.

   Because of the double whammy of boxed floats and default compilation for fixnums, I have a project named Flopsy (basically in the "think" stage right now) that compiles a \*very\* small subset of Scheme into C. Everything is statically typed, the only numbers are floats, and the only vectors are vectors of floats. There are no lists or symbols at all! The idea is that you write your floating-point core code in the Flopsy subset and debug it in your favorite Scheme. Then you compile the core with Flopsy and gcc, and use your favorite Scheme's FFI to invoke it. (Of course, this assumes that your FFI is decently fast....)

6. [Pascal Costanza](http://p-cos.net) says:[3 June 2012 4:57 PM](#1cd49018ff28e45f70819bfc84a429630affb7d4)

   Here is a paper that might interest you: http://people.csail.mit.edu/jrb/Projects/partial-dispatch.htm

7. Pierre De Pascale says:[6 June 2012 12:58 PM](#b379a98fcce28996a7f97cc24787bfc930e85ad1)

   Inline cache or even Polymorphic Inline Cache as defined in the OO literature are not really useful for the implementation functional programming language like Scheme: there is no implicit dispatch (method call)! The dispatch is always explicit, like:

   (define (+ n1 n2)

   (cond

   ((and (fixnum? n1) (fixnum? n2)) (fx+ n1 n2))

   ((and (flonum? n1) (flonum? n2)) (flo+ n1 n2))))

   If you want to apply the idea of IC or PIC to Scheme, the IC should track the true function being called. This allows you to maybe inline the function on the 2nd optimized compilation. You should also perform some kind of redundant code removal. In this case the expression (+ n 42) gets compiled to the following code given the above + definition:

   (let ((n1 n))

   (cond

   ((fixnum? n1) (fx+ n1 42))

   ((flonum? n1) (flo+ n1 42))))

   and if n has been known to be a fixnum, everything gets compiled to (fx+ n1 42).

8. [Tony Garnock-Jones](http://homepages.kcbbs.gen.nz/tonyg/) says:[22 June 2012 2:52 PM](#b98a8402a5681f0b86f31fc4e9fb699d8bf6fce6)

   On that last point, meta-object protocols for IC control, you might find this paper relevant:

   ﻿I. Piumarta and A. Warth, “Open, extensible object models,” 2007.

   http://piumarta.com/software/cola/objmodel2.pdf

Comments are closed.
