---
title: Fuzzers Need Taming
url: https://blog.regehr.org/archives/925
published: "2013-04-09T20:14:52Z"
feed: regehr
guid: http://blog.regehr.org/?p=925
---

# Fuzzers Need Taming

\[This post explains [a paper that we recently made available; it’s going to be presented at PLDI 2013](http://www.cs.utah.edu/~regehr/papers/pldi13.pdf).\]

Random testing tools, or fuzzers, are excellent at finding bugs that human testers miss. A particularly important use case for fuzzing is finding exploitable bugs, and companies such as [Google use clusters to do high-throughput fuzzing](http://blog.chromium.org/2012/04/fuzzing-for-security.html). Whether you have a cluster or just a laptop, the usual workflow for fuzzing is something like this:

1. Build the latest version of the code.
2. Start the fuzzer and then go home for the night or the weekend.
3. Later on, report a bug for each important new issue discovered by the fuzzer.

The problem is that step 3 can be a lot of work if, for example, the fuzzer discovers a few thousand test cases that cause failures. The situation is a bit complicated and next I’ll try to walk through some of the issues.

First, it is typical for some bugs to be triggered thousands of times more often than other bugs. Therefore, if you randomly choose a handful of failure-triggering test cases to look at, they are likely to all trigger the same bug or the same small set of bugs. A corollary is that in a fuzzing run of any length, there will be bugs that are triggered only once. (Clearly this cannot be true in the limit since there can only be a finite number of bugs in any code base. But it holds true for surprisingly long runs.) This graph shows how many times each of a collection of bugs was triggered, for three kinds of bugs, due to fuzzing GCC 4.3.0 and SpiderMonkey 1.6 for a few days:

_[![powerlaw](https://blog.regehr.org/wp-content/uploads/2013/04/powerlaw1.png)](https://blog.regehr.org/archives/925/powerlaw-2)_

Second, the job of triaging failure-triggering test cases can be made much easier if test case reduction is used. For example, here’s a [fairly generic test-case reduction tool](http://delta.tigris.org/) and here’s my group’s [domain-specific tool that does a much better job](http://embed.cs.utah.edu/creduce/) for test cases that trigger bugs in C/C++ compilers. It is possible (though this wasn’t our experience) that a sufficiently good test case reduction tool makes it easy to examine and categorize every failure-triggering test case discovered by a fuzzer.

Third, if we can always fix bugs rapidly, then we can simply fix the first bug found by the fuzzer and then re-run the fuzzer. This is easy to do for small projects, but completely impractical for large projects that often have hundreds or thousands of open bugs at any given time, many of which are non-critical and therefore aren’t going to get fixed in the short term.

Fourth, it is sometimes the case that each bug has an unambiguous symptom. For example, if we have a bug that breaks a data structure’s invariants that is closely followed by an assertion that catches the bug, then we’ll reliably get a message such as “assertion violation at line 712 of foo.c” when this bug executes. If this is the case for all of the bugs in our code base, then we can rapidly find the true set of bugs by doing something simple like “grep Assert fuzzer-output/\* \| sort \| uniq”. On the other hand, in practice there are often many bugs that result in ambiguous symptoms such as crashes or arbitrary incorrect output.

To summarize the issues so far: the problem of triaging a collection of test cases that trigger bugs may be easy or difficult depending on a number of factors. If this is easy, then great, you don’t need our work. If it is hard, then this can make fuzzing a lot less attractive. We have noticed this ourselves and also we heard from [Csmith](http://embed.cs.utah.edu/csmith/) users who decided to stop using the tool not because it stopped finding bugs, but rather because it kept finding bugs that were already known and weren’t going to get fixed in the short term. So we decided to look for an automated solution to this problem:

> **Fuzzer taming problem:** Given a potentially large collection of test cases, each of which triggers a bug, rank them in such a way that test cases triggering distinct bugs are early in the list.
>
> **Sub-problem:** If there are test cases that trigger bugs previously flagged as undesirable, place them late in the list.

The obvious approach is to use [clustering](http://en.wikipedia.org/wiki/Cluster_analysis) to divide the test cases into groups and then to hand the user one test case from each cluster (or, to solve the sub-problem, one bug from each cluster not containing a test case that triggers an undesirable bug). We tried this and it worked ok, but the approach was slow, it seemed overly complicated, and the clustering approach has no built-in way to perform a ranking among clusters. Moreover, we got the sense (but never really proved) that the clustering software we were using was probably trying to deal with outliers in some graceful way. On the other hand, the power-law distribution of bugs triggered by random test cases means that the outliers are likely to be interesting.

[Suresh](http://geomblog.blogspot.com/) pointed us to [Clustering to Minimize the Maximum Intercluster Distance](http://www.cs.ucsb.edu/~teo/papers/Ktmm.pdf), which presents the technique that we ended up using, which is to rank test cases in such a way that each test case in the list is the one that maximizes the distance to the closest previously listed test case. In other words, each test case is the one that is least similar to any test case listed so far. This is called the *furthest point first* (FPF) ordering and it can be used to create a clustering if you run FPF for a while and then form clusters by adding each unlisted point to the cluster defined by the nearest listed point. For our purposes clustering is uninteresting: we just want a ranking. Therefore, we run FPF all the way to the end, giving a list that contains diverse test cases at the head and mostly very similar test cases at the end.

So far I have left out something important: how to compute the distance between two test cases. This is the input to a clustering algorithm or to the FPF computation. The short answer is that we tried a variety of distance functions ranging from Levenshtein distance (or edit distance: the number of character additions, deletions, and alterations that suffices to change one string into another) over test case text to euclidean distance over vectors of code coverage data. Why try multiple distance functions? This is because bugs are diverse and we don’t have any reason to think that we can come up with the one true distance function that accurately captures test case similarity for all bugs. The paper contains more details.

The way to evaluate something like a fuzzer tamer is with a discovery curve: a graph that shows how quickly a particular ranking shows a person inspecting the items in ranked order one member of each class of objects being examined. Here are some discovery curves for SpiderMonkey bugs:

[![js](https://blog.regehr.org/wp-content/uploads/2013/04/js1.png)](https://blog.regehr.org/archives/925/js-2)

And here are the results for some bugs that cause GCC 4.3.0 to generate incorrect object code:

[![gcc](https://blog.regehr.org/wp-content/uploads/2013/04/gcc.png)](https://blog.regehr.org/archives/925/gcc)

The upshot is that the baseline—looking at deduplicated reduced test cases in random order—is not very attractive and that you can discover all bugs much faster using fuzzer taming. The most interesting part of each graph is at the left edge because we would expect a human to inspect maybe 10 or 15 test cases, not 500 or 700. In that region, the tamed results will show you about twice as many bugs as you’d see by examining test cases in random order (some more detailed graphs are in the paper). The curves for clustering in these graphs represent our best attempt to get a ranking out of [x-means](http://www.cs.cmu.edu/~dpelleg/download/xmeans.pdf), a modern clustering algorithm. Of course it is possible that some other clustering technique would do better.

Our ranking approach was inspired by similar approaches that are used to [rank static analysis results](http://www.stanford.edu/~engler/sas-camera-ready.ps). Notice that they also show bug discovery curves.

So what do you do with a fuzzer taming tool? My idea is to bolt together Csmith, C-Reduce, and the fuzzer tamer into a  nice integrated package. Someone who wants to fuzz a compiler should be able to build it, point it at a compiler (or better yet, the compiler’s repo) and then every day or so, glance at a web page that shows them interesting, newly discovered defects in the compiler. The toolchain will look something like this:

[![workflow](https://blog.regehr.org/wp-content/uploads/2013/04/workflow1.png)](https://blog.regehr.org/archives/925/workflow-2)

My guess is that other fuzzing workflows can also benefit from this sort of integration. We’d appreciate feedback on this material. The work described in this post was done by my group in conjunction with Alex Groce and others at Oregon State University.
