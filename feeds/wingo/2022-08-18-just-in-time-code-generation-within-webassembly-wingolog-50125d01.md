---
title: just-in-time code generation within webassembly — wingolog
url: https://wingolog.org/archives/2022/08/18/just-in-time-code-generation-within-webassembly
published: "2022-08-18T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2022/08/18/just-in-time-code-generation-within-webassembly
---

# just-in-time code generation within webassembly — wingolog

## [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)

18 August 2022 3:36 PM

- [webassembly](/tags/webassembly)
- [dynamic languages](/tags/dynamic%20languages)
- [jit](/tags/jit)
- [compilers](/tags/compilers)
- [guile](/tags/guile)
- [igalia](/tags/igalia)
- [scheme](/tags/scheme)
- [wasi](/tags/wasi)
- [python](/tags/python)
- [wasmtime](/tags/wasmtime)
- [javascript](/tags/javascript)
- [spidermonkey](/tags/spidermonkey)

Just-in-time (JIT) code generation is an important tactic when implementing a programming language. Generating code at run-time allows a program to specialize itself against the specific data it is run against. For a program that implements a programming language, that specialization is with respect to the program being run, and possibly with respect to the data that program uses.

The way this typically works is that the program generates bytes for the instruction set of the machine it’s running on, and then transfers control to those instructions.

Usually the program has to put its generated code in memory that is specially marked as executable. However, this capability is missing in WebAssembly. How, then, to do just-in-time compilation in WebAssembly?

### webassembly as a harvard architecture

In a von Neumman machine, like the ones that you are probably reading this on, code and data share an address space. There’s only one kind of pointer, and it can point to anything: the bytes that implement the `sin` function, the number `42`, the characters in `"biscuits"`, or anything at all. WebAssembly is different in that its code is not addressable at run-time. Functions in a WebAssembly module are numbered sequentially from 0, and the WebAssembly `call` instruction takes the callee as an immediate parameter.

So, to add code to a WebAssembly program, somehow you’d have to augment the program with more functions. Let’s assume we will make that possible somehow – that your WebAssembly module that had N functions will now have N+1 functions, and with function N being the new one your program generated. How would we call it? Given that the `call` instructions hard-code the callee, the existing functions 0 to N-1 won’t call it.

