---
title: what does v8 do with that loop? — wingolog
url: https://wingolog.org/archives/2011/06/08/what-does-v8-do-with-that-loop
published: "2011-06-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/06/08/what-does-v8-do-with-that-loop
---

# what does v8 do with that loop? — wingolog

## [what does v8 do with that loop?](/archives/2011/06/08/what-does-v8-do-with-that-loop)

8 June 2011 2:33 PM

- [v8](/tags/v8)
- [javascript](/tags/javascript)
- [compilers](/tags/compilers)
- [self](/tags/self)
- [igalia](/tags/igalia)
- [guile](/tags/guile)

Hi!

I've spent the last month or so swimming in V8. Not the [tomato juice](http://www.v8juice.com/), mind you, though that would be pleasant, but rather the [JavaScript implementation](http://code.google.com/p/v8).

In [my last dispatch](//wingolog.org/archives/2011/05/18/value-representation-in-javascript-implementations) I made the assertion that the current JavaScript implementations are totally undocumented, but in the case of V8 that's not precisely true. There were at least two PhD dissertations written on it, along with a few technical reports and conference proceedings.

I refer of course to the Self work at Stanford and Sun, back in the late eighties and early nineties. I had read a number of their papers, but hadn't seen [Hölzle's thesis](http://research.sun.com/self/papers/hoelzle-thesis.ps.gz) yet. That one, for me, is the best of all, because it explains the Self implementation on an engineering level. The V8 implementation, I mean to say, because the vocabulary is entirely the same: maps, on-stack replacement, lazy deoptimization, etc. These are all parts of V8 now.

And it's no wonder, really. Lars Bak, the lead developer of V8, was there back then, hacking on Self, and then [Strongtalk](http://www.strongtalk.org/history.html), then [HotSpot](http://en.wikipedia.org/wiki/HotSpot), then a little startup built around virtual machines on mobile devices. So you see there's a reason why V8 doesn't have very much big-picture internal documentation -- Bak has been writing the same program for 20 years now; he knows how it works.

(Note however that Bak's name never appears in the V8 commit logs. You can see his hand at work, but never his face. Like [Dr. Claw](http://www.progressiveboink.com/archive/drclaw.html). Actually not very much like Dr. Claw but I just wanted to put that out there. Is Lars Bak Dr. Claw? Is he?)

[![](//wingolog.org/pub/madcatpet.gif)](http://www.progressiveboink.com/archive/drclaw.html)

**enough with the personification**

As you might recall, V8 always compiles JavaScript to native code. The first time V8 sees a piece of code, it compiles it quickly but without optimizing it. The initial unoptimized code is fully general, handling all of the various cases that one might see, and also includes some type-feedback code, recording what types are being seen at various points in the procedure.

At startup, if the "Crankshaft" optimizing compiler is enabled, V8 spawns off a profiling thread. If it notices that a particular unoptimized procedure is hot, it collects the recorded type feedback data for that procedure and uses it to compile an optimized version of the procedure. The old unoptimized code is then replaced with the new optimized code, and the process continues. (This on-stack replacement (OSR) process is covered in Hölzle's thesis.)

The optimized procedure does not actually cover all cases. If a callee procedure starts returning floating-point values where it was returning integer values before, the optimized procedure is deoptimized \-\- a relatively expensive process of recompiling the original procedure to the unoptimized form, replacing the optimized function on the stack with the unoptimized version, and then continuing the computation. This requires that the compiler keep around additional information about *how* to continue the computation -- you could be in the middle of a loop, after all.

Deoptimization is particularly tricky if the optimizing process inlined a callee procedure, and thus has to de-inline, replacing the activation of one optimized procedure call with two or more unoptimized calls. Again, Hölzle's thesis discusses this in depth. Inlining is such a win though that the complexity appears to be worth it.

**assembly**

I wanted to see what V8 does with a simple loop, but one for which lexical inlining isn't possible. Like this:

```
function g () { return 1; }
function f () {
  var ret = 0;
  for (var i = 1; i < 10000000; i++) {
    ret += g ();
  }
  return ret;
}

```

Pretty simple: adding 1 to a counter, 10 million times, but attempting to foil statically apparent inlining possibilities. I entered these two definitions at the `d8` shell, invoked with the `--code_comments --print_code` options.

Note that running V8 in this way is going to spew a lot on your console, as V8 itself warms up. Startup is quick, though. On my modern laptop with an SSD, the debugging shell takes about 17ms to start up. The standard shell takes about 5ms to start. Both of these numbers are with snapshots on; without snapshots, the numbers are more like 32ms and 18ms, respectively. Just for comparison, the JavaScriptCore shell ( `jsc`) takes about 12ms to start here.

Interestingly, V8's profiler decides that the best thing to do here is not to optimize `g` \-\- which it actually can't, as it's so small the unoptimized code is already optimal -- but to inline `g` into `f`, and optimize `f`.

V8 is able to do this inlining because it keeps around the parsed AST of every piece of code it sees. It needs some high-level internal representation, and it turns out that the AST is the easiest one: it's fairly small, it's already the input format to the "full-codegen" unoptimized compiler, and it also has all of the lexical information necessary to do good inlining analysis. Much easier than trying to decompile bytecode, it seems to me.

I did say that I was going to show some assembly, so here we go. This is what `d8` prints out when evaluating `f()`. I've trimmed the output a bit. The comments on the right are my annotations.

```
Instructions (size = 466)
  0  push rbp           ;; Save the frame pointer.
  1  movq rbp,rsp       ;; Set the new frame pointer.
  4  push rsi           ;; Save the callee's "context object".
  5  push rdi           ;; Save the callee's JSFunction object.
  6  subq rsp,0x28      ;; Reserve space for 5 locals.

```

Here we have a standard prelude. The JavaScript calling conventions in V8 pass arguments on the stack, using `rbp` and `rsp` as callee-saved stack and frame pointers. Additionally, information associated with the function itself is passed in `rsi` and `rdi`: the context, and the function object itself. The context is an array optimized for looking up various information that the function needs at runtime, mostly free variables (lexical and global).

In this case it's redundant to take the context out of the function, but it does allow for faster access to the global scope object that was current when the function was defined. In the case of closures, every time the function() expression is evaluated, a new context will be created with a new set of free variables.

Anyway! Moving along:

```
 10  movq rax,rsi        ;; Store the context in a scratch register.
 13  movq [rbp-0x38],rax ;; And save it in the last (4th) stack slot.
 17  cmpq rsp,[r13+0x0]  ;; Check for stack overflow
 21  jnc 28              ;; If we overflowed,
 23  call 0x7f7b20f40a00 ;; Call the overflow handler.

```

Here we store the context again, in the last stack slot, and we check for overflow. On x86-64, V8 reserves `r13` for a pointer to somewhere in the middle of a "global object", holding the GC roots for a given JavaScript execution context. There is a cell in the root list that holds the stack limit, which V8 abuses for other purposes: interrupting loops, breakpoints, deoptimization, etc. Again, Hölzle's thesis has some details on this technique.

The "somewhere in the middle" bit is arranged so that a simple dereference of r13 will allow us to get at the stack limit. V8 will reset the stack limit whenever it needs to interrupt a procedure.

Having passed the stack overflow check, we initialize local variables:

```
 28  movq rax,[rbp+0x10] ;; Receiver object to rcx (unused).
 32  movq rcx,rax        ;;
 35  movq rdx,[rbp-0x38] ;; Global objectcontext to rdx.
 39  movl rbx,(nil)      ;; Loop counter (i) to 0.
 44  movl rax,(nil)      ;; Accumulator (ret) to 0.
 49  jmp 97              ;; Jump over some stuff.

```

Note that the receiver object (the `this` object) is passed as an argument, and thus is above `rbp`.

Following this bit we have some code that appears to be completely dead. I'll include it for completeness, but unless the deoptimization bailouts jump back here, I don't know why it's there.

```
 54  movq rax,rsi        ;; Dead code.
 57  movq rbx,[rbp-0x28]
 61  testb rbx,0x1
 64  jnz 189  (0x7f7b20fa2a7d)
 70  shrq rbx,32
 74  movq rdx,[rbp-0x30]
 78  testb rdx,0x1
 81  jnz 237  (0x7f7b20fa2aad)
 87  shrq rdx,32
 91  movq rcx,[rbp-0x18]
 95  xchgq rax, rdx

```

We'll just forget that happened, shall we? However we are getting closer to the firm tofu of the matter, the loop. First one more check:

```
 97  movq rdx,[rsi+0x2f] ;; Slot 6 of the context: the global object.
101  movq rdi,0x7f7b20e401e8 ;; Location of cell holding `g'
111  movq rdi,[rdi]      ;; Dereference cell
114  movq r10,0x7f7b205d7ba1 ;; The expected address of `g'
124  cmpq rdi,r10        ;; If they're not the same...
127  jnz 371             ;; Deoptimization bailout 2

```

Here we see if the current definition of `g`, the function that we inlined below, is actually the same as when the inlining was performed.

Note that on line 97, since [pointers in V8 are low-bit tagged with `01`](//wingolog.org/archives/2011/05/18/value-representation-in-javascript-implementations), to access slot 6 (0-based) of the context object, we only need to add `0x2f` instead of `0x30`. Clever, right? But we don't actually need the global object here in the main loop, so we could have delayed this load until finding out that deoptimization was necessary. Perhaps it was needed though for GC reasons.

```
133  movq rdx,[rdx+0x27] ;; Another redundant load.
137  cmpl rbx,0x989680   ;; 10000000, you see.
143  jge 178             ;; If i >= 10000000, break.
149  movq rdx,rax        ;; tmp = ret
152  addl rdx,0x1        ;; tmp += 1
155  jo 384              ;; On overflow, deoptimize.
161  addl rbx,0x1        ;; i++
164  movq rax,rdx        ;; ret = tmp
167  cmpq rsp,[r13+0x0]  ;; Reload stack limit.
171  jnc 137             ;; Loop if no interrupt,
173  jmp 306             ;; Otherwise bail out.
178  shlq rax,32         ;; Tag rax as a small integer.
182  movq rsp,rbp        ;; Restore stack pointer.
185  pop rbp             ;; Restore frame pointer.
186  ret 0x8             ;; Return, popping receiver.

```

And that's it! It's a fairly tight loop: `g` is inlined of course, its return value is untagged to a native int32, as are the accumulator and loop counter variables. Of course, improvements are possible -- the loop could be unrolled a few times, range analysis could avoid the deoptimization check, the overflow check could possibly be cheaper, and indeed the whole thing could be folded, but all in all, good job, MAD kittens!

Note the interesting approach to tagging: instead of storing the integer in the lower 32 bits, shifted by one, it is stored in the upper 32 bits, without a shift.

Actually there's some more junk after all of this. Another dead code block, apparently meaning to deal with floating-point values, but totally unreferenced:

```
189  movq r10,[r13-0x38]
193  cmpq [rbx-0x1],r10
197  jnz 397
203  movsd xmm0,[rbx+0x7]
208  cvttsd2sil rbx,xmm0
212  cvtsi2sd xmm1,rbx
216  ucomisd xmm0,xmm1
220  jnz 397
226  jpe 397
232  jmp 74
237  movq r10,[r13-0x38]
241  cmpq [rdx-0x1],r10
245  jnz 410
251  movsd xmm0,[rdx+0x7]
256  cvttsd2sil rdx,xmm0
260  cvtsi2sd xmm1,rdx
264  ucomisd xmm0,xmm1
268  jnz 410
274  jpe 410
280  testl rdx,rdx
282  jnz 301
288  movmskpd xmm2,xmm0
292  andl rdx,0x1
295  jnz 410
301  jmp 91

```

And then there's our stack check handler, saving all of the registers of interest, and calling out:

```
306  push rax
307  push rcx
308  push rdx
309  push rbx
310  push rsi
311  push rdi
312  push r8
314  push r9
316  push r11
318  push r14
320  push r15
322  leaq rsp,[rsp-0x28]
327  movq rsi,[rbp-0x8]
331  xorl rax,rax
333  leaq rbx,[r13-0x20a4b70]
340  call 0x7f7b20fa25c0
345  leaq rsp,[rsp+0x28]
350  pop r15
352  pop r14
354  pop r11
356  pop r9
358  pop r8
360  pop rdi
361  pop rsi
362  pop rbx
363  pop rdx
364  pop rcx
365  pop rax
366  jmp 137

```

And finally the deoptimization bailouts:

```
371  movq r10,0x7f7b20f7c054
381  jmp r10
384  movq r10,0x7f7b20f7c05e
394  jmp r10
397  movq r10,0x7f7b20f7c068
407  jmp r10
410  movq r10,0x7f7b20f7c072
420  jmp r10

```

Whew! I'd say that's long enough for today. I wanted to establish a fixed point on the low level though, so that I could write in the future about how it is that V8 compiles down to this code, what deoptimization is about, and such; filling in the middle, between the assembly and the JavaScript source. Corrections and suggestions in the comments please. Happy hacking!

## related articles

- [v8: a tale of two compilers](/archives/2011/07/05/v8-a-tale-of-two-compilers)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [on-stack replacement in v8](/archives/2011/06/20/on-stack-replacement-in-v8)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)

### 10 responses

1. [RubenV](http://www.savanne.be) says:[8 June 2011 4:32 PM](#76a6e16d9a82688177676da5eb9dc733e35d5d48)

   Good stuff Andy! There's some pretty sweet things going on V8 it seems!

2. Federico Mena Quintero says:[8 June 2011 4:47 PM](#de8f0c36af58efaccdca5484bd755c8cbde9b6c2)

   Keep it up, Andy! These articles are very interesting. I have zero experience with code generation, but your articles make me feel smarter already.

3. Kasper Lund says:[8 June 2011 7:53 PM](#f6afc00bfdaf85a93bb13be0217251519ecc0cd3)

   The "dead" code from offset 54 is actually used to implement on-stack replacement -- and it is needed because V8 may decide to optimize a hot function while it's busy running inside a loop (it clearly did in this case). The code isn't dead at all -- and consequently the code from offset 189 isn't dead either. I encourage you to try running with the --trace-osr option enabled. Enjoy!

4. [Grant Galitz](http://www.grantgalitz.org/) says:[9 June 2011 6:52 AM](#e3a5a6cc889d7aeb030b9f7e46f425ead8a9271e)

   Good Stuff. Seeing assembly code at work is always interesting.

5. Volkan Yazıcı says:[9 June 2011 7:18 AM](#0316bfe6393f250975390e19c4001d98690dfb60)

   Thanks! Again a well-written, informative entry. Are there any plans to integrate a similar feature into Guile?

6. [Peter van der Zee](http://qfox.nl) says:[9 June 2011 2:41 PM](#6edbcd9824b4036fd0629985e241b454900faef6)

   Thank you, I really liked this article. Looking forward to more articles on v8 and tracing/jit :)

7. [Ludovic Courtès](http://chbouib.org/) says:[9 June 2011 4:06 PM](#f24ab81d68f91a7361feec34f71b913d2fc28ceb)

   Interesting stuff, though a bit scary to me (deoptimization...)!

   Psyco does dynamic optimization based on observed types, too, with interesting short papers describing the approach.

8. [Andy Wingo](http://wingolog.org/) says:[9 June 2011 5:26 PM](#37d30ead536beaefeb03a128034d72bad20da885)

   Thanks for the feedback, all. And thanks for the pointer, Kasper! I'll correct my error in a future installment :)

9. [Sytse Sijbrandij](http://www.sytse.com/) says:[10 June 2011 5:55 PM](#632f1ad6eefc1bece7a4c7a1d4681396fed40d67)

   Interesting article and thank you for making it understandable and enjoyable for people who don't have much assembly background.

10. Andreas says:[11 June 2011 2:47 PM](#e4b6bc35794b734849fec228b400c59782760d42)

    "Note however that Bak's name never appears in the V8 commit logs \[..\]"

    Who do you think bak AT chromium DOT org is?

Comments are closed.
