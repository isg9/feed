---
title: Self-Checking Projects
url: https://blog.regehr.org/archives/181
published: "2010-06-23T15:32:45Z"
feed: regehr
guid: http://blog.regehr.org/?p=181
---

# Self-Checking Projects

Matching students up with research projects is entertaining but difficult. The project has to be at the right level of difficulty, has to fit the student’s time frame, and has to interest the student. If grant money is going to be used to pay the student, the work has to fit into the funded project. This is all pretty basic, but slowly I’ve come to recognize a less obvious criterion that can make the difference between success and failure. Some projects have the property of being self-checking: a student can easily, on her own, check whether the implementation parts of the project are correct. For example:

- If a tool that is supposed to find bugs in other programs finds some real bugs, we know that it works, at least to some extent (we can determine if a reported bug is real by inspection). If the tool fails to find real bugs, we can always fall back to seeding software with deliberate errors and try to find them.
- A compiler is highly self-checking because we can ask it to compile large existing programs and see if the resulting executables work as expected.

On the other hand:

- Projects whose top-level result is numerical tend to be very hard to check. I once heard a horror story from a professor who had published a paper based on a student’s work showing that his new algorithm performed much better than the previous best one. Unfortunately, the paper was totally wrong: the student had made an error while implementing the competing algorithm, hurting its performance considerably.
- A formal program verification project is not self-checking because it’s very easy to accidentally prove the wrong properties, or even vacuous properties.

Of course, not all projects need to be self-checking. It’s just that those that aren’t require significant extra attention from the advisor. Additionally, there are some students who will just fail to make progress on a project that isn’t self-checking. It is not exactly the weakest students who do this, but students who somehow lack a strong innate sense of technical right and wrong. In summary, it is useful to ask whether a research project is self checking, and to use the answer to this question to help decide whether the project is a good match for a particular student.

The self-checking principle can be applied to undergraduate education. A course I taught several years ago came with automated test suites that permitted students to determine their score for each homework assignment before turning it in. This approach makes students happy and it is a good match for very difficult assignments that people would otherwise fail to complete. However, when designing a course myself, I have tended to create easier assignments that are not self-checking based on the idea that this encourages students to ask themselves whether they’ve done the right thing–an important skill that is unfortunately somewhat under-developed in CS undergraduates. The weaker half of the students hate this, but it builds character.
