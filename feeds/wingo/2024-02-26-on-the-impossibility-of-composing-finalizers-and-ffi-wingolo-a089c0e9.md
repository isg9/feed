---
title: on the impossibility of composing finalizers and ffi — wingolog
url: https://wingolog.org/archives/2024/02/26/on-the-impossibility-of-composing-finalizers-and-ffi
published: "2024-02-26T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/02/26/on-the-impossibility-of-composing-finalizers-and-ffi
---

# on the impossibility of composing finalizers and ffi — wingolog

## [on the impossibility of composing finalizers and ffi](/archives/2024/02/26/on-the-impossibility-of-composing-finalizers-and-ffi)

26 February 2024 10:05 AM

- [gc](/tags/gc)
- [finalizers](/tags/finalizers)
- [boehm](/tags/boehm)
- [ffi](/tags/ffi)
- [guile](/tags/guile)
- [luajit](/tags/luajit)
- [harfbuzz](/tags/harfbuzz)

While poking the other day at making a Guile binding for [Harfbuzz](https://harfbuzz.org/), I remembered why I don’t much do this any more: it is impossible to compose GC with explicit ownership.

Allow me to illustrate with an example. Harfbuzz has a concept of *blobs*, which are refcounted sequences of bytes. It uses these in a number of places, for example when loading OpenType fonts. You can get a peek at the blob’s contents back with [`hb_blob_get_data`](https://harfbuzz.github.io/harfbuzz-hb-blob.html#hb-blob-get-data), which gives you a pointer and a length.

Say you are in LuaJIT. (To think that for a couple years, I wrote LuaJIT all day long; now I can hardly remember.) You get a blob from somewhere and want to get its data. You define a wrapper for `hb_blob_get_data`:

```
local hb = ffi.load("harfbuzz")
ffi.cdef [[
typedef struct hb_blob_t hb_blob_t;

const char *
hb_blob_get_data (hb_blob_t *blob, unsigned int *length);
]]

```

Presumably you then arrange to release LuaJIT’s reference on the blob when GC collects a Lua wrapper for a blob:

```
ffi.cdef [[
void hb_blob_destroy (hb_blob_t *blob);
]]

function adopt_blob(ptr)
  return ffi.gc(ptr, hb.hb_blob_destroy)
end

```

OK, so let’s say we get a blob from somewhere, and want to copy out its contents as a byte string.

```
function blob_contents(blob)
   local len_out = ffi.new('unsigned int')
   local contents = hb.hb_blob_get_data(blob, len_out)
   local len = len_out[0];
   return ffi.string(contents, len)
end

```

The thing is, this code is as correct as you can get it, but it’s not correct enough. In between the call to `hb_blob_get_data` and, well, anything else, GC could run, and if `blob` is not used in the future of the program execution (the continuation), then it could be collected, causing the `hb_blob_destroy` finalizer to release the last reference on the blob, freeing `contents`: we would then be accessing invalid memory.

Among GC implementors, it is a truth universally acknowledged that a program containing finalizers must be in want of a segfault. The semantics of LuaJIT do not prescribe when GC can happen and what values will be live, so the GC and the compiler are not constrained to extend the liveness of `blob` to, say, the entirety of its lexical scope. It is perfectly valid to collect `blob` after its last use, and so at some point a GC will evolve to do just that.

I chose LuaJIT not to pick on it, but rather because its FFI is very straightforward. All other languages with GC that I am aware of have this same issue. There are but two work-arounds, and neither are satisfactory: either develop a deep and correct knowledge of what the compiler and run-time will do for a given piece of code, and then pray that knowledge does not go out of date, or attempt to manually extend the lifetime of a finalizable object, and then pray the compiler and GC don’t learn new tricks to invalidate your trick.

This latter strategy takes the form of [“remember-this” procedures that
are designed to outsmart the
compiler](https://github.com/ivmai/bdwgc/blob/master/include/gc/gc.h#L1469-L1491). They have mostly worked for the last few decades, but I wouldn’t bet on them in the future.

Another way to look at the problem is that once you have a system working—though, how would you know it’s correct?—then you either never update the compiler and run-time, or you become fast friends with whoever maintains your GC, and probably your compiler too.

For more on this topic, as always Hans Boehm has the first and last word; see for example the 2002 [*Destructors, finalizers, and*
*synchronization*](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=ac6669e60ef9ae9adbf3e596377d731584c27049). These considerations don’t really apply to *destructors*, which are used in languages with ownership and generally run synchronously.

Happy hacking, and be safe out there!

## related articles

- [unexpected concurrency](/archives/2012/02/16/unexpected-concurrency)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)
- [finalizers, guardians, phantom references, et cetera](/archives/2024/07/22/finalizers-guardians-phantom-references-et-cetera)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [whippet hacklog: adding freelists to the no-freelist space](/archives/2025/08/07/whippet-hacklog-adding-freelists-to-the-no-freelist-space)

### 9 responses

1. Ken Micklas says:[26 February 2024 11:58 AM](#95a7c57353be4024b95a29fa82babbc927e21829)

   Surely the language semantics can just specify a definite construct for lifetime extension? If it’s in the spec, no need to assume the absence of “tricks”.

2. [Owen Taylor](https://blog.fishsoup.net) says:[26 February 2024 6:44 PM](#4c98ee4ea6ab32f309b7ba80c698938b20df2fb9)

   More than anything, I think this exposes the weakness of an native FFI interface with raw pointers. In the LuaJIT example, the runtime has no idea of the relationship between “blob” and “contents”, so the GC can run wild and free “blob” first.

   By contrast: in gobject-introspection, the return here would be tagged (transfer none) - and a copy would be made before it’s ever exposed to the GC’ed language.

   If you don’t want that copy to be made - if you want to map hb *blob* t to a language-native buffer interface in some way - then I think the binding has to be hand-crafted in a language with explicit, GC visible references.

   Maybe in *this* case, you could make hf *blob* t a single chunk of memory without a finalizer, so that the GC knows that contents is a pointer inside the block of memory pointed to by blob, but that’s in no way general.

3. J says:[27 February 2024 6:07 AM](#e83e600a727a3b34ff783e95496690b1c231fa04)

   In the LuaJIT example, blob is reachable throughout the function by the blob argument, so it can not be collected until the function returns. ffi.string copies the data. What is the issue?

4. J says:[27 February 2024 6:36 AM](#5184a185fa1c86d37da9a2701676cd5ee9104f8c)

   https://luajit.org/ext *ffi* semantics.html#gc Here blob is “on the Lua stack”, as there is no difference between a function argument and a local.

5. [Alex Dowad](https://alexdowad.github.io) says:[27 February 2024 11:21 AM](#f8e16c42ba2a098570663f6b38daeb4ae6debf0d)

   Andy, you’ve made some nice points here. If I can add just a bit more:

   You wrote that “there are but two workarounds”, but I think you’ve missed what is often the only real workaround:

   Your wrapper objects need a ‘valid?’ flag, and you have to arrange to mark all of the right objects as invalid when one is freed. In your HarfBuzz example, when a font object is freed, all blobs which were embedded in the font must be invalidated as well. So you need to do some bookkeeping behind the scenes.

   (Your bookkeeping code may need to make careful use of weak references to avoid holding all the HarfBuzz objects in memory forever.)

6. [Alex Dowad](https://alexdowad.github.io) says:[27 February 2024 11:29 AM](#1ecb59b1a894af483967d45a3a65debb8f78f9f5)

   Sorry! Clicked the submit button a bit too fast there!

   You also need to make sure that your font wrapper is not freed before your blob wrapper. Right.

   I think the general solution is to give the font wrapper object a ‘free’ method, and ensure you call it in the right places. In other words, you can’t rely on GC finalizers to free the external resource.

   What sucks about this is that if your font object is encapsulated in some other object, *that* object also needs a ‘free’ method, and so on (recursively). In the worst case, you can end up managing the lifetimes of almost all your objects manually, à la C. The slap in the face is that your GC must duplicate the same work to ensure that managed heap memory is also freed!

   This does solve the problem reliably, though, and you don’t have to worry about “the compiler and GC learning a new trick”. You do have to worry about getting the calls to ‘free’ wrong.

7. [Coral](https://correlation.zone/) says:[27 February 2024 12:48 PM](#b4ab5649678fdec3400a49a6d868a2d8a044325e)

   For a safe FFI interface over raw pointers (capabilities...) CHERI (and Cornucopia) offers enough architectural capability to give untrusted unmanaged code a raw pointer without being able to read through it after free; one can hope a enterprising cloud provider ships it!

8. int19h says:[29 February 2024 10:40 AM](#e2fd3b9a27af05c4d90958c5cc881a6dcc21f2eb)

   In the .NET world, we have GCHandle, which is simply a wrapper around a reference to a GC-managed object that has explicit lifetime semantics (i.e. you need to explicitly destroy it for GC to consider it to not be a root anymore).

9. [Dj](https://djdev.me) says:[1 March 2024 3:19 PM](#7f2ccb33f0a34c82157a8cf126e79f6d9b562dc6)

   Very interesting read. Recently, I’ve been working with FFI a lot recently with libffi and also the native interface Deno runtime exposes and can confirm finalization of native objects is a difficult problem indeed. But it is possible to solve, in my opinion.

   Here’s what I think about it:

   For one, FFI is inherently unsafe and one needs to be well-versed in manual memory management. If you are writing to expose a low-level API using FFI to a garbage-collected language, you need to be familiar with the GC semantics as well, to expose a nicely wrapped module for consumption in that targeted language without the users having to consider manual memory management. Goal in that case is just to integrate very well.

   Garbage collectors for sure aren’t always reliable, and your object may not even ever get released. That’s why it’s important to offer something like a `free` or `release` method as well.

   Quoting this: \> attempt to manually extend the lifetime of a finalizable object, and then pray the compiler and GC don’t learn new tricks to invalidate your trick.

   I don’t think this is necessarily a workaround. This may be the correct solution. If another object (say, blob data) obtained from the original object wrapper (blob itself) depends on the lifetime of original object, we can extend the lifetime in a way that the new object holds a strong reference to the original object. So even if you lose a reference to the original object, and the new object is in use, it will stay alive at least as long as that new object. There are nicer ways to do something like this in JavaScript, we have `WeakMap` s! The key is the garbage-collectable (blob contents) and the value (blob, which the contents depend on) is the one being held for at least as long as the lifetime of the key.

   Also, I do not see a problem with the `blob_contents` function. If it was a zero-copy string, it would make sense that we are returning an object that depends on another object but no strong reference is held, which is incorrect. We should explicitly extend the lifetime as mentioned above. But `ffi.string` copies the string so there is no problem there. The contents pointer is obtained from the blob object and immediately used, how would it get collected in between? Unless you are passing around the blob pointer instead of the wrapper in user-land, which is like inviting for all sorts of memory related problems.

   I’d like to mention an interesting problem we faced using Deno FFI related to garbage collection. So Deno FFI allows you to pass Uint8Arrays (as `uint8_t *`) with zero-copy, essentially the underlying pointer. But then the responsibility is on us to not use that pointer after the array is collected.

   What we’d done in a user-land module (to expose SQLite3 bindings, very high performance thanks to v8 fast call api and turbocall JIT in Deno FFI implementation!) is, when passing a string, we first encode it to an Uint8Array and then pass it to FFI. So an intermediate buffer is created, of which reference is not really held anywhere. Now there are two ways to pass buffers to FFI in Deno. One is to set the receiver type to “buffer” where you can pass Uint8Arrays (fast path) or other typed arrays (non-fast), but if you use “pointer” type, you have to first explicitly convert the typed array to pointer using `Deno.UnsafePointer.of`.

   The first way to pass it, is the best one. V8 would pass the underlying pointer to the native API directly for us while also keeping the buffer alive for at least until the FFI call returns. But in the other approach, the buffer could be collected as it was not even stored in a local variable but simply inlined like this: ffi *function(..., Deno.UnsafePointer.of(encodeCString(str)), ...). I discovered that even this can crash if the code runs repeatedly, when doing benchmarks for example! We simply started doing ffi* function(..., encodeCString(str), ...) and the crash was fixed.

Comments are closed.
