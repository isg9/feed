---
title: The Future of Software System Correctness
url: https://blog.regehr.org/archives/340
published: "2011-01-09T22:52:58Z"
feed: regehr
guid: http://blog.regehr.org/?p=340
---

# The Future of Software System Correctness

A few weeks ago I re-read Tanenbaum et al.’s 2006 article [Can We Make Operating Systems Reliable and Secure](http://www.cs.vu.nl/~ast/publications/computer-2006a.pdf). They begin by observing that it would be nice if our general-purpose operating systems were as reliable as our cars and televisions. Unfortunately, Tanenbaum’s vision is being realized in the worst way: as the amount of software in cars and televisions increases, these products are becoming far more prone to malfunctions resulting from software errors. This begs the question: can we create a future where large, important, software-intensive systems work?

Let’s briefly look at some non-solutions to the software problem. First, we can’t just prove everything to be correct. This is way too expensive and most real systems lack formal specifications. At present, it is not even clear that the correct behavior of large systems can be formalized at all, though hopefully this will be possible someday (exercise for the reader: formalize Asimov’s Three Laws in HOL, Coq, or a similar language). Another non-route to safe software is pervasive use of hardware interlocks to prevent decisions made by software from doing damage. This is not going to happen because interlocks are too expensive and in many cases humans are an inappropriate fallback, for example because they are too slow or not in the right place at the right time. There’s no silver-bullet programming language, certainly; nor does traditional software engineering have the answers. A final non-solution would be to reduce our reliance on software systems. For reasons I think we do not fully understand, the universe wants lots of computation to be embedded in it, and we are going to put it there.

Given that all non-dystopian futures for the human race (and many dystopian ones too) include massive reliance on software-intensive systems, and taking into account the obvious impossibility of eliminating all bugs in these systems, the best we can reasonably hope for is a future where bugs do not have serious real-world impact. In other words, they do not (often) make mistakes that cost a lot of money, damage the environment, or kill a lot of people. Realizing this future is going to require many technological innovations and also we’ll need to significantly rethink the process of creating software systems.

The rest of this piece discusses some of the technical ideas that I believe will be most important during the next 25 years or so. Wherever possible, I’ll provide links to examples of the kinds of thinking that I’m talking about. Obviously the software safety problem has many human aspects; I’m not writing about those.

# Automated Generation of Test Inputs

Whitebox testcase generators that take the structure of the system under test into account have improved greatly in the last few years. One of the best examples is [Klee](http://klee.llvm.org/), which is not only open source but (atypically for research products) is engineered well enough that it works for a broad class of inputs other than the ones tested by the original developers. Drawbacks of whitebox testing tools include:

- the path explosion problem prevents them from testing medium to large systems (the current limit seems to be around 10 kloc)
- they do not support complex validity constraints on inputs
- code living behind hash functions and other hard-to-reverse computations is inherently resistant to whitebox techniques

A different approach is to randomly generate well-formed testcases; the recently announced [cross\_fuzz tool](http://lcamtuf.blogspot.com/2011/01/announcing-crossfuzz-potential-0-day-in.html) and my group’s compiler testing tool are good examples, having found ~100 and ~300 bugs respectively in deployed systems. Random testing of this kind has significant, well-known drawbacks, but it can be extremely effective if the generator is carefully designed and tuned.

Future tools for automatically generating test cases will combine the best features of whitebox and constrained random testing. In fact, [an example combining these features](http://research.microsoft.com/en-us/projects/atg/pldi2008.pdf) already exists; it is really cool work but it’s not clear (to me at least) how to reuse its ideas in more complicated situations. I think it’s fair to say that the problems encountered when mixing whitebox-type constraints and sophisticated validity constraints (e.g. “input is a valid collection of sensor inputs to the aircraft” or “input is a valid C program”) are extremely difficult and it will take a while to work out good solutions. Note that you can’t get around the validity problem by saying something like “well, the system should operate properly for all inputs.” Even if this is true, we still want to spend much of our time testing valid inputs and these typically make up an infinitesimal part of the total input space.

Scalability problems in software testing are inherent; the only possible solutions I can see are those that exploit modularity. We need to create smallish pieces of software, test them thoroughly, and then come up with ways to make the testing results say something about compositions of these smaller units (and this reasoning needs to apply at all levels of software construction). While “unit testing is good” may seem too stupidly obvious to be worth saying, there are many complex software systems with substantial internal modularity (GCC and Linux, for example) whose component parts get little or no individual testing.

# Large-Scale Virtualized Environments

Executable models of complex systems — the Internet, the financial system, a fleet of ships, a city-wide automated driving system — are needed. The [National Cyber Range](http://www.whitehouse.gov/files/documents/cyber/DARPA%20-%20NationalCyberRange_FactSheet.pdf) is an example of an effort in this direction. These environments will never capture all the nuances of the emulated system (especially if it includes humans) but they’re a lot better than the alternative which is to bring up fragile systems that have never been tested against meaningful large-scale events, such as an earthquake or a concerted DoS attack in the case of a city-scale automatic driving system.

Increasingly, software verification tools and testcase generators will be applied at the level of these large-scale emulated environments, instead of being applied to individual machines or small networks. What kind of rogue trading program will maximally stress the stock market systems? What kind of compromised vehicles or broken network links will maximally wig out the automated driving system? We’d sure like to know this before something bad happens.

Years ago I did a bunch of reading about federated simulation environments. At that point, the status seemed to be that building the things was a huge amount of work but that most of the technical problems were fairly straightforward. What I would hope is that as software systems increase in number and complexity, the number of kinds of physical objects with which they interact will grow only slowly. In that case. the economics of reuse will lead us to create more interoperable physics engines, astro- and hydrodynamics packages, SoC simulators, etc., greatly reducing the cost of creating future large-scale simulation environments.

# Self-Checking Systems

What would happen if we created a large system under the restriction that 98% of all system resources had to be used for redundancy, error detection, checkpointing and rollback, health monitoring, and related non-functional activities? Would we think of interesting ways to use all those cycles and bytes? Probably so. Large-scale future systems are going to have to put a much larger fraction of their resources into this kind of activity. This will not be a problem because resources are still rapidly getting cheaper. As a random example, the [simplex architecture](https://agora.cs.illinois.edu/download/attachments/10581/IEEESoftware.pdf) is pretty awesome.

# Independent Failure

Independent failures are at the core of all reliability arguments. A major problem with computer systems is that it is very difficult to show that failures will be independent. Correlated failures occur at different levels:

- all Windows machines may be affected by the same zero-day exploit
- hard drives from the same batch may all fail at around 15,000 hours
- a VMM bug or a bad RAM cell can take out multiple VMs on the same physical platform
- n-version programming is [known to not be a panacea](http://sunnyday.mit.edu/papers/nver-tse.pdf)

Simply put, we need to develop better ways to create defensible arguments about independent failure. This is a tough nut to crack and I don’t have good ideas. I did, however, recently see an interesting talk with some [ideas along these lines](ftp://ftp.cs.york.ac.uk/papers/rtspapers/R:Burns:2010g.pdf) for the case of timing faults.

# Margin for Software Systems

Margin is at the center of all reliable engineered systems, and yet the concept of margin is almost entirely absent from software. I’ve [written about this before](https://blog.regehr.org/archives/50), so won’t repeat it here.

# Modularity

Modularity is the only reason we can create large software systems at all. Even so, I’m convinced that modularity is far from being played out as a source of benefit and ideas. Here are some areas where work is needed.

The benefits that we get from modularity are fewer if the interfaces between modules are the wrong ones. Our interfaces tend to be very long lived (consider TCP/IP, the UNIX system call interface, and x86) and the suboptimality of these is a constant low-level drag on system construction. Furthermore, the increasing size of software systems has made it quite difficult to start from scratch, meaning that even academic researchers are quite often constrained by 30 year-old interfaces. [Eddie Kohler’s work](http://www.cs.ucla.edu/~kohler/pubs/) represents an excellent recent body of work on interface re-thinking at the operating system level.

Big system problems often happen due to unintended interactions between components, which are basically failures of modularity. These failures occur because we pretend to be using abstractions, but we’re actually using pieces of code. Real software tends to not be very modular at all with respect to bugs, timing behavior, or memory allocation behavior. Coming to grips with the impact of leaky abstractions on system construction is a critical problem. Plugging the leaks is hard and in most cases it has to be done one at a time. For example, switching to a type-safe language partially plugs the class of abstraction leaks caused by memory safety violations; synchronous languages like Esterel render systems independent of certain kinds of timing behavior; etc. An alternative to plugging leaks is to roll the leaking behaviors into the abstraction. For example, one of my favorite programming books is Leventhal and Saville’s *6502 Assembly Language Subroutines*. Each routine contains documentation like this:

> Registers Used: All
>
> Execution time: NUMBER OF SHIFTS \* (16 + 20 \* LENGTH OF OPERAND IN BYTES) + 73 cycles
>
> Data Memory Required: Two bytes anywhere in RAM plus two bytes on page 0.

Obviously it is challenging to scale this up, but how else are we going to reason about worst-case behavior of timing-sensitive codes?

Increasingly, some of the modules in a software system, like [compilers](http://compcert.inria.fr/) or [microkernels](http://www.ertos.nicta.com.au/research/sel4/), will have been proved correct. In the not-too-distant future we’ll also be seeing verified device drivers, VMMs, and libraries. The impact of verified modules on software testing is not, as far as I know, well-understood. What I mean is: you sure as hell don’t stop testing just because something got proved correct — but how should you test differently?

# Summary

We have a long way to go. The answers are pretty much all about modularity, testing, and the interaction between modularity and testing.
