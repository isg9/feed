---
title: cross-language interfaces between C and&nbsp;C++
url: https://gustedt.wordpress.com/2017/08/08/cross-language-interfaces-between-c-and-c/
published: "2017-08-08T02:16:22Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3101
---

# cross-language interfaces between C and&nbsp;C++

As you know, C and C++ are sister languages that have a lot in common, but that drifted quite apart over the years. In general, neither code of one language can be compiled as the other language, there are too many major and minor twists that make this impossible. Not only are there syntax differences between the two, some common syntax can actually have diverging semantics. So generally, it makes no sense to compile C code with a C++ compiler, and you should look with suspicion at any code or programmer that claims to do so.

Where C and C++ usually agree, though, is on the ABI, the application binary interface, so data structures and functions of one language can be used by the other to some extent. C and C++ also kept a sufficiently wide intersection in there respective specification of interfaces, such that **one header file** can be used from both.

In this post I try collect those parts that are in that intersection, and I propose some coding style that should accommodate both worlds suitably well. But as my personal history goes, this will merely be a POV of a C programmer that wants to provide interfaces for C++.

## Types

C and C++ share a set of common base types and constructs that can easily be used, and others that have subtle differences:

- Integer types such as `char`, `short`, `int`, `long`, … share representation and semantics. But beware that `bool` (or `_Bool`) and enumeration types need special care, see below.
- All floating point types have the same representation and the syntax for `float`, `double` and perhaps `long double` are the same. But complex types have different syntax. C adds a some sort of specifier the real base type, C++ has them as `template` types.
- Array types have the same syntax and representation, but C allows the array bounds to be dynamic, so-called *variable length arrays*, VLA.
- Structure and union types, have the same representation, as long as they don’t declare function members. (C++ calls these POD, plain old data structures.)
- Atomic types have the same representation and semantics, but different syntax. C++ has them as templates. C has two alternative forms, as a *type qualifier* or as a *type specifier*.

### Booleans

The Boolean type in C is “officially” called `_Bool` but a convenience macro exists in that defines `bool` to point to this same type. In fact, this construct only has been invented to ensure *backwards compatibility* for code that had been written before the introduction of the Boolean type. It might eventually be remove from a future version of the C standard. For C++, using `_Bool` makes not much sense, it is ugly and introducing a C feature to C++ that is just meant as temporary (though for a looooong time) will not happen.

As a consequence the easiest way to use Booleans is to use `bool` throughout, and to ensure that C sees the corresponding header include. This can easily be achieved:

```
#ifndef __cplusplus
# include
#endif

extern bool weAreHappy;

```

As said, you should not try to include this header to C++, it just makes no sense.

### Enumeration types

Plain enumeration types themselves should be compatible between C and C++. Enumeration constants have the same value, but have different types. In C they are of type `int`, C++ they are of the enumeration type itself.

As long as you use the constants for their values, all should work out perfectly. But if you try to use variables or function parameters of enumeration types, the difficulties start. C and C++ have different rules for implicit conversion from an to these types, so you better avoid using them, here.

### Atomics

In C++ an atomic type of some base type is specified as a `template`:

```
extern std::atomic flags;

```

In C there are two equivalent writings for this:

```
extern unsigned _Atomic flags;  // an atomic qualifier
extern _Atomic(unsigned) flags; // an atomic specifier
...

```

In common code between C and C++ the latter can be used to accommodate both languages:

```
#ifdef __cplusplus
# include
# define _Atomic(T) std::atomic
#else
# include
#endif

extern _Atomic(unsigned) flags;
...

```

### Complex types

Again, different complex types are specified as `template` types in C++:

```
extern std::complex angle;

```

In C the equivalent writing for this is:

```
extern complex double angle;

```

Both languages guarantee that these types are laid out as two consecutive objects of their base, the first for the real part the second for the imaginary part.

Unfortunately there is no syntax that would be similar to the atomic specifier, above, that would allow to use a simple macro. On the other hand, there are not so many complex types and just using in-place names for the them can save us

```
#ifdef __cplusplus
# include
typedef std::complex cfloat;
typedef std::complex cdouble;
typedef std::complex cldouble;
# define I (cfloat({ 0, 1 }))
#else
# include
typedef complex float cfloat;
typedef complex double cdouble;
typedef complex long double cldouble;
#endif

extern cdouble angle;
...

cdouble angle = 4.0 + 3.0*I;

```

