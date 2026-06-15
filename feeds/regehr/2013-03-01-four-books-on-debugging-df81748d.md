---
title: Four Books on Debugging
url: https://blog.regehr.org/archives/849
published: "2013-03-01T05:39:49Z"
feed: regehr
guid: http://blog.regehr.org/?p=849
---

# Four Books on Debugging

It often seems like the ability to debug complex problems is not distributed very evenly among programmers. No doubt there’s some truth to this, but also people can become better over time. Although the main way to improve is through experience, it’s also useful to keep the bigger picture in mind by listening to what other people have to say about the subject. This post discusses four books about debugging that treat it as a generic process, as opposed to being about Visual Studio or something. I read them in order of increasing size.

### ![](http://ecx.images-amazon.com/images/I/519xzqhWaLL._SL500_AA300_.jpg)Debugging, by David Agans

At 175 pages and less than $15, this is a book everyone ought to own. For example, [Alex](http://web.engr.oregonstate.edu/~alex/) says:

> Agans actually sort of says everything I believe about debugging, just tersely and without much nod in the direction of tools.

This is exactly right.

As the subtitle says, this book is structured around [nine rules](http://debuggingrules.com/debuggingrules.jpg). I wouldn’t exactly say that following these rules makes debugging easy, but it’s definitely the case that ignoring them will make debugging a lot harder. Each rule is explained using real-world examples and is broken down into sub-rules. This is great material and reading the whole thing only takes a few hours. I particularly liked the chapter containing some longer debugging stories with specific notes about where each rule came into play (or should have). Another nice chapter covers debugging remotely—for example from the help desk. Us teachers end up doing quite a bit of this kind of debugging, often late at night before an assignment is due. It truly is a difficult skill that is distinct from hands-on debugging.

This material is not specifically about computer programming, but rather covers debugging as a generic process for identifying the root cause of a broad category of engineering problems including stalled car engines and leaking roofs. However, most of his examples come from embedded computer systems. Unlike the other three books, there’s no specific advice here for dealing with specific tools, programming languages, or even programming-specific problems such as concurrency errors.

[![](http://images.betterworldbooks.com/193/Debug-It-9781934356289.jpg)](http://www.amazon.com/Debug-Repair-Prevent-Pragmatic-Programmers/dp/193435628X/)

### Debug It! by Paul Butcher

Butcher’s book contains a large amount of great material not found in Agans, such as specific advice about version control systems, logging frameworks, mocks and stubs, and release engineering. On the other hand, I felt that Butcher did not present the actual process of debugging as cleanly or clearly, possibly due to the lack of a clear separation between the core intellectual activity and the incidental features of dealing with particular software systems.

![zeller](https://blog.regehr.org/wp-content/uploads/2013/01/zeller.jpg)

### Why Programs Fail, by Andreas Zeller

Agans and Butcher approach debugging from the point of view of the practitioner. Zeller, on the other hand, approaches the topic both as a practitioner and as a researcher. This book, like the previous two, contains a solid explanation of the core debugging concepts and skills. In addition, we get an extensive list of tools and techniques surrounding the actual practice of debugging. A fair bit of this builds upon the kind of knowledge that we’d expect someone with a CS degree to have, especially from a compilers course. Absorbing this book requires a lot more time than Agans or Butcher does.

### [![debugging_by_thinking](https://blog.regehr.org/wp-content/uploads/2013/02/debugging_by_thinking.jpeg)](https://blog.regehr.org/archives/849/debugging_by_thinking) Debugging by Thinking, by Robert Metzger

I have to admit, after reading three books about debugging I was about ready to quit, and at 567 pages Metzger is the longest of the lot. The subtitle “a multidisciplinary approach” refers to six styles of thinking around which the book is structured:

- the way of the detective is about deducing where the culprit is by following a trail of clues

- the way of the mathematician observes that useful parallels can be drawn between methods for debugging and methods for creating mathematical proofs

- the way of the safety expert treats bugs as serious failures and attempts to backtrack the sequence of events that made the bug possible, in order to prevent the same problem from occurring again

- the way of the psychologist analyzes the kinds of flawed thinking that leads people to implement bugs: Was the developer unaware of a key fact? Did she get interrupted while implementing a critical function? Etc.

- the way of the computer scientist embraces software tools and also mathematical tools such as formal grammars

- the way of the engineer is mainly concerned with the process of creating bug-free software: specification, testing, etc.

I was amused that this seemingly exhaustive list omits my favorite debugging analogy: the way of the scientist, where we [formulate hypotheses and test them with experiments](https://blog.regehr.org/archives/199) (Metzger does discuss hypothesis testing, but the material is split across the detective and mathematician). Some of these chapters—the ways of the mathematician, detective, and psychologist in particular—are very strong, while others are weaker. Interleaved with the “six ways” are a couple of lengthy case studies where buggy pieces of code are iteratively improved.

### Summary

All of these books are good. Each contains useful information and each of them is worth owning. On the other hand, you don’t need all four. Here are the tradeoffs:

- [Agans’ book](http://www.amazon.com/Debugging-Indispensable-Software-Hardware-Problems/dp/0814474578/) is *The Prince* or *The Art of War* for debugging: short and sweet and sometimes profound, but not usually containing the most specific advice for your particular situation. It is most directly targeted toward people who work at the hardware/software boundary.
- [Butcher’s book](http://www.amazon.com/Debug-Repair-Prevent-Pragmatic-Programmers/dp/193435628X) is not too long and contains considerably more information that is specific to software debugging than Agans.
- [Zeller’s book](http://www.amazon.com/Why-Programs-Fail-Second-Systematic/dp/0123745152/) has a strong computer science focus and it is most helpful for understanding debugging methods and tools, instead of just building solid debugging skills.
- [Metzger’s book](http://www.amazon.com/Debugging-Thinking-Multidisciplinary-Approach-Technologies/dp/1555583075/) is the most detailed and specific about the various mental tools that one can use to attack software debugging problems.

I’m interested to hear feedback. Also, let me know if you have a favorite book that I left out (again, I’m most interested in language- and system-independent debugging).
