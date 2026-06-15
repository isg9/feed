---
title: How LLVM Optimizes a Function
url: https://blog.regehr.org/archives/1603
published: "2018-09-04T03:51:56Z"
feed: regehr
guid: http://blog.regehr.org/?p=1603
---

# How LLVM Optimizes a Function

An optimizing, ahead-of-time compiler is usually structured as:

1. A frontend that converts source code into an intermediate representation (IR).

2. A target-independent optimization pipeline: a sequence of passes that successively rewrite the IR to eliminate inefficiencies and forms that cannot be readily translated into machine code. Sometimes called the “middle end.”

3. A target-dependent backend that generates assembly code or machine code.

In some compilers the IR format remains fixed throughout the optimization pipeline, in others the format or semantics change. In LLVM the format and semantics are fixed, and consequently it should be possible to run any sequences of passes you want without introducing miscompilations or crashing the compiler.

The sequence of passes in the optimization pipeline is engineered by compiler developers; its goal is to do a pretty good job in a reasonable amount of time. It gets tweaked from time to time, and of course there’s a different set of passes that gets run at each optimization level. A longish-standing topic in compiler research is to use machine learning or something to come up with a better optimization pipeline either in general or else for a specific application domain that is not well-served by the default pipelines.

Some principles for pass design are minimality and orthogonality: each pass should do one thing well, and there should not be much overlap in functionality. In practice, compromises are sometimes made. For example when two passes tend to repeatedly generate work for each other, they may be integrated into a single, larger pass. Also, some IR-level functionality such as constant folding is so ubiquitously useful that it might not make sense as a pass; LLVM, for example, implicitly folds away constant operations as instructions are created.

