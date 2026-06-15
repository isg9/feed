---
title: Reproducibility in Computer Systems Research
url: https://blog.regehr.org/archives/1118
published: "2014-03-18T18:15:36Z"
feed: regehr
guid: http://blog.regehr.org/?p=1118
---

# Reproducibility in Computer Systems Research

[These results about reproducibility in CS](http://reproducibility.cs.arizona.edu/) have been the subject of lively discussion at Facebook and [G+](https://plus.google.com/u/0/+ShriramKrishnamurthi/posts/Tf4D2HKQRKi) lately. The question is, for 613 papers, can the associated software be located, compiled, and run? In contrast with something I often worry about — [producing good software](https://blog.regehr.org/archives/1058) — the bar here is low, since even a system that runs might be without value and may not even support the results in the paper.

It is great that these researchers are doing this work. Probably everyone who has worked in applied CS research for any length of time has wanted to compare their results against previous research and been thwarted by crappy or unavailable software; certainly this has happened to me many times. The most entertaining part of [the paper](http://reproducibility.cs.arizona.edu/tr.pdf) is Section 4.3, called “So, What Were Their Excuses? (Or, The Dog Ate My Program)”. The recommendations in Section 5 are also good, I think all CS grad students and faculty should read over this material. Anecdote 2 (at the very end of the paper) is also great.

On the other hand, it’s not too hard to quibble with the methods here. For example, one of my papers was [placed in the irreproducible category](http://reproducibility.cs.arizona.edu/data/pldi12_RegehrCCEEY12_build.txt) with the reason of:

**> could not install: style, flex, GNU indent, LLVM/clang 3.2**

It’s not clear why installing these dependencies was difficult, but whatever. In a Facebook thread a number of other people reported learning that they have been doing irreproducible research for similar reasons. The [G+ thread](https://plus.google.com/u/0/+ShriramKrishnamurthi/posts/Tf4D2HKQRKi) also contains some strong opinions about, for example, [this build failure](http://reproducibility.cs.arizona.edu/data/oopsla12_KangR12_build.txt).

Another thing we might take issue with is the fact that “reproducible research” usually means something entirely different than successfully compiling some code. Usually, we would say that research is reproducible if an independent team of researchers can repeat the experiment using information from the paper and end up with similar (or at least non-conflicting) results. For a CS paper, you would use the description in the paper to implement the system yourself, run the experiments, and then see if the results agree with those reported.

Although it’s not too hard to put some code on the web, producing code that will compile in 10 or 20 years is not an easy problem. A stable platform is needed. The most obvious one is x86, meaning that we should distribute VM images with our papers, which seems clunky and heavyweight but perhaps it is the right solution. Another reasonable choice is the Linux system call API, meaning that we should distribute statically linked executables or [equivalent](http://www.pgbovine.net/cde.html). Yet another choice (for research areas where this works) might be a particular version of Matlab or some similar language.

**UPDATE:** You can help [watch the watchmen here](http://cs.brown.edu/~sk/Memos/Examining-Reproducibility/).
