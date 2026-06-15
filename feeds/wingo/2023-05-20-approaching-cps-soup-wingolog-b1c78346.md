---
title: approaching cps soup — wingolog
url: https://wingolog.org/archives/2023/05/20/approaching-cps-soup
published: "2023-05-20T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/05/20/approaching-cps-soup
---

# approaching cps soup — wingolog

## [approaching cps soup](/archives/2023/05/20/approaching-cps-soup)

20 May 2023 7:10 AM

- [guile](/tags/guile)
- [igalia](/tags/igalia)
- [spritely](/tags/spritely)
- [cps soup](/tags/cps%20soup)
- [cps](/tags/cps)
- [compilers](/tags/compilers)
- [flow analysis](/tags/flow%20analysis)

Good evening, hackers. Today’s missive is more of a massive, in the sense that it’s another presentation transcript-alike; these things always translate to many vertical pixels.

In my defense, I hardly ever give a presentation twice, so not only do I miss out on the usual per-presentation cost amortization and on the incremental improvements of repetition, the more dire error is that whatever message I might have can only ever reach a subset of those that it might interest; here at least I can be more or less sure that if the presentation would interest someone, that they will find it.

So for the time being I will try to share presentations here, in the spirit of, well, why the hell not.

# CPS Soup

A functional intermediate language

10 May 2023 – Spritely

Andy Wingo

Igalia, S.L.

