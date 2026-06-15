---
title: gcc doesn&#8217;t inline position independent code&nbsp;properly
url: https://gustedt.wordpress.com/2017/05/09/gcc-doesnt-inline-position-independent-code-properly/
published: "2017-05-09T12:27:39Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=2950
---

# gcc doesn&#8217;t inline position independent code&nbsp;properly

When compiling *position independent code*, PIC, the gcc compiler fails to inline many functions that have an external name, that is that are not declared `static`. While this is conforming to the C standard, this is a complete blow to a lot of optimization efforts that I have put into my code in many places.

PIC code is needed if we want to create a shared library, because such a library must be visible for all processes that uses it in their respective address space.

“Usual” compilation may (and does) inline many functions as soon as their definition is visible, and this can lead to a lot of optimization opportunities, e.g constant propagation can be done much more aggressively and whole code paths can be eliminated because they are unreachable.

> PIC code misses a lot of optimization potential

Because PIC code is more complicated that “normal” code, gcc is much less successful with inlining. Indirections make code larger and hide optimization opportunities.

Unfortunately the picture does not even improve for functions that are declared `inline`. Just as a reminder, the C `inline` model is to have an `inline` *definition* of a function in a header file, and to have one extra declaration with `extern` in one translation unit where the external symbol should be generated. Even the gcc specific trick of marking such a function with an attribute `always_inline` doesn’t improve the situation and inlining with gcc for such functions is not satisfactory.

In contrast to that, functions that use gcc’s ancient inlining model `gnu_inline` *are* inlined much more aggressively. In this model, implementors provide two potenially different definitions of a function. One, for the header, is seen by all compilation units, and the other, in just one translation unit, is just wherever a linker symbol is needed.

For general C code, this model is much more difficult to maintain. If we really provide two different functions, we have to be sure that the two implementations of the function have the same semantics. If on the other hand we only want to provide one function we have to use some weird compiler tricks to convince the compiler to implement the linker symbol somewhere.

Luckily, for [Modular C](https://gustedt.wordpress.com/2017/01/26/update-on-modular-c/) we are able to circumvent these problems. Here, an `inline` function is always instantiated in the module that defines it and all importers of that module see the `inline` definition. Internally for the implementation of *Modular C* we can map this simplified model to gcc’s inline model more easily and more efficiently, because we know exactly where a function should be instantiated.

We do that by some weird compilation trick as was already mentioned above. In the `C` module, we provide two different macros.

```
#define C¯own_inline        inline __attribute__((__gnu_inline__, __visibility__("default")))
#define C¯imp_inline extern inline __attribute__((__gnu_inline__, __visibility__("default")))

```

Then, we overload the `inline` keyword with a macro pointing to the first, and switch that macro definition in the middle of the road to the second.

By that, the first version, `C¯own_inline`, is used for the importers such that they see the function favorably for inlining. The second, `C¯imp_inline`, is only used for the compilation of the module itself and forces the generation of the linker symbol. Finding the point in the code where to switch from one definition to the other is relatively easy for *Modular C*: all imports are done in dependency order (and not in declaration order) so the header information for imported modules always precedes code that is proper to a module.

I imagine that for “classical” C with arbitrary include orderings, automating such a strategy would be quite challenging.
