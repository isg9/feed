---
title: I Wrote A String Type
url: /2023/08/09/yarns
published: "2023-08-09T00:00:00Z"
feed: mcyoung
guid: /2023/08/09/yarns
---

# I Wrote A String Type

I write compilers for fun. I can’t help it. Consequently, I also write a lot of parsers. In systems programming, it’s usually a good idea to try to share memory rather than reuse it, so as such my AST types tend to look like this.

```rust highlight
pub enum Expr<'src> {
  Int(u32)
  Ident(&'src str),
  // ...
}

```

[Rust](#code:1)

Whenever we parse an identifier, rather than copy its name into a fresh `String`, we borrow from the input source string. This avoids an extra allocation, an extra copy, and saves a word in the representation. Compilers can be memory-hungry, so it helps to pick a lean representation.

Unfortunately, it’s not so easy for quoted strings. Most strings, like `"all my jelly babies"`, are “literally” in the original source, like an identifier. But strings with escapes aren’t: `\n` is encoded in the source code with the bytes `[0x5c, 0x6e]`, but the actual “decoded” value of a string literal replaces each escape with a single `0x0a`.

The usual solution is a [`Cow<str>`](https://doc.rust-lang.org/std/borrow/enum.Cow.html). In the more common, escape-less verison, we can use `Cow::Borrowed`, which avoids the extra allocation and copy, and in the escaped version, we decode the escapes into a `String` and wrap it in a `Cow::Owned`.

For example, suppose that we’re writing a parser for a language that has quoted strings with escapes. The string `"all my jelly babies"` can be represented as a byte string that borrows the input source code, so we’d use the `Cow::Borrowed` variant. This is most strings in any language: escapes tend to be rare.

For example, if we have the string `"not UTF-8 \xff"`, the actual byte string value is different from that in the source code.

```text highlight
// Bytes in the source.
hex:   6e 6f 74 20 55 54 46 2d 38 20 5c 78 66 66
ascii: n  o  t     U  T  F  -  8     \  x  f  f

// Bytes represented by the string.
hex:   6e 6f 74 20 55 54 46 2d 38 20 ff
ascii: n  o  t     U  T  F  -  8

```

[Plaintext](#code:2)

Escapes are relatively rare, so most strings processed by the parser do not need to pay for an allocation.

However, we still pay for that extra word, since `Cow<str>` is 24 bytes (unless otherwise specified, all byte counts assume a 64-bit system), which is eight more than our `&str`. Even worse, this is bigger than the string data itself, which is 11 bytes.

If most of your strings are small (which is not uncommon in an AST parser), you will wind up paying for significant overhead.

Over the years I’ve implemented various optimized string types to deal with this use-case, in various contexts. I finally got around to putting all of the tricks I know into a library, which I call [`byteyarn`](https://docs.rs/byteyarn/latest/byteyarn/). It advertises the following nice properties.

> [reference](#ref:1)
>
> A `Yarn` is a highly optimized string type that provides a number of useful properties over `String`:
> - Always two pointers wide, so it is always passed into and out of functions in registers.
> - Small string optimization (SSO) up to 15 bytes on 64-bit architectures.
> - Can be either an owned buffer or a borrowed buffer (like `Cow<str>`).
> - Can be upcast to `'static` lifetime if it was constructed from a known-static string.

I’d like to share how these properties are achieved through careful layout optimization.

## [Assumptions](\#assumptions)

We’re going to start by stating assumptions about how our strings will be used:

1. Most strings are not mutated most of the time.
2. Most strings are small.
3. Most strings are substrings.

### [Most Strings are Immutable](\#most-strings-are-immutable)

`String` is modeled after C++’s `std::string`, which is a growable buffer that implements amortized linear-time append. This means that if we are appending `n` bytes to the buffer, we only pay for `n` bytes of `memcpy`.

This is a useful but often unnecessary property. For example, Go strings are immutable, and when building up a large string, you are expected to use `strings.Builder`, which is implemented as essentially a Rust `String`. Java also as a similar story for strings, which allows for highly compact representations of `java.lang.String` s.

In Rust, this kind of immutable string is represented by a `Box<str>`, which is eight bytes smaller than `String`. Converting from `String` to `Box<str>` is just a call to `realloc()` to resize the underlying allocation (which is often cheap[1](#fn:1)) from being `capacity` bytes long to `len` bytes long.

Thus, this assumption means we only need to store a pointer and a length, which puts our memory footprint floor at 16 bytes.

### [Most Strings are Substrings](\#most-strings-are-substrings)

Suppose again that we’re parsing some textual format. Many structural elements will be verbatim references into the textual input. Not only string literals without escapes, but also identifiers.

`Box<str>` cannot hold borrowed data, because it will always instruct the allocator to free its pointer when it goes out of scope. `Cow<str>`, as we saw above, allows us to handle maybe-owned data uniformly, but has a minimum 24 byte overhead. This can’t be made any smaller, because a `Cow<str>` can contain a 24-byte `String` value.

But, we don’t want to store a capacity. Can we avoid the extra word of overhead in `Cow<str>`?

### [Most Strings are Small](\#most-strings-are-small)

Consider a string that is not a substring but which is small. For example, when parsing a string literal like `"Hello, world!\n"`, the trailing `\n` (bytes `0x5c 0x6e`) must be replaced with a newline byte ( `0x0a`). This means we must handle a tiny heap allocation, 14 bytes long, that is smaller than a `&str` referring to it.

This is worse for single character[2](#fn:2) strings. The overhead for a `Box<str>` is large.

- The `Box<str>` struct itself has a pointer field (eight bytes), and a length field (also eight bytes). Spelled out to show all the stored bits, the length is `0x0000_0000_0000_0001`. That’s a lot of zeroes!
- The pointer itself points to a heap allocation, which will not be a single byte! Allocators are not in the business of handing out such small pieces of memory. Instead, the allocation is likely costing us another eight bytes!

So, the string `"a"`, whose data is just a *single byte*, instead takes up 24 bytes of memory.

It turns out that for really small strings we can avoid the allocation altogether, *and* make effective use of all those zeroes in the `len` field.

## [Stealing Bits](\#stealing-bits)

Let’s say we want to stick to a budget of 16 bytes for our `Yarn` type. Is there any extra space left for data in a `(*mut u8, usize)` pair?

*\*cracks Fermi estimation knuckles\**

A `usize` is 64 bits, which means that the length of an `&str` can be anywhere from zero to 18446744073709551615, or around 18 exabytes. For reference, “hundreds of exabytes” is a reasonable ballpark guess for how much RAM exists in 2023 (consider: 4 billion smartphones with 4GB each). More practically, the largest quantity of RAM you can fit in a server blade is measured in terabytes (much more than your measly eight DIMs on your gaming rig).

If we instead use one less bit, 63 bits, this halves the maximum representable memory to nine exabytes. If we take another, it’s now four exabytes. Much more memory than you will ever *ever* want to stick in a string. [Wikpedia asserts](https://en.wikipedia.org/wiki/Wikipedia:Size_of_Wikipedia#Size_of_the_English_Wikipedia_database) that Wikimedia Commons contains around 428 terabytes of media (the articles' text with history is a measly 10 TB).

Ah, but you say you’re programming for a 32-bit machine (today, this likely means either a low-end mobile phone, an embedded micro controller, or WASM).

On a 32-bit machine it’s a little bit harrier: Now `usize` is 32 bits, for a maximum string size of 4 gigabytes (if you remember the 32-bit era, this limit may sound familiar). “Gigabytes” is an amount of memory that you can actually imagine having in a string.

Even then, 1 GB of memory (if we steal two bits) on a 32-bit machine is a lot of data. You can only have four strings that big in a single address space, and every 32-bit allocator in the universe will refuse to serve an allocation of that size. If your strings are comparable in size to the whole address space, you should build your own string type.

The upshot is that every `&str` contains two bits we can reasonably assume are not used. *Free real-estate.* [3](#fn:3)

### [A Hand-Written Niche Optimization](\#a-hand-written-niche-optimization)

Rust has the concept of *niches*, or invalid bit-patterns of a particular type, which it uses for automatic layout optimization of `enum` s. For example, references cannot be null, so the pointer bit-pattern of `0x0000_0000_0000_0000` is never used; this bit-pattern is called a “niche”. Consider:

```rust highlight
enum Foo<'a> {
  First(&'a T),
  Second
}

```

[Rust](#code:3)

An `enum` of this form will not need any “extra” space to store the value that discriminates between the two variants: if a `Foo`’s bits are all zero, it’s `Foo::Second`; otherwise it’s a `Foo::First` and the payload is formed from `Foo`’s bit-pattern. This, incidentally, is what makes `Option<&T>` a valid representation for a “nullable pinter”.

There are more general forms of this: `bool` is represented as a single byte, of which two bit are valid; the other 254 potential bit-patterns are niches. In Recent versions of Rust, `RawFd` has a niche for the all-ones bit-pattern, since POSIX file descriptors are always non-negative `int` s.

By stealing two bits off of the length, we have given ourselves four niches, which essentially means we’ll have a hand-written version of something like this `enum`.

```rust highlight
enum Yarn {
  First(*mut u8, u62),
  Second(*mut u8, u62),
  Third(*mut u8, u62),
  Fourth(*mut u8, u62),
}

```

[Rust](#code:4)

For reasons that will become clear later, we will specifically steal the *high* bits of the length, so that to recover the length, we do two shifts[4](#fn:4) to shift in two high zero bits. Here’s some code that actually implements this for the low level type our string type will be built on.

```rust highlight
#[repr(C)]
#[derive(Copy, Clone)]
struct RawYarn {
  ptr: *mut u8,
  len: usize,
}

impl RawYarn {
  /// Constructs a new RawYarn from raw components: a 2-bit kind,
  /// a length, and a pointer.
  fn from_raw_parts(kind: u8, len: usize, ptr: *mut u8) -> Self {
    assert!(len <= usize::MAX / 4, "no way you have a string that big");

    RawYarn {
      ptr,
      len: (kind as usize & 0b11) << (usize::BITS - 2) | len,
    }
  }

  /// Extracts the kind back out.
  fn kind(self) -> u8 {
    (self.len >> (usize::BITS - 2)) as u8
  }

  /// Extracts the slice out (regardless of kind).
  unsafe fn as_slice(&self) -> &[u8] {
    slice::from_raw_parts(self.ptr, (self.len << 2) >> 2)
  }
}

```

[Rust](#code:5)

Note that I’ve made this type `Copy`, and some functions take it by value. This is for two reasons.

1. There is a type of `Yarn` that is itself `Copy`, although I’m not covering it in this article.

2. It is a two-word struct, which means that on most architectures it is eligible to be passed in a pair of registers. Passing it by value in the low-level code helps promote keeping it in registers. This isn’t always possible, as we will see when we discuss “SSO”.

Let’s chose kind `0` to mean “this is borrowed data”, and kind `1` to be “this is heap-allocated data”. We can use this to remember whether we need to call a destructor.

```rust highlight
pub struct Yarn<'a> {
  raw: RawYarn,
  _ph: PhantomData<&'a str>,
}

const BORROWED: u8 = 0;
const HEAP: u8 = 1;

impl<'a> Yarn<'a> {
  /// Create a new yarn from borrowed data.
  pub fn borrowed(data: &'a str) -> Self {
    let len = data.len();
    let ptr = data.as_ptr().cast_mut();
    Self {
      raw: RawYarn::from_raw_parts(BORROWED, len, ptr),
      _ph: PhantomData,
    }
  }

  /// Create a new yarn from owned data.
  pub fn owned(data: Box<str>) -> Self {
    let len = data.len();
    let ptr = data.as_ptr().cast_mut();
    mem::forget(data);

    Self {
      raw: RawYarn::from_raw_parts(HEAP, len, ptr),
      _ph: PhantomData,
    }
  }

  /// Extracts the data.
  pub fn as_slice(&self) -> &str {
    unsafe {
      // SAFETY: initialized either from uniquely-owned data,
      // or borrowed data of lifetime 'a that outlives self.
      str::from_utf8_unchecked(self.raw.as_slice())
    }
  }
}

impl Drop for Yarn<'_> {
  fn drop(&mut self) {
    if self.raw.kind() == HEAP {
      let dropped = unsafe {
        // SAFETY: This is just reconstituting the box we dismantled
        // in Yarn::owned().
        Box::from_raw(self.raw.as_mut_slice())
      };
    }
  }
}

impl RawYarn {
  unsafe fn as_slice_mut(&mut self) -> &mut [u8] {
    // Same thing as as_slice, basically. This is just to make
    // Box::from_raw() above typecheck.
  }
}

```

[Rust](#code:6)

This gives us a type that strongly resembles `Cow<str>` with only half of the bytes. We can even write code to extend the lifetime of a `Yarn`:

```rust highlight
impl Yarn<'_> {
  /// Removes the bound lifetime from the yarn, allocating if
  /// necessary.
  pub fn immortalize(mut self) -> Yarn<'static> {
    if self.raw.kind() == BORROWED {
      let copy: Box<str> = self.as_slice().into();
      self = Yarn::owned(copy);
    }

    // We need to be careful that we discard the old yarn, since its
    // destructor may run and delete the heap allocation we created
    // above.
    let raw = self.raw;
    mem::forget(self);
    Yarn::<'static> {
      raw,
      _ph: PhantomData,
    }
  }
}

```

[Rust](#code:7)

The remaining two niches can be put to use for optimizing small strings.

## [Small String Optimization](\#small-string-optimization)

C++’s `std::string` also makes the “most strings are small” assumption. In the `libc++` implementation of the standard library, `std::string` s of up to 23 bytes never hit the heap!

C++ implementations do this by using most of the pointer, length, and capacity fields as a storage buffer for small strings, the so-called “small string optimization” (SSO). In `libc++`, in SSO mode, a `std::string`’s length fits in one byte, so the other 23 bytes can be used as storage. The capacity isn’t stored at all: an SSO string always has a capacity of 23.

`RawYarn` still has another two niches, so let’s dedicate one to a “small” representation. In small mode, the kind will be 2, and only the 16th byte will be the length.

This is why we used the two *high* bits of `len` for our scratch space: no matter what mode it’s in, we can easily extract these bits[5](#fn:5). Some of the existing `RawYarn` methods need to be updated, though.

```rust highlight
#[repr(C)]
#[derive(Copy, Clone)]
struct RawYarn {
  ptr: MaybeUninit<*mut u8>,
  len: usize,
}

const SMALL: u8 = 2;

impl RawYarn {
  /// Constructs a new RawYarn from raw components: a 2-bit kind,
  /// a length, and a pointer.
  fn from_raw_parts(kind: u8, len: usize, ptr: *mut u8) {
    debug_assert!(kind != SMALL);
    assert!(len <= usize::MAX / 4, "no way you have a string that big");

    RawYarn {
      ptr: MaybeUninit::new(ptr),
      len: (kind as usize & 0b11) << (usize::BITS - 2) | len,
    }
  }

  /// Extracts the slice out (regardless of kind).
  unsafe fn as_slice(&self) -> &[u8] {
    let (ptr, adjust) = match self.kind() {
      SMALL => (self as *const Self as *const u8, usize::BITS - 8),
      _ => (self.ptr.assume_init(), 0),
    };

    slice::from_raw_parts(ptr, (self.len << 2) >> (2 + adjust))
  }
}

```

[Rust](#code:8)

In the non- `SMALL` case, we shift twice as before, but in the `SMALL` case, we need to get the high byte of the `len` field, so we need to shift down by an additional `usize::BITS - 8`. No matter what we’ve scribbled on the low bytes of `len`, we will always get just the length this way.

We also need to use a different pointer value depending on whether we’re in `SMALL` mode. This is why `as_slice` needs to take a reference argument, since the slice data may be *directly* in `self`!

Also, `ptr` is a `MaybeUninit` now, which will become clear in the next code listing.

We should also provide a way to construct small strings.

```rust highlight
const SSO_LEN: usize = size_of::<usize>() * 2 - 1;

impl RawYarn {
  /// Create a new small yarn. `data` must be valid for `len` bytes
  /// and `len` must be smaller than `SSO_LEN`.
  unsafe fn from_small(data: *const u8, len: usize) -> RawYarn {
    debug_assert!(len <= SSO_LEN);

    // Create a yarn with an uninitialized pointer value (!!)
    // and a length whose high byte is packed with `small` and
    // `len`.
    let mut yarn = RawYarn {
      ptr: MaybeUninit::uninit(),
      len: (SMALL as usize << 6 | len)
          << (usize::BITS - 8),
    };

    // Memcpy the data to the new yarn.
    // We write directly onto the `yarn` variable. We won't
    // overwrite the high-byte length because `len` will
    // never be >= 16.
    ptr::copy_nonoverlapping(
      data,
      &mut yarn as *mut RawYarn as *mut u8,
      data,
    );

    yarn
  }
}

```

[Rust](#code:9)

The precise maximum size of an SSO string is a bit more subtle than what’s given above, but it captures the spirit. The `RawYarn::from_small` illustrates why the pointer value is hidden in a `MaybeUninit`: we’re above to overwrite it with garbage, and in that case it won’t be a pointer at all.

We can update our public `Yarn` type to use the new small representation whenever possible.

```rust highlight
impl<'a> Yarn<'a> {
  /// Create a new yarn from borrowed data.
  pub fn borrowed(data: &'a str) -> Self {
    let len = data.len();
    let ptr = data.as_ptr().cast_mut();

    if len <= SSO_LEN {
      return Self {
        raw: unsafe { RawYarn::from_small(len, ptr) },
        _ph: PhantomData,
      }
    }

    Self {
      raw: RawYarn::from_raw_parts(BORROWED, len, ptr),
      _ph: PhantomData,
    }
  }

  /// Create a new yarn from owned data.
  pub fn owned(data: Box<str>) -> Self {
    if data.len() <= SSO_LEN {
      return Self {
        raw: unsafe { RawYarn::from_small(data.len(), data.as_ptr()) },
        _ph: PhantomData,
      }
    }

    let len = data.len();
    let ptr = data.as_ptr().cast_mut();
    mem::forget(data);

    Self {
      raw: RawYarn::from_raw_parts(HEAP, len, ptr),
      _ph: PhantomData,
    }
  }
}

```

[Rust](#code:10)

It’s also possible to construct a `Yarn` directly from a character now, too!

```rust highlight
impl<'a> Yarn<'a> {
  /// Create a new yarn from borrowed data.
  pub fn from_char(data: char) -> Self {
    let mut buf = [0u8; 4];
    let data = data.encode_utf8(&mut buf);
    Self {
      raw: unsafe { RawYarn::from_small(len, ptr) },
      _ph: PhantomData,
    }
  }
}

```

[Rust](#code:11)

(Note that we do not need to update `Yarn::immortalize()`; why?)

What we have now is a maybe-owned string that does not require an allocation for small strings. However, we still have an extra niche…

## [String Constants](\#string-constants)

String constants in Rust are interesting, because we can actually detect them at compile-time[6](#fn:6).

We can use the last remaining niche, 3, to represent data that came from a string constant, which means that it does not need to be boxed to be immortalized.

```rust highlight
const STATIC: u8 = 3;

impl<'a> Yarn<'a> {
  /// Create a new yarn from borrowed data.
  pub fn from_static(data: &'static str) -> Self {
    let len = data.len();
    let ptr = data.as_ptr().cast_mut();

    if len <= SSO_LEN {
      return Self {
        raw: unsafe { RawYarn::from_small(len, ptr) },
        _ph: PhantomData,
      }
    }

    Self {
      raw: RawYarn::from_raw_parts(STATIC, len, ptr),
      _ph: PhantomData,
    }
  }
}

```

[Rust](#code:12)

This function is identical to `Yarn::borrowed`, except that `data` most now have a static lifetime, and we pass `STATIC` to `RawYarn::from_raw_parts()`.

Because of how we’ve written all of the prior code, this does not require any special support in `Yarn::immortalize()` or in the low-level `RawYarn` code.

The actual `byteyarn` library provides a `yarn!()` macro that has the same syntax as `format!()`. This is the primary way in which yarns are created. It is has been carefully written so that `yarn!("this is a literal")` always produces a `STATIC` string, rather than a heap-allocated string.

## [An extra niche, as a treat?](\#an-extra-niche-as-a-treat)

Unfortunately, because of how we’ve written it, `Option<Yarn>` is 24 bytes, a whole word larger than a `Yarn`. However, there’s still a little gap where we can fit the `None` variant. It turns out that because of how we’ve chosen the discriminants, `len` is zero if and only if it is an empty `BORROWED` string. But this is not the only zero: if the high byte is `0x80`, this is an empty `SMALL` string. If we simply require that no other empty string is ever constructed (by marking `RawYarn::from_raw_parts()` as unsafe and specifying it should not be passed a length of zero), we can guarantee that `len` is *never* zero.

Thus, we can update `len` to be a `NonZeroUsize`.

```rust highlight
#[repr(C)]
#[derive(Copy, Clone)]
struct RawYarn {
  ptr: MaybeUninit<*mut u8>,
  len: NonZeroUsize,  // (!!)
}

impl RawYarn {
  /// Constructs a new RawYarn from raw components: a 2-bit kind,
  /// a *nonzero* length, and a pointer.
  unsafe fn from_raw_parts(kind: u8, len: usize, ptr: *mut u8) {
    debug_assert!(kind != SMALL);
    debug_assert!(len != 0);
    assert!(len <= usize::MAX / 4, "no way you have a string that big");

    RawYarn {
      ptr: MaybeUninit::new(ptr),
      len: NonZeroUsize::new_unchecked(
        (kind as usize & 0b11) << (usize::BITS - 2) | len),
    }
  }
}

```

[Rust](#code:13)

This is a type especially known to the Rust compiler to have a niche bit-pattern of all zeros, which allows `Option<Yarn>` to be 16 bytes too. This also has the convenient property that the all zeros bit-pattern for `Option<Yarn>` is `None`.

## [Conclusion](\#conclusion)

The [`byteyarn`](https://docs.rs/byteyarn/latest/byteyarn/) blurb describes what we’ve built:

> [reference](#ref:2)
>
> A `Yarn` is a highly optimized string type that provides a number of useful properties over `String`:
> - Always two pointers wide, so it is always passed into and out of functions in registers.
> - Small string optimization (SSO) up to 15 bytes on 64-bit architectures.
> - Can be either an owned buffer or a borrowed buffer (like `Cow<str>`).
> - Can be upcast to `'static` lifetime if it was constructed from a known-static string.

There are, of course, some trade-offs. Not only do we need the assumptions we made originally to hold, but we also need to relatively care more about memory than cycle-count performance, since basic operations like reading the length of the string require more math (but no extra branching).

The actual implementation of `Yarn` is a bit more complicated, partly to keep all of the low-level book-keeping in one place, and partly to offer an ergonomic API that makes `Yarn` into a mostly-drop-in replacement for `Box<str>`.

I hope this peek under the hood has given you a new appreciation for what can be achieved by clever layout-hacking.

---

1. Allocators rarely serve you memory with precisely the size you asked for. Instead, they will have some notion of a “size class” that allows them to use more efficient allocation techniques, [which I have written about](/2022/06/07/alkyne-gc).

As a result, if the size change in a `realloc()` would not change the size class, it becomes a no-op, especially if the allocator can take advantage of the current-size information Rust provides it. [↩︎](#fnref:1)

2. Here and henceforth “character” means “32-bit Unicode scalar”. [↩︎](#fnref:2)

3. Now, you might also point out that Rust and C do not allow an allocation whose size is larger than the pointer offset type ( `isize` and `ptrdiff_t`, respectively). In practice this means that the high bit is *always* zero according to the language’s own rules.

This is true, but we need to steal two bits, and I wanted to demonstrate that this is an extremely reasonable desire. 64-bit integers are so comically large. [↩︎](#fnref:3)

4. Interestingly, LLVM will compile `(x << 2) >> 2` to

```x86 highlight
movabs rax,0x3fffffffffffffff
and    rax,rdi
ret

```

[x86 Assembly](#code:14)

If we want to play the byte-for-byte game, this costs 14 bytes when encoded in the Intel variable-length encoding. You would think that two shifts would result in marginally smaller code, but no, since the input comes in in `rdi` and needs to wind up in `rax`.

On RISC-V, though, it seems to decide that two shifts is in fact cheaper, and will even optimize `x & 0x3fff_ffff_ffff_ffff` back into two shifts. [↩︎](#fnref:4)

5. This only works on little endian. Thankfully all computers are little endian. [↩︎](#fnref:5)

6. Technically, a `&'static str` may also point to leaked memory. For our purposes, there is no essential difference. [↩︎](#fnref:6)
