---
title: Picking a Research Topic in Computer Systems
url: https://blog.regehr.org/archives/46
published: "2010-03-21T17:38:14Z"
feed: regehr
guid: http://blog.regehr.org/?p=46
---

# Picking a Research Topic in Computer Systems

This post is a collection of observations and advice for people who want to choose a research topic in computer systems.  I’m not claiming to be some kind of genius in this area, but I have enough ideas that they seemed worth writing down. This advice is probably most useful for graduate students in CS, but may also be helpful for junior profs and for undergrads interested in doing research.

# Picking a research area

By “research area” I mean a sub-area of computer systems, such as object-oriented operating systems, distributed hash tables, or whatever.  This often ends up being a pretty pragmatic decision and it is understood that a good researcher can work concurrently in multiple areas or change area as often as every few years.

The easiest way is to choose an area that a lot of other people are working on.  I’m quite serious, and this method has some important advantages.  First, all those people at MIT and Berkeley aren’t stupid; if they’re working on ultra-reliable byzantine transcoders, there’s probably something to it.  Second, if you work on the same thing others are working on, the odds are good they’ll be interested in your work and will be more willing to fund and publish it.  The potentially serious drawback is that you’ll likely miss the initial, exciting phase of research where lots of new things are being discovered and there’s not yet a lot of commonly accepted wisdom about what the best solutions are.  In the worst case, you’ll miss the entire main period of interest and arrive late when the best people have moved on to other topics.

The research area that you choose should have a chance of making a difference.  Systems research tends to be applied, and a great deal of it is engineering rather than science.  The focus, in most cases, is making something work better for someone.  The more people benefit, the better.

Fundamentally, the area you choose needs to be one that you have some natural affinity for.  You should enjoy working on it and your intuition should strongly suggest that all is not right: there is work that needs to be done.

If you’re a PhD student seeking a research assistantship, it would be practical to choose a research area that your advisor has a grant to work on.

# Departing from the accepted wisdom

Every good research idea is a departure from the accepted wisdom, but it’s important to depart at the right level.  Consider these extremes:

1. You reject the notion of binary computers.  Instead, ternary computation will sidestep most of the fundamental problems faced by computer science.  Everything from instruction sets to complexity theory must be thrown out and rebuilt from scratch.
2. You reject the notion of the semicolon as a statement terminator in programming languages.  Instead, the dollar sign should be used.  A revolution in software productivity will ensue.

The first idea diverges from the status quo in a fundamental way, but it will be difficult to get buy-in from people.  The second departure is too minute to matter, and nobody will care even if you do a big user study showing that the dollar sign is better with p<0.05.  In both cases, the research idea does not feel like one that will change the world.  In contrast, some great examples of departing from the conventional wisdom in the right way can be found in David Patterson’s work: see RISC and RAID.

# Focusing a skeptical personality

If the point is to challenge the accepted wisdom in some fashion, you can’t very well go believing everything people tell you.  Computer systems is not exactly a rigorous area of science and you will hear all manner of ridiculous explanations for things.

Good systems problems, in my experience, come from noticing something wrong, some discrepancy between how the world works and how it should work.  However, it doesn’t appear possible to do good work based on someone else’s observations.  For example, you can tell me that parallel programming is hard or software transactional memory is too slow, but the fact is that if I haven’t seen the problems for myself, if long and bitter experience hasn’t trained my intuition with a thousand little facts about what works and what does not, then I’m probably going to come up with an irrelevant or otherwise bad solution to the problem.

How does this work in practice?  You go about your business building kernels or web servers or whatever, and along the way there are a hundred irritations.  Most of them are incidental, but some are interesting enough that you start pulling at the thread.  Most of these threads die quickly when you discover that the problem was adequately solved elsewhere, is boring, or is too difficult to solve at present.  Every now and then it becomes clear that there’s a real problem to be solved, that nobody else is attacking it (at least in the right way), and that it would be fun to work on.  These are your potential research projects.  There are probably other ways to find research ideas in computer systems, but this is the only one I know of.

Here’s an anecdote.  As an instructor of embedded systems courses I’d long been annoyed by buggy cross compilers for microcontrollers.  Students who are struggling to write correct code do not need this extra level of hassle.  Finally, one day I was showing a small snippet of assembly code in lecture and a student noticed that it was wrong.  I assumed that it was a cut-and-paste error, but it turned out the compiler was mistranslating a 2-line function.  This seemed beyond the pale, so I wrote some more test cases and tested some more compilers and kept finding more and more bugs.  Even at this point, after I’d spent probably 40 hours on the problem, it was not at all clear that I was moving towards any kind of research result.  It was only after hundreds of additional hours of work that it became obvious the project had legs: everyone knows that embedded compilers contain bugs, but probably most people would be surprised that we were able to find (so far) 200 compiler bugs including major wrong-code bugs in every C compiler we’ve tested.  So in this case, the surprising result is quantitative.

# A few more observations

- The best research problems are often those that are not yet of major industrial interest, but that will be addressed by billion-dollar industries in 10-20 years.  Once a problem becomes the focus of intense industrial interest, it becomes a difficult target to attack from academia.  At the very least, you need a seriously new angle.
- Don’t get into a research area late in its life cycle.
- Thomas Edison said it’s 1% inspiration, 99% perspiration.  In computer systems it’s more like 99.9% perspiration: if you’re not careful you can build things for years without getting any research results.
- It’s really not possible to figure out in advance which promising ideas are going to pan out and which ones are distractions.  Keep a number of different ideas in mind and work in directions that cut off as few options as possible.
- Ignore sunk costs.  Always be ready to drop an idea, no matter how proud of it you are.
- If you become aware of someone credible who is working on your exact idea, drop it.  First, at this point you have a 50% chance of winning the race to publication.  Second, duplicated work wastes time and tax dollars.  Life’s too short.
- Often, problems and obstacles that seem insurmountable at first can be flipped around and turned into interesting features of your work or even advantages.
- Periodic re-examination of assumptions is useful.  A recent example I like is [Google native client](http://code.google.com/p/nativeclient/).  Most efforts to isolate untrusted binary code just take whatever the compiler outputs.  The Google project gets good mileage by hacking the compiler so that the code doesn’t contain so many stupid things.  It’s a good idea — who said the compiler was set in stone?
- If your project seems boring, think about dropping it.  If you’re not excited why would anyone else be?
- Writing a couple of paragraphs about an idea has the effect of revealing bad ideas for what they are, and making good ideas better.  It’s almost comical how many of my ideas look silly when written down.  It’s definitely possible to read too much, but it’s probably impossible to write too much.
- Smart people love to solve problems, regardless of their relevance.  Avoid the trap where you improve a result far beyond the point of diminishing returns.
- Code tweaking is seductive; if you do it, consider it time spent on a hobby, not time spent doing research.
- Once you’re invested in a research area, it’s tempting to stay there since you’re on top of the learning curve.  This can turn into a trap.  When it’s time to move on, just do it.  Many of the best researchers change areas several times during their careers.
- Having a grand vision up-front is OK as long as it’s somewhat vague and doesn’t prevent you from seeing the actual situation.
- Computer systems speak to you, in their own way.  Always listen to what they’re telling you.

# Summary

Picking a good research problem is at least half the battle, especially for PhD students.  It’s worth studying why some ideas and approaches are great while others are boring.
