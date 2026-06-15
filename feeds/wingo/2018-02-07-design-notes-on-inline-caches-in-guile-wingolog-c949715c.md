---
title: design notes on inline caches in guile — wingolog
url: https://wingolog.org/archives/2018/02/07/design-notes-on-inline-caches-in-guile
published: "2018-02-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2018/02/07/design-notes-on-inline-caches-in-guile
---

# design notes on inline caches in guile — wingolog

## [design notes on inline caches in guile](/archives/2018/02/07/design-notes-on-inline-caches-in-guile)

7 February 2018 3:14 PM

- [inline caches](/tags/inline%20caches)
- [compilers](/tags/compilers)
- [v8](/tags/v8)
- [igalia](/tags/igalia)
- [guile](/tags/guile)
- [gnu](/tags/gnu)
- [adaptive optimization](/tags/adaptive%20optimization)
- [type feedback](/tags/type%20feedback)

Ahoy, programming-language tinkerfolk! Today's rambling missive chews the gnarly bones of ["inline caches"](https://en.wikipedia.org/wiki/Inline_caching), in general but also with particular respect to the [Guile](https://gnu.org/s/guile) implementation of Scheme. First, a little intro.

**inline what?**

Inline caches are a language implementation technique used to accelerate polymorphic dispatch. Let's dive in to that.

By *implementation technique*, I mean that the technique applies to the language compiler and runtime, rather than to the semantics of the language itself. The effects on the language do exist though in an indirect way, in the sense that inline caches can make some operations faster and therefore more common. Eventually inline caches can affect what users expect out of a language and what kinds of programs they write.

But I'm getting ahead of myself. *Polymorphic dispatch* literally means "choosing based on multiple forms". Let's say your language has immutable strings -- like Java, Python, or Javascript. Let's say your language also has operator overloading, and that it uses `+` to concatenate strings. Well at that point you have a problem -- while you can specify a terse semantics of some core set of operations on strings (win!), you can't choose one representation of strings that will work well for all cases (lose!). If the user has a workload where they regularly build up strings by concatenating them, you will want to store strings as trees of substrings. On the other hand if they want to access characterscodepoints by index, then you want an array. But if the codepoints are all below 256, maybe you should represent them as bytes to save space, whereas maybe instead as 4-byte codepoints otherwise? Or maybe even UTF-8 with a codepoint index side table.

The right representation (form) of a string depends on the myriad ways that the string might be used. The `string-append` operation is *polymorphic*, in the sense that the precise code for the operator depends on the representation of the operands -- despite the fact that the *meaning* of `string-append` is monomorphic!

Anyway, that's the problem. Before inline caches came along, there were two solutions: callouts and open-coding. Both were bad in similar ways. A callout is where the compiler generates a call to a generic runtime routine. The runtime routine will be able to handle all the myriad forms and combination of forms of the operands. This works fine but can be a bit slow, as all callouts for a given operator (e.g. `string-append`) dispatch to a single routine for the whole program, so they don't get to optimize for any particular call site.

One tempting thing for compiler writers to do is to effectively inline the `string-append` operation into each of its call sites. This is "open-coding" (in the terminology of the early Lisp implementations like MACLISP). The advantage here is that maybe the compiler knows something about one or more of the operands, so it can eliminate some cases, effectively performing some compile-time specialization. But this is a limited technique; one could argue that the whole point of polymorphism is to allow for generic operations on generic data, so you rarely have compile-time invariants that can allow you to specialize. Open-coding of polymorphic operations instead leads to code bloat, as the `string-append` operation is just so many copies of the same thing.

Inline caches emerged to solve this problem. They trace their lineage back to Smalltalk 80, gained in complexity and power with Self and finally reached mass consciousness through Javascript. These languages all share the characteristic of being dynamically typed and object-oriented. When a user evaluates a statement like `x = y.z`, the language implementation needs to figure out where `y.z` is actually located. This location depends on the representation of `y`, which is rarely known at compile-time.

