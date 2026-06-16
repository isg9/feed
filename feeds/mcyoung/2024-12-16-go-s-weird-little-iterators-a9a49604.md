---
title: Go's Weird Little Iterators
url: /2024/12/16/rangefuncs
published: "2024-12-16T00:00:00Z"
feed: mcyoung
guid: /2024/12/16/rangefuncs
---

# Go's Weird Little Iterators

A second post on Go silliness (Sunny, aren’t you a C++ programmer?): in 1.23, Go *finally* added custom iterators. Now, back when I was at Google and involved in the Go compiler as “the annoying Rust guy who gets lunch with us”, there were proposals suggesting adding something like this, implemented as either an interface or a func:

```go highlight
type Iter[T any] = func() (T, bool)

```

[Go](#code:1)

This is not what Go did. No, Go did something really weird. And the implementation is *incredible*.

## [What’s an Iterator?](\#whats-an-iterator)

An *iterator*, in the context of programming language design, is a special type of value that can be used to walk through a sequence of values, without necessarily materializing the sequence as whatever the language’s array type is.

But, a proper iterator must fit with the language’s looping construct. An *iterable type* is one which can be used in a for-each loop, such as C++’s `for (T x : y)` or Python’s `for x in y` (modern languages usually only have a for-each loop as their only `for` loop, because C-style for loops are not in anymore).

### [C++ Iterator Pairs](\#c-iterator-pairs)

Every language defines a *desugaring* that defines how custom iteration works in term of the more primitive loops. For example, in C++, when we write `for (T x : y) { ... }` (called a [*range-based for loop*](https://en.cppreference.com/w/cpp/language/range-for), added in C++11), desugars as follows[1](#fn:1):

```cpp highlight
auto&& __range = y;
auto __begin = begin(__range); // ADL
auto __end = end(__range);     // ADL
for (; __begin != __end; ++__begin) {
  T x = *__begin;
  // ...
}

```

[C++](#code:2)

`break`, `continue`, and `return` inside of the loop body require no special handling: they Just Work, because this is just a plain ol for loop.

This begin and end weirdness is because, if the iterator backs an actual array, begin and end can just be pointers to the first element and one-past-the-end and this will Just Work. Before C++11, the convention for C++ iterators was to construct types that imitated pointers; you would usually write loops over non-array types like this:

```cpp highlight
for (auto it = things.begin(); it != things.end(); ++it) {
  whatever(*it);
}

```

[C++](#code:3)

C++ simply codified common (if gross) practice. It is very tedious to implement C++ iterators, though. You need to provide a dummy end iterator, you need to provide some kind of comparison operator, and iterators that don’t return a reference out of `operator*()` are… weird.

Begin and end can be different types (which is how C++20 ranges pretend to be iterable), but being able to query done-ness separately from the next value makes implementation annoying: it means that an iterator that has not begun iteration (i.e., `++` has not been executed yet, because it occurs in the loop’s latch, not its header[3](#fn:3)) needs to do extra work to answer `!= end`, which usually means an extra bool to keep track of whether iteration has started or not.

Here’s what writing an iterator (that is also an iterable usable in a range for-loop) over the non-zero elements of a `std::span<const int>` might look like.

```cpp highlight
struct NonZero {
  std::span<const int> ints;

  auto begin() { return *this; }
  auto end() { return sentinel{}; }

  bool operator==(sentinel) {
    while (!ints.empty()) {
      if (ints[0] == 0) ints = ints.subspan(1);
    }
    return ints.empty();
  }
  bool operator!=(sentinel s) { return !(*this == s); }

  NonZero& operator++() {
    ints = ints.subspan(1);
    return *this;
  }
  NonZero operator++(int) {
    auto prev = *this;
    ++*this;
    return prev;
  }

  const int& operator*() const { return ints[0]; }

 private:
  struct sentinel{};
};

```

[C++](#code:4)

In this case, `operator==` is *not* `const`, which is a bit naughty. Purists might argue that this type should have a constructor, which adjusts `ints` to point to the first non-zero element on construction, and `operator++` to perform the mutation. That would look like this:

```cpp highlight
class NonZero {
 public:
  NonZero(std::span<const int> ints) : ints_(ints) {
    skip_zeros();
  }

  auto begin() { return *this; }
  auto end() { return sentinel{}; }

  bool operator==(sentinel) const { return ints.empty(); }
  bool operator!=(sentinel s) const { return !(*this == s); }

  NonZero& operator++() {
    skip_zeros();
    ints = ints.subspan(1);
    return *this;
  }
  NonZero operator++(int) {
    auto prev = *this;
    ++*this;
    return prev;
  }

  const int& operator*() const { return ints[0]; }

 private:
  struct sentinel{};
  void skip_zeros() {
    while (!ints.empty() && ints[0] == 0) {
      ints = ints.subspan(1);
    }
  }
  std::span<const int> ints_;
};

```

[C++](#code:5)

`std::sentinel_for` (C++’s iterator concepts are terribly named) really wants `operator==` to be `const`, but I could have also just marked `ints` as `mutable` to avoid that. It it’s not already clear, I really dislike this pattern. See [here](https://github.com/mcy/best/blob/main/best/iter/iter.h) for some faffing about with C++ iterators on my part.

### [Java Also Got This Wrong](\#java-also-got-this-wrong)

At least Java provides a standard iterable interface, thankfully.

```java highlight
package java.util;

public interface Iterable<E> {
  Iterator<E> iterator();
}

public interface Iterator<E> {
  boolean hasNext();
  E next();
}

```

[Java](#code:6)

The desugaring of `for (T x : y) { ... }` is then:

```java highlight
for (var $iter = x.iterator(); $iter.hasNext();) {
  T x = $iter.next();
}

```

[Java](#code:7)

Do you see the problem here? Although Java now provides a standard interface, doesn’t require annoying equality comparisons, and doesn’t require an end value, these things are *still* a pain to implement! You still need to be able to query if you’re done before you’ve had a chance to step through the iterator.

Like before, suppose we have an `int[]`, and we want to yield every non-zero value in it. How do we construct an iterator for that?

```java highlight
int[] xs = null;
var it = new Iterator<Integer>() {
  int[] array = xs;
  int idx;

  public boolean hasNext() {
    for (; !done() && array[idx] == 0; idx++) {}
    return !done();
  }

  public Integer next() {
    if (!hasNext()) throw new NoSuchElementException();
    return array[idx++];
  }

  private boolean done() {
    return array == null || idx == array.length;
  }
};

```

[Java](#code:8)

What a pain. Java’s anonymous classes being wordy aside, it’s annoying and error-prone to do this: it’s tempting to accidentally implement `hasNext` by simply checking if the array is empty. (Aside, I hate that `xs.length` throws on null arrays. Just return zero like in Go, c’mon).

Also, it’s no a single-abstract-method interface, so I can’t use a lambda to create an iterator.

At least `break`, `continue`, and `return` Just Work, because the underlying operation is a for loop like before.

### [Rust Does It Better](\#rust-does-it-better)

Rust *also* has a standard iterable interface.

```rust highlight
// mod core::iter

pub trait IntoIterator {
  type Item;
  type Iter: Iterator<Item=Self::Item>;

  fn into_iter() -> Self::Iter;
}

pub trait Iterator {
  type Item;
  fn next() -> Option<Self::Item>;
}

```

[Rust](#code:9)

The desugaring for `for x in y { ... }` is reasonably straightforward, like in Java:

```rust highlight
let mut __it = IntoIterator::into_iter(y);
while let Some(x) = __it.next() {
  // ...
}

```

[Rust](#code:10)

This is so straightforward that it’s not *so* unusual to write it yourself, when you don’t plan on consuming the entire iterator. Alternatively, you can partially iterate over an iterator by taking a mutable reference to it. This is useful for iterators that can yield their remainder.

```rust highlight
let mut it = my_slice.chunks_exact(8);
for chunk in &mut it {
  do_thing(chunk);
}
do_thing(it.remainder());

```

[Rust](#code:11)

`break`, `continue`, and `return` work in the obvious way.

The interface solves the problems C++ and Java had very cleanly: `next` both computes the next item and whether the iterator has more elements. Rust even allows iterators to resume yielding `Some` after yielding `None`, although few algorithms will make use of this.

Implementing the non-zero iterator we’ve been writing so far is quite simple:

```rust highlight
let mut ints: &[i32] = ...;
let it = std::iter::from_fn(move || {
  while ints.get(0) == Some(0) {
    ints = &ints[1..];
  }
  let item = ints.get(0);
  ints = &ints[1..];
  item
});

```

[Rust](#code:12)

However, this can be written far more simply[4](#fn:4) using iterator combinators:

```rust highlight
let ints = &[i32] = ...;
let it = ints.iter().copied().filter(|x| x != 0);

```

[Rust](#code:13)

It requires a little bit of effort to implement some iterators, but most of the common cases are easy to put together with composition.

Python iterators are basically the same thing, but there’s no interface to implement (because Python doesn’t believe in type safety). Lua iterators are similar. The Rust pattern of a function that returns the next item (or a special end-of-sequence value) is relatively popular because of this simplicity and composability, and because they can model a lot of iteration strategies.

## [So, What Did Go Do?](\#so-what-did-go-do)

Well. Go has a range for syntax like many other languages. The syntax looks like this:

```go highlight
for x := range y {
  // ...
}

```

[Go](#code:14)

The `x` can be a list of places, and the `:=` can be plain assignment, `=`. You can also write `for range y { ... }` if the iteration values aren’t needed.

The behavior of this construct, like many others in Go, depends explicitly on the type after `range`. Each range iteration can yield zero or more values; the

These are:

1. For `[]T`, `[n]T`, and `*[n]T`, each step yields an index of the slice and the value at that offset, in order.

2. For `map[K]V`, each step yields a key and a value, in a random order.

3. For `<- chan T`, it desugars into

   ```go highlight
   for {
    x, ok := <-y
    if !ok { break }
    // ...
   }

   ```

   [Go](#code:15)

4. Starting in Go 1.22, ranging on an integer type would desugar into

   ```go highlight
   for x := 0; x < y; i++ {
    // ...
   }

   ```

   [Go](#code:16)

All of these desugars are essentially still just loops, so `break`, `continue`, `goto`, and `return` all work as expected.

But, how do custom types, like weird map types, implement iteration? The usual[5](#fn:5) implementation is [`sync.Map.Range`](https://pkg.go.dev/sync#Map.Range), which looks like this:

```go highlight
func (*Map) Range(yield func(key, value any) bool)

```

[Go](#code:17)

This function will call `yield` for each element in the map. If the function returns `false`, iteration will stop. This pattern is not uncommon, but sometimes libraries omit the `bool` return (like [`container/ring.Ring.Do`](https://pkg.go.dev/container/ring@go1.23.4#Ring.Do)). Some, like [`filepath.WalkDir`](https://pkg.go.dev/path/filepath#WalkFunc), have a more complex interface involving errors.

This is the template for what became *rangefuncs*, a mechanism for using the for-range syntax with certain function values.

## [Rangefuncs](\#rangefuncs)

The word “rangefunc” does not appear in Go’s specification. It is a term used to refer to them in some documentation, within the compiler, and in the runtime.

A rangefunc is any function with one of the following signatures:

- `func(yield func() bool)`
- `func(yield func(V) bool)`
- `func(yield func(K, V) bool)`

They work like `sync.Map.Range` does: the function calls `yield` (hereafter simply called “the yield”) for each element, and stops early if yield returns false. The `iter` package contains types for the second and third of these:

```go highlight
package iter

type Seq[V any] func(yield func(V) bool)
type Seq2[K, V any] func(yield func(K, V) bool)

```

[Go](#code:18)

For example, the `slices` package provides an adaptor for converting a slice into an iterator that ranges over it.

```go highlight
package slices

// All returns an iterator over index-value pairs in the slice
// in the usual order.
func All[Slice ~[]E, E any](s Slice) iter.Seq2[int, E] {
	return func(yield func(int, E) bool) {
		for i, v := range s {
			if !yield(i, v) {
				return
			}
		}
	}
}

```

[Go](#code:19)

So. These things are actually pretty nuts. They break my brain somewhat, because this is the opposite of how iterators usually work. Go calls what I’ve described all the other languages do a “pull iterator”, whereas rangefuncs are “push iterators”.

They have a few obvious limitations. For one, you can’t do smart sizing like with Rust or C++ iterators[6](#fn:6). Another is that you can’t easily “pause” iteration.

But they do have one advantage, which I think is the real reason Go went to so much trouble to implement them (and yes, I will dig into how insane that part is). Using push iterators by default means that users “only” need to write an ordinary for loop packaged into a function. Given that Go makes major performance sacrifices in order to be easy to learn[7](#fn:7), trying to make it so that an iterator packages the actual looping construct it represents makes quite a bit of sense.

Rangefuncs are actually really cool in some respects, because they enable unusual patterns. For example, you can use a rangefunc to provide RAII blocks.

```go highlight
func Open(fs fs.FS, path string) iter.Seq2[fs.File, error] {
  return func(yield func(fs.File, error) bool) {
    f, err := fs.Open(path)
    if err {
      yield(nil, err)
      return
    }

    defer f.Close()
    yield(f, nil)
  }
}

for f, err := range Open(os.DirFS("/"), "etc/passwd") {
  if err != nil {
    return nil
  }

  // ...
}

```

[Go](#code:20)

Being a block that you can put an epilog onto after yielding a single element is quite powerful! You can also use a nilary rangefunc to simply create a block that you can break out of, instead of having to use `goto`.

```go highlight
func Once() func(func() bool) {
  return func(y func() bool) { y() }
}

for range Once() {
  if canDo() {
    break
  }

  do()
}

```

[Go](#code:21)

So wait. You can return out of rangefunc loops. That means that… Go has non-local returns?!

## [Go Now Has Non-Local Returns](\#go-now-has-non-local-returns)

The desugaring for rangefuncs is very complicated. This is because `break`, `continue`, `goto`, and `return` all work in a rangefunc! How does this work? Let’s Godbolt it.

Let’s start with something really basic: a loop body that just calls a function.

```go highlight
package x

import "iter"

func run(s iter.Seq[int]) {
  for x := range s {
    sink(x)
  }
}

func sink(int)

```

[Go](#code:22)

This produces the following assembly output (which I’ve reformatted into Intel syntax, and removed some extraneous ABI things, including a writer barrier where `(*)` is below).

```x86 highlight
x.run:
    push    rbp
    mov     rbp, rsp
    add     rsp, -24
    mov     [rsp + 40], rax
    lea     rax, [type:int]
    call    runtime.newobject
    mov     [rsp + 16], rax
    mov     [rax], internal/abi.RF_READY
    lea     rax, ["type:noalg.struct { F uintptr; X0 *int }"]
    call    runtime.newobject
    lea     rcx, x.run-range1
    mov     [rax], rcx  // (*)
    mov     rcx, [rsp + 16]
    mov     [rax + 8], rcx
    mov     rdx, [rsp + 40]
    mov     rbx, [rdx]
    call    rbx
    mov     rcx, [rsp + 16]
    cmp     [rcx], internal/abi.RF_PANIC
    jeq     panic
    mov     [rcx], internal/abi.RF_EXHAUSTED
    add     rsp, 24
    pop     rbp
    ret
panic:
    mov     rax, internal/abi.RF_MISSING_PANIC
    call    runtime.panicrangestate

x.run-range1:
    push    rbp
    mov     rbp, rsp
    add     rsp, -24
    mov     [rsp + 8], rdx
    mov     rcx, [rdx + 8]
    mov     rdx, [rcx]
    cmp     qword ptr [rdx], internal/abi.RF_READY
    jne     panic2
    mov     [rsp + 16], rcx
    mov     qword ptr [rcx], internal/api.RF_PANIC
    call    x.sink
    mov     rcx, [rsp + 16]
    mov     qword ptr [rcx], internal/abi.RF_READY
    mov     rax, 1
    add     rsp, 24
    pop     rpb
    ret
panic2:
    mov     rax, rdx
    call    runtime.panicrangestate

```

[x86 Assembly](#code:23)

This is a lot to take in, but if we look carefully, we decompile this function into a Go function:

```go highlight
import (
  "internal/abi"
  "runtime"
)

func run(s iter.Seq[int]) {
  __state := abi.RF_PANIC
  s(func(v int) bool {
    if __state != abi.RF_READY {
      runtime.panicrangestate(*state)
    }
    __state = abi.RF_PANIC
    sink(v)  // Loop body
    __state = abi.RF_READY
    return true
  })
  __state = abi.RF_EXHAUSTED
}

```

[Go](#code:24)

Go will actually enforce invariants on the yield it synthesizes in a range for, in order to catch buggy code. In particular, `__state` escapes because `s` is an arbitrary function, so it gets spilled to the heap.

So, what happens when the loop body contains a break? Consider:

```go highlight
package x

import "iter"

func run(s iter.Seq[int]) {
  for x := range s {
    if sink(x) { break }
  }
}

func sink(int)

```

[Go](#code:25)

I’ll spare you the assembly listing, since it’s very similar, so I’ll just reverse-engineer the output directly:

```go highlight
import (
  "internal/abi"
  "runtime"
)

func run(s iter.Seq[int]) {
  __state := abi.RF_PANIC
  s(func(v int) bool {
    if __state != abi.RF_READY {
      runtime.panicrangestate(*state)
    }
    __state = abi.RF_PANIC
    if sink(v) {
      __state = abi.RF_DONE
      return false
    }
    __state = abi.RF_READY
    return true
  })
  __state = abi.RF_EXHAUSTED
}

```

[Go](#code:26)

Non-local returns are much more complicated. Consider:

```go highlight
package x

import "iter"

func run(s iter.Seq[int]) int {
  for x := range s {
    if sink(x) { return x }
  }
  return -1
}

func sink(int)

```

[Go](#code:27)

The resulting assembly is something like this, with some irrelevant code, such as write barriers, removed:

```x86 highlight
x.run:
    push    rbp
    mov     rbp, rsp
    add     rsp, -40
    mov     [rsp + 56], rax
    lea     rax, [type:int]
    call    runtime.newobject
    mov     [rsp + 24], rax
    lea     rax, [type:int]
    call    runtime.newobject
    mov     [rsp + 32], rax
    lea     rax, [type:int]
    call    runtime.newobject
    mov     [rsp + 16], rax
    mov     [rax], internal/abi.RF_READY
    lea     rax, ["type:noalg.struct { F uintptr; X0 *int; X1 *int; X2 *int }"]
    call    runtime.newobject
    lea     rcx, [x.run-range1]
    mov     [rax], rcx
    mov     rcx, [rsp + 16]
    mov     rbx, [rsp + 24]
    mov     rsi, [rsp + 32]
    mov     [rax + 8], rcx
    mov     [rax + 16], rbx
    mov     [rax + 24], rsi
    mov     rdx, [rsp + 56]
    mov     rdi, [rdx]
    call    rdi
    mov     rcx, [rsp + 16]
    cmp     qword ptr [rcx], internal/abi.RF_PANIC
    jeq     panic
    mov     [rcx], internal/abi.RF_EXHAUSTED
    mov     rcx, [rsp + 32]
    cmp     qword ptr [rcx], -1
    jne     resume
    mov     rcx, [rsp + 32]
    mov     rax, [rcx]
    add     rsp, 40
    pop     rbp
    ret
resume:
    mov     rcx, [rsp + 32]
    mov     qword ptr [rcx], -1
    mov     rax, -1
    add     rsp, 40
    pop     rbp
    ret
panic:
    mov     rax, internal/abi.RF_MISSING_PANIC
    call    runtime.panicrangestate

x.run-range1
    push    rbp
    mov     rbp, rsp
    add     rsp, -40
    mov     [rsp + 8], rdx
    mov     rcx, [rdx + 8]
    mov     rbx, [rcx]
    mov     rsi, [rdx + 16]
    mov     rdx, [rdx + 24]
    cmp     rbx, internal/abi.RF_READY
    jne     panic2
    mov     [rsp + 56], rax
    mov     [rsp + 16], rcx
    mov     [rsp + 24], rsi
    mov     [rsp + 32], rdx
    mov     qword ptr [rcx], internal/abi.RF_PANIC
    call    x.sink
    test    al, al
    jeq     cont
    mov     rcx, [rsp + 56]
    mov     rdx, [rsp + 24]
    mov     [rdx], rcx
    mov     rcx, [rsp + 32]
    mov     qword ptr [rcx], -1
    mov     rcx, [rsp + 16]
    mov     qword ptr [rcx], internal/abi.RF_DONE
    xor     eax, eax
    add     rsp, 40
    pop     rbp
    ret
cont:
    mov     rcx, [rsp + 16]
    mov     qword ptr [rcx], internal/abi.RF_READY
    mov     rax, 1
    pop     rpb
    ret
panic:
    mov     rax, rbx
    call    runtime.panicrangestate

```

[x86 Assembly](#code:28)

Try to reverse engineer this yourself, if you like! If you write this out as Go, here’s what you get:

```go highlight
import (
  "internal/abi"
  "runtime"
)

func run(s iter.Seq[int]) (__ret int) {
  var __next int
  __state := abi.RF_PANIC
  s(func(v int) bool {
    if __state != abi.RF_READY {
      runtime.panicrangestate(*state)
    }
    __state = abi.RF_PANIC
    if sink(v) {
      __state = abi.RF_DONE
      __next = -1
      __ret = v
      return false
    }
    __state = abi.RF_READY
    return true
  })
  __state = abi.RF_EXHAUSTED
  if __next == -1 {
    return
  }

  return -1
}

```

[Go](#code:29)

The reason `__next` is an int is because it is also used when exiting the loop via `goto` or a `break`/ `continue` with label. It specifies where to jump to after the call into the rangefunc returns. Each potential control flow out of the loop is assigned some negative number.

The precise details of the lowering have been [exquisitely documented](https://cs.opensource.google/go/go/+/master:src/cmd/compile/internal/rangefunc/rewrite.go) by Russ Cox and David Chase, the primary implementers of the feature.

You might be curious what `runtime.panicrangestate` does. It’s pretty simple, and it lives in `runtime/panic.go`:

```go highlight
package runtime

//go:noinline
func panicrangestate(state int) {
	switch abi.RF_State(state) {
	case abi.RF_DONE:
		panic(rangeDoneError)
	case abi.RF_PANIC:
		panic(rangePanicError)
	case abi.RF_EXHAUSTED:
		panic(rangeExhaustedError)
	case abi.RF_MISSING_PANIC:
		panic(rangeMissingPanicError)
	}
	throw("unexpected state passed to panicrangestate")
}

```

[Go](#code:30)

If you visit this function in [runtime/panic.go](https://cs.opensource.google/go/go/+/refs/tags/go1.23.4:src/runtime/panic.go;l=306?q=panicrangestate&ss=go%2Fgo), you will be greeted by this extremely terrifying comment from Russ Cox immediately after it.

```go highlight
// deferrangefunc is called by functions that are about to
// execute a range-over-function loop in which the loop body
// may execute a defer statement. That defer needs to add to
// the chain for the current function, not the func literal synthesized
// to represent the loop body. To do that, the original function
// calls deferrangefunc to obtain an opaque token representing
// the current frame, and then the loop body uses deferprocat
// instead of deferproc to add to that frame's defer lists.
//
// The token is an 'any' with underlying type *atomic.Pointer[_defer].
// It is the atomically-updated head of a linked list of _defer structs
// representing deferred calls. At the same time, we create a _defer
// struct on the main g._defer list with d.head set to this head pointer.
//
// The g._defer list is now a linked list of deferred calls,
// but an atomic list hanging off:
//
// (increasingly terrifying discussion of concurrent data structures)

```

[Go](#code:31)

This raises one more thing that works in range funcs, seamlessly: `defer`. Yes, despite the yield executing multiple call stacks away, possibly on a different goroutine… `defer` still gets attached to the calling function.

## [Go Now Has Non-Local Defer](\#go-now-has-non-local-defer)

The way defer works is that each G (the goroutine struct, `runtime.g`) holds a linked list of defer records, of type `_defer`. Each call to `defer` sticks one of these onto this list. On function return, Go calls `runtime.deferreturn()`, which essentially executes and pops defers off of the list until it finds one whose stack pointer is not the current function’s stack pointer (so, it must belong to another function).

Rangefuncs throw a wrench in that mix: if `myFunc.range-n` defers, that defer has to be attached to `myFunc`’s defer records somehow. So the list must have a way of inserting in the middle.

This is what this comment is about: when `defer` occurs in the loop body, that defer gets attached to a defer record for that function, using a token that the yield captures; this is later canonicalized when walking the defer list on the way out of `myFunc`. Because the yield can escape onto another goroutine, this part of the `defer` chain has to be atomic.

Incredibly, this approach is extremely robust. For example, if we spawn the yield as a goroutine, and carefully synchronize between that and the outer function, we can force the runtime to hard-crash when `defer` ing to a function that has returned.

```go highlight
package main

import (
	"fmt"
	"sync"
)

func bad() (out func()) {
	var w1, w2 sync.WaitGroup
	w1.Add(1)
	w2.Add(1)

	out = w2.Done
	defer func() { recover() }()
	iter := func(yield func() bool) {
		go yield()
		w1.Wait() // Wait to enter yield().
    // This panics once w1.Done() executes, because
    // we exit the rangefunc while yield() is still
    // running. The runtime incorrectly attributes
    // this to recovering in the rangefunc.
	}

	for range iter {
		w1.Done() // Allow the outer function to exit the loop.
		w2.Wait() // Wait for bad() to return.
		defer fmt.Println("bang")
	}

  return nil // Unreachable
}

func main() {
	resume := bad()
  resume()
  select {}  // Block til crash.
}

```

[Go](#code:32)

This gets us `fatal error: defer after range func returned`. Pretty sick! It accomplishes this by poisoning the token the yield func uses to defer.

I have tried various other attempts at causing memory unsafety with rangefuncs, but Go actually does a really good job of avoiding this. The only thing I’ve managed to do that’s especially interesting is to tear the return slot on a function without named returns, but that’s no worse than tearing any other value (which is still really bad, because you can tear interface values, but it’s not *worse*).

## [Pull Iterators and Coroutines](\#pull-iterators-and-coroutines)

Of course we’re not done. Go provides a mechanism for converting push iterators into pull iterators. Essentially, there is a function that looks like this:

```go highlight
package iter

func Pull[V any](seq Seq[V]) (next func() (V, bool), stop func()) {
  yield := make(chan struct{value V; ok bool})
  pull := make(chan struct{}{})
  go func() {
    seq(func(v V) bool {
      _, ok := <-pull
      if !ok {
        return false
      }
      yield <- struct{value V; ok bool}{v, true}
    })

    close(yield)
  }()

  next = func() (V, bool) {
    pull <- struct{}{}
    return <-yield
  }
  stop = func() { close(pull) }
  return
}

```

[Go](#code:33)

Essentially, you can request values with `next()`, and `stop()` can be used if you finish early. But also, this spawns a whole goroutine and uses channels to communicate and synchronize, which feels very unnecessary.

The implementation doesn’t use goroutines. It uses coroutines.

### [Giving Up on Goroutines](\#giving-up-on-goroutines)

Spawning a goroutine is expensive. Doing so expends scheduler and memory resources. It’s overkill for a helper like this (ironic, because the original premise of Go was that goroutines would be cheap enough to allocate willy-nilly).

Go instead implements this using “coroutines”, a mechanism for concurrency without parallelism. This is intended to make context switching very cheap, because it does not need to go through the scheduler: instead, it uses cooperative multitasking.

The coroutine interface is something like the following. My “userland” implementation will not be very efficient, because it relies on the scheduler to transfer control. The goroutines may run on different CPUs, so synchronization is necessary for communication, even if they are not running concurrently.

```go highlight
package coro

import (
  "runtime"
  "sync"
)

type Coro struct {
  m sync.Mutex
}

func New(f func()) *Coro {
  c := new(Coro)
  c.m.Lock()
  go func() {
    c.m.Lock()
    f()
    c.m.Unlock()
  }
  return c
}

func (c *Coro) Resume() {
  c.m.Unlock()
  c.m.Lock()
}

```

[Go](#code:34)

When we create a coroutine with `coro.New()`, it spawns a goroutine that waits on a mutex. Another goroutine can “take its place” as the mutex holder by calling `c.Resume()`, which allows the coroutine spawned by `coro.New` to resume and enter `f()`.

Using the coroutine as a rendezvous point, two goroutines can perform concurrent work: in the case of `iter.Pull`, one can be deep inside of whatever loops the iterator wants to do, and the other can request values.

Here’s what using my `coro.Coro` to implement `iter.Pull` might look like:

```go highlight
package iter

func Pull[V any](seq Seq[V]) (next func() (V, bool), stop func()) {
  var (
    done bool
    v, z V
  )

  c := coro.New(func() {
    s(func(v1 V) bool {
      c.Resume()  // Wait for a request for a value.
      if done {
        // This means we resumed from stop(). Break out of the
        // loop.
        return false
      }
      v = v1
    })
    if !done {
      // Yield the last value.
      c.Resume()
    }

    v = z
    done = true
  })

  next = func() (V, bool) {
    if done { return z, false }

    c.Resume()      // Request a value.
    return v, true  // Return it.
  }

  stop = func() {
    if done { return }

    done = true // Mark iteration as complete.
    c.Resume()  // Resume the iteration goroutine to it can exit.
  }

  return next, stop
}

```

[Go](#code:35)

If you look at the implementation in [`iter.go`](https://cs.opensource.google/go/go/+/refs/tags/go1.23.4:src/iter/iter.go), it’s basically this, but with a lot of error checking and race detection, to prevent misuse, such as if `next` or `stop` escape to other goroutines.

Now, the main thing that runtime support brings here is that `Resume()` is immediate: it does not go to the scheduler, which might not decide to immediately run the goroutine that last called `Resume()` for a variety of reasons (for example, to ensure wakeup fairness). Coroutines sidestep fairness, by making `Resume()` little more than a jump to the last `Resume()` (with registers fixed up accordingly).

This is not going to be *that* cheap: a goroutine still needs to be allocated, and switching needs to poke and prod the underlying Gs a little bit. But it’s a cool optimization, and I hope coroutines eventually make their way into more things in Go, hopefully as a language or `sync` primitive.

## [Conclusion](\#conclusion)

Congratulations, you have survived over 3000 words of me going on about iterators. Go’s push iterators are a unique approach to a common language design problem (even if it took a decade for them to materialize).

I encountered rangefuncs for the first time earlier this year and have found them absolutely fascinating, both from a “oh my god they actually did that” perspective and from a “how do we express iteration” perspective. I don’t think the result was perfect by any means, and it is unsuitable for languages that need the performance you can only get from pull iterators. I think they would be a great match for a language like Python or Java, though.

I’d like to thank David Chase, an old colleague, for tolerating my excited contrived questions about the guts of this feature.

---

1. Ugh, ok. This is the C++20 desugaring, and there are cases where we do not just call `std::begin()`. In particular, array references and class type references with `.begin()` and `.end()` do not call `std::begin()` and are open-coded. This means that you can’t use ADL to override these types' iterator. [↩︎](#fnref:1)

2. But please don’t use proto3. I’m telling you that as the guy who maintained the compiler. Just don’t. [↩︎](#fnref:2)

3. In compiler jargon, a loop is broken up into three parts: the *header*, which is where the loop is entered, the *body*, which is one step of iteration, and the *latch*, which is the part that jumps back to the start of the body. This is where incrementation in a C-style for loop happens. [↩︎](#fnref:3)

4. And with better performance. Rust’s iterators can provide a size hint to help size containers before a call to `collect()`, via the [`FromIterator`](https://doc.rust-lang.org/std/iter/trait.FromIterator.html) trait. [↩︎](#fnref:4)

5. Some people observed that you can use a channel as a custom iterator, by having a parallel goroutine run a for loop to feed the channel. *Do not do* *this.* It is slow: it has to transit each element through the heap, forcing anything it points to escape. It takes up an extra M and a P in the scheduler, and requires potentially allocating a stack for a G. It’s probably faster to just build a slice and return that, especially for small iterations. [↩︎](#fnref:5)

6. For this reason, I wish that Go had instead defined something along these lines.

```go highlight
package iter

type Seq[V any] interface {
     Iterate(yield func(V) bool)
}

```

[Go](#code:36)

This is functionally identical to what they did, but it would have permitted future extensions such as the following interface:

```go highlight
package iter

type SizedSeq[V any] interface {
     Seq[V]

     SizeHint() (lower, upper int64)
}

```

[Go](#code:37)

This would mean that `slices.Collect` could be enhanced into something like this.

```go highlight
package slices

func Collect[E any](seq iter.Seq[E]) []E {
     var out []E
     if sized, ok := seq.(iter.SizedSeq[E]); ok {
       lower, _ := sized.SizeHint()
       out = make([]E, 0, lower)
     }

     for v := range seq {
       out = append(v)
     }
     return out
}

```

[Go](#code:38)

I don’t think there’s an easy way to patch this up, at this point. [↩︎](#fnref:6)

7. Disclaimer: I am not going to dig into Go’s rationale for rangefuncs. Knowing how the sausage is made, most big Go proposals are a mix of understandable reasoning and less reasonable veiled post-hoc justification to compensate for either Google planning/approvals weirdness or because the design was some principal engineer’s pony. This isn’t even a Go thing, it’s a Google culture problem. I say this as the architect of [Protobuf Editions](https://protobuf.dev/editions/overview), the biggest change to Protobuf since Rob’s misguided proto3[2](#fn:2) experiment. *I* have written this kind of language proposal, on purpose, because bad culture mandated it.

*The purpose of a system is what it does.* It is easier to understand a system by observing its response to stimuli, rather than what it says on the tin. So let’s use that lens.

Go wants to be easy to learn. It intended to replace C++ at Google (lol, lmao), which, of course, failed disastrously, because performance of the things already written in C++ is tied to revenue. They have successfully pivoted to being an easy-to-learn language that makes it easy to onboard programmers regardless of what they already use, as opposed to onboarding them to C++.

This does not mean that Go is user-friendly. In fact, user-friendliness is clearly not a core value. Rob and his greybeard crowd didn’t seem to care about the human aspect of interacting with a toolchain, so Go tooling rarely provides good diagnostics, nor did the language, until the last few years, try to reduce toil. After all, if it is tedious to use but simple, that does make it easy to onboard new programmers.

Rust is the opposite: it is very difficult to learn with a famously steep learning curve; however, it is very accessible, because the implementors have sanded down every corner and sharp edge using diagnostics, error messages, and tooling. C++ is neither of these things. It is very difficult to learn, and most compilers are pretty unhelpful (if they diagnose anything at all).

I think that Go has at least realized the language can be a pain to use in some situations, which is fueled in part by legitimate UX research. This is why Go has generics and other recent advanced language features, like being able to use the `for` syntax with integers or with custom iterators.

I think that rangefuncs are easy to learn in the way Go needs them to be. If you expect more users to want to write rangefuncs than users want to write complicated *uses* of rangefuncs, I think push iterators are the easiest to learn how to use.

I think this is a much more important reason for all the trouble that rangefuncs generate for the compiler and runtime than, say, compatibility with existing code; I have not seen many cases in the wild or in the standard library that conform to the rangefunc signatures. [↩︎](#fnref:7)
