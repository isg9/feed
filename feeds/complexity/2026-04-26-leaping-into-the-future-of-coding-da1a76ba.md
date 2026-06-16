---
title: LEAPing into the Future of Coding
url: https://blog.computationalcomplexity.org/2026/04/leaping-into-future-of-coding.html
published: "2026-04-26T21:16:12Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-1407202525957220168
---

# LEAPing into the Future of Coding

A few months ago in Oxford, Bernard Sufrin, an emeritus fellow, said he's looking to hire a student to implement LEAP (Logic Engine for Argument by Pointing), a way to teach logic by proving basic logic theorems via pointing and clicking. Rahul Santhanam said why not give it to AI. Bernard said AI can't handle this task. I thought why not give it a try.

I gave Claude a single prompt that Bernard helped me formulate:

> Architecture of a system to support proof by pointing in first order logic using LEAP

About an hour later, without giving additional details, we had a working prototype. [Give it a try](https://leap-y2r7.onrender.com/). I put the code and some more details on [GitHub](https://github.com/fortnow/leap/tree/main).

Watching Claude work was amazing. Claude created [an architecture](https://github.com/fortnow/leap/blob/main/Claude%20Architecture.md) based on the paper [Proof by Pointing](https://doi.org/10.1007/3-540-57887-0_94) by Yves Bertot, Gilles Kahn and Laurent Théry.

Claude then asked me:

> Would you like me to proceed with implementing any of these layers?

Why not? So I told Claude to go ahead.

It started coding and it created 78 test cases and kept debugging itself until all those test cases worked out. It took me longer to get the program working on the web than Claude took to program it. I sent [the link](https://leap-y2r7.onrender.com/) to Bernard who responded "Wow! I take it back if the program was *all* synthesized from the prompt."

When I asked Bernard a month later if I could post about this program, he agreed, though he had some additional comments.

> The consolation for me is that the outcome, though surprising and I suppose delightful (though it didn't manage rules for quantifiers), didn't refute my conjecture that Claude et al cannot do "architecture". The architecture (MVC) is stock; the "proof by pointing" hint led it to a paper of the same name which gave details of how to derive terms/formulae/goals in the (unfinished) proof from screen coordinates. Maybe you could give a substantively different prompt.
>
> It may be of interest to know that the Bornat/Sufrin JAPE program, still on [GitHub](https://github.com/RBornat/jape), was eventually a lot more ambitious when it came to selecting things: terms, subterms, goals. But it took more than 2 hours to build!

Bernard has a point. It wasn't just the fifteen-word prompt alone. Claude leaned on the Bertot et al. paper to guide its design and implementation. Still, that Claude did architect, build and debug the system from the prompt and the paper is truly impressive. We've truly crossed a threshold for coding, far beyond what would have been possible just a few months earlier.
