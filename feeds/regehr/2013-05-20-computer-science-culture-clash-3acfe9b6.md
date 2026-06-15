---
title: Computer Science Culture Clash
url: https://blog.regehr.org/archives/952
published: "2013-05-20T20:59:02Z"
feed: regehr
guid: http://blog.regehr.org/?p=952
---

# Computer Science Culture Clash

It’s not uncommon for an empirical CS researcher to get a review saying something like “Sure, these results look good, but we need to reject the paper since the authors never proved anything about the worst case.” Similarly, when I interviewed for faculty jobs ten years ago, a moderately famous professor spent a while grilling me about the worst-case performance of a static analysis tool that I had written. This was, to me, an extremely uninteresting topic but luckily there’s an easy answer for that particular class of tool. I recall noticing that he did not seem particularly interested in what the tool did, or if it was actually useful.

Problems like these are yet another consequence of the deep and wide divide between the math and engineering sides of computer science. In caricature form, the two approaches work like this:

- An engineer sees that characterizing the worst-case performance of a lawnmower is a fool’s errand, and focuses instead on work that will enable the creation of more effective and less expensive mowers.
- A mathematician also sees that there’s no way to make any guarantees about the worst-case performance of a lawnmower, and responds by creating an abstract model of mowing that is easier to do proofs about: “Assume an infinite supply of gasoline, a frictionless engine, and a circular lawn of unit radius…”

The problem occurs when these people need to evaluate each others’ work. Both are likely to turn up their noses.

My own view is that guarantees are a wonderful thing. For example, the purpose of the static analysis tool that I was grilled about while interviewing was to make guarantees about embedded systems (just to be clear: I am very interested in guarantees about embedded systems, and wholly uninterested in guarantees about tools that analyze embedded systems). However, we need to keep in mind that guarantees (and their friends, impossibility results) are mathematical statements that abstract away a lot of real-world detail. Of course, abstraction is at the core of computer science and even the most hard-nosed engineers among us must abstract away details in order to accomplish anything. But some abstractions are good — they capture important details while dropping irrelevant clutter — whereas others drop something essential.

One of my favorite stories where the mathematical side of CS rescued the engineers comes from the early development of optimizing compilers. The basic idea behind optimization is to repeatedly replace part of the program with a faster piece of code that is functionally equivalent. The problem is that without a good definition of “functionally equivalent” some optimizations are going to break some programs and when this happens we have no good basis for determining whether it is the program or the compiler that is at fault. As an example, we might ask if it is OK for an optimizing compiler to remove a delay loop from a program. Clearly this will break programs that rely on the delay. Anyhow, as I understand the history, throughout the 1950s and 1960s this basic ambiguity did not deter the engineers from creating some nice optimizing compilers but there were persistent low-grade problems in figuring out which transformations were legal. Subsequently, people were able to develop theories — dataflow analysis and abstract interpretation — relating program semantics and compiler optimizations in such a way that it became possible to figure out whether any particular optimization is valid or not. These theories are concrete and pervasive enough that, for example, the string “lattice” appears about 250 times in the LLVM source code.

The point is that both the engineering and math sides can benefit from keeping the dialog open. Thus, I’ve tried to come up with a few recommendations for different situations we might find ourselves in.

When submitting papers:

- Is there anything to worry about? If you’re an engineer submitting to SOSP, or a mathematician submitting to STOC, all reviews will be written by your own kind of people, so there’s no problem.

- **Engineers:** Your first impression will likely be that the mathematical side has nothing to say about your work because you are solving real-world problems. This may be true or it may not be, but in any case don’t be ignorant. Read and understand and cite relevant work and describe how it relates to your research.
- **Mathematicians:** The engineers want something that is, or could be, useful. If your work advances the state of the art in a way that could be transitioned into practice, say so and provide some evidence. In other words, help the reviewers understand why your work is of interest to people who are not mathematicians.
- Make your paper’s contributions extremely clear, and moreover make it clear what kind of contributions they are. The second part is difficult and important. Then, regardless of how clear you have been, be ready for the engineers / mathematicians to fail to see the value in the work.
- Be honest about your contributions. If you’re really an engineer, you’re probably on very shaky ground when proposing a fundamental new model of computation. If you’re really a mathematician, don’t start your paper with a page about making life better for users and then spend the remaining nine pages proving lemmas.
- Know when to admit defeat. There are some communities that are simply not going to accept your papers reliably enough that you can make a home there. If you don’t want to change your style of work, you’ll need to take your business elsewhere. It’s their loss.

When reviewing papers:

- Don’t dismiss the work out of hand just because it fails to attack any problem that you care about. Try to understand what the authors are trying to accomplish, and try to evaluate the paper on those terms. This is hard because they will often not state the goals or contributions in a way that makes sense or seems interesting to you.
- **Mathematicians:** If your criticism is going to be something like “but you failed to prove a lower bound,” think carefully about whether the authors’ stated contributions would be strengthened by a lower bound, or rather if it is simply you who would like to work on the lower bound.
- **Engineers:** If your criticism is going to be something like “you failed to evaluate your work on 10,000 machines,” think carefully about whether this makes sense to say. If the authors’ model has rough edges, can they be fixed or are they fundamentally at odds with the real world? Are there any insights in this paper that you or another engineer could use in your own work?
- **Mathematicians:** If your criticism is going to be something like “this is a trivial application of known techniques,” be careful because the gap between knowing something and making it work is often wide. Also, it is a success, rather than a failure, if the engineers were able to readily adapt a result from the mathematical side. Of course, if the adaptation really is trivial then call them on it.

- **Engineers:** If your criticism is going to be something like “this seems needlessly complicated” try to keep in mind that the penalty for complexity in math is less (or at least it affects us differently) than in engineering. Often the added complexity is used to get at a result that would be otherwise unattainable. Of course, if the complexity really is useless, call them on it.

- If you are truly not qualified to review the paper, try to wiggle out of it. Often an editor or PC chair will be able to swap out the paper — but only if they are asked early enough. At the very least, clearly indicate that your review is low-confidence.

I hope that I’ve made it clear that close interaction between the mathematicians and engineers is an opportunity rather than a problem, provided that each side respects and pays attention to the other. I should add that I’ve made mistakes where there was a piece of theoretical CS research that I thought was useless, that turned out not to be. Since then I’ve tried to be a little more humble about this sort of thing.

In this piece I’ve sort of pretended that engineers and mathematicians are separate people. Of course this is not true: most of the best people can operate in either mode, at least to a limited extent.
