---
title: A Few Synthesizing Superoptimizer Results
url: https://blog.regehr.org/archives/1252
published: "2015-07-21T21:34:01Z"
feed: regehr
guid: http://blog.regehr.org/?p=1252
---

# A Few Synthesizing Superoptimizer Results

For this post, I crippled Souper by disabling its path conditions and limiting the depth of harvested expressions to two LLVM instructions. The first goal was to create a nice easy burn-in test for Souper’s instruction synthesizer, which uses a variant of [this method](http://www.eecs.berkeley.edu/~jha/pres/pldi11.pdf); the second goal was to see if depth-limited, path-condition-free expressions would result in optimizations that are easier to understand and implement in LLVM.

We harvested LLVM expressions from SPEC CINT 2006, leaving out omnet, which caused a compiler crash. For each of the ~30,000 unique harvested expressions, we ran Souper’s synthesizer, which attempts to create a functionally equivalent but cheaper version of the expression. The results are ranked by the average of the optimization’s rank in terms of static and dynamic profile counts.

**[Here are the results](https://blog.regehr.org/extra_files/souper-jul-15.html)**

One thing you’ll notice is a lot of this sort of thing:

**```** **%0:i8 = var** **%1:i8 = and 1:i8, %0** **%2:i1 = ne 0:i8, %1** **infer %2** **%3:i1 = trunc %0** **result %3**

**```**

The backends don’t usually care which of these they get, but even so this seems like a nice easy simplification to implement at the IR level.

My favorite optimizations from this batch are where Souper uses bit-level tricks to remove select instructions; here’s an example where the optimized version seems to lead to better code on the architectures that I’ve tried:

**```** **%0:i1 = var** **%1:i64 = var** **%2:i64 = select %0, %1, 1:i64** **%3:i1 = eq 0:i64, %2** **infer %3** **%4:i64 = zext %0** **%5:i1 = ult %1, %4** **result %5**

**```**

On the other hand, it is clear that there are many select-removal “optimizations” in this batch that are just not a good idea. We are not finding it easy to automatically choose which of two LLVM expressions are preferable. For now we just say that most instructions cost 1, select and div/rem cost 3, and constants and phis are free. Obviously this is just a first hack. We’d appreciate feedback on this aspect of the work (and others, of course). And obviously let us know if you find bugs in the results.

We might ask what can cause Souper to not find an optimization. First, Souper’s grab-bag of components for synthesis contains one of each of the following: shl, lshr, ashr, and, or, xor, add, sub, slt, ult, sle, ule, eq, ne, and constant. If an optimization requires, for example, two xors, we won’t find it. Of course we could give the synthesizer more components to work with but that slows it down. Sext/zext/trunc aren’t considered basic components by Souper, but are added as necessary to make bitwidths match up. This leads us to the second cause of missed optimizations: Souper won’t make up bitwidths not found in the original code. So, if there’s a trick where the arguments need to be extended from 32 bits to 64 and then truncated later, we won’t find that. Again this isn’t hard to fix, but it does appear to be hard to fix without slowing down synthesis a lot. Finally, if the solver (Z3 here) uses too much time or memory, or if some limits on loops inside the synthesizer are exceeded, we’ll miss out on whatever we might have found if we were willing to use more resources.

I’d like to distill the synthesis problems that come from optimizing LLVM code into some sort of synthesis benchmark that researchers can use to evaluate their work. We would say that your synthesis engine is better than ours (on this benchmark) if it can save more total static or dynamic profile weight in a given amount of time, or if it can produce equivalent results in less time. Of course we’ll have to agree on a cost model. I just attended the [2015 SMT Workshop](http://smt2015.csl.sri.com/) the other day (where I talked about Souper!) and it sounds like there is plenty of interest in the solver community in providing better support for synthesis, which is great!

We have a medium-term goal of taking an optimization discovered by Souper and generalizing it into something that is very close to what an LLVM developer would implement. Since Souper has no capacity for abstraction, we’ll turn its optimizations into [Alive](https://github.com/nunoplopes/alive) patterns and then derive not only any preconditions that need to hold before the optimization can fire but also code for converting constant values on the left hand side into constants on the right hand side. For example let’s say that Souper finds this optimization:

**```** **%1 = lshr i32 %in, 3** **%out = shl nuw i32 %1, 3** **=>** **%out = and i32 4294967288, %in**

**```**

The general form that we’d like to derive is:

just

**```** **%1 = lshr %in, C** **%out = shl %1, C** **=>** **%out = and -(1<<C), %in**

**```**

Can we derive -(1<<C) from scratch? Time will tell. If this works, it’ll be useful for two reasons. First, it’ll effectively deduplicate Souper’s output: many specific optimizations will all boil down to the same general one. Second, the optimization will be easier to implement in LLVM if a human doesn’t have to do the generalization work. Some [work by Nuno Lopes](http://web.ist.utl.pt/nuno.lopes/pubs/psyco-vmcai14.pdf) contains the basic ideas that we’ll use.

Something else I’ve been thinking about is the fact that all compilers have weaknesses of the kind that we point out in this post. The [OPTGEN](http://pp.info.uni-karlsruhe.de/uploads/publikationen/buchwald15cc.pdf) paper has some concrete data for GCC, LLVM, and Intel CC. I’ve heard that the Microsoft C/C++ compiler could stand to be improved in this respect too. CompCert needs a ton of work, the last time I checked. Wouldn’t it be cool if the work we’re doing for Souper and Alive could apply more broadly than just to LLVM? One observation is that if we deal only with integer values, the IR semantics in each of these compilers is probably broadly the same (leaving out some tricky issues like IR-level undefined behaviors). There’s probably no fundamental reason we can’t just reuse integer optimizations across multiple compilers.

Here are a few offers I’d like to extend:

1. If you are an LLVM frontend developer, I’d like to run Souper on some representative code from your frontend. It would be easiest if you harvested optimized LLVM code into Souper yourself (I’ll have to write up instructions) but if not, you can send me some bitcode files — in this case we won’t get dynamic profile information. A few GB would not be too much. The effect will be that we can help the LLVM middle-end optimizers do a better job on code that you care about.

2. If you develop a compiler other than LLVM, even a closed-source one, let’s figure out a way to translate an integer subset of your IR into Souper, so we can look for missed optimizations. My guess is that if you target a subset of Souper that doesn’t have phis or path conditions, this isn’t very difficult. You will basically end up doing the same kind of depth-bounded expression harvesting that I did.

3. If you are doing research on program synthesis, and if you might be interested in working towards some benchmarks about optimizing LLVM code, let me know and let’s figure out how to make that happen.

The next set of results that I post will be similar to these, but they’ll show off Souper’s ability to exploit the results of one or two of LLVM’s dataflow analyses. Finally I want to add that although I’ve been writing up these posts, Souper is a group effort.
