---
title: Embrace WTF
url: https://blog.regehr.org/archives/540
published: "2011-06-07T14:36:26Z"
feed: regehr
guid: http://blog.regehr.org/?p=540
---

# Embrace WTF

Most people who do quantitative work, especially involving computers, mutter “What the fuck?” or something similar pretty often. Lately I’ve been thinking about WTF in more detail. WTF is good because it stems from dawning recognition of one’s own ignorance, and without recognizing ignorance we cannot eliminate it. Here I am only discussing serious WTF, not the kind of casual WTF that occurs when the mouse pointer gets lost on another screen or the elevator fails to arrive rapidly. WTF comes is several flavors.

## Neutral WTF

This is by far the most common and it’s best described by example. A few years ago I wanted to compute confidence intervals for low-probability events. For example, if I sample a bag of marbles 10,000 times and get 9,999 red marbles and one blue marble, then what is the 95% confidence interval for the proportion of blue marbles in the bag? After a bit of background reading I settled on the [Wilson interval](http://en.wikipedia.org/wiki/Binomial_proportion_confidence_interval#Wilson_score_interval) as meeting my needs and wrote a bit of Perl code implementing it. Since I’m professionally paranoid I also wrote a simple Monte Carlo simulation to check my confidence interval code: it simply repeated the marble sampling experiment many times and computed the number of times that actual probability was contained in the confidence interval. Obviously, by definition, this should happen 95% of the time — but it didn’t. The actual probability was contained in the confidence interval about 92% of the time. The most obvious possibility was a bug in my code but double and triple checking failed to turn up any problems, leading to a nice solid WTF. More exotic possibilities like a systematic flaw in Perl’s random number generator or FP-related numerical issues seemed remote. Eventually I did some more reading and found out that the phenomenon I was observing is well-known; I won’t attempt to explain it but [Interval Estimation for a Binomial Proportion](http://www.stat.duke.edu/~wjang/teaching/S05-293/handout/binomial.pdf) contains an excellent analysis.

Neutral WTF comes from the routine kind of ignorance that plagues all of us, because there is so much to know. We’d rather it didn’t happen, but it’s unavoidable when doing any remotely interesting work.

## Negative WTF

No need to say much about this one. There are many sub-species of negative WTF but for me it generally happens when some research idea is generating really nice-looking results. Eventually something stops making sense and we get WTF that is quickly followed by gnashing of teeth because there’s a major flaw somewhere. Worse, this sometimes happens after I’ve started telling people how great the idea is.

Negative WTF comes from a harmful kind of ignorance — the ignorance that we were doing something important wrong.

## Positive WTF

Positive WTF is elusive and excellent; it occurs when we begin to recognize that we don’t understand something, and finally discover that nobody else understands it either.

My personal favorite example of positive WTF happened four or so years ago after an undergrad pointed out that some assembly code in a lecture for my advanced embedded systems course was wrong. I tracked the error down to a compiler mis-translating a piece of C code containing the volatile qualifier. A while later I became curious about this and wrote a collection of test cases for accesses to volatile-qualified objects and ran it through a number of compilers. When none of the tools translated all of the examples correctly I experienced WTF. Could there really be systematic errors in translating a construct that essentially all operating systems and embedded systems rely on? I talked to people and investigated more and this turned out to be the case. The result was [a paper](http://www.cs.utah.edu/~regehr/papers/emsoft08-preprint.pdf) that is still heavily downloaded from companies who make and use compilers for embedded systems.

My guess is that a lot of great scientific discoveries were preceded by profound WTF. For example, it sounds like that is what happened to [Penzias and Wilson](http://en.wikipedia.org/wiki/Discovery_of_cosmic_microwave_background_radiation).

The broad power of positive WTF crystallized for me recently when I read The Big Short, a pretty awesome book about some people who not only foresaw the self-destruction of the sub-prime home loan market, but also placed big bets against the junk bonds created from these loans. Lewis makes a big deal about the fact that the people placing these bets were in a constant state of WTF: Why is nobody seeing what we’re seeing? Why is nobody doing what we’re doing? Before reading this book I’d have characterized WTF as mainly occurring in math, science, and engineering; it was great to get an extended anecdote from the financial world.

Positive WTF comes from the best possible kind of ignorance: ignorance that, once its shape is known, motivates new discoveries. Disciplined thinking is required to avoid succumbing to the natural temptation to sweep the WTF-inducing issues under the rug. We must embrace WTF because it is a leading indicator that we don’t understand something crucial.
