---
title: Proposal for a CompCert Superoptimizer
url: https://blog.regehr.org/archives/496
published: "2011-03-29T04:11:06Z"
feed: regehr
guid: http://blog.regehr.org/?p=496
---

# Proposal for a CompCert Superoptimizer

[CompCert](http://compcert.inria.fr/) is a C compiler that is provably correct. It is best characterized as *lightly optimizing*: it performs a number of standard optimizations but its code improvements are not aggressive when compared to those performed by GCC, Clang, or any number of commercial tools. This piece is about what I believe would be a relatively interesting and low-effort way to make CompCert more aggressive.

Consider these functions:

> ```
> int foo (int x, int y) {
>   return (x==y) || (x>y);
> }
> ```
>
> ```
> int bar (int x) {
>   return (x>3) && (x>10);
> }
> ```

Clearly foo() can be simplified to return “xâ‰¥y” and bar() can be simplified to return “x>10.” A random version of GCC emits:

> ```
> _foo:
>         xorl    %eax, %eax
>         cmpl    %esi, %edi
>         setge   %al
>         ret
>
> _bar:
>         xorl    %eax, %eax
>         cmpl    $10, %edi
>         setg    %al
>         ret
>
> ```

Any aggressive compiler will produce similar output. In contrast, CompCert 1.8.1 does not perform these optimizations. Its output is bulky enough that I won’t give it here. Collectively, the class of missing optimizations that I’m talking about is called *peephole optimizations*: transformations that operate on a very limited scope and generally remove a redundancy. Opportunities for peephole optimizations can be found even in cleanly written code since inlined functions and results of macro expansions commonly contain redundancies.

An aggressive compiler contains literally hundreds of peephole optimizations, and reproducing most of them in CompCert would be time consuming, not to mention unspeakably boring. Fortunately, there’s a better way: most of these optimizations can be automatically derived. The basic idea is from Henry Massalin who developed a [superoptimizer](http://www.cs.utexas.edu/users/lasr/shangri-la/reference/massalin87superoptimizer.pdf) in 1987; it was [significantly improved](http://www-cs.stanford.edu/~aiken/publications/papers/asplos06.pdf) about 20 years later by Bansal and Aiken. This piece is about how to create a superoptimizer that proves its transformations are sound.

The idea is simple: at a suitable intermediate compilation stage — preferably after the regular optimization passes and before any kind of backend transformation — find all subgraphs of the program dependency graph up to a configurable size. For each subgraph G, enumerate all possible graphs of the CompCert IR up to some (probably smaller) configurable size. For each such graph H, if G and H are equivalent and if H is smaller than G, then replace G with H. Subgraph equivalence can be checked by encoding the problem as an [SMT](http://en.wikipedia.org/wiki/Satisfiability_Modulo_Theories) instance and sending the query to some existing solver. The proof of equivalence needed to make CompCert work comes “for free” because there exist SMT solvers that emit proof witnesses. ( [SMTCoq](http://www.lix.polytechnique.fr/~keller/Recherche/smtcoq.html) is an example of a tool that generates Coq proofs from SMT output.) Repeat until a fixpoint is reached — the program being compiled contains no subgraphs that can be replaced.

As an example, the IR for foo() above would contain this code:

> ```
> t1 = x==y;
> t2 = x>y;
> t3 = t1 || t2;
> ```

When attempting to optimize this subgraph, the superoptimizer would eventually test for equivalence with:

> ```
> t3 = x>=y;
> ```

Since t1 and t2 are not subsequently used, a match would be found and the peephole optimization would fire, resulting in smaller and faster code.

Of course, the superoptimizer that I have sketched is extremely slow. The Bansal and Aiken paper shows how to make the technique fast enough to be practical. All of their tricks should apply here. Very briefly, the speedups include:

1. Testing many harvested sequences at once
2. Reducing the search space using aggressive canonicalization
3. Avoiding most SMT calls by first running some simple equivalence tests
4. Remembering successful transformations in a database that supports fast lookup

The Bansal and Aiken superoptimizer operated on sequences of x86 instructions. Although this had the advantage of permitting the tool to take advantage of x86-specific tricks, it also had a couple of serious disadvantages. First, a short linear sequence of x86 instructions harvested from an executable does not necessarily encode an interesting unit of computation. In contrast, if we harvest subgraphs from the PDG, we are guaranteed to get code that is semantically connected. Second, the Stanford superoptimizer has no ability to see “volatile” memory references that must not be touched — it will consequently break codes that use multiple threads, UNIX signals, hardware interrupts, or hardware device registers.

The technique outlined in this piece is what I call a *weak superoptimizer*: it finds short equivalent code sequences using brute force enumeration. A *strong superoptimizer*, on the other hand, would pose the following queries for each harvested subgraph G:

1. Does there exist a subgraph of size 0 that is equivalent to G? If not…
2. Does there exist a subgraph of size 1 that is equivalent to G? If not…
3. Etc.

Clearly this leans much more heavily on the solver. It is the approach used in the [Denali superoptimizer](http://www.hpl.hp.com/techreports/Compaq-DEC/SRC-RR-171.pdf). Unfortunately, no convincing results about the workability of that approach were ever published (as far as I know), whereas the weak approach appears to be eminently practical.

In summary, this post contains what I think are two relatively exploitable ideas. First, a peephole superoptimizer should be based on subgraphs of the PDG rather than linear sequences of instructions. Second, proof-producing SMT should provide a relatively easy path to verified peephole superoptimization. If successful, the result should be a significant improvement in the quality of CompCert’s generated code.

*Update from 3/29:* A random thing I forgot to include in this post is that it would be easy and useful to teach this kind of superoptimizer to take advantage of (nearly) arbitrary dataflow facts. For example, “x /= 16” (where x is signed) is not equivalent to x >>= 4. However, this equivalence does hold if x can be proved to be non-negative using a standard interval analysis. Encoding a fact like “x â‰¥ 0” in the input to the SMT solver is trivial. The nice thing about dataflow facts is that they give the superoptimizer non-local information.

I should also add that when I said “if H is smaller than G, then replace G with H” of course I really mean “cheaper” rather than “smaller” where the cost of a subgraph is determined using some heuristic or machine model.

Even after adding a nice IR-level superoptimizer to CompCert, a backend superoptimizer just like Bansal and Aiken’s is still desirable: it can find target-dependent tricks and it can also get rid of low-level grunge such as the unnecessary register-register moves that every compiler’s output seems to contain. The main fix to their work that is needed is a way to prevent it from removing or reordering volatile memory references; this should be straightforward.
