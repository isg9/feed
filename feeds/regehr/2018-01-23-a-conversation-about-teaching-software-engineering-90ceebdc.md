---
title: A Conversation about Teaching Software Engineering
url: https://blog.regehr.org/archives/1569
published: "2018-01-23T18:22:40Z"
feed: regehr
guid: http://blog.regehr.org/?p=1569
---

# A Conversation about Teaching Software Engineering

For better or worse, my impressions of software engineering as a field were shaped by a course I took as an undergrad that I thought was mostly not very interesting or useful. We spent a lot of time on waterfalls and stuff, while not covering testing in any detail. For the final project in the class we had to develop an application using a [CASE tool](https://en.wikipedia.org/wiki/Computer-aided_software_engineering) (very hip at the time) where we described the class hierarchy using a GUI and then the tool generated skeletal C++ for us to fill in. Since we knew nothing about designing class hierarchies and also the tool was weird and buggy this all went about as disastrously as you would expect. In the end I learned quite a lot, but the lessons were probably not those intended by the instructor.

24 years later I’m teaching a software engineering class — this probably wouldn’t even happen if my department had any real software engineering faculty! Even so, I’m a true believer: I love the material and feel more strongly about its importance than I do about my more usual subjects like compilers and operating systems. I ignore software process and focus entirely on building skills and habits that I feel will come in handy in any software engineer’s career. If you like it, put a test on it. Read code. Review code. Refactor code. Write assertions. Adhere to coding standards. Design an API. Use a coverage tool, a bug-finding tool, a version control tool, a fuzz tool, a CI tool, to good effect. Repeat until end of semester. No doubt there’s room for improvement but the material seems solid.

Over Christmas break I had a beer with [Daniel Dunbar](http://minormatter.com/zr/), who I should have met long before, but somehow hadn’t. Daniel has done super impressive stuff: he was one of the original [Klee](https://klee.github.io/) authors and also was an early Clang implementer. I told him about my approach to teaching software engineering and picked his brain a bit about the sorts of things he wished people with CS degrees were better at doing. Of course I wasn’t taking notes and forgot most of it. So I mailed:

As I mentioned the other week, I’m teaching a software engineering course this semester, and rather than focusing on any kind of academic approach to this subject, I’ll try to teach them a lot of the real world business of making software that works, starting with all the basics like testing, coverage, assertions, and code reviews.

But you also mentioned some things that I agreed with but that I might not have thought of, or prioritized — the things that you wish people were already good at when you interview them, but they probably aren’t. Is there any chance I could get you to very briefly summarize those things, or point me to a resource you like on this subject? I want to make sure to cover, at least quickly, all of the main high points in this class.

Daniel graciously sent a long reply and also allowed me to reproduce it here:

To start with, I don’t think I know of any resources on the subject. It’s amazing how much obvious little stuff one has to know, but forgets about. Git is a huge source of “things I use every day but forgot I had to learn”, for better or worse.

I guess if I had to come up with a list off the cuff:

1. The experience of maintaining software over time. I think we spend most of our time working with existing code bases and figuring out how to integrate changes into them. This area has a lot of related topics:

   - How do you figure out where to make a change? Tools here include debugging existing workflows to find where something happens, code search, git blame, git grep, git log -G, etc

   - How do you manage making incremental changes? I am a huge believer in always doing incremental work. How do you build a feature while always keeping the code working? Tools here include feature flags, adaptors and stub implementations, forwarding implementations, A/B testing before/after change.

   - How do you find the source of regressions? Tools here are basically bisection, git log -G.

   - How do you handle technical debt? What counts as technical debt? What kinds of debt are painful versus not?

I feel like I probably have read things I liked on these topics, but none of the links are coming to mind now.

2. The experience of making technical decisions. This is a \*huge\* part of development. Topics here:
   - How do you evaluate choices for a dependency? Tools here include benchmarking, analysis of the code, analysis of the software maintainability, etc

   - How do you evaluate when to adopt a dependency versus write your own? Topics here are NIH versus opportunity cost on innovation.

   - How do you convince people to follow a particular choice? Presenting or writing coherent write-ups on engineering tradeoffs is a really under appreciated skill which can have a big impact (the cost of bad decisions are high).
3. How do you debug things? Another big part of development. I would emphasize debug here not just in the “how do I get this working” but even deeper in the “how do I understand what is really happening?”
   - I find that many people tend to give up at really understanding what is going on. “Oh XYZ didn’t work, so I did PDQ'”. The people who don’t give up usually end up understanding a lot more about computers, and then do a better job maturing over their career.

   - Maybe it would be good to teach people (a) don’t give up, and (b) here are all the tools you can use when you might want to give up. A lot of the time the tool people know is stackoverflow, but past that they are lost.

   - Things here include hardware watchpoints, reading the source code, disassembly and reverse engineering.
4. How do you estimate the time to develop software? This is a huge part of a business, people will always want you to do this. Even just getting students to start to think about the process would be good, asking them to make estimates and compare results to them.
   - I have no advice on how to teach this because I still learning a lot here.
5. How do you review code? What makes good review?
   - When is coding style important versus not? What are the pros/cons?

   - Does review catch bugs, or not? Are certain review styles more effective?
6. How do people do release management? This is such an amazingly huge part of what we spend time on, and one that receives very little attention.
   - Do you release from trunk? If so, how do you ensure quality?

   - Do you have stable release branches? If so, how do you ensure bugs are actually fixed? Are people cherry picking fixes? What can go wrong? (I remember once cherry picking a fix that happened to merge incorrectly, but the patch applied an in a way that was still valid C++ code (an if {} clause ended up inside another one). The result was a clang that miscompiled itself.)

   - How do you deal with complicated merge conflicts?

   - People like Nicole Forsgren have research here which it would at least be nice for people to be exposed to.

If I had to come up with the kinds of potential exercises I would love it if someone was trying (no idea if they actually would work):

1. Take a bug in a complex code base at some revision, and ask people to find it and fix it. Compare answers to the one the project actually adopted. A bug where there were several obvious fixes with tradeoffs would be a good point for discussion.

2. Take a new feature which was added to some project, and study how it was done. Not to toot my own horn–its just an example I am familiar with–we migrated Clang to go from producing .s files to doing the assembly in memory. This involved lots of incremental refactoring, a clear switch over at one point (feature flag), A/B testing to compare old to new (i.e. we chose to shoot for binary equivalence of .o files, simply so we could easily test it — made for lots of extra engineering, but easier to guarantee correct). One could dig up how this went from a theoretical concept on a mailing list to an implemented feature.

The best thing here would be to create a hypothetical project which is a mirror of a real project (but don’t make this clear at first) and ask people to design some extension of it. Then, compare the results to what the project actually decided to do, and the discussion around it. For example, analyze what things the project owners antagonized over that the students didn’t think of, and try and figure out why not.

3. Do some systematic study of PRs as a “literature” exercise. How does tone impact response, how do people handle criticism, etc.

4. Do an exercise where teams are forced to produce a project over the lifetime of the course. The exercises should be small and easier, but they have to be built into the same code base. Something that forces people to make software releases would be nice (dunno if there is a way to do this in such a way that people can choose whether or not to use “release branches”, but in a way that has tradeoffs in either direction).

Wow, this is a lot of material, way more than we cover in depth in a semester. Even so, I find it super useful as a high-level vision for the kinds of things students should end up at least having been exposed to.
