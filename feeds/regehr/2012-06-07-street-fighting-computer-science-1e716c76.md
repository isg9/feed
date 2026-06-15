---
title: Street Fighting Computer Science
url: https://blog.regehr.org/archives/724
published: "2012-06-07T03:31:53Z"
feed: regehr
guid: http://blog.regehr.org/?p=724
---

# Street Fighting Computer Science

One of my favorite recent books is [Street Fighting Mathematics](http://mitpress.mit.edu/books/full_pdfs/Street-Fighting_Mathematics.pdf): a collection of techniques and heuristics for rapidly and roughly estimating the solutions to problems that may be very difficult to solve exactly. The book is important because estimation is incredibly useful for understanding the world and because our education system does not do a very good job in teaching it. I think the tacit assumption is that people will pick up street fighting methods on their own.

During the last couple of years I’ve started to wonder how to teach street fighting computer science. The need for rapid estimation comes up all the time, for example, when studying operating systems. How long does it take to read all of a 2 TB hard disk? How long does it take a packet to cross a data center, a city, or a continent? How big is a 36-bit address space? These are very simple problems compared to the ones that Mahajan tackles but even so, not everyone can quickly rattle off approximate answers.

Mahajan’s book is structured around a collection of general-purpose techniques:

- **Dimensional analysis** — In CS terms, the universe has a type system and it can be exploited to make inferences and catch mistakes.
- **Easy cases** — A correct solution must work for all input values — so set parameters to zero, one, infinity, or whatever makes the math simple.
- **Lumping** — Use discrete approximations (perhaps a very coarse one, such as invoking the trapezoid rule with just one trapezoid) instead of calculus.
- **Pictorial proofs** — Use visualization to gain intuition into problems who symbolic forms are hard to understand.
- **Taking out the big part** — Split an answer into a big part and a correction; don’t even compute the correction unless more precision is needed.
- **Analogy** — Find an analogous but simpler problem, solve it, and project the result back to the original problem.

These apply to street fighting CS just as well as street fighting mathematics. For example, taking out the big part is something we do all the time when determining efficiency of algorithms. On the other hand, the bulk of the technical content in Mahajan’s book is a collection of calculus-based problems about the physical world. These aren’t really where we want to be spending most of our time in a CS course (especially if everyone else’s calculus is as rusty as mine). What might we cover instead? Here are a few ideas:

- **Powers of 2 and 10 —** This is basic, but all CS folks need to be able to rapidly convert any power of 2 up to about 40 to the nearest (or at least a close) power of 10, and vice versa. Similarly, converting between hex and binary should be quick.
- **Important constants —** These are mainly sizes (hard disk, CD/DVD, RAM on a PC and smartphone, various caches, etc.), times (packet transmission, cycles, seeks, interrupt latencies, human perceptual constants), and rates (signal propagation in copper and fiber, bandwidth of various interconnects, power consumption of common devices, etc.). [Here’s a nice list of times](https://gist.github.com/2841832). A few important ASCII codes (such as those for 0, A, and a) should be memorized.
- **Counting arguments —** The pigeonhole principle and such.
- **Lopsided distributions —** For example when estimating flight time through a network using measurements, all outliers will be on one side.
- **Rules of thumb —** Amdahl’s Law, Occam’s Razor, etc.

Next, we want a collection of diverse example problems to work out. Probably these will mostly come from people’s everyday experience making computer systems work, but we can also draw on:

- Books by [Martin Gardner](http://www.amazon.com/Colossal-Book-Short-Puzzles-Problems/dp/0393061140/) and A. K. Dewdney (Magic Machine, Tinkertoy Computer, Turing Omnibus, etc.)
- Collections of CS interview questions including [How Would You Move Mt Fuji?](http://www.amazon.com/Would-Move-Mount-Microsofts-Puzzle/dp/B000JBY0RY/)
- Bentley’s essay [The Back of the Envelope](http://www.eecs.harvard.edu/cs261/background/p180-bentley.pdf), from CACM and Programming Pearls; also [The Envelope is Back](http://www.eecs.harvard.edu/cs261/background/p176-bentley.pdf)

Basically I’m just fishing around here to see if this idea has legs; I’d appreciate feedback.
