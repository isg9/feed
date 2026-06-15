---
title: Who Builds the Builder?
url: https://matklad.github.io/2020/08/12/who-builds-the-builder.html
published: "2020-08-12T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2020/08/12/who-builds-the-builder.html
---

# Who Builds the Builder?

Aug 12, 2020

This is a short note on the builder pattern, or, rather, on the builder *method* pattern.

TL;DR: if you have `Foo` and `FooBuilder`, consider adding a `builder` method to `Foo`:

```
struct Foo { ... }

#[derive(Default)]
struct FooBuilder { ... }

impl Foo {
    fn builder() -> FooBuilder {
        FooBuilder::default()
    }
}

impl FooBuilder {
    fn build(self) -> Foo { ... }
}
```

A more minimal solution is to rely just on `FooBuilder::default` or `FooBuilder::new`. There are two problems with that:

*First*, it is hard to discover. Nothing in the docs/signature of `Foo` mentions `FooBuilder`, you need to look elsewhere to learn how to create a `Foo`. I remember being puzzled at how to create a [`GlobSet`](https://docs.rs/globset/0.4.5/globset/struct.GlobSet.html) for exactly this reason. In contrast, the `builder` method is right there on `Foo`, probably the first one.

*Second*, it is more annoying to use, as you need to import *both* `Foo` and `FooBuilder`. With `Foo::builder` method often only one import suffices, as you don’t need to name the builder type.

Case studies:

- [`TextEdit::build`](https://github.com/rust-analyzer/rust-analyzer/commit/7510048ec0a5d5e7136e3ea258954eb244d15baf) from rust-analyzer.

- [`File::with_options`](https://doc.rust-lang.org/std/fs/struct.File.html#method.with_options) from std.

- [`Request`](https://docs.rs/http/0.2.1/http/request/struct.Request.html#method.builder) and [`Response`](https://docs.rs/http/0.2.1/http/response/struct.Response.html#method.builder) from http.

Discussion on [/r/rust](https://www.reddit.com/r/rust/comments/i8kofk/blog_post_who_builds_the_builder/).
