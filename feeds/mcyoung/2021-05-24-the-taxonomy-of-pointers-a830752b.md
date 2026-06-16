---
title: The Taxonomy of Pointers
url: /2021/05/24/ptr-taxonomy
published: "2021-05-24T00:00:00Z"
feed: mcyoung
guid: /2021/05/24/ptr-taxonomy
---

# The Taxonomy of Pointers

Writing `unsafe` in Rust usually involves manual management of memory. Although, ideally, we’d like to exclusively use references for this, sometimes the constraints they apply are too strong. This post is a guide on those constraints and how to weaken them for correctness.

“Unmanaged” languages, like C++ and Rust, provide *pointer* types for manipulating memory. These types serve different purposes and provide different guarantees. These guarantees are useful for the optimizer but get in the way of correctness of low-level code. This is especially true in Rust, where these constraints are very tight.

> [note](#note:1)
>
> This post only surveys *data* pointers. Function pointers are their own beast, but generally are less fussy, since they all have static lifetime[1](#fn:1).

## [Basic C++ Pointers](\#basic-c-pointers)

First, let’s survey C++. We have three pointer types: the traditional C pointer `T*`, C++ references `T&`, and rvalue references `T&&`. These generally have pretty weak guarantees.

Pointers provide virtually no guarantees at all: they can be null, point to uninitialized memory, or point to nothing at all! C++ Only requires that they be aligned[2](#fn:2). They are little more than an address (until they are dereferenced, of course).

References, on the other hand, are intended to be the “primary” pointer type. A `T&` cannot be null, is well-aligned, and is intended to only refer to live memory (although it’s not something C++ can really guarantee for lack of a borrow-checker). References are short-lived.

C++ uses non-nullness to its advantage. For example, Clang will absolutely delete code of the form

```cpp highlight
auto& x = Foo();
if (&x == nullptr) {
  DoSomething();
}

```

[C++](#code:1)

Because references cannot be null, and dereferencing the null pointer is always UB, the compiler may make this fairly strong assumption.

Rvalue references, `T&&`, are not meaningfully different from normal references, beyond their role in overload resolution.

Choosing a C++ (primitive) pointer type is well-studied and not the primary purpose of this blog. Rather, we’re interested in how these map to Rust, which has significantly more complicated pointers.

## [Basic Rust Pointers](\#basic-rust-pointers)

Like C++, Rust has two broad pointer types: `*const T` and `*mut T`, the raw pointers, and `&T` and `&mut T`, the references.

Rust pointer have even fewer constraints than C++ pointers; they need not even be aligned[3](#fn:3)! The `const`/ `mut` specifier is basically irrelevant, but is useful as programmer book-keeping tool. Rust also does not enforce the dreaded strict-aliasing rule[4](#fn:4) on its pointers.

On the other hand, Rust references are among the most constrained objects in any language that I know of. A shared reference `&'a T`, lasting for the lifetime `'a`, satisfies:

- Non-null, and well-aligned (like in C++).
- Points to a valid, initialized `T` for the duration of `'a`.
- `T` is never ever mutated for the duration of the reference: the compiler may fold separate reads into one at will. Stronger still, no `&mut T` is reachable from any thread while the reference is reachable.

Stronger still are `&'a mut T` references, sometimes called *unique* references, because in addition to being well-aligned and pointing to a valid `T` at all times, no other reachable reference ever aliases it in any thread; this is equivalent to a C `T* restrict` pointer.

Unlike C++, which has two almost-identical pointer types, Rust’s two pointer types provide either no guarantees or *all* of them. The following `unsafe` operations are all UB:

```rust highlight
let null = unsafe { &*ptr::null() };

// A reference to u8 need not be sufficiently aligned
// for a reference to u32.
let unaligned = unsafe { &*(&0u8 as *const u8 as *const u32) };

// More on this type later...
let uninit = unsafe { &*MaybeUninit::uninit().as_ptr() };

let x = 0;
unsafe {
  // Not UB in C++ with const_cast!
  let p = &x;
  (p as *const i32 as *mut i32).write(42);
}

// Two mutable references live at the same time pointing to
// the same memory. This would also be fine in C++!
let mut y = 0;
let p1 = unsafe { &*(&mut y as *mut i32) };
let p2 = unsafe { &*(&mut y as *mut i32) };

```

[Rust](#code:2)

### [Wide Pointers](\#wide-pointers)

Rust also provides the slice types `&[T]` [5](#fn:5) (of which you get mutable/immutable reference and pointer varieties) and dynamic trait object types `&dyn Tr` (again, all four basic pointer types are available).

`&[T]` is a `usize` [6](#fn:6) length plus a pointer to that many `T` s. The pointer type of the slice specifies the guarantees on the pointed-to buffer. `*mut [T]`, for example, has no meaningful guarantees, but still contains the length[7](#fn:7). Note that the length is part of the pointer value, *not* the pointee.

`&dyn Tr` is a trait object. For our purposes, it consists of a pointer to some data plus a pointer to a static vtable. `*mut dyn Tr` is technically a valid type[8](#fn:8). Overall, trait objects aren’t really relevant to this post; they are rarely used this way in `unsafe` settings.

## [Weakening the Guarantees](\#weakening-the-guarantees)

Suppose we’re building some kind of data structure; in Rust, data structures will need some sprinkling of `unsafe`, since they will need to shovel around memory directly. Typically this is done using raw pointers, but it is preferable to use the *least weakened* pointer type to allow the compiler to perform whatever optimizations it can.

There are a number of orthogonal guarantees on `&T` and `&mut T` we might want to relax:

- Non-nullness.
- Well-aligned-ness.
- Validity and initialized-ness of the pointee.
- Allocated-ness of the pointee (implied by initialized-ness).
- Global uniqueness of an `&mut T`.

### [Pointer to ZST](\#pointer-to-zst)

The last three of these properties are irrelevant for a zero-sized type. For example, we can generate infinite `&mut ()` with no consequences:

```rust highlight
fn unique_unit() -> &'static mut () {
  unsafe { &mut *(0x1 as *mut ()) }
}

```

[Rust](#code:3)

We materialize a non-null, well-aligned pointer and reborrow it into a static reference; because there is no data to point to, none of the usual worries about the pointee itself apply. However, the pointer itself must still be non-null and well-aligned; `0x1` is not a valid address for an `&[u32; 0]`, but `0x4` is[9](#fn:9).

This also applies to empty slices; in fact, the compiler will happily promote the expression `&mut []` to an arbitrary lifetime:

```rust highlight
fn unique_empty<T>() -> &'static mut [T] {
  &mut []
}

```

[Rust](#code:4)

### [Null References](\#null-references)

The most well-known manner of weakening is `Option<&T>`. Rust guarantees that this is ABI-compatible with a C pointer `const T*`, with `Option::<&T>::None` being a null pointer on the C side. This “null pointer optimization” applies to any type recursively containing at least one `T&`.

```rust highlight
extern "C" {
  fn DoSomething(ptr: Option<&mut u32>);
}

fn do_something() {
  DoSomething(None);  // C will see a `NULL` as the argument.
}

```

[Rust](#code:5)

The same effect can be achieved for a pointer type using the [`NonNull<T>`](https://doc.rust-lang.org/std/ptr/struct.NonNull.html) standard library type: `Option<NonNull<T>>` is identical to `*mut T`. This is most beneficial for types which would otherwise contain a raw pointer:

```rust highlight
struct Vec<T> {
  ptr: NonNull<T>,
  len: usize,
  cap: usize,
}

assert_eq!(size_of::<Vec<u8>>(), size_of::<Option<Vec<u8>>>())

```

[Rust](#code:6)

### [Uninitialized Pointee](\#uninitialized-pointee)

No matter what, a `&T` cannot point to uninitialized memory, since the compiler is free to assume it may read such references at any time with no consequences.

The following classic C pattern is verboten:

```c highlight
Foo foo;
initialize_foo(&foo);

```

[C](#code:7)

Rust doesn’t provide any particularly easy ways to allocate memory without initializing it, too, so this usually isn’t a problem. The [`MaybeUninit<T>`](https://doc.rust-lang.org/core/mem/union.MaybeUninit.html#method.assume_init_ref) type can be used for safely allocating memory without initializing it, via `MaybeUninit::uninit()`.

This type acts as a sort of “optimization barrier” that prevents the compiler from assuming the pointee is initialized. `&MaybeUninit<T>` is a pointer to potentially uninitialized but definitely allocated memory. It has the same layout as `&T`, and Rust provides functions like `assume_init_ref()` for asserting that a `&MaybeUninit<T>` is definitely initialized. This assertion is similar in consequence to dereferencing a raw pointer.

`&MaybeUninit<T>` and `&mut MaybeUninit<T>` should almost be viewed as pointer types in their own right, since they can be converted to/from `&T` and `&mut T` under certain circumstances.

Because `T` is almost a “subtype” of `MaybeUninit<T>`, we are entitled[10](#fn:10) to “forget” that the referent of a `&T` is initialized converting it to a `&MaybeUninit<T>`. This makes sense because `&T` is covariant[11](#fn:11) in `&T`. However, this is not true of `&mut T`, since it’s not covariant:

```rust highlight
let mut x = 0;
let uninit: &mut MaybeUninit<i32> = unsafe { transmute(&mut x) };
*uninit = MaybeUninit::uninit();  // Oops, `x` is now uninit!

```

[Rust](#code:8)

These types are useful for talking to C++ without giving up too many guarantees. `Option<&MaybeUninit<T>>` is an almost perfect model of a `const T*`, under the assumption that most pointers in C++ are valid most of the time.

`MaybeUninit<T>` also finds use in working with raw blocks of memory, such as in a `Vec`-style growable slice:

```rust highlight
struct SliceVec<'a, T> {
  // Backing memory. The first `len` elements of it are
  // known to be initialized, but no more than that.
  data: &'a mut [MaybeUninit<T>],
  len: usize,
}

impl SliceVec<'a, T> {
  fn push(&mut self, x: T) {
    assert!(self.len < data.len());

    self.data[self.len] = MaybeUninit::new(x);
    self.len += 1;
  }
}

```

[Rust](#code:9)

### [Aliased Pointee](\#aliased-pointee)

`&mut T` can never alias any other pointer, but is also the mechanism by which we perform mutation. It can’t even alias with pointers that Rust can’t see; Rust assumes no one else can touch this memory. Thus, `&mut T` is not an appropriate analogue for `T&`.

Like with uninitialized memory, Rust provides a “barrier” wrapper type, [`UnsafeCell<T>`](https://doc.rust-lang.org/std/cell/struct.UnsafeCell.html). `UnsafeCell<T>` is the “interior mutability” primitive, which permits us to mutate through an `&UnsafeCell<T>` so long as concurrent reads and writes do not occur. We may even convert it to a `&mut T` when we’re sure we’re holding the only reference.

`UnsafeCell<T>` forms the basis of the `Cell<T>`, `RefCell<T>`, and `Mutex<T>` types, each of which performs a sort of “dynamic borrow-checking”:

- `Cell<T>` only permits direct loads and stores.
- `RefCell<T>` maintains a counter of references into it, which it uses to dynamically determine if a mutable reference would be unique.
- `Mutex<T>`, which is like `RefCell<T>` but using concurrency primitives to maintain uniqueness.

Because of this, Rust must treat `&UnsafeCell<T>` as always aliasing, but because we can mutate through it, it is a much closer analogue to a C++ `T&`. However, because `&T` assumes the pointee is never mutated, it cannot coexist with a `&UnsafeCell<T>` to the same memory, if mutation is performed through it. The following is explicitly UB:

```rust highlight
let mut x = 0;
let p = &x;

// This is ok; creating the reference to UnsafeCell does not
// immediately trigger UB.
let q = unsafe { transmute::<_, &UnsafeCell<i32>>(&x) };

// But writing to it does!
q.get().write(42);

```

[Rust](#code:10)

The `Cell<T>` type is useful for non-aliasing references to plain-old-data types, which tend to be `Copy`. It allows us to perform mutation without having to utter `unsafe`. For example, the correct type for a shared mutable buffer in Rust is `&[Cell<u8>]`, which can be freely `memcpy`’d, without worrying about aliasing[12](#fn:12).

This is most useful for sharing memory with another language, like C++, which cannot respect Rust’s aliasing rules.

### [Combined Barriers](\#combined-barriers)

To recap:

- Non-nullness can be disabled with `Option<&T>`.
- Initialized-ness can be disabled with `&MaybeUninit<T>`.
- Uniqueness can be disabled with `&UnsafeCell<T>`.

> [warning](#warning:1)
>
> There is no way to disable alignment and validity restrictions: references must always be aligned and have a valid lifetime attached. If these are unachievable, raw pointers are your only option.

We can combine these various “weakenings” to produce aligned, lifetime-bound references to data with different properties. For example:

- `&UnsafeCell<MaybeUninit<T>>` is as close as we can get to a C++ `T&`.
- `Option<&UnsafeCell<T>>` is a like a raw pointer, but to initialized memory.
- `Option<&mut MaybeUninit<T>>` is like a raw pointer, but with alignment, aliasing, and lifetime requirements.
- `UnsafeCell<&[T]>` permits us to mutate the pointer to the buffer and its length, but not the values it points to themselves.
- `UnsafeCell<&[UnsafeCell<T>]>` lets us mutate both the buffer and its actual pointer/length.

Interestingly, there is no equivalent to a C++ raw pointer: there is no way to create a guaranteed-aligned pointer without a designated lifetime[13](#fn:13).

## [Other Pointers](\#other-pointers)

Rust and C++ have many other pointer types, such as smart pointers. However, in both languages, both are built in terms of these basic pointer types. Hopefully this article is a useful reference for anyone writing `unsafe` abstraction that wishes to avoid using raw pointers when possible.

---

1. Except in Go, which synthesizes vtables *on the fly*. Story for another day. [↩︎](#fnref:1)

2. It is, apparently, a little-known fact that constructing unaligned pointers, but then never dereferencing them, is still UB in C++. C++ could, for example, store information in the lower bits of such a pointer. The in-memory representation of a pointer is actually unspecified! [↩︎](#fnref:2)

3. This is useful when paired with the Rust `<*const T>::read_unaligned()` function, which can be compiled down to a normal load on architectures that do not have alignment restrictions, like x86\_64 and aarch64. [↩︎](#fnref:3)

4. Another story for another time. [↩︎](#fnref:4)

5. Comparable to the C++20 [`std::span<T>`](https://en.cppreference.com/w/cpp/container/span) type. [↩︎](#fnref:5)

6. `usize` is Rust’s machine word type, compare `std::uintptr_t`. [↩︎](#fnref:6)

7. The length of a `*mut [T]` can be accessed via the unstable [`<*mut [T]>::len()`](https://doc.rust-lang.org/std/primitive.pointer.html#method.len-1) method. [↩︎](#fnref:7)

8. It is also not a type I have encountered enough to have much knowledge on. For example, I don’t actually know if the vtable half of a `*mut dyn Tr` must always be valid or not; I suspect the answer is “no”, but I couldn’t find a citation for this. [↩︎](#fnref:8)

9. Note that you *cannot* continue to use a reference to freed, zero-sized memory. This subtle distinction is called out in [https://doc.rust-lang.org/std/ptr/index.html#safety](https://doc.rust-lang.org/std/ptr/index.html#safety). [↩︎](#fnref:9)

10. Currently, a `transmute` must be used to perform this operation, but I see no reason way this would permit us to perform an illegal mutation without uttering `unsafe` a second time. In particular, `MaybeUninit::assume_init_read()`, which could be used to perform illegal copies, is an `unsafe` function. [↩︎](#fnref:10)

11. A covariant type `Cov<T>` is once where, if `T` is a subtype of `U`, then `Cov<T>` is a subtype of `Cov<U>`. This isn’t particularly noticeable in Rust, where the only subtyping relationships are `&'a T` subtypes `&'b T` when `'a` outlives `'b`, but is nonetheless important for advanced type design. [↩︎](#fnref:11)

12. `Cell<T>` does not provide synchronization; you still need locks to share it between threads. [↩︎](#fnref:12)

13. I have previously proposed a sort of `'unsafe` or `'!` “lifetime” that is intended to be the lifetime of dangling references (a bit of an oxymoron). This would allow us to express this concept, but I need to flesh out the concept more. [↩︎](#fnref:13)