Here the answer is [`call_indirect`](https://webassembly.github.io/spec/core/exec/instructions.html#xref-syntax-instructions-syntax-instr-control-mathsf-call-indirect-x-y). A bit of a reminder, this instruction take the callee as an operand, not an immediate parameter, allowing it to choose the callee function at run-time. The callee operand is an index into a *table* of functions. Conventionally, table 0 is called the *indirect function table* as it contains an entry for each function which might ever be the target of an indirect call.

With this in mind, our problem has two parts, then: (1) how to augment a WebAssembly module with a new function, and (2) how to get the original module to call the new code.

### late linking of auxiliary webassembly modules

The key idea here is that to add code, the main program should generate a new WebAssembly module containing that code. Then we run a linking phase to actually bring that new code to life and make it available.

System linkers like `ld` typically require a complete set of symbols and relocations to resolve inter-archive references. However when performing a late link of JIT-generated code, we can take a short-cut: the main program can embed memory addresses directly into the code it generates. Therefore the generated module would import memory from the main module. All references from the generated code to the main module can be directly embedded in this way.

The generated module would also import the indirect function table from the main module. (We would ensure that the main module exports its memory and indirect function table via the toolchain.) When the main module makes the generated module, it also embeds a special `patch` function in the generated module. This function would add the new functions to the main module’s indirect function table, and perform any relocations onto the main module’s memory. All references from the main module to generated functions are installed via the `patch` function.

We plan on two implementations of late linking, but both share the fundamental mechanism of a generated WebAssembly module with a `patch` function.

### dynamic linking via the run-time

One implementation of a linker is for the main module to cause the run-time to dynamically instantiate a new WebAssembly module. The run-time would provide the memory and indirect function table from the main module as imports when instantiating the generated module.

The advantage of dynamic linking is that it can update a live WebAssembly module without any need for re-instantiation or special run-time checkpointing support.

In the context of the web, JIT compilation can be triggered by the WebAssembly module in question, by calling out to functionality from JavaScript, or we can use a “pull-based” model to allow the JavaScript host to poll the WebAssembly instance for any pending JIT code.

For WASI deployments, you need a capability from the host. Either you import a module that provides run-time JIT capability, or you rely on the host to poll you for data.

### static linking via wizer

Another idea is to build on [Wizer](https://github.com/bytecodealliance/wizer)‘s ability to take a snapshot of a WebAssembly module. You could extend Wizer to also be able to augment a module with new code. In this role, Wizer is effectively a late linker, linking in a new archive to an existing object.

Wizer already needs the ability to instantiate a WebAssembly module and to run its code. Causing Wizer to ask the module if it has any generated auxiliary module that should be instantiated, patched, and incorporated into the main module should not be a huge deal. Wizer can already run the `patch` function, to perform relocations to patch in access to the new functions. After having done that, Wizer (or some other tool) would need to snapshot the module, as usual, but also adding in the extra code.

As a technical detail, in the simplest case in which code is generated in units of functions which don’t directly call each other, this is as simple as just appending the functions to the code section and then and appending the generated [element segments](https://webassembly.github.io/spec/core/exec/modules.html#element-segments) to the main module’s element segment, updating the appended function references to their new values by adding the total number of functions in the module before the new module was concatenated to each function reference.

### late linking appears to be async codegen

From the perspective of a main program, WebAssembly JIT code generation via late linking appears the same as aynchronous code generation.

For example, take the C program:

```
struct Value;
struct Func {
  struct Expr *body;
  void *jitCode;
};

void recordJitCandidate(struct Func *func);
uint8_t* flushJitCode(); // Call to actually generate JIT code.

struct Value* interpretCall(struct Expr *body,
                            struct Value *arg);

struct Value* call(struct Func *func,
                   struct Value* val) {
  if (func->jitCode) {
    struct Value* (*f)(struct Value*) = jitCode;
    return f(val);
  } else {
    recordJitCandidate(func);
    return interpretCall(func->body, val);
  }
}

```

Here the C program allows for the possibility of JIT code generation: there is a slot in a `Func` instance to fill in with a code pointer. If this program generates code for a given `Func`, it won’t be able to fill in the pointer – it can’t add new code to the image. But, it could tell Wizer to do so, and Wizer could snapshot the program, link in the new function, and patch `&func->jitCode`. From the program’s perspective, it’s as if the code becomes available asynchronously.

### demo!

So many words, right? Let’s see some code! As a sketch for other JIT compiler work, I implemented a little Scheme interpreter and JIT compiler, targetting WebAssembly. See [`interp.cc`](https://github.com/wingo/wasm-jit/blob/main/interp.cc) for the source. You compile it like this:

```
$ /opt/wasi-sdk/bin/clang++ -O2 -Wall \
   -mexec-model=reactor \
   -Wl,--growable-table \
   -Wl,--export-table \
   -DLIBRARY=1 \
   -fno-exceptions \
   interp.cc -o interplib.wasm

```

Here we are compiling with [WASI SDK](https://github.com/WebAssembly/wasi-sdk/releases). I have version 14.

The `-mexec-model=reactor` argument means that this WASI module isn’t just a run-once thing, after which its state is torn down; rather it’s a multiple-entry component.

The two `-Wl,` options tell the linker to export the indirect function table, and to allow the indirect function table to be augmented by the JIT module.

The `-DLIBRARY=1` is used by `interp.cc`; you can actually run and debug it natively but that’s just for development. We’re instead compiling to wasm and running with a WASI environment, giving us `fprintf` and other debugging niceties.

The `-fno-exceptions` is because WASI doesn’t support exceptions currently. Also we don’t need them.

WASI is mainly for non-browser use cases, but this module does so little that it doesn’t need much from WASI and I can just polyfill it in browser JavaScript. So that’s what we have here:

**loading wasm-jit...**



Run JIT!

Each time you enter a Scheme expression, it will be parsed to an internal tree-like intermediate language. You can then run a recursive interpreter over that tree by pressing the “Evaluate” button. Press it a number of times, you should get the same result.

As the interpreter runs, it records any closures that it created. The `Func` instances attached to the closures have a slot for a C++ function pointer, which is initially NULL. Function pointers in WebAssembly are indexes into the indirect function table; the first slot is kept empty so that calling a NULL pointer (a pointer with value 0) causes an error. If the interpreter gets to a closure call and the closure’s function’s JIT code pointer is NULL, it will interpret the closure’s body. Otherwise it will call the function pointer.

If you then press the “JIT” button above, the module will [assemble a fresh WebAssembly module containing JIT code](https://github.com/wingo/wasm-jit/blob/main/interp.cc#L809) for the closures that it saw at run-time. Obviously that’s just one heuristic: you could be more eager or more lazy; this is just a detail.

Although the particular JIT compiler isn’t much of interest—the point being to see JIT code generation at all—it’s nice to see that the fibonacci example sees a good speedup; try it yourself, and try it on different browsers if you can. Neat stuff!

### not just the web

I was wondering how to get something like this working in a non-webby environment and it turns out that [the Python interface to wasmtime](https://github.com/bytecodealliance/wasmtime-py/) is just the thing. I wrote a little [`interp.py`](https://github.com/wingo/wasm-jit/blob/main/interp.py) harness that can do the same thing that we can do on the web; just run as `python3 interp.py`, after having `pip3 install wasmtime`:

```
$ python3 interp.py
...
Calling eval(0x11eb0) 5 times took 1.716s.
Calling jitModule()
jitModule result:
Instantiating and patching in JIT module
...
Calling eval(0x11eb0) 5 times took 1.161s.

```

Interestingly it would appear that the performance of wasmtime’s code (0.232s/invocation) is somewhat better than both SpiderMonkey (0.392s) and V8 (0.729s).

### reflections

This work is just a proof of concept, but it’s a step in a particular direction. As part of [previous work with Fastly](https://bytecodealliance.org/articles/making-javascript-run-fast-on-webassembly), we enabled the SpiderMonkey JavaScript engine to run on top of WebAssembly. When combined with pre-initialization via Wizer, you end up with a system that can start in microseconds: fast enough to instantiate a fresh, shared-nothing module on every HTTP request, for example.

The SpiderMonkey-on-WASI work left out JIT compilation, though, because, you know, WebAssembly doesn’t support JIT compilation. JavaScript code actually ran via the C++ bytecode interpreter. But as we just found out, actually you can compile the bytecode: just-in-time, but at a different time-scale. What if you took a SpiderMonkey interpreter, pre-generated WebAssembly code for a user’s JavaScript file, and then combined them into a single freeze-dried WebAssembly module via Wizer? You get the benefits of fast startup while also getting decent baseline performance. There are many engineering considerations here, but as part of work sponsored by Shopify, we have made good progress in this regard; details in another missive.

I think a kind of “offline JIT” has a lot of value for deployment environments like Shopify’s and Fastly’s, and you don’t have to limit yourself to “total” optimizations: you can still collect and incorporate type feedback, and you get the benefit of taking advantage of adaptive optimization without having to actually run the JIT compiler at run-time.

But if we think of more traditional “online JIT” use cases, it’s clear that relying on host JIT capabilities, while a good MVP, is not optimal. For one, you would like to be able to freely emit direct calls from generated code to existing code, instead of having to call indirectly or via imports. I think it still might make sense to have a language run-time express its generated code in the form of a WebAssembly module, though really you might want native support for compiling that code (asynchronously) from within WebAssembly itself, without calling out to a run-time. Most people I have talked to that work on WebAssembly implementations in JS engines believe that a JIT proposal will come some day, but it’s good to know that we don’t have to wait for it to start generating code and taking advantage of it.

### & out

If you want to play around with the demo, do take a look at the [wasm-jit](https://github.com/wingo/wasm-jit) Github project; it’s fun stuff. Happy hacking, and until next time!

## related articles

- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)

### One response

1. [Remko Tronçon](https://el-tramo.be) says:[18 August 2022 4:48 PM](#8fd4fdae99eb0a256b5c4e77f94b3c49c90162af)

   Great article!

   FYI, WAForth ( https://github.com/remko/waforth ) employs similar techniques (implemented in raw WebAssembly).

   On the plan is also providing a postprocessing phase on all JIT-compiled modules, replacing all call\_indirects by calls, and getting a more 'optimized' binary as a result.

Comments are closed.
