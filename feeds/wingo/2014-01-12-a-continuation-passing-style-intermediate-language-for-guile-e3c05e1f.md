---
title: a continuation-passing style intermediate language for guile — wingolog
url: https://wingolog.org/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile
published: "2014-01-12T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile
---

# a continuation-passing style intermediate language for guile — wingolog

## [a continuation-passing style intermediate language for guile](/archives/2014/01/12/a-continuation-passing-style-intermediate-language-for-guile)

12 January 2014 9:58 PM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [scope](/tags/scope)
- [cps](/tags/cps)
- [ssa](/tags/ssa)
- [anf](/tags/anf)
- [compilers](/tags/compilers)
- [igalia](/tags/igalia)
- [gnu](/tags/gnu)

Happy new year's, hackfolk!

A few weeks ago I wrote about the upcoming Guile 2.2 release, and specifically about its [new register virtual machine](//wingolog.org/archives/2013/11/26/a-register-vm-for-guile). Today I'd like to burn some electrons on another new part in Guile 2.2, its intermediate language.

To recap, we switched from a stack machine to a register machine because, among other reasons, register machines can consume and produce named intermediate results in fewer instructions than stack machines, and that makes things faster.

To take full advantage of this new capability, it is appropriate to switch at the same time from the direct-style intermediate language (IL) that we had to an IL that names all intermediate values. This lets us effectively reason about each subexpression that goes into a computation, for example in common subexpression elimination.

As far as intermediate languages go, basically there are two choices these days: something SSA-based, or something CPS-based. I wrote an [article on SSA, ANF and CPS](//wingolog.org/archives/2011/07/12/static-single-assignment-for-functional-programmers) a few years ago; if you aren't familiar with these or are feeling a little rusty, I suggest you go and take a look.

In Guile I chose a continuation-passing style language. I still don't know if I made the right choice. I'll go ahead and describe Guile's IL and then follow up with some reflections. The description below is abbreviated from a more complete version in Guile's manual.

**guile's cps language**

Guile's CPS language is composed of terms, expressions, and continuations. It was heavily inspired by Andrew Kennedy's [Compiling with Continuations, Continued](http://research.microsoft.com/~akenn/sml/CompilingWithContinuationsContinued.pdf) paper.

A term can either evaluate an expression and pass the resulting values to some continuation, or it can declare local continuations and contain a sub-term in the scope of those continuations.

`$continue` k src exp Evaluate the expression exp and pass the resulting values (if any) to the continuation labelled k. `$letk` conts body Bind conts in the scope of the sub-term body. The continuations are mutually recursive.

Additionally, the early stages of CPS allow for a set of mutually recursive functions to be declared as a term via a `$letrec` term. A later pass will attempt to transform the functions declared in a `$letrec` into local continuations. Any remaining functions are later lowered to `$fun` expressions. More on "contification" later.

Here is an inventory of the kinds of expressions in Guile's CPS language. Recall that all expressions are wrapped in a `$continue` term which specifies their continuation.

`$void` Continue with the unspecified value. `$const` val Continue with the constant value val. `$prim` name Continue with the procedure that implements the primitive operation named by name. `$fun` src meta free body Continue with a procedure. body is the `$kentry` `$cont` of the procedure entry. free is a list of free variables accessed by the procedure. Early CPS uses an empty list for free; only after closure conversion is it correctly populated. `$call` proc args Call proc with the arguments args, and pass all values to the continuation. proc and the elements of the args list should all be variable names. The continuation identified by the term's k should be a `$kreceive` or a `$ktail` instance. `$primcall` name args Perform the primitive operation identified by `name`, a well-known symbol, passing it the arguments args, and pass all resulting values to the continuation. `$values` args Pass the values named by the list args to the continuation. `$prompt` escape? tag handler Push a prompt on the stack identified by the variable name tag and continue with zero values. If the body aborts to this prompt, control will proceed at the continuation labelled handler, which should be a `$kreceive` continuation. Prompts are later popped by `pop-prompt` primcalls.

The remaining element of the CPS language in Guile is the continuation. In CPS, all continuations have unique labels. Since this aspect is common to all continuation types, all continuations are contained in a `$cont` instance:

`$cont` k cont Declare a continuation labelled k. All references to the continuation will use this label.

The most common kind of continuation binds some number of values, and then evaluates a sub-term. `$kargs` is this kind of simple `lambda`.

`$kargs` names syms body Bind the incoming values to the variables syms, with original names names, and then evaluate the sub-term body.

Variables (e.g., the syms of a `$kargs`) should be globally unique. To bind the result of an expression a variable and then use that variable, you would continue from the expression to a `$kargs` that declares one variable. The bound value would then be available for use within the body of the `$kargs`.

`$kif` kt kf Receive one value. If it is a true value, branch to the continuation labelled kt, passing no values; otherwise, branch to kf.

Non-tail function calls should continue to a `$kreceive` continuation in order to adapt the returned values to their uses in the calling function, if any.

`$kreceive` arity k Receive values from a function return. Parse them according to arity, and then proceed with the parsed values to the `$kargs` continuation labelled k.

`$arity` is a helper data structure used by `$kreceive` and also by `$kclause`, described below.

`$arity` req opt rest kw allow-other-keys? A data type declaring an arity. See Guile's manual for details.

Additionally, there are three specific kinds of continuations that can only be declared at function entries.

`$kentry` self tail clauses Declare a function entry. self is a variable bound to the procedure being called, and which may be used for self-references. tail declares the `$cont` wrapping the `$ktail` for this function, corresponding to the function's tail continuation. clauses is a list of `$kclause` `$cont` instances. `$ktail` A tail continuation. `$kclause` arity cont A clause of a function with a given arity. Applications of a function with a compatible set of actual arguments will continue to cont, a `$kargs` `$cont` instance representing the clause body.

**reflections**

Before starting Guile's compiler rewrite, I had no real-world experience with CPS-based systems. I had worked with a few SSA-based systems, and a few more direct-style systems. I had most experience with the previous direct-style system that Guile had, but never had to seriously design another kind of IL, so basically I was ignorant. It shows, I think; but time will tell if it came out OK anyway. At this point I am cautiously optimistic.

As far as fitness for purpose goes, the CPS IL works in the sense that it is part of a self-hosting compiler. I'll say no more on that point other than to mention that it has explicit support for a number of Guile semantic features: multiple-value returns; optional, rest, and keyword arguments; cheap delimited continuations; Guile-native constant literals.

Why not ANF instead? If you recall from my [SSA and CPS article](//wingolog.org/archives/2011/07/12/static-single-assignment-for-functional-programmers), I mentioned that ANF is basically CPS with fewer labels. It tries to eliminate "administrative" continuations, whereas Guile's CPS labels everything. There is no short-hand `let` form.

ANF proponents tout its parsimony as a strength, but I do not understand this argument. I like having labels for everything. In CPS, I have as many labels as there are expressions, plus a few for continuations that don't contain terms. I use them directly in the bytecode compiler; the compiler never has to generate a fresh label, as they are part of the CPS itself.

More importantly, labelling every control-flow point allows me to reason precisely about control flow. For example, if a function is always called with the same continuation, it can be incorporated in the flow graph of the calling function. This is called "contification". It is not the same thing as inlining, as it works for sets of recursive functions as well, and never increases code size. This is a crucial transformation for a Scheme compiler to make, as it turns function calls into `goto` s, and self-function calls into loop back-edges.

Guile's previous compiler did a weak form of contification, but because we didn't have names for all control points it was gnarly and I was afraid to make it any better. Now its contifier is optimal. See Fluet and Weeks' [Contification using Dominators](http://www.cs.rit.edu/~mtf/research/contification/) and Kennedy's [CWCC](http://research.microsoft.com/~akenn/sml/CompilingWithContinuationsContinued.pdf), for more on contification.

One more point in favor of labelling all continuations. Many tranformations can be best cast as a two-phase process, in which you first compute a set of transformations to perform, and then you apply them. Dead-code elimination works this way; first you find the [least fixed-point of live expressions](http://www.pvk.ca/Blog/2012/02/19/fixed-points-and-strike-mandates/), and then you residualize only those expressions. Without names, how are you going to refer to an expression in the first phase? The ubiquitous, thorough labelling that CPS provides does not have this problem.

So I am happy with CPS, relative to ANF. But what about SSA? In my previous article, I asked SSA proponents to imagine returning a block from a block. Of course it doesn't make any sense; SSA is a first-order language. But modern CPS is also first-order, is the thing! Modern CPS distinguishes "continuations" syntactically from functions, which is exactly the same as SSA's distinction between basic blocks and functions. [CPS and SSA really are the same](http://www.cs.princeton.edu/~appel/papers/ssafun.ps) on this level.

The fundamental CPS versus SSA difference is, as [Stephen Weeks noted a decade ago](http://mlton.org/pipermail/mlton/2003-January/023054.html), one of data structures: do you group your expressions into basic blocks stored in a vector (SSA), or do you nest them into a scope tree (CPS)? It's not clear that I made the correct choice.

In practice with Guile's CPS you end up building graphs on the side that describe some aspect of your term. For example you can build a reverse-post-ordered control flow analysis that linearizes your continuations, and assigns them numbers. Then you can compute a bitvector for each continuation representing each one's reachable continuations. Then you can use this reachability analysis to determine the extent of a prompt's body, for example.

But this analysis is all on the side and not really facilitated by the structure of CPS itself; the CPS facilities that it uses are the globally unique continuation and value names of the CPS, and the control-flow links. Once the transformation is made, all of the analysis is thrown away.

Although this seems wasteful, the SSA approach of values having "implicit" names by their positions in a vector (instead of explicit ephemeral name-to-index mappings) is terrifying to me. Any graph transformation could renumber things, or leave holes, or cause vectors to expand... dunno. Perhaps I am too shy of the mutation foot-gun. I find comfort in CPS's minimalism.

One surprise I have found is that I haven't needed to do any dominator-based analysis in any of the paltry CPS optimizations I have made so far. I expect to do so once I start optimizing loops; here we see the cultural difference with SSA I guess, loops being the dominant object of study there. On the other hand I have had to solve flow equations on a few occasions, which was somewhat surprising, though enjoyable.

The optimizations I have currently implemented for CPS are fairly basic. Contification was tricky. One thing I did recently was to make all non-tail `$call` nodes require `$kreceive` continuations; if, as in the common case, extra values were unused, that was reflected in an unused rest argument. This required a number of optimizations to clean up and remove the extra rest arguments for other kinds of source expressions: dead-code elimination, the typical beta/eta reduction, and some code generation changes. It was worth it though, and now with the optimization passes things are faster than they were before.

Well, I find that I am rambling now. I know this is a lot of detail, but I hope that it helps some future compiler hacker understand more about intermediate language tradeoffs. I have been happy with CPS, but I'll report back if anything changes :) And if you are actually hacking on Guile, check the in-progress [manual](http://www.gnu.org/software/guile/docs/master/guile.html/Continuation_002dPassing-Style.html) for all the specifics.

Happy hacking to all, and to all a good hack!

## related articles

- [static single assignment for functional programmers](/archives/2011/07/12/static-single-assignment-for-functional-programmers)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [revisiting common subexpression elimination in guile](/archives/2014/08/25/revisiting-common-subexpression-elimination-in-guile)
- [effects analysis in guile](/archives/2014/05/18/effects-analysis-in-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)

### 6 responses

1. Brandon says:[14 January 2014 11:58 PM](#a973d2e6448edf7e501244482dea3f1e8b4514cd)

   Hi, Andy. Living the dream, I see. Keep hacking on that for which you are truly passionate :)

   --Brandon

2. Jeremy Lakeman says:[16 January 2014 3:48 AM](#721a56c861fca42fa4d49ee5e71886783b41d453)

   In LLVM's SSA form, basic blocks may have a vector of instructions to specify their execution order, but instruction arguments are direct pointers to the corresponding in memory C++ Value\* (ie Instruction, Constant, ...). So manipulating the location of instructions in basic blocks does not implicitly change parameter references. All references must be explicitly modified, though there are of course helper functions for making bulk changes. So you can rest easy on that point of concern at least.

   As the LLVM toolkit has progressed, more target specific optimisations are being written that require in depth knowledge of the specifics of the machine. Some of these target specific transformations may then reveal other more generic optimisation opportunities. eg this target might always calculate the remainder and modulus with a single instruction.

   I've been wondering if there is a need for a more flexible IL framework. Something that could represent everything from lambda expressions and high level C++ templates before they've been specialised, through to target specific machine code operations with explicit register allocations.

   So you might start with a frontend api for building a pure CPS representation from your AST. That perhaps evolves into something closer to an SSA form that uses only the types and abstract operations that are legal for this target, before machine code instructions are explicitly specified and assembled into code sections. Getting closer to the exact semantics of the target machine through each stage of the process, while being able to seamlessly mix partially transformed representations together without losing potentially useful metadata along the way.

   Such a solution might even allow a clean way to specify hand optimised assembly in the middle of some other high level language constructs.

   But then I've only used LLVM as a front end developer, I'm obviously glossing over the complexity of the problem.

3. [saul](http://pasture.kerosenecow.net/) says:[20 January 2014 1:46 PM](#e3ef482af6fc5acd76501e0f84a1de5c22143fee)

   When I was in college (back in the 80s), one of the assignments was to build a microprocessor out of TTL chips. Since creating a return stack was expensive and complicated, I implemented my subroutine calls by storing the return address in the first word at the subroutine's address and starting instruction execution with the word after that. Returning to the calling code was accomplished at the end of the subroutine by performing an indirect jump to the first word of the subroutine.

   In a way, this was sort of a poor man's CPS. Of course, subroutines couldn't be called recursively\* because of the single, fixed location for storing the return address, but the "continuation" was passed to the subroutine, a goto was used to enter the subroutine, and an indirect goto returned from the subroutine. Substitute an allocated object for the fixed storage location, et voila! I'd effectively implemented a CPS processor decades before I'd ever heard of the term.

   \\* The professor thought about penalizing me for not supporting recursion, but then recognized that wasn't in the requirements he'd specified.

4. nickik says:[26 January 2014 2:19 AM](#9fc520cd476b46c60a793798c45c490be9e4255d)

   Hi. So am I understanding this right. At the moment you first to extensive AOT Compilation then you spit out a bytecode. Then you run that bytecode on a interpreter but your goal is to eventually producde native code directly from AOT?

   I am very intrested in why you are doing this, instead of a interpreter with jit or jit without a interpreter?

5. [John Cowan](http://www.ccil.org/~cowan) says:[20 August 2014 7:58 PM](#c31bd9c861061cb889a091c11923eea421482bf5)

   Saul, that's exactly how the DEC PDP-8 and its predecessors worked: the JMS (JuMp to Subroutine) operation was the normal way of calling subroutines. If a subroutine needed to be recursive, it had its own stack and pushed the return address on the stack before recursing.

   By doing indirect fetches through the return address, you could also pick up arguments (or their addresses, depending on the calling convention) that directly followed the JMS in the instruction stream. That replaced the other main use of a stack.

6. [Samuel A. Falvo II](http://sam-falvo.github.io/kestrel) says:[17 November 2014 5:15 AM](#f0e262c34ee64e2bfeb9db002d25f9f3fa38249c)

   Saul, your professor must be very disappointed with practically every RISC architecture out there then, for all RISCs store return addresses in a "link register" (either dedicated or in the general purpose registers). None use a stack. :-)

   Recursion can be supported, of course, by just stashing the 1st word of the subroutine onto a software stack. I'm pretty convinced that early processors which did this, like PDP-8, explains the existance of "let" and "letrec" in languages like Scheme, Lisp, and ML. (Let compiled more efficient code, since it didn't have to save the return address explicitly.)

Comments are closed.