In this post we’ll look at how some of LLVM’s optimization passes work. I’ll assume you’ve read [this piece about how Clang compiles a function](https://blog.regehr.org/archives/1605) or alternatively that you more or less understand how LLVM IR works. It’s particularly useful to understand the SSA (static single assignment) form: [Wikipedia](https://en.wikipedia.org/wiki/Static_single_assignment_form) will get you started and [this book](http://ssabook.gforge.inria.fr/latest/book.pdf) contains more information than you’re likely to want to know. Also see the [LLVM Language Reference](https://llvm.org/docs/LangRef.html) and the [list of optimization passes](https://llvm.org/docs/Passes.html).

We’re looking at how Clang/LLVM 6.0.1 optimizes this C++:

```
bool is_sorted(int *a, int n) {
  for (int i = 0; i < n - 1; i++)
    if (a[i] > a[i + 1])
      return false;
  return true;
}

```

Keep in mind that the optimization pipeline is a busy place and we’re going to miss out on a lot of fun stuff such as:

- inlining, an easy but super-important optimization, which can’t happen here since we’re looking at just one function

- basically everything that is specific to C++ vs C

- autovectorization, which is defeated by the early loop exit

In the text below I’m going to skip over every pass that doesn’t make any changes to the code. Also, we’re not even looking at the backend and there’s a lot going on there as well. Even so, this is going to be a bit of a slog! (Sorry to use images below, but it seemed like the best way to avoid formatting difficulties, I hate fighting with WP themes. Click on the image to get a larger version. I used [icdiff](https://github.com/jeffkaufman/icdiff).)

[Here’s the IR file emitted by Clang](https://blog.regehr.org/extra_files/is_sorted2.ll.txt) (I manually removed the “optnone” attribute that Clang put in) and this is the command line that we can use to see the effect of each optimization pass:

```
opt -O2 -print-before-all -print-after-all is_sorted2.ll

```

The first pass is “ [simplify the CFG](https://github.com/llvm-mirror/llvm/blob/release_60/lib/Transforms/Scalar/SimplifyCFGPass.cpp)” (control flow graph). Because Clang does not optimize, IR that it emits often contains easy opportunities for cleanup:

[![](../extra_files/optimize-function/00.png)](../extra_files/optimize-function/00.png)

Here, basic block 26 simply jumps to block 27. This kind of block can be eliminated, with jumps to it being forwarded to the destination block. The diff is a bit more confusing that it would have to be due to the implicit block renumbering performed by LLVM. The full set of transformations performed by SimplifyCFG is listed in a comment at the top of the pass:

This file implements dead code elimination and basic block merging, along with a collection of other peephole control flow optimizations. For example:

- Removes basic blocks with no predecessors.

- Merges a basic block into its predecessor if there is only one and the predecessor only has one successor.

- Eliminates PHI nodes for basic blocks with a single predecessor.

- Eliminates a basic block that only contains an unconditional branch.

- Changes invoke instructions to nounwind functions to be calls.

- Change things like “if (x) if (y)” into “if (x&y)”.

Most opportunities for CFG cleanup are a result of other LLVM passes. For example, dead code elimination and loop invariant code motion can easily end up creating empty basic blocks.

The next pass to run, [SROA (scalar replacement of aggregates)](https://github.com/llvm-mirror/llvm/blob/release_60/lib/Transforms/Scalar/SROA.cpp), is one of our heavy hitters. The name ends up being a bit misleading since SROA is only one of its functions. This pass examines each alloca instruction (function-scoped memory allocation) and attempts to promote it into SSA registers. A single alloca will turn into multiple registers when it is statically assigned multiple times and also when the alloca is a class or struct that can be split into its components (this splitting is the “scalar replacement” referred to in the name of the pass). A simple version of SROA would give up on stack variables whose addresses are taken, but LLVM’s version interacts with alias analysis and ends up being fairly smart (though the smarts are not needed in our example).

[![](../extra_files/optimize-function/01.png)](../extra_files/optimize-function/01.png)

After SROA, all alloca instructions (and their corresponding loads and stores) are gone, and the code ends up being much cleaner and more amenable to subsequent optimization (of course, SROA cannot eliminate all allocas in general — this only works when the pointer analysis can completely eliminate aliasing ambiguity). As part of this process, SROA had to insert some phi instructions. Phis are at the core of the SSA representation, and the lack of phis in the code emitted by Clang tells us that Clang emits a trivial kind of SSA where communication between basic blocks is through memory, instead of being through SSA registers.

Next is “ [early common subexpression elimination](https://github.com/llvm-mirror/llvm/blob/release_60/lib/Transforms/Scalar/EarlyCSE.cpp)” (CSE). CSE tries to eliminate the kind of redundant subcomputations that appear both in code written by people and in partially-optimized code. “Early CSE” is a fast, simple kind of CSE that looks for trivially redundant computations.

[![](../extra_files/optimize-function/02.png)](../extra_files/optimize-function/02.png)

Here both %10 and %17 do the same thing, so uses of one of the values can be rewritten as uses of the other, and then the redundant instruction eliminated. This gives a glimpse of the advantages of SSA: since each register is assigned only once, there’s no such thing as multiple versions of a register. Thus, redundant computations can be detected using syntactic equivalence, with no reliance on a deeper program analysis (the same is not true for memory locations, which live outside of the SSA universe).

Next, a few passes that have no effect run, and then “ [global variable optimizer](https://github.com/llvm-mirror/llvm/blob/release_60/lib/Transforms/IPO/GlobalOpt.cpp)” which self-describes as:

This pass transforms simple global variables that never have their address taken. If obviously true, it marks read/write globals as constant, deletes variables only stored to, etc.

It makes this change:

[![](../extra_files/optimize-function/03.png)](../extra_files/optimize-function/03.png)

The thing that has been added is a function attribute: metadata used by one part of the compiler to store a fact that might be useful to a different part of the compiler. You can [read about the rationale for this attribute here](https://reviews.llvm.org/D20348).

Unlike other optimizations we’ve looked at, the global variable optimizer is *interprocedural*, it looks at an entire LLVM module. A module is more or less equivalent to a compilation unit in C or C++. In contrast, *intraprocedural* optimizations look at only one function at a time.

The next pass is the “ [instruction combiner](https://github.com/llvm-mirror/llvm/tree/release_60/lib/Transforms/InstCombine)“: InstCombine. It is a large, diverse collection of “peephole optimizations” that (typically) rewrite a handful of instructions connected by data flow into a more efficient form. InstCombine won’t change the control flow of a function. In this example it doesn’t have a lot to do:

[![](../extra_files/optimize-function/04.png)](../extra_files/optimize-function/04.png)

Here instead of subtracting 1 from %1 to compute %4, we’ve decided to add -1. This is a canonicalization rather than an optimization. When there are multiple ways of expressing a computation, LLVM attempts to canonicalize to a (often arbitrarily chosen) form, which is then what LLVM passes and backends can expect to see. The second change made by InstCombine is canonicalizing two sign-extend operations (sext instructions) that compute %7 and %11 to zero-extends (zext). This is a safe transformation when the compiler can prove that the operand of the sext is non-negative. That is the case here because the loop induction variable starts at zero and stops before it reaches n (if n is negative, the loop never executes at all). The final change is adding the “nuw” (no unsigned wrap) flag to the instruction that produces %10. We can see that this is safe by observing that (1) the induction variable is always increasing and (2) that if a variable starts at zero and increases, it would become undefined by passing the sign wraparound boundary next to INT\_MAX before it reaches the unsigned wraparound boundary next to UINT\_MAX. This flag could be used to justify subsequent optimizations.

Next, SimplifyCFG runs for a second time, removing two empty basic blocks:

[![](../extra_files/optimize-function/05.png)](../extra_files/optimize-function/05.png)

“ [Deduce function attributes](https://github.com/llvm-mirror/llvm/blob/release_60/lib/Transforms/IPO/FunctionAttrs.cpp)” annotates the function further:

[![](../extra_files/optimize-function/06.png)](../extra_files/optimize-function/06.png)

“Norecurse” means that the function is not involved in any recursive loop and a “readonly” function does not mutate global state. A “nocapture” parameter is not saved anywhere after its function returns and a “readonly” parameter refers to storage not modified by the function. Also see the [full set of function attributes](https://releases.llvm.org/6.0.0/docs/LangRef.html#function-attributes) and the [full set of parameter attributes](https://releases.llvm.org/6.0.0/docs/LangRef.html#parameter-attributes).

Next, “ [rotate loops](https://github.com/llvm-mirror/llvm/blob/release_60/lib/Transforms/Scalar/LoopRotation.cpp)” moves code around in an attempt to improve conditions for subsequent optimizations:

[![](../extra_files/optimize-function/07_alt.png)](../extra_files/optimize-function/07_alt.png)

Although this diff looks a bit alarming, there’s not all that much happening. We can see what happened more readily by asking LLVM to draw the control flow graph before and after loop rotation. Here are the before (left) and after (right) views:

[![](../extra_files/optimize-function/cfgs1.png)](../extra_files/optimize-function/cfgs1.png)

The original code still matches the loop structure emitted by Clang:

```
  initializer
  goto COND
COND:
  if (condition)
    goto BODY
  else
    goto EXIT
BODY:
  body
  modifier
  goto COND
EXIT:

```

Whereas the rotated loop looks like this:

```
  initializer
  if (condition)
    goto BODY
  else
    goto EXIT
BODY:
  body
  modifier
  if (condition)
    goto BODY
  else
    goto EXIT
EXIT:

```

(Corrected as suggested by Johannes Doerfert below– thanks!)

The point of loop rotation is to remove one branch and to enable subsequent optimizations. I did not find a better description of this transformation on the web ( [the paper that appears to have introduced the term](https://digitalcommons.ohsu.edu/csetech/191/) doesn’t seem all that relevant).

CFG simplification folds away the two basic blocks that contain only degenerate (single-input) phi nodes:

[![](../extra_files/optimize-function/08.png)](../extra_files/optimize-function/08.png)

The instruction combiner rewrites “%4 = 0 s< (%1 - 1)" as "%4 = %1 s> 1″, this kind of thing is useful because it reduces the length of a dependency chain and may also create dead instructions (see [the patch making this happen](https://reviews.llvm.org/D29774)). This pass also eliminates another trivial phi node that was added during loop rotation:

[![](../extra_files/optimize-function/09.png)](../extra_files/optimize-function/09.png)

Next, “ [canonicalize natural loops](https://github.com/llvm-mirror/llvm/blob/release_60/lib/Transforms/Utils/LoopSimplify.cpp)” runs, it self-describes as:

This pass performs several transformations to transform natural loops into a simpler form, which makes subsequent analyses and transformations simpler and more effective.

Loop pre-header insertion guarantees that there is a single, non-critical entry edge from outside of the loop to the loop header. This simplifies a number of analyses and transformations, such as LICM.

Loop exit-block insertion guarantees that all exit blocks from the loop (blocks which are outside of the loop that have predecessors inside of the loop) only have predecessors from inside of the loop (and are thus dominated by the loop header). This simplifies transformations such as store-sinking that are built into LICM.

This pass also guarantees that loops will have exactly one backedge.

Indirectbr instructions introduce several complications. If the loop contains or is entered by an indirectbr instruction, it may not be possible to transform the loop and make these guarantees. Client code should check that these conditions are true before relying on them.

Note that the simplifycfg pass will clean up blocks which are split out but end up being unnecessary, so usage of this pass should not pessimize generated code.

This pass obviously modifies the CFG, but updates loop information and dominator information.

Here we can see loop exit-block insertion happen:

[![](../extra_files/optimize-function/10.png)](../extra_files/optimize-function/10.png)

Next up is “ [induction variable simplification](https://github.com/llvm-mirror/llvm/blob/release_60/lib/Transforms/Scalar/IndVarSimplify.cpp)“:

This transformation analyzes and transforms the induction variables (and computations derived from them) into simpler forms suitable for subsequent analysis and transformation.

If the trip count of a loop is computable, this pass also makes the following changes:

1. The exit condition for the loop is canonicalized to compare the induction value against the exit value. This turns loops like: ‘for (i = 7; i\*i < 1000; ++i)' into 'for (i = 0; i != 25; ++i)'

2. Any use outside of the loop of an expression derived from the indvar is changed to compute the derived value outside of the loop, eliminating the dependence on the exit value of the induction variable. If the only purpose of the loop is to compute the exit value of some derived expression, this transformation will make the loop dead.

The effect of this pass here is simply to rewrite the 32-bit induction variable as 64 bits:

[![](../extra_files/optimize-function/11.png)](../extra_files/optimize-function/11.png)

I don’t know why the zext — previously canonicalized from a sext — is turned back into a sext.

Now “ [global value numbering](https://github.com/llvm-mirror/llvm/blob/release_60/lib/Transforms/Scalar/GVN.cpp)” performs a very clever optimization. Showing this one off is basically the entire reason that I decided to write this blog post. See if you can figure it out just by reading the diff:

[![](../extra_files/optimize-function/12.png)](../extra_files/optimize-function/12.png)

Got it? Ok, the two loads in the loop on the left correspond to a\[i\] and a\[i + 1\]. Here GVN has figured out that loading a\[i\] is unnecessary because a\[i + 1\] from one loop iteration can be forwarded to the next iteration as a\[i\]. This one simple trick halves the number of loads issued by this function. Both LLVM and [GCC](https://gcc.godbolt.org/z/KpHmbm) only got this transformation recently.

You might be asking yourself if this trick still works if we compare a\[i\] against a\[i + 2\]. It turns out that LLVM is unwilling or unable to do this, but GCC is willing to throw [up to four registers at this problem](https://gcc.godbolt.org/z/TrKpFa).

Now “ [bit-tracking dead code elimination](https://github.com/llvm-mirror/llvm/blob/release_60/lib/Transforms/Scalar/BDCE.cpp)” runs:

This file implements the Bit-Tracking Dead Code Elimination pass. Some instructions (shifts, some ands, ors, etc.) kill some of their input bits. We track these dead bits and remove instructions that compute only these dead bits.

But it turns out the extra cleverness isn’t needed since the only dead code is a GEP, and it is trivially dead (GVN removed the load that previously used the address that it computes):

[![](../extra_files/optimize-function/13.png)](../extra_files/optimize-function/13.png)

Now the instruction combiner sinks an add into a different basic block. The rationale behind putting this transformation into InstCombine is not clear to me; perhaps there just wasn’t a more obvious place to do this one:

[![](../extra_files/optimize-function/14.png)](../extra_files/optimize-function/14.png)

Now things get a tiny bit weird, “ [jump threading](https://github.com/llvm-mirror/llvm/blob/release_60/lib/Transforms/Scalar/JumpThreading.cpp)” undoes what “canonicalize natural loops” did earlier:

[![](../extra_files/optimize-function/15.png)](../extra_files/optimize-function/15.png)

Then we canonicalize it back:

[![](../extra_files/optimize-function/16.png)](../extra_files/optimize-function/16.png)

And CFG simplification flips it the other way:

[![](../extra_files/optimize-function/17.png)](../extra_files/optimize-function/17.png)

And back:

[![](../extra_files/optimize-function/18.png)](../extra_files/optimize-function/18.png)

And forth:

[![](../extra_files/optimize-function/19.png)](../extra_files/optimize-function/19.png)

And back:

[![](../extra_files/optimize-function/20.png)](../extra_files/optimize-function/20.png)

And forth:

[![](../extra_files/optimize-function/21.png)](../extra_files/optimize-function/21.png)

Whew, we’re finally done with the middle end! The code at the right is what gets passed to (in this case) the x86-64 backend.

You might be wondering if the oscillating behavior towards the end of the pass pipeline is the result of a compiler bug, but keep in mind that this function is really, really simple and there were a whole bunch of passes mixed in with the flipping and flopping, but I didn’t mention them because they didn’t make any changes to the code. As far as the second half of the middle end optimization pipeline goes, we’re basically looking at a degenerate execution here.

**Acknowledgments:** Some students in my Advanced Compilers class this fall provided feedback on a draft of this post (I also used this material as an assignment). I ran across the function used here in [this excellent set of lecture notes about loop optimizations](https://www.cs.cmu.edu/~fp/courses/15411-f13/lectures/17-loopopt.pdf).
