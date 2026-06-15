---
title: Avoidable Failures of Peer Review
url: https://blog.regehr.org/archives/675
published: "2012-01-30T17:42:47Z"
feed: regehr
guid: http://blog.regehr.org/?p=675
---

# Avoidable Failures of Peer Review

![](https://blog.regehr.org/wp-content/uploads/2012/01/circular_firing_squad-1-1.jpg)

This piece is about a specific kind of peer review failure where a paper is rejected despite there being sufficient evidence to warrant acceptance. In other words, all the facts are available but the wrong decision gets made anyway. In my experience this is extremely common at selective computer science conferences. The idea here is to identify and discuss some of the causes for this kind of peer review failure, in order to make these easier to identify and fight.

# Not in the Current Problem Set

In most academic communities, most of the researchers are attacking a fairly small set of problems at any given time. Moving as a herd is good because it increases the problem-solving power of a research community as people compete and build upon each others’ solutions. Drawbacks of this style of research are that the research topic may remain alive well beyond the point of diminishing returns and also the area can suffer mission drift into irrelevant subtopics.

Hot topics can also crowd out other work. At best, papers not on a hot topic are implicitly held to a higher standard because the reviewers must perform a more significant context switch in order to understand the nuances of the problem being solved. Or, to flip that around, the authors are unable to exploit the fact that everyone already considers the problem to be worth attacking and understands not only its structure but also the state of its major solutions.

A pernicious variant of “not in the current problem set” occurs when the paper being reviewed addresses a problem that the community views as solved. It should not come as a shock if I say that we see some non-standard definitions of “solved” in academia. One of my most jaw-dropping moments in the research world (and this is saying a lot) happened at an NSF workshop some years ago where a well-known formal verification researcher got up in front of a group of industrial people, government people, and academics and said more or less: “We have solved all of the key problems. Why aren’t you people adopting our work?” As another example, a researcher working on verified compilation has told me horror stories about reviews that attempt to invalidate his work by denying the existence of compiler bugs.

Solutions:

- Conferences should actively pursue eclectic work. This is easier said than done, but I often feel that ASPLOS is reasonably eclectic and USENIX ATC used to be.
- Reviewers need to be sensitive to the “we solved that problem” phenomenon. Of course, in some fraction of cases this reaction is justified — but not always.

# Vocal Detractor

Sometimes the reviewers pretty much like a paper — perhaps it even has one or two strong supporters — but one person just wants to kill it. If this person is loud, determined, and forceful this tactic will usually succeed. A frequent vocal detractor will subvert the peer review process. Furthermore, one has to suspect that these people are sometimes politically or ideologically motivated.

Solutions:

- The committee as a whole (and particularly the chair) needs to create an environment in which abusive behavior is not tolerated. Frequent vocal detractors should not be invited to be on conference committees.
- Electronic meetings probably make it more difficult to be a vocal detractor since force of personality is blunted by the medium. On the other hand, electronic meetings tend to fragment the discussion, making systematic use of this tactic harder to detect.

# Conservatism

Ground-breaking research requires boldness and risk-taking, at least in some cases. The NSF explicitly wants research to be “transformative” as opposed to incremental. It follows that just a bit of boldness and risk-taking is also required on the part of the program committees. But too often, we see exactly the opposite: professors display a huge amount of conservatism in the peer review process. Nobody wants to accept a paper that might be embarrassing or wrong, nor do they want to accept a paper where the authors have displayed a less-than-masterful command of the related work, nor do they want to accept a paper where the evaluation section uses a geometric mean instead of arithmetic, or fails to explain Figure 17 in sufficient detail.

A common rhetorical tactic seen at program committee meetings is for someone to say: “Of course we can’t accept this paper, since two reviewers gave negative recommendations.” This happens so frequently that I have a few stock responses:

- On the contrary — we can do whatever we like with this paper.
- Can you tell me again why we all flew out here? To not discuss this paper?
- Might we consider the possibility that some of these reviewers are inexpert, biased, or wrong?

Although conservatism can come from an individual, just as often it emerges out of group dynamics. Someone makes an offhand remark during the discussion of a paper and suddenly the conversation enters a death spiral from which it is impossible to reach a positive consensus.

Conservatism is rooted in the fact that so much of academia is about status. When playing status games, we fear above all losing face. Conservative decisions minimize the risk of looking like idiots, but of course they also minimize the risk of learning something new.

Would it really be that bad to risk publishing some wrong results, if this kind of risk-taking had other benefits such as accepting a larger amount of interesting new work? I don’t think so. Informal post-publication peer review should be able to deal with these problems — and it’s not like conservative acceptance can catch all instances of bad research.

A sub-problem of conservatism is that papers that fail to conform get rejected. In computer systems, paper submissions from non-PhDs in industry are not uncommon. Usually these papers doesn’t quite look right and they often get rejected. In a number of cases I’ve seen reviews so blisteringly ugly that nobody in their right mind who received them would ever submit another paper to that community.

Solutions:

- Accept more papers. At a 10% acceptance rate, conservatism dominates all other considerations since only papers that look perfect will be accepted. At a 40% acceptance rate this is much less likely. Given the trend away from paper conference proceedings, there is little downside.
- Give each program committee member a limited number of whiteballs: unilateral acceptances for papers that have merit, but that are dying in committee. The [OOPSLA 2010 frontmatter](http://portalparts.acm.org/1870000/1869459/fm/frontmatter.pdf) has interesting thoughts on this.

# Conclusion

Avoidable rejections have two main root causes: human nature and the artificial scarcity of accepted papers at prestigious conferences and journals. Human nature can’t be fixed but it can be worked around, for example using the “unilateral accept” mechanism. Artificial scarcity can be fixed.

I wrote up this piece for two reasons. First, in the last couple of years I have found myself on the losing side of many arguments that a paper should be accepted to a good conference; I’m trying to better understand why this happens. Second, I’m co-chairing an embedded software conference this year and need to figure out how I can keep my program committee from rejecting too many papers that could otherwise be accepted.
