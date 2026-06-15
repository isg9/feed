---
title: pre-initialization of garbage-collected webassembly heaps — wingolog
url: https://wingolog.org/archives/2023/03/10/pre-initialization-of-garbage-collected-webassembly-heaps
published: "2023-03-10T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/03/10/pre-initialization-of-garbage-collected-webassembly-heaps
---

# pre-initialization of garbage-collected webassembly heaps — wingolog

## [pre-initialization of garbage-collected webassembly heaps](/archives/2023/03/10/pre-initialization-of-garbage-collected-webassembly-heaps)

10 March 2023 9:20 AM

- [wasm](/tags/wasm)
- [webassembly](/tags/webassembly)
- [wizer](/tags/wizer)
- [gc](/tags/gc)
- [pre-initialization](/tags/pre-initialization)
- [types](/tags/types)
- [igalia](/tags/igalia)

Hey comrades, I just had an idea that I won’t be able to work on in the next couple months and wanted to release it into the wild. They say if you love your ideas, you should let them go and see if they come back to you, right? In that spirit I abandon this idea to the woods.

Basically the idea is [Wizer-like pre-initialization of WebAssembly
modules](https://github.com/bytecodealliance/wizer), but for modules that store their data on the GC-managed heap instead of just in linear memory.

Say you have a WebAssembly module with [GC
types](https://github.com/WebAssembly/gc/blob/master/proposals/gc/MVP.md). It might look like this:

```
(module
  (type $t0 (struct (ref eq)))
  (type $t1 (struct (ref $t0) i32))
  (type $t2 (array (mut (ref $t1))))
  ...
  (global $g0 (ref null eq)
    (ref.null eq))
  (global $g1 (ref $t1)
    (array.new_canon $t0 (i31.new (i32.const 42))))
  ...
  (function $f0 ...)
  ...)

```

You define some struct and array types, there are some global variables, and some functions to actually do the work. (There are probably also tables and other things but I am simplifying.)

If you consider the object graph of an instantiated module, you will have some set of *roots* R that point to GC-managed objects. The live objects in the heap are the roots and any object referenced by a live object.

Let us assume a standalone WebAssembly module. In that case the set of types T of all objects in the heap is closed: it can only be one of the types *`$t0`*, *`$t1`*, and so on that are defined in the module. These types have a partial order and can thus be sorted from most to least specific. Let’s assume that this sort order is just the reverse of the definition order, for now. Therefore we can write a general type introspection function for any object in the graph:

```
(func $introspect (param $obj anyref)
  (block $L2 (ref $t2)
    (block $L1 (ref $t1)
      (block $L0 (ref $t0)
        (br_on_cast $L2 $t2 (local.get $obj))
        (br_on_cast $L1 $t1 (local.get $obj))
        (br_on_cast $L0 $t0 (local.get $obj))
        (unreachable))
      ;; Do $t0 things...
      (return))
    ;; Do $t1 things...
    (return))
  ;; Do $t2 things...
  (return))

```

In particular, given a WebAssembly module, we can generate a function to trace edges in an object graph of its types. Using this, we can identify all live objects, and what’s more, we can take a snapshot of those objects:

```
(func $snapshot (result (ref (array (mut anyref))))
  ;; Start from roots, use introspect to find concrete types
  ;; and trace edges, use a worklist, return an array of
  ;; all live objects in topological sort order
  )

```

Having a heap snapshot is interesting for introspection purposes, but my interest is in having fast start-up. Many programs have a kind of “initialization” phase where they get the system up and running, and only then proceed to actually work on the problem at hand. For example, when you run `python3 foo.py`, Python will first spend some time parsing and byte-compiling `foo.py`, importing the modules it uses and so on, and then will actually run `foo.py`‘s code. Wizer lets you snapshot the state of a module after initialization but before the real work begins, which can save on startup time.

For a GC heap, we actually have similar possibilities, but the mechanism is different. Instead of generating an array of all live objects, we could generate a serialized state of the heap as bytecode, and another function to read the bytecode and reload the heap:

```
(func $pickle (result (ref (array (mut i8))))
  ;; Return an array of bytecode which, when interpreted,
  ;; can reconstruct the object graph and set the roots
  )
(func $unpickle (param (ref (array (mut i8))))
  ;; Interpret the bytecode, building object graph in
  ;; topological order
  )

```

The unpickler is module-dependent: it will need one case to construct each concrete type *`$tN`* in the module. Therefore the bytecode grammar would be module-dependent too.

What you would get with a bytecode-based `$pickle`/ `$unpickle` pair would be the ability to serialize and reload heap state many times. But for the pre-initialization case, probably that’s not precisely what you want: you want to residualize a new WebAssembly module that, when loaded, will rehydrate the heap. In that case you want a function like:

```
(func $make-init (result (ref (array (mut i8))))
  ;; Return an array of WebAssembly code which, when
  ;; added to the module as a function and invoked,
  ;; can reconstruct the object graph and set the roots.
  )

```

Then you would use binary tools to add that newly generated function to the module.

In short, there is a space open for a tool which takes a WebAssembly+GC module M and produces M’, a module which contains a `$make-init` function. Then you use a WebAssembly+GC host to load the module and call the `$make-init` function, resulting in a WebAssembly function `$init` which you then patch in to the original M to make M’’, which is M pre-initialized for a given task.

### Optimizations

Some of the object graph is constant; for example, an instance of a `struct` type that has no mutable fields. These objects don’t have to be created in the init function; they can be declared as new constant global variables, which an engine may be able to initialize more efficiently.

The pre-initialized module will still have an initialization phase in which it builds the heap. This is a constant function and it would be nice to avoid it. Some WebAssembly hosts will be able to run pre-initialization and then snapshot the GC heap using lower-level facilities (copy-on-write mappings, pointer compression and relocatable cages, pre-initialization on an internal level...). This would potentially decrease latency and may allow for cross-instance memory sharing.

### Limitations

There are five preconditions to be able to pickle and unpickle the GC heap:

1. The set of concrete types in a module must be closed.

2. The roots of the GC graph must be enumerable.

3. The object-graph edges from each live object must be enumerable.

4. To prevent cycles, we have to know when an object has been visited: objects must have identity.

5. We must be able to create each type in a module.

I think there are three limitations to this pre-initialization idea in practice.

One is `externref`; these values come from the host and are by definition not introspectable by WebAssembly. Let’s keep the closed-world assumption and consider the case where the set of external reference types is closed also. In that case if a module allows for external references, we can perhaps make its pickling routines call out to the host to (2) provide any external roots (3) identify edges on `externref` values (4) compare `externref` values for identity and (5) indicate some imported functions which can be called to re-create exernal objects.

Another limitation is `funcref`. In practice in the current state of WebAssembly and GC, you will only have a `funcref` which is created by `ref.func`, and which (3) therefore has no edges and (5) can be re-created by `ref.func`. However neither WebAssembly nor the JS API has no way of knowing which function index corresponds to a given funcref. Including function references in the graph would therefore require some sort of host-specific API. Relatedly, function references are not comparable for equality ( `func` is not a subtype of `eq`), which is a little annoying but not so bad considering that function references can’t participate in a cycle. Perhaps a solution though would be to assume (!) that the host representation of a funcref is constant: the JavaScript (e.g.) representations of `(ref.func 0)` and `(ref.func 0)` are the same value (in terms of `===`). Then you could compare a given function reference against a set of known values to determine its index. Note, when function references are expanded to include closures, we will have more problems in this area.

Finally, there is the question of roots. Given a module, we can generate a function to read the values of all reference-typed globals and of all entries in all tables. What we can’t get at are any references from the stack, so our object graph may be incomplete. Perhaps this is not a problem though, because when we unpickle the graph we won’t be able to re-create the stack anyway.

OK, that’s my idea. Have at it, hackers!

## related articles

- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [missing the point of webassembly](/archives/2024/01/08/missing-the-point-of-webassembly)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)
- [a world to win: webassembly for the rest of us](/archives/2023/03/20/a-world-to-win-webassembly-for-the-rest-of-us)
- [wastrel milestone: full hoot support, with generational gc as a treat](/archives/2026/04/09/wastrel-milestone-full-hoot-support-with-generational-gc-as-a-treat)

Comments are closed.
