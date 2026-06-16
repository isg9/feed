---
title: Michael Rabin Passed Away on April 14, 2026, at the age of 94
url: https://blog.computationalcomplexity.org/2026/04/michael-rabin-passed-away-on-april-14.html
published: "2026-04-22T12:55:00Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-5742718346164240539
---

# Michael Rabin Passed Away on April 14, 2026, at the age of 94

Michael Rabin passed away on April 14, 2026 at the age of 94.  (Scott Aaronson has also blogged about his passing, see  [here](https://scottaaronson.blog/?p=9704).)

I had many points to make about him; however, the first one got so long that I will just do that one for today's blog post.

Rabin is an extremely well-rounded \\(\\ldots\\) computer scientist? Computer scientist seems too narrow, and the point of this point is that he was well rounded. So I will start this thought again.

The following is an extremely important question that permeates computer science, mathematics, and I am sure other fields:

Given a problem, how hard is it?

Note that this is a rather broad problem since the terms *problem* and *hard* are very broad.  And by hard I mean upper bounds and lower bounds.

Rabin had worked on this problem in many different domains. I list them in roughly the order of hardness, starting from the hardest.

1) Recursive Algebra: Mathematicians had proven that every field has an algebraic closure (an extension that is algebraically closed).  In 1960 Rabin asked and answered affirmatively: Does every computable field (the elements of the field are computable, \\(\\times\\), \\(+\\), are computable) have a computable algebraic closure. This was an early result in what was later to be Nerode's Recursive Math Program which later became a subcase of the  Friedman/Simpson Reverse Math Program.

2) The Decidability of S2S: In 1969  Rabin proved that the the second order theory of 2 successors (S2S) is decidable. In a course I had with him he taught the proof that the weak second order theory of 1 successor (WS1S) is decidable. I teach that when I teach Automata Theory since it brings together Finite Automata and Decidability.  Here are my slides: [here](https://www.cs.umd.edu/~gasarch/BLOGPAPERS/ws1stalk.pdf)

S2S is one of the only decidable theories where one can actually state theorems in math of interest in them.  (I may blog about that some other time.)

S2S is the decidable theory with the hardest proof of its decidability. Rabin's proof used transfinite induction, though later proofs did not.

*Personal Note:* I was the subreferee on a paper by Gurevich and Harrington that simplified the proof tremendously, back in the early 1980's. Their proof is the one to read now.  Rabin was happy that the proof was simplified. The proof is still hard, just not as hard.

3) In 1974  Fisher & Rabin showed that any algorithm to decide Presburger arithmetic required time \\(\\ge 2^{2^{cn}}\\) for some constant \\(c\\). I asked chatty if better is known and it gave me answers that didn't make sense. An earlier version of this paragraph said so and had some incorrect statements in it. One of the commenters told me what is TRUE which made it easier to look it up. Anyway, there is a triple-exp algorithm, and there is a complexity class that Presburger is complete for---see the comment. Spellcheck thinks Presburger is spelled incorrectly. I can understand why it thinks so, but its wrong.

When Rabin taught this result he pointed out that Hilbert and others not only thought that math was decidable but also that perhaps that algorithm could be used to really do math. The complexity results show that even if a theory is decidable it may still be really hard. (With AI maybe we can use computers to do math, but that is way too big a topic, and too big a tangent, to get into within a blog-obit).

4) In 1972 Rabin proved the following. Given linear forms \\(L\_1(x),\\ldots,L\_m(x)\\) over the reals (so the intent is \\(x\\in R^n \\)) we want to know if there exists \\(x\\in R^n \\) such that, for all \\(1\\le i\\le m \\), \\(L\_i(x)>0 \\).  The model of computation is a decision tree where each internal node can ask a question of the form \\(L(x) {\\rm\\  RELATION\\  } 0 \\) where RELATION can be any of \\(<,=,>\\). Each leaf is labelled YES or NO.  Then the depth of the tree is \\(\\ge 2^{\\Omega(n)} \\).  This is an early paper in decision tree complexity.

