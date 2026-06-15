---
title: Testing Commercial Compilers
url: https://blog.regehr.org/archives/572
published: "2011-08-08T16:33:18Z"
feed: regehr
guid: http://blog.regehr.org/?p=572
---

# Testing Commercial Compilers

A few weeks ago a reader left this comment:

> Just out of curiosity John, have you approached any of the big commercial compiler companies about getting free licenses for their products? I don’t work in the compiler business but if a university research time offered to rigorously test my software, for free, I’d say yes. You can always ignore bug reports after all.

I wanted to devote a post to the answer since it’s something I’ve spent a fair amount of time thinking about. Let’s look at the tradeoffs.

# Pros of Testing Commercial Compilers

The whole reason I started working on this was the fact that bugs in commercial compilers for embedded systems were irritating me and the students in my embedded systems courses. So of course I want these compilers (and “regular compilers” too) to be more correct.

Finding bugs in commercial compilers benefits my project by showing that our work is generally useful. If we only found bugs in open source compilers, people could just explain that these tend to be much buggier than commercial tools. This is false, though it doesn’t stop people from occasionally saying it anyway; for example, see [Chris Hills’ posts here](http://www.velocityreviews.com/forums/t749725-compiler-optimizing-wrong.html). In any case, as a proof of concept it is important for Csmith to find bugs in the highest-quality compilers that are available, regardless of who produces them.

# Cons of Testing Commercial Compilers

First, reporting compiler bugs is hard work. It’s not my job to improve a product that someone else is making money by selling. If someone wants me to do this, they can hire me as a consultant. If they want to do the work on their own, [Csmith is open source](http://embed.cs.utah.edu/csmith/) under a corporation-friendly license.

Second, the people producing these compilers may not have any interest in having me test it. There is, in fact, a potentially serious downside: if I publicize the fact that I found a lot of bugs in safety-critical compiler A, then the people producing compiler B can use this as a selling point. I’ve noticed that sometimes people steeped in the open source world have a difficult time believing this is a serious concern, but it is.

Third, random testing works best when the bug reporting / fixing feedback cycle is rapid. Speed is important because high-probability bugs tend to mask lower-probability bugs. As a tester operating outside of the company, I’m tied to a months-long release cycle as opposed to just updating to the latest revision every morning.

Fourth, commercial tools have a closed process. I can’t see the full set of bug reports, I can’t access the source code or commit log, and perhaps I cannot even mail the developers directly.

Fifth, commercial compilers are often a pain. They probably use the despicable FlexLM. They often run only on Windows. They probably have obscure command-line options. They may target platforms for which I don’t have good simulators.

Finally, the purpose of a commercial compiler is simple: it is to make money for the company selling it. Given finite engineering resources, a company may well be more interested in supporting more platforms and improving benchmark scores than it is in fixing bugs. If they do fix bugs, they’ll begin with the ones that are affecting the most valuable customers.

# Conclusion

In contrast with the points above, acting as a test volunteer for open source compilers is a pleasure. I can see the code and directly interact with developers. Bugs are often fixed rapidly. More important, when these compilers are improved it benefits everyone, not just some company’s customers. I feel a substantial sense of obligation to the open source community that produced the systems I use every day. It seems crazy but I’ve had a Linux machine on my desk continuously for a bit more than 19 years.

I’ve tested a number (>10) of commercial C compilers. Each of them has been found to crash and to generate incorrect output when given conforming code. This is not a surprise — compilers are complicated and all complex software contains bugs. But typically, after this initial proof of concept, I stop testing and go back to working on GCC and LLVM. I’ve heard through the grapevine that Csmith is in use at several compiler companies. The engineers doing this have not shared any details with me and that is fine — as long as the bugs are getting fixed, my project’s goals are being met.
