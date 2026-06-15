---
title: Rust release lightning talk
url: https://graydon2.dreamwidth.org/214016.html
published: "2015-05-27T16:10:29Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:214016
---

# Rust release lightning talk

*I don't often give talks about Rust, but I was in Tokyo recently during the Rust 1.0 release and figured I'd drop in on the Mozilla Tokyo office release party. I figured they'd try to make me talk about something, so I prepared a lightning talk. As is typical of my Rust-project communication style, I presented in point form. This is the contents of the talk:*

## Five lists of six things about Rust:

2. Six ways Rust is fundamentally different from how it started

3. Borrow checker subsumed most other safety mechanisms, lots discarded

4. Much more static: monomorphization of sizes, glue code, and static trait dispatch

5. LLVM: strengths (amazing optimization) and weaknesses (narrow semantics)

6. Between these two points, maybe 100x faster at runtime, 100x slower to compile

7. Grammar, resolution, dispatch, type system much more complex, expressive

8. Adopted standard C platform stack, threads, ABI, linkage, mangling, unwinding

9. Six ways Rust is fundamentally the same as how it started

10. Safety through memory ownership and isolation, no global GC, no aliased mutable state

11. Default immutable, algebraic data types ("ML in C++ clothing")

12. Focus on dense memory layout, interior allocation, minimizing pointer chasing, vectors over lists

13. No global namespace, everything module scoped and crate relative

14. Support for standard tools: gdb, perf, dtrace, objdump

15. RAII, dtors, idempotent uncatchable unwinding ("crash only")

16. Six things we lost along the way

17. Typestate system

18. Effect system

19. Function complexities: parameter modes, argument binding, stack iterators, tail calls

20. Language-integrated runtime for tasks, channels, logging

21. GC pointers, task local GC (yes, rustboot had a real mark/sweep)

22. Dynamic, structural object types

23. Six things we gained along the way

24. Lambdas, with environment capture

25. First class borrowed pointers -- not just parameter modes -- with explicit lifetimes

26. Traits, with associated items

27. Hygienic macros beyond just raw syntax extensions

28. Multiple parallelism modes beyond just actors (ARC, fork/join, SIMD)

29. Cargo, crates.io, a huge vibrant community!

30. Six things I'm irrationally, disproportionately pleased by

31. Rich patterns: slice patterns, range patterns, or patterns

32. Novel consequences of move semantics: static freeze/thaw, iterators that consume their containers

33. Rich syntax extensions: compile time regular expressions, SQL statements, docopt

34. Novel embeddings: postgres and python extensions, crypto libraries, kernels in rust

35. C++ ballpark on benchmarks

36. A 1.0 stable release after nearly a decade of work

*(Ok, maybe my pleasure at some of these is rational and proportionate..)*

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=214016) comments
