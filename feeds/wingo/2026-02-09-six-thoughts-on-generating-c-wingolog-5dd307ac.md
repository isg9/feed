---
title: six thoughts on generating c — wingolog
url: https://wingolog.org/archives/2026/02/09/six-thoughts-on-generating-c
published: "2026-02-09T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2026/02/09/six-thoughts-on-generating-c
---

# six thoughts on generating c — wingolog

## [six thoughts on generating c](/archives/2026/02/09/six-thoughts-on-generating-c)

9 February 2026 1:47 PM

- [c](/tags/c)
- [compilers](/tags/compilers)
- [wastrel](/tags/wastrel)
- [whippet](/tags/whippet)
- [igalia](/tags/igalia)
- [gc](/tags/gc)
- [types](/tags/types)
- [rust](/tags/rust)

So I work in compilers, which means that I write programs that translate programs to programs. Sometimes you will want to target a language at a higher level than just, like, assembler, and oftentimes C is that language. Generating C is less fraught than writing C by hand, as the generator can often avoid the undefined-behavior pitfalls that one has to be so careful about when writing C by hand. Still, I have found some patterns that help me get good results.

Today’s note is a quick summary of things that work for me. I won’t be so vain as to call them “best practices”, but they are my practices, and you can have them too if you like.

### static inline functions enable data abstraction

