---
title: 'a world to win: webassembly for the rest of us — wingolog'
url: https://wingolog.org/archives/2023/03/20/a-world-to-win-webassembly-for-the-rest-of-us
published: "2023-03-20T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/03/20/a-world-to-win-webassembly-for-the-rest-of-us
---

# a world to win: webassembly for the rest of us — wingolog

## [a world to win: webassembly for the rest of us](/archives/2023/03/20/a-world-to-win-webassembly-for-the-rest-of-us)

20 March 2023 9:06 AM

- [gc](/tags/gc)
- [webassembly](/tags/webassembly)
- [wasm](/tags/wasm)
- [scheme](/tags/scheme)
- [garbage collection](/tags/garbage%20collection)
- [browsers](/tags/browsers)
- [js](/tags/js)
- [igalia](/tags/igalia)
- [spritely](/tags/spritely)
- [bobkonf](/tags/bobkonf)
- [delimited continuations](/tags/delimited%20continuations)

Good day, comrades!

Today I’d like to share the good news that WebAssembly is finally coming for the rest of us weirdos.

# A world to win

WebAssembly for the rest of us

17 Mar 2023 – BOB 2023

Andy Wingo

Igalia, S.L.

This is a transcript-alike of a talk that I gave last week at [BOB
2023](https://bobkonf.de/2023/en/program.html), a gathering in Berlin of people that are using “technologies beyond the mainstream” to get things done: Haskell, Clojure, Elixir, and so on. PDF slides [here](https://wingolog.org/pub/2023-bobkonf-wasm-for-the-rest-of-us-slides.pdf), and I’ll link the video too when it becomes available.

## WebAssembly, the story

WebAssembly is an exciting new universal compute platform

WebAssembly: what even is it? Not a programming language that you would write software in, but rather a compilation target: a sort of assembly language, if you will.

## WebAssembly, the pitch

Predictable portable performance

- Low-level
- Within 10% of native

Reliable composition via isolation

- Modules share nothing by default
- No nasal demons
- Memory sandboxing

Compile your code to WebAssembly for easier distribution and composition

If you look at what the characteristics of WebAssembly are as an abstract machine, to me there are two main areas in which it is an advance over the alternatives.

Firstly it’s “close to the metal” – if you compile for example an image-processing library to WebAssembly and run it, you’ll get similar performance when compared to compiling it to x86-64 or ARMv8 or what have you. (For image processing in particular, native still generally wins because the SIMD primitives in WebAssembly are more narrow and because getting the image into and out of WebAssembly may imply a copy, but the general point remains.) WebAssembly’s instruction set covers a broad range of low-level operations that allows compilers to produce efficient code.

The novelty here is that WebAssembly is both portable while also being successful. We language weirdos know that it’s not enough to do something technically better: you have to also succeed in getting traction for your alternative.

The second interesting characteristic is that WebAssembly is (generally speaking) a principle-of-least-authority architecture: a WebAssembly module starts with access to nothing but itself. Any capabilities that an instance of a module has must be explicitly shared with it by the host at instantiation-time. This is unlike DLLs which have access to all of main memory, or JavaScript libraries which can mutate global objects. This characteristic allows WebAssembly modules to be reliably composed into larger systems.

## WebAssembly, the hype

It’s in all browsers! Serve your code to anyone in the world!

It’s on the edge! Run code from your web site close to your users!

Compose a library (eg: Expat) into your program (eg: Firefox), without risk!

It’s the new lightweight virtualization: Wasm is what containers were to VMs! Give me that Kubernetes cash!!!

Again, the remarkable thing about WebAssembly is that it is succeeding! It’s on all of your phones, all your desktop web browsers, all of the content distribution networks, and in some cases it seems set to replace containers in the cloud. Launch the rocket emojis!

## WebAssembly, the reality

WebAssembly is a weird backend for a C compiler

Only some source languages are having success on WebAssembly

What about Haskell, Ocaml, Scheme, F#, and so on – what about *us*?

Are we just lazy? (Well...)

So why aren’t we there? Where is Clojure-on-WebAssembly? Where are the F#, the Elixir, the Haskell compilers? Some early efforts exist, but they aren’t really succeeding. Why is that? Are we just not putting in the effort? Why is it that Rust gets to ride on the rocket ship but Scheme does not?

## WebAssembly, the reality (2)

WebAssembly (1.0, 2.0) is not well-suited to garbage-collected languages

Let’s look into why

As it turns out, there is a reason that there is no good Scheme implementation on WebAssembly: the initial version of WebAssembly is a terrible target if your language relies on the presence of a garbage collector. There have been some advances but this observation still applies to the current standardized and deployed versions of WebAssembly. To better understand this issue, let’s dig into the guts of the system to see what the limitations are.

## GC and WebAssembly 1.0

Where do garbage-collected values live?

For WebAssembly 1.0, only possible answer: linear memory

```
(module
  (global $hp (mut i32) (i32.const 0))
  (memory $mem 10)) ;; 640 kB
```

The primitive that WebAssembly 1.0 gives you to represent your data is what is called *linear memory*: just a buffer of bytes to which you can read and write. It’s pretty much like what you get when compiling natively, except that the memory layout is more simple. You can obtain this memory in units of 64-kilobyte pages. In the example above we’re going to request 10 pages, for 640 kB. Should be enough, right? We’ll just use it all for the garbage collector, with a bump-pointer allocator. The heap pointer / allocation pointer is kept in the mutable global variable `$hp`.

```
(func $alloc (param $size i32) (result i32)
  (local $ret i32)
  (loop $retry
    (local.set $ret (global.get $hp))
    (global.set $hp
      (i32.add (local.get $size) (local.get $ret)))

    (br_if 1
      (i32.lt_u (i32.shr_u (global.get $hp) 16)
                (memory.size))
      (local.get $ret))

    (call $gc)
    (br $retry)))

```

Here’s what an allocation function might look like. The allocation function `$alloc` is like malloc: it takes a number of bytes and returns a pointer. In WebAssembly, a pointer to memory is just an offset, which is a 32-bit integer ( `i32`). (Having the option of a 64-bit address space is planned but not yet standard.)

If this is your first time seeing the text representation of a WebAssembly function, you’re in for a treat, but that’s not the point of the presentation :) What I’d like to focus on is the `(call $gc)` – what happens when the allocation pointer reaches the end of the region?