5) In 1977 the Handbook of Math Logic came out. Rabin did the article on Decidable theories. He mentioned P vs NP as being really important and being the next logical (no pun intended) direction for logic. He was the only author to mention P vs NP.

6) While Rabin did not invent randomized algorithms he worked on them a lot early on.  In 1977 Solovay and Strassen obtained a polytime randomized algorithm for primality.  In 1976 Rabin noticed that Miller's Primality algorithm (which showed that if the Extended Riemann Hypothesis is true then primality is in P) could be modified to be a randomized polynomial time algorithm for primality. While preparing this blog post I noticed that I often hear about the Miller-Rabin algorithm (many cryptography protocols need a primality algorithm) but I hardly ever hear about the Solovay-Strassen algorithm. I asked Google AI why this was. In brief: Miller-Rabin is faster, has a lower probability of error, and is simpler. Is Google AI correct? I think yes since Miller-Rabin is used and Solovay-Strassen is not. I might be employing circular reasoning here.

The Miller-Rabin primality test might be his second best known work, the best known being the last item on this list, the result of Rabin and Scott that NFA's are equivalent to DFA's.  (It's last since it is the lowest complexity class he worked on.)

7) Rabin obtained other randomized algorithms. I mention two:

a) Rabin-Shallit: When teaching automata theory I often give my students the following sets in NP and ask them to determine which ones are known to be in P:

\\( \\{ x \\in N \\colon (\\exists y)\[x=y^2\] \\} \\)

\\( \\{ x \\in N \\colon (\\exists y\_1,y\_2)\[x=y\_1^2+ y\_2^2\] \\} \\)

\\( \\{ x \\in N \\colon (\\exists y\_1,y\_2,y\_3)\[x=y\_1^2+ y\_2^2+ y\_3^2\] \\} \\)

\\( \\{ x \\in N \\colon (\\exists y\_1,y\_2,y\_3,y\_4)\[x=y\_1^2+ y\_2^2+ y\_3^2+y\_4^2\] \\} \\)

\\( \\{ x \\in N \\colon (\\exists y\_1,y\_2,y\_3,y\_4,y\_5)\[x=y\_1^2+ y\_2^2+ y\_3^2+y\_4^2+y\_5^2\] \\} \\)

etc.

The input is in binary so searching for all possible \\(y\_1,y\_2,y\_3,y\_4,\\ldots,y\_n\\) is exponential in the length of the input.

I leave the first three to the reader.

This one:

\\( \\{ x \\in N \\colon (\\exists y\_1,y\_2,y\_3,y\_4)\[x=y\_1^2+ y\_2^2+ y\_3^2+y\_4^2\] \\} \\)

is sort-of a trick question. Every number is the sum of 4 squares, so this set, and all later ones, are trivially in P. But that raises the question: how to find \\( y\_1,y\_2,y\_3,y\_4 \\)? This is a curious case of a problem that is in NP, the decision part is in P (in fact, trivial),  but finding the witness seems hard.

When I first assigned this problem I then looked up what was known about finding the \\(y\_1,y\_2,y\_3,y\_4\\).

In 1986 Rabin and Shallit showed that, assuming ERH, there is a randomized \\( O(\\log^2 n) \\) algorithm. I am surprised that you need both ERH and Randomness. This seems to be a less-well known result though I don't know why since it's a natural question.

b) Karp-Rabin: In 1987 Karp and Rabin obtained a really fast, really simple (that's a plus in the computer science world)  randomized pattern matching algorithm.  Since it is fast and simple I wondered if it is really used. It is! To quote Google AI:

*Yes, the Karp-Rabin algorithm is used in the real world, particularly in scenarios requiring the detection of multiple patterns simultaneously, such as plagiarism detection,data deduplication, and bioinformatics*.

Is Google AI correct? I leave that as an exercise for the reader.

8) In 1979 Rabin devised a cryptosystem whose security is equivalent to factoring. How come RSA became the standard and not Rabin's system? Broadly two possibilities

a) RSA was better.

b) RSA was faster to the marketplace and other random factors.

Which is it? I leave that to the reader.

9) In 1958 Rabin and Scott showed that NFAs are equivalent to DFAs. This may be his best known work.
