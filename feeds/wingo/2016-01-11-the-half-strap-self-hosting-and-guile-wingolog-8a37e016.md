---
title: 'the half strap: self-hosting and guile — wingolog'
url: https://wingolog.org/archives/2016/01/11/the-half-strap-self-hosting-and-guile
published: "2016-01-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2016/01/11/the-half-strap-self-hosting-and-guile
---

# the half strap: self-hosting and guile — wingolog

## [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)

11 January 2016 9:51 PM

- [compilers](/tags/compilers)
- [gnu](/tags/gnu)
- [igalia](/tags/igalia)
- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [compilers](/tags/compilers)
- [self-hosting](/tags/self-hosting)
- [profiling](/tags/profiling)
- [interpreters](/tags/interpreters)

**or, "why does building guile take so friggin long"**

Happy new year's, hackfolk! I don't know about y'all, but I'm feeling pretty good about 2016. Let's make some cool stuff!

Today's article is about Guile and how it builds itself. It's a Scheme implementation mostly written in Scheme, so how it would go about doing that isn't straightforward. And although the performance of Guile is pretty great these days, a user's first experience with it will probably be building it, which is a process that takes approximately forever. Seriously. On this newish laptop with an i7-5600U CPU and four cores it takes like 45 minutes. On older machines it can take even longer. What gives?

Well, fictional reader, it's a good question. I'm glad you asked! Before getting to the heart of the matter, I summarize a bit of background information.

**and then nothing turned itself inside out**

Guile is mostly written in Scheme. Some parts of it are written in C -- some runtime routines, some supporting libraries (the garbage collector, unicode support, arbitrary precision arithmetic), and the bytecode interpreter. The first phase when building Guile is to take the system's C compiler -- a program that takes C source code and produces native machine code -- and use it to build `libguile`, the part of Guile that is written in C.

The next phase is to compile the parts of Guile written in Scheme. Currently we compile to bytecode which is then interpreted by `libguile`, but this discussion would be the same if we compiled Scheme to native code instead of bytecode.

There's a wrinkle, though: the Scheme compiler -- the program that takes a Scheme program and produces bytecode -- is written in Scheme. When we built `libguile`, we could use the system's C compiler. But the system has no Scheme compiler, so how do we do?

The answer is that in addition to a Scheme compiler, Guile also includes a Scheme interpreter. We use the interpreter to load the Scheme compiler, and then use the compiler to produce bytecode from Scheme.

