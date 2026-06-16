---
title: Single Abstract Method Traits
url: /2023/05/11/sam-closures
published: "2023-05-11T00:00:00Z"
feed: mcyoung
guid: /2023/05/11/sam-closures
---

# Single Abstract Method Traits

Rust and C++ both have very similar operational semantics for their “anonymous function” expressions (they call them “closures” and “lambdas” respectively; I will use these interchangably). Here’s what those expressions look like.

```cpp highlight
auto square = [](int x) { return x * x; }

```

[C++](#code:1)

```rust highlight
let square = |x: i32| x * x;

```

[Rust](#code:2)

The type of `square` in both versions is an anonymous type that holds the captures for that closure. In C++, this type provides an `operator()` member that can be used to call it, wheras in Rust, it implements `FnOnce` (and possibly `FnMut` and `Fn`, depending on the captures), which represent a “callable” object.

> For the purposes of this article, I am going to regard “function item values” as being identical to closures that explicitly specify their inputs and outputs for all intents and purposes. This is not completely accurate, because when I write `let x = drop;`, the resulting object is generic, but whenever I say “a closure” in Rust, I am also including these closure-like types too.

There is one thing C++ closures can express which Rust closures can’t: you can’t create a “generic” closure in Rust. In particular, in C++ we can write this code.

```cpp highlight
template <typename Fn>
size_t CallMany(Fn fn) {
  return fn(std::vector{5}) + fn(std::string("foo"));
}

CallMany([](auto& val) { return val.size(); });

```

[C++](#code:3)

The `auto` keyword in a closure in C++ does not work like in Rust. In Rust, if try to write “equivalent” code, `let x = |val| val.len();`, on its own, we get this error:

```rust highlight
error[E0282]: type annotations needed
 --> <source>:4:12
  |
4 |   let x = |val| val.len();
  |            ^^^  --- type must be known at this point
  |
help: consider giving this closure parameter an explicit type
  |
4 |   let x = |val: /* Type */| val.len();
  |               ++++++++++++

```

[Rust](#code:4)

This is because in Rust, a closure argument without a type annotation means “please deduce what this should be”, so it participates in Rust’s type inference, wheras in C++ an `auto` argument means “make this a template parameter of `operator()`”.

How would we implement `CallMany` in Rust, anyways? We could try but we quickly hit a problem:

```rust highlight
trait Len {
  fn len(&self) -> usize;
}

fn call_many(f: impl Fn(???) -> usize) {
  f(vec![5]) + f("foo")
}

```

[Rust](#code:5)

What should we put in the `???`? It can’t be a type parameter of `call_many`, since that has a concrete value in the body of the function. We want to say that `Fn` can accept *any* argument that implements `len`. There isn’t even syntax to describe this, but you could imagine adding a version of `for<...>` that works on types, and write something like this.

```rust highlight
trait Len {
  fn len(&self) -> usize;
}

fn call_many(f: impl for<T: Len> Fn(&T) -> usize) -> usize {
  f(vec![5]) + f("foo")
}

```

[Rust](#code:6)

The imaginary syntax `for<T: Len> Fn(&T) -> usize` means “implements `Fn` for all *all* types `T` that implement `Len`”. This is a pretty intense thing to ask rustc to prove. It is not unachievable, but it would be hard to implement.

> [note](#note:1)
>
> For the purposes of this article, I am going to consider `for<T>` a plausible, if unlikely, language feature. I will neither assume it will ever happen, nor that we should give up on ever having it. This “middle of uncertainty” is important to ensure that we do not *make* adding this feature impossible in the discussion that follows.

## [A Workaround](\#a-workaround)

Let’s examine the `Fn` trait, greatly simplified.

```rust highlight
pub trait Fn<Args> {
  type Output;
  fn call(&self, args: Args) -> Self::Output;
}

```

[Rust](#code:7)

`Fn::call` is analogous to `operator()` in C++. When we say that we want a “generic closure”, we mean that we want to instead have a trait that looks a bit more like this:

```rust highlight
pub trait Fn {
  type Output<Args>;
  fn call<Args>(&self, args: Args) -> Self::Output<Args>;
}

```

[Rust](#code:8)

Notice how `Args` has moved from being a trait parameter to being a function parameter, and `Output` now depends on it. This is a slightly different formulation from what we described above, because we are no longer demanding an infinitude of trait implementations, but now the implementation of one trait with a generic method.

For our specific example, we want something like this.

```rust highlight
trait Len {
  fn len(&self) -> usize;
}

trait Callback {
  fn run(&self, val: impl Len) -> usize;
}

fn call_many(f: impl Callback) -> usize {
  f.run(vec![5]) + f.run("foo")
}

```

[Rust](#code:9)

This compiles and expresses what we want precisely: we want to call `f` on arbitrary `impl Len` types.

But how do we call `call_many`? That starts to get pretty ugly.

```rust highlight
struct CbImpl;
impl Callback for CbImpl {
  fn run(&self, val: impl Len) -> usize {
    val.len()
  }
}

call_many(CbImpl);

```

[Rust](#code:10)

This has the potential to get really, really ugly. I used this pattern for a non-allocating visitor I wrote recently, and it wasn’t pretty. I had to write a macro to cut down on the boilerplate.

```rust highlight
macro_rules! resume_by {
  ($parser:expr, $cb:expr) => {{
    struct Cb<'a, 's> {
      parser: &'a Parser<'s>,
      start: Option<u32>,
    }

    impl<'s> Resume<'s> for Cb<'_, 's> {
      fn resume(
        &mut self,
        visitor: &mut impl Visitor<'s>,
      ) -> Result<(), Error> {
        self.parser.do_with_rewind(
          &mut self.start,
          || ($cb)(self.parser, &mut *visitor),
        )
      }
    }

    Cb { parser: $parser, start: None }
  }};
}

```

[Rust](#code:11)

This macro is, unsurprisingly, quite janky. It also can’t really do captures, because the `$cb` argument that contains the actual code is buried inside of a nested `impl`.

You might think “well Sunny, why don’t you hoist `$cb` into the `Cb` struct?” The problem is now that I need to write `impl<'s, F: FnMut(&Parser<'s>, ???)>` so that I can actually call the callback in the body of `Resume::resume`, but that brings us back to our trait bound problem from the start!

This is a general problem with this type of solution: there is no macro you can write that will capture an arbitrary closure to implement a trait by calling that closure, if the method being implemented is generic, because if you *could*, I wouldn’t have to bother with the macro.

## [Let’s Talk About Java](\#lets-talk-about-java)

Java gets a bad rap but the core language does have some interesting features in it. A very handy one is an *anonymous class*.

Let’s suppose I want to pass a callback into something. In Java 6, which I grew up on, you did it like this:

```java highlight
public interface Callback {
  int run(int arg);
}

public int runMyThing(Callback cb) {
  return cb.run(42);
}

runMyThing(new Callback() {
  public int run(int arg) { return arg * arg; }
});

```

[Java](#code:12)

The `new Interface() {...}` syntax mints a new class on the spot that implements `Interface`. You provide a standard class body between the braces, after the name of the type. You can also do this with a class type too.

Now, this is a bit tedious: I need to re-type the signature of the one method. This is fine if I need to implement a bunch of methods, but it’s a little annoying in the one-method case.

In Java 8 we got lambdas (syntax: `x -> expr`). Java made the interesting choice of not adding a `Function` type to be “the type of lambdas”. For a long time I thought this was a weird cop-out but I have since come to regard it as a masterclass in language design.

Instead, Java’s lambdas are a sort of syntax sugar over this anonymous class syntax.[1](#fn:1) Instead, you need to assign a lambda to an interface type with a single abstract method, and it will use the body of the lambda to implement that one method.

Interfaces compatible with lambdas are called single abstract method (SAM) interfaces.

So, without needing to touch the existing library, I can turn the `new` syntax into this:

```java highlight
runMyThing(x -> x * x);

```

[Java](#code:13)

*chef’s kiss*

Mind, Java *does* provide a mess of “standard function interfaces” in the `java.util.functional` package, and quite a bit of the standard library uses them, but they don’t need to express the totality of functions you might want to capture as objects.

These “SAM closures” give closures a powerful “BYO interface” aspect. Lambdas in Java are not “function objects”, they are extremely lightweight anonymous classes the pertinent interface.

I think this can let us cut the gordian knot of generic closures in Rust.

## [SAM in Rust](\#sam-in-rust)

In what remains I will propose how we can extend the traits that closures implement to be *any* SAM trait, in addition to the traits they implement ipso facto.

What’s a SAM trait in Rust? It’s any trait `T` with precisely ONE method that does not have a default implementation, which must satisfy the following constraints:

1. It must have a `self` parameter with type `Self`, `&Self`, or `&mut Self`.
2. It does not mention `Self` in any part of its argument types, its return type, or its `where` clauses, except for the aforementioned `self` parameter.
3. Has no associated consts and no GATs.
4. All of its supertraits are `Copy`, `Send`, or `Sync`.

These restrictions are chosen so that we have a shot at actually implementing the entire trait.

> In addition to the `Fn` traits, ordinary closures automatically implement `Clone`, `Copy`, `Send`, and `Sync` as appropriate.
>
> None of these traits are SAM, so we can safely allow them to be automatically derived for SAM closures to, under the same rules as for ordinary closures.

To request a SAM closure, I will use the tentative syntax of `impl Trait |args| expr`. This syntax is unambiguously an expression rather than an `impl` item, because a `|` cannot appear in a path-in-type, and `impl $path` must be followed by `{`, `for` or `where`. The precise syntax is unimportant.

Applied to the `call_many` example above, we get this.

```rust highlight
fn call_many(f: impl Callback) -> usize {
  f.run(vec![5]) + f.run("foo")
}

call_many(impl Callback |x| x.len());

```

[Rust](#code:14)

The compiler rewrites this into something like this.

```rust highlight

fn call_many(f: impl Callback) -> usize {
  f.run(vec![5]) + f.run("foo")
}

struct CallbackImpl;
impl Callback for CallbackImpl {
  fn run(&self, x: impl Len) -> usize {
    x.len()
  }
}

call_many(CallbackImpl);

```

[Rust](#code:15)

This rewrite can happen relatively early, before we need to infer a type for `x`. We also need to verify that this trait’s captures are compatible with an `&self` receiver The same rules for when a trait implements `Fn`, `FnMut`, and `FnOnce` would decide which of the three receiver types the closure is compatible with.

Note that SAM closures WOULD NOT implement any `Fn` traits.

### [More Complicated Examples](\#more-complicated-examples)

We are required to name the trait we want but its type parameters can be left up in the air. For example:

```rust highlight
pub trait Tr<T> {
  type Out;
  fn run(x: T, y: impl Display) -> Option<Self::Out>;
}

// We can infer `T = i32` and `Tr::Out = String`.
let tr = impl Tr<_> |x: i32, y| Some(format!("{}, {}"));

```

[Rust](#code:16)

In general, unspecified parameters and associated types result in inference variables, which are resolved in the same way as the parameters of the `Fn` closures are.

In fact, we can emulate ordinary closures using SAM closures.

```rust highlight
let k = 100;
let x = Some(42);
let y = x.map(impl FnOnce(_) -> _ move |x| x * k);

```

[Rust](#code:17)

Note that because `Fn` and `FnMut` have non-trival supertraits we can’t make them out of SAM closures.

One application is to completely obsolete `std::iter::from_fn`.

```rust highlight
fn fibonacci() -> impl Iterator<Item = u64> + Copy {
  let state = [1, 1];
  impl Iterator move || {
    let old = state[0];

    state[0] = state[1];
    state[1] += ret;

    ret
  }
}

```

[Rust](#code:18)

Or, if you need a quick helper implementation of `Debug`…

```rust highlight
impl fmt::Debug for Thing {
  fn fmt(&self, f: fmt::Formatter) -> fmt::Result {
    f.debug_list()
    .entry(&impl Debug |f| {
      write!(f, "something something {}", self.next_thingy())
    })
    .finish();
  }
}

```

[Rust](#code:19)

There are probably additional restrictions we will want to place on the SAM trait, but it’s not immediately clear what the breadth of those are. For example, we probably shouldn’t try to make this work:

```rust highlight
trait UniversalFactory {
  fn make<T>() -> T;
}

let f = impl UniversalFactory || {
  // How do I name T so that I can pass it to size_of?
};

```

[Rust](#code:20)

There are definitely clever tricks you *can* play to make this work, but the benefit seems slim.

## [Future Work](\#future-work)

There’s two avenues for how we could extend this concept. The first is straightforward and desireable; the second is probably unimplementable.

### [Anonymous Trait Impls](\#anonymous-trait-impls)

Backing up from the Java equivalent of lambdas, it seems not unreasonable to have a full-fledged expression version of `impl` that can make captures.

Syntactically, I will use `impl Trait for { ... }`. This is currently unambiguous, although I think that making it so that `{` cannot start a type is probably a non-starter.

Let’s pick something mildly complicated… like `Iterator` with an overriden method. Then we might write something like this.

```rust highlight
let my_list = &foo[...];
let mut my_iterator = impl Iterator for {
  type Item = i32;

  fn next(&mut self) -> Option<i32> {
    let head = *my_list.get(0)?;
    my_list = &my_list[1..];
    Some(head)
  }

  fn count(self) -> usize {
    my_list.len();
  }
};

```

[Rust](#code:21)

The contents of the braces after `for` is an item list, except that variables from the outside are available, having the semantics of captures; they are, in effect, accesses of `self` without the `self.` prefix.

Hammering out precisely how this would interact with the self types of the functions in the body seems… complicated. Pretty doable, just fussy. There are also awkward questions about what `Self` is here and to what degree you’re allowed to interact with it.

### [Trait Inference](\#trait-inference)

Suppose that we could instead “just” write `impl |x| x * x` and have the compiler figure out what trait we want (to say nothing of making this the default behavior and dropping the leading `impl` keyword).

This means that I could just write

```rust highlight
fn call_many(f: impl Callback) -> usize {
  f.run(vec![5]) + f.run("foo")
}

call_many(impl |x| x.len());

```

[Rust](#code:22)

We get into trouble fast.

```rust highlight
trait T1 {
  fn foo(&self);
}

trait T2 {
  fn foo(&self);
}

impl<T: T1> T2 for T {
  fn foo(&self) {
    println!("not actually gonna call T1::foo() lmao");
  }
}

let x = || println!("hello");
T2::foo(&x);  // What should this print?

```

[Rust](#code:23)

If the type of `x` implements `T2` directly, we print `"hello"`, but if we decide it implements `T1` instead, it doesn’t, because we get the blanket impl. If it decides it should implement both… we get a coherence violation.

Currently, rustc does not have to produce impls “on demand”; the trait solver has a finite set of impls to look at. What we are asking the trait solver to do is to, for certain types, attempt to reify impls based on *usage*. I.e., I have my opaque closure type `T` and I the compiler decided it needed to prove a `T: Foo` bound so now it gets to perform type checking to validate whether it has an `impl`.

This seems unimplementable with how the solver currently works. It is not insurmountable! But it would be very hard.

It is possible that there are relaxations of this that are not insane to implement, e.g. the `impl ||` expression is used to initialize an argument to a function that happens to be generic, so we can steal the bounds off of that type variable and hope it’s SAM. But realistically, this direction is more trouble than its worth.

## [Conclusion](\#conclusion)

Generic lambdas are extremely powerful in C++, and allow for very slick API designs; I often miss them in Rust. Although it feels like there is an insurmountable obstruction, I hope that the SAM interface approach offers a simpler, and possibly more pragmatic, approach to making them work in Rust.

---

1. Except for the part where they are extremely not. Where `new T() {}` mints a brand new class and accompanying `.class` file, Java lambdas use this complicated machinery from Java 7 to generate method handles on the fly, via the `invokedynamic` JVM instruction. This, I’m told, makes them much easier to optimize. [↩︎](#fnref:1)