Also beware that common code that uses C and C++ complex types should not use the identifier `I` for other purposes than the complex root of `-1`.

## Objects

The ABI compatibility between C and C++ implies that objects that have any of the shared types have the same *object representation*, that is that their layout of their bytes in memory and the interpretation of these bytes are the same. Syntactically, *named objects*, that is variables and function parameters, are declared the same. Otherwise the whole idea of a common interface specification would be hopeless.

### Temporaries

But C and C++ differ much in their notion of unnamed, *temporary objects*. In C there are two different sorts of temporary objects:

- Return values of functions that contain array types return a temporary object such that the array elements can be addressed through pointer arithmetic. Such temporary objects are unmutable and cease to exist as soon as the expression that contains the function call is terminated.
- *Compound literals* are temporaries that are *explicitly* created. They act as if a variable had been declared in the current scope, and generally all other rules for such variables apply.

In C++, there is no equivalent for the latter. Even temporaries that are explicitly constructed by calling a constructor, will in general only be alive during the evaluation of the expression where they appear. Taking references to such objects can extend their lifetime, though, but the rules here are quite complicated and references are out of scope for common C and C++ interfaces, anyhow.

So using temporaries in interfaces is basically to be ruled out whenever this would be used in C to provide the address of an object to a function that will return that address for further use in the current scope.

See also *variable argument lists* and *macros*, below.

### `Const` qualified objects

In C++ you can place `const` qualified objects in header files such that they act as constants for the underlying type.

```
constexpr unsigned const fortytwo = 42u;

```

This construct (even wihtout `constexpr`) is not allowed in C since it will result in the definition of a `fortytwo` object in all .o files, and thus violate the *one definition* rule. You could, kind of, emulate that feature by declaring the object `static`, but

- That would result in multiple copies, that would reside ad different addresses. Programs that compare pointers to such objects could get confused.
- In C, `static` objects cannot be used from `inline` functions.

For the common interface we are bound to macros. For types that have literals this is simple

```
#define fortytwo 42u

```

For structure types, there is no common solution to this. For C you would use a compound literal inside a macro, for C++ you would use a `const` qualified global object.

## Functions

For the common types that we listed about the function call ABI on a given platform should be the same. That is regardless of the language through which we access the interface, the same rules of representing function parameters and return values in hardware register or on the stack will apply. The first important difference between C and C++ that apply is that C++ has function overloading and therefore has to mangle the types of the arguments in the external name, unless you ask it not to do this. The common idiom to ensure this is

```
#ifdef __cplusplus
extern "C" {
#endif

int toto(void);

double hui(char*);

#ifdef __cplusplus
}
#endif

```

That is we have two C++ zones surrounding the common interface specification, declaring the functions to be “extern” with language interface “C”. The macro `__cplusplus` is guaranteed to be undefined by any C compiler and is guaranteed to be defined by any C++ compiler.

### Functions without parameters

In C, a function interface that is declared syntactically with a empty parameter list in fact has no prototype. Such a function could take any amount and (almost) any type of parameters. There a special rules how arguments to such functions are converted on the calling side.

An function that is known to receive no arguments must therefore be specified differently, namely with `void` as specification for the parameters, such as function `toto`, above.

### Functions that have VLA parameters

A common C idiom to specify a function that receives a 2-dimensional matrix would look like

```
void initialize(size_t n, size_t m, double A[n][m]);

```

There is no syntax in C++ to support this feature, so we cannot use it as such in any common interface specification.

Theoretically C++ could use such a function, too, because the ABI underneath is just two `size_t` and a pointer to `double`. The C type information for the matrix is only assembled at the beginning of the function execution, the caller has nothing to provide in addition to the arguments.

We could be tempted to present a “fake” interface to C++, that only requests a `double*`, but such a cheat can easily backfire, if we use it wrong, accidentally. Better create a small wrapper that receives an appropriate `vector` template.

### Multidimensional array parameters with `const` qualification

The rules for compatibility between types with different qualifications differ between the two languages. In C, a function with a `const` qualified 2D matrix can not easily be called with an argument that is not `const` qualified. Avoid that in shared interfaces. `const` qualified pointer targets are fine, though.

### `restrict` qualification and aliasing

