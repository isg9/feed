---
title: compost, a leaf function compiler for guile — wingolog
url: https://wingolog.org/archives/2014/02/18/compost-a-leaf-function-compiler-for-guile
published: "2014-02-18T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2014/02/18/compost-a-leaf-function-compiler-for-guile
---

# compost, a leaf function compiler for guile — wingolog

## [compost, a leaf function compiler for guile](/archives/2014/02/18/compost-a-leaf-function-compiler-for-guile)

18 February 2014 8:21 PM

- [compilers](/tags/compilers)
- [assembly](/tags/assembly)
- [elf](/tags/elf)
- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [gl](/tags/gl)
- [potluck](/tags/potluck)
- [the void](/tags/the%20void)

What's that out by the woodshed? It's a steaming pile -- it's full of bugs -- it's compost, a leaf function compiler for Guile!

Around this time last year, a few of us cooked up some hack-dishes to bring to a potluck for Guile 2.0's release anniversary. Last year, mine was a little [OpenGL particle demo](//wingolog.org/archives/2013/02/16/opengl-particle-simulation-in-guile).

That demo was neat but it couldn't be as big as I would have liked it to be because it was too slow. So, this year when the potluck season rolled around again I sat down to make a little compiler for the subset of Scheme that you see in inner numeric loops -- bytevector access, arithmetic, and loops.

