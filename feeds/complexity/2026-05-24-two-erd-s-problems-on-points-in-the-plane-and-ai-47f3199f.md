---
title: Two Erdős Problems on Points in the Plane and AI
url: https://blog.computationalcomplexity.org/2026/05/two-erdos-problems-on-points-in-plane.html
published: "2026-05-24T12:24:43Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-2319231633623037559
---

# Two Erdős Problems on Points in the Plane and AI

In a 1946 paper in the American Mathematical Monthly, Paul Erdős posed the Erdős Distinct Distance Problem and the Erdős Unit Distance Problem.

\-\-\------------------------------------------------------------------

THE ERDŐS DISTINCT DISTANCE PROBLEM

A nice high school math competition problem, if it were not fairly well known, is:

*Show that for all sets of \\(n\\) points in the plane, there are at least \\(0.5\\sqrt{n}\\) distinct distances.*

A bit harder is to show that there exist \\(n\\) points in the plane where the number of distinct distances is \\(O(\\frac{n}{(\\log n)^{1/2}})\\). The points are the grid points of a \\(\\sqrt{n}\\times\\sqrt{n}\\) integer grid.

ADDED LATER:  The set of points is \\(\\{ (x,y) \\in Z\\times Z   \\colon    1\\le x,y\\le \\sqrt{n}   \\} \\)

Let \\(g(n)\\) be the minimum number such that, for every set of \\(n\\) points in the plane,

there are \\(g(n)\\) distinct distances. The above results (of Erdős) show that

\\( \\Omega(n^{1/2}) \\le g(n) \\le O(\\frac{n}{(\\log n)^{1/2}}) \\)

Erdős made no conjecture about \\(g(n)\\) in that 1946 paper. However, later papers attribute to him one of two conjectures:

1) \\( (\\forall \\epsilon)\[g(n)\\ge n^{1-\\epsilon}\]\\)

or

2) \\(g(n) = \\Omega(\\frac{n}{(\\log n)^{1/2}})\\)

Those papers incorrectly point to the 1946 paper for where he made those conjectures.  I suspect he made those conjectures later in talks or compilations of open problems.

This was a great problem in that there were many papers making progress on it, each one having new ideas. The current best result is that \\(g(n) \\ge \\Omega(\\frac{n}{\\log n})\\).  This result was obtained by Larry Guth and Netz Katz in 2011 and is, by far, the largest leap forward in progress on the problem.  Brass-Moser-Pach (2005) and later Pach-Sharir (2009) observed that existing techniques could never prove \\(g(n) \\ge \\Omega(n^{8/9+\\epsilon})\\).  I've also heard the observation credited to Rusza. Guth and Katz invented (discovered?) new techniques to obtain their breakthrough.

See my webpage of all papers on the problem [here](https://www.cs.umd.edu/~gasarch/TOPICS/erdos_dist/erdos_dist.html) or the Wikipedia page [here](https://en.wikipedia.org/wiki/Erd%C5%91s_distinct_distances_problem)

Takeaway: The Erdős Distinct Distance Problem saw decades of steady human progress and was eventually almost completely resolved *without computer assistance*.  I would not have emphasized *without computer assistance* one year ago.  Maybe not even one week ago.

\-\-\--------------------------------------------

THE ERDŐS UNIT DISTANCE PROBLEM

Let S be a set of \\(n\\) points in the plane. Some of the distances between points of S may be 1. How many?  What is the maximum number of unit distances that can occur? Denote this by \\(f(n)\\). Erdős showed

\\( n^{1+\\Omega(1/\\log\\log n)} \\le f(n) \\le O(n^{3/2}) \\).

The lower bound is obtained by using the grid points of a scaled version of the \\(\\sqrt{n}\\times\\sqrt{n}\\) integer grid. See a writeup by ChatGPT, [here](https://www.cs.umd.edu/~gasarch/BLOGPAPERS/unitdistance.pdf).  We note for later that this can be phrased as using the points in the Gaussian integers, \\(Z\[i\]\\).

In the 1946 paper Erdős conjectured that, for all \\(\\epsilon\\),  \\(f(n) = O(n^{1+\\epsilon})\\).

Joel Spencer, Endre Szemerédi, and William Trotter in 1984 showed \\(f(n) = O(n^{4/3})\\).

Aside from the Spencer-Szemerédi-Trotter bound, there has been relatively little progress on either the upper or lower bound.

\-\-\--------------------------------

CONTRASTING THE TWO PROBLEMS

*Contrast*: The Erdős Distinct Distance problem has seen extensive progress and a large literature. By contrast, although many papers were written on the Erdős unit distance problem, there was little improvement. That changed on May 20, 2026.

\-\-\----------------------------------------

OPENAI PRODUCES A COUNTEREXAMPLE TO THE ERDŐS CONJECTURE

OpenAI recently produced a counterexample to Erdős's conjecture.  More precisely, OpenAI proved the following:

There exists \\(\\epsilon>0 \\) and a sequence of point  sets \\(P\_i \\subseteq R^2\\) such that, letting \\(\|P\_i\|=n\_i\\):

a) \\(n\_i \\rightarrow\\infty\\), and

b) for all \\(i\\), the number of unit distances in \\(P\_i\\) is at least \\(n\_i^{1+\\epsilon}\\).

