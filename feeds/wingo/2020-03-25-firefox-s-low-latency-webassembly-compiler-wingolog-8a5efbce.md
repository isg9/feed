---
title: firefox's low-latency webassembly compiler — wingolog
url: https://wingolog.org/archives/2020/03/25/firefoxs-low-latency-webassembly-compiler
published: "2020-03-25T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2020/03/25/firefoxs-low-latency-webassembly-compiler
---

# firefox's low-latency webassembly compiler — wingolog

## [firefox's low-latency webassembly compiler](/archives/2020/03/25/firefoxs-low-latency-webassembly-compiler)

25 March 2020 4:29 PM

- [igalia](/tags/igalia)
- [compilers](/tags/compilers)
- [firefox](/tags/firefox)
- [spidermonkey](/tags/spidermonkey)
- [webassembly](/tags/webassembly)
- [bloomberg](/tags/bloomberg)

Good day!

Today I'd like to write a bit about the WebAssembly baseline compiler in Firefox.

**background: throughput and latency**

WebAssembly, as you know, is a virtual machine that is present in web browsers like Firefox. An important initial goal for WebAssembly was to be a good target for compiling programs written in C or C++. You can visit a web page that includes a program written in C++ and compiled to WebAssembly, and that WebAssembly module will be downloaded onto your computer and run by the web browser.

A good virtual machine for C and C++ has to be fast. The *throughput* of a program compiled to WebAssembly (the amount of work it can get done per unit time) should be approximately the same as its throughput when compiled to "native" code (x86-64, ARMv7, etc.). WebAssembly meets this goal by defining an instruction set that consists of similar operations to those directly supported by CPUs; WebAssembly implementations use optimizing compilers to translate this portable instruction set into native code.

