---
title: Guidelines for Research on Finding Bugs
url: https://blog.regehr.org/archives/1038
published: "2013-09-21T05:04:04Z"
feed: regehr
guid: http://blog.regehr.org/?p=1038
---

# Guidelines for Research on Finding Bugs

There’s a lot of research being done on finding bugs in software systems. I do some of it myself. Finding bugs is attractive because it is a deep problem, because it’s probably useful work, and because — at least for some people — finding bugs is highly entertaining. This piece contains a few thoughts about bug-finding research, formulated as guidelines for researchers. Some of these are based on my own experience, others are based on opinions I’ve formed by reviewing lots of papers in this area.

## Target Software That’s Already Pretty Good

Not all software is good. In fact, some of it is downright crappy. Here *crappy* is a technical term: it means a code base containing enough serious known bugs that the developers don’t want to hear about any new ones. Finding bugs in crappy software is like shooting fish in a barrel. Crappy software does not need clever computer science: it needs more resources, new management, or simply to be thrown away. In contrast, software that is *pretty good* (another technical term) may have outstanding bugs, but the developers are actively interested in learning about new ones and will work to fix them. This is the only kind of software that should be targeted by bug-finding tools. Software that is *extremely good* is probably not a good target either — but this code is quite uncommon in the real world.

## Report the Bugs

I’m always surprised when I read a bug-finding paper that claims to have found bugs, but does not mention how the people who developed the buggy software reacted the resulting bug reports. The implication in many cases is that the bugs were identified but not reported. It is a mistake to operate like this. First, when handed a bug, software developers will often discuss it — you can learn a lot by listening in. Some bugs found by bug-finding tools aren’t bugs at all but rather stem from a misunderstanding of the intended behavior. Other bugs are genuine but developers aren’t interested in fixing them — it is always interesting to learn how bugs are prioritized. If a bug-finding research tool is not finding bugs that are useful, then the tool itself is not useful. However, perhaps it can be adapted to filter out the bad ones or to find better ones in the first place. In all cases, feedback from software developers is valuable to the bug-finding researcher. Another reason it’s important to report bugs is that fixed software is better for everyone. Obviously it’s better for the software’s users, but it is also better for bug-finding researchers since bugs tend to hide other bugs. Paradoxically, bug-finding often becomes easier as bugs are fixed. Furthermore, it is important to raise the bar for future bug-finding research: if a dozen different papers all claim credit for finding the same bug, this is hardly a success story. A final reason to report bugs is that the very best outcome that you can report in a paper about a bug-finding tool is that it has found bugs that developers cared about enough to fix.

## Prefer Open Source Software as a Bug-Finding Target

In reporting a large number of compiler bugs, I have found that bugs in open source compilers are more likely to be fixed than bugs in commercial tools. One reason, I think, is that people developing commercial compilers have a tendency to spend their precious time supporting paying customers as opposed to humoring obnoxious researchers. Another reason to prefer open source software is that it usually has a public bug reporting system, even if it’s just a mailing list, so you can listen in and perhaps participate when bugs are being discussed. Also, when a bug is fixed you can immediately try out the fixed version. In contrast, it is unusual to be granted access to unreleased versions of commercial software, so you may have to wait six months or two years for a new release, at which point your students have graduated, your grants have expired, and your bug-finding tool might not compile any longer (it definitely won’t compile after six months if your code interfaces with a fast-moving code base like LLVM).

## Think Carefully when a Bug-Finding Tool Doesn’t

Certain codes are too hardened to be good bug-finding targets. For example, when hunting for integer overflow errors we found none in several popular libraries such as libjpeg. The problem is that this kind of code has been intensely scrutinized by security people who spent a lot of effort looking for similar kinds of problems. Other codes don’t contain good bugs because they are too small. Of course, a failure to find bugs doesn’t mean they don’t exist: it might mean that the bug-finding tool is not yet mature enough or it may simply be based on an incorrect premise. I’ve supervised a couple of casual student projects where they wrote fuzzers for the Linux system call API. Although this is clearly a hardened target, I have confidence that bugs would have been found if the students had persisted (they did not).

One trick I’ve seen people play when a bug-finding tool doesn’t find any bugs is to relabel the work as an exercise in formal verification where the desired result is evidence of absence of bugs rather than evidence of their presence. In some cases this may work, but generally speaking most bug-finding tools have many embedded assumptions and unsoundnesses that — if we are honest — cause their verification claims to be weak. Although there is certainly some overlap between bug-finding and formal verification, the latter sort of work is often conducted differently, using different tools, and attacking much smaller pieces of software.

## Conclusion

I feel like this is all pretty obvious, but that it needs to be said anyway. Other than the [superb Coverity article](http://cacm.acm.org/magazines/2010/2/69354-a-few-billion-lines-of-code-later/fulltext), there’s not a lot of good writing out there about how to do this kind of work. If you know of any, please leave a comment.
