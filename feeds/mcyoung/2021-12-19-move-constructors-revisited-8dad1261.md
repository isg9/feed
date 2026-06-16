---
title: Move Constructors Revisited
url: /2021/12/19/move-ctors-2
published: "2021-12-19T00:00:00Z"
feed: mcyoung
guid: /2021/12/19/move-ctors-2
---

# Move Constructors Revisited

Almost a year ago I developed the [`moveit`](https://github.com/google/moveit) Rust library, which provides primitives for expressing something like C++’s `T&&` and move constructors while retaining Rust’s so-called “destructive move property”: moving a value transfers ownership, rather than doing a funny copy.

In an [earlier blogpost]({{site.baseurl}}/2021/04/26/move-ctors) I described the theory behind this library and some of the motivation, which I feel fairly confident about, especially in how constructors (and their use of pinning) are defined.

However, there is a problem.

## [A Not-so-Quick Recap](\#a-not-so-quick-recap)

The old post is somewhat outdated, since `moveit` uses different names for a lot of things that are geared to fit in with the rest of Rust.

The core abstraction of `moveit` is the constructor, which are types that implement the `New` trait:

```rust highlight
#[must_use]
pub unsafe trait New: Sized {
  /// The type to construct.
  type Output;

  /// Construct a new value in-place using the arguments stored
  /// in `self`.
  unsafe fn new(self, this: Pin<&mut MaybeUninit<Self::Output>>);
}

```

[Rust](#code:1)

A `New` type is not what is being constructed; rather, it represents a *method* of construction, resembling a specialized `Fn` trait. The constructed type is given by the associated type `Output`.

Types that can be constructed are constructed *in place*, unlike most Rust types. This is a property shared by constructors in C++, allowing values to record their own address at the moment of creation. Explaining why this is useful is a bit long-winded, but let’s assume this is a thing we want to be able to do. Crucially, we need the output of a constructor to be pinned, which is why the `this` output parameter is pinned.

Calling a constructor requires creating the output location in advance so that we can make it available to it in time:

```rust highlight
// Create storage for the new value.
let mut storage = MaybeUninit::uninit();

// Pin that storage on the stack; by calling this, we may never move
// `storage` again, even after this goes out of scope.
let uninit = Pin::new_unchecked(&mut storage);

// Now we can call the constructor. It's only unsafe because it assumes
// the provided memory is uninitialized.
my_new.new(uninit.as_mut());

// This is now safe, since `my_new` initialized the value, so we can
// do with it what we please.
let result = uninit.map_unchecked_mut(|mp| mp.assume_init_mut());

```

[Rust](#code:2)

However, this is not quite right. `Pin<P>`’s docs are quite clear that we must ensure that, once we create an `Pin<&mut T>`, we must call `T`’s destructor before its memory is re-used; since reuse is unavoidable for stack data, and `storage` will not do it for us (it’s a `MaybeUninit<T>`, after all), we must somehow run the destructor separately.

### [An “Easy” Solution](\#an-easy-solution)

One trick we could use is to replace `storage` with some kind of wrapper over a `MaybeUninit<T>` that calls the destructor for us:

```rust highlight
struct EventuallyInit<T>(MaybeUninit<T>);
impl<T> Drop for EventuallyInit<T> {
  fn drop(&mut self) {
    unsafe { ptr::drop_in_place(self.0.assume_init_mut()) }
  }
}

```

[Rust](#code:3)

This *works*, but isn’t ideal, because now we can’t write down something like a C++ move constructor without running into the classic C++ problem: all objects must be destroyed unconditionally, so now you can have moved-from state. Giving up Rust’s moves-transfer-ownership (i.e. *affine*) property is bad, but it turns out to be avoidable!

There are also some scary details around panics here that I won’t get into.

### [`&T`, `&mut T`, … `&move T`?](\#t-mut-t--move-t)

`moveit` instead provides a [`MoveRef<'frame, T>`](https://docs.rs/moveit/latest/moveit/move_ref/struct.MoveRef.html) type that tries to capture the notion of what an “owning reference” *could* mean in Rust. An `&move` or `&own` type has been discussed many times, but implementing it in the full generality it would deserve as a language feature runs into some interesting problems due to how `Box<T>`, the heap allocated equivalent, [currently behaves.](https://manishearth.github.io/blog/2017/01/10/rust-tidbits-box-is-special/)

We can think of `MoveRef<'frame, T>` as wrapping the *longest-lived* `&mut T` reference pointing to a particular location in memory. The longest-lived part is crucial, since it means that `MoveRef` is entitled to run its pointee’s destructor:

```rust highlight
// Notice parallels with EventuallyInit<T> above.
impl<T: ?Sized> Drop for MoveRef<'_, T> {
  fn drop(&mut self) {
    unsafe { ptr::drop_in_place(self.ptr) }
  }
}

```

[Rust](#code:4)

No reference to the pointee can ever outlive the `MoveRef` itself, by definition, so this is safe. The owner of a value is that which is entitled to destroy it, and therefore a `MoveRef` *literally* owns its pointee. Of course, this means we can move out of it (which was the whole point of the original blogpost).

Because of this, we are further entitled to arbitrarily pin a `MoveRef` with no consequences: pinning it would consume the unpinned `MoveRef` (for obvious reasons, `MoveRefs` cannot be reborrowed) so no unpinned reference may outlive the pinning operation.

This gives us a very natural solution to the problem above: `result` should not be a `Pin<&mut T>`, but rather a `Pin<MoveRef<'_, T>>`:

```rust highlight
let mut storage = MaybeUninit::uninit();
let uninit = Pin::new_unchecked(&mut storage);
my_new.new(uninit.as_mut());

// This is now safe, since `my_new` initialized the value, so we can
// do with it what we please.
let result = MoveRef::into_pinned(MoveRef::new_unchecked(
  uninit.map_unchecked_mut(|mp| mp.assume_init_mut())
));

```

[Rust](#code:5)

This messy sequence of steps is nicely wrapped up in a macro provided by the library that ensures safe initialization and eventual destruction:

```rust highlight
// Allocate storage on the stack, emplace `my_new` onto it, and pin it
// in an owning reference.
moveit!(let result: Pin<MoveRef<T>> = my_new);

```

[Rust](#code:6)

There is also some reasonably complex machinery that allows us to do something like an owning `Deref`, which I’ll come back to in a bit.

However, there is a small wrinkle that I did not realize when I first designed `MoveRef`: what happens if I `mem::forget` a `MoveRef`?

## [Undefined Behavior, Obviously](\#undefined-behavior-obviously)

Quashing destruction isn’t new to Rust: we can `mem::forget` just about anything, leaking all kinds of resources. And that’s ok! Destructors alone cannot be used in type design to advert `unsafe` catastrophe, a well-understood limitation of the language that we have experience designing libraries around, such as [`Vec::drain()`](https://doc.rust-lang.org/nomicon/leaking.html#drain).

`MoveRef`’s design creates a contradiction:

- `MoveRef` is an owning smart pointer, and therefore can be safely pinned, much like `Box::into_pinned()` enables. Constructors, in particular, are *designed* to generate pinned `MoveRef` s!
- Forgetting a `MoveRef` will cause the pointee destructor to be suppressed, but its storage will still be freed and eventually re-used, a violation of the `Pin` drop guarantee.

This would *appear* to mean that a design like `MoveRef` is not viable at all, and that this sort of “stack box” strategy is always unsound.

> [aside](#aside:1)
>
> What about it? Even though we can trivially create a `Pin<Box<i32>>` via `Box::pin()`, this is a red herring. When we `mem::forget` a `Box`, we also forget about its storage too. Because its storage has been leaked unrecoverably, we are still, *technically*, within the bounds of the `Pin` contract. Only barely, but we’re inside the circle.

Interestingly, the Rust language has to deal with a similar problem; perhaps it suggests a way out?

## [Drop Flags and Dynamic Ownership Transfer](\#drop-flags-and-dynamic-ownership-transfer)

Carefully crafted Rust code emits some very interesting assembly. I’ve annotated the key portion of the output with a play-by-play below.

```rust highlight
#[inline(never)]
pub fn someone_elses_problem(_: Box<i32>) {
  // What goes in here isn't important,it just needs to
  // be an optimizer black-box.
}

pub fn maybe_drop(flag: bool) {
  let x = Box::new(42);
  if flag {
    someone_elses_problem(x)
  }
}

```

[godbolt](https://godbolt.org/clientstate/eyJzZXNzaW9ucyI6W3siY29tcGlsZXJzIjpbeyJpZCI6ImJldGEiLCJvcHRpb25zIjoiLU8ifV0sImlkIjoxLCJsYW5ndWFnZSI6InJ1c3QiLCJzb3VyY2UiOiIjW2lubGluZShuZXZlcildXG5wdWIgZm4gc29tZW9uZV9lbHNlc19wcm9ibGVtKF86IEJveFx1MDAzY2kzMlx1MDAzZSkge1xuICAvLyBXaGF0IGdvZXMgaW4gaGVyZSBpc24ndCBpbXBvcnRhbnQsaXQganVzdCBuZWVkcyB0b1xuICAvLyBiZSBhbiBvcHRpbWl6ZXIgYmxhY2stYm94LlxufVxuXG5wdWIgZm4gbWF5YmVfZHJvcChmbGFnOiBib29sKSB7XG4gIGxldCB4ID0gQm94OjpuZXcoNDIpO1xuICBpZiBmbGFnIHtcbiAgICBzb21lb25lX2Vsc2VzX3Byb2JsZW0oeClcbiAgfVxufVxuIn1dfQ==) [Rust](#code:7)

```x86 highlight
// See Godbolt widget above for full disassembly.
example::maybe_drop:
  // ...

  // Allocate memory.
  call    __rust_alloc

  // Check if allocation failed; panic if so.
  test    rax, rax
  je      .L.scream

  // Write a 42 to the memory.
  mov     dword ptr [rax], 42

  // Check the flag argument (LLVM decided to put it in rbx). If
  // true, we go free the memory ourselves.
  test    bl, bl
  je      .L.actually_our_problem

  // Otherwise, make it someone else's problem; they get to
  // free the memory for themselves.
  mov     rdi, rax
  pop     rbx
  jmp     example::someone_elses_problem

  // ...

```

[x86 Assembly](#code:8)

The upshot is that `maybe_drop` conditions the destructor of `x` on a *flag*, which is allocated next to it on the stack. Rust flips this flag when the value is moved into another function, and only runs the destructor when the flag is left alone. In this case, LLVM folded the flag into the `bool` argument, so this isn’t actually a meaningful perf hit.

These “drop flags” are key to Rust’s ownership model. Since ownership may be transferred dynamically due to reasonably complex control flow, it needs to leave breadcrumbs for itself to figure out whether the value wound up getting moved away or not. This is unique to Rust: in C++, every object is always destroyed, so no such faffing about is necessary.

Similarly, `moveit` can close this soundness hole by leaving itself breadcrumbs to determine if safe code is trying to undermine its guarantees.

In other words: in Rust, it is not sufficient to manage a pointer to manage a memory location; it is necessary to manage an explicit or implicit drop flag as well.

## [A Flagged `MoveRef`](\#a-flagged-moveref)

We can extend `MoveRef` to track an explicit drop flag:

```rust highlight
pub struct MoveRef<'frame, T> {
  ptr: &'frame mut T,

  // Set to `false` once the destructor runs.
  drop_flag: &'frame Cell<bool>,
}

```

[Rust](#code:9)

Wrapping it in a `Cell` is convenient and doesn’t cost us anything, since a `MoveRef` can never be made `Send` or `Sync` anyways. Inside of its destructor, we can flip the flag, much like Rust flips a drop flag when transferring ownership to another function:

```rust highlight
impl<T: ?Sized> Drop for MoveRef<'_, T> {
  fn drop(&mut self) {
    self.drop_flag.set(false);
    unsafe { ptr::drop_in_place(self.ptr) }
  }
}

```

[Rust](#code:10)

But, how should we use it? The easiest way is to change the definition of `moveit!()` to construct a *flag trap*:

```rust highlight
let mut storage = MaybeUninit::uninit();
let uninit = Pin::new_unchecked(&mut storage);

// Create a *trapped flag*, which I'll describe below.
let trap = TrappedFlag::new();

// Run the constructor as before and construct a MoveRef.
my_new.new(uninit.as_mut());
let result = MoveRef::into_pin(MoveRef::new_unchecked(
  Pin::into_inner_unchecked(uninit).assume_init_mut(),
  trap.flag(),  // Creating a MoveRef now requires
                // passing in a flag in addition to
                // a reference to the owned value itself.
));

```

[Rust](#code:11)

The trap is a deterrent against forgetting a `MoveRef`: because the `MoveRef`’s destructor flips the flag, the trap’s destructor will notice if this *doesn’t* happen, and take action accordingly.

> [note](#note:1)
>
> In `moveit`, this is actually implemented by having the `Slot<T>` type carry a reference to the trap, created in the `slot!()` macro. However, this is not a crucial detail for the design.

### [An Earth-Shattering Kaboom](\#an-earth-shattering-kaboom)

The trap is another RAII type that basically looks like this:

```rust highlight
pub struct TrappedFlag(Cell<bool>);
impl Drop for TrappedFlag {
  fn drop(&mut self) {
    if self.0.get() { abort() }
  }
}

```

[Rust](#code:12)

The trap is simple: if the contained drop flag is not flipped, it crashes the program. Because `moveit!()` allocates it on the stack where uses cannot `mem::forget` it, its destructor is guaranteed to run before `storage`’s destructor runs (although Rust does not guarantee destructors run, it *does* guarantee their order).

If a `MoveRef` is forgotten, it won’t have a chance to flip the flag, which the trap will detect. Once the trap’s destructor notices this, it cannot return, either normally or by panic, since this would cause `storage` to be freed. Crashing the program is the only[1](#fn:1) acceptable response.

Some of `MoveRef`’s functions need to be adapted to this new behavior: for example, `MoveRef::into_inner()` still needs to flip the flag, since moving out of the `MoveRef` is equivalent to running the destructor for the purposes of drop flags.

## [A Safer `DerefMove`](\#a-safer-derefmove)

In order for `MoveRef` to be a proper “new” reference type, and not just a funny smart pointer, we also need a `Deref` equivalent:

```rust highlight
pub unsafe trait DerefMove: DerefMut + Sized {
  /// An "uninitialized" version of `Self`.
  type Uninit: Sized;

  /// "Deinitializes" `self`, producing an opaque type that will
  /// destroy the storage of `*self` without calling the pointee
  /// destructor.
  fn deinit(self) -> Self::Uninit;

  /// Moves out of `this`, producing a `MoveRef` that owns its
  /// contents.
  unsafe fn deref_move(this: &mut Self::Uninit)
    -> MoveRef<'_, Self::Target>;
}

```

[Rust](#code:13)

This is the *original* design for `DerefMove`, which had a two-phase operation: first `deinit()` was used to create a destructor-suppressed version of the smart pointer that would only run the destructor for the storage (e.g., for `Box`, only the call to `free()`). Then, `deref_move()` would extract the “inner pointee” out of it as a `MoveRef`. This had the effect of splitting the smart pointer’s destructor, much like we did above on the stack.

This has a number of usability problems. Not only does it *need* to be called through a macro, but `deinit()` isn’t actually safe: failing to call `deref_move()` is just as bad as calling `mem::forget` on the result. Further, it’s not clear where to plumb the drop flags through.

After [many attempts](https://github.com/google/moveit/pull/23#issuecomment-963549869) to graft drop flags onto this design, I replaced it with a completely new interface:

```rust highlight
pub unsafe trait DerefMove: DerefMut + Sized {
  /// The "pure storage" form of `Self`, which owns the storage
  /// but not the pointee.
  type Storage: Sized;

  /// Moves out of `this`, producing a [`MoveRef`] that owns
  /// its contents.
  fn deref_move<'frame>(
    self,
    storage: DroppingSlot<'frame, Self::Storage>,
  ) -> MoveRef<'frame, Self::Target>
  where
    Self: 'frame;
}

```

[Rust](#code:14)

`Uninit` has been given the clearer name of `Storage`: a type that owns *just* the storage of the moved-from pointer. The two functions were merged into a single, *safe* function that performs everything in one step, emitting the storage as an out-parameter.

The new `DroppingSlot<T>` is like a `Slot<T>`, but closer to a safe version of the `EventuallyInit<T>` type from earlier: its contents are not necessarily initialized, but if they are, it destroys them, and it only does so when its drop flag is set.

`Box` is the most illuminating example of this trait:

```rust highlight
unsafe impl<T> DerefMove for Box<T> {
  type Storage = Box<MaybeUninit<T>>;

  fn deref_move<'frame>(
    self,
    storage: DroppingSlot<'frame, Box<MaybeUninit<T>>>,
  ) -> MoveRef<'frame, T>
  where
    Self: 'frame
  {
    // Dismantle the incoming Box into the "storage-only part".
    let this = unsafe {
      Box::from_raw(Box::into_raw(self).cast::<MaybeUninit<T>>())
    };

    // Put the Box into the provided storage area. Note that we
    // don't need to set the drop flag; `DroppingSlot` does
    // that automatically for us.
    let (storage, drop_flag) = storage.put(this);

    // Construct a new MoveRef, converting `storage` from
    // `&mut Box<MaybeUninit<T>>` into `&mut T`.
    unsafe { MoveRef::new_unchecked(storage.assume_init_mut(), drop_flag) }
  }
}

```

[Rust](#code:15)

`MoveRef`’s own implementation illustrates the need for the explicit lifetime bound:

```rust highlight
unsafe impl<'a, T: ?Sized> DerefMove for MoveRef<'a, T> {
  type Storage = ();

  fn deref_move<'frame>(
    self,
    _: DroppingSlot<'frame, ()>,
  ) -> MoveRef<'frame, T>
  where
    Self: 'frame
  {
    // We can just return directly; this is a mere lifetime narrowing.
    self
  }
}

```

[Rust](#code:16)

Since this is fundamentally a lifetime narrowing, this can only compile if we insist that `'a: 'frame`, which is implied by `Self: 'frame`. Earlier iterations of this design enforced it via a `MoveRef<'frame, Self>` receiver, which turned out to be unnecessary.

## [Conclusions](\#conclusions)

As of writing, I’m still in the process of self-reviewing [this change](https://github.com/google/moveit/pull/25), but at this point I feel *reasonably* confident that it’s correct; this article is, in part, written to convince myself that I’ve done this correctly.

The new design will also enable me to finally complete my implementation of a constructor and pinning-friendly vector type; this issue came up in part because the vector type needs to manipulate drop flags in a complex way. For this reason, the *actual* implementation of drop flags actually uses a counter, not a single boolean.

I doubt this is the last issue I’ll need to chase down in `moveit`, but for now, we’re ever-closer to true owning references in Rust.

---

1. Arguably, running the skipped destructor is *also* a valid remediation strategy. However, this is incompatible with what the user requested: they asked for the destructor to be supressed, not for it to be run at a later date. This would be somewhat surprising behavior, which we could warn about for the benefit of `unsafe` code, but ultimately the incorrect choice for non-stack storage, such as a `MoveRef` referring to the heap. [↩︎](#fnref:1)
