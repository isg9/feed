---
title: inside javascriptcore's low-level interpreter — wingolog
url: https://wingolog.org/archives/2012/06/27/inside-javascriptcores-low-level-interpreter
published: "2012-06-27T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2012/06/27/inside-javascriptcores-low-level-interpreter
---

# inside javascriptcore's low-level interpreter — wingolog

## [inside javascriptcore's low-level interpreter](/archives/2012/06/27/inside-javascriptcores-low-level-interpreter)

27 June 2012 11:25 AM

- [javascript](/tags/javascript)
- [javascriptcore](/tags/javascriptcore)
- [webkit](/tags/webkit)
- [igalia](/tags/igalia)
- [jsc](/tags/jsc)
- [compilers](/tags/compilers)
- [v8](/tags/v8)

Good day, hackers! And hello to the rest of you, too, though I fear that this article isn't for you. In the vertical inches that follow, we're going to nerd out with JavaScriptCore's new low-level interpreter. So for those of you that are still with me, refill that coffee cup, and get ready for a look into a lovely hack!

**hot corn, cold corn**

Earlier this year, JavaScriptCore got a new interpreter, the LLInt. ("Low-level interpreter", you see.) As you probably know, JavaScriptCore is the [JavaScript implementation of the WebKit project](//wingolog.org/archives/2011/10/28/javascriptcore-the-webkit-js-implementation). It's used in web browsers like [Safari](http://www.apple.com/safari/) and [Epiphany](http://projects.gnome.org/epiphany/), and also offers a stable API for embedding in non-web applications.

In this article, we'll walk through some pieces of the LLInt. But first, we need to describe the problem that the LLInt solves.

So, what is the fundamental problem of implementing JavaScript? Of course there are lots of things you could say here, but I'm going to claim that in the context of web browsers, it's the tension between the need to optimize small amounts of hot code, while minimizing overhead on large amounts of cold code.

Web browsers are like little operating systems unto themselves, into which you as a user install and uninstall hundreds of programs a day. The vast majority of code that a web browser sees is only executed once or twice, so for cold code, the name of the game is to do as little work as possible.

For cold code, a good bytecode interpreter can beat even a very fast native compiler. Bytecode can be more compact than executable code, and quicker to produce.

All of this is a long introduction into what the LLInt actually is: it's a better bytecode interpreter for JSC.

**context**

Before the introduction of the LLInt, the so-called "classic interpreter" was just a C++ function with a bunch of labels. Every kind of bytecode instruction had a corresponding label in the function.

As a hack, in the classic interpreter, bytecode instructions are actually wide enough to hold a pointer. The opcode itself takes up an entire word, and the arguments also take up entire words. It is strange to have such bloated bytecode, but there is one advantage, in that the opcode word can actually store the address of the label that implements the opcode. This is called direct threading, and presumably its effects on the branch prediction table are sufficiently good so as to be a win.

Anyway, it means that Interpreter.cpp has a method that looks like this:

```
// vPC is pronounced "virtual program counter".
JSValue Interpreter::execute(void **vPC)
{
  goto *vPC;
//...
  // add dst op1 op2: Add two values together.
op_add:
  {
    int dst = (int)vPC[1];
    int op1 = (int)vPC[2];
    int op2 = (int)vPC[3];

    fp[dst] = add(fp[op1], fp[op2]);

    vPC += 4;
    goto *vPC;
  }

  // jmp offset: Unconditional branch.
op_jmp:
  {
    ptrdiff_t offset = (ptrdiff_t)vPC[1];
    vPC += offset;
    goto *vPC;
  }
//...
}

```

It's a little different [in practice](http://trac.webkit.org/browser/trunk/Source/JavaScriptCore/interpreter/Interpreter.cpp#L1966), but the essence is there.

OK, so what's the problem? Well, readers who have been with me for a while might recall my article on [static single assignment form](//wingolog.org/archives/2011/07/12/static-single-assignment-for-functional-programmers/), used as an internal representation by compilers like GCC and LLVM. One conclusion that I had was that SSA is a great first-order IR, but that its utility for optimizing higher-order languages is less clear. However, this computed-goto thing that I showed above ( `goto *vPC`) is a form of higher-order programming!

Indeed, as the [GCC internals manual](http://gcc.gnu.org/onlinedocs/gccint/Edges.html#Edges) notes:

> Computed jumps contain edges to all labels in the function referenced from the code. All those edges have EDGE\_ABNORMAL flag set. The edges used to represent computed jumps often cause compile time performance problems, since functions consisting of many taken labels and many computed jumps may have very dense flow graphs, so these edges need to be handled with special care.

Basically, GCC doesn't do very well at compiling interpreter loops. At -O3, it does get everything into registers on my machine, but it residualizes 54 KB of assembly, whereas I only have 64 KB of L1 instruction cache. Many other machines just have 32 KB of L1 icache, so for those machines, this interpreter is a lose. If I compile with -Os, I get it down to 32 KB, but that's still a tight fit.

Besides that, implementing an interpreter from C++ means that you can't use the native stack frame to track the state of a computation. Instead, you have two stacks: one for the interpreter, and one for the program being interpreted. Keeping them in sync has an overhead. Consider what GCC does for `op_jmp` at -O3:

```
op_jmp:
    mov    %rbp,%rax
    ; int currentOffset = vPC - bytecode + 1
    sub    0x58(%rbx),%rax
    sar    $0x3,%rax
    add    $0x1,%eax
    ; callFrame[CurrentOffset] = currentOffset
    mov    %eax,-0x2c(%r11)

    ; ptrdiff_t offset = (ptrdiff_t)vPC[1]
    movslq 0x8(%rbp),%rax

    ; vPC += offset
    lea    0x0(%rbp,%rax,8),%rbp

    ; goto *vPC
    mov    0x0(%rbp),%rcx
    jmpq   *%rcx

```

First there is this strange prelude, which effectively stores the current vPC into an address on the stack. To be fair to GCC, this prelude is part of the [DEFINE\_OPCODE](http://trac.webkit.org/browser/trunk/Source/JavaScriptCore/interpreter/Interpreter.cpp#L1961) macro, and is not an artifact of compilation. Its purpose is to let other parts of the runtime see where a computation is. I tried, but I can't convince myself that it is necessary to do this at the beginning of every instruction, so maybe this is just some badness that should be fixed, if the classic interpreter is worth keeping.

The rest of the opcode is similar to the version that the LLInt produces, as we will see, but it is less compact.

**the compiler to make the code you want**

The goal of compilation is to produce good code. It can often be a good technique to start from the code you would like to have, and then go backwards and see how to produce it. It's also a useful explanatory technique: we can take a look at the machine code of the LLInt, and use it to put the LLInt's source in context.

In that light, here is the code corresponding to `op_jmp`, as part of the LLInt:

```
llint_op_jmp:
    ; offset += bytecode[offset + 1]
    add    0x8(%r10,%rsi,8),%esi
    ; jmp *bytecode[offset]
    jmpq   *(%r10,%rsi,8)

```

That's it! Just 9 bytes. There is the slight difference that the LLInt moves around an offset instead of a pointer into the bytecode. The whole LLInt is some 14 KB, which fits quite confortably into icache, even on mobile devices.

This assembly code is part of the LLInt as compiled for my platform, x86-64. The source of the LLInt is written in a custom low-level assembly language. Here is the source for the `jmp` opcode:

```
_llint_op_jmp:
    traceExecution()
    dispatchInt(8[PB, PC, 8])

```

You can define macros in this domain-specific language. Here's the definition of `dispatchInt`:

```
macro dispatchInt(advance)
    addi advance, PC
    jmp [PB, PC, 8]
end

```

At this point it's pretty clear how the source corresponds to the assembly. The WebKit build system will produce a version of the LLInt customized for the target platform, and link that interpreter into the JavaScriptCore library. There are backend code generators for ARMv7, X86-32, and X86-64. These interpreters mostly share the same source, modulo the [representation differences for JSValue on 32-bit and 64-bit systems](//wingolog.org/archives/2011/05/18/value-representation-in-javascript-implementations).

Incidentally, the ["offlineasm" compiler](http://trac.webkit.org/browser/trunk/Source/JavaScriptCore/offlineasm) that actually produces the native code is written in Ruby. It's clear proof that Ruby can be fast, as long as you don't load it at runtime ;-)

**but wait, there's more**

We take for granted that low-level programs should be able to determine how their data is represented. C and C++ are low-level languages, to the extent that they offer this kind of control. But what is not often remarked is that for a virtual machine, its code is also data that needs to be examined at runtime.

I have written before about JSC's [optimizing DFG compiler](//wingolog.org/archives/2011/10/28/javascriptcore-the-webkit-js-implementation). In order to be able to optimize a computation that is in progress, or to abort an optimization because a speculation failed -- in short, to be able to use [OSR](//wingolog.org/archives/2011/06/20/on-stack-replacement-in-v8) to tier up or down -- you need to be able to replace the current stack frame. Well, with the classic interpreter written in C++, you can't do that, because you have no idea what's in your stack frame! In contrast, with the LLInt, JSC has precise control over the stack layout, allowing it to tier up and down directly from the interpreter.

This can obviate the need for the baseline JIT. Currently, WebKitGTK+ and Apple builds still include the baseline, quick-and-dirty JIT, as an intermediate tier between the LLInt and the DFG JIT. I don't have sure details about about long-term plans, but I would speculate that recent work on the DFG JIT by Filip Pizlo and others has had the goal of obsoleting the baseline JIT.

Note that in order to tier directly to the optimizing compiler, you need type information. Building the LLInt with the DFG optimizer enabled causes the interpreter to be instrumented to record value profiles. These profiles record the types of values seen by instructions that load and store values from memory. Unlike V8, which stores this information in executable code as part of the inline caches, in the LLInt these value profiles are in non-executable memory.

Spiritually speaking, I am [all for run-time code generation](//wingolog.org/archives/2011/06/21/security-implications-of-jit-compilation). But it must be admitted that web browsers are a juicy target, and while it is unfair to make safe languages pay for bugs in C++ programs, writable text is an attack surface. Indeed in some environments, JIT compilation is prohibited by the operating system -- and in that sort of environment, a fast interpreter like the LLInt is a big benefit.

There are a few more LLInt advantages, like being able to use the CPU's overflow flags, doing better register allocation, and retaining less garbage (JSC scans the stack conservatively). But the last point I want to mention is memory: an interpreter doesn't have to generate executable code just to display a web page. Obviously, you need to complement the LLInt with an optimizing compiler for hot code, but laziness up front can improve real-world page load times.

**in your webs, rendering your kittens**

So that's the LLInt: a faster bytecode interpreter for JSC.

I wrote this post because I finally got around to turning on the LLInt in the [WebKitGTK+](http://webkitgtk.org/) port a couple weeks ago. That means that in September, when GNOME 3.6 comes out, not only will you get an [Epiphany](http://projects.gnome.org/epiphany/) with the one-process-per-tab WebKit2 backend, you also get cheaper and faster JavaScript with the LLInt. Those of you that want the advance experience can try it out with last Monday's [WebKitGTK+ 1.9.4 release](http://article.gmane.org/gmane.os.opendarwin.webkit.gtk/1164).

## related articles

- [stranger in these parts](/archives/2012/05/16/stranger-in-these-parts)
- [heap object representation in spidermonkey](/archives/2018/10/11/heap-object-representation-in-spidermonkey)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [JavaScriptCore, the WebKit JS implementation](/archives/2011/10/28/javascriptcore-the-webkit-js-implementation)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)
- [understanding webassembly code generation throughput](/archives/2020/04/14/understanding-webassembly-code-generation-throughput)

### 10 responses

1. Julian Andres Klode says:[27 June 2012 12:15 PM](#9ddda2caeca9d6c7a7c4c39f92786a62442b8670)

   In my stupid byte code interpreters, I noticed that creating a function for each op-code and calling via a static vtable from the interpreter loop performed much better than using labels, and stuff (it also sometimes considerably changed performance when removing redundant checks otherwise, getting much slower, probably due to even more inlining).

   This might be caused by the loop requiring far less code, and thus be friendlier to CPU caches as well, so the overhead of the indirect call was more than compensated by the instruction cache friendlyness.

2. Rene Dudfield says:[27 June 2012 2:10 PM](#3a71624130c9b005a10e3a1292a3b053e3533528)

   @Julian : do you have any code to link to, or a minimal example? I'm having trouble figuring out what you're doing exactly... sounds good. Storing function pointers to each opcode?

3. John Cowan says:[27 June 2012 2:33 PM](#874825e9d43725a825ec81ebeab6803d0651e998)

   Wow, that's quite a result. Those big case-statement interpreters were designed because procedure calling was so slow you couldn't afford to do it on each bytecode. Cobol programmers could have a whole career from entry-level coder to retirement without ever once writing a subroutine with separate lexical scope (as opposed to Cobol PERFORM A THROUGH B, which is more like Basic's GOSUB), because call performance was so intolerably bad on big iron that the only calls they wrote were to assembly-language routines that invoked the database. How things have changed!

4. [wingo](http://wingolog.org/) says:[27 June 2012 4:01 PM](#03d95a0cca98b912e3336681e198675942bf976f)

   I thought it was clear, but there was some confusion out there, so I should probably state explicitly that this was not my work at all. The LLInt was mostly written by Apple hacker Filip Pizlo.

   Carry on, intertubes, carry on!

5. Julian Andres Klode says:[27 June 2012 6:00 PM](#5d9ca6cd3687f2c57392f54a9f3785bbd9159cfb)

   @Rene: Nope, no code. This was mostly just some personal experiments. Basically, assume you have a context of some sort, and various functions like:

   op\_add(context \*, instruction \*);

   Then you just create a table like

   table = { \[OP\_ADD\] = op\_add, \[OP\_SUB\] = op\_sub, ... };

   and call the function via pointer in the loop

   while (TRUE)

   {

   decode\_instruction();

   table\[op-code\](context, instruction);

   }

   instead of embedding all those op-code specific stuff into the main loop. Thus my interpreter loop is very small.

6. Peter Ilberg says:[28 June 2012 8:02 AM](#4f1c57ec00b066005f7a1527e8ea0e9e9b6db3b4)

   Here's a nice discussion of how to write fast interpreter loops:

   http://www.emulators.com/docs/nx25\_nostradamus.htm

7. Rene Dudfield says:[28 June 2012 9:49 AM](#ac7f4ab22df2feefeb05e24adc15f56ddf2fa6cd)

   Thanks for the example Julian. As you say, I imagine the pointers could all be cached in a single cache line, and so could the look up for the pointer. Perhaps it is using the data cache as well as the code cache, and probably less branch predictions.

   The LLInt approach of writing the interpreter in a portable assembler (like luajit2 does) does seem to result in a very fast result.

   With LLInt, it should also be possible to output C code like Orc (see Oil Runtime Compiler, http://code.entropywave.com/orc/) does. This would provide better portability, and also a way to check the assembler version is correct by comparing results to the C version.

   @Peter: thanks for the link... the "The Nostradamus Distributor" is truly insane! I wonder how that compares with the newer approaches taken by LLInt/luajit2.

8. [Peter Goodman](http://www.ioreader.com) says:[29 June 2012 7:00 AM](#0af41c4808b4558372bdc1a6597516bb78979676)

   This might be of interest to you: http://www.cs.toronto.edu/~matz/dissertation/matzDissertation-latex2html/matzDissertation-latex2html.html

   The key takeaway being context threading, as opposed to direct threading.

9. John Yates says:[7 July 2012 0:21 AM](#bf8e6cfb6c54396f66d75c475676b94b3b65cea7)

   Having programmed for 40+ years I have written a number of interpreters...

   How big did you say that L1 I-cache was? 64KB? 32KB? One time I had to fit the heart of a high performance x86 interpreter into 8KB :-) A total of 2K 4-byte RISC instructions. It was programmed entirely in hand scheduled Alpha assembly language. Due to a quirk of corporate history and vocabulary we called it an emulator:

   http://www.hpl.hp.com/hpjournal/dtj/vol9num1/vol9num1art1.pdf

   Interpreting the variable length x86 instruction set presents some interesting challenges. My main loop wrapped a dispatch to 8 byte aligned "case labels". The Alpha had enough integer registers to allow me to map the eight x86 integer registers, the PC, an 8 byte sliding instruction window, and a bunch of interpreter state permanently into registers.

   I maintained the 8 bytes between PC and PC+7 in an Alpha register. The leading 2 bytes were used to fetch a 4 byte dispatch address from a table of 64K entries. Because my dispatch targets were 8 byte aligned (better use of Alpha dual instruction issue) I used the low order 3 bits to encode the instruction length. (Although x86 instructions can be up to 15 bytes the overwhelming number of executed instructions are 1 to 8 bytes long.)

   I prefetched the instruction window refill data and the dispatch pointer for the next sequential instruction as part of the main dispatch. This approach overlapped those two memory accesses with the interpreter's computed transfer of control and instruction execution. Unless the current instruction altered sequential flow by the time I got back to the top of the loop I already had the new dispatch address, the length of the new instruction and data to replenish the instruction window. (I was actually part of the Alpha chip team at the time so it should come as no surprise that this pipelined approach to instruction execution is exactly what hardware CPUs do.)

   The other crucial technique was "tail merging" jumping to common continuations. I often describe a interpreter as a big river with many, many tributaries. The dispatch take you to the head of some tributary that then merges with some other tributaries which in turn merge with some others... Ultimately one reach the main river for final bookkeeping and a new dispatch.

10. Mathew Zaleski says:[2 April 2013 2:27 PM](#5089ec4d79e6b0e71f6d37d2bfcb41904e5085cb)

    This is very interesting. In the CTI work, we concentrated on branch mispredictions caused by the indirect branches that end each virtual body. We were thinking in terms of smallish virtual instructions -- like those of Java or Ocaml -- which are only a few cycles long. For these, misprediction delays are a killer.

    Well, JavaScript has big, fat, polymorphic instructions, so evidently the mispredictions aren't the holdup -- it's the icache misses.

Comments are closed.
