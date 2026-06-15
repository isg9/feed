---
title: Grading on a Curve
url: https://blog.regehr.org/archives/332
published: "2011-01-05T23:43:14Z"
feed: regehr
guid: http://blog.regehr.org/?p=332
---

# Grading on a Curve

Although university professors aren’t trained as teachers, many parts of teaching — lecturing, answering questions, creating tests — come naturally to most of us. Assigning grades, on the other hand, is a part of the game that I’ve never felt very comfortable about. Part of the problem is that as a student I found grades to be kind of irrelevant, so it’s disconcerting to realize how important grades are for many students.

It’s easy to just use a standard 90/80/70/60 scale but this often feels unsatisfactory. One problem is that the desirable grades are compressed in the top 17% of the range of scores: the statistical significance of the median A- vs. the median B+ seems seriously questionable, for example. Grades aren’t required to be statistically significant, but certainly we should avoid grading practices that are especially sensitive to noise.

A different grading approach is to decide beforehand that a fixed fraction of the class will get As, another fraction will get Bs, etc. While this doesn’t reduce the fundamental noisiness in students’ scores, it enables me to use the entire range of scores and it avoids the (I would argue artificial) coupling between students’ scores and specific grades. However, students really hate this approach: they feel much more comfortable being judged against an objective standard (nevermind that this doesn’t really exist). Also, if students feel they are competing with each other, they’re less likely to help each other out; that is undesirable since ideally, they learn as much or more from each other as they do from me. Finally, I’ve seen instances where a small class is abnormally strong or weak: in these cases it makes less sense to apply a bell curve.

My sense is that most professors mix these approaches, for example by adjusting the A/B/C/D scale to something other than 90/80/70/60. I’ve talked to several people who make a scatter plot of scores and then put the grade boundaries between clusters of students. This approach has a certain logic to it, but one of my colleagues argues that it’s statistical nonsense. When pressed, he failed to describe a convincing model of student behavior under which one could argue about the relative amounts of signal and noise in grades, but my feeling is that his intuition is sound. An even worse curving practice is “curve to highest” where everyone is graded against the highest achieved score instead of the maximum possible score. This is bad because the highest score is by definition an outlier — we should try to minimize the effect of outliers, not amplify it.

Over the years I’ve evolved a pretty simple approach to assigning grades in “real classes” (as opposed to graduate seminars and such): I look at the grades that would be assigned using a 90/80/70/60 scale and decide — based on what I know from watching students work, talking to them, looking at their code, etc. — if the grades are appropriate or if they are too low. In the latter case, I curve students scores by giving then back a fixed fraction of points they missed. For example, I may give back 1 point for every 3 points missed, turning a 70% in the course into an 80%, a 0% into a 33%, etc. This is simple and has a pleasing rationale: my assumption is that some missed points are because the student failed to study or whatever, but other missed points are because I worded a question poorly, bungled a lecture, or similar. This approach also doesn’t compress scores near the top very much. In principle, if the 90/80/70/60 scale leads to scores that are too high, I should apply a reverse curve. In practice, this would piss off the students too much, and anyway I’m not sure that I’ve seen it happen.

I’d be interested to hear others’ rationales for assigning grades in a particular way. Probably this discussion is very US-specific — due to working on graduate admissions I’m well aware that grading conventions differ considerably around the world.
