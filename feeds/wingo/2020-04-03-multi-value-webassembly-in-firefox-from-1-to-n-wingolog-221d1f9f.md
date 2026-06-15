---
title: 'multi-value webassembly in firefox: from 1 to n — wingolog'
url: https://wingolog.org/archives/2020/04/03/multi-value-webassembly-in-firefox-from-1-to-n
published: "2020-04-03T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2020/04/03/multi-value-webassembly-in-firefox-from-1-to-n
---

# multi-value webassembly in firefox: from 1 to n — wingolog

## [multi-value webassembly in firefox: from 1 to n](/archives/2020/04/03/multi-value-webassembly-in-firefox-from-1-to-n)

3 April 2020 10:56 AM

- [igalia](/tags/igalia)
- [compilers](/tags/compilers)
- [firefox](/tags/firefox)
- [spidermonkey](/tags/spidermonkey)
- [webassembly](/tags/webassembly)
- [bloomberg](/tags/bloomberg)

Greetings, hackers! Today I'd like to write about something I worked on recently: implementation of the [multi-value](https://github.com/WebAssembly/multi-value/blob/master/proposals/multi-value/Overview.md) future feature of WebAssembly in Firefox, as sponsored by [Bloomberg](https://techatbloomberg.com/).

In the "minimum viable product" version of WebAssembly published in 2018, there were a few artificial restrictions placed on the language. [Functions could only return a single value](https://webassembly.github.io/spec/core/valid/types.html#function-types); if a function would naturally return two values, it would have to return at least one of them by writing to memory. [Loops couldn't take parameters](https://webassembly.github.io/spec/core/valid/instructions.html#xref-syntax-instructions-syntax-instr-control-mathsf-loop-t-xref-syntax-instructions-syntax-instr-mathit-instr-ast-xref-syntax-instructions-syntax-instr-control-mathsf-end); any loop state variables had to be stored to and loaded from indexed local variables at each iteration. Similarly, any block that would naturally return more than one result would also have to do so via locals.

This restruction is lifted with the multi-value proposal. [Function types now map from *result type* to *result type*](https://webassembly.github.io/multi-value/core/syntax/types.html#function-types), where a [result type is a sequence of value types](https://webassembly.github.io/multi-value/core/syntax/types.html#result-types). That is to say, just as functions can take multiple arguments, they can return multiple results. Similarly, with the multi-value proposal, [block types are now the same as function types](https://webassembly.github.io/multi-value/core/syntax/instructions.html#syntax-blocktype): loops and blocks can take arguments and return any number of results. This change improves the expressiveness of WebAssembly as a compilation target; a C++ program compiled to multi-value WebAssembly can be encoded in fewer bytes than before. Multi-value also establishes a base for other language extensions. For example, the exception handling proposal builds on multi-value to [pass multiple values to catch blocks](https://github.com/WebAssembly/exception-handling/blob/master/proposals/Exceptions.md#exception-data-extraction).

So, that's multi-value. You would think that relaxing a restriction would be easy, but you'd be wrong! This task took me 5 months and had a number of interesting gnarly bits. This article is part one of two about interesting aspects of implementing multi-value in Firefox, specifically focussing on blocks. We'll talk about multi-value function calls next week.

**multi-value in blocks**

In the last article, I presented the [basic structure of Firefox's WebAssembly support](https://wingolog.org/archives/2020/03/25/firefoxs-low-latency-webassembly-compiler): there is a [baseline compiler optimized for low latency](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5//js/src/wasm/WasmBaselineCompile.cpp) and an [optimizing compiler optimized for throughput](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/wasm/WasmIonCompile.cpp). (There is also [Cranelift](https://github.com/bytecodealliance/wasmtime/tree/master/cranelift), a new experimental compiler that may replace the current implementation of the optimizing compiler; but that doesn't affect the basic structure.)

The optimizing compiler applies traditional compiler techniques: SSA graph construction, where values flow into and out of graphs using the usual defs-dominate-uses relationship. The only control-flow joins are loop entry and (possibly) block exit, so the addition of loop parameters means in multi-value there are some new phi variables in that case, and the expansion of block result count from \[0,1\] to \[0, *n*\] means that you may have more block exit phi variables. But these compilers are built to handle these situations; you just build the SSA and let the optimizing compiler go to town.

The problem comes in the baseline compiler.

**from 1 to *n***

Recall that the baseline compiler is optimized for compiler speed, not compiled speed. If there are only ever going to be 0 or 1 result from a block, for example, the baseline compiler's internal data structures will use something like a `Maybe` to represent that block result.

If you then need to expand this to hold a vector of values, the naïve approach of using a `Vector` would mean heap allocation and indirection, and thus would regress the baseline compiler.

In this case, and in many other similar cases, the solution is to use [value tagging](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/wasm/WasmOpIter.h#35) to represent 0 or 1 value type directly in a word, and the general case by linking out to an external vector. As block types are function types, they actually appear as function types in the WebAssembly type section, so they are already parsed; the BlockType in that case can just refer out to already-allocated memory.

In fact this value-tagging pattern applies [all](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/wasm/WasmOpIter.h#109) [over](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/wasm/WasmOpIter.h#224) [the](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/jit/LIR.h#109) [place](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/jit/BacktrackingAllocator.h#115). (The `jit/` links above are for the optimizing compiler, but they relate to function calls; will write about that next week.) I have a bit of pause about value tagging, in that it's gnarly complexity and I didn't measure the speed of alternative implementations, but it was a useful migration strategy: value tagging minimizes performance risk to existing specialized use cases while adding support for new general cases. Gnarly it is, then.

**control-flow joins**

I didn't mention it in the last article, but there are two important invariants regarding stack discipline in the baseline compiler. Recall that there's a virtual stack, and that some elements of the virtual stack might be present on the machine stack. There are four kinds of virtual stack entry: [register, constant, local, and spilled](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/wasm/WasmBaselineCompile.cpp#2102). Locals indicate local variable reads and are mostly like registers in practice; when registers spill to the stack, locals do too. (Why spill to the temporary stack instead of leaving the value in the local variable slot? Because locals are mutable. A `local.get` captures a local variable value at its point of execution. If future code changes the local variable value, you wouldn't want the captured value to change.)

Digressing, the stack invariants:

1. Spilled values precede registers and locals on the virtual stack. If *u* and *v* are virtual stack entries and *u* is older than *v*, then if *u* is in a register or is a local, then *v* is not spilled.

2. Older values precede newer values on the machine stack. Again for *u* and *v*, if they are both spilled, then *u* will be farther from the stack pointer than *v*.

There are five fundamental stack operations in the baseline compiler; let's examine them to see how the invariants are guaranteed. Recall that before multi-value, targets of non-local exits (e.g. of the [`br` instruction](https://webassembly.github.io/spec/core/syntax/instructions.html#syntax-instr-control)) could only receive 0 or 1 value; if there is a value, it's passed in a well-known register (e.g. `%rax` or `%xmm0`). (On 32-bit machines, 64-bit values use a well-known pair of registers.)

[push( *v*)](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/wasm/WasmBaselineCompile.cpp#3664-3725)Results of WebAssembly operations never push spilled values, neither onto the virtual nor the machine stack. *v* is either a register, a constant, or a reference to a local. Thus we guarantee both (1) and (2).[pop() -> *v*](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/wasm/WasmBaselineCompile.cpp#3727-4073)Doesn't affect older stack entries, so (1) is preserved. If the newest stack entry is spilled, you know that it is closest to the stack pointer, so you can pop it by first loading it to a register and then incrementing the stack pointer; this preserves (2). Therefore if it is later pushed on the stack again, it will not be as a spilled value, preserving (1).[spill()](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/wasm/WasmBaselineCompile.cpp#3456-3579)When spilling the virtual stack to the machine stack, you first traverse stack entries from new to old to see how far you need to spill. Once you get to a virtual stack entry that's already on the stack, you know that everything older has already been spilled, because of (1), so you switch to iterating back towards the new end of the stack, pushing registers and locals onto the machine stack and updating their virtual stack entries to be spilled along the way. This iteration order preserves (2). Note that because known constants never need to be on the machine stack, they can be interspersed with any other value on the virtual stack.[return( *height*, *v*)](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/wasm/WasmBaselineCompile.cpp#4084-4283)This is the stack operation corresponding to a block exit (local or nonlocal). We drop items from the virtual and machine stack until the stack height is *height*. In WebAssembly 1.0, if the target continuation takes a value, then the jump passes a value also; in that case, before popping the stack, *v* is placed in a well-known register appropriate to the value type. Note however that *v* is not pushed on the virtual stack at the return point. Popping the virtual stack preserves (1), because a stack and its prefix have the same invariants; popping the machine stack also preserves (2).[capture( *t*)](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/wasm/WasmBaselineCompile.cpp#4285-4338)Whereas return operations happen at block exits, capture operations happen at the target of block exits (the continuation). If no value is passed to the continuation, a capture is a no-op. If a value is passed, it's in a register, so we just push that register onto the virtual stack. Both invariants are obviously preserved.

Note that a value passed to a continuation via return() has a brief instant in which it has no name -- it's not on the virtual stack -- but only a location -- it's in a well-known place. capture() then gives that floating value a name.

Relatedly, there is another invariant, that the allocation of old values on block entry is the same as their allocation on block exit, so that all predecessors of the block exit flow all values via the same places. This is preserved by [spilling on block entry](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/wasm/WasmBaselineCompile.cpp#8673-8699). It's a big hammer, but effective.

So, given all this, how do we pass multiple values via return()? We don't have unlimited registers, so the `%rax` strategy isn't going to work.

The answer for the baseline compiler is informed by our *lean into the stack machine* principle. Multi-value returns are allocated in such a way that a capture() can push them onto the virtual stack. Because spilled values must precede registers, we therefore [allocate older results on the stack, and put the last result in a register (or register pair for i64 on 32-bit platforms)](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/wasm/WasmStubs.h#128-239). Note that it's possible in theory to allocate multiple results to registers; we'll touch on this next week.

Therefore the implementation of return( *height*, *v1*.. *vn*) is straightforward: we first pop register results, then spill the remaining virtual stack items, then shuffle stack results down towards *height*. This should result in a `memmove` of contiguous stack results towards the frame pointer. However because const values aren't present on the machine stack, depending on the stack height difference, it may mean a split between [moving some values toward the frame pointer and some towards the stack pointer, then filling in by spilling constants](https://searchfox.org/mozilla-central/rev/4ccefc3181f9d237ef4ca8bd17b4e7c101ddf7b5/js/src/wasm/WasmBaselineCompile.cpp#4156-4253). It's gnarly, but it is what it is. Note that the links to the return and capture implementations above are to the post-multi-value world, so you can see all the details there.

**that's it!**

In summary, the hard part of multi-value blocks was reworking internal compiler data structures to be able to represent multi-value block types, and then figuring out the low-level stack manipulations in the baseline compiler. The optimizing compiler on the other hand was pretty easy.

When it comes to calls though, that's another story. We'll get to that one next week. Thanks again to [Bloomberg](https://techatbloomberg.com/) for supporting this work; I'm really delighted that [Igalia](https://igalia.com) and Bloomberg have been working together for a long time (coming on 10 years now!) to push the web platform forward. A special thanks also to Mozilla's Lars Hansen for his patience reviewing these patches. Until next week, then, stay at home & happy hacking!

## related articles

- [understanding webassembly code generation throughput](/archives/2020/04/14/understanding-webassembly-code-generation-throughput)
- [multi-value webassembly in firefox: a binary interface](/archives/2020/04/08/multi-value-webassembly-in-firefox-a-binary-interface)
- [firefox's low-latency webassembly compiler](/archives/2020/03/25/firefoxs-low-latency-webassembly-compiler)
- [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)
- [es6 generator and array comprehensions in spidermonkey](/archives/2014/03/07/es6-generator-and-array-comprehensions-in-spidermonkey)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)

Comments are closed.
