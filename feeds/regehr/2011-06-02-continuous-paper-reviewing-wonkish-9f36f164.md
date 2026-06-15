---
title: Continuous Paper Reviewing (Wonkish)
url: https://blog.regehr.org/archives/538
published: "2011-06-02T04:33:42Z"
feed: regehr
guid: http://blog.regehr.org/?p=538
---

# Continuous Paper Reviewing (Wonkish)

People running conferences often have a single round of reviewing: papers are assigned to program committee members, reviews are due some weeks later, and then after reviewing is finished everyone decides which papers to accept. This works but there’s room for improvement. First, not all papers need the same number of reviews. Second, the number of reviews that each person has to produce may be large (like 20+ papers), causing problems for those of us who have poor time management skills.

Other conferences review papers in two rounds. The first round might assign three reviewers to each paper, the second round two or three more. This not only spreads out the work over more time, but also enables an optimization where papers that are obviously going to be rejected do not need to make it to the second round.

Since reviewing is typically managed electronically using a system like [HotCRP](http://www.cs.ucla.edu/~kohler/hotcrp/), it should be straightforward to spread reviews out over more rounds, or even make it into a continuous process. My idea is that it would be nice to add a tiny bit of statistical significance to the [all-too-random paper acceptance process](http://www.cs.washington.edu/homes/tom/support/confreview.pdf). As reviews arrive, the system — which knows the total number of papers and can be told the likely number of papers that will be accepted — can start to compute confidence intervals for each paper’s “true” review score. The system would ask for additional reviews for a paper until either some maximum number of reviews is reached (six, perhaps) or it can be established with (say) 90% confidence that the paper will be accepted or rejected using fewer than the maximum number of reviews. Statistics about the distribution of review scores should be easy to dig up from previous conferences. In addition to being statistically superior (I hope) and spreading out the work, this scheme would be more tolerant of reviewers who fall behind.

The most obvious problem seems to be that reviewers won’t have hard deadlines. This can perhaps be solved by asking for some number of reviews per week and relentlessly harassing reviewers who fall behind. On that note I’m not sure why I’m writing a blog post right now since I am behind on some paper reviews.
