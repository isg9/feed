---
title: a register vm for guile — wingolog
url: https://wingolog.org/archives/2013/11/26/a-register-vm-for-guile
published: "2013-11-26T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2013/11/26/a-register-vm-for-guile
---

# a register vm for guile — wingolog

## [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)

26 November 2013 10:07 PM

- [guile](/tags/guile)
- [compilers](/tags/compilers)
- [scheme](/tags/scheme)
- [gnu](/tags/gnu)
- [igalia](/tags/igalia)
- [register vm](/tags/register%20vm)
- [vm](/tags/vm)
- [stack vm](/tags/stack%20vm)
- [javascriptcore](/tags/javascriptcore)
- [v8](/tags/v8)
- [scheme](/tags/scheme)
- [javascript](/tags/javascript)

Greetings, hacker comrades! Tonight's epistle is gnarly nargery of the best kind. See, we just landed a new virtual machine, compiler, linker, loader, assembler, and debugging infrastructure in [Guile](http://gnu.org/s/guile/), and stories like that don't tell themselves. Oh no. I am a firm believer in [Steve Yegge's Big Blog Theory](http://steve-yegge.blogspot.fr/2008/01/blogging-theory-201-size-does-matter.html). There are nitties and gritties and they need explication.

**a brief brief history**

As most of you know, Guile is an implementation of Scheme. It started about 20 years ago as a fork of [SCM](http://people.csail.mit.edu/jaffer/SCM.html).

I think this lines-of-code graph pretty much sums up the history:

![](//wingolog.org/pub/many-scheme.png)

That's from the [Ohloh](http://www.ohloh.net/p/guile/analyses/latest/languages_summary), in case you were wondering. Anyway the story is that in the beginning it was all C, pretty much: Aubrey Jaffer's SCM, just packaged as a library. And it was C people making it, obviously. But Scheme is a beguiling language, and over time Guile has had a way of turning C hackers into Scheme hackers.

I like to think of this graph as showing my ignorance. I started using Guile about 10 years ago, and hacking on it in 2008 or so. In the beginning I was totally convinced by the "C for speed, Scheme for flexibility" thing -- to the extent that I was willing to write off Scheme as inevitably slow. But that's silly of course, and one needs no more proof than the great performance JavaScript implementations have these days.

In 2009, we merged in a bytecode VM and a compiler written in Scheme itself. All that is pretty nifty stuff. We released that version of Guile as [2.0 in 2011](http://article.gmane.org/gmane.lisp.guile.devel/11654), and that's been good times. But it's time to move onward and upward!

A couple of years ago I wrote an [article on JavaScriptCore](//wingolog.org/archives/2011/10/28/javascriptcore-the-webkit-js-implementation), and in it I spoke longingly of register machines. I think that's probably when I started to make sketches towards Guile 2.2, after having spent time with JavaScriptCore's bytecode compiler and interpreter.

Well, it took a couple of years, but Guile 2.2 is finally a thing. No, we haven't even made any prereleases yet, but the important bits have landed in master. This is the first article about it.

**trashing your code**

Before I start trashing Guile 2.0, I think it's important to say what it does well. It has a great inlining pass -- better than any mainstream language, I think. Its startup time is pretty good -- around 13 milliseconds on my machine. Its runs faster than other "scripting language" implementations like Python (CPython) or Ruby (MRI). The debugging experience is delightful. You get native POSIX threads. Plus you get all the features of a proper Scheme, like macros and delimited continuations and all of that!

But the Guile 2.0 VM is a stack machine. That means that its instructions usually take their values from the stack, and produce values (if appropriate) by pushing values onto the stack.

The problem with stack machines is that they penalize named values. If I realize that a computation is happening twice and I factor it out to a variable, that means in practice that I allocate a stack frame slot to the value. So far so good. However, to use the value, I have to emit an instruction to fetch the value for use by some other instruction; and to store it, I likewise have to have another instruction to do that.

For example, in Guile 2.0, check out the bytecode produced for this little function:

```
scheme@(guile-user)> ,disassemble (lambda (x y)
                                    (let ((z (+ x y)))
                                      (* z z)))

   0    (assert-nargs-ee/locals 10)     ;; 2 args, 1 local
   2    (local-ref 0)                   ;; `x'
   4    (local-ref 1)                   ;; `y'
   6    (add)
   7    (local-set 2)                   ;; `z'
   9    (local-ref 2)                   ;; `z'
  11    (local-ref 2)                   ;; `z'
  13    (mul)
  14    (return)

```

This is silly. There are seven instructions in the body of this procedure, not counting the prologue and epilogue, and only two of them are needed. The cost of interpreting a bytecode is largely dispatch cost, which is linear in the number of instructions executed, and we see here that we could be some 7/2 = 3.5 times as fast if we could somehow make the operations reference their operands by slot directly.

**register vm to the rescue**

The solution to this problem is to use a "register machine". I use scare quotes because in fact this is a virtual machine, so unlike a CPU, the number of "registers" is unlimited, and in fact they are just stack slots accessed by index.

So in Guile 2.2, our silly procedure produces the following code:

```
scheme@(guile-user)> ,disassemble (lambda (x y)
                                    (let ((z (+ x y)))
                                      (* z z)))

   0    (assert-nargs-ee/locals 3 1)    ;; 2 args, 1 local
   1    (add 3 1 2)
   2    (mul 3 3 3)
   3    (return 3)

```

This is optimal! There are four things that need to happen, and there are four opcodes that do them. Receiving operands and sending values is essentially free -- they are indexed accesses off of a pointer stored in a hardware register, into memory that is in cache.

This is a silly little example, but especially in loops, Guile 2.2 stomps Guile 2.0. A simple count-up-to-a-billion test runs in 9 seconds on Guile 2.2, compared to 24 seconds in Guile 2.0. Let's make a silly graph!

![](//wingolog.org/pub/guile-billion.png)

Of course if we compare to V8 for example we find that V8 does a loop-to-a-billion in about 1 second, or 9 times faster. There is some way to go. There are a couple of ways that I could generate better bytecode for this loop, for another 30% speed boost or so, but ultimately we will have to do native compilation. And we will! But that is another post.

**gritties**

[Here's the VM.](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=libguile/vm-engine.c;h=4ae2aa7285aa28207f0bf0847c1f5f2bcc8f4df3;hb=c30edbbd5b6cfeeca6b8a66578107df17ec96a51#l430) It's hairy in the prelude, and the whole thing is `#include` d twice in another C file (for a debugging and a non-debugging mode; terrible), but I think it's OK for being in C. (If it were in C++ it could be nicer in various ways.)

The calling convention for this VM is that when a function is called, it receives its arguments on the stack. The stack frame looks like this:

```
   /------------------\
   | Local N-1        | <- sp
   | ...              |
   | Local 1          |
   | Local 0          | <- fp
   +==================+
   | Return address   |
   | Dynamic link     |
   +==================+
   :                  :

```

Local 0 holds the procedure being called. Free variables, if any, are stored inline with the (flat) closure. You know how many arguments you get by the difference between the stack pointer (SP) and the frame pointer (FP). There are a number of opcodes to bind optional arguments, keyword arguments, rest arguments, and to skip to other case-lambda clauses.

After deciding that a given clause applies to the actual arguments, a prelude opcode will reset the SP to have enough space to hold all locals. In this way the SP is only manipulated in function prologues and epilogues, and around calls.

Guile's stack is expandable: it is originally only a page or two, and it expands (via `mremap` if possible) by a factor of two on every overflow, up to a configurable maximum. At expansion you have to rewrite the saved FP chain, but nothing else points in, so it is safe to move the stack.

To call a procedure, you put it and its arguments in contiguous slots, with no live values below them, and two empty slots for the saved instruction pointer (IP) and FP. Getting this right requires some compiler sophistication. Then you reset your SP to hold just the arguments. Then you branch to the procedure's entry, potentially bailing out to a helper if it's not a VM procedure.

To return values, a procedure shuffles the return values down to start from slot 1, resets the stack pointer to point to the last return value, and then restores the saved FP and IP. The calling function knows how many values are returned by looking at the SP. There are convenience instructions for returning and receiving a single value. Multiple values can be returned on the stack easily and efficiently.

Each operation in Guile's VM consists of a number of 32-bit words. The lower 8 bits in the first word indicate the opcode. The width and layout of the operands depends on the word. For example, MOV takes two 12-bit operands. Of course, 4096 locals may not be enough. For that reason there is also LONG-MOV which has two words, and takes two 24-bit operands. In LONG-MOV there are 8 bits of wasted space, but I decided to limit the local frame address space to 24 bits.

In general, most operations cannot address the full 24-bit space. For example, there is ADD, which takes two 8-bit operands and one 8-bit destination. The plan is to have the compiler emit some shuffles in this case, but I haven't hit it yet, and it was too tricky to try to get right in the bootstrapping phase.

JavaScriptCore avoids the address space problem by having all operands be one full pointer wide. This wastes a lot of memory, but they lazily compile and can throw away bytecode and reparse from source as needed, neither of which are true for Guile. We aim to do a good ahead-of-time compilation, to enable self-hosting of the compiler.

JSC's pointer-wide operands do provide the benefit of allowing the "opcode" word to actually hold the address of the label, instead of an index to a table of addresses. This is a great trick, but again it's not applicable to Guile as we don't want to relocate bytecode that we load from disk.

Relative jumps in Guile's VM are 24 bits wide, and are measured in 32-bit units, giving us effectively a 26 bit jump space. Relative references -- references to static data, or other procedures -- are 32 bits wide. I certainly hope that four gigabytes in a compilation unit is enough! By the time it is a problem, hopefully we will be doing native compilation.

Well, those are the basics of Guile's VM. There's more to say, but I already linked to the source, so that should be good enough :) In some future dispatch, we'll talk about the other parts of Guile 2.2. Until then!

## related articles

- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [two paths, one peak: a view from below on high-performance language implementations](/archives/2015/11/03/two-paths-one-peak-a-view-from-below-on-high-performance-language-implementations)
- [on generators](/archives/2013/02/25/on-generators)
- [stranger in these parts](/archives/2012/05/16/stranger-in-these-parts)
- [JavaScriptCore, the WebKit JS implementation](/archives/2011/10/28/javascriptcore-the-webkit-js-implementation)
- [a schemer at jsconf.eu](/archives/2011/10/04/a-schemer-at-jsconf-eu)

### 20 responses

1. [Arne Babenhauserheide](http://draketo.de) says:[26 November 2013 10:29 PM](#3f1743210de627528170596b8a11e0e0ae6c2746)

   Thank you for the slight indepth view - and the very interesting article!

   The graphics actually helped to make it feel shorter than it is - I have to remember that trick ☺ (not that it is not obvious - I just tend to forget it…).

   Do the performance benefits hold for other tasks than counting upwards? In that case, the performance of guile would now be beat python quite substantially (though if I wanted to do any high performance work in python, I’d just use cython - which accounts to simply wrapping native C code in simpler syntax).

2. Renato says:[27 November 2013 0:10 AM](#9706b1381ea9b6464631c7e9b0acea4eed0b9941)

   Wow great work. I was super excited about the emacs port from bt Templeton, but it looks like isn't going to happen. :'(

3. djcb says:[27 November 2013 0:24 AM](#97dc9b582048167c6dc7fddfb814960ccc8e6aac)

   Thank you Andy for the continuing fantastic work on Guile (together with all the other contributors).

4. [Peter Bex](http://www.more-magic.net) says:[27 November 2013 10:17 AM](#be5f88bc608f74cd7e11ac64986b4cba12e45072)

   What exactly does the loop counting upward look like, and on what machine (32 or 64 bits) did you try?

   I'm asking because V8 would be using flonums all the way, and Guile might be switching to bignums, which would probably be slower in any case, so it's a bit of a strange comparison.

5. [wingo](http://wingolog.org/) says:[27 November 2013 10:30 AM](#a67ac437d56771fc00e555fea3c3f40e533100e6)

   Heya Peter,

   V8 doesn't nan-box -- it has fixnums ("smi" values). 1 billion fits in a smi. Anyway the optimizing compiler will unbox to int32, so you end up with a tight assembly loop.

   1 billion is also within the fixnum range for both 32-bit and 64-bit Guile.

   Here were the loops:

   Scheme: (let lp ((n 0)) (when (< n 1000000000) (lp (1+ n))))

   JS: function f() { for (var i = 0; i < 1000000000; i++); }

   Though both loops are dead, I'm pretty sure neither compiler is eliminating them, so I think we are testing apples and apples.

6. [Peter Bex](http://www.more-magic.net) says:[27 November 2013 10:46 AM](#fcad57dbdc01701005f57e64af36e5efced1e216)

   Hi Andy,

   Thanks for your reply. I was under the impression that the Ecmascript standard mandated all numbers be double precision floating-point, or is using fixnums where possible a V8 extension/optimisation?

7. [wingo](http://wingolog.org/) says:[27 November 2013 10:57 AM](#35b67680c5377383ff3e9c9e04298dc5c6ce0257)

   JS numbers are semantically doubles. In some cases they are truncated to integers in a certain range -- for example for bit-shift operators. But generally an engine is free to choose any representation it likes for numbers, as long as the semantics are preserved. In that regard, fixnums that overflow to heap flonums is a valid and potentially profitable implementation strategy. But much better is to use runtime type feedback to compile optimized versions of functions or loops that choose native representations -- unboxed integers and doubles in the appropriate register. All engines do this. I'm pretty sure that the loop above is getting optimized in this way.

8. [Peter Bex](http://www.more-magic.net) says:[27 November 2013 11:01 AM](#c73ed2a91f430e70a3e020518026a9b278f66839)

   Hm, that's very interesting. This overflow detection must check for 54 bit overflow (instead of 64 bit overflow), to preserve the weird number jumps you get in floating point arithmetic. If that's not done, you should be able to differentiate implementations that use this optimizations from those that don't.

   It all sounds awfully hacky :)

9. [wingo](http://wingolog.org/) says:[27 November 2013 11:07 AM](#9ec7da8bef771e74a352e0d906fc63eee0845435)

   Overflow usually happens at the 32-bit level, from 32-bit fixnum (in 64-bit case where other 32 bits are the tag) or 32-bit fixnum-and-tag-bit. The whole numbers-are-doubles thing is the hack; all else are mitigations :)

10. [John Cowan](http://www.ccil.org/~cowan) says:[27 November 2013 10:55 PM](#9749d8335a8e39a659fc6b0c9002e1efec5ac3c6)

    "Die Consen und Fixnumen hat der liebe McCarthy gemacht, alles andere ist Hackerenwerk." —Henry Baker (slightly edited), after Kronecker

11. Snoopy says:[1 December 2013 11:24 AM](#dc576255c2e245c4000b25eb30f1dc42c4dc2536)

    "Well, those are the basics of Guile's VM. There's more to say, but I already linked to the source, so that should be good enough :)"

    Er... the Guile 2.0 documentation is excellent, I hope that Guile 2.2 will carry on this tradition.

12. tomás zerolo says:[7 December 2013 5:55 PM](#e521f7954c3ee8de28c28235580e549ceb4bfd94)

    Absolutely exciting.

    Sorry for this "me too" post, but there, it had to be said. Thanks, Andy!

13. Brandon says:[10 December 2013 9:36 PM](#0e0fd1ddf250ea5bf377dca52d0ef0abca27d809)

    So can this be viewed as an incremental step toward native compilation? Are you eventually going to target a compiler back-end like LLVM?

14. [Frank McCabe](http://www.frankmccabe.com) says:[15 January 2014 9:04 PM](#82f818dbda936ec4a3a0447a56bd9e197ce66868)

    A stack VM has a big advantage of simplicity and relative cleanliness.

    The issue with the extra operations is not 'real'; because with the register approach you have to have instructions that can read and write to different kinds of locations (arguments/locals/environment/global). This has the effect of complicating the implementation of all instructions.

    On the other hand, if you are doing assembly, then a stack push/pop instruction need not result in any actual instructions being emitted. A stack push can be handled by recording in an assembly-time table a valid source for an actual operation. Then, when actually performing the operation, you can match up the appropriate sources and destination for the emitted code.

15. nickik says:[26 January 2014 1:58 AM](#8ba774e2ba5fcfeaa24a31c85627996bcf996f60)

    Should you not compare yourself against the LuaJit Interpreter instead of the V8. It seams like Jit Vs Interpreter is not really fair.

    I am currently also think about a register maschine for a lisp like language and I always look at LuaJit for what is probebly best. It seams the LuaJit Interpreter is beating V8 a lot, so its doing something right.

    Have you looked in that direction as well? Is there anything there that would make it hard to adapt it for a lisp like language, specially functional code?

16. [wingo](http://wingolog.org/) says:[19 April 2014 12:03 PM](#22eb29cc4718732690773b434fadcdc9499c3f20)

    Microbenchmark with Lua 5.2:

    ```
    $ cat /tmp/micro.lua
    do
      local v=0
      while v < 1000000000 do
        v = v + 1
      end
    end
    $ time lua5.2 /tmp/micro.lua

    real	0m17.262s
    user	0m17.284s
    sys	0m0.008s

    ```

    With LuaJIT 2.0.3:

    ```
    $ time luajit /tmp/micro.lua

    real	0m1.198s
    user	0m1.196s
    sys	0m0.000s

    ```

    With guile 2.0:

    ```
    $ cat /tmp/micro.scm
    (let lp ((n 0))
      (when (< n #e1e9)
        (lp (1+ n))))
    $ time guile /tmp/micro.scm

    real	0m19.573s
    user	0m19.592s
    sys	0m0.004s

    ```

    Guile 2.2 (from today):

    ```
    $ time guile /tmp/micro.scm

    real	0m6.875s
    user	0m6.872s
    sys	0m0.008s

    ```

17. michael says:[23 April 2015 9:15 PM](#5e53ee84287bee72fffe014d304e75d21db13fd7)

    Are all the loads and stores in the stack example really necessary?

    If I implement your example function in forth, I get

    : squaresum ( x y -- \[x+y\]^2 )

    \+ ( x+y )

    dup ( x+y x+y )

    \\* ( \[x+y\]^2 )

    ; ( leave result on stack )

    Three instructions, not seven! (the bits in parentheses are stack effect comments). It seems like the stack code that guile 2.0 generates is suboptimal. In particular, the sequence (local-set 2) (local-ref 2) (local-ref 2) could be replaced by a single (dup), as z is already on the stack.

    Moreover, in your stack machine example, you assume that the arguments are not on the stack and need to be put there using load instructions, in your register example you assume that the arguments are already in registers ready to be used; this seems like an unfair comparison.

    I agree that register VM based interpreters tend to be faster than stack-based ones, but I think that in your example you are not really comparing like with like.

18. solrize says:[9 November 2015 8:47 AM](#54fad5ea93fdbc4f197c8795ed8281b8baeee450)

    Andy, I just tried the benchmark in your comment of 19 April 2014 under Guile 2.1.1 on a Scaleway.com ARM7 server. It used:

    real 35m19.507s

    user 42m8.180s

    sys 0m6.580s

    That's pretty pessimal! After a few minutes I thought it was looping and went and did something else while letting it run longer. The ARM cpu by all indications is about 8x slower per core than a current fast x86. It took about 7 hours to fully build 2.1.1, as you predicted. Lua 5.2 takes 2m10s and Luajit takes 2.26 seconds. A comparable gforth benchmark takes between 20s and 1m20s depending on how it's coded. So I suspect something is going wrong here.

19. [wingo](https://wingolog.org/) says:[9 November 2015 8:56 AM](#7bae6657ab44309ee3dc077751b20478cf830800)

    I was running those tests on a 64-bit machine. Guile uses 2 tag bits for fixnums, then they are signed, so your most-positive-fixnum was less than #e1e9. Looping to #e5e8 instead should do the trick.

    That said, Guile does unboxing of floats now, and could unbox integers too, and perhaps we should start doing this, as that would give us on-stack 64-bit integers for free on 32-bit systems. Something to consider!

20. solrize says:[9 November 2015 8:57 AM](#2e58e8d553c5e8f5707f1834a4f6c9ddff0bd130)

    Thanks! 5e8 iterations runs in 32.75 seconds which is about properly scaled with your x86 test.

Comments are closed.
