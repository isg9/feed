---
title: Things You Never Wanted To Know About Go Interfaces
url: /2024/12/12/go-abi
published: "2024-12-12T00:00:00Z"
feed: mcyoung
guid: /2024/12/12/go-abi
---

# Things You Never Wanted To Know About Go Interfaces

Lately I’ve been finding myself writing a bit of Go, and I’ve picked up various fun “layout secrets” that help inform how I write code to minimize hidden allocations, and generally be kind to the optimizer. This article is a series of notes on the topic.

This post is about Go implementation details, so they can probably break you at any time if you rely on it. On the other hand, Hyrum’s law is a bitch, so taking your chances may not be that bad. After all, they’re probably never going to be able to properly clean up the mess people made with `//go:linkname` with runtime symbols…

As with many of my other posts, I’ll assume a basic familiarity with being able to read assembly. I’m using x86 for this post, but it’s worth looking at my [RISC-V post](/2021/11/29/assembly-1) for a refresher.

## [GC Shapes](\#gc-shapes)

The most basic Go-specific concept when it comes to type layouts is the *shape* of a type. This is an implementation detail of Go’s garbage collector that leaks through the `unsafe` package.

Like in most native programming languages, every Go type has a size (the number of bytes that type takes up in memory) and an alignment (a power of two that every pointer to that type must be divisible by). Go, like most other languages, requires that size be divisible by the alignment: that is, the size is equal to the stride of an array of that type.

