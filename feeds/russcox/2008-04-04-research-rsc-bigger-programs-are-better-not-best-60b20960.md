---
title: 'research!rsc: Bigger Programs are Better, not Best'
url: https://research.swtch.com/beaver
published: "2008-04-04T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/beaver
---

# research!rsc: Bigger Programs are Better, not Best

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Bigger Programs are Better, not Best Russ Cox April 4, 2008 *research.swtch.com/beaver* Posted on Friday, April 4, 2008.

When computer scientists study algorithms, they are interested in how much time or working space an algorithm requires, as a function of the input size: quicksort uses *O*( *n* log *n*) time, but only *O*( *n*) space. For the most part, computer scientists don't study how much *code* a problem requires. In fact, part of what it means to be an algorithm is that it can be implemented as a fixed-size program that works regardless of the input size. Intuitively, though, bigger programs should be able to do more than smaller programs.

It is pretty easy to prove that bigger programs are better, or at least that they can do more. To start, we need to precisely define what we mean by the size a program. These kinds of discussions often use Turing machines, but these days not many people are comfortable with Turing machines, so I'm going to use Scheme. \[ [؟](http://en.wikipedia.org/wiki/Irony_mark)\]

Let's define a Scheme program's *size* to be the number of left parentheses it contains. We need to disallow arbitrarily large parenthesis-free expressions, so that there are only a finite number of Scheme programs of a given size. Let's limit functions to three arguments and integer literals to be numbers less than 100. These requirements might make Scheme a little more verbose, but they don't make it any less powerful. Notice that these programs are just expressions that don't take any input.

If we look at all the Scheme programs of a given size, some of those programs evaluate to integers: `(+ 1 (+ 2 3))` has size 2 and evaluates to 6. Let's define *B*( *n*) to be the largest integer that can be produced by evaluating a Scheme program of size *n*. The example demonstrates that *B*(3) is at least 6.

The formal statement of the “bigger is better” assertion is that *B*( *n* +1) > *B*( *n*) for any *n*. For any given *n*, the definition of *B* requires that there be some Scheme program `e` of size *n* that evaluates to *B*( *n*). Then `(+ 1 e)` has size *n* +1 and evaluates to 1+ *B*( *n*). This demonstrates that *B*( *n* +1) is at least 1+ *B*( *n*), so *B*( *n* +1) > *B*( *n*).

There you have it. Bigger programs are better. But it turns out that no matter how big a program you write, it can't compute *B*( *n*). *B*( *n*) gets so big so fast that no fixed-size program can keep up. We can prove this too.

Consider any Scheme function `f` that takes a single integer argument. This is a lambda expression, not a standalone program like we've been considering, but it still has some size *s*. Define *n* = 2 *s* +1. We can write a Scheme program `n1`, also of size *s*, that computes 2 *s* +2 = *n* +1\. The program looks like `(+ 2 (+ 2 ... (+ 2 (+ 2 2)) ...))`. Now consider the Scheme program `(f n1)`, wihch has size *s* + *s* +1 = *n*. Since it has size *n*, it cannot compute a number bigger than *B*( *n*). The value of the argument to `f` is *n* +1, and *B*( *n* +1) > *B*( *n*), so `f` cannot be computing *B*. Essentially, *B* grows faster than any function you can implement in Scheme.\*

There you have it. The function *B* cannot be computed. Bigger has bested the computer.

The original presentation of this result is Tibor Rado's 1962 paper “ [**On Non-Computable Functions**](http://pdos.csail.mit.edu/~rsc/rado62beaver.pdf)” (PDF, 320kB), which appeared in the Bell System Technical Journal. Rado used Turing machines, not Scheme programs, and called the function *Σ*( *n*), not *B*( *n*). He also defined a function *S*( *n*) which is the maximum length of any computation by a Turing machine of size *n* that eventually stops. Rado made a game of trying to make Turing machines of a given size that ran for as long as possible (but stopped), and he called it the “Busy Beaver game.”

The function Σ is sometimes called the Busy Beaver function, leading to a slew of papers by otherwise respectable computer scientists and mathematicians [about busy beavers](http://www.google.com/search?&q=busy-beaver+turing+machine). In particular, a favorite pastime is to attempt to compute *Σ*( *n*) and *S*( *n*) by hand, for small values of *n*. This requires analyzing all possible Turing machines of the given size to figure out whether each eventually stops, and if so, how long it runs and what number it prints.

Using a computer to analyze the easy machines and then doing the hard ones by hand, Rado and his student Shen Lin proved that *Σ*(1) = 1, *S*(1) = 1, *Σ*(2) = 4, and *S*(2) = 6 in their 1965 paper “ [Computer Studies of Turing Machine Problems](http://doi.acm.org.libproxy.mit.edu/10.1145/321264.321270)” (subscription required). Lin proved in his Ph.D. thesis that *Σ*(3) = 6 and *S*(3) = 21. In 1983, Allen Brady proved that *Σ*(4) = 13 and *S*(4) = 107.

Rona Machlin and Quentin Stout summarized the situation nicely in their 1990 paper “ [The Complex Behavior Of Simple Machines](http://www.eecs.umich.edu/~qstout/abs/busyb.html):”

> Brady predicted that there will never be a proof of the values of *Σ*(5) and *S*(5). We are just slightly more optimistic, and are lead to recast a parable due to Erdös (who spoke in the context of determining Ramsey numbers): suppose a vastly superior alien force lands and announces that they will destroy the planet unless we provide a value of the S function, along with a proof of its correctness. If they ask for *S*(5) we should put all of our mathematicians, computer scientists, and computers to the task, but if they ask for *S*(6) we should immediately attack because the task is hopeless.

Michiel Wijers has a [good bibliography](http://www.win.tue.nl/~wijers/bb-index.htm). Allen Brady has recently been exploring [ternary
Turing machines](http://www.cse.unr.edu/~al/BusyBeaver.html).

\\* It doesn't matter that we used Scheme and Rado used Turing machines, because Scheme can simulate a Turing machine and vice versa. In fact, so far no feasible computational model has yet been found that can't be simulated by a Turing machine (or, by extension, by Scheme). The [Church-Turing
thesis](http://en.wikipedia.org/wiki/Church-Turing_thesis) is the name for the hypothesis that anything we would reasonably consider computable can be computed by a Turing machine (or lambda calculus or a general-recursive function, both of which are equivalent). The exact details of what you can do with a given size differs from model to model, but the net result is the same: even a Turing machine can't compute our Scheme-based *B*( *n*). The result would work for any modern programming language, but Scheme makes the proofs particularly elegant.

(Comments originally posted via Blogger.)

- [Screwperman](http://www.blogger.com/profile/11014201732055198675)(April 4, 2008 10:35 PM) If I'm not mistaken, some quicksorts take O(lg n) space.

Good point, though.
