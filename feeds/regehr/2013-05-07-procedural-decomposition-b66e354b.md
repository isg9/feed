---
title: Procedural Decomposition
url: https://blog.regehr.org/archives/942
published: "2013-05-07T02:51:51Z"
feed: regehr
guid: http://blog.regehr.org/?p=942
---

# Procedural Decomposition

While teaching a CS class I spend quite a bit of time looking over the shoulders of students whose code doesn’t work. Sometimes they have a simple mistake and I’ll either point it out or ask a question that will lead them to the problem. However, other times the code is just generally not very good and I’ve always sort of struggled to find the right things to say; it’s hard because it’s usually a matter of judgement and taste.

Recently I’ve started to notice a pattern where students who are having trouble getting their code to work often have not done enough *procedural decomposition*. That is, they have failed to appropriately break up a large, difficult programming problem into a collection of smaller ones. You are probably thinking to yourself that this is pretty basic stuff, and perhaps it is when we’re solving easy problems. On the other hand, I’ve come to believe that procedural decomposition for hard problems is probably one of those skills that we can keep getting better at for as long as we care to keep writing code.

# An Example

In my operating systems class this spring, I asked the students to do a bit of hacking on Linux’s driver for the Minix filesystem. The Minix FS is a reasonable starting point because it’s about as simple as realistic filesystems get. The assignment was to change the code so that instead of representing a file as a tree of disk block numbers, a file is represented as a list of extents. An extent is a collection of contiguous disk blocks that can be represented as a (base, length) pair. For purposes of this assignment, we make the unrealistic assumption that indirect extents are not needed.

The crux of this assignment was rewriting the Minix FS driver’s get\_block() function, which finds a block of a file on the disk and then maps it into the block cache; this is nontrivial because the requested block must be created if it doesn’t already exist (and a “create” flag is set). My first attempt at get\_block() — as well as many students’ first attempts — was a monolithic function. However, the logic turns out to be just hairy enough that this is difficult to get right. On the other hand, the job can be broken down into several pieces:

- tiny utility functions for packing and unpacking the (base, length) pairs that are used to represent extents on disk
- a lookup function that finds a block of a file in the extent list if it exists, and fails otherwise
- an “extend file” function that makes a file bigger either by one block or by a specified number of blocks

Once these are available, get\_block() needs only about a dozen lines of code. The tricky thing isn’t figuring out this particular decomposition — which is kind of obvious — but rather recognizing that it’s time to think about breaking up the problem rather than attacking it all at once. The temptation to keep banging on a function that seems to be almost correct is strong.

# What’s Going On?

I asked some colleagues: Are we teaching procedural decomposition? And are we doing a good job? Well, of course we are teaching this and in fact one of the people who does that here, Matthew Flatt, is an author of [How To Design Programs](http://htdp.org/), one of the best books I know of for teaching abstraction and decomposition (and it’s free). But even so, the fact remains that in upper-division classes I’m getting a good number of students who are not performing procedural decomposition even in a situation where it is clearly necessary.

Since I’ve started thinking about procedural decomposition more, I’ve noticed that I don’t do enough of it either. When I catch myself writing poorly abstracted code, I try to analyze what I’m thinking and usually it boils down to a mixture of laziness (“if I can just get this stupid function to work, I won’t have to figure out how to break it up into pieces”) and overconfidence in my ability to deal with a complex case analysis all at once. I’m guessing that similar things are going on in students’ heads.

The purpose of the rest of this piece is to take a closer look at procedural decomposition and how we might be able to do a better job teaching it. But first, a quick note about terminology: the thing I’m calling procedural decomposition is also sometimes referred to as procedural abstraction, functional abstraction, or functional decomposition.

# Reuse Isn’t the Main Thing

I think a lot of us were taught that the main reason we break code into functions is to facilitate reuse. In this view, we primarily write quicksort as a separate function so that we can call it from multiple locations in our program. It’s usually not too hard to notice when this kind of decomposition is necessary: we just get tired of writing quicksort from scratch after the second or third time. In contrast, procedural decomposition of a single problem is more difficult because it’s not as obvious where to introduce abstraction.

# Decomposition and Problem Difficulty

I sometimes notice myself doing a much nicer job of procedural decomposition when solving simple problems than when solving complex ones. Of course if I were programming rationally it would be the other way around. Perhaps this effect can be understood by saying that I have a fixed amount of brain power to devote to a programming problem. In this case, when I’m totally overwhelmed by complexity I have no resources left for the added burden of figuring out how to best perform procedural decomposition. Of course, once I step back, delete some code, and find a good decomposition, I can surmount some sort of complexity barrier and probably find a nice landscape of simple code on the other side.

Years ago I heard Randy Pausch gave a talk describing how people who tried to fly a virtual magic carpet were overwhelmed by the many degrees of freedom offered by a prototype version of the controls. In order to simplify the situation, they left the carpet’s throttle at the maximum setting. This reduced the degrees of freedom by one, but with the unfortunate side effect of causing them to careen stupidly through the game running into everything. It seems like this might be a good analogy for how many of us approach a difficult programming problem.

# Decomposition and Language Choice

There’s clearly a link between procedural decomposition and programming language choice. For example, I learned to program in early microcomputer BASIC dialects (TI BASIC, GW-BASIC, and Applesoft BASIC) where the support for procedural abstraction was not even as good as it is in assembly language. I sometimes fear that spending a few formative years writing BASIC may have stunted my capacity for abstraction.

One of the things that made the old BASIC implementations such turds is that they had no support for creating multiple namespaces. C is better, but the lack of nested functions and multiple return values can make it hard to perform elegant procedural decomposition there. C++ and Java don’t support nested functions either, but the object model seems to approximate them well enough for practical purposes. Modern languages with tuples, nested functions, and first-class functions seem to offer the best support for procedural decomposition.

Historically, we were discouraged from performing too much procedural decomposition because function calls added overhead. C/C++ programmers have been known to write horrific macros in order to get decomposition without increasing runtime. The modern wisdom — regardless of programming language — is that any performance-oriented compiler will do a pretty good job inlining functions that are small and that don’t have a ton of call sites. When the compiler has good dynamic information, such as a JIT compiler or an AOT compiler with access to profile data, it can do a considerably better job. However, even an AOT compiler without profile data can often do pretty well given good heuristics and perhaps some hints from developers. In any case, the days of allowing performance considerations to influence our procedural decomposition are mostly over.

# Teaching Procedural Decomposition

I think that too often, our programming assignments are so canned that a solid grasp of procedural abstraction isn’t necessary to solve them. Moreover, we tend to not do a great job of grading on code quality — and of course the students are allowed to throw away their code after handing it in, so they have little incentive to do real software engineering. Of course there are plenty of students who write very good code because they can, and out of a general sense of pride in their work. But they’re not the ones I’m talking about trying to help here.

Maybe something we need to do is help students learn to recognize the danger signs where the complexity of a piece of code is starting to get out of control. Personally, I get sort of a panicky feeling where I become more and more certain that each fix that I add is breaking something else. I hate this so much that it’s making my skin prickle just writing about it. Usually, the best thing to do when this happens is step back and rethink the structure of the code, and probably also delete a bunch of what’s already written.

Finally, we probably need to do more [code reading](https://blog.regehr.org/archives/940) in classes. This is difficult or impossible for a large class, but probably can be done well with up to 10 or 15 students. As a random example, the Minix FS module for Linux kernel contains a rather nice decomposition of the functionality for [managing the block tree for each file](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/fs/minix/itree_common.c?id=refs/tags/v3.9).
