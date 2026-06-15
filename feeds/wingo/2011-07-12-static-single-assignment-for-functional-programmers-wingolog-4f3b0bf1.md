---
title: static single assignment for functional programmers — wingolog
url: https://wingolog.org/archives/2011/07/12/static-single-assignment-for-functional-programmers
published: "2011-07-12T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/07/12/static-single-assignment-for-functional-programmers
---

# static single assignment for functional programmers — wingolog

## [static single assignment for functional programmers](/archives/2011/07/12/static-single-assignment-for-functional-programmers)

12 July 2011 4:39 PM

- [compilers](/tags/compilers)
- [ssa](/tags/ssa)
- [cps](/tags/cps)
- [scheme](/tags/scheme)
- [igalia](/tags/igalia)
- [v8](/tags/v8)
- [mlton](/tags/mlton)
- [guile](/tags/guile)
- [anf](/tags/anf)

Friends, I have an admission to make: I am a functional programmer.

By that I mean that [lambda](http://en.wikipedia.org/wiki/Lambda_calculus) is my tribe. And you know how tribalism works: when two tribes meet, it's usually to argue and not to communicate.

So it is that I've been well-indoctrinated in the lore of the lambda calculus, continuation-passing style intermediate languages, closure conversion and lambda lifting. But when it comes to ideas from outside our tribe, we in the lambda tribe tune out, generally.

At the last Scheme workshop in Montreal, some poor fellow had the temerity to mention SSA on stage. (SSA is what the "machine tribe" uses as an intermediate langauge in their compilers.) I don't think the "A" was out of his mouth before Olin Shivers' booming drawl started, "d'you mean CPS?" (CPS is what "we" use.) There were titters from the audience, myself included.

But there are valuable lessons to be learned from SSA language and the optimizations that it enables, come though it may from another tribe. In this article I'd like to look at what the essence of SSA is. To do so, I'll start with explaining the functional programming story on intermediate languages, as many of my readers are not of my tribe. Then we'll use that as a fixed point against which SSA may be compared.

**the lambda tribe in two sentences**

In the beginning was the [lambda](http://en.wikipedia.org/wiki/Lambda_calculus). God saw it, realized he didn't need anything else, and stopped there.

![](//wingolog.org/pub/the-creation-of-lisp.jpg)

*functional programmers got god's back*

Hey, it's true, right? The lambda-calculus is great because of its expressivity and precision. In that sense this evaluation is a utilitarian one: the lambda-calculus allows us to reason about computation with precision, so it is worth keeping around.

I don't think that Church was thinking about digital computers when he came up with the lambda-calculus back in the 1930s, given that digital computers didn't exist yet. Nor was McCarthy thinking about computers when he came up with Lisp in the 1960s. But one of McCarthy's students did hack it up, and that's still where we are now: translating between the language of the lambda-calculus and machine language.

This translation process is compilation, of course. For the first 20 years or so of practicing computer science, compilers (and indeed, languages) were very ad-hoc. In the beginning they didn't exist, and you just wrote machine code directly, using switches on a control panel or other such things, and later, assembly language. But eventually folks figured out parsing, and you get the first compilers for high-level languages.

I've written before about [C not being a high-level assembly language](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know.html), but back then, FORTRAN was indeed such a language. There wasn't much between the parser and the code generator. Everyone knows how good compilers work these days: you parse, you optimize, then you generate code. The medium in which you do your work is your *intermediate language*. A good intermediate language should be simple, so your optimizer can be simple; expressive, so that you can easily produce it from your source program; and utilitarian, in that its structure enables the kinds of optimizations that you want to make.

The lambda tribe already had a good intermediate language in this regard, in the form of the lambda-calculus itself. In many ways, solving a logic problem in the lambda-calculus is a lot like optimizing a program. Copy propagation is [beta-reduction](http://en.wikipedia.org/wiki/Lambda_calculus#.CE.B2-reduction). [Inlining](http://en.wikipedia.org/wiki/Inlining) is copy propagation extended to lambda expressions. Eta-conversion of continuations eliminates "forwarding blocks" -- basic blocks which have no statements, and just jump to some other continuation. Eta-conversion of functions eliminates functional trampolines.

**continuation-passing style**

But I'm getting ahead of myself. In the lambda tribe, we don't actually program in the lambda-calculus, you see. If you read any of our papers there's always a section in the beginning that defines the language we're working in, and then defines its semantics as a translation to the lambda-calculus.

This translation is always possible, for any programming language, and indeed Peter Landin did so in 1965 for Algol. Landin's original translations used his "J operator" to capture continuations, allowing a more direct translation of code into the lambda-calculus.

I wrote more on [Landin, letrec, and the Y combinator](//wingolog.org/archives/2009/08/10/goto-) a couple of years ago, but I wanted to mention one recent paper that takes a modern look at J, [A Rational Deconstruction of Landin's J Operator](http://www.brics.dk/RS/06/17/index.html). This paper is co-authored by V8 hacker Kevin Millikin, and cites work by V8 hackers Mads Sig Ager and Lasse R. Nielsen. Indeed all three seem to have had the privilege of having Olivier Danvy as PhD advisor. That's my tribe!

Anyway, J was useful in the context of Landin's abstract SECD machine, used to investigate the semantics of programs and programming languages. However it does not help the implementor of a compiler to a normal machine, and intermediate languages are all about utility. The answer to this problem, for functional programmers, was to convert the source program to what is known as *continuation-passing style* (CPS).

With CPS, the program is turned inside out. So instead of `(+ 1 (f (+ 2 3)))`, you would have:

```
lambda return
  let c1 = 1
    letcont k1 t1 = _+ return c1 t1
      letcont k2 t2 = f k1 t2
        let c2 = 2, c3 = 3
          _+ k2 c2 c3

```

Usually the outer lambda is left off, as it is implicit. Every call in a CPS program is a tail call, for the purposes of the lambda calculus. Continuations are explicitly represented as lambda expressions. Every function call or primitive operation takes the continuation as an argument. Papers in this field usually use Church's original lambda-calculus notation instead of the ML-like notation I give here. Continuations introduced by a CPS transformation are usually marked as such, so that they can be efficiently compiled later, without any flow analysis.

Expressing a program in CPS has a number of practical advantages:

- CPS is capable of expressing higher-order control-flow, for languages in which functions may be passed as values.

- All temporary values are named. Unreferenced names represent dead code, or code compiled for effect only. Referenced names are the natural input to a register allocator.

- Continuations correspond to named basic blocks. Their names in the source code correspond to a natural flow analysis simply by tracing the definitions and uses of the names. Flow analysis enables more optimizations, like code motion.

- Full beta-reduction is sound on terms of this type, even in call-by-value languages.

Depending on how you implement your CPS language, you can also attach notes to different continuations to help your graph reduce further: this continuation is an effect context (because its formal parameter is unreferenced in its body, or because you knew that when you made it), so its caller can be processed for effect and not for value; this one is of variable arity (e.g. can receive one or two values), so we could jump directly to the right handler, depending on what we want; etc. Guile's compiler is not in CPS right now, but I am thinking of rewriting it for this reason, to allow more transparent handling of control flow.

Note that nowhere in there did I mention Scheme's `call-with-current-continuation`! For me, the utility of CPS is in its explicit naming of temporaries, continuations, and its affordances for optimization. Call/cc is a rare construct in Guile Scheme, that could be optimized better with CPS, but one that I don't care a whole lot about, because it's often impossible to prove that the continuation doesn't escape, and in that case you're on the slow path anyway.

So that's CPS. Continuations compile to jumps within a function, and functions get compiled to closures, or labels for toplevel functions. The best reference I can give on it is Andrew Kennedy's 2007 paper, [Compiling With Continuations, Continued](http://research.microsoft.com/~akenn/sml/CompilingWithContinuationsContinued.pdf). CWCC is a really fantastic paper and I highly recommend it.

**a digression: anf**

CPS fell out of favor in the nineties, in favor of what became known as Administrative Normal Form, or ANF. ANF is like CPS except instead of naming the continuations, the code is left in so-called "direct-style", in which the continuations are implicit. So my previous example would look like this:

```
let c2 = 2, c3 = 3
  let t2 = + c2 c3
    let t1 = f t2
      let c1 = 1
        + c1 t1

```

There are ANF correspondences for CPS reductions, like the beta-rule. See the [Essence of Compiling With Continuations](http://www.ccs.neu.edu/scheme/pubs/pldi93-fsdf.ps.gz) paper, which introduced ANF and sparked the decline of the original CPS formulation, for more.

This CPS-vs-ANF discussion still goes on, even now in 2011. In particular, Kennedy's CWCC paper is quite compelling. But the debate has been largely mooted by the advances made by the machine tribe, as enabled by their SSA intermediate language.

**the machine tribe in two sentences**

In the beginning was the

`Segmentation fault (core dumped)`

(Just kidding, guys & ladies!)

Instead of compiling abstract ideas of naming and control to existing hardware, as the lambda tribe did, the machine tribe took as a given the hardware available, and tries to expose the capabilities of the machine to the programmer.

The machine tribe doesn't roll with closures, continuations, or tail calls. But they do have loops, and they crunch a lot of data. The most important thing for a compiler of a machine-tribe language like C is to produce efficient machine code for loops.

Clearly, I'm making some simplifications here. But if you look at a machine-tribe language like Java, you will be dealing with many control-flow constructs that are built-in to the language ( `for`, `while`, etc.) instead of layered on top of recursion like loops in Scheme. What this means is that large, important parts of your program have already collapsed to a first-order control-flow graph problem. Layering other optimizations on top of this like inlining ( [the mother of all optimizations](//wingolog.org/archives/2011/07/05/v8-a-tale-of-two-compilers#788347f5d21641a7115ba069f58715848dba9850)) only expands this first-order flow graph. More on "first-order" later.

So! After decades of struggling with this problem, after having abstracted away from assembly language to three-address register transfer language, finally the machine folks came up with something truly great: static single-assignment (SSA) form. The arc here is away from the machine, and towards more abstraction, in order to be able to optimize better, and thus generate better code.

It's precisely for this utilitarian reason that SSA was developed. Consider one of the earliest SSA papers, [Global Value Numbers and Redundant Comparisons](http://www.cs.wustl.edu/~cytron/cs531/Resources/Papers/valnum.pdf) by Rosen, Wegman, and Zadeck. Rosen et al were concerned about being able to move invariant expressions out of loops, extending the "value numbering" technique to operate across basic blocks. But the assignment-oriented intermediate languages that they had been using were getting in the way of code motion.

To fix this issue, Rosen et al switched from the assignment-oriented model of the machine tribe to the *binding-oriented* model of the lambda tribe.

In SSA, variables are never mutated (assigned); they are bound once and then left alone. Assignment to a source-program variable produces a new binding in the SSA intermediate language.

For the following function:

```
function clamp (x, lower, upper) {
  if (x < lower)
    x = lower;
  else if (x > upper)
    x = upper;
  return x;
}

```

The SSA translation would be:

```
entry:
  x0, lower0, upper0 = args;
  goto b0;

b0:
  t0 = x0 < lower0;
  goto t0 ? b1 : b2;

b1:
  x1 = lower0;
  goto exit;

b2:
  t1 = x0 > upper0;
  goto t1 ? b3 : exit;

b3:
  x2 = upper0;
  goto exit;

exit:
  x4 = phi(x0, x1, x2);
  return x4;

```

SSA form breaks down a procedure into basic blocks, each of which ends with a branch to another block, either conditional or unconditional. Usually temporary values receive their own names as well, as it facilitates optimization.

**phony functions**

The funny thing about SSA is the last bit, the "phi" function. Phi functions are placed at control-flow joins. In our case, the value of `x` may be proceed from the argument or from the assignment in the first or second `if` statement. The `phi` function indicates that.

But you know, lambda tribe, I didn't really get what this meant. What *is* a phi function? It doesn't help to consider where the name comes from, that the original IBM compiler hackers put in a "phony" function to merge the various values, but considered that "phi" was a better name if they wanted to be taken seriously by journal editors.

Maybe phi functions are intuitive to the machine tribe; I don't know. I doubt it. But fortunately there is another interpretation: that each basic block is a function, and that a phi function indicates that the basic block has an argument.

Like this:

```
entry x lower upper =
  letrec b0 = let t0 = x0 < lower
                if t0 then b1() else b2()
         b1 = let x = lower
                exit(x);
         b2 = let t1 = x > upper0
                if t1 then b3() else exit(x)
         b3 = let x = upper
                exit(x);
         exit x = x
    b0()

```

Here I have represented basic blocks as named functions instead of labels. Instead of phi functions, we allow the blocks to take a number of arguments; the call sites determine the values that the phi function may take on.

Note that all calls to blocks are tail calls. Reminds you of CPS, doesn't it? For more, see Richard Kelsey's classic paper, [A Correspondence Between Continuation-Passing Style and Static Single Assignment Form](http://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.3.6773), or my earlier article about [Landin, Steele, letrec, and labels](//wingolog.org/archives/2009/08/10/goto-).

But for a shorter, readable, entertaining piece, see Appel's SSA is Functional Programming. I agree with Appel that we in the lambda-tribe get too hung up on our formalisms, when sometimes the right thing to do is draw a flow-graph.

**so what's the big deal?**

If it were only this, what I've said up to now, then SSA would not be as interesting as CPS, or even ANF. But SSA is not just about binding, it is also about control flow. In order to place your phi functions correctly, you need to build what is called a *dominator tree*. One basic block is said to dominate another if all control paths must pass through the first before reaching the second.

For example, the entry block always dominates the entirety of a function. In our example above, `b0` also dominates every other block. However though `b1` does branch to `exit`, it does not dominate it, as `exit` may be reached on other paths.

It turns out that you need to place phi functions wherever a definition of a variable meets a use of the variable that is not strictly dominated by the definition. In our case, that means we place a phi node on `exit`. The dominator tree is a precise, efficient control-flow analysis that allows us to answer questions like this one (where do I place a phi node?).

For more on SSA and dominators, see the very readable 1991 paper by Cytron, Ferrante, Rosen, Wegman, and Zadeck, [Efficiently Computing Static Single Assignment Form and the Control Dependence Graph](http://www.cs.cmu.edu/afs/cs/academic/class/15745-s07/www/papers/cytron-efficientSSA.pdf).

Typical implementations of SSA embed in each basic block pointers to the predecessors and successors of the blocks, as well as the block's dominators and (sometimes) post-dominators. (A predecessor is a block that precedes the given node in the control-flow graph; a successor succeeds it. A post-dominator is like a dominator, but for the reverse control flow; search the tubes for more.) There are well-known algorithms to calculate these links in linear time, and the SSA community has developed a number of optimizations on top of this cheap flow information.

In contrast, the focus in the lambda tribe has been more on *inter* procedural control flow, which -- as far as I can tell -- no one does in less than O(N2) time, which is, as my grandmother would say, "just turrible".

I started off with a mention of global value numbering (GVN) on purpose. This is still, 20+ years later, *the* big algorithm for code motion in JIT compilers. HotSpot C1 and V8 both use it, and [it just landed in IonMonkey](https://bugzilla.mozilla.org/show_bug.cgi?id=659729). GVN is well-known, well-studied, and it works. It results in loop-invariant code motion: if an invariant definition reaches a loop header, it can be hoisted out of the loop. In contrast I don't know of anything from the lambda tribe that really stacks up. There probably is something, but it's certainly not as well-studied.

**why not ssa?**

Peoples of the machine tribe, could you imagine returning a block as a value? Didn't think so. It doesn't make sense to return a label. But that's exactly what the lambda-calculus is about. One may represent blocks as functions, and use them as such, but one may also pass them as arguments and return them as values. Such blocks are of a higher order than the normal kind of block that is a jump target. Indeed it's the only way to express recursion in the basic lambda calculus.

That's what I mean when I say that CPS is good as a higher-order intermediate language, and when I say that SSA is a good first-order intermediate language.

If you have a fundamentally higher-order language, one in which you need to loop by recursion, then you have two options: do whole-program analysis to aggressively closure-convert your program to be first-order, and then you can use SSA, or use a higher-order IL, and use something more like CPS.

[MLton](http://mlton.org/) is an example of a compiler that does the former. Actually, MLton's [SSA implementation](http://mlton.org/cgi-bin/viewsvn.cgi/mlton/trunk/mlton/ssa/ssa-tree.sig?rev=7061&view=markup) is simply lovely. They do represent blocks as functions with arguments instead of labels and phi functions.

But if you can't do whole-program analysis -- maybe because you want to extend your program at runtime, support separate compilation, or whatever -- then you can't use SSA as a global IL. That's not to say that you shouldn't identify first-order segments of your program and apply SSA-like analysis and optimization on them, of course! That's really where the lambda tribe should go.

**finally**

[![](//wingolog.org/pub/lambda-whut.jpg)](http://www.flickr.com/photos/spencer77/)

I wrote this because I was in the middle of V8's Crankshaft compiler and realized I didn't understand some of the idioms, so I went off to read a bunch of papers. At the same time, I wanted to settle the CPS-versus-ANF question for my Guile work. (Guile currently has a direct-style compiler, for which there are precious few optimizations; this fact is mostly a result of being difficult to work with the IL.)

This post summarizes my findings, but I'm sure I made a mistake somewhere. Please note any corrections in the comments.

## related articles

- [effects analysis in guile](/archives/2014/05/18/effects-analysis-in-guile)
- [a continuation-passing style intermediate language for guile](/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)

### 7 responses

1. [Andreas Zwinkau](http://beza1e1.tuxen.de) says:[13 July 2011 12:09 PM](#bbd6a28fa7ca96ccf1e238bf69347f8cb37bef49)

   I don't see any mistakes, but i only know SSA and no CPS/ANF.

   What is a phi-function? You may see it as a funny way to denote copies, because this is how they are deconstructed/removed. I'd write your example in SSA-form like this:

   function clamp (x0, lower, upper) {

   if (x0 < lower)

   x1 = lower;

   else if (x0 > upper)

   x2 = upper;

   x3 = phi(x1,x2,x0);

   return x3;

   }

   Since copies are noops, this can be simplified:

   function clamp (x, lower, upper) {

   if (x < lower);

   else if (x > upper);

   x2 = phi(lower,upper,x);

   return x2;

   }

   And deconstructed to:

   function clamp (x0, lower, upper) {

   if (x < lower)

   x2 = lower;

   else if (x > upper)

   x2 = upper;

   else

   x2 = x;

   return x2;

   }

   For example, LLVM does this in the backend. Since x2 is assigned twice, this is not in SSA-form anymore.

   Also, note that SSA-form is not a kind or type of IL. It is a property of a representation. Also, there is nothing that about SSA, that you could not do without it. It just encodes analyses like "reaching definition" into the program representation.

   Finally, for a pet peeve of mine, SSA actually makes the concept of variables unecessary. Since any operand has exactly one defining operation, you can represent that just by a reference to this operation. This makes stuff like copy-propagation implicit, since copies are noops. See http://pp.info.uni-karlsruhe.de/publication.php?id=braun11wir

2. [Vladimir Sedach](http://carcaddar.blogspot.com/) says:[13 July 2011 5:50 PM](#0d2461f701f0450e5c842896c6ac3408ee789dae)

   Thanks for the article! This is a really good introduction.

3. Verte says:[14 July 2011 6:43 AM](#5216f91c735048aec41081e29e52e7f6233734c2)

   "In contrast, the focus in the lambda tribe has been more on \*inter\*procedural control flow, which -- as far as I can tell -- no one does in less than O(N^2) time, which is, as my grandmother would say, "just turrible"."

   Does what in less than O(N^2) time?

   If you do certain things lazily, you can get below that for the common case. Whether you can or not depends on what you're actually trying to analyse, though.

4. [Don Stewart](http://donsbot.wordpress.com) says:[14 July 2011 1:04 PM](#89d53e359753e0b900b0cc8eee8b497061052807)

   Note that you can convert between SSA and ANF form. See the 2003 paper, "A Functional Perspective on SSA Optimisation Algorithms",

   Chakravarty, Keller and Zadarnowski, which introduces (and implements) a bi-directional translation between SSA and ANF.

   http://www.jantar.org/papers/chakravarty03perspective.pdf

5. Matthew Swank says:[20 July 2011 2:47 PM](#379339a91011c658b00d7f0df4369206c4ce912b)

   If SSA can't handle functions as values, how does it deal with constructs like computed gotos?

6. [Andy Wingo](http://wingolog.org/) says:[25 July 2011 2:53 PM](#b1ff1a42b247d13d7526ad15e21fe533e62152c2)

   Good point about names, Andreas! I suspect the same thing applies to continuations in CPS.

   Verte, I was referring to the kCFA control flow analysis algorithms, of which only 0CFA is polynomial. Higher-order (k > 0) seems to take exponential time!

   Matthew, from [gccint](http://gcc.gnu.org/onlinedocs/gccint/Edges.html#Edges):

   > computed jumps
   >
   > Computed jumps contain edges to all labels in the function referenced from the code. All those edges have EDGE\_ABNORMAL flag set. The edges used to represent computed jumps often cause compile time performance problems, since functions consisting of many taken labels and many computed jumps may have very dense flow graphs, so these edges need to be handled with special care...

7. John Leuner says:[16 March 2013 7:11 PM](#a748973ac927953fa0b76b103725bf1027a76b8b)

   mlton seems to have moved to github:

   https://github.com/MLton/mlton/blob/master/mlton/ssa/ssa-tree.sig

Comments are closed.
