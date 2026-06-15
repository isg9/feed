---
title: malloc as a service — wingolog
url: https://wingolog.org/archives/2020/10/13/malloc-as-a-service
published: "2020-10-13T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2020/10/13/malloc-as-a-service
---

# malloc as a service — wingolog

## [malloc as a service](/archives/2020/10/13/malloc-as-a-service)

13 October 2020 1:34 PM

- [webassembly](/tags/webassembly)
- [compilers](/tags/compilers)
- [malloc](/tags/malloc)
- [gc](/tags/gc)
- [llvm](/tags/llvm)
- [emscripten](/tags/emscripten)
- [igalia](/tags/igalia)
- [js](/tags/js)

Greetings, internet! Today I have the silliest of demos for you: malloc-as-a-service.

**loading walloc...**



The above input box, if things managed to work, loads up a simple bare-bones malloc implementation, and exposes "malloc" and "free" bindings. But the neat thing is that it's built without emscripten: it's a standalone C file that compiles directly to WebAssembly, with no JavaScript run-time at all. I share it here because it might come in handy to people working on WebAssembly toolchains, and also because it was an amusing experience to build.

**wat?**

The name of the allocator is "walloc", in which the w is for WebAssembly.

Walloc was designed with the following priorities, in order:

1. Standalone. No stdlib needed; no emscripten. Can be included in a project without pulling in anything else.

2. Reasonable allocation speed and fragmentation/overhead.

3. Small size, to minimize download time.

4. Standard interface: a drop-in replacement for malloc.

5. Single-threaded (currently, anyway).

