---
title: wastrel, a profligate implementation of webassembly — wingolog
url: https://wingolog.org/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly
published: "2025-10-30T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly
---

# wastrel, a profligate implementation of webassembly — wingolog

## [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)

30 October 2025 10:19 PM

- [webassembly](/tags/webassembly)
- [wasm](/tags/wasm)
- [gc](/tags/gc)
- [whippet](/tags/whippet)
- [c](/tags/c)
- [wastrel](/tags/wastrel)
- [whiffle](/tags/whiffle)
- [compilers](/tags/compilers)
- [igalia](/tags/igalia)
- [guile](/tags/guile)

Hey hey hey good evening! Tonight a quick note on [wastrel](https://codeberg.org/andywingo/wastrel), a new WebAssembly implementation.

### a wasm-to-native compiler that goes through c

Wastrel compiles Wasm modules to standalone binaries. It does so by emitting C and then compiling that C.

Compiling Wasm to C isn’t new: [Ben Smith](https://binji.github.io/) wrote [wasm2c](https://github.com/WebAssembly/wabt/tree/main/wasm2c) back in the day and these days most people in this space use [Bastien
Müller](https://turbolent.com/)‘s [w2c2](https://github.com/turbolent/w2c2). These are great projects!

Wastrel has two or three minor differences from these projects. Let’s lead with the most important one, despite the fact that it’s as yet vaporware: Wastrel aims to support automatic memory managment via [WasmGC](https://v8.dev/blog/wasm-gc-porting), by embedding the [Whippet
garbage collection library](https://github.com/wingo/whippet). (For the wingolog faithful, you can think of Wastrel as a [Whiffle](https://wingolog.org/archives/2023/11/13/i-accidentally-a-scheme) for Wasm.) This is the whole point! But let’s come back to it.

The other differences are minor. Firstly, the CLI is more like [wasmtime](https://wasmtime.dev): instead of privileging the production of C, which you then incorporate into your project, Wastrel also compiles the C (by default), and even runs it, like `wasmtime run`.

Unlike wasm2c (but like w2c2), Wastrel implements [WASI](https://wasi.dev). Specifically, WASI 0.1, sometimes known as “WASI preview 1”. It’s nice to be able to take the [wasi-sdk](https://github.com/WebAssembly/wasi-sdk)‘s C compiler, compile your program to a binary that uses WASI imports, and then run it directly.

In a past life, I once took a week-long sailing course on a 12-meter yacht. One thing that comes back to me often is the way the instructor would insist on taking in the bumpers immediately as we left port, that to sail with them was *no muy marinero*, not very seamanlike. Well one thing about Wastrel is that it emits nice C: nice in the sense that it avoids many useless temporaries. It does so with a lightweight [effects
analysis](https://wingolog.org/archives/2014/05/18/effects-analysis-in-guile), in which as temporaries are produced, they record which bits of the world they depend on, in a coarse way: one bit for the contents of all global state (memories, tables, globals), and one bit for each local. When compiling an operation that writes to state, we flush all temporaries that read from that state (but only that state). It’s a small thing, and I am sure it has very little or zero impact after [SROA](https://llvm.org/doxygen/SROA_8cpp_source.html) turns locals into SSA values, but we are vessels of the divine, and it is important for vessels to be C worthy.

Finally, w2c2 at least is built in such a way that you can instantiate a module multiple times. Wastrel doesn’t do that: the Wasm instance is statically allocated, once. It’s a restriction, but that’s the use case I’m going for.

### on performance

Oh buddy, who knows?!? What is real anyway? I would love to have proper perf tests, but in the meantime, I compiled [coremark](https://github.com/eembc/coremark) using my GCC on x86-64 (-02, no other options), then also compiled it with the current wasi-sdk and then ran with w2c2, wastrel, and wasmtime. I am well aware of the many pitfalls of benchmarking, and so I should not say anything because it is irresponsible to make conclusions from useless microbenchmarks. However, we’re all friends here, and I am a dude with hubris who also believes blogs are better out than in, and so I will give some small indications. Please obtain your own salt.

So on coremark, Wastrel is some 2-5% percent slower than native, and w2c2 is some 2-5% slower than that. Wasmtime is 30-40% slower than GCC. Voilà.

My conclusion is, Wastrel provides state-of-the-art performance. Like w2c2. It’s no wonder, these are simple translators that use industrial compilers underneath. But it’s neat to see that performance is close to native.

### on wasi

OK this is going to sound incredibly arrogant but here it is: writing Wastrel was easy. I have worked on Wasm for a while, and on [Firefox’s
baseline
compiler](https://wingolog.org/archives/2020/03/25/firefoxs-low-latency-webassembly-compiler), and Wastrel is kinda like a baseline compiler in shape: it just has to avoid emitting boneheaded code, and can leave the serious work to someone else (Ion in the case of Firefox, GCC in the case of Wastrel). I just had to use the [Wasm libraries I already
had](https://codeberg.org/spritely/hoot/src/branch/main/module/wasm) and make it emit some C for each instruction. It took 2 days.

WASI, though, took two and a half weeks of agony. Three reasons: One, you can be sloppy when implementing just wasm, but when you do WASI you have to implement an ABI using sticks and glue, but you have no glue, it’s all just `i32`. Truly excruciating, it makes you doubt everything, and I had to refactor Wastrel to use C’s meager type system to the max. (Basically, structs-as-values to avoid type confusion, but via inline functions to avoid overhead.)

Two, WASI is not huge but not tiny either. Implementing [`poll_oneoff`](https://github.com/WebAssembly/WASI/blob/main/legacy/preview1/docs.md#poll_oneoff) is annoying. And so on. Wastrel’s [WASI
implementation](https://codeberg.org/andywingo/wastrel/src/branch/main/resource/wasip1.h) is thin but it’s still a couple thousand lines of code.

Three, WASI is underspecified, and in practice what is “conforming” is a function of what the Rust and C toolchains produce. I used [wasi-testsuite](https://github.com/WebAssembly/wasi-testsuite) to burn down most of the issues, but it was a slog. I neglected email and important things but now things pass so it was worth it maybe? Maybe?

### on wasi’s filesystem sandboxing

WASI preview 1 has this [“rights”](https://github.com/WebAssembly/WASI/blob/main/legacy/preview1/docs.md#rights) interface that associated capabilities with file descriptors. I think it was an attempt at replacing and expanding file permissions with a capabilities-oriented security approach to sandboxing, but it was only a veneer. In practice most WASI implementations effectively implement the sandbox via a permissions layer: for example the process has capabilities to access the parents of preopened directories via `..`, but the WASI implementation has to actively prevent this capability from leaking to the compiled module via run-time checks.

Wastrel takes a different approach, which is to use Linux’s filesystem namespaces to build a tree in which only the exposed files are accessible. No run-time checks are necessary; the system is secure by construction. He says. It’s very hard to be categorical in this domain but a true capabilities-based approach is the only way I can have any confidence in the results, and that’s what I did.

The upshot is that Wastrel is only for Linux. And honestly, if you are on MacOS or Windows, what are you doing with your life? I get that it’s important to meet users where they are but it’s just gross to build on a corporate-controlled platform.

The current versions of WASI keep a vestigial capabilities-based API, but given that the goal is to compile POSIX programs, I would prefer if [wasi-filesystem](https://github.com/WebAssembly/wasi-filesystem) leaned into the approach of WASI just having access to a filesystem instead of a small set of descriptors plus scoped `openat`, `linkat`, and so on APIs. The security properties would be the same, except with fewer bug possibilities and with a more conventional interface.

### on wtf

So Wastrel is Wasm to native via C, but with an as-yet-unbuilt GC aim. Why?

This is hard to explain and I am still workshopping it.

Firstly I am annoyed at the WASI working group’s focus on shared-nothing architectures as a principle of composition. Yes, it works, but garbage collection also works; we could be building different, simpler systems if we leaned in to a more capable virtual machine. Many of the problems that WASI is currently addressing are ownership-related, and would be comprehensively avoided with automatic memory management. Nobody is really pushing for GC in this space and I would like for people to be able to build out counterfactuals to the shared-nothing orthodoxy.

Secondly there are quite a number of languages that are targetting WasmGC these days, and it would be nice for them to have a good run-time outside the browser. I know that Wasmtime is working on GC, but it needs competition :)

Finally, and selfishly, [I have a GC library](https://github.com/wingo/whippet)! I would love to spend more time on it. One way that can happen is for it to prove itself useful, and maybe a Wasm implementation is a way to do that. Could Wastrel on [`wasm_of_ocaml`](https://github.com/ocaml-wasm/wasm_of_ocaml) output beat `ocamlopt`? I don’t know but it would be worth it to find out! And I would love to get Guile programs compiled to native, and perhaps with [Hoot](https://codeberg.org/spritely/hoot) and Whippet and Wastrel that is a possibility.

Welp, there we go, blog out, dude to bed. Hack at y’all later and wonderful wasming to you all!

## related articles

- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [six thoughts on generating c](/archives/2026/02/09/six-thoughts-on-generating-c)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [preliminary notes on a nofl field-logging barrier](/archives/2024/10/03/preliminary-notes-on-a-nofl-field-logging-barrier)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)

### 2 responses

1. [Nick Fitzgerald](https://fitzgen.com) says:[3 November 2025 6:29 PM](#14fe83ecbf0ac3067c54d3a70824f7a2d7f19abe)

   Very cool! Excited to see where this goes.

   Nobody is really pushing for GC in this space

   FWIW, I wrote up https://github.com/WebAssembly/component-model/issues/525 which sketches GC support in the component model’s canonical ABI which would in turn allow GC languages to speak directly with WASI APIs. I prototyped it in `wasm-tools` and started prototyping it in Wasmtime. Shifting priorities pulled me elsewhere, but I’ll get back to it eventually.

   But also GC won’t magically Make Everything Better. Sure, if the data being passed across the ABI boundary is immutable, and its representation matches what the other side expects (e.g. types defined within the same rec groups, subtyping, and whatnot), then we can avoid forcing copies and pass shared references to the producer’s GC objects directly. But if a toolchain’s (or program’s) at-rest representation of the data isn’t compatible with the ABI, then they will need to make a copy anyways before passing it across the ABI boundary. And the same on the consumer side: if they want to consume the data in a different representation from what the ABI gives them, they’ll need to transform it as well.

   I think this kind of ABI mismatch will be more common than one might think, unfortunately, due to, for example, a language’s string type being represented not as an `(array i8)` directly, but as something like `(sub $my-lang-top-type (struct $super-fields... (array i8)))` where the super type fields contain like a type id and/or a virtual table or similar. I tried to alleviate this to some degree in the above-linked issue, and the way it is sort of generic over mutability, subtyping, and rec groups, but fully resolving the issue requires a lot more machinery, flexibility, and complexity than exists today and which we might not collectively have the appetite for.

   Happy hacking, Nick

2. [Nick Fitzgerald](https://fitzgen.com) says:[3 November 2025 6:31 PM](#13fe408667825302086677982bb53d5a22e6e021)

   In my previous comment, “Nobody is really pushing for GC in this space” was supposed to be prefixed by a greater-than sign, but it seems the comment system stripped it rather than turning it into “and gt ;”.

   For confused readers: that bit is a quote from the article and below it is my response to that quote.

Comments are closed.
