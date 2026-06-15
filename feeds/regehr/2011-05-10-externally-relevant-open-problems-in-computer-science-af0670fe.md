---
title: Externally Relevant Open Problems in Computer Science
url: https://blog.regehr.org/archives/532
published: "2011-05-10T20:22:14Z"
feed: regehr
guid: http://blog.regehr.org/?p=532
---

# Externally Relevant Open Problems in Computer Science

Most academic fields have some externally relevant problems: problems whose solutions are interesting or useful to people who are totally ignorant of, and uninterested in, the field itself. For example, even if I don’t want to know anything about virology, I would still find a cure for the common cold to be an excellent thing. Even if I failed calculus I will find it interesting when physicists invent a room-temperature superconductor or figure out why the universe exists.

This piece is about computer science’s externally relevant open problems. I have several reasons for exploring this topic. First, it’s part of my ongoing effort to figure out which problems matter most, so I can work on them. Second, I review a lot of proposals and papers and often have a strong gut reaction that a particular piece of work is either useful or useless. An idea I wanted to explore is that a piece of research is useless precisely when it has no transitive bearing on any externally relevant open problem.

A piece of work has “transitive bearing” on a problem if it may plausibly make the problem easier to solve. Thus, an esoteric bit of theoretical physics may be totally irrelevant or it may be the key to room temperature superconductivity. Of course, nobody thinks they’re doing irrelevant work. Nevertheless, it’s hard to escape the conclusion that a lot of CS (and other) research is in fact irrelevant. My third motivation for writing this piece is that I think everyone should do this sort of analysis on their own, rather than just believing the accepted wisdom about which research topics are worth working on. The analysis is important because the accepted wisdom is — at best — a lagging indicator.

Below is my list. I’ll be interested to see what people think I’ve missed (but note that I’m deliberately leaving out problems like P vs. NP which I don’t think are externally relevant).

# Giving More People Meaningful Control Over Computation

There are plenty of people — scientists, analysts, managers, etc. — who are not programmers but who would benefit from gaining better control over computers. I’m using the phrase “meaningful control over computation” as a bit of a hedge because it’s clear that most of these people don’t have 2-5 years to spare in which to become competent programmers. The goal is to give people the power that they need to solve their problems without miring them in the Turing tarpit. A good example is the class of spreadsheet programming languages which expose a useful kind of computation without most of the problems traditionally associated with programming. Overall, this problem is maybe 1/3 programming language design, 1/3 user interface design, and 1/3 domain specific.

# Trustworthy Automation

Can we create computer systems that do what we want them to do? This problem encompasses both security and reliability. It’s a good problem to work on because solutions have not only have short-term economic benefit but in the long run they directly support getting humans off the rock, which [as I’ve said before](https://blog.regehr.org/archives/43) is something we need to be working very hard on.

The whole problem of specifying what we want systems to do is enormously hard, and in fact we generally have no precise ideas about this. Even highly mathematical objects like compilers are most commonly specified in human language, if at all. Moreover, the programming languages that we use to specify computations contain elements such as floating point computations, fixed-width integers, and excessive use of side effects — all of which seem designed to impede reasoning.

# Intelligent Systems

Can computers be created that interface gracefully with humans? How can augmented cognition be used to sidestep limitations of humans and computers? Which sub-problems of AI are achievable and useful? Here I mean “intelligent” literally, not in the sense it is usually used, where it means “not as obviously bad as the previous thing.” Of course, the goal of AI may turn out to conflict with “trustworthy automation” but we’ll have to cross that bridge when we come to it.

# Observing, Simulating, Modeling, and Predicting Everything

The universe produces a gigantic amount of data at scales from atoms to galaxies. Luckily, the theoretical bounds on the amount of computation that can be done using the available energy are very generous. The ugly little space heaters with which we currently compute have not even started to scratch the surface of what’s possible.

This one is pretty vague, but I’m not sure right now how to improve it.

# Further Reading

Vernor Vinge and Hans Moravec have created some of the best unconstrained thinking about computers. Asimov was pretty good in his day, but [Sladek nailed it](http://en.wikipedia.org/wiki/Tik-Tok_%28novel%29). The transhumanist material on the web seems pretty bad, as is Kurzweil’s stuff. Dertouzos’s *Unfinished Revolution* was not inspiring. There must be more (good and bad) writing on this subject, please make suggestions.

**Update from evening of 5/10:** This list of external problems is of course subjective. I’d be very interested to see other people’s blog posts describing what they think CS has to offer the non-CS world.
