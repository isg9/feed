---
title: Better Trait Resolution in Rust
url: /2023/04/04/trait-rez-wishlist
published: "2023-04-04T00:00:00Z"
feed: mcyoung
guid: /2023/04/04/trait-rez-wishlist
---

# Better Trait Resolution in Rust

Traits are the core of polymorphism in Rust. Let’s review:

```rust highlight
trait Stringer {
  fn string(&self) -> String;
}

impl Stringer for i32 {
  fn string(&self) -> String {
    format!("{self}")
  }
}

// Prints `42`.
println!("{}", 42.string());

```

[godbolt](https://godbolt.org/clientstate/eyJzZXNzaW9ucyI6W3siZXhlY3V0b3JzIjpbeyJjb21waWxlciI6eyJpZCI6ImJldGEiLCJvcHRpb25zIjoiIn0sImNvbXBpbGVyT3V0cHV0VmlzaWJsZSI6dHJ1ZSwiY29tcGlsZXJWaXNpYmxlIjpmYWxzZX1dLCJpZCI6MSwibGFuZ3VhZ2UiOiJydXN0Iiwic291cmNlIjoicHViIGZuIG1haW4oKXtcbnRyYWl0IFN0cmluZ2VyIHtcbiAgZm4gc3RyaW5nKFx1MDAyNnNlbGYpIC1cdTAwM2UgU3RyaW5nO1xufVxuXG5pbXBsIFN0cmluZ2VyIGZvciBpMzIge1xuICBmbiBzdHJpbmcoXHUwMDI2c2VsZikgLVx1MDAzZSBTdHJpbmcge1xuICAgIGZvcm1hdCEoXCJ7c2VsZn1cIilcbiAgfVxufVxuXG4vLyBQcmludHMgYDQyYC5cbnByaW50bG4hKFwie31cIiwgNDIuc3RyaW5nKCkpO1xuXG59XG4ifV19) [Rust](#code:1)

Notice that we call the trait method `Stringer::string` directly on the value in question. This means that traits (at least, those currently in scope) inject their methods into the namespace of anything that implements them.

Now, this isn’t immediately a problem, because Rust’s namespace lookup rules are such that methods *inherent* to a type are searched for first:

```rust highlight
trait Stringer {
  fn string(&self) -> String;
}

struct Woofer;
impl Stringer for Woofer {
  fn string(&self) -> String {
    format!("woof")
  }
}

impl Woofer {
  fn string(&self) -> String {
    format!("bark")
  }
}

// Prints `bark`.
println!("{}", Woofer.string());

```

[godbolt](https://godbolt.org/clientstate/eyJzZXNzaW9ucyI6W3siZXhlY3V0b3JzIjpbeyJjb21waWxlciI6eyJpZCI6ImJldGEiLCJvcHRpb25zIjoiIn0sImNvbXBpbGVyT3V0cHV0VmlzaWJsZSI6dHJ1ZSwiY29tcGlsZXJWaXNpYmxlIjpmYWxzZX1dLCJpZCI6MSwibGFuZ3VhZ2UiOiJydXN0Iiwic291cmNlIjoicHViIGZuIG1haW4oKXtcbnRyYWl0IFN0cmluZ2VyIHtcbiAgZm4gc3RyaW5nKFx1MDAyNnNlbGYpIC1cdTAwM2UgU3RyaW5nO1xufVxuXG5zdHJ1Y3QgV29vZmVyO1xuaW1wbCBTdHJpbmdlciBmb3IgV29vZmVyIHtcbiAgZm4gc3RyaW5nKFx1MDAyNnNlbGYpIC1cdTAwM2UgU3RyaW5nIHtcbiAgICBmb3JtYXQhKFwid29vZlwiKVxuICB9XG59XG5cbmltcGwgV29vZmVyIHtcbiAgZm4gc3RyaW5nKFx1MDAyNnNlbGYpIC1cdTAwM2UgU3RyaW5nIHtcbiAgICBmb3JtYXQhKFwiYmFya1wiKVxuICB9XG59XG5cbi8vIFByaW50cyBgYmFya2AuXG5wcmludGxuIShcInt9XCIsIFdvb2Zlci5zdHJpbmcoKSk7XG5cbn1cbiJ9XX0=) [Rust](#code:2)

This means that traits cannot *easily* break downstream code by adding new methods, but there are a few possible hazards:

- If the owner of a type adds a method with the same name as a trait method, it will override direct (i.e., `foo.string()`) calls to that trait method, even if the type owner is unaware of the trait method.

- If traits `A` and `B` are in scope, and `String` implements both, and we call `str.foo()` (which resolves to `A::foo()`), and later `B` adds a new method `B::foo()`, the callsite for `String` will break. `A` and `B`’s owners do not need to be aware of each other for this to happen.

Of course, Rust has a disambiguation mechanism. Given any trait implementation `Foo: Bar`, we can reference its items by writing `<Foo as Bar>::baz`. However, this syntax is *very* unweildy (it doesn’t work with method chaining), so it doesn’t get used. As a result, small evolution hazards can build up in a large codebase.

Those who know me know that I often talk about a syntax that I call `foo.Trait::method()`, or “qualified method call syntax”. In this post, I want to discuss this syntax in more detail, and some related ideas, and how they factor into type and trait design.

## [Paths-as-Methods](\#paths-as-methods)

This idea isn’t new; others have proposed it, and it forms the core of Carbon’s version of trait method calls (you can read more about Carbon’s name lookup story [here](https://github.com/carbon-language/carbon-lang/blob/trunk/docs/design/expressions/member_access.md)).

Let’s recreate the original example in Carbon (bear in mind that I am not an expert on this language, and the semantics are still up in the air).

```carbon highlight
interface Stringer {
  fn String[self: Self]() -> String;
}

external impl i32 as Stringer {
  fn String[self: Self]() -> String { ... }
}

fn Run() {
  Carbon.Print(42.(Stringer.String)())
}

```

[Carbon](#code:3)

Notice `42.(Stringer.String)()`: Carbon *requires* that we qualify the method call, because `42` has the concrete type `i32`. If this were in a generic context and we had a type variable bounded by `Stringer`, we could just write `x.String()`; no ambiguity.

In Carbon, all qualification uses `.`, so they have to add parens. Because Rust uses `::` for qualifying paths, we don’t have this syntactic abiguity, so we can augment the syntax to allow more path expressions after the `.`.

The current grammar is

```highlight
MethodCallExpression :
   Expression . PathExprSegment (CallParams?)

PathExprSegment :
   PathIdentSegment (:: GenericArgs)?

```

[Plaintext](#code:4)

That is, exactly one identifier and an optional turbofish. I would like to see this extended to allow any `QualifiedPathInExpression` after the `.` and before the parens. This would allow, for example:

- `expr.Trait::method()`
- `expr.Self::method()`
- `expr.<Foo as Trait>::method()`
- `expr.::path::to::Trait::<Args>::method::<MoreArgs>()`

These would all be desugared as the equivalent UFCS, taking into account that method call syntax can trigger autoref.

- `Trait::method(expr)`
- `Self::method(expr)`
- `<Foo as Trait>::method(expr)`
- `::path::to::Trait::<Args>::method::<MoreArgs>(expr)`

The method would still need to be valid to have been called via `.` syntax; I’m not proposing we add the following, even though it is unambiguous.

```rust highlight
fn square(x: i32) -> i32 {
  x * x
}

// Would be equivalent to `square(42)`.
42.self::square()

```

[Rust](#code:5)

Trait method callers can now use qualified method call syntax where they might want to use UFCS without issues around wordiness.

## [Impl Modality](\#impl-modality)

Of course, this isn’t the only idea from Carbon’s interfaces worth stealing; Carbon also has a notion of “external” and “internal” `impl` s; I will call these “ `impl` modes”.

An external `impl` is like the one we showed above, whose methods can only be found by qualified lookup: `foo.(Bar.Baz)()`. An internal `impl` is one which is “part” of a type.

```carbon highlight
interface Stringer {
  fn String[self: Self]() -> String;
}

class Woofer {
  impl as Stringer {
    fn String[self: Self]() -> String { ... }
  }
}

fn Run() {
  let w: Woofer = ...;
  w.String()  // Unqualified!
}

```

[Carbon](#code:6)

This also implies that we don’t need to import `Stringer` to call `w.String()`.

There are definitely traits in Rust which fit into these modes.

`Clone` and `Iterator` almost always want to be internal. An iterator exists to implement `Iterator`, and cloning is a fundamental operation. Because both of these traits are in the prelude, it’s not a problem, but it is a problem for traits provided by a non- `std` crate, like [`rand::Rng`](https://docs.rs/rand/latest/rand/trait.Rng.html). The lack of a way to do this leads to the proliferation of `prelude` modules and namespace pollution. (I think that `prelude` s are bad library design.)

On the other hand, something like `Debug` wants to be external very badly. It almost never makes sense to call `foo.fmt()`, since that gets called for you by `println!` and friends; not to mention that all of the `std::fmt` traits have a method `fmt()`, making such a call likely to need disambiguation with UFCS. `Borrow` is similar; it exists to be a bound for things like `Cow` more than to provide the `.borrow()` method.

There’s also a third mode, which I will call “extension impls”. These want to inject methods into a type, either to extend it, like [`itertools`](https://docs.rs/itertools/latest/itertools/), or as part of some framework, like [`tap`](https://docs.rs/tap/latest/tap/). This use of traits is somewhat controversial, but I can sympathize with wanting to have this.

If we have paths-as-methods, we can use this classification to move towards something more like the Carbon model of method lookup, without impacting existing uses.

My strawman is to add a `#[mode]` attribute to place on trait `impls`, which allows a caller to select the behavior:

- `#[mode(extension)]` is today’s behavior. The `impl`’s trait must be in scope so that unqualified calls like `foo.method()` resolve to it.
- `#[mode(internal)]` makes it so that `foo.method()` can resolve to a method from this `impl` without its trait being in scope[1](#fn:1). It can only be applied to `impl` s that are such that you could write a corresponding inherent impl, so things like `#[mode(internal)] impl<T> Trait for T { .. }` are forbidden.
- `#[mode(external)]` makes it so that `foo.method()` never resolves to a method from this `impl`. It must be called as `Trait::method(foo)` or `foo.Trait::method()`.

Every trait would be `#[mode(extension)]` if not annotated, and it would be easy to migrate to external-by-default across an edition. Similarly, we could change whether a `std` impl is external vs extension based on the edition of the caller, and provide a `cargo fix` rewrite to convert from `foo.method()` to `foo.Trait::method()`.

It may also make sense for traits to be able to specify the default modality of their `impl` s, but I haven’t thought very carefully about this.

Note that moving in the external -> extension -> internal direction is not a breaking change, but moving the other way is.

## [What about Delegation?](\#what-about-delegation)

A related feature that I will touch on lightly is delegation; that is, being able to write something like the following:

```rust highlight
// crate a
struct Foo { ... }
impl Foo {
  fn boing() -> i32 { ... }
}

// crate b
trait Bar {
  fn boing() -> i32;
}

impl Bar for Foo {
  use Foo::boing;
}

```

[Rust](#code:7)

The `use` in the `impl` indicates that we want to re-use `Foo::boing` to implement `<Foo as Bar>::boing`. This saves us having to write out a function signature, and results in less work for the compiler because that’s one less function we risk asking LLVM to codegen for us (at scale, this is a Big Deal).

You could imagine using delegation instead of `#[mode]`:

```rust highlight
struct Woofer;
impl Stringer for Woofer {
  fn string(&self) -> String {
    format!("woof")
  }
}

impl Woofer {
  use Stringer::*;
}

```

[Rust](#code:8)

The reason I haven’t gone down this road is because delegation is a very large feature, and doesn’t give us a clean way to express `#[mode(external)]`, which is a big part of what I want. A delegation-compatible way to express this proposal is to not add `#[mode(internal)]`, and add `use Trait::method;` and `use Trait::*;` (and no other variations) inside of inherent `impl` blocks.

## [Conclusion](\#conclusion)

I don’t have the personal bandwidth to write RFCs for any of this stuff, but it’s something I talk about a lot as a potential evolution hazard for Rust. I hope that putting these ideas to paper can help make name resolution in Rust more robust.

---

1. This needs to carry a bunch of other restrictions, because it’s equivalent to adding inherent methods to the implee. For example, none of the methods can have the same name as a method in any other inherent or internal impl block, and internal impl methods should take lookup priority over extension impl methods during name lookup. [↩︎](#fnref:1)