The  OpenAI announcement is [here](https://openai.com/index/model-disproves-discrete-geometry-conjecture/). The technical paper is [here](https://cdn.openai.com/pdf/74c24085-19b0-4534-9c90-465b8e29ad73/unit-distance-proof.pdf).

Note that the author is OpenAI. There are no human co-authors.  However,(1)  Lije Chen used an internal OpenAI model and (2) Mark Selke and Metaab Sawhney verified correctness. (For a comment on the lack of human authors, see Sebastien Bubeck's [comment below](https://blog.computationalcomplexity.org/2026/05/two-erdos-problems-on-points-in-plane.html?showComment=1779727243132#c6137786223904113449).) The \\(\\epsilon\\) is not given though one could go through the paper and find it. It would be very small.

After the initial AI-generated argument (or AI-assisted?) human mathematicians streamlined and clarified the proof. The result is a paper by Noga Alon, Thomas Bloom, W.T. Gowers, Daniel Litt, Will Sawin, Arul Shankar, Jacob Tsimermann, Victor Wang, and Melanie Matchett Wood  that explains the history, the proof, and their reactions to it. That paper is [here](http://arxiv.org/abs/2605.20695). They obtain \\(\\epsilon \\sim 6\\times 10^{-38}\\).

Recall that Erdős's lower bound was obtained by using points in \\(Z\[i\]\\). The new result used more complicated number fields. Indeed, the degree of the number fields used goes to infinity.  For more details see the Alon et al. proof.

Will Sawin wrote a paper where he obtained \\(\\epsilon=0.014114\\). Quoting his paper:

*To do this, we make explicit and sharpen every step of the argument. Interestingly, this rarely makes the argument much more complex. The total length of this paper is essentially the same as the original OpenAI writeup, though longer than the simplified version prepared by human authors \[Alon et a.\].*

His paper is [here](https://arxiv.org/pdf/2605.20579)

\-\-\-----------------------------------------

MY POINTS

1) The Erdős's Unit Distance problem is a well known and interesting conjecture, so this is not picking out some obscure problem.

2) AI-generated or AI-assisted? The announcement by OpenAI, and the Alon et al. paper, indicate that the proof is AI-generated.

Fellow blogger Gary Marcus disagrees and argues that it is AI-assisted.  See his post [here](https://garymarcus.substack.com/p/checking-the-math-behind-openai-and).

ADDED LATER: An interview with Gary Marcus is [here](https://www.youtube.com/watch?v=iFYF_e1GSGI).

I think this is a hard question, and perhaps there is more of a continuum.  However, I trust Alon et al., so I will go with AI-generated.

3) Humans were still needed to verify and  clean up the proof. In the future, we will all be grad students, verifying and cleaning up what AI outputs.

4) The final proof was readable. One concern about AI proofs is that they would be unreadable and hard to verify. That was not the case here.

5) The ideas needed for the solution already existed; however:

a) The right combination was hard to find.

b) The relevant techniques used, algebraic number theory, are not standard tools in combinatorial geometry.

c) It was widely believed that Erdős's conjecture was true.

d) Producing a counterexample seems less impressive than proving the theorem.  It would be of interest to see if OpenAI could prove the Guth-Katz result; however, that would not work now since Guth-Katz is already out there for OpenAI to see. Perhaps OpenAI should try to improve Guth-Katz.

6) In the short term, this result and what it portends, is good: math problems we care about will be resolved with the help of AI, perhaps solely by AI. But in the long run we may lose the ability---or at least the patience---to do the proofs ourselves. Note that I am assuming that AI will have future successes---see item 10-ONE for a counterthought.

7) AI seems to be good at combining known concepts. Is it good at coming up with new ones? Are humans? The distinction between combining known ideas and coming up with new ones is very thin.

8) Two contrary lessons:

a) You should know many fields of mathematics so that you can use ideas from one field in another, like the AI did.

b) You should know some field of math really well so you may do something new in it that current AI could not have done.

9) If an AI produces a new treatment for cancer that is cheaper and better than what is known I do not think we will care that an AI did it (though we will have humans check it). Is mathematics similar? Do we care that an AI did it?  Medicine primarily values outcomes; Mathematics also values understanding. Does reading a proof that AI did give the same understanding as coming up with it yourself?  Can AI help with that as well?

10) I suggest two futures:

ONE: While this AI-generated (or AI-assisted) result is impressive, it will be a rare occurrence. This result was actually a counterexample. The needed math was known. The result was interesting. This is a perfect storm that we might not see again for a while.

TWO: Even before the AI revolution, when I came up with a math problem I wanted solved, I would seek help, perhaps too early. My curiosity far exceeds my ego.  Since AI makes it easy to get help, my fear is that eventually we will all be Bill Gasarch---scary.