Aliasing rules are different in C and C++, and they must be because of C++’s reference concept. Therefore you must be really careful in what properties you assume for pointers. C has the possibility to specify with `restrict` that the object behind a pointer can only be accessed through that pointer alone. This is a powerful contract, and places an important constraint on the caller of a function.

C++ has no equivalent of this. It is often advocated that for C++ one should just define `restrict` as a macro that is replaced by nothing. That misses the point to educate the caller of the function to be careful to pass distinct arguments to all the parameters.

Avoid this feature if you can for cross-language interfaces. If you can’t, document the fact that arguments must be distinct thoroughly.

### `inline` functions

`Inline` has slightly differing semantics between C and C++, but that can usually be mended with some care. The difference lies in the instantiation of such a function, in case the linker needs a copy of it. C++ takes magically care of that, but for C, it is up to the programmer to provide exactly one such an instantiation in some of the object files that are linked together. So we have to ensue that the library that implements our interface provides such an instantiation.

**But**, *inline* functions are first of all functions, so all that is said above about the slippery slope of language differences applies. If you use this feature, keep the functions to a minimum and delegate most of the implementation of the underlying feature to one of the languages.

### Variadic functions

Variadic functions are complicated beasts, they have complicated rules for argument conversion (promotion) and they have no inherent mechanism that would help to know the number of arguments that a call has received. Creating new interfaces with that functionality should be avoided. That is, their use should be restricted to the few well established functions of the C library such as `printf` or `scanf`.

Both languages have features that can replace these, most of the time. Obviously, C++’s variadic templates are out of the scope for a common interface with C. For the common use case of a list of arguments that all have the same `const` qualified type, a temporary array parameter can be used through an intermediate macro call, see below.

### Type generic interfaces

C and C++ have diametrically opposed strategies to implement type generic function interfaces. C++ has function overloading and default arguments, C has `_Generic` primary expressions and uses macros to implement the latter. C’s mechanism is not easily extendable, that is you can only glue together functions for a known list of types. If you want to support a new type, you’d have to change your `_Generic` expression or macro.

There is not much common ground, here, so if you want to provide such features for both languages, you would have to implement them differently for both languages.

A simple mechanism would be to create the different functions for the type list in C and use a suitable naming convention, say something like `hu_flt` and `hu_dbl`. Where the C code would use a macro

```
#define hu(X)            \
_Generic((X),            \
         float: hu_flt,  \
         double: hu_dbl) \
(X)

```

C++ would just interface this with some `template` specializations:

```
template inline auto hu(T x);

template inline auto hu(float x) { return hu_flt(x); }
template inline auto hu(double x) { return hu_dbl(x); }

```

Both should result in equally efficient executable code, namely in the compiled program calls to `hu` should be directly replaced by the call to the corresponding C function.

If your type generic interface provides compile time constants such as for C with

```
#define needed(X)        \
_Generic((X),            \
         float: 37,      \
         double: 51)

```

you can use a similar mechanism for C++ that uses `constexpr` instead of `inline`:

```
template constexpr auto needed(T x);

template constexpr auto needed(float x) { return 37; }
template constexpr auto needed(double x) { return 51; }

```

## Macros

The preprocessor for C and C++ should nowadays produce equivalent text replacements. The intent of both standard committees, C and C++, is to keep them completely in sync. We already have seen some examples above where the preprocessor comes to our rescue when declaring common interfaces.

In particular, relatively recently C++’s preprocessor also has been equipped with `variadic` macros. These are macros that can receive a varying number of arguments. To conclude, let’s look into a more sophisticated example that provides an variadic interface, but with just the same type for all parameters. Suppose we have a simple function that takes an array of `double`:

```
size_t median_vec(size_t len, double arr[]);

```

The idea is that we want to provide a similar interface for both languages that just takes a fixed list of values, constructs an array and passes it to the function.

```
# define ASIZE(...)  /* some macro magic that determines the length of the argument list */

#ifndef __cplusplus
# define ARRAY(T, ...) ((T const[]){ __VA_ARGS__ })                  // compound literal
#else
# define ARRAY(T, ...) (std::initializer_list({ __VA_ARGS__ }).begin()) // standard initializer array
#endif

#define median(...) median_vec(ASIZE(__VA_ARGS__), ARRAY(double, __VA_ARGS__))
...
size_t med = median(0, 7, a, 33, b, c);

```