The size an alignment of a type can be queried by the intrinsics [`unsafe.Sizeof`](https://pkg.go.dev/unsafe#Sizeof) and [`unsafe.Alignof`](https://pkg.go.dev/unsafe#Alignof). These are very unwieldy in generic code, so I like to define a couple of helpers[1](#fn:1):

```go highlight
func Size[T any]() int {
  var v T
  return int(unsafe.Sizeof(v))
}

func Align[T any]() int {
  var v T
  return int(unsafe.Alignof(v))
}

```

[Go](#code:1)

Together, these two quantities are called the *layout* of a type (a term common to many native languages). However, the *shape* of a type also records what pieces thereof contain *pointers*. This is because memory visible to the GC (such as globals, heap memory, or stack roots) is typed, and the GC needs to know which parts of those types are pointers that it needs to trace through.

Because all pointers have the same size and alignment (4 or 8 bytes depending on the system) the pointer words of a type can be represented as a bitset, one bit for every 4 or 8 bytes in the type. This, in fact, is the representation used by the GC[2](#fn:2).

In particular, this means that whether a field is to be interpreted as an `unsafe.Pointer` or as a `uintptr` is a static property of the type. As we will see when we discuss interfaces, this restriction prevents a few layout optimizations.

## [Slices and Strings](\#slices-and-strings)

Go is very public about the layout of slices and strings. A slice is

```go highlight
type slice[T] struct {
  data     *T
  len, cap int
}

```

[Go](#code:2)

`len` and `cap` are extracted by their eponymous builtins, and `data` can be obtained using `unsafe.SliceData` (or `&s[0]` if the slice is nonempty, but that costs a bounds-check).

A string has the same layout as a `[]byte`, except for a capacity:

```go highlight
type string struct {
  data *byte
  len  int
}

```

[Go](#code:3)

Despite essentially being slices, Go treats strings subtly differently. Strings are `comparable`, so they can be used as map keys. They are also immutable, which enables a handful of optimizations. Immutability is also why they are `comparable`: Go made the mistake of not keeping `const` from C, but they really want map keys to be `const`.

There is nothing stopping us from aliasing strings to data pointed to by a slice: after all, `strings.Builder` does it to avoid a copy in `String()`. We can implement this easily enough with some `unsafe`:

```go highlight
func StringAlias(data []byte) string {
  return unsafe.String(unsafe.SliceData(data), len(data))
}

```

[Go](#code:4)

Doing this is perfectly safe, so long as data is not mutated while the returned string is accessible. This allows virtually any slice type to be used as a key in a map, with some caveats.

1. Types which contain alignment padding cannot be used, because Go does not promise that it zeros memory returned by `new`.

2. Types which contain pointers will cause those pointers to become unreachable if the only reference is the aliased string; this is because the pointed to data’s shape contains no pointer words.

3. Incomparable types and interfaces will be compared by address (that is, maps, channels and funcs).

### [Dynamic Arrays with Reflection](\#dynamic-arrays-with-reflection)

Now, this isn’t the only to accomplish this: you can create dynamically-sized array types using reflection, like so:

```go highlight
func Slice2Array[T any](s []T) any {
  if s == nil { return nil }

  var v T
  elem := reflect.TypeOf(v)
  array := reflect.ArrayOf(len(s), elem)

  // NOTE: NewAt will return a reflect.Value containing a
  // pointer, not an array!
  refl := reflect.NewAt(array, unsafe.SliceData(s))
  refl = refl.Elem() // Dereference to get a pointer-to-array.
  return refl.Interface()
}

```

[Go](#code:5)

This will return an `any` whose type is `[len(s)]T`. You can even type assert it for static array sizes. This any is suitable for placing into a `map[any]T`, just as if we had built it with e.g. `any([...]byte("foo"))`

However, and this is not at all obvious from the code here, calling `refl.Interface()` will perform a copy of the whole array. `Interface()` delegates through a few functions until it calls `reflect.packEface()`.

The code this function ( [found here](https://cs.opensource.google/go/go/+/master:src/reflect/value.go;l=119?q=packEface&ss=go%2Fgo)) is reproduced below:

```go highlight
package reflect

// packEface converts v to the empty interface.
func packEface(v Value) any {
	t := v.typ()
	var i any
	e := (*abi.EmptyInterface)(unsafe.Pointer(&i))
	// First, fill in the data portion of the interface.
	switch {
	case t.IfaceIndir():
		if v.flag&flagIndir == 0 {
			panic("bad indir")
		}
		// Value is indirect, and so is the interface we're making.
		ptr := v.ptr
		if v.flag&flagAddr != 0 {
			c := unsafe_New(t)
			typedmemmove(t, c, ptr)
			ptr = c
		}
		e.Data = ptr
	case v.flag&flagIndir != 0:
		// Value is indirect, but interface is direct. We need
		// to load the data at v.ptr into the interface data word.
		e.Data = *(*unsafe.Pointer)(v.ptr)
	default:
		// Value is direct, and so is the interface.
		e.Data = v.ptr
	}
	// Now, fill in the type portion. We're very careful here not
	// to have any operation between the e.word and e.typ assignments
	// that would let the garbage collector observe the partially-built
	// interface value.
	e.Type = t
	return i
}

```

[Go](#code:6)

The switch determines precisely how the interface data pointer is computed. It turns out that (almost all) array types return true for `t.IfaceIndr()`, so the first case is selected, which triggers a copy (that being the call to `unsafe_New()` followed by a `typedmemmove`). This copy is to ensure that the value of the resulting interface cannot be mutated.

Now, if only we knew the layout of Go’s interfaces, we might be able to get somewhere here…

## [The Layout of Go’s Interfaces](\#the-layout-of-gos-interfaces)

Oh, yes, that’s what this article is about. So, if we look at the `runtime2.go` file in the runtime (yes, that’s what it’s called), nestled among the giant scheduler types for Gs, Ps, and Ms, we’ll find a couple of structs that really elucidate what’s going on:

```go highlight
package runtime

type funcval struct {
	fn uintptr
	// variable-size, fn-specific data here
}

type iface struct {
	tab  *itab
	data unsafe.Pointer
}

type eface struct {
	_type *_type
	data  unsafe.Pointer
}

```

[Go](#code:7)

`funcval` is the layout of a `func()`, more on that later. `iface` is the layout of your “usual” interface, consisting of an `itab` (an interface table, or what Go calls a vtable) and a pointer to some data. `eface` is the layout of `any` (the artist formerly known as `interface{}`, hence the name: *e* mpty inter *face*).

`eface` having its own layout is an optimization. Because `any` exists to be downcast from dynamically, storing the type directly cuts out a pointer load when doing a type switch on an `any` specifically. If we look at what an `itab` is (which is “just” an [`abi.ITab`](https://cs.opensource.google/go/go/+/refs/tags/go1.23.4:src/internal/abi/iface.go;l=14)):

```go highlight
package abi

// The first word of every non-empty interface type contains an *ITab.
// It records the underlying concrete type (Type), the interface type
// it is implementing (Inter), and some ancillary information.
//
// allocated in non-garbage-collected memory
type ITab struct {
	Inter *InterfaceType
	Type  *Type
	Hash  uint32     // copy of Type.Hash. Used for type switches.
	Fun   [1]uintptr // fun[0]==0 means Type does not implement Inter.
}

```

[Go](#code:8)

### [Codegen for Interface Operations](\#codegen-for-interface-operations)

An `ITab` contains the same type it would have as an `any`, which makes the generated code for a function that upcasts an interface to `any` very simple[3](#fn:3):

```go highlight
package foo

func Upcast(i MyIface) any {
  return i
}

```

[Go](#code:9)

```x86 highlight
foo.F:
    test    rax, rax
    jeq     nil
    mov     rax, [rax + 8]
nil:
    ret

```

[x86 Assembly](#code:10)

In the register ABI, the x86 argument (and return) registers are `rax`, `rbx`, `rcx`, `rdi`, `rsi`, `r8`, `r9`, `r10` and `r11` (with `rdx` reserved for passing a closure capture, more on that later; `r14` holds a pointer to the currently running G).

The `*ITab` comes in on `rax` and the data pointer on `rbx`. First, we need to check if this is the nil interface, identified by having a nil itab (or type, in the case of `any`). If it is nil, we just return: `rax:rbx` already contain the data of a nil `any`. Otherwise, we load `ITab.Type`, at offset 8, into `rax`, and return.

How do interface function calls work?

```go highlight
package foo

type MyIface interface {
  Method(int) int
}

func Call(m MyIface) int {
  return m.Method(42)
}

```

[Go](#code:11)

```x86 highlight
foo.Call:
    cmp     rsp, [r14 + 16]
    jls     grow
    push    rbp
    mov     rsp, rbp
    add     rsp, -16
    mov     [rsp], rax
    mov     [rsp + 8], rbx
    mov     rcx, [rax + 24]
    mov     rax, rbx
    mov     rbx, 42
    call    rcx
    add     rsp, 16
    pop     rbp
    ret
grow:
    nop
    mov     [rsp + 8], rax
    mov     [rsp + 16], rbx
    call    runtime.morestack_noctxt
    mov     rax, [rsp + 8]
    mov     rbx, [rsp + 16]
    jmp     foo.Call

```

[x86 Assembly](#code:12)

This function seems to be doing a lot more than it actually is. Part of it is that its prologue has to do a call to `runtime.morestack_noctxt()`, which is simply a call to `runtime.morestack` that clobbers `rdx`, the closure capture parameter. The meat of it comes when it loads `[rax + 24]`, the first element of `ITab.Fun`. It then moves the data pointer in `rbx` to `rax`, the argument into `rbx`, and issues the call.

What about upcasts? An upcast to a concrete type is quite simple: simply compare the type in the interface (either directly or in the `*ITab`) to a particular statically-known one. Downcasting to an interface (sometimes called a *sidecast*) is much more complicated, because it essentially requires a little bit of reflection.

```go highlight
package foo

type MyIface interface {
  Method(int) int
}

func Downcast(m any) MyIface {
  return m.(MyIface)
}

```

[Go](#code:13)

```x86 highlight
foo.Downcast:
    cmp     rsp, [r14 + 16]
    jls     grow
    push    rpb
    mov     rbp, rsp
    add     rsp, -24
    mov     [rsp], rax
    mov     [rsp + 8], rbx
    test    rax, rax
    jeq     nil
    mov     rcx, [foo..typeAssert0]
    mov     rdx, [rcx]
    mov     rsi, [rax + 16]
hashProbe:
    mov     rdi, rsi
    and     rsi, rdx
    shl     rsi, 4
    mov     r8, [rcx + rsi + 8]
    cmp     rax, r8
    jeq     found
    lea     rsi, [rdi + 1]
    test    r8, r8
    jnz     hashProbe
    mov     [rsp + 8], rbx
    mov     rbx, rax
    leq     rax, [foo..typeAssert0]
    call    runtime.typeAssert
    mov     rbx, [rsp + 8]
    jmp     done
found:
    mov     rax, [rcx + rsi + 16]
done:
    add     rsp, 24
    pop     rpb
    ret
nil:
    lea     rax, [type:foo.MyIface]
    call    runtime.panicnildottype
grow:
    // Same as it was in foo.Call above.
    jmp     foo.Downcast

```

[x86 Assembly](#code:14)

When we request an interface downcast, the Go compiler synthesizes a symbol of type `abi.TypeAssert`. Its definition is reproduced below.

```go highlight
package abi

type TypeAssert struct {
	Cache   *TypeAssertCache
	Inter   *InterfaceType
	CanFail bool
}
type TypeAssertCache struct {
	Mask    uintptr
	Entries [1]TypeAssertCacheEntry
}
type TypeAssertCacheEntry struct {
	// type of source value (a *runtime._type)
	Typ uintptr
	// itab to use for result (a *runtime.itab)
	// nil if CanFail is set and conversion would fail.
	Itab uintptr
}

```

[Go](#code:15)

The first thing this function does is check if `rax` contains 0, i.e., if this is a nil `any`, and panics if that’s the case (that’s a call to `runtime.panicnildottype`). It then loads `foo..typeAssert0`, a synthetic global variable containing an `abi.TypeAssert` value. It loads the `Cache` field, as well as the `Hash` field of the `abi.Type` attached to the `any`. It masks off the low bits using `typeAssert0.Cache.Mask`, and uses that to start probing the very simple open-addressed hash table located in `typeAssert0.Cache.Entries`.

If it finds a `TypeAssertCacheEntry` with the type we’re looking for (compared by address), we’ve found it. We load that entry’s `Itab` value into `rax` to change the value from being an `any` to being a `MyIface`, and we’re done.

If it finds a `TypeAssertCacheEntry` with a nil `Typ` pointer, we’re forced to hit the slow path, implemented at `runtime.typeAssert()`. This dynamically builds an itab by searching the method set of the type inside the `any`.

This then calls the reflection code in `runtime.getitab()`, which is what actually performs the messy search through the method set, comparing the names and signatures of methods with those in the interface, to produce an itab at runtime.

Then, it shoves this the resulting itab into the global itab cache, which is protected by a global lock! There are lots of scary atomics in this code. There are many places where this can potentially panic, bubbling up a type assertion failure to the user.

When `runtime.getitab()` returns, `runtime.typeAssert()` will *maybe* [4](#fn:4) update the type assertion cache, and return the new itab. This allows the code in our function to return directly, without needing to take another trip into the `hashProbe` loop.

In theory, PGO could be used to pre-fill the cache, but I couldn’t find any code in the compiler that indicates that this is something they do. In the meantime, you can optimize a hot type assert ahead of time by asserting to a known common type:

```go highlight
func DoSomething(r io.Reader) {
  var rs io.ReadSeeker
  if f, ok := r.(*os.File); ok {
    // Check for a known implementation first. This only costs
    // a pointer comparison with the *abi.Type in the itab.
    rs = f
  } else if f, ok := r.(io.ReadSeeker); ok {
    // Do an interface type assertion. This would eventually
    // learn os.File, but the branch above skips that "warmup"
    // time. It also lets the hardware branch predictor allocate
    // a prediction slot just for os.File.
    rs = f
  } else {
    // ...
  }
}

```

[Go](#code:16)

Type switches, incidentally, use a very similar caching mechanism for switches that include interface types among the cases.

### [What Was That About Indirect Interfaces?](\#what-was-that-about-indirect-interfaces)

Back when we were hacking arrays into existence with reflection, there was some trouble in `reflect.Value.Interface()`, where it would do a seemingly unnecessary copy.

This is because an interface’s data pointer must be a pointer. If you cram, say, an `int` into an `any`, Go will spill it to the heap. This is often called *boxing*, but the Go runtime refers to it as an “indirect interface”.

```go highlight
package foo

func Int2Any(x int) any {
  return x
}

```

[Go](#code:17)

```x86 highlight
foo.Int2Any:
  push     rbp
  mov      rbp, rsp
  add      rsp, -8
  call     runtime.convT64
  move     rbx, rax
  lea      rax, [type:int]
  add      rsp, 8
  pop      rbp
  ret

```

[x86 Assembly](#code:18)

Like many other managed languages, Go will skip boxing very small values by instead returning pointers into some global array.

Now, this boxing could be avoided: after all, an `int` is no larger than a pointer, so we could cram it into the data pointer field directly. However, the GC *really* doesn’t like that: the GC assumes it can trace through any pointer. Now, the GC *could* treat interfaces differently, and look at the type/itab pointer to determine if the data value pointer or a scalar. However, this would add significant complexity to both the representation of shapes, and to the tracing code in the GC, resulting in more branches and slower tracing.

However, if the type being wrapped in an interface happens to be a pointer, it can just use that pointer value directly.

```go highlight
package foo

func Int2Any(x int) any {
  return x
}

```

[Go](#code:19)

```x86 highlight
foo.Int2Any:
  move     rbx, rax
  lea      rax, [type:int]
  ret

```

[x86 Assembly](#code:20)

Any type that has the same shape as a pointer will be indirect. This includes maps, channels, and funcs. It also includes one element arrays of such types, such as `[1]*int` and `[1]chan error`, and single-field structs of such types. Curiously, this does not include structs which contain a zero-sized field before the pointer-sized field, even though those have the same shape as a pointer.

This means it’s generally not safe to play games with forging an interface out of a pointer to some type: whether that type is indirect in an interface is a subtle implementation detail of the compiler.

And of course, it’s important to remember that if you want to return a value by interface, you had best hope it can get inlined, so the compiler can promote the heap allocation to the stack.

## [Function Pointers](\#function-pointers)

The last thing to look at are Go’s function pointers. For the longest time, I assumed they had the same layout as an interface: a pointer to closure data, and a hardware function pointer.

It turns out the layout is weirder: let’s revisit the `runtime.funcval` we found in `runtime2.go` earlier.

```go highlight
package runtime

type funcval struct {
	fn uintptr
	// variable-size, fn-specific data here
}

```

[Go](#code:21)

This unusual layout is best understood by looking at the generated assembly.

```go highlight
package foo

func Call(
  f func(int) int,
  x int,
) int {
  return f(x)
}

```

[Go](#code:22)

```x86 highlight
foo.Call:
    cmp     rsp, [r14 + 16]
    jls     grow
    push    rpb
    mov     rpb, rsp
    add     rsp, -8
    mov     rcx, [rax]
    mov     rdx, rax
    mov     rax, rbx
    call    rcx
    add     rsp, 8
    pop     rbp
    ret
grow:
    // Same as before.
    jmp     foo.Call

```

[x86 Assembly](#code:23)

To call `f`, first we interpret it as a `*funcval` and load `f.fn` into a temporary. That is, the first word pointed to by `rax` (which holds `f` on function entry). Then, we place `f` in `rdx`, the closure context register. The reason for using this extra magic register will become clear shorter. Then, we arrange the rest of the arguments in their usual registers, and we jump to the address stored in `f.fn`.

Inside of `f`, captures are accessed by offsetting from `rdx`. What does one of those closures look like?

```go highlight
package foo

func Capture(x int) func(int) int {
  return func(y int) int {
    return x * y
  }
}

```

[Go](#code:24)

```x86 highlight
foo.Capture:
    cmp     rsp, [r14 + 16]
    jls     grow
    push    rpb
    mov     rpb, rsp
    add     rsp, -16
    mov     [rsp], rax
    lea     rax, ["type:noalg.struct { F uintptr; X0 int }"]
    call    runtime.newobject
    lea     rcx, foo.Capture.func1
    mov     [rax], rcx
    mov     rcx, [rsp]
    mov     [rax + 8], rcx
    add     rsp, 16
    pop     rbp
    ret
grow:
    // Same as before.
    jmp     foo.Capture

foo.Capture.func1:
    mov     rcx, [rdx + 8]
    imul    rax, rcx
    ret

```

[x86 Assembly](#code:25)

All `Capture` is doing is allocating a `funcval` with a single `int` capture; that’s the `{ F uintptr; X0 int }` in the code above. It then places the address of `Capture.func1`, which implements the callback, into `F`, and the argument of `Capture` into `X0`.

What about when returning a reference to a function? In that case, all that happens is it returns a reference to a global containing the address of the function.

```go highlight
package foo

func Capture(x int) func(int) int {
  return Id
}

func Id(x int) int {
  return x
}

```

[Go](#code:26)

```x86 highlight
foo.Capture:
    lea     rax, [foo.Id·f]
    ret

foo.Id:
    ret

foo.Id·f:
    .quad foo.Id

```

[x86 Assembly](#code:27)

Because we pass the closure arguments in an extra register not used by regular functions, we don’t need to create a thunk for this case.

Unfortunately, we do need to create a thunk for methods, even methods with a pointer receiver. This is because of the following incompatible constraints:

1. The receiver pointer for a method must point exactly to the value the method is called on. It can’t be a fixed offset before, because that would create an out-of-bounds pointer, which the GC does not tolerate.

2. The closure pointer must point to the start of the funcval, *not* its captures, because adjusting the pointer to point to the captures would cause it to point one-past-the-end of a value, which the GC *also* does not tolerate!

Thus, *even* if methods accepted a pointer receiver via `rdx`, closures and methods disagree about where that pointer should be passed.

Of course, there are adjustments we can make to fix this problem. For example, we could require that all `funcval` values have at least one capture. No-capture `funcvals` would have a synthetic `_ byte` field. This is not unlike how a non-empty struct whose final field is empty will be padded with an extra `_ byte` field: this is specifically to avoid a pointer to that field being a past-the-end pointer. The cost is that every non-capturing closure costs twice as much binary size.

Another fix is to make the GC blind to the pointer in `rdx`. This will never be the only pointer by which a value is reachable, so it would be safe to replace `mov rdx, rax` with a `lea rdx, [rax + 8]`. The GC would never know!

Until then, beware that writing `return foo.Method` secretly allocates 16 bytes or so. (Aside: I used to sit next to the Go team at Google, and I remember having a conversation with Austin Clements about this. Apparently I misremembered, because until recently I thought Go already implemented this optimization!)

## [Conclusion](\#conclusion)

If you made it this far this is probably you right now:

![Sunny, dizzy with register names.](/assets/images/stickers/dizzy.png)

This isn’t intended to be as polished as most of my articles, but there’s been enough things I’ve come across that I wanted to write this all up for my own reference.

---

1. `Sizeof` and `Alignof` are intrinsics, so the compiler will turn them into constants. However, they are only constants if the type being measured is not generic, so wrapping them in a function like this doesn’t actually hurt in generic code. [↩︎](#fnref:1)

2. Except for very large types that would have more words than can be recorded by an array of size [`abi.MaxPtrmaskBytes`](https://cs.opensource.google/go/go/+/refs/tags/go1.23.4:src/internal/abi/type.go;l=803;bpv=0). For larger types, we use GC programs! A GC program is an LZ-compressed bitset serving the same purpose as the pointer bitset most smaller types use. See [gcprog.go](https://cs.opensource.google/go/go/+/refs/tags/go1.23.4:src/cmd/internal/gcprog/gcprog.go).

In fact, `reflection` knows how to create programs on the fly for most types! See [reflect/type.go](https://cs.opensource.google/go/go/+/refs/tags/go1.23.4:src/reflect/type.go;l=2658). [↩︎](#fnref:2)

3. I will be writing assembly examples in Intel-syntax x86. Go’s assembly syntax is horrible and an impediment to the point I’m making. [↩︎](#fnref:3)

4. Maybe? Well, the cache will only get updated about 0.1% of the time. This is to amortize the costs of growing the cache. I assume they benchmarked this, and found that the cost of growing the cache makes it only worthwhile when that assertion is getting hammered. [↩︎](#fnref:4)
