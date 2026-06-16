---
title: The Trevisan Award and the Decimal Digits of Powers of 2
url: https://scottaaronson.blog/?p=9732
published: "2026-05-10T19:54:41Z"
feed: aaronson
guid: https://scottaaronson.blog/?p=9732
---

# The Trevisan Award and the Decimal Digits of Powers of 2

WHOA … I’ve won the inaugural [Luca Trevisan Award for Expository Work in Theoretical Computer Science](https://www.sigact.org/prizes/trevisan.html)! This has a particular meaning for me as someone who [knew](https://scottaaronson.blog/?p=8057) [Luca Trevisan](https://en.wikipedia.org/wiki/Luca_Trevisan) as well as I did for 25 years — who had him as a professor and thesis committee member, whose blog bounced off his blog, who benefitted tremendously from *his* expository work in TCS — before Luca tragically succumbed to cancer two years ago.

As I told the committee, receiving this award makes me want to use my blog for more actual CS theory exposition, and less venting, in order to retrospectively become worthier of such an honor.

I’m ridiculously grateful to the entire TCS community — my people, my homies — for tolerating me to do what I do.

If you’re curious, here’s the official citation:

> The inaugural Luca Trevisan Award for Expository Work in Theoretical Computer Science is awarded to Scott Aaronson for his sustained and high-impact inspirational efforts to explain and promote our field to broad audiences. His blog Shtetl-Optimized has hosted remarkably frequent and elaborate posts over more than two decades, and has become a central meeting place for wide-ranging conversations across the TCS and Physics communities. Scott Aaronson’s blog posts contain crystal-clear, informative expositions of exciting new results, calibrated evaluations of technological claims, and profound analyses of topics in these fields and (way) beyond. The uniquely enthusiastic and witty style of his writings (including his book *Quantum Computing Since Democritus* and his other lecture notes and surveys), lectures, and interviews have made him a top invitee for both popular and professional appearances, attracting large audiences. These qualities have inspired many students to enter the field, and made Scott Aaronson a go-to person for journalists and scientists alike looking for a definitive word on the latest scientific activity in TCS and quantum computing.

---

In the rest of this post, I’m going to start practicing what I preached—y’know, about turning over more of this blog to *actual exposition*, of a kind that the Trevisan Award could plausibly be meant to encourage. I’ll start with something that’s been on my back burner for the past couple months: namely, the (lightly edited) transcript of a talk that I gave this spring at UT Austin’s undergraduate math club. So, without further ado, and in memory of Luca…

---

**On the Decimal Digits of Powers of 2**

by Scott Aaronson

Hi! I’ve given six previous talks here at UT’s math club, some on relatively “important” topics—Gödel’s Theorem, time travel, [Huang’s proof of the Sensitivity Conjecture](https://arxiv.org/abs/1907.00847), and so on.

Today, I want to talk about an *un* important question, one that my son Daniel, who was then 8, and who’s sitting here in the front row (along with his sister Lily), asked me a few months ago.

Daniel asked: which powers of 2 can you double without needing to carry digits? Clearly 1, 2, 4, 32, and 1024 all have this property, their doubles being 2, 4, 8, 64, and 2048 respectively. Are there any others?

Since I happen to have the powers of 2 up to 220 = 1,048,576 committed to memory since childhood, I confirmed that there were no other examples up to there: 128, 256, 512, 2048, etc. all require carries. So I told Daniel: I can’t *find* any other example, and on that basis, I *conjecture* that there aren’t any more. But if that conjecture is true, I don’t know if it will *ever* be proven, by humans or even AI!

Then I googled it, and saw that this is a [known question](https://math.stackexchange.com/questions/253468/how-many-powers-of-2-are-easy-to-double) (not *very* well known, but there’s a StackExchange post about it). And indeed it had been checked up to 2millions, and no other counterexample had been found.

---

Why did I become confident so quickly that yes, 1024 is *probably* the last example of a power of 2 that can be doubled without carrying?

Because of the heuristic that the decimal digits of 2n are more or less “random,” apart from various constraints that are irrelevant here (like that the last digit always cycles among 2, 4, 6, and 8). And 2n has about n/log210 decimal digits. Since only 0, 1, 2, 3, 4 can be doubled without carrying, the probability of 2n being a counterexample should therefore be about $$ (\\frac{1}{2} )^{n / \\log\_2 10}. $$

So, if we’ve already checked up to (say) n=1000, then the probability of a larger counterexample should be at most

$$ \\sum\_{n=1001}^{\\infty} (\\frac{1}{2} )^{n / \\log\_2 10} $$

which, when we sum the geometric series, is exceedingly close to 0.

---

Ah, but why did I say that I don’t know if the conjecture will ever be proven? Because it seems to belong a large class of similar statements, none of which mathematicians have had any idea how to prove.

**[Variant](https://math.stackexchange.com/questions/1873371/is-216-65536-the-only-power-of-2-that-has-no-digit-which-is-a-power-of) of a conjecture by Jeffrey Shallit:** 65,536 is the only power of 2 that has no power of 2 among its decimal digits.

**[Freeman Dyson’s conjecture (2005):](https://www.edge.org/response-detail/11675)** There’s no power of 2 for which, when you reverse the decimal digits, you get a power of 5.

**[Paul Erdös’s conjecture (1979):](https://arxiv.org/abs/2202.13256)** For every n≥9, there’s at least one ‘2’ in the base-3 representation of 2n.

Or looking even more broadly:

**Conjecture:** The decimal expansion of π is not all 6’s and 7’s after some finite point.

(This would follow from the stronger conjecture that π is [base-10 normal](https://en.wikipedia.org/wiki/Normal_number)—that is, that every finite pattern of decimal digits occurs in it with the limiting frequency you’d expect.)

Or:

**Conjecture:** π+e is an irrational number.

What all the above conjectures have in common, and what I find so fascinating about them, is that they seem *hopeless to prove* for exactly the same reason why they seem *almost certainly true*. Namely, they all seem to be true “merely” because it would be too insane of a coincidence were they false!

The trouble is, that’s not the sort of reason that seems amenable to turning into a proof. Fermat’s Last Theorem is an interesting exception that proves the rule here. That xn+yn=zn has no nontrivial integer solution for n≥3, *did* seem almost certainly true on statistical grounds for n≥4 (and for the n=3 case, a proof goes back to Euler). And of course, FLT *was* ultimately provable. But Wiles’s eventual proof exploited a lucky connection between the Fermat equation and deep, fancy things like modular forms and elliptic curves. At no point did the proof formalize the statistical argument that a 12-year-old could understand, for why the theorem is “almost certainly true.” It simply had nothing to do with the statistical argument.

So then, if you wanted to prove conjectures like my son Daniel’s, or like Shallit’s or Dyson’s or Erdös’s above, the question would be: could these “recreational” problems about base-10 representations ever be connected to anything similarly deep? Right now, it’s very hard to see how they could.

Still, all hope is not lost! Here’s a striking theorem that I learned about when I researched this:

**[Theorem (James Maynard, 2016):](https://arxiv.org/abs/1604.01041)** For every digit a from 0 to 9, there are infinitely many primes with no a’s in their base-10 representation.

The proof uses heavy Fourier-analytic techniques. Likewise, *presumably* there are infinitely many primes that you can double without carrying—2, 3, 11, 13, 23, 31, …—because the primes are much denser than the powers of 2! And presumably there are infinitely many primes that are missing any two decimal digits, or even whose decimal digits consist entirely of 0’s and 1’s. But Maynard’s techniques are not yet powerful enough to prove such things.

---

Even though I promised a topic of no importance today, I can’t resist pointing out a potential connection here to one of the biggest questions on earth right now, and something that’s professionally interested me for the past four years: namely, the question of how to align powerful AIs with human values and prevent them from destroying the world.

[Paul Christiano](https://en.wikipedia.org/wiki/Paul_Christiano), and the [Alignment Research Center](https://www.alignment.org/) in Berkeley that he founded, have developed a [whole program](https://www.alignment.org/blog/a-computational-no-coincidence-principle/) for how to make AI safe that depends on the possibility of formalizing “heuristic arguments”—that is, the kinds of arguments that convince us that the above conjectures are all almost certainly true, even without proofs of them.

The intuition here is that we’ll *never* have a rigorous proof that, for example, a real-world neural network will behave safely under all circumstances—it’s just too complicated. The best we can hope for is an argument that, e.g., “for this model to scheme against humans would require a crazy unexplained coincidence in its weights.” But how can we hope to formalize such arguments? As baby test cases, can we at least formalize our intuitions for why π is normal, or why Daniel’s conjecture is true, in some principled way?

ARC has tried: there’s a [2022 paper](https://arxiv.org/abs/2211.06738) by Christiano, Neyman, and Xu on “Formalizing the presumption of independence.” But it’s tricky, and ARC itself would be the first to agree that the existing results are unsatisfying. How do we even formalize the intuition that, for example, you should be willing to bet at even odds against the 10100th digit of π being a 5?

---

In the rest of this talk, I’d like to circle back to Daniel’s original question about powers of 2, and show you some things that *can* be proved about it—with thanks to Greg Kuperberg and my other friends on Facebook, and in some cases to GPT5Pro.

Let’s start with the following easier question. Is there a power of 2 whose decimal digits *start* with 31415? Or with the complete works of Shakespeare, encoded by letter values in some suitable way? Or with a googolplex digits all of which can be doubled without carrying (as Daniel wanted)?

I claim that the answers are yes, yes, and yes! How do we prove this?

The key fact we’ll use is simply that log102 is an irrational number. (If you don’t remember the proof: suppose log102 = a/b. Then 10a/b = 2, so 10a=2b. But this has no integer solutions other than a=b=0.)

Suppose we want k as a prefix, where 10d-1 ≤ k ≤ 10d. Then we want integers n,r such that

$$ k 10^r \\le 2^n \\le (k+1) 10^r, $$

i.e. taking the base-10 log,

$$ \\log\_{10}k + r \\le n \\log\_{10}2 \\le \\log\_{10}(k+1) + r. $$

In other words, the *fractional part* of n log102 needs to lie between the fractional part of log10(k), and the fractional part of log10(k+1) (where again, k is given).

But now we can appeal to the following **Key Fact:** if α is any irrational number, then the set

{the fractional part of nα : n∈N}

is dense in the interval \[0,1\]. Or equivalently, if I rotate around and around the unit circle by 2απ radians each time, then if α is irrational, I’ll eventually get arbitrarily close to any given point on the unit circle.

Why is the key fact true? Just the pigeonhole principle! Clearly, for any ε>0, the fact that α is irrational means that there must be two points, xα and yα, whose fractional parts are distinct yet closer together than ε. But then, by adding multiples of (x-y)α, we can get our fractional part to be ε-close to *anything* in \[0,1\].

And to sum up, this is why there must be a power of 2 whose decimal representation starts with the complete works of Shakespeare, or with a googolplex carry-free digits! (Indeed, from the above discussion, we could even extract an efficient algorithm for constructing that power of 2.)

---

So much for the *first* decimal digits of 2n. Now let’s look at 2n‘s *last* decimal digits!

Here there are some complications, arising from the twin facts that

(a) 10 is composite, and

(b) 2 is one of its factors.

But we can deal with those complications!

For starters: what are the possible last decimal digits of 2n?

1, 2, 4, 8, 6, 2, 4, 8, 6, …

So, there’s an initial 1, but then we cycle forever through the even nonzero digits.

What about the last *two* digits of 2n? If you’ve never tried this before, it’s instructive to work it out:

01, 02, 04, 08, 16, 32, 64, 28, 56, 12, 24, 48, 96, 92, 84, 68, 36, 72, 44, 88, 76, 52, 04, …

So, there’s an initial 01 and 02, but after that, we cycle forever through 20=4×5 possibilities, namely all the possible multiples of 4 whose last digits are nonzero.

You can check that the general pattern is: the last k decimal digits of 2n have an initial segment that looks like 001, 002, 004, 008, …, 2k-1. And then there’s an eternal cycle of length 4×5k-1, where the last digit can be any of 2,4,6,8, while every other digit can *either* be any possible even digit *or* any possible odd digit, depending on the digits to its right—in (if you want to say this a fancier way) a recursively defined embedding of the powers of 2 mod 5k into the cyclic group Z/10k. So, there’s an initial “runup” as you fill out all the needed powers of 2, but then once that’s done, you just cycle around forever in an embedding of Z/5k into Z/10k, because

(a) 2 happens to be a primitive element of Z/5k for any k, and

(b) 5 divides 10.

So in particular, and relevant to Daniel’s conjecture: there exists a power of 2 whose *last* googolplex digits can all be doubled without carrying. Why? For the last digit, you can pick 2 or 4. Then, for the last googolplex digits but one, you can pick 1 or 3 for those constrained to be odd, and 0, 2, or 4 for those constrained to be even. Lots of choices that work!

---

So, we can avoid carries in the leftmost digits of 2n, we can avoid carries in the rightmost digits … so that “merely” leaves all the digits in the middle, where who the hell knows! Empirically, the digits seem to pass every standard randomness test that you can throw at them. So in particular, e.g., the fraction of the digits that are in {0,1,2,3,4} seems to converge inexorably towards 50%, so that it’s extremely plausible to conjecture that the fraction is less than 49% or more than 51% for only finitely many values of n. But of course, that’s presumably even *harder* to prove than Daniel’s original conjecture.

---

OK, last topic. Suppose we want to program a computer to check Daniel’s conjecture up to 2n, for some huge n. What algorithm will do that most efficiently? A naive algorithm would just calculate 1, 2, 4, …, 2n and check all the digits of all of them. That takes O(n2) time, since each 2k, for k=0,1,…,n, has ~klog102 = O(k) decimal digits.

We can improve this to roughly O(n) time, by simply noticing that we only need to check O(1) digits per power of 2 *in expectation*, presumably, until we find the first digit that requires carrying. Then we don’t even need to *compute* the remaining digits: we can simply move on to the next power of 2. (This sort of trick is used all over the place in the design of fast algorithms.)

But when I posted about Daniel’s problem on Facebook, my friend [Greg Kuperberg](https://www.math.ucdavis.edu/~greg/) (who’s a mathematician at UC Davis) noticed that further improvements are possible. To wit: 8×6 = 48, which again ends in 8. So, 8×16 ends in 8, as does 8×16k for every k≥0. Meaning: no 2n where n=3+4k can possibly be a counterexample to Daniel’s conjecture. They’re all ruled out!

Likewise, 64×1,048,576 ends in 64, so no 2n of the form n=6+20k can be a counterexample. They’re all ruled out as well.

We can keep going this way, filling out the “search tree” of potential counterexamples to Daniel’s conjecture via breadth-first search. At the root of the search tree, we try all possibilities for the last digit. One level deeper, we try all possibilities for the second-to-last digit, and so on. As we go, we prune subtrees according to constraints like the ones above that we keep discovering and adding.

When I worked this out, I got an algorithm for checking Daniel’s conjecture up to 2n, which under reasonable assumptions takes time only O(nα), where α = 1-log52 ≈ 0.569, and space only polylog(n).

[Paul Crowley](https://www.linkedin.com/in/ciphergoth/) (who’s my Facebook friend) then actually implemented this algorithm, and he tells me that he used it to check that Daniel’s conjecture holds all the way up to $$2^{10^{21}}$$ (!!), using 40 minutes on a 128-core machine.

So, to return at last to the first thing I told Daniel: yes, I think his conjecture is almost certainly true, even though I have no idea when, if ever, the human race or its successors will have a proof.
