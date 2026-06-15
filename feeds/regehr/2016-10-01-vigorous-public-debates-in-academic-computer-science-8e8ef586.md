---
title: Vigorous Public Debates in Academic Computer Science
url: https://blog.regehr.org/archives/1430
published: "2016-10-01T23:18:14Z"
feed: regehr
guid: http://blog.regehr.org/?p=1430
---

# Vigorous Public Debates in Academic Computer Science

The other day a non-CS friend remarked to me that since computer science is a quantitative, technical discipline, most issues probably have an obvious objective truth. Of course this is not at all the case, and it is not uncommon to find major disagreements even when all parties are apparently reasonable and acting in good faith. Sometimes these disagreements spill over into the public space.

The purpose of this post is to list a collection of public debates in academic computer science where there is genuine and heartfelt disagreement among intelligent and accomplished researchers. I sometimes assign these as reading in class: they are a valuable resource for a couple of reasons. First, they show an important part of science that often gets swept under the rug. Second, they put discussions out into the open where they are widely accessible. In contrast, I’ve heard of papers that are known to be worthless by all of the experts in the area, but only privately — and this private knowledge is of no help to outsiders who might be led astray by the bad research. For whatever reasons (see [this tweet](https://twitter.com/moyix/status/780908820407128065) by Brendan Dolan-Gavitt) the culture in CS does not seem to encourage retracting papers.

- N-version programming is a software development method where several implementations of a specification are run in parallel and voting is used to determine the correct result. [Knight and Leveson wrote a paper](http://sunnyday.mit.edu/papers/nver-tse.pdf) showing that the assumption of independent faults in independent implementations may not be a good one. This finding did not sit well with the proponents of n-version programming and while I cannot find online copies of their rebuttals, Knight and Leveson’s [reply to the criticisms](http://sunnyday.mit.edu/critics.pdf) includes plenty of quotes. This is great reading, a classic of the genre.

- Starting in the late 1980s, Ken Birman’s group was advocating [causal and totally ordered multicast](http://www.cs.cornell.edu/courses/cs614/2003sp/papers/BSS91.pdf). Cheriton and Skeen were less than impressed and [wrote 15 pages to that effect](https://www.cs.rice.edu/~alc/comp520/papers/Cheriton_Skeen.pdf). Then we have [Birman’s 32-page response to the criticisms](http://www.cs.princeton.edu/courses/archive/fall07/cos518/papers/catocs-limits-response.pdf). Also see [Neha Narula’s take on the debate](http://dsrg.pdos.csail.mit.edu/2013/06/13/cheriton-and-skeen/).

- [This 1991 paper](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.104.1363&rep=rep1&type=pdf) introduced log-based filesystems. In 1993 [Seltzer et al. published this paper](https://www.usenix.org/legacy/publications/library/proceedings/sd93/seltzer.pdf) describing and evaluating an implementation of a log-based filesystem, and [this followup in 1995](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/07/usenix-winter95.pdf). John Ousterhout, one of the authors of the original paper, [disagreed with the evaluation](http://www.eecs.harvard.edu/~margo/papers/usenix95-lfs/supplement/ouster_critique1.html). Seltzer and her coauthors [rebutted his critique](http://www.eecs.harvard.edu/~margo/papers/usenix95-lfs/supplement/rebuttal.html), and Ousterhout had, as far as I know, [the last word](http://www.eecs.harvard.edu/~margo/papers/usenix95-lfs/supplement/ouster_critique2.html).

- [Linus v Tanenbaum](http://www.oreilly.com/openbook/opensources/book/appa.html) on the merits of microkernels is another classic. Also see some (one-sided) [comments on a reincarnation of the debate here](http://www.cs.vu.nl/~ast/reliable-os/). Related, Hand et al. published a paper [Are Virtual Machine Monitors Microkernels Done Right?](https://www.usenix.org/legacy/event/hotos05/final_papers/full_papers/hand/hand.pdf); in response, Heiser et al. wrote a paper [with the same title](http://cgi.di.uoa.gr/~mema/courses/mde518/papers/heiser.pdf) but coming to the other conclusion.

- [Social Processes and Proofs of Theorems and Programs](http://www.yodaiken.com/papers/p271-de_millo.pdf) is a provocative opinion piece about the role of formal methods in software development. Dijkstra called it “a very obnoxious paper” ( [see page 14 of this transcript](http://conservancy.umn.edu/bitstream/handle/11299/107247/oh330ewd.pdf)) and wrote a response called “ [On a Political Pamphlet from the Middle Ages](http://www.cs.utexas.edu/users/EWD/transcriptions/EWD06xx/EWD638.html).” [DeMillo et al. replied](http://dl.acm.org/citation.cfm?id=1005891&CFID=675484827&CFTOKEN=97157491): “We must begin by refusing to concede that our confidence in a piece of real software has ever been increased by a proof of its correctness…” See the [letters to the editor of CACM](http://research.microsoft.com/en-us/um/people/lamport/pubs/letter-to-editor.pdf) responding to this article, [Victor Yodaiken’s take on this debate](http://www.yodaiken.com/2008/11/10/dijkstra-versus-perlis/), and also [three more shots fired in 2010](http://cacm.acm.org/magazines/2010/1/55739-more-debate-please/fulltext), two by Moshe Vardi and one by the original paper’s authors.

- Sticking with Dijkstra, he and John Backus had a (only partially public) spat, [nicely written up here](https://medium.com/@acidflask/this-guys-arrogance-takes-your-breath-away-5b903624ca5f).

I’d like to fill any holes in this list, please leave a comment if you know of a debate that I’ve left out!

Here are some more debates pointed out by readers:

- [Software-based attestation](http://www.netsec.ethz.ch/publications/papers/swatt.pdf) offers a protocol for checking that a remote system contains the memory image that we expect it to have. [On the Difficulty of Software-Based Attestation of Embedded Devices](https://pdfs.semanticscholar.org/fc14/909505a02a484811ff70ccb326905f352d0a.pdf) presents concrete attacks on SWATT. Perrig and van Doorn [did not agree that the attacks were valid](http://www.netsec.ethz.ch/publications/papers/perrig-ccs-refutation.pdf) and, finally, [Francillon et al. responded to the refutation](https://pdfs.semanticscholar.org/657a/7b4270581c655763df0f5a1ddb79cb7cb946.pdf).

- See the debate about code pointer integrity, links in [comment #1](https://blog.regehr.org/archives/1430#comment-18897)
- See the debate about automated program repair, links in [comment #2](https://blog.regehr.org/archives/1430#comment-18898)
- See the debate about the Java memory model, links in [comment #4](https://blog.regehr.org/archives/1430#comment-18905)
- [Go-to Statement Considered Harmful](https://www.cs.utexas.edu/users/EWD/ewd02xx/EWD215.PDF),

  [Goto Considered Harmful Considered Harmful](http://web.archive.org/web/20090320002214/http://www.ecn.purdue.edu/ParaMount/papers/rubin87goto.pdf), and

  [Goto Considered Harmful Considered Harmful Considered Harmful](http://dl.acm.org/citation.cfm?doid=22899.315729) (several different letters in this PDF, and sorry about the paywall)
