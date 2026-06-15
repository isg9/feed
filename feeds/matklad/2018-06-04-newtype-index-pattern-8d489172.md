---
title: Newtype Index Pattern
url: https://matklad.github.io/2018/06/04/newtype-index-pattern.html
published: "2018-06-04T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2018/06/04/newtype-index-pattern.html
---

# Newtype Index Pattern

Jun 4, 2018

Similarly to the [previous post](/2018/05/24/typed-key-pattern.html), we will once again add types to the Rust code which works perfectly fine without them. This time, we’ll try to improve the pervasive pattern of using indexes to manage cyclic data structures.

## [The problem](\#The-problem)

Often one wants to work with a data structure which contains a cycle of some form: object `foo` references `bar`, which references `baz` which references `foo` again. The textbook example here is a graph of vertices and edges. In practice, however, true graphs are a rare encounter. Instead, you are more likely to see a tree with parent pointers, which contains a lot of trivial cycles. And sometimes cyclic graphs are implicit: an `Employee` can be the head of a `Department`, and `Department` has a `Vec` personal. This is sort-of a graph in disguise: in usual graphs, all vertices are of the same type, and here `Employee` and `Department` are different types.

Working with such data structures is hard in any language. To arrive at a situation when `A` points to `B` which points back to `A`, some form of mutability is required. Indeed, either `A` or `B` must be created first, and so it can not point to the other immediately after construction. You can paper over this mutability with `let rec`, as in OCaml, or with laziness, as in Haskell, but it is still there.

Rust tends to surface subtle problems in the form of compile-time errors, so implementing such graphs in Rust is challenging. The three usual approaches are:

- reference counting, explanation by [nrc](https://github.com/nrc/r4cppp/blob/master/graphs/README.md#rcrefcellnode),

- arena and real cyclic references, explanation by [simonsapin](https://exyr.org/2018/rust-arenas-vs-dropck/) (this one is really neat!),

- arena and integer indices, explanation by [nikomatsakis](http://smallcultfollowing.com/babysteps/blog/2015/04/06/modeling-graphs-in-rust-using-vector-indices/).

(apparently, rewriting a Haskell monad tutorial in Rust results in a graphs blog post).

I personally like the indexing approach the most. However it presents an interesting readability challenge. With references, you have a `foo` of type `&Foo`, and it is immediately clear what that `foo` is, and what you can do with it. With indexes, however, you have a `foo: usize`, and it is not obvious that you somehow can get a `Foo`. Even worse, if indexes are used for two types of objects, like `Foo` and `Bar`, you may end up with `thing: usize`. While writing the code with `usize` actually works pretty well (I don’t think I’ve ever used the wrong index type), reading it later is more complicated, because `usize` is much less suggestive of what you could do.

## [Newtype trick](\#Newtype-trick)

One way to ameliorate this problem is to introduce a newtype wrapper around `usize`:

```
struct Foo;

#[derive(Debug, Copy, Clone, Ord, PartialOrd, Eq, PartialEq, Hash)]
struct FooIdx(usize);

struct Arena {
    foos: Vec,
}

impl Arena {
    fn foo(&self, foo: FooIdx) -> &Foo {
        &self.foos[foo.0]
    }
}
```

Here, “one should use `FooIdx` to index into `Vec`” is still just a convention. A cool thing about Rust is that we can turn this convention into a property verified during type checking. By adding an appropriate impl, we should be able to index into `Vec` with `FooIdx` directly:

```
#[test]
fn direct_indexing(foos: Vec, idx: FooIdx) {
    let _foo: &Foo = &foos[idx];
}
```

The impl would look like this:

```
use std::ops;

impl ops::Index for Vec {
    type Output = Foo;

    fn index(&self, index: FooIdx) -> &Foo {
        &self[index.0]
    }
}
```

## [Coherence](\#Coherence)

It’s insightful to study why this impl is allowed. In Rust, types, traits and impls are separate. This creates a room for a problem: what if there are two impl blocks for a given (trait, type) pair? The obvious choice is to forbid to have two impls in the first place, and this is what Rust does.

Actually enforcing this restriction is tricky! The simplest rule of “error if a set of crates currently compiled contains duplicate impls” has severe drawbacks. First of all, this is a global check, which requires the knowledge of all compiled crates. This postpones the check until the later stages of compilation. It also plays awfully with dependencies, because two completely unrelated crates might fail the compilation if present simultaneously. What’s more, it doesn’t actually solve the problem, because the compiler does not necessary know the set of all crates beforehand. For example, you may load additional code at runtime via dynamic libraries, and silent bad things might happen if you program and dynamic library have duplicate impls.

To be able to combine crates freely, we want a much stronger property: not only the set of crates currently compiled, but all existing and even future crates must not violate the one impl restriction. How on earth is it possible to check this? Should `cargo publish` look for conflicting impls across all of the crates.io?

Luckily, and this is stunningly beautiful, it is possible to loosen this world-global property to a local one. In the simplest form, we can place a restriction that `impl Foo for Bar` can appear either in the crate that defines `Foo`, or in the one that defines `Bar`. Crucially, whichever one defines the impl has to use the other, which makes it possible to detect the conflict.

This is all really nifty, but we’ve just defined an `Index` impl for `Vec`, and both `Index` and `Vec` are from the standard library! How is it possible? The trick is that `Index` has a type parameter: `trait Index`. It is a template for a trait of sorts, and we get a “real” trait when we substitute type parameter with a type. Because `FooIdx` is a local type, the resulting `Index` trait is also considered local. The precise rules here are quite tricky, [this RFC](https://github.com/rust-lang/rfcs/pull/2451) explains them pretty well.

## [More impls](\#More-impls)

Because `Index` and `Index` are different traits, one type can implement both of them. This is convenient for containers which hold distinct types:

```
struct Arena {
    foos: Vec,
    bars: Vec,
}

impl ops::Index for Arena { ... }

impl ops::Index for Arena { ... }
```

It’s also helpful to define arithmetic operations and conversions for the newtyped indexes. I’ve put together a [`typed_index_derive`](https://crates.io/crates/typed_index_derive) crate to automate this boilerplate via a proc macro, the end result looks like this:

```
#[macro_use]
extern crate typed_index_derive;

struct Spam(String);

#[derive(
    // Usual derives for plain old data
    Debug, Copy, Clone, Ord, PartialOrd, Eq, PartialEq, Hash,

    TypedIndex
)]
#[typed_index(Spam)] // index into `&[Spam]`
struct SpamIdx(usize); // could be `u32` instead of `usize`

fn main() {
    let spams = vec![Spam("foo".into()), Spam("bar".into()), Spam("baz".into())];

    // Conversions between `usize` and `SpamIdx`
    let idx: SpamIdx = 1.into();
    assert_eq!(usize::from(idx), 1);

    // Indexing `Vec` with `SpamIdx`, `IndexMut` works as well
    assert_eq!(&spams[idx].0, "bar");

    // Indexing `Vec` is rightfully forbidden
    // vec![1, 2, 3][idx]
    // error: slice indices are of type `usize` or ranges of `usize`

    // It is possible to  add/subtract `usize` from an index
    assert_eq!(&spams[idx - 1].0, "foo");

    // The difference between two indices is `usize`
    assert_eq!(idx - idx, 0usize);
}
```

Discussion on [/r/rust](https://www.reddit.com/r/rust/comments/8ohaj4/blog_post_newtype_index_pattern/).
