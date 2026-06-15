---
title: A Guide to Better Scripty Code for Academics
url: https://blog.regehr.org/archives/1153
published: "2014-05-27T19:34:40Z"
feed: regehr
guid: http://blog.regehr.org/?p=1153
---

# A Guide to Better Scripty Code for Academics

\[ [Suresh](http://www.cs.utah.edu/~suresh/web/) suggested that I write a piece about unit testing for scripty academic software, but the focus changed somewhat while I was writing it.\]

Several kinds of software are produced at universities. At one extreme we have systems like [Racket](http://racket-lang.org/) and [ACL2](http://www.cs.utexas.edu/users/moore/acl2/) and [HotCRP](http://read.seas.harvard.edu/~kohler/hotcrp/) that are higher quality than most commercial software. Also see the [ACM Software System Award winners](http://awards.acm.otherg/software_system/year.cfm) (though not all of them came from academia). I wrote [an earlier post](https://blog.regehr.org/archives/1058) about how hard it is to produce that kind of code.

This piece is about a different kind of code: the scripty stuff that supports research projects by running experiments, computing statistics, drawing graphs, and that sort of thing. Here are some common characteristics of this kind of code:

- It is often written in several different programming languages; for example R for statistics, Matplotlib for pretty pictures, Perl for file processing, C/C++ for high performance, and plenty of shell scripts and makefiles to tie it all together. Code in different languages may interact through the filesystem and also it may interact directly.
- It seldom has users outside of the research group that produced it, and consequently it usually embeds assumptions about its operating environment: OS and OS version, installed packages, directory structure, GPU model, cluster machine names, etc.
- It is not usually explicitly tested, but rather it is tested through use.

The problem is that when there aren’t any obvious errors in the output, we tend to believe that this kind of code is correct. This isn’t good, and it causes many of us to have some legitimate anxiety about publishing incorrect results. In fact, I believe that incorrect results are published frequently (though many of the errors are harmless). So what can we do? Here’s a non-orthogonal list.

## Never Ignore Problems

Few things in research are worse than discovering a major error way too late and then finding out that someone else had noticed the problem months earlier but didn’t say anything. For example we’ll be tracking down an issue and will find a comment in the code like this:

```
  # dude why do I have to mask off the high bits or else this segfaults???

```

Or, worse, there’s no comment and we have to discover the offending commit the hard way — by understanding it. In any case, at this point we pull out our hair and grind our teeth because if the bug had been tracked down instead of hacked around, there would have been huge savings in terms of time, energy, and maybe face. As a result of this kind of problem, most of us have trained ourselves to be hyper-sensitive to little signs that the code is horked. But this only works if all members of the group are onboard.

## Go Out of Your Way to Find Problems

Failing to ignore problems is a very low bar. We also have to actively look for bugs in the code. The problem is that because human beings don’t like being bothered with little details such as code that does not work, our computing environments tend to hide problems by default. It is not uncommon for dynamically and weakly typed programming languages to (effectively) just make up crap when you do something wrong, and of course these languages are the glue that makes everything work. To some extent this can be worked around by turning on flags such as `-Wall` in gcc and `use warnings; ` `use strict;` in Perl. Bugs that occur when crossing layers of the system, such as calling into a different language or invoking a subprocess, can be particularly tricky. My bash scripts became a lot less buggy once I discovered the `-e` option. Many languages have a lint-like tool and C/C++ have Valgrind and UBSan.

One really nice thing about scripty research code is that there’s usually no reason to recover from errors. Rather, all dials can be set to “fail early, fail fast” and then we interactively fix any problems that pop up.

The basic rule is that if your programming environment supports optional warnings and errors, turn them all on (and then maybe turn off the most annoying ones). This tends to have a gigantic payoff in terms of code quality relative to effort. Also, internal sanity checks and [assertions](https://blog.regehr.org/archives/1091) are worth their weight in gold.

## Fight Confirmation Bias

When doing science, we formulate and test hypotheses. Although we are supposed to be objective, objectivity is difficult, and there’s even a term for this. According to Wikipedia:

> [Confirmation bias](http://en.wikipedia.org/wiki/Confirmation_bias) is the tendency of people to favor information that confirms their beliefs or hypotheses.

Why is this such a serious problem? For one thing, academia attracts very smart people who are accustomed to being correct. Academia also attracts people who prefer to work in an environment where bad ideas do not lead to negative economic consequences, if you see what I mean. Also, our careers depend on having good ideas that get good results. So we need our ideas to be good ones — the incentives point to confirmation bias.

How can we fight confirmation bias? Well, most of us who have been working in the field for more than a few years can easily bring to mind a few examples where we felt like fools after making a basic mistake. This is helpful in maintaining a sense of humility and mild research paranoia. Another useful technique is to assume that the people doing previous work were intelligent, reasonable people: if implementing their ideas does not give good results, then maybe we should figure out what we did wrong. In contrast, it is easy to get into the mindset that the previous work wasn’t very good. Evidence of this kind of thinking can be seen in the dismissive related work sections that one often sees.

## Write Unit Tests

Modern programming languages come with good unit testing frameworks and I’ve noticed that the better students tend to instinctively write unit tests when they can. In contrast, us old fogies grew up as programmers long before the current testing culture developed and we have a harder time getting ourselves to do this.

But does unit testing even make sense for scripty code? In many cases it clearly doesn’t. On the other hand, Suresh gives the example where they are comparing various versions of an algorithm; in such a situation we might be able to run various data sets through all versions of the algorithm and make sure their results are consistent with each other. In other situations we’re forced to re-implement a statistical test or some other piece of fairly standard code; these can often be [unit tested using easy cases](https://blog.regehr.org/archives/858). Mathematical functions often have properties that support straightforward smoke tests. For example, a function that computes the mean or median of a list should compute the same value when fed the same list twice.

## Write Random Testers

It is often the case that an API that can be unit tested can also be fuzzed. Two things are required: a test-case generator and an oracle. The test-case generator can do something easy like randomly shuffling or subsetting existing data sets or it can make up new data sets from scratch. The oracle decides whether the code being tested is behaving correctly. Oracles can be weak (looking for crashes) or strong (looking for correct behavior). Many modern programming languages have a [QuickCheck](http://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/quick.pdf)-like tool which can make it easier to create a fuzzer. [This blog post](https://blog.regehr.org/archives/737) and [this one](https://blog.regehr.org/archives/896) talk about random testing (as do plenty of others, this being one of my favorite subjects).

## Clean Up and Document After the Deadline

As the deadline gets closer, the code gets crappier, including the 12 special cases that are necessary to produce those weird graphs that reviewer 2 wants. Cleaning this up and also documenting how the graphs for the paper were produced is surely one of the best investments we could make with our time.

## Better Tooling

Let’s take it as a given that we’re doing code reviews, using modern revision control, unit testing frameworks, static and dynamic analysis tools, etc. What other tool improvements do we want to see? [Phil Guo’s thesis](http://pgbovine.net/publications/Philip-Guo_PhD-dissertation_software-tools-for-research-programming.pdf) has several examples showing how research programming could be improved by tools support. There’s a lot of potential for additional good work here.

## Summary

There are plenty of easy ways to make scripty research code better. The important thing is that the people who are building the code — usually students — are actually doing this stuff and that they are receiving proper supervision and encouragement from their supervisors.