There's another wrinkle, though, and I bet you can guess what it is :) The Scheme interpreter is also written in Scheme. It used to be that Guile's Scheme interpreter was written in C, but that made it impossible to tail-call between compiled and interpreted code. So some six years ago, I [rewrote the interpreter in Scheme](https://wingolog.org/archives/2009/12/09/in-which-our-protagonist-forgoes-modesty).

As I mention in that article, Guile actually has *two* Scheme interpreters: the one in Scheme and one in C that is only used to compile the one in Scheme, and never used again. The bootstrap interpreter written in C avoids the problem with tail calls to compiled code because when it runs, there is no compiled code.

So in summary, Guile's build has the following general phases:

1. The system C compiler builds `libguile`.

2. The bootstrap C interpreter in `libguile` loads the Scheme compiler and builds `eval.go` from `eval.scm`. (Currently `.go` is the extension for compiled Guile code. The extension predates the Go language. Probably we switch to `.so` at some point, though.)

3. The Scheme interpreter from `eval.go` loads the Scheme compiler and compiles the rest of the Scheme code in Guile, including the Scheme compiler itself.

In the last step, Guile compiles each file in its own process, allowing for good parallelization. This also means that as the compiler builds, the compiler itself starts running faster because it can use the freshly built `.go` files instead having to use the interpreter to load the source `.scm` files.

**so what's slow?**

Building `libguile` is not so slow; it takes about a minute on my laptop. Could be faster, but it's fine.

Building `eval.go` is slow, but at two and half minutes it's bearable.

Building the rest of the Scheme code is horribly slow though, and for me takes around 40 or 50 minutes. What is going on?

The crucial difference between building `libguile` and building the `.go` files is that when we build `libguile`, we use the C compiler, which is itself a highly optimized program. When we build `.go` files, we use the Scheme compiler, which hasn't yet been compiled! Indeed if you rebuild all the Scheme code using a *compiled* Scheme compiler instead of an *interpreted* Scheme compiler, you can rebuild all of Guile in about 5 minutes. (Due to the way the Makefile dependencies work, the easiest way to do this if you have a built Guile is `rm bootstrap/ice-9/eval.go && make -jN`.)

The story is a bit complicated by parallelism, though. Usually if you do a `make -j4`, you will be able to build 4 things at the same time, taking advantage of 4 cores (if you have them). However Guile's Makefile rules are arranged in such a way that the initial `eval.go` compile is done serially, when nothing else is running. This is because the bootstrap interpreter written in C uses C stack space as temporary storage. It could be that when compiling bigger files, the C interpreter might run out of stack, and with C it's hard to detect exactly how much stack you have. Indeed, sometimes we get reports of strange bootstrap failures that end up being because Guile was built with `-O0` and the compiler decided to use much more stack space than we usually see. We try to fix these, usually by raising the static stack limits that Guile's C interpreter imposes, but we certainly don't want a limitation in the bootstrap interpreter to affect the internal structure of the rest of Guile. The

bootstrap interpreter's only job is to load the compiler and build `eval.go`, and isn't tested in any other way.

So `eval.go` is build serially. After that, compilation can proceed in parallel, but goes more slowly before speeding up. To explain that, I digress!

**a digression on interpreters**

When Scheme code is loaded into Guile from source, the process goes like this:

1. Scheme code is loaded from disk or wherever as a stream of bytes.

2. The *reader* parses that byte stream into S-expressions.

3. The *expander* runs on the S-expressions, expanding macros and lowering Scheme code to an internal language called "Tree-IL".

Up to here, the pipeline is shared between the interpreter and the compiler. If you're compiling, Guile will take the Tree-IL, run the [partial evaluator](https://wingolog.org/archives/2011/10/11/partial-evaluation-in-guile) on it, lower to CPS, optimize that CPS, and then emit bytecode. The next time you load this file, Guile will just `mmap` in the `.go` file and skip all of the other steps. Compilation is great!

But if you are interpreting, a few more things happen:

1. The *memoizer* does some analysis on the Tree-IL and turns variable references into two-dimensional (depth, offset) references on a chained environment. See [the story time article](https://wingolog.org/archives/2013/11/02/scheme-quiz-time) for more; scroll down about halfway for the details. The goal is to do some light compilation on variable access so that the interpreter will have to do less work, and also prevent closures from hanging on to too much data; this is the "flat closure" optimization, for the interpreter.

2. The interpreter "compiles" the code to a chain of closures. This is like the classic direct-threading optimization, but for a tree-based interpreter.

The closure-chaining strategy of the interpreter is almost exactly as in described in [SICP's `analyze` pass](https://mitpress.mit.edu/sicp/full-text/book/book-Z-H-26.html#%_sec_4.1.7). I came up with it independently, but so did Jonathan Rees in 1982 and Marc Feeley in 1986, so I wasn't surprised when I found the prior work!

Back in 2009 when we switched to the `eval`-in-Scheme, we knew that it would result in a slower interpreter. This is because instead of the interpreter being compiled to native code, it was compiled to bytecode. Also, Guile's Scheme compiler wasn't as good then, so we knew that we were leaving optimizations on the floor. Still, the switch to an evaluator in Scheme enabled integration of the compiler, and we thought that the interpreter speed would improve with time. I just took a look and with this silly loop:

```
(let lp ((n 0)) (if (< n #e1e7) (lp (1+ n))))

```

Guile 1.8's interpreter written in C manages to run this in 1.1 seconds. Guile 2.0's interpreter written in Scheme and compiled to the old virtual machine does it in 16.4 seconds. Guile 2.1.1's interpreter, with the closure-chaining optimization, a couple of peephole optimizations in the interpreter, and compiled using the better compiler and VM from Guile 2.2, manages to finish in 2.4 seconds. So we are definitely getting better, and by the time we compile `eval.scm` to native code I have no doubt that we will be as good as the old C implementation. (Of course, when compiled to Guile 2.2's VM, the loop finishes in 55 *milli* seconds, but comparing a compiler and an interpreter is no fair.)

The up-shot for bootstrap times is that once the interpreter is compiled, the build currently runs a little slower, because the compiled `eval.go` interpreter is a bit slower than the bootstrap interpreter in `libguile`.

**bottom up, top down**

Well. Clearly I wanted to share a thing with you about interpreters; thank you for following along :) The salient point is that Guile's interpreter is now pretty OK, though of course not as good as the compiler. Still, Guile 2.0 builds in 12 minutes, while Guile 2.2 builds in 40 or 50, and Guile 2.2 has a faster interpreter. What's the deal?

There are a few factors at play but I think the biggest is that Guile 2.2's compiler is simply much more sophisticated than Guile 2.0's compiler. Just loading it up at bootstrap-time takes longer than loading Guile 2.0's compiler, because there's more code using more macro abstractions than in Guile 2.0. The expander has to do more work, and the evaluator has to do more work. A compiler is a program that runs on programs, and interpreting a bigger program is going to be slower than interpreting a smaller program.

It's a somewhat paradoxical result: to make programs run faster, we needed a better compiler, but that better compiler is bigger, and so it bootstraps from source more slowly. Some of the improvements to generated code quality were driven by a desire to have the compiler run faster, but this only had the reverse effect on bootstrap time.

Unfortunately, Guile 2.2's compiler also runs slow when it's fully compiled: compiling one largeish module in Guile 2.2 compared to 2.0 takes 10.7 seconds instead of 1.9. (To reproduce, `,time (compile-file "module/ice-9/psyntax-pp.scm")` from a Guile 2.0 or 2.2 REPL.) How can we explain this?

Understanding this question has taken me some time. If you do a normal profile of the code using [statprof](https://www.gnu.org/software/guile/docs/master/guile.html/Statprof.html#Statprof), you get something like this:

```
> ,profile (compile-file "module/ice-9/psyntax-pp.scm")
%     cumulative   self
time   seconds     seconds  procedure
 12.41      1.61      1.61  language/cps/intmap.scm:393:0:intmap-ref
  6.35      1.05      0.82  vector-copy
  5.92     13.09      0.77  language/cps/intset.scm:467:5:visit-branch
  5.05      0.71      0.65  language/cps/intmap.scm:183:0:intmap-add!
  4.62      1.40      0.60  language/cps/intset.scm:381:2:visit-node
  3.61      0.93      0.47  language/cps/intset.scm:268:0:intset-add
  3.46      0.49      0.45  language/cps/intset.scm:203:0:intset-add!
  3.17      1.01      0.41  language/cps/intset.scm:269:2:adjoin
  3.03      1.46      0.39  language/cps/intmap.scm:246:2:adjoin
[...]

```

("Cumulative seconds" can be greater than the total number of seconds for functions that have multiple activations live on the stack.)

These results would seem to unequivocally indicate that the [switch to persistent data structures](https://wingolog.org/archives/2015/07/27/cps-soup) in the new compiler is to blame. This is a somewhat disheartening realization; I *love* working with the new data structures. They let me write better code and think about bigger things.

Seeing that most of the time is spent in intmap and intset manipulations, I've tried off and on over the last few months to speed them up. I tried at one point replacing hot paths with C -- no speedup, so I threw it away. I tried adding an alternate intmap implementation that, for transient packed maps, would store the map as a single vector; no significant speedup, binned it. I implemented integer unboxing in the hopes that it would speed up the results; more about that in another missive. I stared long and hard at the generated code, looking for opportunities to improve it (and did make some small improvements). Even when writing this article, the results are such a shame that I put the article on hold for a couple weeks while I looked into potential improvements, and managed to squeak out another 10%.

In retrospect, getting no speedup out of C hot paths should have been a hint.

For many years, a flat statistical profile with cumulative/self timings like the one I show above has been [my go-to performance diagnostic](https://wingolog.org/tags/profiling). Sometimes it does take a bit of machine sympathy to understand, though; when you want to know what's calling a hot function, usually you look farther down the list for functions that don't have much self time but whose cumulative time matches the function you're interested in. But this approach doesn't work for hot functions that are called from many, many places, as is the case with these fundamental data structure operations.

Indeed at one point I built a [tool to visualize statistical stack samples](https://wingolog.org/archives/2009/02/09/visualizing-statistical-profiles-with-chartprof), the idea being you often want to see how a program gets to its hot code. This tool was useful but its output could be a bit overwhelming. Sometimes you'd have to tell it to generate PDF instead of PNG files because the height of the image exceeded Cairo's internal limits. The tool also had too many moving pieces to maintain. Still, the core of the idea was a good one, and I incorporated the non-graphical parts of it into Guile proper, where they sat unused for a few years.

Fast-forward to now, where faced with this compiler performance problem, I needed some other tool to help me out. It turns out that in the 2.0 to 2.2 transition, I had to rewrite the profiler's internals anyway to deal with the new VM. The old VM could identify a frame's function by the value in local slot 0; the new one has to look up from instruction pointer values. Because this lookup can be expensive, the new profiler just writes sampled instruction pointer addresses into an array for later offline analysis, eventual distilling to a flat profile. It turns out that this information is exactly what's needed to do a tree profile like I did in chartprof. I had to add cycle detection to prevent the graphs from being enormous, but cycle detection makes much more sense in a tree output than in a flat profile. The result, distilled a bit:

```
> ,profile (compile-file "module/ice-9/psyntax-pp.scm") #:display-style tree
100.0% read-and-compile at system/base/compile.scm:208:0
  99.4% compile at system/base/compile.scm:237:0
    99.4% compile-fold at system/base/compile.scm:177:0
      75.3% compile-bytecode at language/cps/compile-bytecode.scm:568:0
        73.8% lower-cps at language/cps/compile-bytecode.scm:556:0
          41.1% optimize-higher-order-cps at language/cps/optimize.scm:86:0
            [...]
          29.9% optimize-first-order-cps at language/cps/optimize.scm:106:0
            [...]
          1.5% convert-closures at language/cps/closure-conversion.scm:814:0
            [...]
          [...]
        [...]
      20.5% emit-bytecode at language/cps/compile-bytecode.scm:547:0
        18.5% visit-branch at language/cps/intmap.scm:514:5
          18.5% #x7ff420853318 at language/cps/compile-bytecode.scm:49:15
            18.5% compile-function at language/cps/compile-bytecode.scm:83:0
              18.5% allocate-slots at language/cps/slot-allocation.scm:838:0
                [...]
      3.6% compile-cps at language/tree-il/compile-cps.scm:1071:0
        2.5% optimize at language/tree-il/optimize.scm:31:0
        0.6% cps-convert/thunk at language/tree-il/compile-cps.scm:924:0
        0.4% fix-letrec at language/tree-il/fix-letrec.scm:213:0
  0.6% compile-fold at system/base/compile.scm:177:0
    0.6% save-module-excursion at ice-9/boot-9.scm:2607:0
      0.6% #x7ff420b95254 at language/scheme/compile-tree-il.scm:29:3
        [...]

```

I've uploaded the full file [here](https://wingolog.org/pub/psyntax-profile-2016-01-10.txt), for the curious Guile hacker.

So what does it mean? The high-order bit is that we spend some 70% of the time in the optimizer. Indeed, running the same benchmark but omitting optimizations gets a much more respectable time:

```
$ time meta/uninstalled-env \
  guild compile -O0 module/ice-9/psyntax-pp.scm -o /tmp/foo.go
wrote `/tmp/foo.go'

real	0m3.050s
user	0m3.404s
sys	0m0.060s

```

One of the results of this investigation was that we should first compile the compiler with `-O0` (no optimizations), then compile the compiler with `-O2` (with optimizations). This change made it into the 2.1.1 release a couple months ago.

We also spend around 18.5% of time in slot allocation -- deciding what local variable slots to allocate to CPS variables. This takes time because we do a precise live variable analysis over the CPS, which itself has one variable for every result value and a label for every program point. Then we do register allocation, but in a way that could probably be optimized better. Perhaps with -O0 we should use a different strategy to allocate slots: one which preserves the values of variables that are available but dead. This would actually be an easier allocation task. An additional 1.5% is spent actually assembling the bytecode.

Interestingly, partial evaluation, CPS conversion, and a couple of other small optimizations together account for only 3.6% of time; and reading and syntax expansion account for only 0.6% of time. This is good news at least :)

**up in the trees, down in the weeds**

Looking at the top-down tree profile lets me see that the compiler is spending most of its time doing things that the Guile 2.0 compiler doesn't do: loop optimizations, good slot allocations, and so on. To an extent, then, it's to be expected that the Guile 2.2 compiler is slower. This also explains why the C fast-paths weren't so effective at improving performance: the per-operation costs for the already pretty low and adding C implementations wasn't enough of a speedup to matter. The problem was not that `intmap-ref` et al were slow, it was that code was calling them a lot.

Improving the optimizer has been a bit challenging, not least due to the many axes of "better". Guile's compiler ran faster before the switch to ["CPS soup"](https://wingolog.org/archives/2015/07/27/cps-soup) and persistent data structures, but it produced code that ran slower because I wasn't able to write the optimizations that I would have liked. Likewise, Guile 2.0's compiler ran faster, because it did a worse job. But before switching to CPS soup, Guile's compiler also used more memory, because [per-program-point and per-variable computations were unable to share space with each other](https://wingolog.org/archives/2014/07/01/flow-analysis-in-guile).

I think the top-down profiler has given me a better point of view in this instance, as I can reason about what I'm doing on a structural level, which I wasn't able to understand from the flat profile. Still, it's possible to misunderstand the performance impact of leaf functions when they are spread all over a tree, and for that reason I think we probably need both kinds of profilers.

In the case of Guile's compiler I'm not sure that I'll change much at this point. We'll be able to switch to native compilation without a fundamental compiler rewrite. But spending most of the time in functions related to data structures still seems pretty wrong to me on some deep level -- what if the data structures were faster? What if I wrote the code in some other way that didn't need the data structures so much? It gnaws at me. It gnaws and gnaws.

**the half strap**

Unfortunately, while compiling Scheme to native code will probably speed up the compiler, it won't necessarily speed up the bootstrap. I think the compiler has some 800 KB of source code right now, and let's say that we're able to do native compilation with 1200 KB. So 50% more code, but probably the result is two to ten times faster on average: a win, in terms of compiler speed, when compiled. But for bootstrap time, because in the beginning of the bootstrap most of the compiler isn't compiled, it could well be a slowdown.

This is the disadvantage of bootstrapping from an interpreter -- the more compiler you write, the slower your strap.

Note that this is different from the case where you bootstrap from a compiled Scheme compiler. In our case we do a half-bootstrap, first building an interpreter in C, compiling the interpreter in Scheme, then bootstrapping off that.

It's a common trope in compiler development where the heroic, farsighted compiler hacker refuses to add optimizations unless they make the compiler bootstrap faster. Dybvig says as much in his "History of Chez Scheme" paper. Well, sure -- if you're willing to accept complete responsibility for bootstrapping. From my side, I'm terrified that I could introduce some error in a binary that could reproduce itself worm-like into all my work and it make it impossible to change anything. You think I jest, but the [Sanely Bootstrappable Common Lisp](http://www.doc.gold.ac.uk/~mas01cr/papers/s32008/sbcl.pdf) papers instilled me with fear. Want to change your tagging scheme? You can't! Want to experiment with language, start programming using features from your own dialect? You can't! No, thank you. I value my sanity more than that.

Incidentally, this also answers a common question people have: can I use some existing Guile to compile a new Guile? The answer is tricky. You can if the two Guiles implement the same language and virtual machine. Guile-the-language is fairly stable. However, due to the way that the VM and the compiler are co-developed, some of the compiler is generated from data exported by `libguile`. If that information happens to be the same on your Guile, then yes, it's possible. Otherwise no. For this reason it's not something we describe, besides cross-compilers from the same version. Just half strap: it takes a while but it's fairly fool-proof.

**and that's it!**

Thanks for reading I guess. Good jobbies! Next time, some words on Lua. Until then, happy strapping!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [guile 2.2 omg!!!](/archives/2017/03/15/guile-2-2-omg)
- [a lambda is not (necessarily) a closure](/archives/2016/02/08/a-lambda-is-not-necessarily-a-closure)

### 9 responses

1. Carlos says:[11 January 2016 10:56 PM](#e26b464f31202f3851aa5bbc0a48417966a220db)

   The sanely bootstrappable papers actually exist to show that the flexibility you want such as changing tagging schemes is straightforward under the sbcl build model! In fact, what's so nice is that sbcl native compiles twice to bootstrap, without any throwaway compilers or interpreters, giving a nice 5 min build time compared to 35 minute interpreted bootstrap. It's very similar to how c self builds.

2. Jan Morén says:[11 January 2016 11:41 PM](#66169f4e67cffb04146beb5796aa86238c41e54b)

   From a language designers point of view the bootstrap time is surely interesting. And for the language implementor - such as yourself - that spends a good time every week waiting for the thing to compile I'm sure it looms large.

   But the vast majority of users never get closer to the building process than "apt-get install guile" the average number or compiles per user is only a rounding error away from zero.

   And in such a case, the bootstrap time (distinct from guile code compile time) really ought to have little to no weight when deciding on language implemention?

3. [John Cowan](http://www.ccil.org/~cowan) says:[12 January 2016 1:54 AM](#4660b629c486b97444edd36aece254ac783e9b6a)

   Are the .go files machine independent or nearly so? If so, then they could be included in Guile release tarballs. That's what Chicken does: the machine-independent generated C is included in the release tarball, making the initial installation fast. Then if you update using git, you have a working Chicken that you can use to build the new Chicken, provided you don't wait too long between rebuilds. (Obviously there is a tradeoff between download time and install time, but download time is not usually a big deal nowadays.)

   If you insist on building Chicken from git from scratch without the use of a release tarball, you can still do that if you have a single Chicken compiler binary as a mini-bootstrap; a few such binaries are provided on the download site. It doesn't even have to match perfectly: I have bootstrapped the Cygwin version of Chicken straight out of git using the MinGW compiler binary. However, this mechanism doesn't correspond to anything in Guile.

   Jan: Unfortunately, apt-get and other package installers are still more likely to install Guile 1.x rather than Guile 2.x.

4. Thomas says:[13 January 2016 4:38 PM](#285eb33c0e32ef0d07bcbd13e6f7e34a838899a8)

   I agree with Carlos that you might have missed the point of Christophe Rhode's paper on SBCL. It requires another Common Lisp to bootstrap, rather than being fully self-bootstrapping, and that gives you exactly the freedom you seem to want. Doing that frees you from inheriting junk from the host environment.

   In fact, it sounds like Guile is already doing this, just in a poor form. The C-based interpreter is just a very, very, very inefficient Scheme. Rather than require that particular Scheme implementation, allow other Schemes to stand in. That could be an old version of Guile, or a C-based Scheme from another implementation.

   If you really want to speed up the bootstrapping phase, ditch the interpreter. Make a naive C-based Scheme compiler that's just good enough to run the Guild compiler. Any features you don't use in the Guile compiler (full continuations, for example?) wouldn't be needed. And it would run much faster than a naive interpreter.

5. [wingo](https://wingolog.org/) says:[13 January 2016 9:02 PM](#d3a2496a963f659c425e5c8a1e3d8971ec3a0037)

   I think Thomas and Carlos missed my second point about bootstrapping, which is about common language. SBCL is sanely bootstrappable because it is written in Common Lisp, and Common Lisp is a more or less portable language with a number of high quality implementations. If we switched Guile right now to be bootstrapped from itself, there is no other Scheme that could bootstrap it, so we'd be in a pretty bad way. If instead we built some other Guile to help us bootstrap, then we would have the proverbial two problems :)

6. Carlos says:[13 January 2016 10:07 PM](#9c78bd6eab0980897e86cd89b6f3c014f2797334)

   Isn't it possible for the guile compiler to be written in portable scheme, which could bootstrap off of any portable scheme implementation? Then the extra guile specific runtime things could be loaded after the compiler compiles itself. Similar to how SBCL's compiler is written without CLOS, and compiles itself, then loads CLOS into the target runtime image "warm load". I imagine the issue here would be the fact that scheme is notoriously under specified and therefore making the guile compiler portable across scheme implementations for bootstrap would be difficult. (although I still think that the subset of Common Lisp the sbcl compiler is written in isn't particularly more expressive than the RSRN specifications)

7. Thomas says:[14 January 2016 2:48 PM](#4cfd757361d966fb2150866c634fd0eaa9eda9ad)

   I would be quite surprised if the Guile compiler is written in Guile, not Scheme, in any deep way. Would it really be difficult to make a portable or portable-ish layer atop R5RS that would implement the extensions the compiler depends on? Bootstrapping Gambit from source takes about 3.5 minutes on a modern machine. A gambit-hosted guile compiler should be able to compile the guile sources in a blink of an eye.

   Or, like I suggested before, implement a naive Guile-to-C compiler good enough to handle the parts of the language used in the compiler. It will be a lot more performant than a naive interpreter.

8. [Christophe](http://www.doc.gold.ac.uk/~mas01cr/) says:[14 January 2016 3:13 PM](#5c14661ac3955689cda38a489b05bd7177686ebc)

   I sort-of wasn't going to say this, but one reason that it's not too painful to do the sanely-bootstrappable thing for CL but would probably be a bit more for a Scheme is that CL is a much bigger language. So we can write the SBCL compiler in sufficiently portable CL without giving up structures, conditions, restarts, special variables, bignums, IEEE floating point numbers, and so on. We have to give up generic functions for unrelated boring reasons, but in principle we wouldn't have to (though we would have to give up the MOP, most likely); there are some lurking pits of unportability, notably around floating point, but also around warning severity, and some developer time is spent smoothing those things over.

   I think I agree with (what I read to be) Andy's view, that although Guile-strap from an interpreter is slow, Guile-strap from Guile leads to tentacles and bootstrapping horrors further down the line, and if it's not viable to Guile-strap from (portable) Scheme then you might be in the most reasonable sweet spot.

9. [John Cowan](http://www.ccil.org/~cowan) says:[14 January 2016 3:46 PM](#e0e784ac829edc8659bdfa35300891476cdaad1b)

   *CL is a much bigger language*

   This is one of the motivations for R7RS-large.

   *implement a naive Guile-to-C compiler*

   To be useful in a bootstrap it would have to be written in C, which nobody wants to do even for a naive compiler. Greenspun's Tenth Law would kick in instantly.

Comments are closed.
