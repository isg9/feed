---
title: Abstract machines and the compilers that love/hate them
url: https://graydon2.dreamwidth.org/264181.html
published: "2019-01-05T21:37:07Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:264181
---

# Abstract machines and the compilers that love/hate them

Observations noted in [From Krivine’s machine to the Caml implementations (Leroy, 2005)](https://xavierleroy.org/talks/zam-kazam05.pdf):

> In every area where abstract machines help, there are arguably better alternatives:
>
> • Exposing reduction order: CPS transformation, structured operational semantics, reduction contexts.
>
> • Closures and environments: explicit substitutions.
>
> • Efficient execution: optimizing compilation to real machine code.
>
> • Intermediate languages: RTL, SSA, CPS, A-normal forms, etc.
>
> • Code distribution: ANDF (?).
>
> **Abstract machines do many things well but none very well.**
>
> This we will illustrate in the case of efficient execution of Caml.
>
> \[...\]
>
> In retrospect, the journey could have been shorter by not going through abstract machines at all.
>
> But maybe it would never have taken place without the "detour" through abstract machines.

Observations noted in [The Verified CakeML Compiler Backend (extended version, Tan et al., 2018)](https://www.cs.cmu.edu/~yongkiat/files/cakeml-jfp.pdf):

> The previous compiler compiled from source to a single IL, then to stack-machine-based bytecode and finally to x86-64. The bytecode was designed so that each operation mapped to a fixed sequence of x86 instructions, and it was also designed to make verification of the GC as easy as possible. Unfortunately, the ease of verification also meant that the compiler had poor performance – **we found the bytecode IL too low level for functional programming optimisations (multi-argument functions, lambda lifting, etc.) and too high level for backend optimisations**. For example, it naively followed the semantics and allocated a closure on each additional argument to a function, pattern matches were not compiled efficiently (even for exhaustive, non-nested patterns), and the bytecode compiler only used registers as temporary storage within single bytecode instructions. The new version addresses all of these problems and splits each improvement into its own phase and IL in order to keep the verification of different parts as separate and as understandable as possible.

(emphases mine)

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=264181) comments