## GC and WebAssembly 1.0 (2)

What hides behind `(call $gc)` ?

Ship a GC over linear memory

Stop-the-world, not parallel, not concurrent

But... **roots**.

The first thing to note is that you have to provide the `$gc` yourself. Of course, this is doable – this is what we do when compiling to a native target.

Unfortunately though the multithreading support in WebAssembly is somewhat underpowered; it lets you share memory and use atomic operations but you have to create the threads outside WebAssembly. In practice probably the GC that you ship will not take advantage of threads and so it will be rather primitive, deferring all collection work to a stop-the-world phase.

## GC and WebAssembly 1.0 (3)

Live objects are

- the roots
- any object referenced by a live object

Roots are globals and locals in active stack frames

**No way to visit active stack frames**

What’s worse though is that you have no access to roots on the stack. A GC has to keep live objects, as defined circularly as any object referenced by a root, or any object referenced by a live object. It starts with the roots: global variables and any GC-managed object referenced by an active stack frame.

But there we run into problems, because in WebAssembly (any version, not just 1.0) you can’t iterate over the stack, so you can’t find active stack frames, so you can’t find the stack roots. (Sometimes people want to support this as a [low-level
capability](https://github.com/WebAssembly/exception-handling/issues/105) but generally speaking the consensus would appear to be that overall performance will be better if the engine is the one that is responsible for implementing the GC; but that is foreshadowing!)

## GC and WebAssembly 1.0 (3)

Workarounds

- handle stack for precise roots
- spill all possibly-pointer values to linear memory and collect conservatively

Handle book-keeping a drag for compiled code

Given the noniterability of the stack, there are basically two work-arounds. One is to have the compiler and run-time maintain an explicit stack of object roots, which the garbage collector can know for sure are pointers. This is nice because it lets you move objects. But, maintaining the stack is overhead; the state of the art solution is rather to create a side table (a “stack map”) associating each potential point at which GC can be called with instructions on how to find the roots.

The other workaround is to spill the whole stack to memory. Or, possibly just pointer-like values; anyway, you conservatively scan all words for things that might be roots. But instead of having access to the memory to which the WebAssembly implementation would spill your stack, you have to do it yourself. This can be OK but it’s sub-optimal; see [my recent post on the Whippet garbage
collector](https://wingolog.org/archives/2023/02/07/whippet-towards-a-new-local-maximum) for a deeper discussion of the implications of conservative root-finding.

## GC and WebAssembly 1.0 (4)

Cycles with external objects (e.g. JavaScript) uncollectable

A pointer to a GC-managed object is an offset to linear memory, need capability over linear memory to read/write object from outside world

No way to give back memory to the OS

Gut check: gut says no

If that were all, it would already be not so great, but it gets worse! Another problem with linear-memory GC is that it limits the potential for composing a number of modules and the host together, because the garbage collector that manages JavaScript objects in a web browser knows nothing about your garbage collector over your linear memory. You can easily create memory leaks in a system like that.

Also, it’s pretty gross that a reference to an object in linear memory requires arbitrary read-write access over all of linear memory in order to read or write object fields. How do you build a reliable system without invariants?

Finally, once you collect garbage, and maybe you manage to compact memory, you can’t give anything back to the OS. There are proposals in the works but they are not there yet.

If the BOB audience had to choose between [Worse is Better and The Right
Thing](https://www.dreamsongs.com/WorseIsBetter.html), I think the BOB audience is much closer to the Right Thing. People like that feel instinctual revulsion to ugly systems and I think GC over linear memory describes an ugly system.

## GC and WebAssembly 1.0 (5)

*There is already a high-performance concurrent parallel compacting GC in the browser*

Halftime: C++ N – Altlangs 0

The kicker is that WebAssembly 1.0 requires you to write and deliver a terrible GC when there is already probably a great GC just sitting there in the host, one that has hundreds of person-years of effort invested in it, one that will surely do a better job than you could ever do. WebAssembly as hosted in a web browser should have access to the browser’s garbage collector!

I have the feeling that while those of us with a soft spot for languages with garbage collection have been standing on the sidelines, Rust and C++ people have been busy on the playing field scoring goals. Tripping over the ball, yes, but eventually they do manage to make within striking distance.

## Change is coming!

Support for built-in GC set to ship in Q4 2023

With GC, the material conditions are now in place

Let’s compile *our* languages to WebAssembly

But to continue the sportsball metaphor, I think in the second half our players will finally be able to get out on the pitch and give it the proverbial 110%. Support for garbage collection is coming to WebAssembly users, and I think even by the end of the year it will be shipping in major browsers. This is going to be big! We have a chance and we need to sieze it.

## Scheme to Wasm

Spritely + Igalia working on Scheme to WebAssembly

Avoid truncating language to platform; bring whole self

- **Value representation**
- Varargs
- Tail calls
- Delimited continuations
- Numeric tower

Even with GC, though, WebAssembly is still a weird machine. It would help to see the concrete approaches that some languages of interest manage to take when compiling to WebAssembly.

In that spirit, the rest of this article/presentation is a walkthough of the approach that I am taking as I work on a WebAssembly compiler for Scheme. (Thanks to [Spritely](https://spritely.institute) for supporting this work!)

Before diving in, a meta-note: when you go to compile a language to, say, JavaScript, you are mightily tempted to cut corners. For example you might implement numbers as JavaScript numbers, or you might omit implementing continuations. In this work I am trying to not cut corners, and instead to implement the language faithfully. Sometimes this means I have to work around weirdness in WebAssembly, and that’s OK.

When thinking about Scheme, I’d like to highlight a few specific areas that have interesting translations. We’ll start with value representation, which stays in the GC theme from the introduction.

## Scheme to Wasm: Values

```
;;       any  extern  func
;;        |
;;        eq
;;     /  |   \
;; i31 struct  array

```

The unitype: `(ref eq)`

Immediate values in `(ref i31)`

- fixnums with 30-bit range
- chars, bools, etc

Explicit nullability: `(ref null eq)` vs `(ref eq)`

The GC extensions for WebAssembly are phrased in terms of a type system. Oddly, there are three top types; as far as I understand it, this is the result of a compromise about how WebAssembly engines might want to represent these different kinds of values. For example, an opaque JavaScript value flowing into a WebAssembly program would have type `(ref extern)`. On a system with [NaN
boxing](https://wingolog.org/archives/2011/05/18/value-representation-in-javascript-implementations), you would need 64 bits to represent a JS value. On the other hand a native WebAssembly object would be a subtype of `(ref any)`, and might be representable in 32 bits, either because it’s a 32-bit system or because of pointer compression.

Anyway, three top types. The user can define subtypes of `struct` and `array`, instantiate values of those types, and access their fields. The life cycle of reference-typed objects is automatically managed by the run-time, which is just another way of saying they are garbage-collected.

For Scheme, we need a common supertype for all values: [the unitype, in
Bob Harper’s memorable
formulation](https://medium.com/@samth/on-typed-untyped-and-uni-typed-languages-8a3b4bedf68c). We can use `(ref any)`, but actually we’ll use `(ref eq)` – this is the supertype of values that can be compared by (pointer) identity. So now we can code up `eq?`:

```
(func $eq? (param (ref eq) (ref eq))
           (result i32)
  (ref.eq (local.get a) (local.get b)))

```

Generally speaking in a Scheme implementation there are *immediates* and *heap objects*. Immediates can be encoded in the bits of a value, whereas for heap object the bits of a value encode a reference (pointer) to an object on the garbage-collected heap. We usually represent small integers as immediates, as well as booleans and other oddball values.

Happily, WebAssembly gives us an immediate value type, `i31`. We’ll encode our immediates there, and otherwise represent heap objects as instances of `struct` subtypes.

## Scheme to Wasm: Values (2)

Heap objects subtypes of `struct`; concretely:

```
(struct $heap-object
  (struct (field $tag-and-hash i32)))
(struct $pair
  (sub $heap-object
    (struct i32 (ref eq) (ref eq))))

```

GC proposal allows subtyping on structs, functions, arrays

Structural type equivalance: explicit tag useful

We actually need to have a common struct supertype as well, for two reasons. One is that we need to be able to hash Scheme values by identity, but for this we need an embedded lazily-initialized hash code. It’s a bit annoying to take the per-object memory hit but it’s a reality, and the JVM does it this way, so it must not be so terrible.

The other reason is more subtle: WebAssembly’s type system is built in such a way that types that are “structurally” equivalent are indistinguishable. So a pair has two fields, besides the hash, but there might be a number of other fundamental object types that have the same shape; you can’t fully rely on WebAssembly’s dynamic type checks ( `ref.test` et al) to be able to query the type of a value. Instead we re-use the low bits of the hash word to include a type tag, which might be 1 for pairs, 2 for vectors, 3 for closures, and so on.

## Scheme to Wasm: Values (3)

```
(func $cons (param (ref eq)
                   (ref eq))
            (result (ref $pair))
  (struct.new_canon $pair
    ;; Assume heap tag for pairs is 1.
    (i32.const 1)
    ;; Car and cdr.
    (local.get 0)
    (local.get 1)))

(func $%car (param (ref $pair))
            (result (ref eq))
  (struct.get $pair 1 (local.get 0)))

```

With this knowledge we can define `cons`, as a simple call to `struct.new_canon pair`.

I didn’t have time for this in the talk, but there is a ghost haunting this code: the ghost of nominal typing. See, in a web browser at least, every heap object will have its first word point to its “hidden class” / “structure” / “map” word. If the engine ever needs to check that a value is of a specific shape, it can do a quick check on the map word’s value; if it needs to do deeper introspection, it can dereference that word to get more details.

Under the hood, testing whether a `(ref eq)` is a pair or not should be a simple check that it’s a `(ref struct)` (and not a fixnum), and then a comparison of its map word to the run-time type corresponding to `$pair`. If subtyping of `$pair` is allowed, we start to want inline caches to handle polymorphism, but the checking the map word is still the basic mechanism.

However, as I mentioned, we only have structural equality of types; two `(struct (ref eq))` type definitions will define the same type and have the same map word (run-time type / RTT). Hence the `_canon` in the name of `struct.new_canon $pair`: we create an instance of `$pair`, with the *canonical* run-time-type for objects having `$pair`-shape.

In earlier drafts of the WebAssembly GC extensions, users could define their own RTTs, which effectively amounts to nominal typing: not only does this object have the right structure, but was it created with respect to this particular RTT. But, this facility was cut from the first release, and it left ghosts in the form of these `_canon` suffixes on type constructor instructions.

For the Scheme-to-WebAssembly effort, we effectively add back in a degree of nominal typing via type tags. For better or for worse this results in a so-called “open-world” system: you can instantiate a separately-compiled WebAssembly module that happens to define the same types and use the same type tags and it will be able to happily access the contents of Scheme values from another module. If you were to use nominal types, you would’t be able to do so, unless there were some common base module that defined and exported the types of interests, and which any extension module would need to [import](https://github.com/WebAssembly/proposal-type-imports/blob/main/proposals/type-imports/Overview.md).

```
(func $car (param (ref eq)) (result (ref eq))
  (local (ref $pair))
  (block $not-pair
    (br_if $not-pair
      (i32.eqz (ref.test $pair (local.get 0))))
    (local.set 1 (ref.cast $pair) (local.get 0))
    (br_if $not-pair
      (i32.ne
        (i32.const 1)
        (i32.and
          (i32.const 0xff)
          (struct.get $heap-object 0 (local.get 1)))))
    (return_call $%car (local.get 1)))

  (call $type-error)
  (unreachable))

```

In the previous example we had `$%car`, with a funny `%` in the name, taking a `(ref $pair)` as an argument. But in the general case (barring compiler heroics) `car` will take an instance of the unitype `(ref eq)`. To know that it’s actually a pair we have to make two checks: one, that it is a struct and has the `$pair` shape, and two, that it has the right tag. Oh well!

## Scheme to Wasm

- *Value representation*
- **Varargs**
- Tail calls
- Delimited continuations
- Numeric tower

But with all of that I think we have a solid story on how to represent values. I went through all of the basic value types in Guile and checked that they could [all be represented using GC
types](https://gitlab.com/spritely/guile-hoot-updates/-/blob/main/examples/basic-types.wat), and it seems that all is good. Now on to the next point: varargs.

## Scheme to Wasm: Varargs

```
(list 'hey)      ;; => (hey)
(list 'hey 'bob) ;; => (hey bob)
```

Problem: Wasm functions strongly typed

```
(func $list (param ???) (result (ref eq))
  ???)
```

Solution: Virtualize calling convention

In WebAssembly, you define functions with a type, and it is impossible to call them in an unsound way. You must call `$car` exactly 2 arguments or it will not compile, and those arguments have to be of specific types, and so on. But Scheme doesn’t enforce these restrictions on the language level, bless its little miscreant heart. You can call `car` with 5 arguments, and you’ll get a run-time error. There are some functions that can take a variable number of arguments, doing different things depending on incoming argument count.

How do we square these two approaches to function types?

```
;; "Registers" for args 0 to 3
(global $arg0 (mut (ref eq)) (i31.new (i32.const 0)))
(global $arg1 (mut (ref eq)) (i31.new (i32.const 0)))
(global $arg2 (mut (ref eq)) (i31.new (i32.const 0)))
(global $arg3 (mut (ref eq)) (i31.new (i32.const 0)))

;; "Memory" for the rest
(type $argv (array (ref eq)))
(global $argN (ref $argv)
        (array.new_canon_default
          $argv (i31.const 42) (i31.new (i32.const 0))))
```

Uniform function type: argument count as sole parameter

Callee moves args to locals, possibly clearing roots

The approach we are taking is to virtualize the calling convention. In the same way that when calling an x86-64 function, you pass the first argument in `$rdi`, then `$rsi`, and eventually if you run out of registers you put arguments in memory, in the same way we’ll pass the first argument in the `$arg0` global, then `$arg1`, and eventually in memory if needed. The function will receive the number of incoming arguments as its sole parameter; in fact, all functions will be of type `(func (param i32))`.

The expectation is that after checking argument count, the callee will load its arguments from globals / memory to locals, which the compiler can do a better job on than globals. We might not even emit code to null out the argument globals; might leak a little memory but probably would be a win.

You can imagine a world in which `$arg0` actually gets globally allocated to `$rdi`, because it is only live during the call sequence; but I don’t think that world is this one :)

## Scheme to Wasm

- *Value representation*
- *Varargs*
- **Tail calls**
- Delimited continuations
- Numeric tower

Great, two points out of the way! Next up, tail calls.

## Scheme to Wasm: Tail calls

```
;; Call known function
(return_call $f arg ...)

;; Call function by value
(return_call_ref $type callee arg ...)
```

Friends – I almost cried making this slide. We Schemers are used to working around the lack of tail calls, and I could have done so here, but it’s just such a relief that these functions are just going to be there and I don’t have to think much more about them. Technically speaking the [proposal](https://github.com/WebAssembly/tail-call) isn’t merged yet; checking the [phases
document](https://github.com/WebAssembly/proposals) it’s at the last station before headed to the great depot in the sky. But, soon soon it will be present and enabled in all WebAssembly implementations, and we should build systems now that rely on it.

## Scheme to Wasm

- *Value representation*
- *Varargs*
- *Tail calls*
- **Delimited continuations**
- Numeric tower

Next up, my favorite favorite topic: delimited continuations.

## Scheme to Wasm: Prompts (1)

Problem: Lightweight threads/fibers, exceptions

Possible solutions

- Eventually, built-in coroutines
- [binaryen](https://github.com/WebAssembly/binaryen)’s asyncify (not yet ready for GC); see Julia

- **Delimited continuations**

“Bring your whole self”

Before diving in though, one might wonder why bother. Delimited continuations are a building-block that one can use to build other, more useful things, notably exceptions and light-weight threading / fibers. Could there be another way of achieving these end goals without having to implement this relatively uncommon primitive?

For fibers, it is possible to [implement them in terms of a built-in
coroutine
facility](https://wingolog.org/archives/2018/05/16/lightweight-concurrency-in-lua). The standards body seems willing to include a coroutine primitive, but it seems far off to me; not within the next 3-4 years I would say. So let’s put that to one side.

There is a more near-term solution, to use [asyncify to implement
coroutines](https://github.com/WebAssembly/design/issues/1294#issuecomment-520231415) somehow; but my understanding is that asyncify is not ready for GC yet.

For the Guile flavor of Scheme at least, delimited continuations are table stakes of their own right, so given that we will have them on WebAssembly, we might as well use them to implement fibers and exceptions in the same way as we do on native targets. Why compromise if you don’t have to?

## Scheme to Wasm: Prompts (2)

Prompts delimit continuations

```
(define k
  (call-with-prompt ’foo
    ; body
    (lambda ()
      (+ 34 (abort-to-prompt 'foo)))
    ; handler
    (lambda (continuation)
      continuation)))

(k 10)       ;; ⇒ 44
(- (k 10) 2) ;; ⇒ 42
```

`k` is the `_` in `(lambda () (+ 34 _))`

There are a few ways to implement [delimited
continuations](https://wingolog.org/archives/2010/02/26/guile-and-delimited-continuations), but my usual way of thinking about them is that a delimited continuation is a slice of the stack. One end of the slice is the *prompt* established by `call-with-prompt`, and the other by the continuation of the call to `abort-to-prompt`. Capturing a slice pops it off the stack, copying it out to the heap as a callable function. Calling that function splats the captured slice back on the stack and resumes it where it left off.

## Scheme to Wasm: Prompts (3)

Delimited continuations are stack slices

Make stack explicit via minimal continuation-passing-style conversion

- Turn all calls into tail calls
- Allocate return continuations on explicit stack
- Breaks functions into pieces at non-tail calls

This low-level intuition of what a delimited continuation is leads naturally to an implementation; the only problem is that we can’t slice the WebAssembly call stack. The workaround here is similar to the varargs case: we virtualize the stack.

The mechanism to do so is a continuation-passing-style (CPS) transformation of each function. Functions that make no calls, such as leaf functions, don’t need to change at all. The same goes for functions that make only tail calls. For functions that make non-tail calls, we split them into pieces that preserve the only-tail-calls property.

## Scheme to Wasm: Prompts (4)

Before a non-tail-call:

- Push live-out vars on stacks (one stack per top type)
- Push continuation as funcref
- Tail-call callee

Return from call via pop and tail call:

```
(return_call_ref (call $pop-return)
                 (i32.const 0))
```

After return, continuation pops state from stacks

Consider a simple function:

```
(define (f x y)
  (+ x (g y))

```

Before making a non-tail call, a “tailified” function will instead push all live data onto an explicitly-managed stack and tail-call the callee. It also pushes on the return continuation. Returning from the callee pops the return continuation and tail-calls it. The return continuation pops the previously-saved live data and continues.

In this concrete case, tailification would split `f` into two pieces:

```
(define (f x y)
  (push! x)
  (push-return! f-return-continuation-0)
  (g y))

(define (f-return-continuation-0 g-of-y)
  (define k (pop-return!))
  (define x (pop! x))
  (k (+ x g-of-y)))

```

Now there are no non-tail calls, besides calls to run-time routines like `push!` and `+` and so on. This transformation is implemented by [tailify.scm](https://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/language/cps/tailify.scm;h=f9ebb63d20b7ad736e74c39327d854de0fb798af;hb=refs/heads/wip-tailify).

## Scheme to Wasm: Prompts (5)

`abort-to-prompt`:

- Pop stack slice to reified continuation object
- Tail-call new top of stack: prompt handler

Calling a reified continuation:

- Push stack slice
- Tail-call new top of stack

No need to wait for effect handlers proposal; you can have it all now!

The salient point is that the stack on which `push!` operates (in reality, probably four or five stacks: one in linear memory or an array for types like `i32` or `f64`, three for each of the managed top types `any`, `extern`, and `func`, and one for the stack of return continuations) are managed by us, so we can slice them.

Someone asked in the talk about whether the explicit memory traffic and avoiding the return-address-buffer branch prediction is a source of inefficiency in the transformation and I have to say, yes, but I don’t know by how much. I guess we’ll find out soon.

## Scheme to Wasm

- *Value representation*
- *Varargs*
- *Tail calls*
- *Delimited continuations*
- **Numeric tower**

Okeydokes, last point!

## Scheme to Wasm: Numbers

Numbers can be immediate: fixnums

Or on the heap: bignums, fractions, flonums, complex

Supertype is still `ref eq`

Consider imports to implement bignums

- On web: `BigInt`
- On edge: Wasm support module (mini-gmp?)

Dynamic dispatch for polymorphic ops, as usual

First, I would note that sometimes the compiler can unbox numeric operations. For example if it infers that a result will be an inexact real, it can use unboxed `f64` instead of library routines working on heap flonums ( `(struct i32 f64)`; the initial i32 is for the hash and tag). But we still need a story for the general case that involves dynamic type checks.

The basic idea is that we get to have fixnums and heap numbers. Fixnums will handle most of the integer arithmetic that we need, and will avoid allocation. We’ll inline most fixnum operations as a fast path and call out to library routines otherwise. Of course fixnum inputs may produce a bignum output as well, so the fast path sometimes includes another slow-path callout.

We want to minimize binary module size. In an ideal compile-to-WebAssembly situation, a small program will have a small module size, down to a minimum of a kilobyte or so; larger programs can be megabytes, if the user experience allows for the download delay. Binary module size will be dominated by code, so that means we need to plan for aggressive dead-code elimination, minimize the size of fast paths, and also minimize the size of the standard library.

For numbers, we try to keep module size down by leaning on the platform. In the case of bignums, we can punt some of this work to the host; on a JavaScript host, we would use `BigInt`, and on a WASI host we’d compile an external bignum library. So that’s the general story: inlined fixnum fast paths with dynamic checks, and otherwise library routine callouts, combined with aggressive whole-program dead-code elimination.

## Scheme to Wasm

- *Value representation*
- *Varargs*
- *Tail calls*
- *Delimited continuations*
- *Numeric tower*

Hey I think we did it! Always before when I thought about compiling Scheme or Guile to the web, I got stuck on some point or another, was tempted down the corner-cutting alleys, and eventually gave up before starting. But finally it would seem that the stars are aligned: we get to have our Scheme and run it too.

## Miscellenea

Debugging: The wild west of DWARF; prompts

Strings: `stringref` host strings spark joy

JS interop: Export accessors; Wasm objects opaque to JS. `externref`.

JIT: [A whole ’nother talk!](https://wingolog.org/archives/2022/08/18/just-in-time-code-generation-within-webassembly)

AOT: wasm2c

Of course, like I said, WebAssembly is still a weird machine: as a compilation target but also at run-time. Debugging is a right proper mess; perhaps some other article on that some time.

How to represent strings is a surprisingly gnarly question; there is tension within the WebAssembly standards community between those that think that [it’s possible for JavaScript and WebAssembly to share an
underlying string
representation](https://github.com/WebAssembly/stringref/blob/main/proposals/stringref/Overview.md), and those that think that it’s a fool’s errand and that copying is the only way to go. I don’t know which side will prevail; perhaps more on that as well later on.

Similarly the whole interoperation with JavaScript question is very much in its early stages, with the current situation choosing to [err on the
side of nothing rather than the wrong
thing](https://github.com/WebAssembly/gc/issues/279). You can pass a WebAssembly `(ref eq)` to JavaScript, but JavaScript can’t do anything with it: it has no prototype. The state of the art is to also ship a JS run-time that wraps each wasm object, proxying exported functions from the wasm module as object methods.

Finally, some language implementations really need JIT support, like PyPy. There, that’s a whole ‘nother talk!

## WebAssembly for the rest of us

With GC, WebAssembly is now ready for us

Getting our languages on WebAssembly now a S.M.O.P.

Let’s score some goals in the second half!

```
(visit-links
 "gitlab.com/spritely/guile-hoot-updates"
 "wingolog.org"
 "wingo@igalia.com"
 "igalia.com"
 "mastodon.social/@wingo")
```

WebAssembly has proven to have some great wins for C, C++, Rust, and so on – but now it’s our turn to get in the game. GC is coming and we as a community need to be getting our compilers and language run-times ready. Let’s put on the coffee and bang some bytes together; it’s still early days and there’s a world to win out there for the language community with the best WebAssembly experience. The game is afoot: happy consing!

## related articles

- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [ahead-of-time wasm gc in wastrel](/archives/2026/02/06/ahead-of-time-wasm-gc-in-wastrel)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [missing the point of webassembly](/archives/2024/01/08/missing-the-point-of-webassembly)
- [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)
- [wastrel milestone: full hoot support, with generational gc as a treat](/archives/2026/04/09/wastrel-milestone-full-hoot-support-with-generational-gc-as-a-treat)

### One response

1. [Arne Babenhauserheide](https://www.draketo.de) says:[22 March 2023 9:03 PM](#5d0f70657043791c01701adc5a366ebf63ed4d11)

   Thank you for the wonderful writeup!

   I enjoyed reading it a lot and I now hope for great Guile support on the web!

Comments are closed.
