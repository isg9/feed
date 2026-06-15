---
title: Software Engineering Takeaways
url: https://blog.regehr.org/archives/1594
published: "2018-05-15T17:35:36Z"
feed: regehr
guid: http://blog.regehr.org/?p=1594
---

# Software Engineering Takeaways

I had a great time this spring teaching a software engineering course for a new professional masters degree program created by my department. Since I didn’t use slides or hand out lecture notes, some students were asking if maybe I could write up a summary of what I wanted them to learn in the course. This sounded like a good idea and I figured I’d make a blog post out of it.

This course didn’t have a textbook, but we ended up going through much of [Writing Solid Code](http://writingsolidcode.com/) and also several chapters of [Code Complete](https://www.amazon.com/Code-Complete-Practical-Handbook-Construction/dp/0735619670/). I particularly like Appendix D of the 2nd edition of Writing Solid Code, where Maguire describes how he conducts an interview for a programming position. The students also completed about a dozen (mostly) programming assignments, individually and in groups, that I’m not going to talk about here.

## Testing

Being good at testing is sort of a superpower and computer science programs usually don’t do a great job teaching it. And testing truly is hard to teach since there’s a lot to it and so much of testing is either highly situational or else is about engaging the problem with the right mindset and plenty of energy: you can’t test well unless you truly want to make things fail. My approach to teaching this material is to focus on what I think of as testing’s three pillars:

1. **Test cases.** These come from regressions, from systematic or randomized test-case generators, from requirements and specifications, and from corner cases in the implementation. In a few special situations, exhaustive testing works. Test cases should rarely be discarded but they can be segregated, for example into fast and slow ones, which run at different times. Test cases should exercise all relevant sources of input: not just file inputs, but also environment variables, user inputs, and things like the Windows registry, if the system under test depends on those things. The time at which parts of the test case arrive may be important. Though I forgot to give it to the class, Cem Kaner’s [What Is a Good Test Case?](http://www.kaner.com/pdfs/GoodTest.pdf) is excellent.

2. **Oracles.** Test cases are only useful insofar as we can tell if the system under test processes them correctly. Some oracles are easy and broadly applicable, such as detecting crashes, excessive resource use, and violations of programming language rules (e.g. using ASan and UBSan). The most useful oracles, however, are specific to the system under test. The one we see most often is to specify, by hand, the expected behavior of the system, based on our understanding of the requirements or specification. Differential testing — checking one implementation of a spec against another — is highly powerful, and more broadly applicable than it first appears: the reference implementation doesn’t need to implement the full functionality of the system, and sometimes it is simply a different mode of the same system. For example, an optimizing compiler can be tested against a non-optimizing version of itself. [Function-inverse pairs](https://blog.regehr.org/archives/708) make an excellent oracle, when applicable. In [metamorphic testing](https://en.wikipedia.org/wiki/Metamorphic_testing) we change the test case (add irrelevant levels of nesting to a C program, or remove dead code from it) in a way that shouldn’t change how it works. [Assertions](https://blog.regehr.org/archives/1091) and [checkReps](http://www.pgbovine.net/programming-with-rep-invariants.htm) are extremely valuable oracles. Finding good oracles requires creativity and attention to detail, but the potential rewards are high.

3. **Coverage.** Every serious programming language has one or more tools for measuring code coverage, with the goal of finding code not exercised by a test suite. There are a lot of variants on coverage, but in practice we seldom see anything beyond line or branch coverage. If the coverage induced by a test suite is bad, the test suite itself is bad. If the coverage is good, then the test suite might be good, but [this is not a sure thing](http://www.exampler.com/testing-com/writings/coverage.pdf).

Testing should be as frictionless as possible: investments in automation and parallelization often pay off. In class we saw how incredibly easy it is to run the tests for a project in Github on every commit using Travis.

One of my favorite kinds of testing is [burning in an ADT implementation](https://blog.regehr.org/archives/896) — this exercise brings the three pillars of testing together in a very satisfying way.

[How SQLite is Tested](https://www.sqlite.org/testing.html) is great supplemental reading.

## Tool and Library Ecosystems

One of the defining characteristics of modern software engineering is its reliance on tooling to help us create large projects rapidly, with lots of high-quality libraries to build on. Every programming language has one or more collections of tools, each of which forms a more or less coherent ecosystem for getting programming tasks done: editing, refactoring, linting, profiling, building, packaging, measuring coverage, debugging, etc. The tricky bit for newcomers is to rapidly learn the shape of an ecosystem in order to avoid wasting time doing jobs that could be better accomplished with tool support.

## Modularity

If testing is the reason that large software sometimes works, then modularity is the reason it can be constructed in the first place. Modularity is kind of a tough topic to cover in a course but we did spend time on API design, where I mainly wanted to convey that it’s hard to get right and there are plenty of pitfalls to avoid: excess mutability, unclear ordering constraints among API calls, long lists of parameters, poor choices for default values, hard-to-follow memory ownership protocols, cryptic names, and unwise exposure of implementation details. Something I tried to show the class is how a small group of people working closely together on a well-defined task can almost always succeed, but that — without some care and experience — multiple small groups who have accomplished tasks separately will invariably have problems integrating things they have developed separately.

## Code Reviews

Testing only gets us so far and a huge number of problems can be headed off by putting two or more sets of eyes on code before it lands, whether this is done in Github or in a meeting room. We did a bit of code reviewing in class, though I feel like we should have done this more. Some aspects I find important are trying to stay constructive and objective, having clear goals for the code review, and making sure that outcomes get written down and acted upon.

## Coding Styles

My view is that the details aren’t nearly as important as just having and following a coding style. This include source code formatting details, identifier naming, language features to avoid, etc. This can result in a code base that is more approachable to newcomers and where some kinds of bugs are easier to spot. The various [Google documents](https://google.github.io/styleguide/) seem as good examples as any. Coding styles are particularly important for languages that are older, cruftier, and less safe. For example, [LLVM lives in a C++ style](https://llvm.org/docs/CodingStandards.html) that I generally find to be comfortable and tasteful.

## Source Control Systems

Git is more or less the only game in town so there’s not much to talk about there. The thing I tried to convince the students here is that we don’t just “use git.” Rather, it’s crucial to define a sensible git-based workflow and get everyone onboard with it. The main thing is that for a variety of common use cases, we have a fairly mechanical set of steps to follow; git is confusing and badly designed enough that workflow innovation is best left to people with a lot of experience.

## Critical Systems and Responsibility

We are far past the point of pretending that software is either ethically neutral or uniformly positive, and the 90s-era Internet optimism sounds pretty naive at this point. A significant fraction of today’s CS students are going to work on, at some point, software that is very much ethically non-neutral. [This is a good post](https://calebhearth.com/talks/dont-get-distracted/) on that topic. We read various documents related to the Toyota unintended acceleration case and discussed other software safety problems like the Therac-25.

## Backwards Compatibility

People tend to underestimate its importance and there are great stories on both sides. Microsoft’s commitment is incredible: a long time ago I had access to the Windows 2000 sources and ran across crazy things, like a copy of Windows 3.1 (in x86 assembly!) and routines for detecting legacy applications and implementing various buggy behaviors that they depended on. On the other side we have, for example, the Python 2/3 debacle.

## Static Analysis

I tried to convince students that static analysis is useful, starting with compiler warnings. We read [A Few Billion Lines of Code Later](https://cacm.acm.org/magazines/2010/2/69354-a-few-billion-lines-of-code-later/fulltext) and they spent some time looking at issues pointed out by the Clang static analyzer.

## Engineering for Performance

The message here is a bit subtle: we want to discourage premature optimization while still encouraging people to learn to build software that isn’t fundamentally too slow to meet its goals. So on one hand we don’t want to write everything in C “for speed” but on the other hand we need to avoid showstoppers such as Python, a bottleneck data structure stored on disk, a quadratic algorithm, etc.

For performance tuning, of course measurement is everything, so we need profilers and maybe also hardware-specific utilities like perf. It doesn’t hurt to know the strengths and weaknesses of both the compiler and the hardware platform, and we should always be ready to read some assembly. Chapters 25 and 26 of Code Complete 2e are a good resource for all of this, though they do not present a very nuanced view of what one can expect the compiler to accomplish.

## Reporting Bugs

Getting a software developer to stop doing what they wanted to do, and to spend the day fixing a defect instead, can be difficult, but there are a few simple ingredients that can make bug reports vastly more effective. The report should be framed politely and matter-of-factly; implying that the developers are incompetent is rarely helpful. The issue must be reproducible and the bug report must describe all of the circumstances necessary to make the bug show itself. However, all nonessential circumstances should be left out, including any part of the test case that is not necessary for triggering the bug.

## Software Process

There are plenty of [software development process models](https://en.wikipedia.org/wiki/Software_development_process), but I guess I don’t find any of these to be particularly compelling, so we didn’t spend much time on them. On the other hand, the elements of software process — estimation, requirements, modeling, architecture, implementation, quality assurance, maintenance, etc. — are hard to argue with. I spend a bit of effort trying to prepare students for the huge diversity in software development organizations they might see out in the world, from startups to banks to Googles to IBMs. I tried to get them used to the fact that in some cases management may have wildly unrealistic ideas about software development and that requirements can change at the drop of a hat.

## The Human Element

Effectively reviewing someone’s code requires a light touch; you have to critique the code rather than judging its author. On the other side, it can be very difficult to gracefully process criticism of a piece of code that you’ve bled and sweated over for months. A bug report is really just an argument that a person should make a change to a piece of software. It is hard to implement a piece of software that solves someone’s problem without putting yourself in their place. Similarly, it is very difficult to solve a problem for someone you don’t like or respect or at least empathize with to some degree. UI design requires understanding the needs of people who don’t share your own abilities and cultural background. Ethical concerns are common, and probably should be thought about a lot more often than they are. Team dynamics are complex and managing people is really not easy. In summary, software development is fundamentally a human endeavor and we’d all do well to keep that in mind.
