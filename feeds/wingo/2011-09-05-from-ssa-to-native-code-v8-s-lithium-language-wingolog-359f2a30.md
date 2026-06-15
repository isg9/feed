---
title: 'from ssa to native code: v8''s lithium language — wingolog'
url: https://wingolog.org/archives/2011/09/05/from-ssa-to-native-code-v8s-lithium-language
published: "2011-09-05T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/09/05/from-ssa-to-native-code-v8s-lithium-language
---

# from ssa to native code: v8's lithium language — wingolog

## [from ssa to native code: v8's lithium language](/archives/2011/09/05/from-ssa-to-native-code-v8s-lithium-language)

5 September 2011 11:25 AM

- [igalia](/tags/igalia)
- [v8](/tags/v8)
- [javascript](/tags/javascript)
- [compilers](/tags/compilers)
- [ssa](/tags/ssa)

So, friends, about a month ago I posted [a look at the Hydrogen intermediate language](//wingolog.org/archives/2011/08/02/a-closer-look-at-crankshaft-v8s-optimizing-compiler), as part of an [ongoing series](//wingolog.org/tags/v8) on the internals of the [V8 JavaScript implementation](http://code.google.com/p/v8/). Hydrogen is the high-level part of V8's Crankshaft optimizing compiler, where the optimizations happen. This article tells the story of Lithium, Crankshaft's low-level, machine-dependent intermediate language.

**totally metal**

So here's the deal: you parsed your JavaScript functions to AST objects ( [parse all the functions!](http://hyperboleandahalf.blogspot.com/2010/06/this-is-why-ill-never-be-adult.html)). You emitted some simple native code, quickly but not optimized, and you sat back and waited. After a little while, your profiling told you that some function was hot, so you re-parsed it, generated a graph of Hydrogen instructions and optimized that graph. Now you need to generate code. And that's where we are! Generate all the code!

But that's not what V8 does here. Instead of going directly to native code, it translates the HGraph to a chunk of Lithium code, then emits machine code from those Lithium instructions. Unlike Hydrogen, which is an [SSA form](//wingolog.org/archives/2011/07/12/static-single-assignment-for-functional-programmers) in which instructions implicitly define values, the Lithium form is closer to [three-address code](http://en.wikipedia.org/wiki/Three_address_code), with labels and gotos, and explicitly names its operands (if any) and results (if any).

These temporary values are initially declared with a number of constraints, such as "must be in a double-precision register". Later, a register allocator considers the constraints and liveness ranges, allocating each value to a specific register. Code generation happens after register allocation.

**an example**

Let us take a look at an example to see how these things interact. Given that Lithium is target-specific, we'll just choose one of the targets; ia32 and x64 are the most advanced, so we'll just take x64. (That's what they call it.)

```
LInstruction* LChunkBuilder::DoAdd(HAdd* instr) {
  if (instr->representation().IsInteger32()) {
    ASSERT(instr->left()->representation().IsInteger32());
    ASSERT(instr->right()->representation().IsInteger32());
    LOperand* left =
        UseRegisterAtStart(instr->LeastConstantOperand());
    LOperand* right =
        UseOrConstantAtStart(instr->MostConstantOperand());
    LAddI* add = new LAddI(left, right);
    LInstruction* result = DefineSameAsFirst(add);
    if (instr->CheckFlag(HValue::kCanOverflow)) {
      result = AssignEnvironment(result);
    }
    return result;
  } else if (instr->representation().IsDouble()) {
    return DoArithmeticD(Token::ADD, instr);
  } else {
    ASSERT(instr->representation().IsTagged());
    return DoArithmeticT(Token::ADD, instr);
  }
  return NULL;
}

```

As an aside, this is fairly typical for Crankshaft code: tight, lots of asserts, custom type predicates instead of RTTI, self-explanatory enumerated values, and much more oriented towards function calls than in-place mutations. The new objects are created in a "zone", under the covers, so that once the compiler is done, it can free the whole zone at once. I reflowed the `left` and `right` lines; otherwise it really is a paragraph of text.

Unfortunately there are not a lot of big-picture comments, hence this blog series.

Anyway, here we have three cases. The first two are for when the HValue has been has been given an unboxed representation, as an int32 or as a double. The last is for tagged values, which can be small integers ( `Smi` values) or heap-allocated doubles. In these latter cases, we dispatch to the `DoArithmeticD` and `DoArithmeticT` helpers, which create generic `LArithmeticD` and `LArithmeticT` instructions. But we see that in the first case, there is a special `LAddI` instruction.

When there is a special case in Crankshaft, it either means that we are working around some strange corner of JavaScript, or that something is being optimized. In this case, it is an optimization, reflecting the importance of integer addition. The concrete optimizations are:

1. Immediate operands. If one of the addends is a constant at compile-time, it can be encoded into the instruction stream directly.

2. Overflow check removal. If the range analysis (see my [previous article](//wingolog.org/archives/2011/08/02/a-closer-look-at-crankshaft-v8s-optimizing-compiler)) showed that the value would not overflow, then no overflow check is emitted. (Otherwise, an overflow check will be inserted, branching to a deoptimization bailout. The bailout will need to know what variables are live, in that case, hence the `AssignEnvironment` call; more on that later.)

The `UseRegisterAtStart` helper call indicates that the operand should be in a register, and that it will be live at the start of the instruction but not the end. This latter constraint is needed because x86 instructions typically clobber the first operand. `UseRegisterOrConstant` only allocates a register if the operand is not constant and so cannot be immediate. Finally `DefineSameAsFirst` constrains the output register to be the same as the first operand. All of these calls additionally inform the register allocator of the uses and definitions, in SSA parlance, so that the allocator can build accurate liveness ranges.

Just to be complete, the double arithmetic case (not shown) is bit more straightforward, in that there are no overflow checks or immediates. Tagged arithmetic, on the other hand, always dispatches to an inline cache. (I find this to be fascinating. Typically language implementations inline fixnum operations, but V8 does not because their inline caches also perform type feedback, allowing unboxing optimizations where possible. In V8, tagged integer arithmetic is never inlined, by default.)

**allocate all the registers?**

Why bother with this Lithium thing? Why not just generate code directly? The answer is that having a target-specific Lithium language allows target-specific code generation and temporary allocation, while permitting the register allocator itself to be generic. As an example, see the [ARM DoAdd implementation](http://code.google.com/p/v8/source/browse/branches/bleeding_edge/src/arm/lithium-arm.cc#1362).

Most everyone knows, but the idea of register allocation is to store your temporaries in machine registers instead of in memory, so that things run fast. It's tricky though, because nested procedure calls can trash your registers, so you need to "spill" them to memory in certain cases. You also need to avoid thrashing values between registers and memory. V8 also tries to use registers when calling internal runtime helpers, which can result in a fair amount of register shuffling.

That's all traditional compiler stuff. But in a garbage-collected language, we also have the complication of knowing which registers hold garbage-collected values and which don't. For every point in the code in which GC could run, the collector will need to mark those pointers, and possibly relocate them. This complicates the allocator.

Another complication is that besides the normal entry point, optimized functions often have an additional entry point, due to [on-stack replacement](//wingolog.org/archives/2011/06/20/on-stack-replacement-in-v8). That entry point has to take values from memory and pack them into registers in a way that lets the computation proceed.

Finally, the allocator plays a role in permitting so-called "lazy deoptimization". V8 hacker Søren Gjesse writes, in a rare but pleasantly exegetic [message](http://groups.google.com/group/v8-dev/msg/7bef791ea0e27f31) to the list:

> Lazy deoptimization is what happens when all code in a context is forced to be deoptimized either due to starting debugging or due to a global assumption on the optimized code that no longer holds. The reason for the term "lazy" is that actual deoptimization will not happen until control returns to the optimized function. The way we do this lazy deoptimization is somewhat fragile and we have had quite a few bugs in that code. It is based on destructively patching the optimized code placing a call just after each call forcing deoptimization. And actually the patching is not right after the return point but after the gap code inserted at the return point. For the deferred stack check the code up to restoring the registers (popad in IA-32) is considered the gap code.

**gaps?**

The gap code that Søren referred to is the set of moves that shuffles registers back into place after a call. Translation of Hydrogen to Lithium inserts LGap instructions between each LInstruction. When the allocator decides that it needs to spill, restore, or shuffle some registers, it records these moves on an LGap.

[![](//wingolog.org/pub/windmills.jpg)](http://gallium.inria.fr/~xleroy/publi/parallel-move.pdf)

A later pass serializes these [parallel moves](http://gallium.inria.fr/~xleroy/publi/parallel-move.pdf) into a specific order, trying to use the minimum number of temporaries. I link to Xavier Leroy's paper above, not because V8 uses Coq, but because of the nice introduction to the problem.

Anyway, the actual algorithm used by the Lithium register allocator appears to be a version of [linear-scan](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.27.2462&rep=rep1&type=pdf), with live range splitting. This part of V8 actually is well-commented, so if you can understand the Sarkar paper, then head on over to [ye olde google code](http://code.google.com/p/v8/source/browse/branches/bleeding_edge/src/lithium-allocator.cc) and check it out.

**generate all the code!**

The result of register allocation is that all LOperands have been assigned specific machine registers, and some moves have been recorded in the LGap instructions. Now, finally, we generate code: each Lithium instruction is traversed in order, and they all write assembly into a buffer. See [the LAddI code generator](http://code.google.com/p/v8/source/browse/branches/bleeding_edge/src/x64/lithium-codegen-x64.cc#1275), as an example.

All of the JavaScript engines generate code using assemblers written in C++. Some people still assume that compilers must generate assembly, in text, but it's much easier and faster just to call methods on an object that writes bytes into a buffer. This approach also makes it easy to add [macro instructions](http://en.wikipedia.org/wiki/Assembly_language#Macros), just by subclassing the assembler. Finally, since modern JS implementations have to muck around with the bits all the time (disassembly, source debugging, backtraces, garbage collection (tracing and moving), deoptimization, OSR, inline caches, type feedback), it's easiest to maintain your invariants if you control the bit generation.

V8's assembler originally came from the [Strongtalk](http://strongtalk.org/) project, where some of the V8 hackers came from, long ago. This assembler writes code into memory, going forward, and then when it's finished, it appends "relocation information" to the code, written backwards. Relocation information lets V8 mark interesting places in the generated code: pointers that might need to be relocated (after garbage collection, for example), correspondences between the machine program counter and source locations, etc.

As anyone who's hacked on debuggers knows, the problem with debug information is that it is *large*. Without clever encodings, the amount of debugging data generated by a compiler can kill runtime performance, simply because of the amount of data that needs to be paged into memory. Of course this concern also applies at compile-time, and especially so for JIT compilers. The Strongtalk assembler's main contribution to V8 (I think) is a [complicated bit-encoding](http://code.google.com/p/v8/source/browse/branches/bleeding_edge/src/assembler.cc#102) that tries to compress this data as far as possible. The assembler also provides iterators for traversing this data at runtime.

**finish line**

And now, dear readers, I believe that we have come full circle on this look at V8. We started low, looking at the [assembly generated by V8 for a simple loop](//wingolog.org/archives/2011/06/08/what-does-v8-do-with-that-loop). We elaborated this low-level view with a look at [on-stack replacement](//wingolog.org/archives/2011/06/20/on-stack-replacement-in-v8), though without understanding how such information was available. So we jumped back to the top, taking a [high-level look at V8's compilers](//wingolog.org/archives/2011/07/05/v8-a-tale-of-two-compilers). We drilled down to [Hydrogen](//wingolog.org/archives/2011/08/02/a-closer-look-at-crankshaft-v8s-optimizing-compiler), and here we covered Lithium, register allocation, and code generation.

I hope it's been an entertaining series to read; it has certainly been enlightening to write. Thanks very much to my employer [Igalia](http://igalia.com/) for supporting my work on this. Comments and corrections are welcome, as always, and happy hacking.

## related articles

- [a closer look at crankshaft, v8's optimizing compiler](/archives/2011/08/02/a-closer-look-at-crankshaft-v8s-optimizing-compiler)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)
- [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)
- [heap object representation in spidermonkey](/archives/2018/10/11/heap-object-representation-in-spidermonkey)
- [effects analysis in guile](/archives/2014/05/18/effects-analysis-in-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)

### 5 responses

1. Ben Gildenstein says:[5 September 2011 1:45 PM](#9ba752933b18634a3ceec21a38643904177e9570)

   Thanks for the very interesting read! It would be nice to allocate a paragraph to talk about the performance benefits of this optimization step, even if only your best guess.

   Thanks again!

2. [Vyacheslav Egorov](http://mrale.ph) says:[5 September 2011 4:30 PM](#43045dee86f6897e1d8fced9fcabd0cbbdac88ec)

   Minor correction:

   \> This assembler writes code into memory, going forward, and then when it's finished, it appends "relocation information" to the code, written backwards.

   During generation code and reloc info are generated simultaneously (code grows forward, reloc info backward as you have noticed). When code generation is finished they are separated and materialized as different heap objects: code as Code object and reloc info as a ByteArray.

3. [Erik Corry](http://code.google.com/p/v8) says:[5 September 2011 7:16 PM](#c6d61590abcebe5c7abe25290248f509eecbfcca)

   Nice writeup as usual. I don't think Søren has ever been accused of exegesis before. Small correction:

   Typically language implementations inline fixnum operations, but V8 does not because their inline caches also perform type feedback, allowing unboxing optimizations where possible. In V8, tagged integer arithmetic is never inlined, by default.)

   This is true, but the context is generation of optimized code. The ICs in optimized code are AFAIK never mined for their type information (perhaps they should be). It's only the IC call sites in the unoptimized code that are used in this way. In the optimized code I think we just call the IC because we don't know what to optimize for. If we knew what the inputs were going to be we would generate untagged inlined code, but since we know nothing, we call the IC and the IC call site patching will select the most appropriate code if and when that path in the function is reached.

4. [Kim Schulz](http://schulz.dk) says:[6 September 2011 9:11 AM](#abd84be878202f14f0c864c2e4018b381d807b75)

   Thanks for a very informative series of articles on this. It is a joy to read in-depth blog articles like this - especially when it is about something as brilliant as V8. Keep up the good work.

   Cheers,

   Kim Schulz, Denmark

5. Bo Wu says:[30 December 2011 10:02 PM](#e7535b5bfc27bb1c1e6d9abc783d688c84522eba)

   Truly interesting posts on V8. I enjoyed reading them!

Comments are closed.
