---
title: Advanced Compilers Weeks 3-5
url: https://blog.regehr.org/archives/1428
published: "2016-09-29T05:41:12Z"
feed: regehr
guid: http://blog.regehr.org/?p=1428
---

# Advanced Compilers Weeks 3-5

This continues a [previous post](https://blog.regehr.org/archives/1419).

We went through the lattice theory and introduction to dataflow analysis parts of [SPA](https://cs.au.dk/~amoeller/spa/). I consider this extremely good and important material, but I’m afraid that the students looked pretty bored. It may be the case that this material is best approached by first looking at practical aspects and only later going into the theory.

One part of SPA that I’m not super happy with is the material about combining lattices (section 4.3). This is a useful and practical topic but the use cases aren’t really discussed. In class we went through some examples, for example [this function that cannot be optimized by either constant propagation or dead code elimination alone](https://godbolt.org/g/z5PQgR), but can be optimized by their reduced product: [conditional constant propagation](https://scholarship.rice.edu/bitstream/handle/1911/16807/9610626.PDF?sequence=1). Which, as you can see, is implemented by both LLVM and GCC. Also, [this example](https://godbolt.org/g/gpogyj) cannot be optimized by either sign analysis or parity analysis, [but can be optimized using their reduced product](http://www.csse.monash.edu.au/~mbanda/papers/tplp95.pdf).

We didn’t go into them, but I pointed the class to the foundational papers for [dataflow analysis](https://drona.csa.iisc.ernet.in/~deepakd/pav/kildall-popl73.pdf) and [abstract interpretation](http://www.di.ens.fr/~cousot/publications.www/CousotCousot-POPL-77-ACM-p238--252-1977.pdf).

I gave an assignment to implement subtract and bitwise-and transfer functions for the interval abstract domain for signed 5-bit integers. The bitwidth is small so I can rapidly do exhausive testing of students’ code. Their subtract had to be correct and maximally precise — about half of the class accomplished this. Their bitwise-and had to be correct and more precise than always returning top, and about half of the class accomplished this as well (a maximally precise bitwise-and operator for intervals is not at all easy — try it!). Since not everyone got the code right, I had them fix bugs (if any) and resubmit their code for this week. I hope everyone will get it right this time! Also I will give prizes to students whose bitwise-and operator is on the Pareto frontier (out of all submitted solutions) for throughput vs precision and code size vs precision. Here are the results with the Pareto frontier in blue and the minimum and maximum precision in red (narrower intervals are better).

![](https://blog.regehr.org/extra_files/size-hw4.png)

![](https://blog.regehr.org/extra_files/speed-hw4.png)

Impressively, student k implemented an optimally precise bitwise-and transfer function! Student c’s transfer function returned an answer other than top only for intervals of width 1. Mine (labeled JOHN) looked at the number of leading zeroes in both operands.

We looked at the LLVM implementation of the bitwise domain (“known bits”, they call it) which lives in [ValueTracking.cpp](https://llvm.org/svn/llvm-project/llvm/branches/release_39/lib/Analysis/ValueTracking.cpp). This analysis doesn’t have a real fixpoint computation, it rather simply walks up the dataflow graph in a recursive fashion, which is a bit confusing since it is a forward dataflow analysis that looks at nodes in the backward direction. The traversal stops at depth 6, and isn’t cached, so the code is really very easy to understand.

We started to look at how LLVM works, I went partway through some [lecture notes by David Chisnall](http://llvm.org/devmtg/2012-04-12/Slides/Workshops/David_Chisnall.pdf). We didn’t focus on the LLVM implementation yet, but rather looked at the design, with a bit of focus on SSA, which is worth spending some time on since it forms the foundation for most modern compilers. I had the students read the first couple of chapters of this [drafty SSA book](http://ssabook.gforge.inria.fr/latest/book.pdf).

Something I’d appreciate feedback on is what (besides SSA) have been the major developments in ahead-of-time compiler technology over the last 25 years or so. Loop optimizations and vectorization have seen major advances of course, as have verified compilers. In this class I want to steer clear of PL-level innovations.

Finally, former Utah undergrad and current Googler Chad Brubaker visited the class and gave a guest lecture on [UBSan in production Android](http://android-developers.blogspot.com/2016/05/hardening-media-stack.html): very cool stuff! Hopefully this motivated the class to care about using static analysis to remove integer overflow checks, since they will be doing assignments on that very topic in the future.
