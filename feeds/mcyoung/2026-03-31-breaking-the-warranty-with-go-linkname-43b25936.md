---
title: Breaking the Warranty with go:linkname
url: /2026/03/31/shame
published: "2026-03-31T00:00:00Z"
feed: mcyoung
guid: /2026/03/31/shame
---

# Breaking the Warranty with go:linkname

Let’s talk about linker symbols.

In C, when you write out a function signature, like this, you are informing the compiler that a symbol of appropriate ABI will exist at link time.

```c highlight
void* malloc(size_t bytes);

```

[C](#code:1)

You can prefix this with `extern`, but that’s technically not necessary and it irks me a little when people do it. You can also do this with globals, and in this case `extern` is required:

```c highlight
extern int somewhere_else;

```

[C](#code:2)

“Will exist at link time” means that you are promising the compiler that some other file contains the same signature, but as a *definition*, i.e., no `extern` and, in the case of a function, a body.

Of course there is no need for the declaration and definition to line up in ABI. That isn’t even checked at link time. And you can reach into whatever symbols you want in any library so long as they are not marked as `static`, because C has no notion of symbol visibility beyond a single translation unit (in C, this is a single `.c` file).

C++ also has this problem; a much scarier version of it, because C++ has complex symbol-mangling rules that incorporate the (approximate) type of a symbol into them, necessary because of overload resolution. You can `extern "C++"` other people’s symbols in C++, but it’s much harder.

Rust *also* has `extern`. To declare (but not define) a symbol, you use an `extern "Rust"` block. However, declaring symbols with mangled names in Rust is much, much harder, because Rust symbol names include the versions of crates that symbols come from in the form of a hash. This is because Rust allows multiple versions of the same crate to coexist in one linking artifact, and this hash disambiguates that in a package-manager agnostic fashion.

Go has `extern`, except it’s a funny magic comment because Go gets weird with it like that (I hate this).

In Go, you can declare a symbol like this:

```go highlight
//go:linkname foo_stub github.com/victim/pkg.foo
func foo_stub()

```

[Go](#code:3)

This will instruct the linker that `foo_stub` is a local alias for the symbol `github.com/victim/pkg.foo` – yes, Go symbols outside of the standard library really do look like that to the linker.

`foo_stub` can also have a body, in which case github.com/victim/pkg.foo must look like this:

```go highlight
//go:linkname foo
func foo()

```

[Go](#code:4)

This means that this package is expecting some *other* package to define one of its symbols for it.

Now, this is a huge problem, because in the former variant, there is no requirement that the victim package consents to having its symbol table rummaged in. Go only addressed this in Go 1.23, where now, for standard library packages only, only symbols explicitly opted into `linkname` can be used like this by non-standard library packages.

The upshot is phew, we solved this problem, why am I writing about it.

Well – because it left behind some lovely official-unofficial APIs.

> I am going to be very snarky. I am being snarky because: (a) I am friends with the people who are thanklessly cleaning up this mess and (b) I do *gestures* *exasperatedly at programming languages* for a living.
>
> I am like this about every programming language, as my friends who work on rustc can tell you.

## [The Hall of Shame](\#the-hall-of-shame)

When closing the linkname hole, the Go project surveyed existing modules with 50+ dependents that were very naughty and used runtime symbols inappropriately, and they compiled a list of all such symbols.

These symbols have a trivial `linkname` comment allowing them to continue to be `linkname` d without breaking anyone, along with very stern admonitions and a list of notable perpetrators. You can find all of these comments by [searching for the admonition](https://cs.opensource.google/search?q=%22Notable%20members%20of%20the%20hall%20of%20shame%20include%22&ss=go%2Fgo).

In practice, this means these symbols will be supported forever and ever to prevent wide ecosystem breakage.

But writing the `linkname` stubs is very error prone. You have to match the signature exactly, but not so exactly that you use the names of unexported types. So you need to know enough about Go’s ABI to know what you can change. You also probably want to mark some things as `//go:noescape`. Escape analysis conservatively assumes that body-less functions always escape their arguments. This assumption can be lifted with this *other* magic comment.

Knowing which functions are safe to `//go:noescape` is finicky business. That’s where I come in.

I wrote a [fun little library](https://github.com/mcy/shame) which uses the admonition to automatically generate bindings for every naughty `linkname` in the standard library.

I did this for a few reasons.

1. I have a love-hate relationship with Go. The language itself is gross and warty but the runtime is very fun. I file [weird bugs](https://github.com/golang/go/issues?q=is%3Aissue%20state%3Aopen%20author%3Amcy) against it.

2. A flavor of harm reduction. Not documenting the naughty things you could do leads to people who are *not* compiler nerds into reverse engineering how language implementation details without knowing why, and wind up hurting themselves. I would rather there be something well-documented they can reach for instead.

3. A few of the core Go maintainers are friends of mine, and I kind of wonder if they will be more amused or annoyed that I did this…

## [Notable Members](\#notable-members)

The top-level package `shame` provides nicely-named, documented wrappers to the most useful of the bindings (in my opinion). Most have to do with low-level memory operations. There *are* scheduler stubs, but I don’t provide wrappers for those because attempting to interact directly with the scheduler is like sneaking up on a skittish mule from behind.

The main non-wrapper thing is `*shame.Type`, which is just a wrapper over `*internal/abi.Type`. The latter actually has survived leaking out into the real world[1](#fn:1), but most of the interesting operations require one.

You can easily get one from either an `any` of that type, or a `reflect.Type`, by either ripping the type pointer out of the `any` (which is literally an `*abi.Type`), or the data pointer out of the `reflect.Type`. `*shame.Type` organizes these operations into a type, to bridge the unchecked but fast operations of `shame` with the safe but slow `reflect` package.

### [Direct Allocator Calls](\#direct-allocator-calls)

For me, the two most interesting wrapper functions are `shame.Alloc` and `shame.Grow`.

`shame.Alloc` directly calls `runtime.mallocgc`, which allocates from the heap. It takes a size, a type (which is used to get a GC shape for the allocation) and whether or not to zero the result. What’s interesting is that Go provides *no way to get unzeroed memory*, by design: it is very difficult to use such memory safely. In fact, `mallocgc` will still require that memory which will contain pointers is zeroed, since otherwise the GC could get very confused.

```go highlight
ptr := shame.TypeFor[unsafe.Pointer]()
buf := shame.Alloc(8*n, ptr, true)

```

[Go](#code:5)

The other interesting bit, of course, is that you can allocate memory of arbitrary shape without needing to go through reflection. This is… less immediately interesting than being able to dodge the zeroing cost.

The latter is the slice resizing primitive that `append` can call: `runtime.growslice`. This is Go’s version of `realloc`, which takes a pointer, a size, and a desired new size, resizing the underlying storage if necessary and copying elements into the new buffer.

The Go version is much more powerful. It will only copy elements within a specific prefix of the buffer, and you only tell it how many elements you want to add; it returns the new capacity, which snaps to the next allocator size class. `append` will show this behavior: `make([]T, 0, n)` always has a capacity of `n`, but `append([]T(nil), make([]T, n)...)` may have a larger capacity.

```go highlight
// Allocate 56 (rounded up to a size class) bytes of
// uninitialized memory.
uninit := shame.Grow(shame.Slice{}, shame.TypeFor[byte](), 56)

```

[Go](#code:6)

Like `shame.Alloc`, `shame.Grow` will return uninitialized memory in some cases. This is because calls to `runtime.growslice` generated by the compiler are followed up by an appropriate call to `runtime.memmove` to add the new elements, so zeroing that region, if it is pointer-free, is wasteful.

While `shame.Alloc` does not provide large benefits over `reflect.New`, `shame.Grow` is much more powerful than `reflect.AppendSlice`. This is because `reflect.AppendSlice` does offer an equivalent to the `append(xs, make([]T, n)...)` idiom (i.e., what `slices.Grow` does).

The compiler recognizes the latter pattern and instead of rewriting it into `runtime.growslice` followed by `runtime.memmove`, it follows it up with a call to `runtime.memclrNoHeapPointers` instead (if `T` is pointer-free; if `T` has pointers, the memory is zeroed automatically by the allocator).

`shame.Alloc` also helps avoid having to interact with `runtime.growslice`’s weird ABI (the ABI is the way it is for the benefit of register allocation).

### [Heap Queries](\#heap-queries)

There is some limited functionality for asking the GC about pointers. One important feature of Go’s GC is that it can handle *internal pointers*: given any pointer the GC knows about, it can calculate the base address for the corresponding GC allocation, as well as the corresponding GC memory page.

`shame.FindObject` gives access to this feature. You can use it to stick finalizers on any random pointer, or, given a pointer into a slice allocated by `make` or `append`, the base address of the slice.

A related function used to implement `-d=checkptr` can be used to determine if a pointer came from the stack. `shame.IsStackPointer` offers this functionality.

### [Dynamic Map Operations](\#dynamic-map-operations)

You can already perform dynamic operations on maps using reflection, but they can trigger unnecessary allocations. But someone `linknamed` every single map shim, so we can use them directly.

You can also get access to pointers *into* a map, allowing for much faster RMW operations.

```go highlight
func upsert[V any](m map[string]V, k string, v ...V) []V {
  p := (*[]V)(shame.MapAssignString(shame.TypeOf(m), shame.MapData(m), k))
  *p = append(*p, v...)
  return *p
}

```

[Go](#code:7)

The `MapAssign*` family of functions will look up a value, or allocate a fresh entry for it, and return a pointer to that value for the caller to mutate. Unfortunately, the API will not tell you if the returned value was freshly allocated or not.

The `MapIndex*` family of functions can be used to obtain pointers to values in a map, which makes it easy to avoid the cost of copying a large value of a map and then copying it back in (in normal-people Go, this is achieved by using `map[K]*V` instead).

And of course, these functions, along with `shame.MapDelete` and `shame.NewMapIter`, allow for dynamic map operations without the potential escape-to-heap costs of reflection.

### [Data Race Generator](\#data-race-generator)

There is one other fun API: `shame.ChanIndex`. This will extract the `n` th element from the ring buffer inside of a channel. What does this mean, exactly?

Every Go channel is secretly a ring buffer with a mutex in front of it plus a lockless list for holding waiters. Sending to a channel locks the mutex, pushes to the list, and unlocks the mutex. Receiving pulls from the list. It’s a little fussier than that because of the need to handle waiters: sending on an empty channel wakes some goroutine blocked on it, and sending on a full channel blocks until a goroutine receives.

`shame.ChanIndex` doesn’t care about any of that. It just indexes into the buffer that backs the linked list without looking at the lock or anything else. This is exactly as useful as you think: it gives you an approximately random value drawn from the recent sends on the channel, potentially racing with a receiving goroutine if you decide to write to that pointer.

I included this API because I think it’s very funny someone `linkname` d this.

## [Should I Use It?](\#should-i-use-it)

Lol, no. This is about harm reduction. If you were already going to do this, this makes the thing you were going to do much less dangerous, but it’s still a poor life decision.

I actually didn’t know the full breadth of exactly what had survived the `linkname` purge of 2024 before I did this, so I’m kinda glad I did. It was also fun tracking down the uses that lead to this, because you might of many of those libraries “oh, I bet they had a really good reason”. The answer will often disappoint you. :P

If I were in the Go project’s shoes (and you should be glad I’m not) I would have just broken everyone. This kind of reaching into internal APIs needs to be punished, or else the inexorable march of Hyrum’s law will take you to the dark place C++ has been in for the last decade, where they are incapable of upgrading basic algorithm implementations because of ABI promises Microsoft made for them.

And I say this as a serial abuser of this kind of thing. Go has been making a lot of strides in being serious about performance. I’ve spent a lot of time giving the project serious feedback on their very bold SIMD intrinsics effort, and they take my missed optimization bugs seriously[2](#fn:2).

I mean, use my library. I will gladly help you come to terms with your mistakes after you break something. :)

---

1. Well, you can do things like this, but it really gets into “keep both pieces” territory…

```go highlight
import "reflect"

var ptrbytes, _ = reflect.ValueOf(reflect.TypeFor[int]()).
     Elem().
     Field(0).Type.
     FieldByName("PtrBytes")

// Pointers emulates internal/abi.Type.Pointers.
func Pointers(t reflect.Type) bool {
     return reflect.ValueOf(t).Elem().Field(0).Field(ptrbytes.Index[0]).Uint() != 0
}

```

[Go](#code:8)

```
[↩︎](#fnref:1)
```
2. Beyond Google’s foolish personnel policies crushing their ability to get useful things done and leniency with the peanut gallery on the issue tracker making me reluctant to do anything less esoteric than filing escape analysis bugs… [↩︎](#fnref:2)
