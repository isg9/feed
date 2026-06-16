---
title: Default Methods in Go
url: /2025/08/25/go-default-methods
published: "2025-08-25T00:00:00Z"
feed: mcyoung
guid: /2025/08/25/go-default-methods
---

# Default Methods in Go

Go’s interfaces are very funny. Rather than being explicitly implemented, like in Java or Rust, they are simply a collection of methods (a “method set”) that the concrete type must *happen* to have. This is called structural typing, which is the opposite of nominal typing.

Go interfaces are very cute, but this conceptual simplicity leads to a lot of implementation problems (a theme with Go, honestly). It removes a lot of intentionality from implementing interfaces, and there is no canonical way to document that `A` satisfies[1](#fn:1) `B`, nor can you *avoid* conforming to interfaces, especially if one forces a particular method on you.

It also has very quirky results for the language runtime. To cast an interface value to another interface type (via the type assertion syntax `a.(B)`), the runtime essentially has to use reflection to go through the method set of the concrete type of `a`. I go into detail on how this is implemented [here](/2024/12/12/go-abi#codegen-for-interface-operations).

Because of their structural nature, this *also* means that you can’t add new methods to an interface without breaking existing code, because there is no way to attach default implementations to interface methods. This results in very silly APIs because someone screwed up an interface.

## [flag.Value is a Mess](\#flagvalue-is-a-mess)

For example, in the standard library’s package `flag`, the interface `flag.Value` represents a value which can be parsed as a CLI flag. It looks like this:

```go highlight
type Value interface {
  // Get a string representation of the value.
  String() string

  // Parse a value from a string, possibly returning an error.
  Set(string) error
}

```

[Go](#code:1)

`flag.Value` also has an optional method, which is only specified in the documentation. If the concrete type happens to provide `IsBoolFlag() bool`, it will be queries for determining if the flag should have bool-like behavior. Essentially, this means that something like this exists in the flag library:

```go highlight
var isBool bool
if b, ok := value.(interface{ IsBoolFlag() bool }); ok {
  isBool = b.IsBoolFlag()
}

```

[Go](#code:2)

The `flag` package already uses reflection, but you can see how it might be a problem if this interface-to-interface cast happens regularly, even taking into account Go’s caching of cast results.

There is also `flag.Getter`, which exists because they messed up and didn’t provide a way for a `flag.Value` to unwrap into the value it contains. For example, if a flag is defined with `flag.Int`, and then that flag is looked up with `flag.Lookup`, there’s no straightforward way to get the int out of the returned `flag.Value`.

Instead, you have to side-cast to `flag.Getter`:

```go highlight
type Getter interface {
  Value

  // Returns the value of the flag.
  Get() any
}

```

[Go](#code:3)

As a result, `flag.Lookup("...").(flag.Getter)` needs to do a lot more work than if `flag.Value` had just added `Get() any`, with a default return value of `nil`.

It turns out that there is a rather elegant workaround for this.

## [Struct Embeddings](\#struct-embeddings)

Go has this quirky feature called *embedding*, where a a field in a struct is declared without a name:

```go highlight
type (
  A int
  B struct{
    A
    Foo int
  }
)

```

[Go](#code:4)

The `A`-typed embedded field behaves as if we had declared the field `A A`, but selectors on `var b B` will search in `A` if they do not match something on the `B` level. For example, if `A` has a method `Bar`, and `B` does not, `b.Bar()` will resolve to `b.A.Bar()`. However, if `A` has a method `Foo`, `b.Foo` resolves to `b.Foo`, not `b.A.Foo`, because `b` has a field `Foo`.

Importantly, any methods from `A` which `B` does not already have will be added to `B`’s method set. So this works:

```go highlight
type (
  A int
  B struct{ A }
  C interface {
    Foo()
    Bar()
  }
)

func (A) Foo() {}
func (B) Bar() {}

var _ C = B{}  // B satisfies C.

```

[Go](#code:5)

Now, suppose that we were trying to add `Get() any` to `flag.Value`. Let’s suppose that we had also defined `flag.ValueDefaults`, a type that all satisfiers of `flag.Value` must embed. Then, we can write the following:

```go highlight
type Value interface{
  String() string
  Set(string) error

  Get() any  // New method.
}

type ValueDefaults struct{}

func (ValueDefaults) Get() { return nil }

```

[Go](#code:6)

Then, no code change is required for all clients to pick up the new implementation of `Get()`.

### [Required Embeds](\#required-embeds)

Now, this only works if we had required in the first place that anyone satisfying `flag.Value` embeds `flag.ValueDefaults`. How can we force that?

A little-known Go feature is that interfaces can have unexported methods. The way these work, for the purposes of interface conformance, is that exported methods are matched just by their name, but unexported methods must match both name and package.

So, if we have an interface like `interface { foo() }`, then `foo` will only match methods defined in the same package that this interface expression appears. This is useful for preventing satisfaction of interfaces.

However, there is a loophole: embedding inherits the *entire* method set, including unexported methods. Therefore, we can enhance `Value` to account for this:

```go highlight
type Value interface{
  String() string
  Set(string) error

  Get() any  // New method.

  value() // Unexported!
}

type ValueDefaults struct{}

func (ValueDefaults) Get() { return nil }
func (ValueDefaults) value() {}

```

[Go](#code:7)

Now, it’s impossible for any type defined outside of this package to satisfy `flag.Value`, without embedding `flag.ValueDefaults` (either directly or through another embedded `flag.Value`).

### [Exported Struct Fields](\#exported-struct-fields)

Now, another problem is that you can’t control the name of embedded fields. If the embedded type is `Foo`, the field’s name is `Foo`. Except, it’s not based on the name of the type itself; it will pick up the name of a type alias. So, if you want to unexport the defaults struct, you can simply write:

```go highlight
type MyValue struct {
  valueDefaults

  // ...
}

type valueDefaults = flag.ValueDefaults

```

[Go](#code:8)

This also has the side-effect of hiding all of `ValueDefaults`’ methods from `MyValue`’s documentation, despite the fact that exported and fields methods are still selectable and callable by other packages (including via interfaces). As far as I can tell, this is simply a bug in `godoc`, since this behavior is not documented.

### [What About Same-Name Methods?](\#what-about-same-name-methods)

There is still a failure mode: if a user type satisfying `flag.Value` happened to define a `Get` method with a different interface. In this case, that `Get` takes precedence, and changes to `flag.Value` will break users.

There are two workarounds:

1. Tell people not to define methods on their satisfying type, and if they do, they’re screwed. Because satisfying `flag.Value` is now explicit, this is not too difficult to ask for.

2. Pick a name for new methods that is unlikely to collide with anything.

Unfortunately, this runs into a big issue with structural typing, which is that it is very difficult to avoid making mistakes when making changes, due to the lack of intent involved. A similar problem occurs with C++ templates, where the interfaces defined by concepts are implicit, and can result in violating contract expectations.

Go has historically be relatively cavalier about this kind of issue, so I think that breaking people based on this is fine.

And of course, you cannot retrofit a default struct into a interface; you have to define it from day one.

## [Using Defaults](\#using-defaults)

Now that we have defaults, we can also enhance `flag.Value` with bool flag detection:

```go highlight
type Value interface{
  String() string
  Set(string) error

  Get() any
  IsBoolFlag() bool

  value() // Unexported!
}

type ValueDefaults struct{}

func (ValueDefaults) Get() { return nil }
func (ValueDefaults) IsBoolFlag() { return false }
func (ValueDefaults) value() {}

```

[Go](#code:9)

Now `IsBoolFlag` is more than just a random throw-away comment on a type.

We can also use defaults to speed up side casts. Many functions around the `io` package will cast an `io.Reader` into an `io.Seeker` or `io.ReadAt` to perform more efficient I/O.

In a hypothetical world where we had defaults structs for all the `io` interfaces, we can enhance `io.Reader` with a `ReadAt` default method that by default returns an error.

```go highlight
type Reader interface {
  Read([]byte) (int, error)
  ReadAt([]byte, int64) (int, error)

  reader()
}

type ReaderDefaults struct{}

func (ReaderDefaults) ReadAt([]byte, int64) (int, error) {
  return 0, errors.ErrUnsupported
}
func (ReaderDefaults) reader() {}

```

[Go](#code:10)

We can do something similar for `io.Seeker`, but because it’s a rather general interface, it’s better to keep `io.Seeker` as-is. So, we can add a conversion method:

```go highlight
type Reader interface {
  Seeker() io.Seeker

  // ...
}

type ReaderDefaults struct{}

func (ReaderDefaults) Seeker([]byte, int64) io.Seeker {
  return nil
}

```

[Go](#code:11)

Here, `Reader.Seeker()` converts to an `io.Seeker`, returning `nil` if that’s not possible. How is this faster than `r.(io.Seeker)`? Well, consider what this would look like in user code:

```go highlight
type MyIO struct{
  io.ReaderDefaults
  // ...
}

func (m *MyIO) Read(b []byte) (int, error) { /* ... */ }
func (m *MyIO) Seek(offset int64, whence int) (int64, error) { /* ... */ }

func (m *MyIO) Seeker() io.Seeker {
  return m
}

```

[Go](#code:12)

Calling `r.Seeker()`, if `r` is an `io.Reader` containing a `MyIO`, lowers to the following machine code:

1. Load the function pointer for `Seeker` out of `r`’s itab.
2. Perform an indirect jump on that function pointer.
3. Inside of `(*MyIO).Seeker`, load a pointer to the itab symbol `go:itab.*MyIO,io.Seeker` and `m` into the return slots.
4. Return.

The main cost of this conversion is the indirect jump, compared to, at minimum, hitting a hashmap lookup loop for the cache for `r.(io.Seeker)`.

Does this performance matter? Not for I/O interfaces, probably, but it *can* matter for some uses!

## [Shouldn’t This Be a Language Feature?](\#shouldnt-this-be-a-language-feature)

Yes, it should, but here we are. Although making it a language feature has a few rather unfortunate quirks that we need to keep in mind.

Suppose we can define defaults on interface methods somehow, like this:

```go highlight
type Foo interface{
  Bar()
  Baz()
}

func (f Foo) Baz() {
  // ...
}

```

[Go](#code:13)

Then, any type which provides `Bar()` automatically satisfies `Foo`. Suppose `MyFoo` satisfies `Foo`, but does not provide `Baz`. Then we have a problem:

```go highlight
var x MyFoo
x.Baz()       // Error!
Foo(x).Baz()  // Ok!

```

[Go](#code:14)

Now, we might consider looking past that, but it becomes a big problem with reflection. If we passed `Foo(x)` into `reflect.ValueOf`, the resulting `any` conversion would discard the defaulted method, meaning that it would not be findable by `reflect.Value.MethodByName()`. Oops.

So we need to somehow add `Baz` to `MyFoo`’s method set. Maybe we say that if `MyFoo` is ever converted into `Foo`, it gets the method. But this doesn’t work, because the compiler might not be able to see through something like `any(MyFoo{...}).(Foo)`. This means that `Baz` must be applied unconditionally. But, now we have the problem that if we have another interface `interface { Bar(); Baz(int) }`, `MyFoo` would need to receive incompatible signatures for `Baz`.

Again, we’re screwed by the non-intentionality of structural typing.

### [Missing Methods](\#missing-methods)

Ok, let’s forget about default method implementations, that doesn’t seem to be workable. What if we make some methods optional, like `IsBoolFlag()` earlier? Let’s invent some syntax for it.

```go highlight
type Foo interface {
  Bar()
  ?Baz() // Optional method.
}

```

[Go](#code:15)

Then, suppose that `MyFoo` provides `Bar` but not `Baz` (or `Baz` with the wrong signature). Then, the entry in the itab for `Baz` would contain a nil function pointer, such that `x.Baz()` panics! To determine if `Baz` is safe to call, we would use the following idiom:

```go highlight
if x.Baz != nil {
  x.Baz()
}

```

[Go](#code:16)

The compiler is already smart enough to elide construction of funcvals for cases like this, although it does mean that `x.Func` in general, for an interface value `x`, requires an extra `cmov` or similar to make sure that `x.Func` is nil when it’s a missing method.

All of the use cases described above would work Just Fine using this construction, though! However, we run into the same issue that `Foo(x)` appears to have a larger method set than `x`. It is not clear if `Foo(x)` should conform to `interface { Bar(); Baz() }`, where `Baz` is required. My intuition would be no: `Foo` is a strictly weaker interface. Perhaps it might be necessary to avoid the method access syntax for optional methods, but that’s a question of aesthetics.

This idea of having nulls in place of function pointers in a vtable is not new, but to my knowledge is not used especially widely. It would be very useful in C++, for example, to be able to determine if no implementation was provided for a non-pure virtual function. However, the nominal nature of C++’s virtual functions does not make this *as big* of a need.

### [Related Interfaces](\#related-interfaces)

Another alternative is to store a related interfaces’ itabs on in an itab. For example, suppose that we invent the syntax `A<-` within an `interface{}` to indicate that that interface will likely get cast to `A`. For example:

```go highlight
type (
  A interface{
    Foo()
  }

  B interface{
    Bar()

    A<-
  }
)

```

[Go](#code:17)

Satisfying `B` does not require satisfying `A`. However, the `A<-` must be part of public API, because a `interface{ Bar() }` cannot be used in place of an `interface{ A<- }`

Within `B`’s itab, after all of the methods, there is a pointer to an itab for `A`, if the concrete type for this itab also happens to satisfy `A`. Then, a cast from `B` to `A` is just loading a pointer from the itab. If the cast would fail, the loaded pointer will be `nil`.

I had always assumed that Go did an optimization like this for embedding interfaces, but no! *Any* inter-interface conversion, including upcasts, goes through the whole type assertion machinery! Of course, Go cannot hope to generate an itab for every possible subset of the method set of an interface (exponential blow-up), but it’s surprising that they don’t do this for embedded interfaces, which are Go’s equivalent of superinterfaces (present in basically every language with interfaces).

Using this feature, we can update `flag.Value` to look like this:

```go highlight
type Value interface {
  String() string
  Set(string) error

  BoolValue<-
  Getter<-
}

type BoolValue interface {
  Value
  IsBoolFlag() bool
}

type Getter interface {
  Value
  Get() any
}

```

[Go](#code:18)

Unfortunately, because `A<-` changes the ABI of an interface, it does not seem possible to actually add this to existing interfaces, because the following code is valid:

```go highlight
type A interface {
  Foo()
}

var (
  a A
  b interface{ Foo() }
)

a, b = b, a

```

[Go](#code:19)

Even though this fix seems really clean, it doesn’t work! The only way it could work is if PGO determines that a particular interface conversion `A` to `B` happens a lot, and updates the ABI of all interfaces with the method set of `A`, program-globally, to contain a pointer to a `B` itab if available.

## [Conclusion](\#conclusion)

Go’s interfaces are pretty bad; in my opinion, a feature that looks good on a slide, but which results in a lot of mess due to its granular and intention-less nature. We can sort of patch over it with embeds, but there’s still problems.

Due to how method sets work in Go, it’s very hard to “add” methods through an interface, and honestly at this point, any interface mechanism that makes it impossible (or expensive) to add new functions is going to be a huge problem.

Missing methods seems like the best way out of this problem, but for now, we can stick to the janky embedded structs.

---

1. Go uses the term “implements” to say that a type satisfies an interface. I am instead intentionally using the term “satisfies”, because it makes the structural, passive nature of implementing an interface clearer. This is also more in-line with interfaces’ use as generic constraints.

Swift uses the term “conform” instead, which I am avoiding for this reason. [↩︎](#fnref:1)
