---
title: Writing Solid Code
url: https://blog.regehr.org/archives/834
published: "2012-10-30T16:20:50Z"
feed: regehr
guid: http://blog.regehr.org/?p=834
---

# Writing Solid Code

After 10 short years as a university-level CS instructor, I’ve finally figured out the course I was born to teach. It’s called “Writing Solid Code” and covers the following topics:

- Testing—There are lots of books on software testing but few that emphasize the thing I need students to learn, which is simply how to break code with minimum effort. There’s no book at all that covers fuzzing in a useful way (though [Alex](http://eecs.oregonstate.edu/people/groce) plans to write it). [Lessons Learned in Software Testing](http://www.amazon.com/Lessons-Learned-Software-Testing-Kaner/dp/0471081124/) is one of the better books I know of.
- Debugging—I covered [my view of debugging](https://blog.regehr.org/archives/199) here a while ago and also plan to borrow material from nice books by [Zeller](http://www.amazon.com/Why-Programs-Fail-Second-Systematic/dp/0123745152/), [Agans](http://www.amazon.com/Debugging-Indispensable-Software-Hardware-Problems/dp/0814474578/), [Butcher](http://www.amazon.com/Debug-Repair-Prevent-Pragmatic-Programmers/dp/193435628X/), and [Metzger](http://www.amazon.com/Debugging-Thinking-Multidisciplinary-Approach-Technologies/dp/1555583075/).
- Assertions—I try to not be too dogmatic about stuff but CODE HAS TO HAVE ASSERTIONS. That is all there is to it. I’m in the middle of a long post about this and hope to publish it sometime in November.
- Medium-weight formal methods—Probably using [Frama-C](http://frama-c.com/) to statically verify that assertions hold and language rules (array bounds etc.) are not violated on any path through the code.
- Code reviews—I’ve been reading [Wiegers](http://www.amazon.com/Peer-Reviews-Software-Practical-Guide/dp/0201734850/) but realistically I think the main thing is just to do a lot of code reviews.

Ideally this course would not be needed. Rather, the material would be implicit in the contents of lots of other courses and students would naturally acquire strong skills in defensive programming, testing, debugging, and code reviews. We do not appear to be doing a very good job of encouraging these skills to develop. Rather, we’re giving the students a four-year sequence of little fire-and-forget programming assignments with zero focus on code quality and where the code is thrown away after the due date. I believe this is doing them a disservice.

In my embedded software class this fall I’ve been trying a different approach where the students are working in groups and testing their own, and each others’, code in multiple rounds, and we’re doing some lightweight code reviews in class. This has sucked up a lot of time but I’m hoping that it ends up being worthwhile as a way to force students to keep revisiting and revising their code.

I’d appreciate pointers, suggestions, etc.
