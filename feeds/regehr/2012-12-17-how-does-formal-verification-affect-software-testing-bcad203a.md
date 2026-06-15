---
title: How Does Formal Verification Affect Software Testing?
url: https://blog.regehr.org/archives/742
published: "2012-12-17T04:26:03Z"
feed: regehr
guid: http://blog.regehr.org/?p=742
---

# How Does Formal Verification Affect Software Testing?

This has been a difficult piece to write and I’ve already deleted everything and started over more than once. So, I’m going to take the easy way out and structure it as a sequence of questions and answers.

**What does *formal verification* mean?** Something like “using mathematical techniques to convincingly argue that a piece of software implements a specification.” The specification may be very domain-specific, as in “the robot may not injure a human being or, through inaction, allow a human being to come to harm,” or it may be generic, as in “no execution of the system dereferences a null pointer.” So far, we know extremely little about what might be involved in verifying a specification such as [Asimov’s Laws](http://en.wikipedia.org/wiki/Three_Laws_of_Robotics) that identifies entities in the real world, as opposed to entities inside the software system.

**Do we need both testing and formal verification?** Yes. It’s absurd to think we could do without testing. Testing is how we create systems that work. On the other hand, testing alone suffers from needle-in-haystack problems where critical software almost certainly has surprising behaviors that we need to know about, but we don’t know how to create test inputs that trigger them.

**Why would testing and formal verification interact at all? Aren’t these separate activities?** Today, testing and formal verification are largely independent activities. I’ve heard of a very small number of cases where formal verification is considered an acceptable substitute for running some kinds of tests. In the long run we can hope to see more of this.

**Under what conditions can formal verification reduce the amount of testing that is required?** This is difficult. The basic observation is that formal verification tends to eliminate the possibility of certain types of bugs in a narrow layer of the system—the one that is being verified. To be concrete, let’s say that I prove that my robot software faithfully follows Asimov’s laws, but then I compile it with a buggy compiler, run it on a buggy operating system, run it on a buggy processor, or run it inside a robot with a defective camera that cannot reliably recognize human beings. This robot, of course, will fail to follow Asimov’s laws; the proofs about its software are rendered irrelevant by violated assumptions. These problems need to be discovered via testing (which, of course, may also fail to reveal them). To make formal verification apply to a wider range of layers, we need verified operating systems, compilers, libraries, chips, and more. Of course, people are working on all of these things, though at present the chaining together of correctness proofs for a realistic software stack has not yet happened.

Another problem with formal verification is that the specifications themselves are often quite complicated and difficult to debug; gaining confidence in a specification requires a testing process not very different from how we gain confidence in code. Even if we have non-buggy formal specifications, it can be unclear what the specification means in practice, and it can be unclear that the specification describes all of the behaviors that we might care about. Usability considerations are an example of a property of software that we care about, that can be tested, but that may not be possible to formally specify and verify.

But let’s get back to the question: Under what conditions can formal verification replace testing? My answer would be:

- The formal verification tool is considered to be trustworthy.
- The specification is considered to be trustworthy.
- The specification is considered to capture all properties of interest.
- Layers of the system not being verified (compiler, OS, etc.) are considered to be trustworthy.

Of course, it may be difficult to satisfy all of these criteria. In contrast, testing makes many fewer assumptions, but rather provides end-to-end checks of the compiler, the OS, the code being developed, etc.

**Why do we want formal verification to replace some kinds of testing?** The reason is simple: cost. Although formal verification is currently quite expensive, techniques are improving every year. Testing techniques are also improving, but not necessarily at a very rapid rate. The fundamental problem with testing is that we can’t do a very good job of it in any realistic amount of time. The great hope of formal verification is that once the large initial investment is made to get it going, incremental re-verification will be relatively inexpensive, permitting high-quality software to be delivered rapidly. As the technologies improve, even the initial costs are likely to come down. In the long run, reduction in cost, not reduction in defects, will be the killer app for formal methods.

**How about the other way around: When can testing replace formal verification?** Testing all possible behaviors of a system *is* formal verification; it is a kind of proof by exhaustion. Unfortunately, exhaustive testing is only feasible in very narrow circumstances.

**Can testing and formal verification interact in other ways?** Definitely, and I think the possibilities here are very exciting. We are already seeing excellent systems like [Klee](http://klee.llvm.org/) and [SAGE](http://research.microsoft.com/en-us/um/people/pg/public_psfiles/talk-issta2010.pdf) where formal methods are used to produce test cases. In the future we will start to see patchwork verification efforts where part of the code is verified to be correct, part of it is verified only to be free of certain bugs, and part of it is not verified at all. At that point we will need Klee-like tools that take these different levels of verification into account and generate test cases that maximally increase our confidence in the correctness of the unverified code. Even a fully formally verified software stack may contain some surprises, for example discontinuities in the functions it implements or perhaps unwarranted sensitivity to changes in input variables. We want test case generators for these too.