However for any given reference `y.z` in the source code, there is a finite set of concrete representations of `y` that will actually flow to that call site at run-time. Inline caches allow the language implementation to specialize the `y.z` access for its particular call site. For example, at some point in the evaluation of a program, `y` may be seen to have representation R1 or R2. For R1, the `z` property may be stored at offset 3 within the object's storage, and for R2 it might be at offset 4. The inline cache is a bit of specialized code that compares the type of the object being accessed against R1 , in that case returning the value at offset 3, otherwise R2 and offset r4, and otherwise falling back to a generic routine. If this isn't clear to you, Vyacheslav Egorov write a [fine article describing and implementing the object representation optimizations enabled by inline caches](http://mrale.ph/blog/2012/06/03/explaining-js-vms-in-js-inline-caches.html).

Inline caches also serve as input data to later stages of an adaptive compiler, allowing the compiler to selectively inline (open-code) only those cases that are appropriate to values actually seen at any given call site.

**but how?**

The classic formulation of inline caches from Self and early V8 actually patched the code being executed. An inline cache might be allocated at address `0xcabba9e5` and the code emitted for its call-site would be `jmp 0xcabba9e5`. If the inline cache ended up bottoming out to the generic routine, a new inline cache would be generated that added an implementation appropriate to the newly seen "form" of the operands and the call-site. Let's say that new IC (inline cache) would have the address `0x900db334`. Early versions of V8 would actually patch the machine code at the call-site to be `jmp 0x900db334` instead of `jmp 0xcabba6e5`.

Patching machine code has a number of disadvantages, though. It inherently target-specific: you will need different strategies to patch x86-64 and armv7 machine code. It's also expensive: you have to flush the instruction cache after the patch, which slows you down. That is, of course, if you are allowed to patch executable code; on many systems that's impossible. Writable machine code is a potential vulnerability if the system may be vulnerable to remote code execution.

Perhaps worst of all, though, patching machine code is not thread-safe. In the case of early Javascript, this perhaps wasn't so important; but as JS implementations gained parallel garbage collectors and JS-level parallelism via "service workers", this becomes less acceptable.

For all of these reasons, the modern take on inline caches is to implement them as a memory location that can be atomically modified. The call site is just `jmp *loc`, as if it were a virtual method call. Modern CPUs have "branch target buffers" that predict the target of these indirect branches with very high accuracy so that the indirect jump does not become a pipeline stall. (What does this mean in the face of the Spectre v2 vulnerabilities? Sadly, God only knows at this point. Saddest panda.)

**cry, the beloved country**

I am interested in ICs in the context of the Guile implementation of Scheme, but first I will make a digression. Scheme is a very monomorphic language. Yet, this monomorphism is entirely cultural. It is in no way essential. Lack of ICs in implementations has actually fed back and encouraged this monomorphism.

Let us take as an example the case of property access. If you have a pair in Scheme and you want its first field, you do `(car x)`. But if you have a vector, you do `(vector-ref x 0)`.

What's the reason for this nonuniformity? You could have a generic `ref` procedure, which when invoked as `(ref x 0)` would return the field in `x` associated with 0. Or `(ref x 'foo)` to return the `foo` property of `x`. It would be more orthogonal in some ways, and it's completely valid Scheme.

We don't write Scheme programs this way, though. From what I can tell, it's for two reasons: one good, and one bad.

The good reason is that saying `vector-ref` means more to the reader. You know more about the complexity of the operation and what side effects it might have. When you call `ref`, who knows? Using concrete primitives allows for better program analysis and understanding.

The bad reason is that Scheme implementations, Guile included, tend to compile `(car x)` to much better code than `(ref x 0)`. Scheme implementations in practice aren't well-equipped for polymorphic data access. In fact it is standard Scheme practice to abuse the "macro" facility to manually inline code so that that certain performance-sensitive operations get inlined into a closed graph of monomorphic operators with no callouts. To the extent that this is true, Scheme programmers, Scheme programs, and the Scheme language as a whole are all victims of their implementations. JavaScript, for example, does not have this problem -- to a small extent, maybe, yes, performance tweaks and tuning are always a thing but JavaScript implementations' ability to burn away polymorphism and abstraction results in an entirely different character in JS programs versus Scheme programs.

**it gets worse**

On the most basic level, Scheme is the call-by-value lambda calculus. It's well-studied, well-understood, and eminently flexible. However the way that the syntax maps to the semantics hides a constrictive monomorphism: that the "callee" of a call refer to a lambda expression.

Concretely, in an expression like `(a b)`, in which `a` is not a macro, `a` must evaluate to the result of a `lambda` expression. Perhaps by reference (e.g. `(define a (lambda (x) x))`), perhaps directly; but a lambda nonetheless. But what if `a` is actually a vector? At that point the Scheme language standard would declare that to be an error.

The semantics of Clojure, though, would allow for `((vector 'a 'b 'c) 1)` to evaluate to `b`. Why not in Scheme? There are the same good and bad reasons as with `ref`. Usually, the concerns of the language implementation dominate, regardless of those of the users who generally want to write terse code. Of course in some cases the implementation concerns *should* dominate, but not always. Here, Scheme could be more flexible if it wanted to.

**what have you done for me lately**

Although inline caches are not a miracle cure for performance overheads of polymorphic dispatch, they are a tool in the box. But what, precisely, can they do, both in general and for Scheme?

To my mind, they have five uses. If you can think of more, please let me know in the comments.

Firstly, they have the classic named property access optimizations as in JavaScript. These apply less to Scheme, as we don't have generic property access. Perhaps this is a deficiency of Scheme, but it's not exactly low-hanging fruit. Perhaps this would be more interesting if Guile had more generic protocols such as Racket's iteration.

Next, there are the arithmetic operators: addition, multiplication, and so on. Scheme's arithmetic is indeed polymorphic; the addition operator `+` can add any number of complex numbers, with a distinction between exact and inexact values. On a representation level, Guile has fixnums (small exact integers, no heap allocation), bignums (arbitrary-precision heap-allocated exact integers), fractions (exact ratios between integers), flonums (heap-allocated double-precision floating point numbers), and compnums (inexact complex numbers, internally a pair of doubles). Also in Guile, arithmetic operators are a "primitive generics", meaning that they can be extended to operate on new types at runtime via GOOPS.

The usual situation though is that any particular instance of an addition operator only sees fixnums. In that case, it makes sense to only emit code for fixnums, instead of the product of all possible numeric representations. This is a clear application where inline caches can be interesting to Guile.

Third, there is a very specific case related to dynamic linking. Did you know that most programs compiled for GNU/Linux and related systems have inline caches in them? It's a bit weird but the ["Procedure Linkage Table"](https://www.airs.com/blog/archives/41) (PLT) segment in ELF binaries on Linux systems is set up in a way that when e.g. `libfoo.so` is loaded, the dynamic linker usually doesn't eagerly resolve all of the external routines that `libfoo.so` uses. The first time that `libfoo.so` calls `frobulate`, it ends up calling a procedure that looks up the location of the `frobulate` procedure, then patches the binary code in the PLT so that the next time `frobulate` is called, it dispatches directly. To dynamic language people it's the weirdest thing in the world that the C/C++/everything-static universe has at its cold, cold heart a hash table and a dynamic dispatch system that it doesn't expose to any kind of user for instrumenting or introspection -- any user that's not a malware author, of course.

But I digress! Guile can use ICs to lazily resolve runtime routines used by compiled Scheme code. But perhaps this isn't optimal, as the set of primitive runtime calls that Guile will embed in its output is finite, and so resolving these routines eagerly would probably be sufficient. Guile could use ICs for inter-module references as well, and these should indeed be resolved lazily; but I don't know, perhaps the current strategy of using a call-site cache for inter-module references is sufficient.

Fourthly (are you counting?), there is a general case of the former: when you see a call `(a b)` and you don't know what `a` is. If you put an inline cache in the call, instead of having to emit checks that `a` is a heap object and a procedure and then emit an indirect call to the procedure's code, you might be able to emit simply a check that `a` is the same as `x`, the only callee you ever saw at that site, and in that case you can emit a direct branch to the function's code instead of an indirect branch.

Here I think the argument is less strong. Modern CPUs are already very good at indirect jumps and well-predicted branches. The value of a devirtualization pass in compilers is that it makes the side effects of a virtual method call concrete, allowing for more optimizations; avoiding indirect branches is good but not necessary. On the other hand, Guile does have polymorphic callees ( [generic functions](https://www.gnu.org/software/guile/docs/docs-2.0/guile-ref/Methods-and-Generic-Functions.html#Methods-and-Generic-Functions)), and call ICs could help there. Ideally though we would need to extend the language to allow generic functions to feed back to their inline cache handlers.

Finally, ICs could allow for cheap tracepoints and breakpoints. If at every breakable location you included a `jmp *loc`, and the initial value of `*loc` was the next instruction, then you could patch individual locations with code to run there. The patched code would be responsible for saving and restoring machine state around the instrumentation.

Honestly I struggle a lot with the idea of debugging native code. GDB does the least-overhead, most-generic thing, which is patching code directly; but it runs from a separate process, and in Guile we need in-process portable debugging. The debugging use case is a clear area where you want adaptive optimization, so that you can emit debugging ceremony from the hottest code, knowing that you can fall back on some earlier tier. Perhaps Guile should bite the bullet and go this way too.

**implementation plan**

In Guile, monomorphic as it is in most things, probably only arithmetic is worth the trouble of inline caches, at least in the short term.

Another question is how much to specialize the inline caches to their call site. On the extreme side, each call site could have a custom calling convention: if the first operand is in register A and the second is in register B and they are expected to be fixnums, and the result goes in register C, and the continuation is the code at L, well then you generate an inline cache that specializes to all of that. No need to shuffle operands or results, no need to save the continuation (return location) on the stack.

The opposite would be to call ICs as if their were normal procedures: shuffle arguments into fixed operand registers, push a stack frame, and when the IC returns, shuffle the result into place.

Honestly I am looking mostly to the simple solution. I am concerned about code and heap bloat if I specify to every last detail of a call site. Also maximum speed comes with an adaptive optimizer, and in that case simple lower tiers are best.

**sanity check**

To compare these impressions, I took a look at V8's current source code to see where they use ICs in practice. When I worked on V8, the compiler was entirely different -- there were two tiers, and both of them generated native code. Inline caches were everywhere, and they were gnarly; every architecture had its own implementation. Now in V8 there are two tiers, not the same as the old ones, and the lowest one is a bytecode interpreter.

As an adaptive optimizer, V8 doesn't need breakpoint ICs. It can always deoptimize back to the interpreter. In actual practice, to debug at a source location, V8 will patch the bytecode to insert a "DebugBreak" instruction, which has its own support in the interpreter. V8 also supports optimized compilation of this operation. So, no ICs needed here.

Likewise for generic type feedback, V8 records types as data rather than in the classic formulation of inline caches as in Self. I think WebKit's JavaScriptCore uses a similar strategy.

V8 does use inline caches for property access (loads and stores). Besides that there is an inline cache used in calls which is just used to record callee counts, and not used for direct call optimization.

Surprisingly, V8 doesn't even seem to use inline caches for arithmetic (any more?). Fair enough, I guess, given that JavaScript's numbers aren't very polymorphic, and even with a system with fixnums and heap floats like V8, floating-point numbers are rare in cold code.

The dynamic linking and relocation points don't apply to V8 either, as it doesn't receive binary code from the internet; it always starts from source.

**twilight of the inline cache**

There was a time when inline caches were recommended to solve all your VM problems, but it would seem now that their heyday is past.

ICs are still a win if you have named property access on objects whose shape you don't know at compile-time. But improvements in CPU branch target buffers mean that it's no longer imperative to use ICs to avoid indirect branches (modulo Spectre v2), and creating direct branches via code-patching has gotten more expensive and tricky on today's targets with concurrency and deep cache hierarchies.

Besides that, the type feedback component of inline caches seems to be taken over by explicit data-driven call-site caches, rather than executable inline caches, and the highest-throughput tiers of an adaptive optimizer burn away inline caches anyway. The pressure on an inline cache infrastructure now is towards simplicity and ease of type and call-count profiling, leaving the speed component to those higher tiers.

In Guile the bounded polymorphism on arithmetic combined with the need for ahead-of-time compilation means that ICs are probably a code size and execution time win, but it will take some engineering to prevent the calling convention overhead from dominating cost.

Time to experiment, then -- I'll let y'all know how it goes. Thoughts and feedback welcome from the compilerati. Until then, happy hacking :)

## related articles

- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [two paths, one peak: a view from below on high-performance language implementations](/archives/2015/11/03/two-paths-one-peak-a-view-from-below-on-high-performance-language-implementations)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [lightening run-time code generation](/archives/2019/05/24/lightening-run-time-code-generation)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)

### One response

1. Iliyan says:[7 February 2018 5:09 PM](#c515261d61f9a6c2fc108202d132a8e76f35a140)

   Hi, in "Concretely, in an expression like (a b), in which a is not a macro, a must evaluate to the result of a lambda expression.", I think you meant to say "...a must evaluate to a lambda expression."

Comments are closed.
