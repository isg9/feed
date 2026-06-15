---
title: accessing webassembly reference-typed arrays from c++ — wingolog
url: https://wingolog.org/archives/2022/08/23/accessing-webassembly-reference-typed-arrays-from-c
published: "2022-08-23T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2022/08/23/accessing-webassembly-reference-typed-arrays-from-c
---

# accessing webassembly reference-typed arrays from c++ — wingolog

## [accessing webassembly reference-typed arrays from c++](/archives/2022/08/23/accessing-webassembly-reference-typed-arrays-from-c)

23 August 2022 10:35 AM

- [webassembly](/tags/webassembly)
- [compilers](/tags/compilers)
- [llvm](/tags/llvm)
- [emscripten](/tags/emscripten)
- [igalia](/tags/igalia)
- [gc](/tags/gc)
- [reference types](/tags/reference%20types)
- [clang](/tags/clang)

The WebAssembly [garbage collection proposal](https://github.com/WebAssembly/gc/blob/main/proposals/gc/MVP.md) is coming soonish (really!) and will extend WebAssembly with the the capability to create and access arrays whose memory is automatically managed by the host. As long as some system component has a reference to an array, it will be kept alive, and as soon as nobody references it any more, it becomes "garbage" and is thus eligible for collection.

(In a way it's funny to define the proposal this way, in terms of what happens to garbage objects that by definition aren't part of the program's future any more; really the interesting thing is the new things you can do with live data, defining new data types and representing them outside of linear memory and passing them between components without copying. But "extensible-arrays-structs-and-other-data-types" just isn't as catchy as "GC". Anyway, I digress!)

One potential use case for garbage-collected arrays is for passing large buffers between parts of a WebAssembly system. For example, a webcam driver could produce a stream of frames as reference-typed arrays of bytes, and then pass them by reference to a sandboxed WebAssembly instance to, I don't know, identify cats in the images or something. You get the idea. Reference-typed arrays let you avoid copying large video frames.

A lot of image-processing code is written in C++ or Rust. With WebAssembly 1.0, you just have linear memory and no reference-typed values, which works well for these languages that like to think of memory as having a single address space. But once you get reference-typed arrays in the mix, you effectively have multiple address spaces: you can't address the contents of the array using a normal pointer, as you might be able to do if you `mmap`'d the buffer into a program's address space. So what do you do?

**reference-typed values are special**

The broader question of C++ and GC-managed arrays is, well, too broad for today. The set of array types is infinite, because it's not just arrays of `i32`, it's also arrays of arrays of `i32`, and arrays of those, and arrays of records, and so on.

So let's limit the question to just arrays of `i8`, to see if we can make some progress. So imagine a C function that takes an array of `i8`:

```
void process(array_of_i8 array) {
  // ?
}

```

If you know WebAssembly, there's a clear translation of the sort of code that we want:

```
(func (param $array (ref (array i8)))
  ; operate on local 0
  )

```

The WebAssembly function will have an array as a parameter. But, here we start to run into more problems with the LLVM toolchain that we use to compile C and other languages to WebAssembly. When the C front-end of LLVM (clang) compiles a function to the LLVM middle-end's intermediate representation (IR), it models all local variables (including function parameters) as mutable memory locations created with `alloca`. Later optimizations might turn these memory locations back to SSA variables and thence to registers or stack slots. But, a reference-typed value has no bit representation, and it can't be stored to linear memory: there is no alloca that can hold it.

Incidentally this problem is not isolated to future extensions to WebAssembly; the `externref` and `funcref` data types that landed in WebAssembly 2.0 and in all browsers are also reference types that can't be written to main memory. Similarly, the table data type which is also part of shipping WebAssembly is not dissimilar to GC-managed arrays, except that they are statically allocated at compile-time.

At Igalia, my colleagues Paulo Matos and Alex Bradbury have been hard at work to solve this gnarly problem and finally expose reference-typed values to C. The full details and final vision are probably a bit too much for this article, but some bits on the mechanism will help.

Firstly, note that LLVM has a fairly traditional breakdown between front-end (clang), middle-end ("the IR layer"), and back-end ("the MC layer"). The back-end can be quite target-specific, and though it can be annoying, we've managed to get fairly good support for reference types there.

In the IR layer, we are currently representing GC-managed values as opaque pointers into non-default, non-integral address spaces. LLVM attaches an address space (an integer less than 224 or so) to each pointer, mostly for OpenCL and GPU sorts of use-cases, and we abuse this to prevent LLVM from doing much reasoning about these values.

This is a bit of a theme, incidentally: get the IR layer to avoid assuming anything about reference-typed values. We're fighting the system, in a way. As another example, because LLVM is really oriented towards lowering high-level constructs to low-level machine operations, it doesn't necessarily preserve types attached to pointers on the IR layer. Whereas for WebAssembly, we need exactly that: we reify types when we write out WebAssembly object files, and we need LLVM to pass some types through from front-end to back-end unmolested. We've had to change tack a number of times to get a good way to preserve data from front-end to back-end, and probably will have to do so again before we end up with a final design.

Finally on the front-end we need to generate an `alloca` in different address spaces depending on the type being allocated. And because reference-typed arrays can't be stored to main memory, there are semantic restrictions as to how they can be used, which need to be enforced by clang. Fortunately, this set of restrictions is similar enough to what is imposed by the ARM C Language Extensions (ACLE) for scalable vector (SVE) values, which also don't have a known bit representation at compile-time, so we can piggy-back on those. [This patch hasn't landed yet](https://reviews.llvm.org/D122215), but who knows, it might land soon; in the mean-time we are going to run ahead of upstream a bit to show how you might define and use an array type definition. Further tacks here are also expected, as we try to thread the needle between exposing these features to users and not imposing too much of a burden on clang maintenance.

**accessing array contents**

All this is a bit basic, though; it just gives you enough to have a local variable or a function parameter of a reference-valued type. Let's continue our example:

```
void process(array_of_i8 array) {
  uint32_t sum;
  for (size_t idx = 0; i < __builtin_wasm_array_length(array); i++)
    sum += (uint8_t)__builtin_wasm_array_ref_i8(array, idx);
  // ...
}

```

The most basic way to extend C to access these otherwise opaque values is to expose some builtins, say `__builtin_wasm_array_length` and so on. Probably you need different intrinsics for each scalar array element type ( `i8`, `i16`, and so on), and one for arrays which return reference-typed values. We'll talk about arrays of references another day, but focusing on the `i8` case, the C builtin then lowers to a dedicated [LLVM intrinsic](https://llvm.org/docs/LangRef.html#id1780), which passes through the middle layer unscathed.

In C++ I think we can provide some nicer syntax which preserves the syntactic illusion of array access.

I think this is going to be sufficient as an MVP, but there's one caveat: SIMD. You can indeed have an array of `i128` values, but you can only access that array's elements as `i128`; worse, you can't load multiple data from an `i8` array as `i128` or even `i32`.

Compare this to to the [memory control proposal](https://github.com/WebAssembly/memory-control/blob/master/proposals/memory-control/Overview.md), which instead proposes to map buffers to non-default memories. In WebAssembly, you can in theory ( [and perhaps soon in practice](https://github.com/WebAssembly/multi-memory/blob/main/proposals/multi-memory/Overview.md)) have multiple memories. The easiest way I can see on the toolchain side is to use the address space feature in clang:

```
void process(uint8_t *array __attribute__((address_space(42))),
             size_t len) {
  uint32_t sum;
  for (size_t idx = 0; i < len; i++)
    sum += array[idx];
  // ...
}

```

How exactly to plumb the mapping between address spaces which can only be specified by number from the front-end to the back-end is a little gnarly; really you'd like to declare the set of address spaces that a compilation unit uses symbolically, and then have the linker produce a final allocation of memory indices. But I digress, it's clear that with this solution we can use SIMD instructions to load multiple bytes from memory at a time, so it's a winner with respect to accessing GC arrays.

Or is it? Perhaps there could be SIMD extensions for packed GC arrays. I think it makes sense, but it's a fair amount of (admittedly somewhat mechanical) specification and implementation work.

**& future**

In some future bloggies we'll talk about how we will declare new reference types: first some basics, then some [more integrated visions for reference types and C++](https://github.com/Igalia/ref-cpp). Lots going on, and this is just a brain-dump of the current state of things; thoughts are very much welcome.

## related articles

- [on "binary security of webassembly"](/archives/2020/10/15/on-binary-security-of-webassembly)
- [malloc as a service](/archives/2020/10/13/malloc-as-a-service)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)

Comments are closed.