Emscripten includes a couple of good malloc implementations ( [dlmalloc](https://github.com/emscripten-core/emscripten/blob/master/system/lib/dlmalloc.c) and [emmalloc](https://github.com/emscripten-core/emscripten/blob/master/system/lib/emmalloc.cpp)) which probably you should use instead. But if you are really looking for a bare-bones malloc, walloc is fine.

You can check out all the details at the [walloc project page](https://github.com/wingo/walloc); a selection of salient bits are below.

Firstly, to build walloc, it's just a straight-up compile:

```
clang -DNDEBUG -Oz --target=wasm32 -nostdlib -c -o walloc.o walloc.c

```

The resulting `walloc.o` is a conforming WebAssembly file on its own, but which also contains [additional symbol table and relocation sections](https://github.com/WebAssembly/tool-conventions/blob/master/Linking.md) which allow `wasm-ld` to combine separate compilation units into a single final WebAssembly file. `walloc.c` on its own doesn't import or export anything, in the WebAssembly sense; to make bindings visible to JS, you need to add a little wrapper:

```
typedef __SIZE_TYPE__ size_t;

#define WASM_EXPORT(name) \
  __attribute__((export_name(#name))) \
  name

// Declare these as coming from walloc.c.
void *malloc(size_t size);
void free(void *p);

void* WASM_EXPORT(walloc)(size_t size) {
  return malloc(size);
}

void WASM_EXPORT(wfree)(void* ptr) {
  free(ptr);
}

```

If you compile that to `exports.o` and link via `wasm-ld --no-entry --import-memory -o walloc.wasm exports.o walloc.o`, you end up with the `walloc.wasm` used in the demo above. See your inspector for the URL.

The resulting wasm file is about 2 kB (uncompressed).

Walloc isn't the smallest allocator out there. A simple bump-pointer allocator that never frees is the fastest thing you can have. There is also an alternate allocator for Rust, [wee\_alloc](https://github.com/rustwasm/wee_alloc), which is said to be smaller than walloc, though I think it is less space-efficient for small objects. But still, walloc is pretty small.

**implementation notes**

When a C program is compiled to WebAssembly, the resulting wasm module (usually) has associated linear memory. It can be linked in a way that the memory is created by the module when it's instantiated, or such that the module is given a memory by its host. The above example passed `--import-memory` to the linker, allowing the host to bound memory usage for the module instance.

The linear memory has the usual data, stack, and heap segments. The data and stack are placed first. The heap starts at the `&__heap_base` symbol. (This symbol is computed and defined by the linker.) All bytes above `&__heap_base` can be used by the wasm program as it likes. So `&__heap_base` is the lower bound of memory managed by walloc.

```
                                              memory growth ->
+----------------+-----------+-------------+-------------+----
| data and stack | alignment | walloc page | walloc page | ...
+----------------+-----------+-------------+-------------+----
^ 0              ^ &__heap_base            ^ 64 kB aligned

```

Interestingly, there are a few different orderings of data and stack used by different toolchains. It used to even be the case that the stack grew up. This diagram from the recent ["Everything Old is New Again: Binary Security of WebAssembly" paper by Lehmann et al](https://www.usenix.org/conference/usenixsecurity20/presentation/lehmann) is a good summary:

![](https://wingolog.org/pub/stack-data-heap-sec20-lehmann.png)

The sensible thing to prevent accidental overflow (underflow, really) is to have the stack grow down to 0, with data at higher addresses. But this can cause WebAssembly code that references data to take up more bytes, because addresses are written using variable-length ["LEB"](https://en.wikipedia.org/wiki/LEB128) encodings that favor short offsets, so it isn't the default, right now at least.

Anyway! The upper bound of memory managed by walloc is the total size of the memory, which is aligned on 64-kilobyte boundaries. (WebAssembly ensures this alignment.) Walloc manages memory in 64-kb pages as well. It starts with whatever memory is initially given to the module, and will expand the memory if it runs out. The host can specify a maximum memory size, in pages; if no more pages are available, walloc's `malloc` will simply return `NULL`; handling out-of-memory is up to the caller.

Walloc has two allocation strategies: small and large objects.

**big bois**

A large object is more than 256 bytes.

There is a global freelist of available large objects, each of which has a header indicating its size. When allocating, walloc does a best-fit search through that list.

```
struct large_object {
  struct large_object *next;
  size_t size;
  char payload[0];
};
struct large_object* large_object_free_list;

```

Large object allocations are rounded up to 256-byte boundaries, including the header.

If there is no object on the freelist that can satisfy an allocation, walloc will expand the heap by the size of the allocation, or by half of the current walloc heap size, whichever is larger. The resulting page or pages form a large object that can satisfy the allocation.

If the best object on the freelist has more than a chunk of space on the end, it is split, and the tail put back on the freelist. A chunk is 256 bytes.

```
+-------------+---------+---------+-----+-----------+
| page header | chunk 1 | chunk 2 | ... | chunk 255 |
+-------------+---------+---------+-----+-----------+
^ +0          ^ +256    ^ +512                      ^ +64 kB

```

As each page is 65536 bytes, and each chunk is 256 bytes, there are therefore 256 chunks in a page. The first chunk in a page that begins an allocated object, large or small, contains a header chunk. The page header has a byte for each of the 256 chunks in the page. The byte is 255 if the corresponding chunk starts a large object; otherwise the byte indicates the size class for packed small-object allocations (see below).

```
+-------------+---------+---------+----------+-----------+
| page header | large object 1    | large object 2 ...   |
+-------------+---------+---------+----------+-----------+
^ +0          ^ +256    ^ +512                           ^ +64 kB

```

When splitting large objects, we avoid starting a new large object on a page header chunk. A large object can only span where a page header chunk would be if it includes the entire page.

Freeing a large object pushes it on the global freelist. We know a pointer is a large object by looking at the page header. We know the size of the allocation, because the large object header precedes the allocation. When the next large object allocation happens after a free, the freelist will be compacted by merging adjacent large objects.

**small fry**

Small objects are allocated from segregated freelists. The granule size is 8 bytes. Small object allocations are packed in a chunk of uniform allocation size. There are size classes for allocations of each size from 1 to 6 granules, then 8, 10, 16, and 32 granules; 10 sizes in all. For example, an allocation of e.g. 12 granules will be satisfied from a 16-granule chunk. Each size class has its own free list.

```
struct small_object_freelist {
  struct small_object_freelist *next;
};
struct small_object_freelist small_object_freelists[10];

```

When allocating, if there is nothing on the corresponding freelist, walloc will allocate a new large object, then change its chunk kind in the page header to the size class. It then goes through the fresh chunk, threading the objects through each other onto a free list.

```
+-------------+---------+---------+------------+---------------------+
| page header | large object 1    | granules=4 | large object 2' ... |
+-------------+---------+---------+------------+---------------------+
^ +0          ^ +256    ^ +512    ^ +768       + +1024               ^ +64 kB

```

In this example, we imagine that the 4-granules freelist was empty, and that the large object freelist contained only large object 2, running all the way to the end of the page. We allocated a new 4-granules chunk, splitting the first chunk off the large object, and pushing the newly trimmed large object back onto the large object freelist, updating the page header appropriately. We then thread the 4-granules (32-byte) allocations in the fresh chunk together (the chunk has room for 8 of them), treating them as if they were instances of `struct freelist`, pushing them onto the global freelist for 4-granules allocations.

```
           in fresh chunk, next link for object N points to object N+1
                                 /--------\
                                 |        |
            +------------------+-^--------v-----+----------+
granules=4: | (padding, maybe) | object 0 | ... | object 7 |
            +------------------+----------+-----+----------+
                               ^ 4-granule freelist now points here

```

The size classes were chosen so that any wasted space (padding) is less than the size class.

Freeing a small object pushes it back on its size class's free list. Given a pointer, we know its size class by looking in the chunk kind in the page header.

**and that's it**

Hey have fun with the thing! Let me know if you find it useful. Happy hacking and until next time!

## related articles

- [on "binary security of webassembly"](/archives/2020/10/15/on-binary-security-of-webassembly)
- [accessing webassembly reference-typed arrays from c++](/archives/2022/08/23/accessing-webassembly-reference-typed-arrays-from-c)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)

Comments are closed.