When I learned C, in the early days of [GStreamer](https://gstreamer.freedesktop.org/) (oh bless its heart it still has the same web page!), we used lots of preprocessor macros. Mostly we got the message over time that [many macro uses should have
been inline functions](https://gcc.gnu.org/onlinedocs/gcc/Inline.html); macros are for token-pasting and generating names, not for data access or other implementation.

But what I did not appreciate until much later was that always-inline functions remove any possible performance penalty for data abstractions. For example, in [Wastrel](https://codeberg.org/andywingo/wastrel), I can describe a bounded range of WebAssembly memory via a `memory` struct, and an access to that memory in another struct:

```
struct memory { uintptr_t base; uint64_t size; };
struct access { uint32_t addr; uint32_t len; };

```

And then if I want a writable pointer to that memory, I can do so:

```
#define static_inline \
  static inline __attribute__((always_inline))

static_inline void* write_ptr(struct memory m, struct access a) {
  BOUNDS_CHECK(m, a);
  char *base = __builtin_assume_aligned((char *) m.base_addr, 4096);
  return (void *) (base + a.addr);
}

```

(Wastrel usually omits any code for `BOUNDS_CHECK`, and just relies on memory being mapped into a `PROT_NONE` region of an appropriate size. We use a macro there because if the bounds check fails and kills the process, it’s nice to be able to use `__FILE__` and `__LINE__`.)

Regardless of whether explicit bounds checks are enabled, the `static_inline` attribute ensures that the abstraction cost is entirely burned away; and in the case where bounds checks are elided, we don’t need the `size` of the memory or the `len` of the access, so they won’t be allocated at all.

If `write_ptr` wasn’t `static_inline`, I would be a little worried that somewhere one of these `struct` values would get passed through memory. This is mostly a concern with functions that return structs by value; whereas in e.g. AArch64, returning a `struct memory` would use the same registers that a call to `void (*)(struct memory)` would use for the argument, the SYS-V x64 ABI only allocates two general-purpose registers to be used for return values. I would mostly prefer to not think about this flavor of bottleneck, and that is what static inline functions do for me.

### avoid implicit integer conversions

C has an odd set of default integer conversions, for example promoting `uint8_t` to `signed int`, and also has weird boundary conditions for signed integers. When generating C, we should probably sidestep these rules and instead be explicit: define static inline `u8_to_u32`, `s16_to_s32`, etc conversion functions, and turn on `-Wconversion`.

Using static inline cast functions also allows the generated code to assert that operands are of a particular type. Ideally, you end up in a situation where all casts are in your helper functions, and no cast is in generated code.

### wrap raw pointers and integers with intent

[Whippet](https://github.com/wingo/whippet) is a garbage collector written in C. A garbage collector cuts across all data abstractions: objects are sometimes viewed as absolute addresses, or ranges in a paged space, or offsets from the beginning of an aligned region, and so on. If you represent all of these concepts with `size_t` or `uintptr_t` or whatever, you’re going to have a bad time. So Whippet has [`struct gc_ref`](https://github.com/wingo/whippet/blob/main/api/gc-ref.h#L9-L11), [`struct gc_edge`](https://github.com/wingo/whippet/blob/main/api/gc-edge.h#L6-L8), and the like: single-member structs whose purpose it is to avoid confusion by partitioning sets of applicable operations. A `gc_edge_address` call will never apply to a `struct gc_ref`, and so on for other types and operations.

This is a great pattern for hand-written code, but it’s particularly powerful for compilers: you will often end up compiling a term of a known type or kind and you would like to avoid mistakes in the residualized C.

For example, when compiling WebAssembly, consider [`struct.set`‘s
operational
semantics](https://webassembly.github.io/spec/core/exec/instructions.html#xref-syntax-instructions-syntax-instr-struct-mathsf-struct-set-x-i): the textual rendering states, “Assert: Due to validation, *val* is some `ref.struct structaddr`.” Wouldn’t it be nice if this assertion could translate to C? Well in this case it can: with single-inheritance subtyping (as WebAssembly has), you can make a forest of pointer subtypes:

```
typedef struct anyref { uintptr_t value; } anyref;
typedef struct eqref { anyref p; } eqref;
typedef struct i31ref { eqref p; } i31ref;
typedef struct arrayref { eqref p; } arrayref;
typedef struct structref { eqref p; } structref;

```

So for a `(type $type_0 (struct (mut f64)))`, I might generate:

```
typedef struct type_0ref { structref p; } type_0ref;

```

Then if I generate a field setter for `$type_0`, I make it take a `type_0ref`:

```
static inline void
type_0_set_field_0(type_0ref obj, double val) {
  ...
}

```

In this way the types carry through from source to target language. There is a similar type forest for the actual object representations:

```
typedef struct wasm_any { uintptr_t type_tag; } wasm_any;
typedef struct wasm_struct { wasm_any p; } wasm_struct;
typedef struct type_0 { wasm_struct p; double field_0; } type_0;
...

```

And we generate little cast routines to go back and forth between `type_0ref` and `type_0*` as needed. There is no overhead because all routines are static inline, and we get pointer subtyping for free: if a `struct.set $type_0 0` instruction is passed a subtype of `$type_0`, the compiler can generate an upcast that type-checks.

### fear not `memcpy`

In WebAssembly, accesses to linear memory are not necessarily aligned, so we can’t just cast an address to (say) `int32_t*` and dereference. Instead we `memcpy(&i32, addr, sizeof(int32_t))`, and trust the compiler to just emit an unaligned load if it can (and it can). No need for more words here!

### for ABI and tail calls, perform manual register allocation

So, [GCC finally has
`__attribute__((musttail))`](https://gcc.gnu.org/onlinedocs/gcc/Statement-Attributes.html#index-musttail-statement-attribute): praise be. However, when compiling WebAssembly, it could be that you end up compiling a function with, like 30 arguments, or 30 return values; I don’t trust a C compiler to reliably shuffle between different stack argument needs at tail calls to or from such a function. It could even refuse to compile a file if it can’t meet its `musttail` obligations; not a good characteristic for a target language.

Really you would like it if all function parameters were allocated to registers. You can ensure this is the case if, say, you only pass the first *n* values in registers, and then pass the rest in global variables. You don’t need to pass them on a stack, because you can make the callee load them back to locals as part of the prologue.

What’s fun about this is that it also neatly enables multiple return values when compiling to C: simply go through the set of function types used in your program, allocate enough global variables of the right types to store all return values, and make a function epilogue store any “excess” return values—those beyond the first return value, if any—in global variables, and have callers reload those values right after calls.

### what’s not to like

Generating C is a local optimum: you get the industrial-strength instruction selection and register allocation of GCC or Clang, you don’t have to implement many peephole-style optimizations, and you get to link to to possibly-inlinable C runtime routines. It’s hard to improve over this design point in a marginal way.

There are drawbacks, of course. As a Schemer, my largest source of annoyance is that I don’t have control of the stack: I don’t know how much stack a given function will need, nor can I extend the stack of my program in any reasonable way. I can’t iterate the stack to precisely enumerate embedded pointers ( [but perhaps that’s
fine](https://wingolog.org/archives/2024/09/07/conservative-gc-can-be-faster-than-precise-gc)). I certainly can’t slice a stack to capture a delimited continuation.

The other major irritation is about side tables: one would like to be able to implement so-called [zero-cost
exceptions](https://devblogs.microsoft.com/oldnewthing/20220228-00/?p=106296), but without support from the compiler and toolchain, it’s impossible.

And finally, source-level debugging is gnarly. You would like to be able to embed DWARF information corresponding to the code you residualize; I don’t know how to do that when generating C.

(Why not Rust, you ask? Of course you are asking that. For what it is worth, I have found that lifetimes are a frontend issue; if I had a source language with explicit lifetimes, I would consider producing Rust, as I could machine-check that the output has the same guarantees as the input. Likewise if I were using a Rust standard library. But if you are compiling *from* a language without fancy lifetimes, I don’t know what you would get from Rust: fewer implicit conversions, yes, but less mature tail call support, longer compile times... it’s a wash, I think.)

Oh well. Nothing is perfect, and it’s best to go into things with your eyes wide open. If you got down to here, I hope these notes help you in your generations. For me, once my generated C type-checked, it worked: very little debugging has been necessary. Hacking is not always like this, but I’ll take it when it comes. Until next time, happy hacking!

## related articles

- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [v8's precise field-logging remembered set](/archives/2024/01/05/v8s-precise-field-logging-remembered-set)
- [parallel futures in mobile application development](/archives/2023/06/15/parallel-futures-in-mobile-application-development)
- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)
- [on "binary security of webassembly"](/archives/2020/10/15/on-binary-security-of-webassembly)

### 2 responses

1. [Andrew C](https://aconz2.github.io) says:[9 February 2026 5:09 PM](#f3148a114a9014a166a4ac5ece93fe9a87355de9)

   You would like to be able to embed DWARF information corresponding to the code you residualize

   Does #line do what you want https://gcc.gnu.org/onlinedocs/cpp/Line-Control.html ?

2. Manny Corpus says:[10 February 2026 6:53 AM](#dcdcde200b11b4a0a7a9e6627012f2ee50878a9b)

   Does #line do what you want

   No, if course not.

Comments are closed.