The result is [compost](https://gitorious.org/guile-compost/). Compost compiles inner loops into native x86-64 machine code that operates on unboxed values.

As you would imagine, compost-compiled code is a lot faster than code interpreted by Guile's bytecode interpreter. I go from being able to compute and render 5K particles at 60 fps up to 400K particles or so -- an 80-fold improvement. That's swell but it gets sweller. The real advantage is that with fast inner loops, I can solve more complicated problems.

Like this one!

(Videos on the internet are a surprisingly irritating problem. Maybe because it's not a cat? Check [wingolog.org/pub/](//wingolog.org/pub/) for various other versions of 1300-bodies-512x416 if that doesn't play for you.)

Last year's demo hard-coded a gravitational attractor at (0, 0, 0). This one has no hard-coded attractor -- instead, each particle attracts each other. This so-called [n-body simulation](http://en.wikipedia.org/n-body_simulation) is an n-squared problem, so you need to be really efficient with the primitives to scale up, and even then the limit approaches quickly.

With compost, I can get to about 1650 particles at 60 frames per second, using 700% CPU on this 4-core 2-thread-per-core i7-3770 machine, including display with the free software radeon drivers. Without compost -- that is to say, just with Guile's bytecode virtual machine -- I max out at something more like 120 particles, and only 200% CPU.

The rest of this post describes how compost works. If compilers aren't your thing, replace the rest of the words with cat noises.

**meow meow meow meow meow meow meow meow**

The interface to compost is of course a macro, `define/compost`. Here's a little loop to multiply two vectors into a third, element-wise:

```
(use-modules (compost syntax) (rnrs bytevectors))
(define/compost (multiply-vectors (dst bytevector?)
                                  (a bytevector?)
                                  (b bytevector?)
                                  (start exact-integer?)
                                  (end exact-integer?))
  (let lp ((n start))
    (define (f32-ref bv n)
      (bytevector-ieee-single-native-ref bv (* n 4)))
    (define (f32-set! bv n val)
      (bytevector-ieee-single-native-set! bv (* n 4) val))
    (when (< n end)
      (f32-set! dst n (* (f32-ref a n) (f32-ref b n)))
      (lp (1+ n)))))

```

It's low-level but that's how we roll. If you evaluate this form and all goes well, it prints out something like this at run-time:

```
;;; loading /home/wingo/.cache/guile/compost/rmYZoT-multiply-vectors.so

```

This indicates that compost compiled your code into a shared object at macro-expansion time, and then printed out that message when it loaded it at runtime. If composting succeeds, compost writes out the compiled code into a loadable shared object ( `.so` file). It then residualizes a call to `dlopen` to load that file at run-time, followed by code to look up the `multiply-vectors` symbol and create a foreign function. If composting fails, it prints out a warning and falls back to normal Scheme (by residualizing a plain lambda).

In the beginning of the article, I called compost a "leaf function compiler". Composted functions should be "leaf functions" -- they shouldn't call other functions. This restriction applies only to the low-level code, however. The first step in composting is to run the function through Guile's normal source-to-source optimizers, resulting in a [CPS term](//wingolog.org/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile). The upshot is that you can use many kinds of abstraction inside the function, like the little `f32-ref`/ `f32-set!` helpers above, but in the end Guile should have inlined or contified them all away. It's a restriction, but hey, this is just a little hack.

Let's look at some assembly. We could get disassembly just by calling `objdump -d /home/wingo/.cache/guile/compost/rmYZoT-multiply-vectors.so`, but let's do it a different way. Let's put that code into a file, say "/tmp/qux.scm", and add on this code at the end:

```
(define size #e1e8) ;; 100 million
(define f32v (make-f32vector size 2.0))
(multiply-vectors f32v f32v f32v 0 size)

```

OK. Now we run Guile under GDB:

```
$ gdb --args guile /tmp/qux.scm
(gdb) b 'multiply-vectors'
Function "multiply-vectors" not defined.
Make breakpoint pending on future shared library load? (y or [n]) y
Breakpoint 1 ('multiply-vectors') pending.
(gdb) r
Starting program: /opt/guile/bin/guile /tmp/qux.scm
[New Thread 0x7ffff604b700 (LWP 13729)]
;;; loading /home/wingo/.cache/guile/compost/Kl0Xpc-multiply-vectors.so

Breakpoint 1, 0x00007ffff5322000 in multiply-vectors () from /home/wingo/.cache/guile/compost/Kl0Xpc-multiply-vectors.so
(gdb) step
multiply-vectors () at /tmp/qux.scm:12
12	    (when (< n end)

```

Word! GDB knows about the symbol, `multiply-vectors`. That's top. We are able to step into it, and it prints Scheme code!

Both of these swell things are because compost residualizes its compiled code as [ELF files](//wingolog.org/archives/2014/01/19/elf-in-guile), and actually re-uses Guile's linker. The ELF that we generate can be loaded by `dlopen`, and its symbol tables and DWARF debugging information are known to GDB.

(In my last article on ELF I mentioned that Guile had no plans to use the system dynamic library loader ( `dlopen`). That's still true; Guile has its own loader. I used the system loader in this place, though, just because I thought it was a neat hack.)

We can tell GDB to disassemble the next line:

```
(gdb) set disassemble-next-line on
(gdb) step
9	      (bytevector-ieee-single-native-ref bv (* n 4)))
=> 0x00007ffff532201d :	4c 0f af c9	imul   %rcx,%r9
(gdb)
13	      (f32-set! dst n (* (f32-ref a n) (f32-ref b n)))
=> 0x00007ffff532203b :	f2 0f 59 c1	mulsd  %xmm1,%xmm0
   0x00007ffff532203f :	49 b9 04 00 00 00 00 00 00 00	movabs $0x4,%r9

```

GDB does OK with these things, but it doesn't have special support for Scheme, and really you would want column pointers, not just lines. That data is in the DWARF but it's not processed by GDB. Anyway here's the disassembly:

```
(gdb) disassemble
Dump of assembler code for function multiply-vectors:
   0x00007ffff5322000 <+0>:	push   %rbx
   0x00007ffff5322001 <+1>:	push   %rbp
   0x00007ffff5322002 <+2>:	push   %r12
   0x00007ffff5322004 <+4>:	push   %r13
   0x00007ffff5322006 <+6>:	push   %r14
   0x00007ffff5322008 <+8>:	push   %r15
   0x00007ffff532200a <+10>:	cmp    %r8,%rcx
   0x00007ffff532200d <+13>:	jge    0x7ffff5322060
   0x00007ffff5322013 <+19>:	movabs $0x4,%r9
   0x00007ffff532201d <+29>:	imul   %rcx,%r9
   0x00007ffff5322021 <+33>:	cvtss2sd (%rsi,%r9,1),%xmm0
   0x00007ffff5322027 <+39>:	movabs $0x4,%r9
   0x00007ffff5322031 <+49>:	imul   %rcx,%r9
   0x00007ffff5322035 <+53>:	cvtss2sd (%rdx,%r9,1),%xmm1
=> 0x00007ffff532203b <+59>:	mulsd  %xmm1,%xmm0
   0x00007ffff532203f <+63>:	movabs $0x4,%r9
   0x00007ffff5322049 <+73>:	imul   %rcx,%r9
   0x00007ffff532204d <+77>:	cvtsd2ss %xmm0,%xmm15
   0x00007ffff5322052 <+82>:	movss  %xmm15,(%rdi,%r9,1)
   0x00007ffff5322058 <+88>:	inc    %rcx
   0x00007ffff532205b <+91>:	jmpq   0x7ffff532200a
   0x00007ffff5322060 <+96>:	movabs $0x804,%rdi
   0x00007ffff532206a <+106>:	mov    %rdi,%rax
   0x00007ffff532206d <+109>:	pop    %r15
   0x00007ffff532206f <+111>:	pop    %r14
   0x00007ffff5322071 <+113>:	pop    %r13
   0x00007ffff5322073 <+115>:	pop    %r12
   0x00007ffff5322075 <+117>:	pop    %rbp
   0x00007ffff5322076 <+118>:	pop    %rbx
   0x00007ffff5322077 <+119>:	retq
End of assembler dump.
(gdb)

```

Now if you know assembly, this is pretty lame stuff -- it saves registers it doesn't use, it multiplies instead of adds to get the bytevector indexes, it loads constants many times, etc. It's a proof of concept. Sure beats heap-allocated floating-point numbers, though.

**safety and semantics**

Compost's goal is to match Guile's semantics, while processing values with native machine operations. This means that it needs to assign concrete types and representations to all values in the function. To do this, it uses the preconditions, return types from primitive operations, and types from constant literals to infer other types in the function. If it succeeds, it then chooses representations (like "double-precision floating point") and assigns values to registers. If the types don't check out, or something is unsupported, compilation bails and runtime will fall back on Guile's normal execution engine.

There are a couple of caveats, however.

One is that compost assumes that small integers do not promote to bignums. We could remove this assumption with better range analysis. Compost does do some other analysis, like sign analysis to prove that the result of `sqrt` is real. Complex numbers will cause compost to bail.

Compost also doesn't check bytevector bounds at run-time. This is terrible. I didn't do it though because to do this nicely you need to separate the bytevector object into two variables: the pointer to the contents and the length. Both should be register-allocated separately, and range analysis would be nice too. Oh well!

Finally, compost is really limited in terms of the operations it supports. In fact, if register allocation would spill on the function, it bails entirely :)

**the source**

If it's your thing, have fun [over on yon gitorious](https://gitorious.org/guile-compost/). Compost needs Guile from git, and the demos need Figl, the GL library. For me this project is an ephemeral thing; a trial for future work, with some utility now, but not something I really want to support. Still, if it's useful to you, have at it.

**coda**

I woke up this morning at 5 thinking about the universe, and how friggin big it is. I couldn't go back to sleep. All these particles swirling and interacting in parallel, billions upon zillions, careening around, somehow knowing what forces are on them, each somehow a part of the other. I studied physics and I never really boggled at the universe in the way I did this morning thinking about this n-body simulation. Strange stuff.

I remember back in college when I was losing my catholicism and I would be at a concert or a dance show or something and I would think "what is this, this is just particles vibrating, bodies moving in the nothing; there is no meaning here." It was a thought of despair and unmooring. I don't suffer any more over it, but mostly because I don't think about it any more. I still don't believe in some omniscient skydude or skylady, but if I did, I know he or she has got a matrix somewhere with every particle's position and velocity.

Swirl on, friends, and until next time, happy hacking.

## related articles

- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [elf in guile](/archives/2014/01/19/elf-in-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [dltool mines dwarf](/archives/2012/06/19/dltool-mines-dwarf)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)

### 2 responses

1. John Cowan says:[19 February 2014 9:48 PM](#eab4368e1f0dd87b6f7ff461f4f566698a89b34d)

   I've got a currently back-burnered project called Flopsy (as in *Peter Rabbit*) that's a little bit like this. It's specifically about floating-point code, which is usually extremely unoptimized. It processes a subset of Scheme and generates C; there's a small library consisting of some R7RS things plus SRFI 43, the vector library. The only datatypes are doubles, vectors of doubles, booleans, fixnums, and constant strings (for debug I/O, basically).

   However, it doesn't do type inference: it extends Scheme's ? and ! conventions into a full postposed sigil system. No sigil is double, \* is vector of doubles, ? is boolean, % is fixnum (yes, from Basic-Plus), $ is string (ditto), and ! is void (returned from procedures). The length of a vector v is stored at v\[-1\], from the C perspective.

   The result is vanilla code that you can compile with an AOT compiler and call from any Scheme via its FFI.

2. [Arne Babenhauserheide](http://draketo.de) says:[7 August 2014 4:56 PM](#918a5165208793b2de371cf59fd683308f4c4f18)

   I did not write it here yet, though I thought it everytime I read this article: This is awesome!

   I hope that some day general Guile code will benefit from things you played with in compost.

Comments are closed.
