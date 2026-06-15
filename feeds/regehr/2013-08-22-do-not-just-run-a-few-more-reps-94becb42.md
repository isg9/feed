---
title: Do Not Just Run a Few More Reps
url: https://blog.regehr.org/archives/1024
published: "2013-08-22T03:21:04Z"
feed: regehr
guid: http://blog.regehr.org/?p=1024
---

# Do Not Just Run a Few More Reps

It’s frustrating when an experiment reveals an almost, but not quite, statistically significant effect. When this happens, the overwhelming temptation is to run a few more repetitions in order to see if the result creeps into significance. Intuitively, more data should provide more reliable experimental results. This is not necessarily the case.

Let’s look at a concrete example. We’ve been handed a coin and wish to determine if it is fair. In other words, we’ll conduct an experiment designed to see if the null hypothesis (“the coin is fair”) can be rejected at the desired level of confidence: 95% in this case. Just to review, informally a result with 95% confidence means that we are willing to accept a 5% chance that our experiment reaches a false conclusion.

Consider this graph:

![](https://blog.regehr.org/extra_files/conf2.png)

The x-axis shows a range of experiments we might be willing to perform: flipping the coin between 10 and 100 times. The red line is the lower confidence bound: the smallest number of heads that indicates a fair coin at the 95% level; the green line indicates the expected number of heads; and, the dark blue line indicates the upper confidence bound: the largest number of heads that is consistent with the coin being fair at the 95% level. The remaining three lines are examples of the kind of random walk we get by actually flipping a coin (or consulting a PRNG) 100 times.

Let’s say that we have decided to flip the coin 40 times. In this case, the 95% confidence interval (using the Wilson score) has a lower bound of 14.1 and an upper bound of 25.9. If we perform this coin flip experiment many times, the coin is declared to be fair 96% of the time — a good match for our desired level of confidence. If we instead flip the coin 80 times, the 95% confidence interval is between 31.4 and 48.6. This new experiment only finds the coin to be fair about 94% of the time. This mismatch between desired and actual coverage probability — which is often, but not always, minor — is an inherent feature of binomial confidence intervals and [I’ve written about it before](https://blog.regehr.org/archives/836).

So now let’s get back to the point of this post. The question is: How are the results affected if, instead of selecting a number of repetitions once and for all, we run a few reps, check for significance, run a few more reps if we don’t like the result, etc. The setup that we’ll use here is:

1. Flip the coin 10 times

2. Check for an unfair coin — if so, stop the experiment with the result “unfair”

3. If we’ve flipped the coin 100 times, stop the experiment with the result “fair”

4. Flip the coin

5. Go back to step 2

The graphical interpretation of this procedure, in terms of the figure above, is to look for any part of the random walk to go outside of the lines indicating the confidence intervals. The light blue and brown lines do not do this but the purple line does, briefly, between 60 and 70 flips.

The punchline is that this new “experimental procedure” is able declare a fair coin to be unfair about 37% of the time! Obviously this makes a mockery of the intended 95% level of confidence, which is supposed to reach an incorrect conclusion only 5% of the time.

Does this phenomenon of “run a few more reps and check for significance” actually occur? It does. For example:

- People running experiments do this.

- Reviewers of papers complain that the results are not “significant enough” and ask the experimenter to run a few more reps.

- I’ve seen benchmarking frameworks that automatically run a performance test until a desired level of confidence is achieved.

In contrast, near the start of [this paper](http://arxiv.org/ftp/arxiv/papers/1205/1205.4251.pdf) there’s a nice story about experimenters doing the right thing.
