---
title: 'Amazing: Erdős’ Unit Distance Problem was Disproved! It was achieved by AI!'
url: https://gilkalai.wordpress.com/2026/05/21/amazing-erdos-unit-distance-problem-was-disproved-it-was-achieved-by-ai/
published: "2026-05-20T23:01:57Z"
feed: gilkalai
guid: http://gilkalai.wordpress.com/?p=31739
---

# Amazing: Erdős’ Unit Distance Problem was Disproved! It was achieved by AI!

[![](https://gilkalai.wordpress.com/wp-content/uploads/2026/05/openai-udp.png)](https://openai.com/index/model-disproves-discrete-geometry-conjecture/)

Paul Erdős’s, in his 1946  paper published in the American Mathematical Monthly, posed two general questions about the distribution of distances determined by a finite set of points in a metric space.

1\. **Unit Distance Problem:** At most how many times can the same distance (say, distance 1) occur among a set of n points?

2\. **Distinct Distances Problem:** What is the minimum number of distinct distances determined by a set of n points?

Erdős conjectured that in the plane the number of unit distances determined by n points is at most ![n^{1+c/loglog n}](https://s0.wp.com/latex.php?latex=n%5E%7B1%2Bc%2Floglog+n%7D&bg=ffffff&fg=333333&s=0&c=20201002), for a positive constant c, but the best known upper bound, due to Spencer, Szemeredi, and Trotter is only ![O(n^{4/3})](https://s0.wp.com/latex.php?latex=O%28n%5E%7B4%2F3%7D%29&bg=ffffff&fg=333333&s=0&c=20201002).

As for the Distinct Distances Problem, the order of magnitude of the conjectured minimum is ![n/\sqrt{log n}](https://s0.wp.com/latex.php?latex=n%2F%5Csqrt%7Blog+n%7D&bg=ffffff&fg=333333&s=0&c=20201002).  In 2010 a sensational paper of Guth and Katz presents a proof of an almost tight lower bound of the order of ![n/log n.](https://s0.wp.com/latex.php?latex=n%2Flog+n.&bg=ffffff&fg=333333&s=0&c=20201002) Janos Pach wrote about it [in this 2010 post](https://gilkalai.wordpress.com/2010/11/20/janos-pach-guth-and-katzs-solution-of-erdos-distinct-distances-problem/). See also [this post](https://gilkalai.wordpress.com/2008/07/17/extermal-combinatorics-ii-some-geometry-and-number-theory/) from 2008, that explains (among other things) the proof for the upper bound.

I have just learned that an internal model of OpenAI have very recently disproved Erdős’ conjecture for the unit distance problem and found an example with more than ![n^{1+\epsilon}](https://s0.wp.com/latex.php?latex=n%5E%7B1%2B%5Cepsilon%7D&bg=ffffff&fg=333333&s=0&c=20201002) unit distances. This is truly amazing. The construction relies on algebraic number theory.

Here is [Open AI announcement](https://openai.com/index/model-disproves-discrete-geometry-conjecture/) and [technical paper](https://cdn.openai.com/pdf/74c24085-19b0-4534-9c90-465b8e29ad73/unit-distance-proof.pdf).

The proof is presented, explained and discussed in the paper [Remarks on the disproof of the unit distance conjecture](https://cdn.openai.com/pdf/74c24085-19b0-4534-9c90-465b8e29ad73/unit-distance-remarks.pdf) by Alon, Bloom, Gowers, Litt, Sawin, Shankar,Tsimerman, Wang, and Matchett Wood. (I learned about it from [an answer](https://mathoverflow.net/a/511485/1532) to an MO question on the topic of mathematics and AI.)

Like the computer-based proof of the four color theorem in 1976 by Appel and Haken, this may well be a scientific landmark whose importance goes beyond combinatorics and beyond mathematics. I will add a link to the Open AI document when I will have it. (Done.)

There is also a short [Open AI video](https://www.youtube.com/watch?v=Br4l9YjCyRU) where the result is presented by Tim Gowers and by four members of the team, Lijie Chen, Mark Sellke, Mehtaab Sawhney, and Sebastien (Seb) Bubeck.

**Updates:** Will Sawin [proved a lower bound](https://arxiv.org/abs/2605.20579v1) of ![n^{1.014};](https://s0.wp.com/latex.php?latex=n%5E%7B1.014%7D%3B&bg=ffffff&fg=333333&s=0&c=20201002) This was further improved to ![n^{1.0318}](https://s0.wp.com/latex.php?latex=n%5E%7B1.0318%7D&bg=ffffff&fg=333333&s=0&c=20201002); There are interesting [comments on the ErdosProblems site;](https://www.erdosproblems.com/forum/thread/90) The list of best constants so far can be found in [Terence Tao’s Github](https://teorth.github.io/optimizationproblems/constants/84a.html). Sawin also proved in the [same paper](https://arxiv.org/abs/2605.20579v1) a limit 1.2143 for the new method.

The result is reported and discussed on [the Geomblog](http://blog.geomblog.org/2026/05/the-unit-distances-problem.html), [computational complexity](https://blog.computationalcomplexity.org/2026/05/two-erdos-problems-on-points-in-plane.html), and [Shtetl Optimized](https://scottaaronson.blog/?p=9782); There is an interesting [commentary by Erik Hoel](https://www.theintrinsicperspective.com/p/dumbo-could-already-fly). Following the OpenAI result, Anthropic reported being able to autonomously disprove the unit-distance conjecture (in its strongest form) with its system. Inspired by the disproof of Erdős’ unit distance conjecture, Thomas Bloom, Will Sawin, Carl Schildkraut, and Dmitrii Zhelezov [disproved](https://arxiv.org/abs/2605.28781) Erdős-Szemeredi Sum-Product Conjecture (over the reals). (See the [2008 post](https://gilkalai.wordpress.com/2008/07/17/extermal-combinatorics-ii-some-geometry-and-number-theory/) mentioned above for the statement and connection.) Cosmin Pohoata whote a [detailed beautiful post](https://pohoatza.wordpress.com/2026/06/01/another-one-bites-the-dust-the-elekes-ronyai-problem/) (on the sum-product breakthrough) his result inspired by these two breakthroughs: a disproof of the [Elekes–Rónyai problem](https://arxiv.org/abs/1601.06404).   Thomas Bloom wrote a detailed [beautiful blog post](https://www.erdosproblems.com/forum/thread/blog:6) about the developments. Here is [an interesting videotaped lecture of Noga Alon](https://youtu.be/KbNctTQnVHI?si=cfVkxMfOD1k7HJd8).

**Remarks (about the problem):** For an approach to improve the upper bound for the problem see the paper [Erdős’s unit distance problem and rigidity](https://arxiv.org/abs/2507.15679), by János Pach, Orit E. Raz, József Solymosi  (2025). We discussed a lecture by Orit Raz on the topic in [this post](https://gilkalai.wordpress.com/2025/12/21/novembers-lectures-2025/). Other posts (in “Combinatorics and more”) related to the Erdos distance problem and to related problems are listed in [this comment](https://gilkalai.wordpress.com/2026/05/21/amazing-erdos-unit-distance-problem-was-disproved-it-was-achieved-by-ai/#comment-100038).

**Remarks (About the AI+math):** This breakthrough adds to several others remarkable applications of AI to Math. See for example [this post](https://terrytao.wordpress.com/2026/05/03/primitive-sets-and-von-mangoldt-chains-erdos-problem-1196-and-beyond/) over Terry Tao’s blog on Erdős problem 1196 which was settled by Open AI model a few weeks ago. Very recently, a team at DeepMind  [used their model](https://arxiv.org/abs/2605.22763v1) to solve several open problems and their proofs come *with Lean verification*.  From ancient times (January 2026) let me mention an interesting blog post [Problem 728 and the use of AI on Erdős problems](https://www.erdosproblems.com/forum/thread/blog:2), by Kevin Baretto. (June 3, 2026) There is an interesting [Leiden Declaration on Artificial Intelligence and Mathematics.](https://leidendeclaration.ai/)

Closer to me personally: Ori Gurel-Gurevich, Asaf Nachmias, and Sushant Sachdeva used an internal model of OpenAI to settle a problem about [electrical flow in graphs](https://arxiv.org/abs/2605.24130); (I just met Asaf a few days ago in a special Day of Tel-Aviv University devoted to AI and Math.) Pablo Soberón used  LLM-based tools in the brainstorming stage of a recent truly remarkable paper: [Tverberg cores and Kalai’s cascade conjecture](https://arxiv.org/abs/2605.21305).
