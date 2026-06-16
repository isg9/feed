---
title: The Alkyne GC
url: /2022/06/07/alkyne-gc
published: "2022-06-07T00:00:00Z"
feed: mcyoung
guid: /2022/06/07/alkyne-gc
---

# The Alkyne GC

[Alkyne](https://github.com/mcy/alkyne) is a scripting language I built a couple of years ago for generating configuration blobs. Its interpreter is a naive AST walker[1](#fn:1) that uses ARC[2](#fn:2) for memory management, so it’s pretty slow, and I’ve been gradually writing a [new evaluation engine](https://github.com/mcy/alkyne/tree/new-engine) for it.

This post isn’t about Alkyne itself, that’s for another day. For now, I’d like to write down some notes for the GC I wrote[3](#fn:3) for it, and more generally provide an introduction to memory allocators (especially those that would want to collude with a GC).

This post is intended for people familiar with the basics of low-level programming, such as pointers and syscalls. Alkyne’s GC is intended to be simple while still having reasonable performance. This means that the design contains all the allocator “tropes,” but none of the hairy stuff.

My hope is readers new to allocators or GCs will come away with an understanding of these tropes and the roles they play in a modern allocator.

> Thank you to James Farrell, Manish Goregaokar, Matt Kulukundis, JeanHeyd Meneide, Skye Thompson, and Paul Wankadia for providing feedback on various drafts of this article. This was a tough one to get right. :)

## [Trailhead](\#trailhead)

The Alkyne GC is solving a very specific problem, which allows us to limit what it actually needs to do. Alkyne is an “embeddable” language like JavaScript, so its heap is not intended to be big; in fact, for the benefit of memory usage optimizations, it’s ideal to use 32-bit pointers (a 4 gigabyte address space).

The heap needs to be able to manage arbitrarily-large allocations (for lists), and allocations as small as eight bytes (for floats[4](#fn:4)). Allocation should be reasonably quick, but due to the size of the heap, walking the entire heap is totally acceptable.

Because we’re managing a fixed-size heap, we can simply ask the operating system for a contiguous block of that size up-front using the `mmap()` syscall. An Alkyne pointer is simply a 32-bit offset into this giant allocation, which can be converted to and from a genuine CPU pointer by adding or subtracting the base address of the heap.

```text highlight

 4GB Heap
 +-------------------------------------------------+
 |                x                                |
 +-------------------------------------------------+
 ^                ^
 base             base + ptr_to_x

```

[Plaintext](#code:1)

The OS won’t actually reserve 4GB of memory for us; it will only allocate one system page (4KB) at a time. If we read or write to a particular page in the heap for the first time, the OS will only then find physical RAM to back it[5](#fn:5).

Throughout, we’ll be working with this fixed-size heap, and won’t think too hard about where it came from. For our purposes, it is essentially a `Box<[u8]>`, but we’ll call it a `Heap<[u8]>` to make it clear this memory we got from the operating system (but, to be clear, the entire discussion applies just as well to an ordinary gigantic `Box<[u8]>`)

The Alkyne language does not have threads, so we can eschew concurrency. This significantly reduces the problems we will need to solve. Most modern allocators and garbage collectors are violently concurrent by nature, and unfortunately, much too advanced for one article. There are links below to fancier GCs you can poke around in.

## [A Heap of Trouble](\#a-heap-of-trouble)

To build a garbage collector, we first need an allocator. We could “just”[6](#fn:6) use the system heap as a source of pages, but most garbage collectors collude with the allocator, since they will want to use similar data structures. Thus, if we are building a garbage collector, we might as well build the allocator too.

An allocator, or “memory heap” (not to be confused with a min-heap, an unrelated but *wicked* data structure), services requests for *allocations*: unique leases of space in the managed heap of various sizes, which last for lifetimes not known until runtime. These allocations may also be called *objects*, and a heap may be viewed as a general-purpose object pool.

The most common API for a heap is:

```rust highlight
trait Allocator {
  // Returns a *unique pointer* managed by this allocator
  // to memory as large as requested, and as aligned
  // as we'd like.
  //
  // Returns null on failure.
  unsafe fn alloc(&mut self, size: usize, align: usize) -> *mut u8;
  // Frees a pointer returned by `Alloc` may be called at
  // most once.
  unsafe fn free(&mut self, ptr: *mut u8);
}

```

[Rust](#code:2)

> [note](#note:1)
>
> Originally the examples were in C++, which I feel is more accessible (lol) but given that Alkyne itself is written in Rust I felt that would make the story flow better.

This is the “malloc” API, which is actually very deficient; ideally, we would do something like Rust’s [`Allocator`](https://doc.rust-lang.org/std/alloc/trait.Allocator.html), which requires providing size *and* alignment to both the allocation *and* deallocation functions.

Unfortunately[7](#fn:7), this means I need to explain alignment.

### [Good Pointers, Evil Pointers, Lawful Pointers, Chaotic Pointers](\#good-pointers-evil-pointers-lawful-pointers-chaotic-pointers)

“Alignment” is a somewhat annoying property of a pointer. A pointer is aligned to N bytes (always a power of 2) if its address is divisible by N. A pointer is “well-aligned” (or just “aligned”) if its address is aligned to the natural alignment of the thing it points to. For ints, this is *usually* their size; for structs, it is the maximum alignment among the alignments of the fields of that struct.

Performing operations on a pointer requires that it be aligned[8](#fn:8). This is annoying because it requires some math. Specifically we need three functions:

```rust highlight
/// Checks that `ptr` is aligned to an alignment.
fn is_aligned(ptr: Int, align: usize) -> bool {
  ptr & (align - 1) == 0
}

/// Rounds `ptr` down to a multiple of `align`.
fn align_down(ptr: Int, align: usize) -> Int {
  ptr & !(align - 1)
}

/// Rounds `ptr` up to a multiple of `align`.
fn align_up(ptr: Int, align: usize) -> Int {
  // (I always look this one up. >_>)
  align_down(ptr + align - 1, align)
}

/// Computes how much needs to be added to `ptr` to align it.
fn misalign(ptr: Int, align: usize) -> usize {
  align_up(ptr, align) - ptr
}

```

[Rust](#code:3)

(Exercise: prove these formulas.)

For the rest of the article I will assume I have these three functions available at any time for whatever type of integer I’d like (including raw pointers which are just boutique[9](#fn:9) integers).

Also we will treat the `Heap<[u8]>` holding our entire heap as being infinitely aligned; i.e. as a pointer it is aligned to all possible alignments that could matter (i.e. page-aligned, 4KB as always). (For an ordinary `Box<[u8]>`, this is not true.)

### [The Trivial Heap](\#the-trivial-heap)

Allocating memory is actually very easy. *Arenas* are the leanest and meanest in the allocator food chain; they simply don’t bother to free any memory.

This means allocation is just incrementing a cursor indicating where the hitherto-unallocated memory is.

```text highlight

 +-------------------------------------------------+
 | Allocated        | Free                         |
 +-------------------------------------------------+
                    ^
                    cursor

```

[Plaintext](#code:4)

Our allocator is as simple as `return ptr++;`.

This is straightforward to implement in code:

```rust highlight
struct Arena {
  heap: Heap<[u8]>,
  cursor: usize,
}

impl Allocator for Arena {
  unsafe fn alloc(&mut self, size: usize, align: usize) -> *mut u8 {
    // To get an aligned pointer, we need to burn some "alignment
    // padding". This is one of the places where alignment is
    // annoying.
    let needed = size + misalign(self.heap.as_ptr(), align);

    // Check that we're not out of memory.
    if self.heap.len() - self.cursor < needed {
      return ptr::null_mut();
    }

    // Advance the cursor and cut off the end of the allocated
    // section.
    self.cursor += needed;
    &mut self.heap[self.cursor - size] as *mut u8;
  }

  unsafe fn free(&mut self, ptr: *mut u8) {
    // ayy lmao
  }
}

```

[Rust](#code:5)

Arenas are very simple, but far from useless! They’re great for holding onto data that exists for the context of a “session”, such as for software that does lots of computations and then exits (a compiler) or software that handles requests from clients, where lots of data lives for the duration of the request and no longer (a webserver).

They are not, however, good for long-running systems. Eventually the heap will be exhausted if objects are not recycled.

Making this work turns out to be hard\[citation-needed\]. This is the “fundamental theorem” of allocators:

> [theorem](#thm:1)
>
> Handing out memory is easy. Handing it out **repeatedly** is hard.

Thankfully, over the last fifty years we’ve mostly figured this out. Allocator designs can get pretty gnarly.

## [Four Tropes](\#four-tropes)

From here, we will gradually augment our allocator with more features to allow it to service all kinds of requests. For this, we will implement four common allocator features:

1. Blocks and a block cache.
2. Free lists.
3. Block merging and splitting.
4. Slab allocation.

All four of these are present in some form in most modern allocators.

### [Blocks](\#blocks)

The first thing we should do is to deal in fixed-size blocks of memory of some minimum size. If you ask `malloc()` for a single byte, it will probably give you like 8 bytes on most systems. No one is asking `malloc()` for single bytes, so we can quietly round up and not have people care. (Also, Alkyne’s smallest heap objects are eight bytes, anyways.)

Blocks are also convenient, because we can keep per-block metadata on each one, as a header before the user’s data:

```rust highlight
#[repr(C)]
struct Block {
  header: Header,
  data: [u8; BLOCK_SIZE],
}

```

[Rust](#code:6)

To allow blocks to be re-used, we can keep a cache of recently freed blocks. The easiest way to do this is with a stack. Note that the heap is now made of `Block` s, not plain bytes.

To allocate storage, first we check the stack. If the stack is empty, we revert to being an arena and increment the cursor. To free, we push the block onto the stack, so `alloc()` can return it on the next call.

```rust highlight
struct BlockCache {
  heap: Heap<[Block]>,
  cursor: usize,
  free_stack: Vec<*mut Block>,
}

impl Allocator for BlockCache {
  unsafe fn alloc(&mut self, size: usize, align: usize) -> *mut u8 {
    // Check that the size isn't too big. We don't need to
    // bother with alignment, because every block is
    // infinitely-aligned, just like the heap itself.
    if size > BLOCK_SIZE {
      return ptr::null_mut();
    }

    // Try to serve a block from the stack.
    if let Some(block) = self.free_stack.pop() {
      return &mut *block.data as *mut u8;
    }

    // Fall back to arena mode.
    if self.cursor == self.heap.len() {
      return ptr::null_mut();
    }
    self.cursor += 1;
    &mut self.heap[self.cursor].data as *mut u8
  }

  unsafe fn free(&mut self, ptr: *mut u8) {
    // Use pointer subtraction to find the start of the block.
    let block = ptr.sub(size_of::<Header>()) as *mut Block;
    self.free_stack.push(block);
  }
}

```

[Rust](#code:7)

This allocator has a problem: it relies on the system allocator! `Heap` came directly from the OS, but `Vec` talks to `malloc()` (or something like it). It also adds some pretty big overhead: the `Vec` needs to be able to resize, since it grows as more and more things are freed. This can lead to long pauses during `free()` as the vector resizes.

Cutting out the middleman gives us more control over this overhead.

### [Free Lists](\#free-lists)

Of course, no one has ever heard of a “free stack”; everyone uses free lists! A free list is the cache idea but implemented as an *intrusive linked list*.

A linked list is this data structure:

```rust highlight
enum Node<T> {
  Nil,
  Cons(Box<(T, Node<T>)>),
  //   ^~~ oops I allocated again
}

```

[Rust](#code:8)

This has the same problem of needing to find an allocator to store the nodes. An intrusive list avoids that by making the nodes *part* of the elements. The `Header` we reserved for ourselves earlier is the perfect place to put it:

```rust highlight
struct Header {
  /// Pointer to the next and previous blocks in whatever
  /// list the block is in.
  next: *mut Block,
  prev: *mut Block,
}

```

[Rust](#code:9)

In particular we want to make sure block are in doubly-linked lists, which have the property that any element can be removed from them without walking the list.

```text highlight

   list.root
     |
     v
 +-> Block--------------------------+
 |   | next | null | data data data |
 |   +------------------------------+
 +-----/------+
      /       |
     v        |
 +-> Block--------------------------+
 |   | next | prev | data data data |
 |   +------------------------------+
 +-----/------+
      /       |
     v        |
 +-> Block--------------------------+
 |   | next | prev | data data data |
 |   +------------------------------+
 +-----/------+
      /       |
     v        |
     Block--------------------------+
     | null | prev | data data data |
     +------------------------------+

```

[Plaintext](#code:10)

We also introduce a `List` container type that holds the root node of a list of blocks, to give us a convenient container-like API. This type looks like this:

```rust highlight
struct List {
  /// The root is actually a sacrificial block that exists only to
  /// make it possible to unlink blocks in the middle of a list. This
  /// needs to exist so that calling unlink() on the "first" element
  /// of the list has a predecessor to replace itself with.
  root: *mut Block,
}

impl List {
  /// Pushes a block onto the list.
  unsafe fn push(&mut self, block: *mut Block) {
    let block = &mut *block;
    let root = &mut *self.root;
    if !root.header.next.is_null() {
      let first = &mut *root.header.next;
      block.header.next = first;
      first.header.prev = block;
    }
    root.header.next = block;
    block.header.prev = root;
  }

  /// Gets the first element of the list, if any.
  unsafe fn first(&mut self) -> Option<&mut Block> {
    let root = &mut *self.root;
    root.header.next.as_mut()
  }
}

```

[Rust](#code:11)

We should also make it possible to ask a block whether it is in any list, and if so, remove it.

```rust highlight
impl Block {
  /// Checks if this block is part of a list.
  fn is_linked(&self) -> bool {
    // Only the prev link is guaranteed to exist; next is
    // null for the last element in a list. Sacrificial
    // nodes will never have prev non-null, and can't be
    // unlinked.
    !self.header.prev.is_null()
  }

  /// Unlinks this linked block from whatever list it's in.
  fn unlink(&mut self) {
    assert!(self.is_linked());
    if !self.header.next.is_null() {
      let next = &mut *self.header.next;
      next.header.prev = self.header.prev;
    }

    // This is why we need the sacrificial node.
    let prev = &mut *self.header.prev;
    prev.header.next = self.header.next;

    self.header.prev = ptr::null_mut();
    self.header.next = ptr::null_mut();
  }
}

```

[Rust](#code:12)

Using these abstractions we can upgrade `BlockCache` to `FreeList`. We only need to rename `free_stack` to `free_list`, and make a one-line change:

```rust highlight
- if let Some(block) = self.free_stack.pop() {
+ if let Some(block) = self.free_list.first() {
+   block.unlink();
    return &mut *block.data as *mut u8;
 }

```

[Rust](#code:13)

Hooray for encapsulation!

This is a very early `malloc()` design, similar to the one described in the K&R C book. It does have big blind spot: it can only serve up blocks up to a fixed size! It’s also quite wasteful, because all allocations are served the same size blocks: the bigger we make the maximum request, the more wasteful `alloc(8)` gets.

### [Block Splitting (Alkyne’s Way)](\#block-splitting-alkynes-way)

The next step is to use a block splitting/merging scheme, such as the [*buddy system*](https://en.wikipedia.org/wiki/Buddy_memory_allocation). Alkyne does not precisely use a buddy system, but it does something similar.

Alkyne does not have fixed-size blocks. Like many allocators, it defines a “page” of memory as the atom that it keeps its data structures. Alkyne defines a page to be 4KB, but other choices are possible: TCMalloc uses 8KB pages.

In Alkyne, pages come together to form contiguous, variable-size *reams* (get it?). These take the place of blocks.

#### [Page Descriptors](\#page-descriptors)

Merging and splitting makes it hard to keep headers at the start of reams, so Alkyne puts them all in a giant array somewhere else. Each page gets its own “header” called a page descriptor, or [`Pd`](https://github.com/mcy/alkyne/blob/a62ad3b7ee70268625da640c1edeea8ff7116512/src/eval2/gc.rs#L416).

The array of page descriptors lives at the beginning of the heap, and the actual pages follow after that. It turns out that this array has a maximum size, which we can use to pre-compute where the array ends.

Currently, each `Pd` is 32 bytes, in addition to the 4KB it describes. If we divide 4GB by 32 + 4K, it comes out to around four million pages (4067203 to be precise). Rounded up to the next page boundary, this means that pages begin at the 127102nd 4K boundary after the `Heap<[u8]>` base address, or an offset of `0x7c1f400` bytes.

Having them all in a giant array is also very useful, because it means the GC can trivially find *every allocation* in the whole heap: just iterate the `Pd` array!

So! This is our heap:

```text highlight
+---------------------------------------+  <-- mmap(2)'d region base
| Pd | Pd | Pd | Pd | Pd | Pd | Pd | Pd | \
+---------------------------------------+ |--- Page Descriptors
| Pd | Pd | Pd | Pd | Pd | Pd | Pd | Pd | |    for every page we can
+---------------------------------------+ |    ever allocate.
: ...                                   : |
+---------------------------------------+ |
| Pd | Pd | Pd | Pd | Pd | Pd | Pd | Pd | /
+---------------------------------------+  <-- Heap base address
| Page 0                                | \    = region + 0x7c1f400
|                                       | |
|                                       | |--- 4K pages corresponding
+---------------------------------------+ |    to the Pds above.
| Page 1                                | |    (not to scale)
|                                       | |
|                                       | |
+---------------------------------------+ |
: ...                                   | |
+---------------------------------------+ |
| Page N                                | |
|                                       | |
|                                       | /
+---------------------------------------+
  (not to scale by a factor of about 4)

```

[Plaintext](#code:14)

Each one of those little `Pd` s looks something like this:

```rust highlight
#[repr(C)]
struct Pd {
  gc_bits: u64,
  prev: Option<u32>,
  next: Option<u32>,
  len: u16,
  class: SizeClass,
  // More fields...
}

```

[Rust](#code:15)

`prev` and `next` are the intrusive linked list pointers used for the free lists, but now they are indices into the `Pd` array. The other fields will be used for this and the trope that follows.

Given a pointer into a page, we can get the corresponding `Pd` by `align_down()`‘ing to a page boundary, computing the index of the page (relative to the heap base), and then index into the `Pd` array. This process can be reversed to convert a pointer to a `Pd` into a pointer to a page, so going between the two is very easy.

> [note](#note:2)
>
> I won’t cover this here, but Alkyne actually wraps `Pd` pointers in a special `PdRef` type that also carries a reference to the `Heap`; this allows implementing functions like `is_linked()`, `unlink()`, and `data()` directly.
>
> I won’t show how this is implemented, since it’s mostly boilerplate.

#### [Reams of Pages](\#reams-of-pages)

There is one giant free list that contains all of the reams. Reams use their first page’s descriptor to track all of their metadata, including the list pointers for the free list. The `len` field additionally tracks how many *additional* pages are in the ream. `gc_bits` is set to 1 if the page is in use and 0 otherwise.

To allocate N continuous pages from the free ream list:

1. We walk through the free ream list, and pick the first one with at least N pages.
2. We “split” it: the first N pages are returned to fulfill the request.
3. The rest of the ream is put back into the free list.
4. If no such ream exists, we allocate a max-sized ream[10](#fn:10) (65536 pages), and split that as above.

In a sense, each ream is an arena that we allocate smaller reams out of; those reams cannot be “freed” back to the ream they came from. Instead, to free a ream, we just stick it back on the main free list.

If we ever run out, we turn back into an arena and initialize the next uninitialized `Pd` in the big ol’ `Pd` array.

```rust highlight
struct ReamAlloc {
  heap: Heap<[Page]>,
  cursor: usize,
  free_list: List,
}

/// The offset to the end of the maximally-large Pd array.
/// This can be computed ahead of time.
const PAGES_START: usize = ...;

impl Allocator for ReamAlloc {
  unsafe fn alloc(&mut self, size: usize, align: usize) -> *mut u8 {
    // We don't need to bother with alignment, because every page is
    // already infinitely aligned; we only allocate at the page
    // boundary.
    let page_count = align_up(size, 4096) / 4096;

    // Find the first page in the list that's big enough.
    // (Making `List` iterable is an easy exercise.)
    for pd in &mut self.list {
      if pd.len < page_count - 1 {
        continue
      }
      if pd.len == page_count - 1 {
        // No need to split here.
        pd.unlink();
        return pd.data();
      }

      // We can chop off the *end* of the ream to avoid needing
      // to update any pointers.
      let new_ream = pd.add(page_count);
      new_ream.len = page_count - 1;
      pd.len -= page_count;

      return new_ream.data();
    }

    // Allocate a new ream. This is more of the same arena stuff.
  }
  unsafe fn free(&mut self, ptr: *mut u8) {
    // Find the Pd corresponding to this page's pointer. This
    // will always be a ream's first Pd assuming the user
    // didn't give us a bad pointer.
    let pd = Pd::from_ptr(ptr);
    self.free_list.push(pd);
  }
}

```

[Rust](#code:16)

This presents a problem: over time, reams will shrink and never grow, and eventually there will be nothing left but single pages.

Top fix this, we can merge reams (not yet implemented in Alkyne). Thus:

1. Find two adjacent, unallocated reams.
2. Unlink the second ream from the free list.
3. Increase the length of the first ream by the number of pages in the second.

```rust highlight
// `reams()` returns an iterator that walks the `Pd` array using
// the `len` fields to find the next ream each time.
for pd in self.reams() {
  if pd.gc_bits != 0 { continue }
  loop {
    let next = pd.add(pd.len + 1);
    if next.gc_bits != 0 { break }
    next.unlink();
    pd.len += next.len + 1;
  }
}

```

[Rust](#code:17)

We don’t need to do anything to the second ream’s `Pd`; by becoming part of the first ream, it is subsumed. Walking the heap requires using reams’ lengths to skip over currently-invalid `Pd` s, anyways.

We have two options for finding mergeable reams. Either we can walk the entire heap, as above, or, when a ream is freed, we can check that the previous and following reams are mergeable (finding the previous ream would require storing the length of a ream at its first *and* last `Pd`).

Which merging strategy we use depends on whether we’re implementing an ordinary `malloc`-like heap or a garbage collector; in the `malloc` case, merging on free makes more sense, but merging in one shot makes more sense for Alkyne’s GC (we’ll see why in a bit).

### [Slabs and Size Classes](\#slabs-and-size-classes)

A *slab allocator* is a specialized allocator that allocates a single type of object; they are quite popular in kernels as pools of commonly-used object. The crux of a slab allocator is that, because everything is the same size, we *don’t* need to implement splitting and merging. The `BlockCache` above is actually a very primitive slab allocator.

Our `Pd` array is also kind of like a slab allocator; instead of mixing them in with the variably-sized blocks, they all live together with no gaps in between; entire pages are dedicated just to `Pd` s.

The Alkyne page allocator cannot allocate pages smaller than 4K, and making them any smaller increases the relative overhead of a `Pd`. To cut down on book-keeping, we slab-allocate small objects by defining *size classes*.

A size class is size of smaller-than-a-page object that Alkyne will allocate; other sizes are rounded up to the next size class. Entire pages are dedicated to holding just objects of the same size; these are called small object pages, or simply *slabs*. The size class is tracked with the `class` field of the `Pd`.

Each size class has its own free list of partially-filled slabs of that size. For slabs, `gc_bits` field becomes a bitset that tracks which slots in the page are currently in-use, reducing the overhead for small objects to only a little over a single bit each!

In the diagram below, bits set in the 32-bit, little-endian bitset indicate which slots in the slab (no to scale!) are filled with three-letter words. (The user likes cats.)

```text highlight

  Pd--------------------------------------------+
  | gc_bits: 0b01010011111010100110000010101011 |
  +---------------------------------------------+

 Page--------------------------------------------+
 | cat | ink |     | hat |     | jug |     | fig |
 +-----------------------------------------------+
 |     |     |     |     |     | zip | net |     |
 +-----------------------------------------------+
 |     | aid |     | yes |     | war | cat | van |
 +-----------------------------------------------+
 | can | cat |     |     | rat |     | urn |     |
 +-----------------------------------------------+

```

[Plaintext](#code:18)

Allocating an object is a bit more complicated now, but now we have a really, really short fast path for small objects:

1. Round up to the next highest size class, or else to the next page boundary.
2. If a slab size class… a. Check the pertinent slab list for a partially-filled slab. i. If there isn’t one, allocate a page per the instructions below and initialize it as a slab page. b. Find the next available slot with `(!gc_bits).count_trailing_zeros()`, and set that bit. c. Return `page_addr + slab_slot_size * slot`.
3. Else, if a single page, allocate from the single-page list. a. If there isn’t one, allocate from the ream list as usual.
4. Else, multiple pages, allocate a ream as usual.

Allocating small objects is very fast, since the slab free lists, if not empty, will always have a spot to fill in `gc_bits`. Finding the empty spot in the bitset is a few instructions (a `not` plust a `ctz` or equivalent on most hardware).

Alkyne maintains a separate free list for single free pages to speed up finding such pages to turn into fresh slabs. This also minimizes the need to allocate single pages off of large reams, which limits fragmentation.

Alkyne’s size classes are the powers of two from 8 (the smallest possible object) to 2048. For the classes 8, 16, and 32, which would have more than 64 slots in the page, we use up to 56 bytes on the page itself to extend `gc_bits`; 8-byte pages can only hold 505 objects, instead of the full 512, a 1% overhead.

Directly freeing an object via is now tricky, since we do not a priori know the size.

1. Round the pointer up to the next page boundary, and obtain that page’s `Pd`.
2. If this is a start-of-ream page, stick it into the appropriate free list (single page or ream, depending on the size of the ream).
3. Else, we can look at `class` to find the size class, and from that, and the offset of the original pointer into the page, the index of the slot.
4. Clear the slot’s index in `gc_bits`.
5. If the page was full before, place it onto the correct slab free list; if it becomes empty, place it into the page free list.

At this point, we know whether the page just became partially full or empty, and can move it to the correct free list.

Size classes are an important allocator optimization. TCMalloc takes this to an . These constants are generated by some crazy script based on profiling data.

## [Intermission](\#intermission)

Before continuing to the GC part of the article, it’s useful to go over what we learned.

A neat thing about this is that most of these tricks are somewhat independent. While giving feedback for an early draft, Matt Kulukundis shared [this awesome talk](https://www.youtube.com/watch?v=LIb3L4vKZ7U) that describes how to build complex allocators out of simple ones, and covers many of the same tropes as we did here. This perspective on allocators actually blew my mind.

Good allocators don’t just use one strategy; the use many and pick and chose the best one for the job based on expected workloads. For example, Alkyne expects to allocate many small objects; the slab pages were originally only for float objects, but it turned out to simplify a lot of the code to make *all* small objects be slab-allocated.

Even size classes are a deep topic: TCMalloc uses [GWP telemetry](https://research.google/pubs/pub36575/) from Google’s fleet to inform its *many* tuning parameters, including its [comically large](https://github.com/google/tcmalloc/blob/master/tcmalloc/size_classes.cc) tables of size classes.

At this point, we have a pretty solid allocator. Now, let’s get rid of the free function.

## [Throwing out the Trash](\#throwing-out-the-trash)

Garbage collection is very different from manual memory management in that frees are performed in *batches* without cue from the user. There are no calls to `free()`; instead, we need to figure out which calls to `free()` we *can* make on the user’s behalf that they won’t notice (i.e., without quietly freeing pointers the user can still reach, resulting in a use-after-free bug). We need to do this as fast as we can.

Alkyne is a “tracing GC”. Tracing GCs walk the “object graph” from a root set of known-reachable objects. Given an object `a`, it will *trace* through any data in the object that it knows is actually a GC pointer. In the object graph, `b` is reachable from `a` if one can repeatedly trace through GC pointers to get from `a` to `b`.

Alkyne uses tracing to implement garbage collection in a two-step process, commonly called “mark-and-sweep”.

*Marking* consists of traversing the entire graph from a collection of reachable-by-definition values, such as things on the stack, and recording each object that is visited. Every object *not* so marked must therefore be definitely unreachable and can be reclaimed; this reclamation is called *sweeping*.

Alkyne reverses the order of operations somewhat: it “sweeps” first and then marks, i.e., it marks every value as dead and then, as it walks the graph, marks every block as alive. It then rebuilds the free lists to reflect the new marks, allowing the blocks to be reallocated. This is sometimes called “mark and don’t sweep”, but fixing up the free lists is effectively a sweeping step.

![](https://upload.wikimedia.org/wikipedia/commons/4/4a/Animation_of_the_Naive_Mark_and_Sweep_Garbage_Collector_Algorithm.gif)Marking and sweeping! (via Wikipedia, CC0)

Alkyne is a “stop-the-world” (STW) GC. It needs to pause all program execution while cleaning out the heap. It is possible to build GCs that do not do this (I believe modern HotSpot GCs very rarely stop the world), but also very difficult. Most GCs are world-stopping to some degree.

One thing we do not touch on is *when* to sweep. This is a more complicated and somewhat hand-wavy tuning topic that I’m going to quietly sweep under the rug by pointing you to [how Go does it](https://cs.opensource.google/go/go/+/master:src/runtime/mgcpacer.go).

### [Heap Armageddon and Resurrection](\#heap-armageddon-and-resurrection)

Delicate freeing of individual objects is quite difficult, but scorching the earth is very easy. To do this, we walk the whole `Pd` array (see, I said this would be useful!) and blow away every `gc_bits`. This leaves the heap in a broken state where every pointer appears to be dangling. This is “armageddon”.

To fix this up, we need to “resurrect” any objects we shouldn’t have killed (oops). The roots are objects in the Alkyne interpreter stack[11](#fn:11). To mark an object, we convert a pointer to it into a `Pd` via the page- `Pd` correspondence, and mark it as alive by “allocating” it.

We then use our knowledge[12](#fn:12) of Alkyne objects’ heap layout to find pointers to other objects in the heap (for example, the intepreter *knows* it’s looking at a list and can *just find* the list elements within, which are likely pointers themselves). If we trace into an object and find it has been marked as allocated, we don’t recurse; this avoids infinite recursion when encountering cycles.

> [note](#note:3)
>
> It’s a big hard to give a code example for this, because the “mark” part that’s part of the GC is mixed up with interpreter code, so there isn’t much to show in this case. :(

At the end of this process, every reachable object will once again be alive, but anything we couldn’t reach stays dead.

### [Instant Apocalypse](\#instant-apocalypse)

(Alkyne currently does not make this optimization, but really should.)

Rather than flipping every bit, we flip the global *convention* for whether 0 or 1 means “alive”, implemented by having a global `bool` specifying which is which at any given time; this would alternate from sweep to sweep. Thus, killing every living object is now a single operation.

This works if the allocated bit of objects in the free lists is never read, and only ever overwritten with the “alive” value when allocated, so that all of the dead objects suddenly becoming alive isn’t noticed. This does not work with slab-allocated small objects: pages may be in a mixed state where they are partially allocated and partially freed.

We can still make this optimization by adding a second bit that tracks whether the page contains *any* living objects, using the same convention. This allows delaying the clear of the allocated bits for small objects to when the page is visited, which also marks the whole page as alive.

Pages that were never visited (i.e., still marked as dead) can be reclaimed as usual, ignoring the allocated bits.

### [Free List Reconciliation](\#free-list-reconciliation)

At this point, no pointers are dangling, but newly emptied out pages are not in the free lists they should be in. To fix this, we can walk over all `Pd` s and put them where they need to go if they’re not full. This is the kinda-but-not-really sweep phase.

The code for this is simpler to show than explaining it:

```rust highlight
for pd in self.reams() {
  if pd.gc_bits == 0 {
    pd.unlink();
    if pd.len == 0 {
      unsafe { self.page_free_list.push(pd) }
    } else {
      unsafe { self.ream_free_list.push(pd) }
    }
  } else if pd.is_full() {
    // GC can't make a not-full-list become full, so we don't
    // need to move it.
  } else {
    // Non-empty, non-full lists cannot be reams.
    debug_assert!(pd.class != SizeClass::Ream);

    pd.unlink();
    unsafe {
      self.slab_free_lists[pd.class].push(pd)
    }
  }
}

```

[Rust](#code:19)

Of course, this will also shuffle around all pages that did not become partially empty or empty while marking. If the “instant apocalypse” optimization is used, this step must still inspect every `Pd` and modify the free lists.

However, it is a completely separate phase: all it does is find pages that did not survive the previous mark phase. This means that user code can run between the phases, reducing latency. If it turns out to be very expensive to sweep the whole heap, it can even be run less often than mark phases[13](#fn:13).

This is also a great chance to merge reams, because we’re inspecting every page anyways; this is why the merging strategy depends on wanting to be a GC’s allocator rather than a normal `malloc()`/ `free()` allocator.

…and that’s it! That’s garbage collection. The setup of completely owning the layout of blocks in the allocator allows us to cut down significantly on memory needed to track objects in the heap, while keeping the mark and sweep steps short and sweet. A garbage collector is like any other data structure: you pack in a lot of complexity into the invariants to make the actual operations very quick.

## [Conclusion](\#conclusion)

Alkyne’s GC is intended to be super simple because I didn’t want to think too hard about it (even though I clearly did lmao). The GC layouts are a whole ’nother story I have been swirling around in my head for months, which is described [here](https://github.com/mcy/alkyne/blob/a62ad3b7ee70268625da640c1edeea8ff7116512/src/eval2/value.rs#L78). The choices made there influenced the design of the GC itself.

There are still many optimizations to make, but it’s a really simple but realistic GC design, and I’m pretty happy with it!

### [A Note on Finalizers (Tools of the Devil!)](\#a-note-on-finalizers-tools-of-the-devil)

Alkyne also does not provide finalizers. A finalizer is the GC equivalent of a destructor: it gets run after the GC declares an object dead. Finalizers complicate a GC significantly by their very nature; they are called in unspecified orders and can witness broken GC state; they can stall the entire program (if they are implemented to run during the GC pause in a multi-threaded GC) or else need to be called with a zombie argument that either can’t escape the finalizer or, worse, must be resurrected if it does!

If finalizers depend on each other, they can’t be run at all, for the same reason an ARC cycle cannot be broken; this weakness of ARC is one of the major benefits of an omniscient GC.

Java’s [documentation for `Object.finalize()`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Object.html#finalize()) is a wall of text of lies, damned lies, and ghost stories.

I learned earlier (the week before I started writing this article) that Go ALSO has finalizers and that they are [similarly cursed](https://pkg.go.dev/runtime#SetFinalizer). Go does behave somewhat more nicely than Java (finalizers are per-value and avoid zombie problems by unconditionally resurrecting objects with a finalizer).

### [Further Reading](\#further-reading)

Here are some other allocators that I find interesting and worth reading about, some of which have inspired elements of Alkyne’s design.

[TCMalloc](https://google.github.io/tcmalloc/design.html) is Google’s crazy thread-caching allocator. It’s really fast and really cool, but I work for Google, so I’m biased. But it uses radix trees! Radix trees are cool!!!

Go [has a garbage collector](https://cs.opensource.google/go/go/+/master:src/runtime/mgc.go) that has well-known performance properties but does not perform any wild optimizations like moving, and is a world-stopping, incremental GC.

[Oilpan](https://chromium.googlesource.com/chromium/src/+/master/third_party/blink/renderer/platform/heap/BlinkGCAPIReference.md) is the Chronimum renderer’s GC (you know, for DOM elements). It’s actually grafted onto C++ and has a very complex API reflective of the subtleties of GCs as a result.

[libboehm](https://www.hboehm.info/gc/) is another C/C++ GC written by Hans Boehm, one of the world’s top experts on concurrency.

[Orinoco](https://v8.dev/blog/trash-talk) is V8’s GC for the JavaScript heap (i.e., Chronimum’s *other* GC). It is a *generational* or *moving GC* that can defragment the heap over time by moving things around (and updating pointers). It also has a separate sub-GC just for short-lived objects.

[Mesh](https://arxiv.org/abs/1902.04738) is a non-GC allocator that can do compacting via clever use of `mmap(2)`.

[`upb_Arena`](https://github.com/protocolbuffers/upb/blob/1cf8214e4daa1d0dd9777c987697e82c2a3c6584/upb/upb.c#L117) is an arena allocator that uses free-lists to allows fusing arenas together. This part of the μpb Protobuf runtime.

---

1. In other words, it uses recursion along a syntax tree, instead of a more efficient approach that compiles the program down to bytecode. [↩︎](#fnref:1)

2. *A* utomatic *R* eference *C* ounting is an automatic memory management technique where every heap allocation contains a counter of how many pointers currently point to it; once pointers go out of scope, they decrement the counter; when the counter hits zero the memory is freed.

   This is used by Python and Swift as the core memory management strategy, and provided by C++ and Rust via the `std::shared_ptr<T>` and `Arc<T>` types, respectively. [↩︎](#fnref:2)

3. [This is the file.](https://github.com/mcy/alkyne/blob/a62ad3b7ee70268625da640c1edeea8ff7116512/src/eval2/gc.rs) It’s got fairly complete comments, but they’re written for an audience familiar with allocators and garbage collectors. [↩︎](#fnref:3)

4. This is a tangent, but I should point out that Alkyne does not do [“NaN
   boxing”](https://leonardschuetz.ch/blog/nan-boxing/). This is a technique used by some JavaScript runtimes, like Spidermonkey, which represent dynamically typed values as either ordinary floats, or pointers hidden in the mantissas of 64-bit IEEE 754 signaling NaNs.

   Alkyne instead uses something like V8’s [Smi pointer compression](https://v8.dev/blog/pointer-compression), so our heap values are four bytes, not eight. Non-Smi values that aren’t on the stack (which uses a completely different representation) can only exist as elements of lists or objects. Alkyne’s slab allocator design (described below) is focused on trying to minimize the overhead of all floats being in their own little allocations. [↩︎](#fnref:4)

5. The operating system’s own physical page allocator is actually solving the same problem: given a vast range of memory (in this case, physical RAM), allocate it. The algorithms in this article apply to those, too.

   Operating system allocators can be slightly fussier because they need to deal with virtual memory mappings, but that is a topic for another time. [↩︎](#fnref:5)

6. As you might expect, these scare-quotes are load-bearing. [↩︎](#fnref:6)

7. I tried leaving this out of the first draft, and failed. So many things would be simpler without fussing around with alignment. [↩︎](#fnref:7)

8. Yes yes most architectures can cope with unaligned loads and stores but compilers rather like to pretend that’s not true. [↩︎](#fnref:8)

9. Boutique means [provenance](https://llvm.org/docs/LangRef.html#pointer-aliasing-rules) in French. [↩︎](#fnref:9)

10. Currently Alkyne has a rather small max ream size. A better way to approach this would be to treat the entire heap as one gigantic ream at the start, which is always at the bottom of the free list. [↩︎](#fnref:10)

11. In GC terms, these are often called “stack roots”. [↩︎](#fnref:11)

12. The interpreter simply *knows* this and can instruct the GC appropriately.

    In any tracing GC, the compiler or interpreter must be keenly aware of the layouts of types so that it can generate the appropriate tracing code for each.

    This is why grafting GCs to non-GC’d languages is non-trivial, even though people have totally done it: [libboehm](https://www.hboehm.info/gc/) and [Oilpan](https://chromium.googlesource.com/chromium/src/+/master/third_party/blink/renderer/platform/heap/BlinkGCAPIReference.md) are good (albeit sometimes controversial) examples of how this can be done. [↩︎](#fnref:12)

13. With “instant apocalypse”, this isn’t quite true; after two mark phases, pages from the first mark phase will appear to be alive, since the global “alive” convention has changed twice. Thus, only pages condemned in every other mark phase will be swept; sweeping is most optimal after an odd number of marks. [↩︎](#fnref:13)
