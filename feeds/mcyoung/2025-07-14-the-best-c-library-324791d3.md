---
title: The Best C++ Library
url: /2025/07/14/best
published: "2025-07-14T00:00:00Z"
feed: mcyoung
guid: /2025/07/14/best
---

# The Best C++ Library

It’s no secret that my taste in programming languages is very weird for a programming language ~~enthusiast~~ professional. Several of my [last](https://mcyoung.xyz/2025/07/07/nosplit/) [few](https://mcyoung.xyz/2025/04/21/go-arenas/) [posts](https://mcyoung.xyz/2024/12/16/rangefuncs/) are about Go, broadly regarded as the programming language equivalent of eating plain oatmeal for breakfast.

To make up for that, I’m going to write about the programming language equivalent of diluting your morning coffee with [Everclear](https://en.wikipedia.org/wiki/Everclear). I am, of course, talking about C++.

If you’ve ever had the misfortune of doing C++ professionally, you’ll know that the C++ standard library is really bad. Where to begin?

Well, the associative containers are terrible. Due to bone-headed API decisions, [`std::unordered_map`](https://en.cppreference.com/w/cpp/container/unordered_map.html) MUST be a closed-addressing, array-of-linked-lists map, not a Swisstable, despite closed-addressing being an outdated technology. [`std::map`](https://www.cppreference.com/w/cpp/container/map.html), which is not what you usually want, *must* be a red-black tree. It can’t be a b-tree, like every sensible language provides for the ordered map.

[`std::optional`](https://en.cppreference.com/w/cpp/utility/optional.html) is a massive pain in the ass to use, and is full of footguns, like `operator*`. `std::variant` is also really annoying to use. [`std::filesystem`](https://en.cppreference.com/w/cpp/filesystem.html) is full of sharp edges. And where are the APIs for signals?

Everything is extremely wordy. [`std::hardware_destructive_interference_size`](https://en.cppreference.com/w/cpp/thread/hardware_destructive_interference_size.html) could have been called `std::cache_line`. [`std::span::subspan`](https://en.cppreference.com/w/cpp/container/span/subspan) could have used `opeartor[]`. The standard algorithms are super wordy, because they deal with iterator pairs. Oh my god, iterator pairs. They added [`std::ranges`](https://en.cppreference.com/w/cpp/ranges.html), which do not measure up to Rust’s [`Iterator`](https://doc.rust-lang.org/std/iter/trait.Iterator.html) at all!

I’m so mad about all this! The people in charge of C++ clearly, actively hate their users![1](#fn:1) They want C++ to be as hard and unpleasant as possible to use. Many brilliant people that I am lucky to consider friends and colleagues, including Titus Winters, JeanHeyd Meneide, Matt Fowles-Kulukundis, and Andy Soffer, have tried and mostly failed[2](#fn:2) to improve the language.

This is much to say that I believe C++ in its current form is unfixable. But that’s only due to the small-mindedness of a small cabal based out of Redmond. What if we could do whatever we wanted? What if we used C++’s incredible library-building language features to build a brand-new language?

For the last year-or-so I’ve been playing with a wild idea: what would C++ look like if we did it over again? Starting from an empty C++20 file with no access to the standard library, what can we build in its place?

## [Starting Over](\#starting-over)

Titus started Abseil while at Google, whose namespace, `absl`, is sometimes said to stand for “a better standard library”[3](#fn:3). To me, Abseil is important because it was an attempt to work with the existing standard library and make it better, while retaining a high level of implementation quality that a C++ shop’s home-grown utility library won’t have, and a uniformity of vision that [Boost](https://www.boost.org/) is too all-over-the-place to achieve.

Rather than trying to coexist with the standard library, I want to surpass it. As a form of performance art, I want to discover what the standard library would look like if we designed it *today*, in 2025.

In this sense, I want to build something that isn’t just *better*. It should be the C++ standard library from the best possible world. It is the best possible library. This is why my library’s namespace is `best`.

In general, I am trying not to directly copy either what C++, or Abseil, or Rust, or Go did. However, each of them has really interesting ideas, and the best library probably lies in some middle-ground somewhere.

The rest of this post will be about what I have achieved with `best` so far, and where I want to take it. You can look at the code [here](https://github.com/mcy/best).

### [Building a Foundation](\#building-a-foundation)

We’re throwing out everything, and that includes `<type_traits>`. This is a header which shows its age: alias templates were’t added until C++14, and variable templates were added in C++17. As a result, many things that really aught to be concepts have names like `best::is_same_v`. All of these now have concept equivalents in `<concepts>`.

I have opted to try to classify type traits into separate headers to make them easier to find. They all live under `//best/meta/traits`, and they form the leaves of the dependency graph.

For example, `arrays.h` contains all of the array traits, such as `best::is_array`, `best::un_array` (to remove an array extent), and `best::as_array`, which applies an extent to a type T, such that `best::as_array<T, 0>` is not an error.

`types.h` contains very low-level metaprogramming helpers, such as:

- `best::id` and `best::val`, the identity traits for type- and value-kinded traits.
- `best::same<...>`, which returns whether an entire *pack* of types is all equal.
- `best::lie`, our version of `std::declval`.
- `best::select`, our `std::conditional_t`.
- `best::abridge`, a “symbol compression” mechanism for shortening the names of otherwise huge symbols.

`funcs.h` provides `best::tame`, which removes the qualifiers from an [abominable function type](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/p0172r0.html). `quals.h` provides `best::qualifies_to`, necessary for determining if a type is “more const” than another. `empty.h` provides a standard empty type that interoperates cleanly with `void`.

On top of the type traits is the metaprogramming library `//best/meta`, which includes generalized constructibility traits in `init.h` (e.g., to check that you can, in fact, initialize a `T&` from a `T&&`, for example). `tlist.h` provides a very general type-level heterogenous list abstraction; a parameter pack as-a-type.

The other part of “the foundation” is `//best/base`, which mostly provides access to intrinsics, portability helpers, macros, and “tag types” such as our versions of `std::in_place`. For example, `macro.h` provides `BEST_STRINGIFY()`, `port.h` provides `BEST_HAS_INCLUDE()`, and `hint.h` provides `best::unreachable()`.

`guard.h` provides our version of the Rust `?` operator, which is not an expression because statement expressions are broken in Clang.

Finally, within `//best/container` we find `best::object`, a special type for turning any C++ type into an object (i.e., a type that you can form a reference to). This is useful for manipulating any type generically, without tripping over the assign-through semantics of references. For example, `best::object<T&>` is essentially a pointer.

### [“ADT” Containers](\#adt-containers)

On top of this foundation we build the basic algebraic data types of `best`: `best::row` and `best::choice`, which replace `std::tuple` and `std::variant`.

`best::row<A, B, C>` is a heterogenous collection of values, stored inside of `best::object` s. This means that `best::row<int&>` has natural rebinding, rather than assign-through, semantics.

Accessing elements is done with `at()`: `my_row.at<0>()` returns a reference to the first element. Getting the first element is so common that you can also use `my_row.first()`. Using `my_row.object<0>()` will return a reference to a `best::object` instead, which can be used for rebinding references. For example:

```cpp highlight
int x = 0, y = 0;
best::row<int&> a{x};
a.at<0>() = 42;     // Writes to x.
a.object<0>() = y;  // Rebinds a.0 to y.
a.at<0>() = 2*x;    // Writes to y.

```

[C++](#code:1)

There is also `second()` and `last()`, for the other two most common elements to access.

`best::row` is named so in reference to database rows: it provides many operations for slicing and dicing that `std::tuple` does not.

For example, in addition to extracting single elements, it’s also possible to access contiguous subsequences, using `best::bounds`: `a.at<best::bounds{.start = 1, .end = 10}>()`! There are also a plethora of mutation operations:

- `a + b` concatenates tuples, copying or moving as appropriate ( `a + BEST_MOVE(b)` will move out of the elements of `b`, for example).
- `a.push(x)` returns a copy of `a` with `x` appended, while `a.insert<n>(x)` does the same at an arbitrary index.
- `a.update<n>(x)` *replaces* the `n` th element with `x`, potentially of a different type.
- `a.remove<n>()` deletes the `n` th element, while `a.erase<...>()` deletes a contiguous range.
- `a.splice<best::bounds{...}>(...)` splices a row into another row, offering a general replace/delete operation that all of the above operations are implemented in terms of.
- `gather()` and `scatter()` are even more general, allowing for non-contiguous indexing.

Meanwhile, `std::apply` is a method now: `a.apply(f)` calls `f` with `a`’s elements as its arguments. `a.each(f)` is similar, but instead expands to `n` unary calls of `f`, one with each element.

And *of course*, `best::row` supports structured bindings.

Meanwhile, `best::choice<A, B, C>` contains precisely one value from various types. There is an underlying `best::pun<A, B, C>` type that implements a variadic untagged union that works around many of C++’s bugs relating to unions with members of non-trivial type.

The most common way to operate on a choice is to `match` on it:

```cpp highlight
best::choice<int, *int, void> z = 42;
int result = z.match(
  [](int x) { return x; },
  [](int* x) { return *x; },
  [] { return 0; }
);

```

[C++](#code:2)

Which case gets called here is chosen by overload resolution, allowing us to write a default case as `[](auto&&) { ... }`.

Which variant is currently selected can be checked with `z.which()`, while specific variants can be accessed with `z.at()`, just like a `best::row`, except that it returns a `best::option<T&>`.

`best::choice` is what all of the other sum types, like `best::option` and `best::result`, are built out of. All of the clever layout optimizations live here.

Speaking of `best::option<T>`, that’s our option type. It’s close in spirit to what [`Option<T>`](https://doc.rust-lang.org/std/option/enum.Option.html) is in Rust. `best` has a generic niche mechanism that user types can opt into, allowing `best::option<T&>` to be the same size as a pointer, using `nullptr` for the `best::none` variant.

`best::option` provides the usual transformation operations: `map`, `then`, `filter`. Emptiness can be checked with `is_empty()` or `has_value()`. You can even pass a predicate to `has_value()` to check the value with, if it’s present: `x.has_value([](auto& x) { return x == 42; })`.

The value can be accessed using `operator*` and `operator->`, like `std::optional`; however, this operation is checked, instead of causing UB if the option is empty. `value_or()` can be used to unwrap with a default; the default can be any number of arguments, which are used to construct the default, or even a callback. For example:

```cpp highlight
best::option<Foo> x;

// Pass arguments to the constructor.
do_something(x.value_or(args, to, foo));

// Execute arbitrary logic if the value is missing.
do_something(x.value_or([] {
  return Foo(...);
}))

```

[C++](#code:3)

`best::option<void>` also Just Works (in fact, `best::option<T>` is a `best::choice<void, T>` internally), allowing for truly generic manipulation of optional results.

`best::result<T, E>` is, unsurprisingly, the analogue of Rust’s [`Result<T, E>`](https://doc.rust-lang.org/std/result/enum.Result.html). Because it’s a `best::choice` internally, `best::result<void, E>` works as you might expect, and is a common return value for I/O operations.

It’s very similar to `best::option`, including offering `operator->` for accessing the “ok” variant. This enables succinct idioms:

```cpp highlight
if (auto r = fallible()) {
  r->do_something();
} else {
  best::println("{}", *r.err());
}

```

[C++](#code:4)

`r.ok()` and `r.err()` return `best::option` s containing references to the ok and error variants, depending on which is actually present; meanwhile, a `best::option` can be converted into a `best::result` using `ok_or()` or `err_or()`, just like in Rust.

`best::result` s are constructed using `best::ok` and `best::err`. For example:

```cpp highlight
best::result<Foo, Error> r = best::ok(args, to, foo);

```

[C++](#code:5)

These internally use `best::args`, a wrapper over `best::row` that represents a “delayed initialization” that can be stored in a value. It will implicitly convert into any type that can be constructed from its elements. For example:

```cpp highlight
Foo foo = best::args(args, to, foo);  // Calls Foo::Foo(args, to, foo).

```

[C++](#code:6)

Also, every one of the above types is a structural type, meaning it can be used for non-type template parameters!

### [Memory and Pointers](\#memory-and-pointers)

Of course, all of these ADTs need to be built on top of pointer operations, which is where `//best/memory` comes in. `best::ptr<T>` is a generalized pointer type that provides many of the same operations as Rust’s raw pointers, including offsetting, copying, and indexing. Like Rust pointers, `best::ptr<T>` can be a fat pointer, i.e., it can carry additional metadata on top of the pointer. For example, `best::ptr<int[]>` remembers the size of the array.

Providing metadata for a `best::ptr` is done through a member alias called `BestPtrMetadata`. This alias should be private, which `best` is given access to by befriending `best::access`. Types with custom metadata will usually not be directly constructible (because they are of variable size), and must be manipulated exclusively through types like `best::ptr`.

Specifying custom metadata allows specifying what the pointer dereferences to. For example, `best::ptr<int[]>` dereferences to a `best::span<int>`, meaning that all the span operations are accessible through `operator->`: for example, `my_array_ptr->first()`.

Most of this may seem a bit over-complicated, since ordinary C++ raw pointers and references are fine for most uses. However, `best::ptr` is the foundation upon which `best::box<T>` is built on. `best::box<T>` is a replacement for [`std::unique_ptr<T>`](https://en.cppreference.com/w/cpp/memory/unique_ptr.html) that fixes its const correctness and adds Rust [`Box`](https://doc.rust-lang.org/std/boxed/struct.Box.html)-like helpers. `best::box<T[]>` also works, but unlike `std::unique_ptr<T[]>`, it remembers its size, just like `best::ptr<T[]>`.

`best::box` is parameterized by its allocator, which must satisfy `best::allocator`, a much less insane API than what [`std::allocator`](https://en.cppreference.com/w/cpp/memory/allocator.html) offers. `best::malloc` is a singleton allocator representing the system allocator.

`best::span<T>`, mentioned before, is the contiguous memory abstraction, replacing [`std::span`](https://en.cppreference.com/w/cpp/container/span.html). Like `std::span`, `best::span<T, n>` is a fixed-length span of `n` elements. Unlike `std::span`, the second parameter is a `best::option<size_t>`, not a `size_t` that uses `-1` as a sentinel.

`best::span<T>` tries to approximate the API of [Rust slices](https://doc.rust-lang.org/std/primitive.slice.html), providing indexing, slicing, splicing, search, sort, and more. Naturally, it’s also iterable, both forwards and backwards, and provides splitting iterators, just like Rust.

Slicing and indexing is always bounds-checked. Indexing can be done with `size_t` values, while slicing uses a `best::bounds`:

```cpp highlight
best::span<int> xs = ...;
auto x = xs[5];
auto ys = xs[{.start = 1, .end = 6}];

```

[C++](#code:7)

`best::bounds` is a generic mechanism for specifying slicing bounds, similar to Rust’s [range types](https://doc.rust-lang.org/std/ops/struct.Range.html). You can specify the start and end (exclusive), like `x..y` in Rust. You can also specify an inclusive end using `.inclusive_end = 5`, equivalent to Rust’s `x..=y`. And you can specify a count, like C++’s slicing operations prefer: `{.start = 1, .count = 5}`. `best::bounds` itself provides all of the necessary helpers for performing bounds checks and crashing with a nice error message. `best::bounds` is also iterable, as we’ll see shortly.

`best::layout` is a copy of Rust’s [`Layout`](https://doc.rust-lang.org/std/alloc/struct.Layout.html) type, providing similar helpers for performing C++-specific size and address calculations.

### [Iterators](\#iterators)

C++ iterator pairs suck. C++ ranges suck. `best` provides a new paradigm for iteration that is essentially just Rust [`Iterator` s](https://doc.rust-lang.org/std/iter/trait.Iterator.html) hammered into a C++ shape. This library lives in `//best/iter`.

To define an iterator, you define an *iterator implementation type*, which must define a member function named `next()` that returns a `best::option`:

```cpp highlight
class my_iter_impl final {
  public:
    best::option<int> next();
};

```

[C++](#code:8)

This type is an implementation detail; the actual iterator type is `best::iter<my_iter_impl>`. `best::iter` provides all kinds of helpers, just like `Iterator`, for adapting the iterator or consuming items out of it.

Iterators can override the behavior of some of these adaptors to be more efficient, such as for making `count()` constant-time rather than linear. Iterators can also offer extra methods if they define the member alias `BestIterArrow`; for example, the iterators for `best::span` have a `->rest()` method for returning the part of the slice that has not been yielded by `next()` yet.

One of the most important extension points is `size_hint()`, analogous to [`Iterator::size_hint()`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.size_hint), for right-sizing containers that the iterator is converted to, such as a `best::vec`.

And of course, `best::iter` provides `begin/end` so that it can be used in a C++ range-for loop, just like C++20 ranges do. `best::int_range<I>` [4](#fn:4), which `best::bounds` is an instantiation of, is also an iterator, and can be used much like Rust ranges would:

```cpp highlight
for (auto i : best::int_range<int>{.start = 1, .count = 200}) {
  // ...
}

```

[C++](#code:9)

`best::int_range` will carefully handle all of the awkward corner cases around overflow, such as `best::int_range<uint8_t>{.end_inclusive = 255}`.

### [Heap Containers](\#heap-containers)

Iterators brings us to the most complex container type that’s checked in right now, `best::vec`. Not only can you customize its allocator type, but you can customize its small vector optimization type.

In `libc++`, `std::string` s of at most 23 bytes are stored *inline*, meaning that the strings’s own storage, rather than heap storage, is used to hold them. `best::vec` generalizes this, by allowing any trivially copyable type to be inlined. Thus, a `best::vec<int>` will hold at most five `int` s inline, on 64-bit targets.

`best::vec` mostly copies the APIs of `std::vector` and Rust’s [`Vec`](https://doc.rust-lang.org/std/vec/struct.Vec.html). Indexing and slicing works the same as with `best::span`, and all of the `best::span` operations can be accessed through `->`, allowing for things like `my_vec->sort(...)`.

I have an active (failing) PR which adds `best::table<K, V>`, a general hash table implementation that can be used as either a map or a set. Internally it’s backed by a Swisstable[5](#fn:5) implementation. Its API resembles neither `std::unordered_map`, `absl::flat_hash_map`, or Rust’s [`HashMap`](https://doc.rust-lang.org/std/collections/struct.HashMap.html). Instead, everything is done through a general entry API, similar to that of Rust, but optimized for clarity and minimizing hash lookups. I want to get it merged soonish.

Beyond `best::table`, I plan to add *at least* the following containers:

- `best::tree`, a btree map/set with a similar API.
- `best::heap`, a simple min-heap implementation.
- `best::lru`, a `best::table` with a linked list running through it for in-order iteration and oldest-member eviction.
- `best::ring`, a ring buffer like [`VecDeque`](https://doc.rust-lang.org/std/collections/struct.VecDeque.html).
- `best::trie`, a port of my [`twie` crate](https://docs.rs/twie/latest/twie/).

Possible other ideas: [Russ’s sparse array](https://research.swtch.com/sparse), splay trees, something like Java’s [`EnumMap`](https://docs.oracle.com/javase/8/docs/api/java/util/EnumMap.html), bitset types, and so on.

### [Text Handling](\#text-handling)

`best`’s string handling is intended to resemble Rust’s as much as possible; it lives within `//best/text`. `best::rune` is the Unicode scalar type, which is such that it is *always* within the valid range for a Unicode scalar, but including the unpaired surrogates. It offers a number of relatively simple character operations, but I plan to extend it to all kinds of character classes in the future.

`best::str` is our replacement for [`best::string_view`](https://en.cppreference.com/w/cpp/string/basic_string_view.html), close to Rust’s [`str`](https://doc.rust-lang.org/std/primitive.str.html): a sequence of valid UTF-8 bytes, with all kinds of string manipulation operations, such as rune search, splitting, indexing, and so on.

`best::rune` and `best::str` use compiler extensions to ensure that when constructed from literals, they’re constructed from *valid* literals. This means that the following won’t compile!

```cpp highlight
best::str invalid = "\xFF";

```

[C++](#code:10)

`best::str` is a `best::span` under the hood, which can be accessed and manipulated the same way as the underlying `&[u8]` to `&str` is.

`best::strbuf` is our [`std::string`](https://en.cppreference.com/w/cpp/string/basic_string.html) equivalent. There isn’t very much to say about it, because it works just like you’d expect, and provides a Rust [`String`](https://doc.rust-lang.org/std/string/struct.String.html)-like API.

Where this library really shines is that everything is parametrized over encodings. `best::str` is actually a `best::text<best::utf8>`; `best::str16` is then `best::text<best::utf16>`. You can write your own text encodings, too, so long as they are relatively tame and you provide rune encode/decode for them. `best::encoding` is the concept

`best::text` is always validly encoded; however, sometimes, that’s not possible. For this reason we have `best::pretext`, which is “presumed validly encoded”; its operations can fail or produce replacement characters if invalid code units are found. There is no `best::pretextbuf`; instead, you would generally use something like a `best::vec<uint8_t>` instead.

Unlike C++, the fact that a `best::textbuf` is a `best::vec` under the hood is part of the public interface, allowing for cheap conversions and, of course, we get `best::vec`’s small vector optimization for free.

`best` provides the following encodings out of the box: `best::utf8`, `best::utf16`, `best::utf32`, `best::wtf8`, `best::ascii`, and `best::latin1`.

### [Formatting](\#formatting)

`//best/text:format` provides a Rust `format!()`-style text formatting library. It’s as easy as:

```cpp highlight
auto str = best::format("my number: 0x{:08x}", n);

```

[C++](#code:11)

Through the power of compiler extensions and `constexpr`, the format is actually checked at compile time!

The available formats are the same as Rust’s, including the `{}` vs `{:?}` distinction. But it’s actually way more flexible. You can use any ASCII letter, and types can provide multiple custom formatting schemes using letters. By convention, `x`, `X`, `b`, and `o` all mean numeric bases. `q` will quote strings, runes, and other text objects; `p` will print pointer addresses.

The special format `{:!}` “forwards from above”; when used in a formatting implementation, it uses the format specifier the caller used. This is useful for causing formats to be “passed through”, such as when printing lists or `best::option`.

Any type can be made formattable by providing a friend template ADL extension (FTADLE) called `BestFmt`. This is analogous to implementing a trait like `fmt::Debug` in Rust, however, all formatting operations use the same function; this is similar to [`fmt.Formatter`](https://pkg.go.dev/fmt#Formatter) in Go.

The `best::formatter` type, which gets passed into `BestFmt`, is similar to Rust’s [`Formatter`](https://doc.rust-lang.org/std/fmt/struct.Formatter.html). Beyond being a sink, it also exposes information on the specifier for the formatting operation via `current_spec()`, and helpers for printing indented lists and blocks.

`BestFmtQuery` is a related FTADLE that is called to determine what the valid format specifiers for this type are. This allows the format validator to reject formats that a type does not support, such as formatting a `best::str` with `{:x}`.

`best::format` returns (or appends to) a `best::strbuf`; `best::println` and `best::eprintln` can be used to write to stdout and stderr.

### [Reflection](\#reflection)

Within the metaprogramming library, `//best/meta:reflect` offers a basic form of reflection. It’s not C++26 reflection, because that’s wholely overkill. Instead, it provides a method for introspecting the members of structs and enums.

For example, suppose that we want to have a default way of formatting arbitrary [aggregate](https://en.cppreference.com/w/cpp/language/aggregate_initialization.html) structs. The code for doing this is actually devilishly simple:

```cpp highlight
void BestFmt(auto& fmt, const best::is_reflected_struct auto& value) {
  // Reflect the type of the struct.
  auto refl = best::reflect<decltype(value)>;
  // Start formatting a "record" (key-value pairs).
  auto rec = fmt.record(refl.name());

  // For each field in the struct...
  refl.each([&](auto field) {
    // Add a field to the formatting record...
    rec.field(
      field.name(),   // ...whose name is the field...
      value->*field,  // ...and with the appropriate value.
    );
  });
}

```

[C++](#code:12)

`best::reflect` provides access to the fields (or enum variants) of a user-defined type that opts itself in by providing the `BestReflect` FTADLE, which tells the reflection framework what the fields are. The simplest version of this FTADLE looks like this:

```cpp highlight
friend constexpr auto BestReflect(auto& mirror, MyStruct*) {
  return mirror.infer();
}

```

[C++](#code:13)

`best::mirror` is essentially a “reflection builder” that offers fine-grained control over what reflection actually shows of a struct. This allows for hiding fields, or attaching *tags* to specific fields, which generic functions can then introspect using `best::reflected_field::tags()`.

The functions on `best::reflected_type` allow iterating over and searching for specific fields (or enum variants); these `best::reflected_field` s provide metadata about a field (such as its name) and allow accessing it, with the same syntax as a pointer-to-member: `value->*field`.

Explaining the full breadth (and implementation tricks) of `best::reflect` would be a post of its own, so I’ll leave it at that.

### [Unit Tests and Apps](\#unit-tests-and-apps)

`best` provides a unit testing framework under `//best/test`, like any good standard library should. To define a test, you define a special kind of global variable:

```cpp highlight
best::test MyTest = [](best::test& t) {
  // Test code.
};

```

[C++](#code:14)

This is very similar to a Go unit test, which defines a function that starts with `Test` and takes a `*testing.T` as its argument. The `best::test&` value offers test assertions and test failures. Through the power of looking at debuginfo, we can extract the name `MyTest` from the binary, and use that as the name of the test directly.

That’s right, this is a C++ test framework with *no macros at all*!

Meanwhile, at `//best/cli` we can find a robust CLI parsing library, in the spirit of [`#[derive(clap::Parser)]`](https://docs.rs/clap/latest/clap/_derive/index.html) and other similar Rust libraries. The way it works is you first define a reflectable struct, whose fields correspond to CLI flags. A very basic example of this can be found in `test.h`, since test binaries define their own flags:

```cpp highlight
struct test::flags final {
  best::vec<best::strbuf> skip;
  best::vec<best::strbuf> filters;

  constexpr friend auto BestReflect(auto& m, flags*) {
    return m.infer()
      .with(best::cli::app{.about = "a best unit test binary"})
      .with(&flags::skip,
            best::cli::flag{
              .arg = "FILTER",
              .help = "Skip tests whose names contain FILTER",
            })
      .with(&flags::filters,
            best::cli::positional{
              .name = "FILTERS",
              .help = "Include only tests whose names contain FILTER",
            });
  }
};

```

[C++](#code:15)

Using `best::mirror::with`, we can apply tags to the individual fields that describe how they should be parsed and displayed as CLI flags. A more complicated, full-featured example can be found at [`toy_flags.h`](https://github.com/mcy/best/blob/main/best/cli/toy_flags.h), which exercises most of the CLI parser’s features.

`best::parse_flags<MyFlags>(...)` can be used to parse a particular flag struct from program inputs, independent of the actual `argv` of the program. A `best::cli` contains the actual parser metadata, but this is not generally user-accessible; it is constructed automatically using reflection.

Streamlining top-level app execution can be done using `best::app`, which fully replaces the `main()` function. Defining an app is very similar to defining a test:

```cpp highlight
best::app MyApp = [](MyFlags& flags) {
  // Do something cool!
};

```

[C++](#code:16)

This will automatically record the program inputs, run the flag parser for `MyFlags` (printing `--help` and existing, when requested), and then call the body of the lambda.

The lambda can either return `void`, an `int` (as an exit code) or even a `best::result`, like Rust. `best::app` is also where the `argv` of the program can be requested by other parts of the program.

## [What’s Next?](\#whats-next)

There’s still a lot of stuff I want to add to `best`. There’s no synchronization primitives, neither atomics nor locks or channels. There’s no I/O; I have a work-in-progress PR to add `best::path` and `best::file`. I’d like to write my own math library, `best::rc` (reference-counting), and portable SIMD. There’s also some other OS APIs I want to build, such as signals and subprocesses. I want to add a robust PRNG, time APIs, networking, and stack symbolization.

Building the best C++ library is a lot of work, not the least because C++ is a very tricky language and writing exhaustive tests is tedious. But it manages to make C++ fun for me again!

I would love to see contributions some day. I don’t expect anyone to actually use this, but to me, it proves C++ could be so much better.

---

1. They are also [terrible people](https://patricia.no/2022/03/08/cppcon.html). [↩︎](#fnref:1)

2. I will grant that JeanHeyd has made significant process where many people believed was impossible. He appears to have the indomitable willpower of a shōnen protagonist. [↩︎](#fnref:2)

3. I have heard an apocryphal story that the namespace was going to be `abc` or `abcl`, because it was “Alphabet’s library”. This name was ultimately shot down by the office of the CEO, or so the legend goes. [↩︎](#fnref:3)

4. This may get renamed to `best::interval` or even `best::range` We’ll see! [↩︎](#fnref:4)

5. The fourth time I’ve written one in my career, lmao. I also wrote a [C implementation](https://github.com/google/cwisstable) at one point. My friend Matt has an [excellent introduction](https://www.youtube.com/watch?v=ncHmEUmJZf4) to the Swisstable data structure. [↩︎](#fnref:5)
