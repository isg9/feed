---
title: std::tuple the Hard Way
url: /2022/07/13/tuples-the-hard-way
published: "2022-07-13T00:00:00Z"
feed: mcyoung
guid: /2022/07/13/tuples-the-hard-way
---

# std::tuple the Hard Way

Let’s talk about C++ templates.

C++ is famous for relegating important functionality often built into the language to its standard library[1](#fn:1). C++11 added a number of very useful class templates intended to make generic programming easier. By far the most complicated is `std::tuple<>`, which is literally just a tuple.

It turns out that implementing `std::tuple<>` is complicated. [Very, very complicated.](https://github.com/llvm/llvm-project/blob/main/libcxx/include/tuple)

Naively, we think that we can just splat a variadic pack into a struct:

```cpp highlight
template <typename... Types>
class tuple {
  Types... values;
};

```

[godbolt](https://godbolt.org/clientstate/eyJzZXNzaW9ucyI6W3siZXhlY3V0b3JzIjpbeyJjb21waWxlciI6eyJpZCI6ImNsYW5nX3RydW5rIiwib3B0aW9ucyI6IiJ9LCJjb21waWxlck91dHB1dFZpc2libGUiOnRydWUsImNvbXBpbGVyVmlzaWJsZSI6ZmFsc2V9XSwiaWQiOjEsImxhbmd1YWdlIjoiYysrIiwic291cmNlIjoidGVtcGxhdGUgXHUwMDNjdHlwZW5hbWUuLi4gVHlwZXNcdTAwM2VcbmNsYXNzIHR1cGxlIHtcbiAgVHlwZXMuLi4gdmFsdWVzO1xufTtcbiJ9XX0=) [C++](#code:1)

If you click through to Godbolt, you’ll see it doesn’t: this feature doesn’t exist in C++[2](#fn:2) (normally, you’d do `std::tuple<Types...>`, but we need to write down `std::tuple` somehow). The usual approach is to use some kind of recursive template, which can tend to generate a lot of code.

However, C++ does actually have tuples built into the language, as a C++11 feature… lambdas! As an extra challenge, we’re going to try to minimize the number of templates that the compiler needs to instantiate; `std::tuple` is famously bad about this and can lead to very poor build performance.

For our tuple library type, we need to solve the following problems:

- How do we implement `std::tuple()` and `std::tuple(args...)`?
- How do we implement `std::apply`?
- How do we implement `std::tuple_element`?
- How do we implement `std::get`?

## [The Power of \[\](){}](\#the-power-of-)

Alright, let’s back up. In C++11, we got *lambdas*, which are expressions that expand to anonymous functions. In C++, lambdas are *closures*, meaning that they capture (“close over”) their environment.

This is a lambda in action:

```cpp highlight
int x = 5;
auto add = [x] (int y) { return x + y; }
int z = add(8);  // 13

```

[C++](#code:2)

The `[x]` syntax is the *captures*. To represent a lambda, C++ creates an anonymous, one-time-use class. It has the captures as members (whether they be references or values) and provides the necessary `operator()`. In other words, this is approximately the desugaring:

```cpp highlight
auto MakeFn(int x) {
  return [x] (int y) {
    return x + y;
  };
}

```

[C++](#code:3)

```cpp highlight
auto MakeFn(int x) {
  struct _Lambda {
    auto operator()(int y) const {
      return x + y;
    };
    const int x;
  }
  return _Lambda{x};
}

```

[C++](#code:4)

Note the `const` s in `_Lambda`. By default, captured values are stored inline but marked `const`, and the `operator()` member is also `const`. We can remove that specifier in both location with the `mutable` keyword:

```cpp highlight
auto MakeFn(int x) {
  return [x] (int y) mutable {
    return x + y; // ^^^^^^^
  };
}

```

[C++](#code:5)

```cpp highlight
auto MakeFn(int x) {
  struct _Lambda {
    auto operator()(int y) {
      return x + y;
    };
    int x;
  }
  return _Lambda{x};
}

```

[C++](#code:6)

Lambdas can capture anything from their scope. In addition to values, they will capture any types visible from that location. This means that, if constructed in a function template, the generated class will effectively capture that template’s arguments. Thus:

```cpp highlight
template <typename... Args>
auto CaptureMany(Args... args) {
  return [args...] () { /*whatever*/ };
};

```

[C++](#code:7)

This will create a new anonymous class capturing an arbitrary number of arguments, depending on the *parameters passed to `CaptureMany()`*. This will form the core of our tuple type.

Now, let’s stick it into a class.

## [Lambda-Typed Data Members](\#lambda-typed-data-members)

We don’t want to leak the lambda into the template parameters of our tuple class, so we need it to be strictly in terms of the class’s template parameters. This is straightforward with `decltype`.

```cpp highlight
template <typename... Types>
class Tuple {
 private:
  decltype(TupleLambda(Types{}...)) lambda_;
};

```

[C++](#code:8)

Regardless of what our C++ compiler calls the type, we are able to use it as a field. However, a problem arises when we try to write down the main “in-place” constructor, which consists of the usual forwarding-reference and `std::forward` boilerplate[3](#fn:3):

```cpp highlight
template <typename... Types>
class Tuple {
 public:
  template <typename... Args>
  Tuple(Args&&... args) : lambda_(
    TupleLambda(std::forward<Args>(args)...)) {}
  // ...
};

```

[C++](#code:9)

The initialization for lambda\_ doesn’t work, because the return type of `TupleLambda` is wrong! The compiler is required to synthesize a new type for every specialization of `TupleLambda`, and so `TupleLambda<Types...>()` and `TupleLambda<Args...>` return different types!

### [A `new` Kind of Initialization](\#a-new-kind-of-initialization)

This requires a major workaround. We’d still like to use our lambda, but we need to give it a type that allows us to construct it before calling the constructors of `Types...`. We can’t use `Types...`, so we’ll do a switcheroo.

The following is boilerplate for a type that can hold a `T` in it but which can be constructed before we construct the `T`.

```cpp highlight
template <typename T>
class alignas(T) StorageFor {
 public:
  // Constructor does nothing.
  StorageFor() = default;

  // Constructs a T inside of data_.
  template <typename... Args>
  void Init(Args&&... args) {
    new (reinterpret_cast<T*>(&data_)) T(
      std::forward<Args>(args)...);
  }

  // Allow dereferencing a StorageFor into a T, like
  // a smart pointer.
  const T* get() const { return reinterpret_cast<const T*>(&data_); }
  T* get() { return reinterpret_cast<T*>(&data_); }
  const T& operator*() const { return *get(); }
  T& operator*() { return *get(); }
  const T* operator->() const { return get(); }
  T* operator->() { return get(); }
 private:
  char data_[sizeof(T)];
};

```

[C++](#code:10)

There’s a lot going on here. Let’s break it down.

1. `alignof(T)` ensures that even though the only member is a `char` array, this

```rust highlight
pub fn read(x: &i32) -> i32 {
    *x
}

```

[Rust](#code:11)

2. The constructor does nothing; the `T` within is only constructed when `Init()` is called with `T`’s constructor arguments.

3. `Init()` forwards its arguments just like our non-functional constructor for `Tuple`. This time, the arguments get sent into `T`’s constructor via *placement- `new`*. Placement- `new` is special syntax that allows us to call a constructor directly on existing memory. It’s spelled like this: `new (dest) T(args);`.

4. `operator*`/ `operator->` turn `StorageFor` into a smart pointer over `T`, which will be useful later. The signatures of these functions aren’t important; it’s library boilerplate.

We can use this type like this:

```cpp highlight
// Create some storage.
StorageFor<std::string> my_string;

// Separately, initialize it using std::string's constructor
// form char[N].
my_string.Init("cool type!");

// Print it out.
absl::PrintF("%s\n", *my_string);

// Destroy it. This must be done manually because StorageFor<T>
// has a trivial destructor.
using ::std::string;
my_string->~string();

```

[C++](#code:12)

How does this help us?

### [Constructors Inside-Out](\#constructors-inside-out)

`StorageFor<T>` will be the types that our lambda captures, making it possible to give it a consistent type without knowing which arguments we’ll use to initialize the contents.

```cpp highlight
template <typename... Types>
class Tuple {
 private:
  template <typename... Args>
  static auto TupleLambda(Args... args) {
    return [args...] { /* ??? */ };
  }

  decltype(TupleLambda(StorageFor<Types>{}...)) lambda_ =
    TupleLambda(StorageFor<Types>{}...);
};

```

[C++](#code:13)

But now we’re in another bind: how do we *call* the constructors? Even with placement-new, we can’t reach into the lambda’s data, and the layout of a lambda is compiler-specific. However, that’s from the outside. What if we accessed the lambda from the *inside*?

We modify the lambda to itself be generic and take a pack of forwarding references as arguments, which we can then pass into `Init()`:

```cpp highlight
template <typename... Types>
class Tuple {
 private:
  template <typename... Args>
  static auto TupleLambda(Args... args) {
    return [args...] (auto&&... init_args) {
      (args.Init(std::forward<decltype(init_args)>(init_args)), ...);
    };
  }
  // ...
}

```

[C++](#code:14)

That’s a serious mouthful. Let’s break it down.

1. `[args...] (auto&&... init_args) {` declares a *generic* lambda. This means that there’s an imaginary `template <typename... Args>` on the `operator()` of the generated class. Because the argument type is `Args&&`, and `Args` is a template parameter of `operator()`, `init_args` is a pack of forwarding references. This is a C++14 feature.

2. `Init(std::forward<decltype(init_args)>(init_args))` is a forwarded constructor argument. Nothing new here.

3. The outer `(<expr>, ...)` that the placement- `new` is wrapped in is a *pack* *fold*, which uses an operator to fold a pack of values into one. For example, `(foo + ...)` computes the sum of all elements in a pack. In our case, we’re folding with the comma operator `,`. All this does is discard the elements of the pack (which are all `void`, regardless). This is a C++17 feature[4](#fn:4)

Taken together, this causes the constructor of each type in `Types...` to be run on the respective `StorageFor<T>` captures by the lambda when `TupleLambda()` was originally called. The double-nesting of a function-within-a-function can be a bit confusing: `TupleLambda()` is not what calls `T`’s constructor!

Actually, this won’t compile because `Init()` is not `const`, but the lambda’s `operator()` is. This is easily fixed by adding the `mutable` keyword:

```cpp highlight
template <typename... Types>
class Tuple {
 private:
  template <typename Args>
  static auto TupleLambda(Args... args) {
    return [args...] (auto&&... init_args) mutable {
      // ...                               ^^^^^^^
    };
  }
  // ...
}

```

[C++](#code:15)

We also need to mark the `lambda_` parameter as `mutable` so that `const` functions can all it. We’ll just need to be careful we don’t actually mutate through it. This is necessary because we cannot (at least until C++23) write to the captures of a lambda and still be able to call it in `const` contexts:

```cpp highlight
template <typename... Types>
class Tuple {
 private:
  mutable decltype(TupleLambda(StorageFor<Types>{}...)) lambda_ =
    TupleLambda(StorageFor<Types>{}...);
  // ...
};

```

[C++](#code:16)

Now, our constructor looks like this:

```cpp highlight
template <typename... Types>
class Tuple {
 public:
  template <typename... Args>
  Tuple(Args&&... args) {
    lambda_(std::forward<Args>(args)...);
  }
  // ...
}

```

[C++](#code:17)

### [More Constructors!](\#more-constructors)

We have `std::tuple(args)` but we still need `std::tuple`. But, we’ve already used up our one chance to touch the captures of the lambda… we can’t write down a lambda that has both a variadic `operator()` (many generic arguments) and a niladic `operator()` (no arguments).

But we can make it take a lambda itself! In this case, all that our “storage lambda” does now is call a callback with a pack of references. Calling `lambda_()` effectively “unpacks” it:

```cpp highlight
template <typename... Types>
class Tuple {
 private:
  template <typename Args>
  static auto TupleLambda(Args... args) {
    return [=] (auto callback) mutable -> decltype(auto) {
      return callback(args...);
    };
  }
  // ...
}

```

[C++](#code:18)

The `decltype(auto)` bit simply ensures that if `callback` returns a reference, then so does `lambda_`. By default, lambdas return `auto`, which will never deduce a reference (you’d need to write `auto&`, which conversely cannot deduce a value). Instead of using “ `auto` deduction”, we can use the special `decltype(auto)` type to request “ `decltype` deduction”, which *can* deduce both references and non-references. This comes in handy later.

Now we can refactor the two constructors to call `lambda_` with different lambda arguments. Our original constructor will pass in the original body of `lambda_`, which calls Init() with `args`. The new constructor will simply call `Init()` with no args.

```cpp highlight
template <typename... Types>
class Tuple {
 public:
  template <typename... Args>
  Tuple(Args&&... args) {
    lambda_([&] (StorageFor<Types>&... places) {
      (places.Init(std::forward<decltype(args)>(args)), ...);
    });
  }
  Tuple() {
    lambda_([] (StorageFor<Types>&... places) {
      (places.Init(), ...);
    });
  }
  // ...
}

```

[C++](#code:19)

We need to implement the destructor too, since `StorageFor<T>` will not destroy the `T` we’re squirreling away inside, but this is still really easy:

```c++ highlight
template <typename... Types>
class Tuple {
 public:
  ~Tuple() {
    lambda_([] (StorageFor<Types>&... places) {
      (places->~Types(), ...);
    });
  }
  // ...
}

```

[C++](#code:20)

Copy and move are similar, but require interleaving two calls of `lambda_`:

```cpp highlight
template <typename... Types>
class Tuple {
 public:
  Tuple(const Tuple& that) {
    lambda_([&] (StorageFor<Types>&... these) {
      // Carefully take a const&, to make sure we don't call a
      // mutable-ref constructor.
      that.lambda_([&] (const StorageFor<Types>&... those) {
        (new (these.get()) Types(*those), ...);
      });
    });
  }

  Tuple(Tuple&& that) {
    lambda_([&] (StorageFor<Types>&... these) {
      that.lambda_([&] (StorageFor<Types>&... those) {
        // Avoid std::move to cut down on instantiation.
        (new (these) Types(static_cast<Types&&>(*those)), ...);
      });
    });
  }
  // ...
};

```

[C++](#code:21)

Copy/move assignment are basically identical; I’ll leave those as an exercise!

This gives us our complete set of constructors. We’ll throw in deduction guides[5](#fn:5) to avoid needing to implement `make_tuple`:

```cpp highlight
template <typename... Types>
Tuple(Types...) -> Tuple<Types...>;
template <typename... Types>
Tuple(const Tuple<Types...>&) -> Tuple<Types...>;

int main() {
  Tuple tup{1, 2, "foo", "bar"};
  Tuple tup2 = tup;
}

```

[C++](#code:22)

This works up until we try to write `Tuple tup2 = tup;` Overload resolution will incorrectly route to the variadic constructor rather than the copy constructor, so a little bit of SFINAE is needed to grease the compiler’s wheels.

Keeping in the spirit of avoiding extra instantiation logic, we’ll use placement- `new` inside of a `decltype` as an ersatz `std::enable_if`:

```cpp highlight
template <typename... Args,
          decltype((new (nullptr) Types(std::declval<Args>()), ...))
            = nullptr>
Tuple(Args&&... args) {
  // ...
}

```

[C++](#code:23)

This verifies that we can actually construct a `Types` from a `Args` (for each member of the pack). Because this is occurring in an unevaluated context, we can safely placement- `new` on `nullptr`. All `new` expressions produce a pointer value, and a comma-fold produces the last value in the fold, so the overall `decltype()` is `T*`, where `T` is the last element of the pack.

This `decltype()` is the type of a non-type template parameter, which we can default to `nullptr`, so the user never notices it.

Ok. We have all of our constructors. The code so far is at this footnote: [6](#fn:6).

Onwards to `std::apply`.

## [Unpacking, Again](\#unpacking-again)

`std::apply(f, tup)` is a relatively straight-forward function: call `f` by splatting `tup`’s elements int `f` as a pack. Because of how we’ve implemented `lambda_`, this is actually super simple:

```cpp highlight
template <typename... Types>
class Tuple {
 public:
  template <typename F>
  decltype(auto) apply(F&& f) {
    lambda_([&] (StorageFor<Types>&... places) -> decltype(auto) {
      return std::invoke(std::forward<F>(f), *places...);
    });
  }
  // ...
};

```

[C++](#code:24)

(We’re possibly returning a reference, so note the `decltype(auto)` s.)

`lambda_` is basically a funny `std::apply` already, just with the wrong arguments. The `*places` fixes this up. With some repetition, we can write down `const`\- and `&&`-qualified overloads. We can even introduce a free function just like the one in the standard library:

```cpp highlight
template <typename F, typename Tup>
decltype(auto) apply(F&& f, Tup&& t) {
  return std::forward<Tup>(t).apply(std::forward<F>(f));
}

```

[C++](#code:25)

The other unpacking operation, `std::get`, is trickier. This is usually where things get really hairy, because we need to get the `i` th type out of the lambda. There are many approaches for doing this, most of which involve recursive templates. I’ll present two approaches that don’t use recursive templates directly, but which can still be a bit slow, built-time-wise.

This is the function we need to implement:

```cpp highlight
template <typename... Types>
class Tuple {
 public:
  template <size_t i>
  auto& get();
  // ...
};

```

[C++](#code:26)

### [Cheating with `std::make_index_sequence`](\#cheating-with-stdmake-index-sequence)

`std::make_index_sequence` is a funny type-level function that produces a pack of integers from `0` to `i`, given just `i`. This is usually fast, since most compilers will have intrinsics for doing it without needing to instantiate `i` templates. For example, in Clang, this is `__make_integer_seq`, which is used by [libc++](https://github.com/llvm/llvm-project/blob/main/libcxx/include/__utility/integer_sequence.h).

Thus, we can turn the problem of implementing `get` with a single `i` to implementing `get` with a pack:

```cpp highlight
template <typename... Types>
class Tuple {
 public:
  template <size_t i>
  auto& get() {
    return GetImpl(std::make_index_sequence<i>{});
  }
 private:
  template <size_t... less_than_i>
  /* ??? */ GetImpl(std::index_sequence<less_than_i...>);
  // ...
};

```

[C++](#code:27)

We can then use this pack to cook up just the right lambda to grab just the capture we want out of `lambda_`. Specifically, we want a lambda that picks out its `i` th argument. Basically we want to write something with arguments like `(auto..., auto, auto...)`, but somehow use the `less_than_i` pack to control the size of the first argument pack.

We can whip up a class template for this:

```cpp highlight
template <size_t>
struct Sink {
  template <typename T>
  Sink(T&&) {}
};

```

[C++](#code:28)

`Sink<n>` is a type that is implicitly convertible from anything, and has a dummy parameter we can key an expansion off-of. Hence `GetImpl()` looks like this:

```cpp highlight
template <typename... Types>
class Tuple {
 private:
  template <size_t... less_than_i>
  auto& GetImpl(std::index_sequence<less_than_i...>) {
    return lambda_(
      [] (Sink<less_than_i>..., auto& the_one, auto...) -> auto& {
        return the_one;
      });
  }
  // ...
};

```

[C++](#code:29)

We can then provide the type of the `i` th element as a member type alias, using `decltype`:

```cpp highlight
template <typename Tuple, size_t i>
using TupleType = std::remove_reference_t<
  decltype(std::declval<Tuple>().template get<i>())>;

```

[C++](#code:30)

(The `template` keyword isn’t doing anything interesting; it’s just for syntactic disambiguation.)

We can, as usual, repeat implementations for `const`/ `&&` qualifiers.

### [Cheating Harder with `__type_pack_element`](\#cheating-harder-with---type-pack-element)

If we’re ok being Clang-specific, Clang just gives us a magic type function that selects out of a pack. This means we can implement `TupleType` in terms of it:

```cpp highlight
template <typename... Types>
class Tuple {
 private:
  template <size_t i>
  using type = __type_pack_element<i, Types...>;
  // ...
};

template <typename Tuple, size_t i>
using TupleType = Tuple::template Type<i>;

```

[C++](#code:31)

Then, we can use `void*` to swindle the type system, since we don’t need to go to any effort to learn the `i` th type now:

```cpp highlight
template <typename... Types>
class Tuple {
 public:
  template <size_t i>
  type<i>& get() {
    return lambda_([] (StorageFor<Types>&... places) -> decltype(auto) {
      void* erased[] = {places.get()...};
      return *reinterpret_cast<type<i>*>(erased[i]);
    });
  }
  // ...
};

```

[C++](#code:32)

(We’re returning a reference, so again note the `decltype(auto)`.)

With that we have all of the functions we set out to implement. For kicks, we can add the relevant `std` specializations to enable structured bindings on our type (along with our `get` member function):

```cpp highlight
namespace std {
template <typename... Types>
struct tuple_size<Tuple<Types...>>
    : std::integral_constant<size_t, sizeof...(Types)> {};
template <size_t i, typename... Types>
struct tuple_element<i, Tuple<Types...>> {
  using type = typename Tuple<Types...>::template type<i>;
};
}  // namespace std

```

[C++](#code:33)

Now we can see everything in action:

```cpp highlight
int main() {
  Tuple tup{1, 2, "foo", "bar", nullptr};
  tup.apply([](auto, auto, auto a, auto b, auto) {
    std::printf("%s %s\n", a, b);
  });

  auto [a, b, c, d, e] = tup;
  std::printf("%d %d\n", a, b);
}

```

[C++](#code:34)

The full code can be found at this footnote: [7](#fn:7).

## [The Damage](\#the-damage)

So, the end result is most of an implementation of `std::tuple<>`. Let’s see how well it builds. We’re going to compile the following code for `n` from 0 to 150 and measure how long it takes.

```cpp highlight
tuple t{/* 0 repeated n times */};
t.get<0>();
// ...
t.get<n>();

```

[C++](#code:35)

And here’s the results on Clang 11 (what I had on-hand) on my Zen 2 machine:

!\[\](/assets/images/tuple-benchmark.png)

We seem to beat libstdc++ by a factor of around 2, but libc++ appears to have us beat. This is because libc++ makes even more aggressive use of Clang’s intrinsics than we did, allowing them to do significantly better. Interestingly, using the builtin makes us perform *worse*. I’m actually not sure why this is.

But ultimately, this wasn’t really about beating libc++: it’s about having fun with C++ templates.

---

1. Arguably, because WG21, the body that standardizes C++, is bad at language evolution, but that’s not why we’re here. [↩︎](#fnref:1)

2. The Circle compiler totally laughs in our faces, though, because it *has* *this exact syntax*. [https://github.com/seanbaxter/circle/tree/master/tuple#circle-tuple](https://github.com/seanbaxter/circle/tree/master/tuple#circle-tuple) [↩︎](#fnref:2)

3. Basically every in-place constructor in C++ looks like this. It takes a variadic pack as a template parameter, and then takes `&&` if that as its arguments. `Args&&` here is a *forwarding reference*, which means it is `T&` or `T&&` depending on the callsite. This overrides the usual template deduction rules, and is important for making sure that e.g. `std::move` propagates correctly.

We cannot write `Types&&` instead, because that would not be a forwarding reference. `T&&` refers to a forwarding reference argument only on a function template where `T` is a parameter of *that function* and not an enclosing entity. [↩︎](#fnref:3)

4. If C++17 is too much to ask, polyfilling isn’t too hard. Instead of `(<expr>, ...);`, we can write `(void)(int[]){(<expr>, 0)...};`, even if `<expr>` is a void expression. `(<expr>, 0)` is still a comma operator call, which discards the result of `<expr>` as before. The pack expands into an array of integers (a `int[]`), which we then discard with `(void)`. This still has the behavior of evaluating `<expr>` once for each element of the pack. [↩︎](#fnref:4)

5. A *deduction guide* is a special piece of syntax introduced in C++17 intended to aid deducing the types of constructor calls. When we write `std::tuple(a, b, c)`, the template arguments of `std::tuple` are deduced. However, the constructor call may not give sufficient information to properly deduce them, because we may be calling a constructor template.

The syntax looks like this:

```cpp highlight
template <args>
MyType(args) -> MyType<types>;

```

[C++](#code:36)

This tells the compiler that when it encounters a call to a constructor of `MyTypes` that deduces the given types as its arguments, it should deduce the type after the `->` for the template arguments of `MyType`, which can be arbitrary template argument expressions. [↩︎](#fnref:5)

6. ```cpp highlight
   ```

#include <new> #include <utility>

template <typename T> class alignas(T) StorageFor { public: StorageFor() = default; template <typename... Args> void Init(Args&&... args) { new (reinterpret_cast<T*>(&data_)) T( std::forward<Args>(args)...); }

```
 const T* get() const { return reinterpret_cast<const T*>(&data_); }
 T* get() { return reinterpret_cast<T*>(&data_); }
 const T& operator*() const { return *get(); }
 T& operator*() { return *get(); }
 const T* operator->() const { return get(); }
 T* operator->() { return get(); }
private:
 char data_[sizeof(T)];
```

};

template <typename... Types> class Tuple { public: Tuple() { lambda_([] (StorageFor<Types>&... places) { (places.Init(), ...); }); }

```
 template <typename... Args,
           decltype((new (nullptr) Types(std::declval<Args>()), ...))
             = nullptr>
 Tuple(Args&&... args) {
   lambda_([&] (StorageFor<Types>&... places) {
     (places.Init(std::forward<decltype(args)>(args)), ...);
   });
 }

 Tuple(const Tuple& that) {
   lambda_([&] (StorageFor<Types>&... these) {
     that.lambda_([&] (const StorageFor<Types>&... those) {
       (new (these.get()) Types(*those), ...);
     });
   });
 }

 Tuple(Tuple&& that) {
   lambda_([&] (StorageFor<Types>&... these) {
     that.lambda_([&] (StorageFor<Types>&... those) {
       (new (these) Types(static_cast<Types&&>(*those)), ...);
     });
   });
 }

 ~Tuple() {
   lambda_([] (StorageFor<Types>&... places) {
     (places->~Types(), ...);
   });
 }

private:
 template <typename... Args>
 static auto TupleLambda(Args... args) {
   return [=] (auto callback) mutable -> decltype(auto) {
     return callback(args...);
   };
 }

 mutable decltype(TupleLambda(StorageFor<Types>{}...)) lambda_ =
   TupleLambda(StorageFor<Types>{}...);
```

};

template <typename... Types> Tuple(Types...) -> Tuple<Types...>; template <typename... Types> Tuple(const Tuple<Types...>&) -> Tuple<Types...>;

int main() { Tuple tup{1, 2, "foo", "bar"}; Tuple tup2 = tup; }

```



[godbolt](https://godbolt.org/clientstate/eyJzZXNzaW9ucyI6W3siY29tcGlsZXJzIjpbeyJpZCI6ImNsYW5nX3RydW5rIiwib3B0aW9ucyI6Ii0tc3RkPWMrKzE3In1dLCJpZCI6MSwibGFuZ3VhZ2UiOiJjKysiLCJzb3VyY2UiOiIjaW5jbHVkZSBcdTAwM2NuZXdcdTAwM2VcbiNpbmNsdWRlIFx1MDAzY3V0aWxpdHlcdTAwM2VcblxudGVtcGxhdGUgXHUwMDNjdHlwZW5hbWUgVFx1MDAzZVxuY2xhc3MgYWxpZ25hcyhUKSBTdG9yYWdlRm9yIHtcbiBwdWJsaWM6XG4gIFN0b3JhZ2VGb3IoKSA9IGRlZmF1bHQ7XG4gIHRlbXBsYXRlIFx1MDAzY3R5cGVuYW1lLi4uIEFyZ3NcdTAwM2VcbiAgdm9pZCBJbml0KEFyZ3NcdTAwMjZcdTAwMjYuLi4gYXJncykge1xuICAgIG5ldyAocmVpbnRlcnByZXRfY2FzdFx1MDAzY1QqXHUwMDNlKFx1MDAyNmRhdGFfKSkgVChcbiAgICAgIHN0ZDo6Zm9yd2FyZFx1MDAzY0FyZ3NcdTAwM2UoYXJncykuLi4pO1xuICB9XG5cbiAgY29uc3QgVCogZ2V0KCkgY29uc3QgeyByZXR1cm4gcmVpbnRlcnByZXRfY2FzdFx1MDAzY2NvbnN0IFQqXHUwMDNlKFx1MDAyNmRhdGFfKTsgfVxuICBUKiBnZXQoKSB7IHJldHVybiByZWludGVycHJldF9jYXN0XHUwMDNjVCpcdTAwM2UoXHUwMDI2ZGF0YV8pOyB9XG4gIGNvbnN0IFRcdTAwMjYgb3BlcmF0b3IqKCkgY29uc3QgeyByZXR1cm4gKmdldCgpOyB9XG4gIFRcdTAwMjYgb3BlcmF0b3IqKCkgeyByZXR1cm4gKmdldCgpOyB9XG4gIGNvbnN0IFQqIG9wZXJhdG9yLVx1MDAzZSgpIGNvbnN0IHsgcmV0dXJuIGdldCgpOyB9XG4gIFQqIG9wZXJhdG9yLVx1MDAzZSgpIHsgcmV0dXJuIGdldCgpOyB9XG4gcHJpdmF0ZTpcbiAgY2hhciBkYXRhX1tzaXplb2YoVCldO1xufTtcblxudGVtcGxhdGUgXHUwMDNjdHlwZW5hbWUuLi4gVHlwZXNcdTAwM2VcbmNsYXNzIFR1cGxlIHtcbiBwdWJsaWM6XG4gIFR1cGxlKCkge1xuICAgIGxhbWJkYV8oW10gKFN0b3JhZ2VGb3JcdTAwM2NUeXBlc1x1MDAzZVx1MDAyNi4uLiBwbGFjZXMpIHtcbiAgICAgIChwbGFjZXMuSW5pdCgpLCAuLi4pO1xuICAgIH0pO1xuICB9XG5cbiAgdGVtcGxhdGUgXHUwMDNjdHlwZW5hbWUuLi4gQXJncyxcbiAgICAgICAgICAgIGRlY2x0eXBlKChuZXcgKG51bGxwdHIpIFR5cGVzKHN0ZDo6ZGVjbHZhbFx1MDAzY0FyZ3NcdTAwM2UoKSksIC4uLikpXG4gICAgICAgICAgICAgID0gbnVsbHB0clx1MDAzZVxuICBUdXBsZShBcmdzXHUwMDI2XHUwMDI2Li4uIGFyZ3MpIHtcbiAgICBsYW1iZGFfKFtcdTAwMjZdIChTdG9yYWdlRm9yXHUwMDNjVHlwZXNcdTAwM2VcdTAwMjYuLi4gcGxhY2VzKSB7XG4gICAgICAocGxhY2VzLkluaXQoc3RkOjpmb3J3YXJkXHUwMDNjZGVjbHR5cGUoYXJncylcdTAwM2UoYXJncykpLCAuLi4pO1xuICAgIH0pO1xuICB9XG5cbiAgVHVwbGUoY29uc3QgVHVwbGVcdTAwMjYgdGhhdCkge1xuICAgIGxhbWJkYV8oW1x1MDAyNl0gKFN0b3JhZ2VGb3JcdTAwM2NUeXBlc1x1MDAzZVx1MDAyNi4uLiB0aGVzZSkge1xuICAgICAgdGhhdC5sYW1iZGFfKFtcdTAwMjZdIChjb25zdCBTdG9yYWdlRm9yXHUwMDNjVHlwZXNcdTAwM2VcdTAwMjYuLi4gdGhvc2UpIHtcbiAgICAgICAgKG5ldyAodGhlc2UuZ2V0KCkpIFR5cGVzKCp0aG9zZSksIC4uLik7XG4gICAgICB9KTtcbiAgICB9KTtcbiAgfVxuXG4gIFR1cGxlKFR1cGxlXHUwMDI2XHUwMDI2IHRoYXQpIHtcbiAgICBsYW1iZGFfKFtcdTAwMjZdIChTdG9yYWdlRm9yXHUwMDNjVHlwZXNcdTAwM2VcdTAwMjYuLi4gdGhlc2UpIHtcbiAgICAgIHRoYXQubGFtYmRhXyhbXHUwMDI2XSAoU3RvcmFnZUZvclx1MDAzY1R5cGVzXHUwMDNlXHUwMDI2Li4uIHRob3NlKSB7XG4gICAgICAgIChuZXcgKHRoZXNlKSBUeXBlcyhzdGF0aWNfY2FzdFx1MDAzY1R5cGVzXHUwMDI2XHUwMDI2XHUwMDNlKCp0aG9zZSkpLCAuLi4pO1xuICAgICAgfSk7XG4gICAgfSk7XG4gIH1cblxuICB-VHVwbGUoKSB7XG4gICAgbGFtYmRhXyhbXSAoU3RvcmFnZUZvclx1MDAzY1R5cGVzXHUwMDNlXHUwMDI2Li4uIHBsYWNlcykge1xuICAgICAgKHBsYWNlcy1cdTAwM2V-VHlwZXMoKSwgLi4uKTtcbiAgICB9KTtcbiAgfVxuXG4gcHJpdmF0ZTpcbiAgdGVtcGxhdGUgXHUwMDNjdHlwZW5hbWUuLi4gQXJnc1x1MDAzZVxuICBzdGF0aWMgYXV0byBUdXBsZUxhbWJkYShBcmdzLi4uIGFyZ3MpIHtcbiAgICByZXR1cm4gWz1dIChhdXRvIGNhbGxiYWNrKSBtdXRhYmxlIC1cdTAwM2UgZGVjbHR5cGUoYXV0bykge1xuICAgICAgcmV0dXJuIGNhbGxiYWNrKGFyZ3MuLi4pO1xuICAgIH07XG4gIH1cblxuICBtdXRhYmxlIGRlY2x0eXBlKFR1cGxlTGFtYmRhKFN0b3JhZ2VGb3JcdTAwM2NUeXBlc1x1MDAzZXt9Li4uKSkgbGFtYmRhXyA9XG4gICAgVHVwbGVMYW1iZGEoU3RvcmFnZUZvclx1MDAzY1R5cGVzXHUwMDNle30uLi4pO1xufTtcblxudGVtcGxhdGUgXHUwMDNjdHlwZW5hbWUuLi4gVHlwZXNcdTAwM2VcblR1cGxlKFR5cGVzLi4uKSAtXHUwMDNlIFR1cGxlXHUwMDNjVHlwZXMuLi5cdTAwM2U7XG50ZW1wbGF0ZSBcdTAwM2N0eXBlbmFtZS4uLiBUeXBlc1x1MDAzZVxuVHVwbGUoY29uc3QgVHVwbGVcdTAwM2NUeXBlcy4uLlx1MDAzZVx1MDAyNikgLVx1MDAzZSBUdXBsZVx1MDAzY1R5cGVzLi4uXHUwMDNlO1xuXG5pbnQgbWFpbigpIHtcbiAgVHVwbGUgdHVwezEsIDIsIFwiZm9vXCIsIFwiYmFyXCJ9O1xuICBUdXBsZSB0dXAyID0gdHVwO1xufVxuIn1dfQ==) [C++](#code:37)


    [↩︎](#fnref:6)
7. ```cpp highlight
#include <cstddef>
#include <cstdio>
#include <functional>
#include <new>
#include <type_traits>
#include <utility>

template <typename T>
class alignas(T) StorageFor {
    public:
     StorageFor() = default;
     template <typename... Args>
     void Init(Args&&... args) {
       new (reinterpret_cast<T*>(&data_)) T(
         std::forward<Args>(args)...);
     }

     const T* get() const { return reinterpret_cast<const T*>(&data_); }
     T* get() { return reinterpret_cast<T*>(&data_); }
     const T& operator*() const { return *get(); }
     T& operator*() { return *get(); }
     const T* operator->() const { return get(); }
     T* operator->() { return get(); }
    private:
     char data_[sizeof(T)];
};


template <size_t>
struct Sink {
     template <typename T>
     Sink(T&&) {}
};

template <typename... Types>
class Tuple {
     #if USE_CLANG_INTRINSIC
     template <size_t i>
     using type = __type_pack_element<i, Types...>;
     #endif

    public:
     Tuple() {
       lambda_([] (StorageFor<Types>&... places) {
         (places.Init(), ...);
       });
     }

     template <typename... Args,
               decltype((new (nullptr) Types(std::declval<Args>()), ...))
                 = nullptr>
     Tuple(Args&&... args) {
       lambda_([&] (StorageFor<Types>&... places) {
         (places.Init(std::forward<decltype(args)>(args)), ...);
       });
     }

     Tuple(const Tuple& that) {
       lambda_([&] (StorageFor<Types>&... these) {
         that.lambda_([&] (const StorageFor<Types>&... those) {
           (new (these.get()) Types(*those), ...);
         });
       });
     }

     Tuple(Tuple&& that) {
       lambda_([&] (StorageFor<Types>&... these) {
         that.lambda_([&] (StorageFor<Types>&... those) {
           (new (these) Types(static_cast<Types&&>(*those)), ...);
         });
       });
     }

     ~Tuple() {
       lambda_([] (StorageFor<Types>&... places) {
         (places->~Types(), ...);
       });
     }

     template <typename F>
     decltype(auto) apply(F&& f) const& {
       lambda_([&] (const StorageFor<Types>&... places) -> decltype(auto) {
         return std::invoke(std::forward<F>(f), *places...);
       });
     }
     template <typename F>
     decltype(auto) apply(F&& f) & {
       lambda_([&] (const StorageFor<Types>&... places) -> decltype(auto) {
         return std::invoke(std::forward<F>(f), *places...);
       });
     }
     template <typename F>
     decltype(auto) apply(F&& f) const&& {
       lambda_([&] (const StorageFor<Types>&... places) -> decltype(auto) {
         return std::invoke(std::forward<F>(f),
           static_cast<const Types&&>(*places)...);
       });
     }
     template <typename F>
     decltype(auto) apply(F&& f) && {
       lambda_([&] (const StorageFor<Types>&... places) -> decltype(auto) {
         return std::invoke(std::forward<F>(f),
           static_cast<Types&&>(*places)...);
       });
     }

     template <typename F, typename Tup>
     friend decltype(auto) apply(F&& f, Tup&& t) {
       return std::forward<Tup>(t).apply(std::forward<F>(f));
     }

     #if USE_CLANG_INTRINSIC
     template <size_t i>
     const type<i>& get() const& {
       return lambda_([] (const StorageFor<Types>&... places) -> decltype(auto) {
         const void* erased[] = {places.get()...};
         return *reinterpret_cast<const type<i>*>(erased[i]);
       });
     }

     template <size_t i>
     type<i>& get() & {
       return lambda_([] (StorageFor<Types>&... places) -> decltype(auto) {
         void* erased[] = {places.get()...};
         return *reinterpret_cast<type<i>*>(erased[i]);
       });
     }

     template <size_t i>
     const type<i>&& get() const&& {
       return lambda_([] (const StorageFor<Types>&... places) -> decltype(auto) {
         const void* erased[] = {places.get()...};
         return static_cast<const type<i>&&>(
           *reinterpret_cast<const type<i>*>(erased[i]));
       });
     }

     template <size_t i>
     type<i>&& get() && {
       return lambda_([] (StorageFor<Types>&... places) -> decltype(auto) {
         void* erased[] = {places.get()...};
         return static_cast<type<i>&&>(
           *reinterpret_cast<type<i>*>(erased[i]));
       });
     }

     #else // USE_CLANG_INTRINSIC

     template <size_t i>
     const auto& get() const& {
       return GetImpl(*this, std::make_index_sequence<i>{});
     }

     template <size_t i>
     auto& get() & {
       return GetImpl(*this, std::make_index_sequence<i>{});
     }

     template <size_t i>
     const auto&& get() const&& {
       auto& val = GetImpl(*this, std::make_index_sequence<i>{});
       return static_cast<decltype(val)&&>(val);
     }

     template <size_t i>
     auto&& get() && {
       auto& val =  GetImpl(*this, std::make_index_sequence<i>{});
       return static_cast<decltype(val)&&>(val);
     }

     #endif // USE_CLANG_INTRINSIC

     template <size_t i, typename Tup>
     friend decltype(auto) get(Tup&& t) {
       return std::forward<Tup>(t).template get<i>();
     }

    private:
     template <typename... Args>
     static auto TupleLambda(Args... args) {
       return [=] (auto callback) mutable -> decltype(auto) {
         return callback(args...);
       };
     }

     template <typename Tup, size_t... less_than_i>
     friend decltype(auto) GetImpl(Tup&& t, std::index_sequence<less_than_i...>) {
       return std::forward<Tup>(t).lambda_(
         [] (Sink<less_than_i>..., auto& the_one, auto...) -> auto& {
           return the_one;
         });
     }

     mutable decltype(TupleLambda(StorageFor<Types>{}...)) lambda_ =
       TupleLambda(StorageFor<Types>{}...);
};

#if USE_CLANG_INTRINSIC
template <typename Tuple, size_t i>
using TupleType = typename Tuple::template Type<i>;
#else
template <typename Tuple, size_t i>
using TupleType = std::remove_reference_t<
     decltype(std::declval<Tuple>().template get<i>())>;
#endif

template <typename... Types>
Tuple(Types...) -> Tuple<Types...>;
template <typename... Types>
Tuple(const Tuple<Types...>&) -> Tuple<Types...>;

namespace std {
template <typename... Types>
struct tuple_size<Tuple<Types...>>
       : std::integral_constant<size_t, sizeof...(Types)> {};
template <size_t i, typename... Types>
struct tuple_element<i, Tuple<Types...>> {
     using type = TupleType<Tuple<Types...>, i>;
};
}  // namespace std

int main() {
     Tuple tup{1, 2, "foo", "bar", nullptr};
     tup.apply([](auto, auto, auto a, auto b, auto) {
       std::printf("%s %s\n", a, b);
     });

     auto [a, b, c, d, e] = tup;
     std::printf("%d %d\n", a, b);
}

```

[godbolt](https://godbolt.org/z/eYeb9Y6cn) [C++](#code:38)

```
[↩︎](#fnref:7)```