Last week I gave a training talk to [Spritely
Institute](https://spritely.instutute) collaborators on the intermediate representation used by [Guile](https://gnu.org/s/guile)‘s compiler.

## CPS Soup

Compiler: Front-end to Middle-end to Back-end

Middle-end spans gap between high-level source code (AST) and low-level machine code

Programs in middle-end expressed in intermediate language

CPS Soup is the language of Guile’s middle-end

An *intermediate representation* (IR) (or *intermediate language*, IL) is just another way to express a computer program. Specifically it’s the kind of language that is appropriate for the middle-end of a compiler, and by “appropriate” I meant that an IR serves a purpose: there has to be a straightforward transformation to the IR from high-level abstract syntax trees (ASTs) from the front-end, and there has to be a straightforward translation from IR to machine code.

There are also usually a set of necessary source-to-source transformations on IR to “lower” it, meaning to make it closer to the back-end than to the front-end. There are usually a set of optional transformations to the IR to make the program run faster or allocate less memory or be more simple: these are the *optimizations*.

“CPS soup” is Guile’s IR. This talk presents the essentials of CPS soup in the context of more traditional IRs.

## How to lower?

High-level:

```
(+ 1 (if x 42 69))
```

Low-level:

```
  cmpi $x, #f
  je L1
  movi $t, 42
  j L2
L1:
  movi $t, 69
L2:
  addi $t, 1
```

How to get from here to there?

Before we dive in, consider what we might call the dynamic range of an intermediate representation: we start with what is usually an algebraic formulation of a program and we need to get down to a specific sequence of instructions operating on registers (unlimited in number, at this stage; allocating to a fixed set of registers is a back-end concern), with explicit control flow between them. What kind of a language might be good for this? Let’s attempt to answer the question by looking into what the standard solutions are for this problem domain.

## 1970s

Control-flow graph (CFG)

```
graph := array
block := tuple
inst  := goto B
       | if x then BT else BF
       | z = const C
       | z = add x, y
       ...

BB0: if x then BB1 else BB2
BB1: t = const 42; goto BB3
BB2: t = const 69; goto BB3
BB3: t2 = addi t, 1; ret t2
```

Assignment, not definition

Of course in the early days, there was no intermediate language; compilers translated ASTs directly to machine code. It’s been a while since I dove into all this but the milestone I have in my head is that it’s the 70s when compiler middle-ends come into their own right, with Fran Allen’s work on flow analysis and optimization.

In those days the intermediate representation for a compiler was a graph of basic blocks, but unlike today the paradigm was assignment to locations rather than definition of values. By that I mean that in our example program, we get `t` assigned to in two places (BB1 and BB2); the actual definition of `t` is implicit, as a storage location, and our graph consists of assignments to the set of storage locations in the program.

## 1980s

Static single assignment (SSA) CFG

```
graph := array
block := tuple
phi   := z := φ(x, y, ...)
inst  := z := const C
       | z := add x, y
       ...
BB0: if x then BB1 else BB2
BB1: v0 := const 42; goto BB3
BB2: v1 := const 69; goto BB3
BB3: v2 := φ(v0,v1); v3:=addi t,1; ret v3
```

Phi is phony function: `v2` is `v0` if coming from first predecessor, or `v1` from second predecessor

These days we still live in Fran Allen’s world, but with a twist: we no longer model programs as graphs of assignments, but rather graphs of definitions. The introduction in the mid-80s of so-called “static single-assignment” (SSA) form graphs mean that instead of having two assignments to `t`, we would define two different values `v0` and `v1`. Then later instead of reading the value of the storage location associated with `t`, we define `v2` to be *either* `v0` or `v1`: the former if we reach the use of `t` in BB3 from BB1, the latter if we are coming from BB2.

If you think on the machine level, in terms of what the resulting machine code will be, this *either* function isn’t a real operation; probably register allocation will put `v0`, `v1`, and `v2` in the same place, say `$rax`. The function linking the definition of `v2` to the inputs `v0` and `v1` is purely notational; in a way, you could say that it is *phony*, or not real. But when the creators of SSA went to submit this notation for publication they knew that they would need something that sounded more rigorous than “phony function”, so they instead called it a “phi” (φ) function. Really.

## 2003: MLton

Refinement: phi variables are basic block args

```
graph := array
block := tuple
```

Inputs of phis implicitly computed from preds

```
BB0(a0): if a0 then BB1() else BB2()
BB1(): v0 := const 42; BB3(v0)
BB2(): v1 := const 69; BB3(v1)
BB3(v2): v3 := addi v2, 1; ret v3
```

SSA is still where it’s at, as a conventional solution to the IR problem. There have been some refinements, though. I learned of one of them from [MLton](https://mlton.org/); I don’t know if they were first but they had the idea of interpreting phi variables as *arguments* to basic blocks. In this formulation, you don’t have explicit phi instructions; rather the “ `v2` is either `v1` or `v0`” property is expressed by `v2` being a parameter of a block which is “called” with either `v0` or `v1` as an argument. It’s the same semantics, but an interesting notational change.

## Refinement: Control tail

Often nice to know how a block ends (e.g. to compute phi input vars)

```
graph := array
block := tuple
control := if v then L1 else L2
         | L(v, ...)
         | switch(v, L1, L2, ...)
         | ret v
```

One other refinement to SSA is to note that basic blocks consist of some number of instructions that can define values or have side effects but which otherwise exhibit fall-through control flow, followed by a single instruction that transfers control to another block. We might as well store that control instruction separately; this would let us easily know how a block ends, and in the case of phi block arguments, easily say what values are the inputs of a phi variable. So let’s do that.

## Refinement: DRY

Block successors directly computable from control

Predecessors graph is inverse of successors graph

```
graph := array
block := tuple
```

Can we simplify further?

At this point we notice that we are repeating ourselves; the successors of a block can be computed directly from the block’s terminal control instruction. Let’s drop those as a distinct part of a block, because when you transform a program it’s unpleasant to have to needlessly update something in two places.

While we’re doing that, we note that the predecessors array is also redundant, as it can be computed from the graph of block successors. Here we start to wonder: am I simpliying or am I removing something that is fundamental to the algorithmic complexity of the various graph transformations that I need to do? We press on, though, hoping we will get somewhere interesting.

## Basic blocks are annoying

Ceremony about managing insts; array or doubly-linked list?

Nonuniformity: “local” vs ‘\`global’' transformations

Optimizations transform graph A to graph B; mutability complicates this task

- Desire to keep A in mind while making B
- Bugs because of spooky action at a distance

Recall that the context for this meander is Guile’s compiler, which is written in Scheme. Scheme doesn’t have expandable arrays built-in. You can build them, of course, but it is annoying. Also, in Scheme-land, functions with side-effects are conventionally suffixed with an exclamation mark; after too many of them, both the writer and the reader get fatigued. I know it’s a silly argument but it’s one of the things that made me grumpy about basic blocks.

If you permit me to continue with this introspection, I find there is an uneasy relationship between instructions and locations in an IR that is structured around basic blocks. Do instructions live in a function-level array and a basic block is an array of instruction indices? How do you get from instruction to basic block? How would you hoist an instruction to another basic block, might you need to reallocate the block itself?

And when you go to transform a graph of blocks... well how do you do that? Is it in-place? That would be efficient; but what if you need to refer to the original program during the transformation? Might you risk reading a stale graph?

It seems to me that there are too many concepts, that in the same way that SSA itself moved away from assignment to a more declarative language, that perhaps there is something else here that might be more appropriate to the task of a middle-end.

## Basic blocks, phi vars redundant

Blocks: label with args sufficient; “containing” multiple instructions is superfluous

Unify the two ways of naming values: every var is a phi

```
graph := array
block := tuple
inst  := L(expr)
       | if v then L1() else L2()
       ...
expr  := const C
       | add x, y
       ...
```

I took a number of tacks here, but the one I ended up on was to declare that basic blocks themselves are redundant. Instead of containing an array of instructions with fallthrough control-flow, why not just make every instruction a control instruction? (Yes, there are arguments against this, but do come along for the ride, we get to a funny place.)

While you are doing that, you might as well unify the two ways in which values are named in a MLton-style compiler: instead of distinguishing between basic block arguments and values defined within a basic block, we might as well make all names into basic block arguments.

## Arrays annoying

Array of blocks implicitly associates a label with each block

Optimizations add and remove blocks; annoying to have dead array entries

Keep labels as small integers, but use a map instead of an array

```
graph := map
```

In the traditional SSA CFG IR, a graph transformation would often not touch the structure of the graph of blocks. But now having given each instruction its own basic block, we find that transformations of the program necessarily change the graph. Consider an instruction that we elide; before, we would just remove it from its basic block, or replace it with a no-op. Now, we have to find its predecessor(s), and forward them to the instruction’s successor. It would be useful to have a more capable data structure to represent this graph. We might as well keep labels as being small integers, but allow for sparse maps and growth by using an integer-specialized map instead of an array.

## This is CPS soup

```
graph := mapcont>
cont  := tupleterm>
term  := continue to L
           with values from expr
       | if v then L1() else L2()
       ...
expr  := const C
       | add x, y
       ...

```

SSA is CPS

This is exactly what CPS soup is! We came at it “from below”, so to speak; instead of the heady fumes of the lambda calculus, we get here from down-to-earth basic blocks. (If you prefer the other way around, you might enjoy [this article from a long time
ago](https://wingolog.org/archives/2011/07/12/static-single-assignment-for-functional-programmers).) The remainder of this presentation goes deeper into what it is like to work with CPS soup in practice.

## Scope and dominators

```
BB0(a0): if a0 then BB1() else BB2()
BB1(): v0 := const 42; BB3(v0)
BB2(): v1 := const 69; BB3(v1)
BB3(v2): v3 := addi v2, 1; ret v3
```

What vars are “in scope” at BB3? `a0` and `v2`.

Not `v0`; not all paths from BB0 to BB3 define `v0`.

`a0` always defined: its definition *dominates* all uses.

BB0 dominates BB3: All paths to BB3 go through BB0.

Before moving on, though, we should discuss what it means in an SSA-style IR that variables are defined rather than assigned. If you consider variables as locations to which values can be assigned and which initially hold garbage, you can read them at any point in your program. You might get garbage, though, if the variable wasn’t assigned something sensible on the path that led to reading the location’s value. It sounds bonkers but it is still the C and C++ semantic model.

If we switch instead to a definition-oriented IR, then a variable never has garbage; the single definition always precedes any uses of the variable. That is to say that all paths from the function entry to the use of a variable must pass through the variable’s definition, or, in the jargon, that *definitions dominate uses*. This is an invariant of an SSA-style IR, that all variable uses be dominated by their associated definition.

You can flip the question around to ask what variables are available for use at a given program point, which might be read equivalently as which variables are in scope; the answer is, all definitions from all program points that dominate the use site. The “CPS” in “CPS soup” stands for *continuation-passing style*, a dialect of the lambda calculus, which has also has a history of use as a compiler intermediate representation. But it turns out that if we use the lambda calculus in its conventional form, we end up needing to maintain a lexical scope nesting at the same time that we maintain the control-flow graph, and the lexical scope tree can fail to reflect the dominator tree. I go into this topic in more detail in [an old
article](https://wingolog.org/archives/2015/07/27/cps-soup), and if it interests you, please do go deep.

## CPS soup in Guile

Compilation unit is intmap of label to cont

```
cont := $kargs names vars term
      | ...
term := $continue k src expr
      | ...
expr := $const C
      | $primcall ’add #f (a b)
      | ...
```

Conventionally, entry point is lowest-numbered label

Anyway! In Guile, the concrete form that CPS soup takes is that a program is an intmap of *label* to *cont*. A cont is the smallest labellable unit of code. You can call them blocks if that makes you feel better. One kind of cont, `$kargs`, binds incoming values to variables. It has a list of variables, *vars*, and also has an associated list of human-readable names, *names*, for debugging purposes.

A `$kargs` contains a *term*, which is like a control instruction. One kind of term is `$continue`, which passes control to a continuation *k*. Using our earlier language, this is just `goto *k*`, with values, as in MLton. (The *src* is a source location for the term.) The values come from the term’s *expr*, of which there are a dozen kinds or so, for example `$const` which passes a literal constant, or `$primcall`, which invokes some kind of primitive operation, which above is `add`. The primcall may have an immediate operand, in this case `#f`, and some variables that it uses, in this case `a` and `b`. The number and type of the produced values is a property of the primcall; some are just for effect, some produce one value, some more.

## CPS soup

```
term := $continue k src expr
      | $branch kf kt src op param args
      | $switch kf kt* src arg
      | $prompt k kh src escape? tag
      | $throw src op param args
```

Expressions can have effects, produce values

```
expr := $const val
      | $primcall name param args
      | $values args
      | $call proc args
      | ...
```

There are other kinds of terms besides `$continue`: there is `$branch`, which proceeds either to the false continuation *kf* or the true continuation *kt* depending on the result of performing *op* on the variables *args*, with immediate operand *param*. In our running example, we might have made the initial term via:

```
(build-term
  ($branch BB1 BB2 'false? #f (a0)))

```

The definition of `build-term` (and `build-cont` and `build-exp`) is in the [`(language cps)`](https://www.gnu.org/software/guile/manual/html_node/Building-CPS.html) module.

There is also `$switch`, which takes an unboxed unsigned integer *arg* and performs an array dispatch to the continuations in the list *kt*, or *kf* otherwise.

There is `$prompt` which continues to its *k*, having pushed on a new continuation delimiter associated with the var *tag*; if code aborts to *tag* before the prompt exits via an `unwind` primcall, the stack will be unwound and control passed to the handler continuation *kh*. If *escape?* is true, the continuation is escape-only and aborting to the prompt doesn’t need to capture the suspended continuation.

Finally there is `$throw`, which doesn’t continue at all, because it causes a non-resumable exception to be thrown. And that’s it; it’s just a handful of kinds of term, determined by the different shapes of control-flow (how many continuations the term has).

When it comes to values, we have about a dozen expression kinds. We saw `$const` and `$primcall`, but I want to explicitly mention `$values`, which simply passes on some number of values. Often a `$values` expression corresponds to passing an input to a phi variable, though `$kargs` vars can get their definitions from any expression that produces the right number of values.

## Kinds of continuations

Guile functions untyped, can multiple return values

Error if too few values, possibly truncate too many values, possibly cons as rest arg...

Calling convention: contract between val producer & consumer

- both on call and return side

Continuation of `$call` unlike that of `$const`

When a `$continue` term continues to a `$kargs` with a `$const 42` expression, there are a number of invariants that the compiler can ensure: that the `$kargs` continuation is always passed the expected number of values, that the *vars* that it binds can be allocated to specific locations (e.g. registers), and that because all predecessors of the `$kargs` are known, that those predecessors can place their values directly into the variable’s storage locations. Effectively, the compiler determines a *custom calling convention* between each `$kargs` and its predecessors.

Consider the `$call` expression, though; in general you don’t know what the callee will do to produce its values. You don’t even generally know that it will produce the right number of values. Therefore `$call` can’t (in general) continue to `$kargs`; instead it continues to `$kreceive`, which expects the return values in well-known places. `$kreceive` will check that it is getting the right number of values and then continue to a `$kargs`, shuffling those values into place. A *standard calling* *convention* defines how functions return values to callers.

## The conts

```
cont := $kfun src meta self ktail kentry
      | $kclause arity kbody kalternate
      | $kargs names syms term
      | $kreceive arity kbody
      | $ktail
```

$kclause, $kreceive very similar

Continue to $ktail: return

$call and return (and $throw, $prompt) exit first-order flow graph

Of course, a `$call` expression could be a tail-call, in which case it would continue instead to `$ktail`, indicating an exit from the first-order function-local control-flow graph.

The calling convention also specifies how to pass arguments to callees, and likewise those continuations have a fixed calling convention; in Guile we start functions with `$kfun`, which has some metadata attached, and then proceed to `$kclause` which bridges the boundary between the standard calling convention and the specialized graph of `$kargs` continuations. (Many details of this could be tweaked, for example that the `case-lambda` dispatch built-in to `$kclause` could instead dispatch to distinct functions instead of to different places in the same function; historical accidents abound.)

As a detail, if a function is *well-known*, in that all its callers are known, then we can lighten the calling convention, moving the argument-count check to callees. In that case `$kfun` continues directly to `$kargs`. Similarly for return values, optimizations can make `$call` continue to `$kargs`, though there is still some value-shuffling to do.

## High and low

CPS bridges AST (Tree-IL) and target code

High-level: vars in outer functions in scope

Closure conversion between high and low

Low-level: Explicit closure representations; access free vars through closure

CPS soup is the bridge between parsed Scheme and machine code. It starts out quite high-level, notably allowing for nested scope, in which expressions can directly refer to free variables. Variables are small integers, and for high-level CPS, variable indices have to be unique across all functions in a program. CPS gets lowered via [closure
conversion](https://wingolog.org/archives/2016/02/08/a-lambda-is-not-necessarily-a-closure), which chooses specific representations for each closure that remains after optimization. After closure conversion, all variable access is local to the function; free variables are accessed via explicit loads from a function’s closure.

## Optimizations at all levels

Optimizations before and after lowering

Some exprs only present in one level

Some high-level optimizations can merge functions (higher-order to first-order)

Because of the broad remit of CPS, the language itself has two dialects, high and low. The high level dialect has cross-function variable references, first-class abstract functions (whose representation hasn’t been chosen), and recursive function binding. The low-level dialect has only specific ways to refer to functions: labels and specific closure representations. It also includes calls to function labels instead of just function values. But these are minor variations; some optimization and transformation passes can work on either dialect.

## Practicalities

Intmap, intset: Clojure-style persistent functional data structures

Program: `intmap`

Optimization: `program→program`

Identify functions: `(program,label)→intset`

Edges: `intmap>`

Compute succs: `(program,label)→edges`

Compute preds: `edges→edges`

I mentioned that programs were intmaps, and specifically in Guile they are Clojure/Bagwell-style persistent functional data structures. By *functional* I mean that intmaps (and intsets) are values that can’t be mutated in place (though we do have the [transient
optimization](https://clojure.org/reference/transients)).

I find that immutability has the effect of deploying a sense of calm to the compiler hacker – I don’t need to worry about data structures changing out from under me; instead I just structure all the transformations that you need to do as functions. An optimization is just a function that takes an intmap and produces another intmap. An analysis associating some data with each program label is just a function that computes an intmap, given a program; that analysis will never be invalidated by subsequent transformations, because the program to which it applies will never be mutated.

This pervasive feeling of calm allows me to tackle problems that I wouldn’t have otherwise been able to fit into my head. One example is the novel [online CSE
pass](https://github.com/wingo/online-cse/blob/main/online-cse.md); one day I’ll either wrap that up as a paper or just capitulate and blog it instead.

## Flow analysis

```
A[k] = meet(A[p] for p in preds[k])
         - kill[k] + gen[k]
```

Compute available values at labels:

- A: `intmap>`
- meet: `intmap-intersect`
- -, +: `intset-subtract`, `intset-union`
- kill\[k\]: values invalidated by cont because of side effects
- gen\[k\]: values defined at k

But to keep it concrete, let’s take the example of flow analysis. For example, you might want to compute “available values” at a given label: these are the values that are candidates for common subexpression elimination. For example if a term is dominated by a `car x` primcall whose value is bound to `v`, and there is no path from the definition of V to a subsequent `car x` primcall, we can replace that second duplicate operation with `$values (v)` instead.

There is a standard solution for this problem, which is to solve the flow equation above. I wrote about this at length [ages
ago](https://wingolog.org/archives/2014/07/01/flow-analysis-in-guile), but looking back on it, the thing that pleases me is how easy it is to decompose the task of flow analysis into manageable parts, and how the types tell you exactly what you need to do. It’s easy to compute an initial analysis A, easy to define your meet function when your maps and sets have built-in intersect and union operators, easy to define what addition and subtraction mean over sets, and so on.

## Persistent data structures FTW

- meet: `intmap-intersect`
- -, +: `intset-subtract`, `intset-union`

Naïve: O(nconts \* nvals)

Structure-sharing: O(nconts \* log(nvals))

Computing an analysis isn’t free, but it is manageable in cost: the structure-sharing means that `meet` is usually trivial (for fallthrough control flow) and the cost of `+` and `-` is proportional to the log of the problem size.

## CPS soup: strengths

Relatively uniform, orthogonal

Facilitates functional transformations and analyses, lowering mental load: “I just have to write a function from foo to bar; I can do that”

Encourages global optimizations

Some kinds of bugs prevented by construction (unintended shared mutable state)

We get the SSA optimization literature

Well, we’re getting to the end here, and I want to take a step back. Guile has used CPS soup as its middle-end IR for about 8 years now, enough time to appreciate its fine points while also understanding its weaknesses.

On the plus side, it has what to me is a kind of low cognitive overhead, and I say that not just because I came up with it: Guile’s development team is small and not particularly well-resourced, and we can’t afford complicated things. The simplicity of CPS soup works well for our development process (flawed though that process may be!).

I also like how by having every variable be potentially a phi, that any optimization that we implement will be global (i.e. not local to a basic block) by default.

Perhaps best of all, we get these benefits while also being able to use the existing SSA transformation literature. Because CPS is SSA, the lessons learned in SSA (e.g. loop peeling) apply directly.

## CPS soup: weaknesses

Pointer-chasing, indirection through intmaps

Heavier than basic blocks: more control-flow edges

Names bound at continuation only; phi predecessors share a name

Over-linearizes control, relative to sea-of-nodes

Overhead of re-computation of analyses

CPS soup is not without its drawbacks, though. It’s not suitable for JIT compilers, because it imposes some significant constant-factor (and sometimes algorithmic) overheads. You are always indirecting through intmaps and intsets, and these data structures involve significant pointer-chasing.

Also, there are some forms of lightweight flow analysis that can be performed naturally on a graph of basic blocks without looking too much at the contents of the blocks; for example in our available variables analysis you could run it over blocks instead of individual instructions. In these cases, basic blocks themselves are an optimization, as they can reduce the size of the problem space, with corresponding reductions in time and memory use for analyses and transformations. Of course you could overlay a basic block graph on top of CPS soup, but it’s not a well-worn path.

There is a little detail that not all phi predecessor values have names, since names are bound at successors (continuations). But this is a detail; if these names are important, little `$values` trampolines can be inserted.

Probably the main drawback as an IR is that the graph of conts in CPS soup over-linearizes the program. There are [other intermediate
representations](https://dl.acm.org/doi/pdf/10.1145/207110.207154) that don’t encode ordering constraints where there are none; perhaps it would be useful to marry CPS soup with sea-of-nodes, at least during some transformations.

Finally, CPS soup does not encourage a style of programming where an analysis is incrementally kept up to date as a program is transformed in small ways. The result is that we end up performing much redundant computation within each individual optimization pass.

## Recap

CPS soup is SSA, distilled

Labels and vars are small integers

Programs map labels to conts

Conts are the smallest labellable unit of code

Conts can have terms that continue to other conts

Compilation simplifies and lowers programs

Wasm vs VM backend: a question for another day :)

But all in all, CPS soup has been good for Guile. It’s just SSA by another name, in a simpler form, with a functional flavor. Or, it’s just CPS, but first-order only, without lambda.

In the near future, I am interested in seeing what a [new
GC](https://wingolog.org/archives/2023/02/07/whippet-towards-a-new-local-maximum) will do for CPS soup; will bump-pointer allocation palliate some of the costs of pointer-chasing? We’ll see. A tricky thing about CPS soup is that I don’t think that anyone else has tried it in other languages, so it’s hard to objectively understand its characteristics independent of Guile itself.

Finally, it would be nice to engage in the academic conversation by publishing a paper somewhere; I would like to see interesting criticism, and blog posts don’t really participate in the citation graph. But in the [limited time available to
me](https://wingolog.org/archives/2023/04/18/sticking-point), faced with the choice between hacking on something and writing a paper, it’s always been hacking, so far :)

Speaking of limited time, I probably need to hit publish on this one and move on. Happy hacking to all, and until next time.

## related articles

- [needed-bits optimizations in guile](/archives/2024/09/26/needed-bits-optimizations-in-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [flow analysis in guile](/archives/2014/07/01/flow-analysis-in-guile)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [unboxing in guile](/archives/2016/01/19/unboxing-in-guile)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)

Comments are closed.
