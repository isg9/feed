---
title: Academic Bug-Finding Projects in the Long Run
url: https://blog.regehr.org/archives/719
published: "2012-05-30T03:45:48Z"
feed: regehr
guid: http://blog.regehr.org/?p=719
---

# Academic Bug-Finding Projects in the Long Run

A number of computer science researchers, including me, have made careers out of creating tools that automate, at least partially, the process of finding bugs in computer programs. Recent work like this can be found in almost any good systemsy conference proceedings such as SOSP, OSDI, ASPLOS, ICSE, or PLDI. Examples go back many years, but a good early-modern paper in this mold is “WARLOCK–A Static Data Race Detection Tool” from USENIX 1993.

How do bug-finding papers get published? Two things are required. First, bugs should be found. A few bugs is fine if they are high-value, deep, or otherwise interesting, but it’s also ok to find a large number of garden variety bugs. It’s not required that the bugs are turned into bug reports that are well-received by the people who maintain the buggy software, but that is definitely a plus. It’s not even required that a bug-finding tool (or algorithm) finds any bugs at all, though I personally don’t have much use for that sort of paper. The second thing that is required is some level of novelty in the bug-finding tool. For example, the novel thing about [Csmith](http://embed.cs.utah.edu/csmith/) was taking undefined and unspecified behavior seriously in a program generator and, in particular, the way that it interleaves program generation and static analysis.

So now you know how it works: someone creates a tool, finds some bugs, writes a paper, and that’s that. The problem, as I’ve increasingly come to realize, is that this cycle is not as useful as we would hope. For example, let’s take one of my tools, [IOC](http://embed.cs.utah.edu/ioc/) (here I’m using “my” in a bit of a broad sense: I wrote very little of IOC, though I did cause it to be created). We created the tool, found some integer overflow bugs, wrote a paper. At this point, if I want to maximize things about me that can be measured (papers, citations, grants, etc.), then I’ll stop right there and move on to the next project. But this is a problem — at the time my paper is published, I’ll have only found and reported a tiny fraction of the integer overflow bugs lurking in all the C/C++ codes that are out there. Furthermore, even if I find and report, for example, all of the integer overflows in the Perl interpreter or the Linux kernel, it’s not like these code bases are going to stay clean. Overflow mistakes are so easy to make that developers will just add them back in unless they get frequent feedback from a tool. So, how can we extract more benefit from a tool?

- One possibility is that my actual tool doesn’t matter since people will read the paper and implement similar techniques. From a selfish point of view this is the best case since my work has impact without my doing any additional work, but given the flood of CS papers, it’s a lot to expect that busy people in the real world will re-implement all of the useful bug-finding work that’s out there.
- If I open-source my tool, people can use it. However, the bar is pretty high if (as is the case for IOC) people have to build a patched Clang, build their project using the new compiler and some special flags, run some tests, and then interpret the tool’s output. Some academic bug-finding tools are easier to use than this (and in fact IOC will be easier to use if we can get it included in LLVM), some are harder, and many are impossible since the source is not available or the tool is effectively unusable.
- If I create a company based around my tool (think [Coverity](http://www.coverity.com/)) then it becomes widely available, but so far I haven’t been interested in facing the big lifestyle change that comes along with a startup. Also, I believe that the products of publicly funded research should be open source.

The point is that the hit-and-run style of bug-finding research is depressingly inadequate as a way of improving software quality in the long run. We need to do better.
