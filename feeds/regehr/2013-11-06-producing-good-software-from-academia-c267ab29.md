---
title: Producing Good Software From Academia
url: https://blog.regehr.org/archives/1058
published: "2013-11-06T23:56:26Z"
feed: regehr
guid: http://blog.regehr.org/?p=1058
---

# Producing Good Software From Academia

Writing and maintaining good software from academia isn’t easy. I’ve been thinking about this because last week my student [Yang Chen](http://www.cs.utah.edu/~chenyang/) defended his thesis. While I’m of course very happy for him, I’m also depressed since Yang’s departure will somewhat decimate the capacity of my group to rapidly produce good code. Yang looked over my group’s repositories the other day and it turns out he has committed about 200,000 lines of code while working with me.

Specifically, I’m bummed about not having a very good story for maintaining tools like [Csmith](http://embed.cs.utah.edu/csmith/) and [C-Reduce](http://embed.cs.utah.edu/creduce/). Ironically, I was the original C-Reduce developer and maintained it for several years but then Yang stepped in and wrote 30,000 lines of C++ that made it get much better results — and this work became part of his thesis. However, I’m not super interested in maintaining his code myself: it’s pretty big and pretty hairy and it interacts closely with Clang — a fast-moving project that requires active effort to keep up with. Csmith was never my code base, though I did contribute a little in its early days. The intent was always to throw it away after we figured out the right way to do random program generation. Well, guess what? That goal remains just a goal, and in the meantime it would be nice if we could keep working on Csmith, which continues to be useful to people. Prior to C-Reduce and Csmith my group produced a number of tools that I’d have liked to maintain, but that got abandoned due to lack of development power.

I suspect my situation is a common one for mid-career CS professors who work in systems, software engineering, security, PL, and other engineering-oriented parts of the field. So how should we go about producing good code? Here are some possibilities.

# Forget About Coding

I’ve heard people say that it’s not our job in academia to produce good software. I find that to be a silly position to take. Our job is to do research that has impact. If we need to produce good software to do that job, then producing good software is part of the job. There are plenty of other reasons to write code:

- Ideas without implementations often have gaps and flaws that would have become immediately apparent if an implementation had been attempted.

- When you write code, you explore and reject a large number of program designs; with each rejected choice you learn something. I’m convinced that the cumulative effect of these small pieces of design feedback is very important in developing and maintaining a solid intuition for research.

- Feedback from users is also very valuable; you can’t get users without writing code that is at least moderately usable.

Some researchers are capable of doing top-quality work without these types of feedback. Many more are not.

# Embrace Crappiness

Even when we reserachers do produce software, the code is often released as a versionless tarball with no particular license. The makefile tends to refer to specific paths on the system where the software was developed. There might be a few test cases and if we’re lucky a README. The tarball was uploaded around the time the final copy for some paper was submitted and it has not been updated since then. In all likelihood, the only way to compile it is on some old version of Linux. This version, needless to say, must be determined by trial and error since it’s not mentioned in the README. Is this sounding a little bit familiar? Also see [the CRAPL](http://matt.might.net/articles/crapl/). Of course, a crappy software release is better than none at all, and at least it serves the goal of reproducibility, assuming that compilation challenges can be overcome. Due to sites like Github, the research software has improved at least a bit, but the bit-rotted tarball is far from dead.

I’m not going to pretend that I haven’t repeatedly embraced crappiness. In fact the ability to do so is one of the main perks of being a researcher in the first place. On the other hand, when an idea ends up having legs, the crapware option stops making sense.

# Professor as Hacker

It’s no secret that maybe a third of CS professors are completely inept at writing code. At the other extreme we have people like Xavier Leroy and Matthew Flatt who are principal developers and maintainers of large pieces of high-quality software. When I go to lunch with Matthew he’s always like “Yeah… I really didn’t like the ARM code coming out of [Racket](http://racket-lang.org/) so I rewrote the jitter this morning.” Most of us are in the middle somewhere and as far as I can tell, many of us are bummed out that we have very little time to write code anymore.

So in case that wasn’t clear, the problem with maintaining the code ourselves is that we don’t scale very well. For example, I can and do maintain some small things such as the 2,000-line main body of C-Reduce that is now dwarfed by Yang’s C++, but I can’t keep taking on new tasks.

# Hire Long-Term Research Staff

A time-honored way to produce high-quality software from academia is to run a research empire large enough that it can sustain full-time research stuff over a period of at least a decade. These staffers are people who like academia and are proficient at code craftsmanship, but who lack a desire to run the show.

For many of us, the problem with this plan is that empire builders become managers who spend most of their time acquiring and keeping grant money. The real work gets done via delegation. I recall a time when my advisor in grad school was going to about one PI meeting per month — that cannot have been fun. Empirically, there’s very little overlap between the set of professors who are PIs on major grants capable of supporting long term staff and the set of professors who write code and are otherwise in touch with the low-level details of their operations.

# Leverage Open Source

In this scenario, a research group develops a prototype, open sources it, and then a community of volunteers takes over subsequent development. This is the dream outcome. It could happen, but I don’t think it is very common. We’ve gotten a number of patches for Csmith but most of them have been either relatively simple cleanups or pretty substantial sets of changes that were done to support someone’s custom compiler and that we haven’t been too eager to integrate into our version of Csmith since they would make our own work harder. Csmith isn’t very modular. With C-Reduce, we’ve had better luck — we’ve gotten plenty of good bug reports, a number of patches, and even some new features. But overall, it is not easy to build a new open source community. George Necula, who developed the wonderful [CIL](http://www.cs.berkeley.edu/~necula/cil/) tool, once complained that CIL had a lot of users but very few were contributing back. Maintenance of CIL was eventually taken over by volunteers, though it may have now stagnated a bit — LLVM has moved into this research niche in a big way.

# A Chain of Students

A final alternative is to try to recruit students who have, or can acquire, strong code craftsmanship skills and then put some of the code development and maintenance burden on them. In this model a new student spends some time developing and maintaining existing codes before, and in conjunction with, working on her own projects.

As I said at the top of this piece, this is a strategy that I have used to some extent. It sort of works but has various problems. First, a very light touch is required: it isn’t fair to burden students with too much work that does not contribute to their own progress towards a degree. Second, even PhD students aren’t around for all that long, in the larger scheme of things. Third, not all students are interested in or capable of maintaining large codes, or of writing code that is worth maintaining.

# Conclusion

In general, academia is setup to reward quick projects that result in a few papers and maybe a few tarballs linked to grad students’ web pages. Long term development and maintenance of code requires either heroic effort by the PI or else big funding.

A possibility that I deliberately left out of this piece since it isn’t “producing good software from academia” is spinning off a company. I have a huge amount of respect for academics who form startups, but this option seems so invasive in terms of overall lifestyle that I haven’t seriously considered it yet.