There is another dimension of fast, though: not just work per unit time, but also time until first work is produced. If you want to go play [Doom 3 on the web](http://wasm.continuation-labs.com/d3demo/), you care about frames per second but also time to first frame. Therefore, WebAssembly was designed not just for high throughput but also for low latency. This focus on low-latency compilation expresses itself in two ways: binary size and binary layout.

On the size front, WebAssembly is optimized to encode small files, reducing download time. One way in which this happens is to use a [variable-length encoding](https://webassembly.github.io/spec/core/binary/values.html#integers) anywhere an instruction needs to specify an integer. In the usual case where, for example, there are fewer than 128 local variables, this means that a [`local.get` instruction](https://webassembly.github.io/spec/core/syntax/instructions.html#syntax-instr-variable) can refer to a local variable using just one byte. Another strategy is that WebAssembly programs target a stack machine, reducing the need for the instruction stream to explicitly load operands or store results. Note that size optimization only goes so far: it's assumed that the bytes of the encoded module will be compressed by gzip or some other algorithm, so sub-byte entropy coding is out of scope.

On the layout side, the WebAssembly binary encoding is sorted by design: definitions come before uses. For example, there is a [section of type definitions](https://webassembly.github.io/spec/core/binary/modules.html#binary-typesec) that occurs early in a WebAssembly module. Any use of a declared type can only come after the definition. In the case of functions which are of course mutually recursive, function type declarations come before the actual definitions. In theory this allows web browsers to take a one-pass, streaming approach to compilation, starting to compile as functions arrive and before download is complete.

**implementation strategies**

The goals of high throughput and low latency conflict with each other. To get best throughput, a compiler needs to spend time on code motion, register allocation, and instruction selection; to get low latency, that's exactly what a compiler should not do. Web browsers therefore take a two-pronged approach: they have a compiler optimized for throughput, and a compiler optimized for latency. As a WebAssembly file is being downloaded, it is first compiled by the quick-and-dirty low-latency compiler, with the goal of producing machine code as soon as possible. After that "baseline" compiler has run, the "optimizing" compiler works in the background to produce high-throughput code. The optimizing compiler can take more time because it runs on a separate thread. When the optimizing compiler is done, it replaces the baseline code. ( [The actual heuristics about whether to do baseline + optimizing ("tiering") or just to go straight to the optimizing compiler are a bit hairy](https://searchfox.org/mozilla-central/source/js/src/wasm/WasmCompile.cpp#270-418), but this is a summary.)

This article is about the WebAssembly baseline compiler in Firefox. It's a surprising bit of code and I learned a few things from it.

**design questions**

Knowing what you know about the goals and design of WebAssembly, how would you implement a low-latency compiler?

It's a question worth thinking about so I will give you a bit of space in which to do so.

.

.

.

After spending a lot of time in Firefox's WebAssembly baseline compiler, I have extracted the following principles:

1. The function is the unit of compilation

2. One pass, and one pass only

3. Lean into the stack machine

4. No noodling!

In the remainder of this article we'll look into these individual points. Note, although I have done a good bit of hacking on this compiler, its design and original implementation comes mainly from Mozilla hacker Lars Hansen, who also currently maintains it. All errors of exegesis are mine, of course!

**the function is the unit of compilation**

As we mentioned, in the binary encoding of a WebAssembly module, all definitions needed by any function come before all function definitions. This naturally leads to a partition between two phases of bytestream parsing: an initial serial phase that collects the set of global type definitions, annotations as to which functions are imported and exported, and so on, and a subsequent phase that compiles individual functions in an essentially independent manner.

The advantage of this approach is that compiling functions is a natural task unit of parallelism. If the user has a machine with 8 virtual cores, the web browser can keep one or two cores for the browser itself and farm out WebAssembly compilation tasks to the rest. The result is that the compiled code is available sooner.

Taking functions to be the unit of compilation also allows for an easy "tier-up" mechanism: after the baseline compiler is done, the optimizing compiler can take more time to produce better code, and when it is done, it can swap out the results on a per-function level. All function calls from the baseline compiler go through a jump table indirection, to allow for tier-up. In SpiderMonkey there is no mechanism currently to tier down; if you need to debug WebAssembly code, you need to refresh the page, causing the wasm code to be compiled in debugging mode. For the record, SpiderMonkey can only tier up at function calls (it doesn't do OSR).

This simple approach does have some down-sides, in that it leaves intraprocedural optimizations on the table (inlining, contification, custom calling conventions, speculative optimizations). This is mitigated in two ways, the most obvious being that LLVM or whatever produced the WebAssembly has ideally already done whatever inlining might be fruitful. The second is that WebAssembly is designed for predictable performance. In JavaScript, an implementation needs to do run-time type feedback and speculative optimizations to get good performance, but the result is that it can be hard to understand why a program is fast or slow. The designers and implementers of WebAssembly in browsers all had first-hand experience with JavaScript virtual machines, and actively wanted to avoid unpredictable performance in WebAssembly. Therefore there is currently a kind of détente among the various browser vendors, that everyone has agreed that they won't do speculative inlining -- yet, anyway. Who knows what will happen in the future, though.

Digressing, the summary here is that the baseline compiler receives an individual function body as input, and generates code just for that function.

**one pass, and one pass only**

The WebAssembly baseline compiler makes one pass through the bytecode of a function. Nowhere in all of this are we going to build an abstract syntax tree or a graph of basic blocks. Let's follow through how that works.

Firstly, [`emitFunction`](https://searchfox.org/mozilla-central/source/js/src/wasm/WasmBaselineCompile.cpp#12827) simply emits a prologue, then the body, then an epilogue. `emitBody` is basically a [big loop](https://searchfox.org/mozilla-central/source/js/src/wasm/WasmBaselineCompile.cpp#11813) that consumes opcodes from the instruction stream, dispatching to opcode-specific code emitters (e.g. [`emitAddI32`](https://searchfox.org/mozilla-central/source/js/src/wasm/WasmBaselineCompile.cpp#7440)).

The opcode-specific code emitters are also responsible for validating their arguments; for example, `emitAddI32` is wrapped in an [assertion that there are two `i32` values on the stack](https://searchfox.org/mozilla-central/source/js/src/wasm/WasmBaselineCompile.cpp#11821). This validation logic is shared by a templatized [codestream iterator](https://searchfox.org/mozilla-central/source/js/src/wasm/WasmOpIter.h#549) so that it can be re-used by the optimizing compiler, as well as by the publicly-exposed [`WebAssembly.validate`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/validate) function.

A corollary of this approach is that machine code is emitted in bytestream order; if the WebAssembly instruction stream has an `i32.add` followed by a `i32.sub`, then the machine code will have an `addl` followed by a `subl`.

WebAssembly has a syntactically limited form of non-local control flow; it's not `goto`. Instead, instructions are contained in a tree of nested *control blocks*, and control can only exit nonlocally to a containing control block. There are three kinds of control blocks: jumping to a `block` or an `if` will continue at the end of the block, whereas jumping to a `loop` will continue at its beginning. In either case, as the compiler keeps a stack of nested control blocks, it has the set of valid jump targets and can use the usual assembler logic to patch forward jump addresses when the compiler gets to the block exit.

**lean into the stack machine**

This is the interesting bit! So, WebAssembly instructions target a stack machine. That is to say, there's an abstract stack onto which evaluating `i32.const 32` pushes a value, and if followed by `i32.const 10` there would then be `i32(32) | i32(10)` on the stack (where new elements are added on the right). A subsequent `i32.add` would pop the two values off, and push on the result, leaving the stack as `i32(42)`. There is also a fixed set of local variables, declared at the beginning of the function.

The easiest thing that a compiler can do, then, when faced with a stack machine, is to emit code for a stack machine: as values are pushed on the abstract stack, emit code that pushes them on the machine stack.

The downside of this approach is that you emit a fair amount of code to do read and write values from the stack. Machine instructions generally take arguments from registers and write results to registers; going to memory is a bit superfluous. We're willing to accept suboptimal code generation for this quick-and-dirty compiler, but isn't there something smarter we can do for ephemeral intermediate values?

Turns out -- yes! The baseline compiler keeps an abstract [*value stack*](https://searchfox.org/mozilla-central/source/js/src/wasm/WasmBaselineCompile.cpp#2097) as it compiles. For example, compiling `i32.const 32` pushes nothing on the machine stack: it just adds a `ConstI32` node to the value stack. When an instruction needs an operand that turns out to be a `ConstI32`, it can either encode the operand as an immediate argument or [load it into a register](https://searchfox.org/mozilla-central/source/js/src/wasm/WasmBaselineCompile.cpp#3730).

Say we are evaluating the `i32.add` discussed above. After the add, where does the result go? For the baseline compiler, the answer is always "in a register" via pushing a new `RegisterI32` entry on the value stack. The baseline compiler includes a [stupid register allocator](https://searchfox.org/mozilla-central/source/js/src/wasm/WasmBaselineCompile.cpp#742) that [spills the value stack to the machine stack if no register is available](https://searchfox.org/mozilla-central/source/js/src/wasm/WasmBaselineCompile.cpp#3456), updating value stack entries from e.g. `RegisterI32` to `MemI32`. Note, a `ConstI32` never needs to be spilled: its value can always be reloaded as an immediate.

The end result is that the baseline compiler avoids lots of stack store and load code generation, which speeds up the compiler, and happens to make faster code as well.

Note that there is one limitation, currently: control-flow joins can have multiple predecessors and can pass a value (in the current WebAssembly specification), so the allocation of that value needs to be agreed-upon by all predecessors. As in this code:

```
(func $f (param $arg i32) (result i32)
  (block $b (result i32)
    (i32.const 0)
    (local.get $arg)
    (i32.eqz)
    (br_if $b) ;; return 0 from $b if $arg is zero
    (drop)
    (i32.const 1))) ;; otherwise return 1
;; result of block implicitly returned

```

When the `br_if` branches to the block end, where should it put the result value? The baseline compiler effectively punts on this question and just puts it in a well-known register (e.g., `$rax` on x86-64). Results for block exits are the only place where WebAssembly has "phi" variables, and the baseline compiler allocates all integer phi variables to the same register. A hack, but there we are.

**no noodling!**

When I started to hack on the baseline compiler, I did a lot of code reading, and eventually came on code like this:

```
void BaseCompiler::emitAddI32() {
  int32_t c;
  if (popConstI32(&c)) {
    RegI32 r = popI32();
    masm.add32(Imm32(c), r);
    pushI32(r);
  } else {
    RegI32 r, rs;
    pop2xI32(&r, &rs);
    masm.add32(rs, r);
    freeI32(rs);
    pushI32(r);
  }
}

```

I said to myself, this is silly, why are we only emitting the add-immediate code if the constant is on top of the stack? What if instead the constant was the deeper of the two operands, why do we then load the constant into a register? I asked on the chat channel if it would be OK if I improved codegen here and got a response I was not expecting: no noodling!

The reason is, performance of baseline-compiled code essentially doesn't matter. Obviously let's not pessimize things but the reason there's a baseline compiler is to emit code quickly. If we start to add more code to the baseline compiler, the compiler itself will slow down.

For that reason, changes are only accepted to the baseline compiler if they are necessary for some reason, *or* if they improve latency as measured using some real-world benchmark (time-to-first-frame on Doom 3, for example).

This to me was a real eye-opener: a compiler optimized not for the quality of the code that it generates, but rather for how fast it can produce the code. I had seen this in action before but this example really brought it home to me.

The focus on compiler throughput rather than compiled-code throughput makes it pretty gnarly to hack on the baseline compiler -- care has to be taken when adding new features not to significantly regress the old. It is much more like hacking on a production JavaScript parser than your traditional SSA-based compiler.

**that's a wrap!**

So that's the WebAssembly baseline compiler in SpiderMonkey / Firefox. Until the next time, happy hacking!

## related articles

- [understanding webassembly code generation throughput](/archives/2020/04/14/understanding-webassembly-code-generation-throughput)
- [multi-value webassembly in firefox: a binary interface](/archives/2020/04/08/multi-value-webassembly-in-firefox-a-binary-interface)
- [multi-value webassembly in firefox: from 1 to n](/archives/2020/04/03/multi-value-webassembly-in-firefox-from-1-to-n)
- [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)
- [es6 generator and array comprehensions in spidermonkey](/archives/2014/03/07/es6-generator-and-array-comprehensions-in-spidermonkey)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)

### One response

1. [Marius Gedminas](https://gedmin.as) says:[27 March 2020 3:34 PM](#1ba9729696887cc8b254a4a97c05e9bb7e0df612)

   Would you mind defining "OSR"? On-Stack Replacement? Wikipedia lists the term on the disambiguation page but has no article about it; Stack Overflow seems to have some explanation at https://stackoverflow.com/a/9105846/110151. It seems to match the context.

Comments are closed.
