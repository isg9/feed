---
title: 'Too Much Milk: The Secret Origin Story'
url: https://blog.regehr.org/archives/1145
published: "2014-05-05T17:46:20Z"
feed: regehr
guid: http://blog.regehr.org/?p=1145
---

# Too Much Milk: The Secret Origin Story

When I first taught operating systems 12 years ago, I based my teaching materials on a set of slides inherited from John Carter, the previous instructor at Utah. I was generally happy with these slides, and I’ve continued to evolve them since then, but one thing I was always curious about was the “too much milk” example that is used to introduce concurrency problems. It goes something like: “You see that you’re out of milk and leave to go buy some. A few minutes later your roommate comes home and sees that there’s no milk. Later: OH NO THERE’S TOO MUCH MILK because of a synchronization error between humans.” This always struck me as a bit of an idiosyncratic way to go about introducing synchronization but I didn’t have any reason to replace it and didn’t think about it much until one time while procrastinating on lecture prep I Googled “too much milk” and learned the shocking truth: some large fraction of operating systems courses at research universities use this same example. [See this for yourself](https://www.google.com/search?q=%22too+much+milk%22+%22operating+systems%22).

The intriguing possibility is that too much milk could be used as sort of a mitochondrial DNA to trace the evolution of OS course notes. I started to envision creating a tool for analyzing the lineage of Powerpoint files, but then totally forgot about it until a few weeks ago. While chatting with Ryan Stutsman about teaching OS, for some reason I mentioned the Too Much Milk phenomenon. He thought that this had probably originated with his advisor John Ousterhout. John then explained:

> I believe that the “too much milk” example owes its popularity to me, in that I have been using it ever since the early 1980s in my courses. For many years in the 1980s and 1990s, whenever a PhD student graduated in the systems area, Dave Patterson gave them a complete set of lecture videos for a couple of systems classes, including my CS 162 lectures. As a result, a lot of the material from my CS 162 lectures has spread all across the country through Berkeley graduates, including the “too much milk” example.
>
> However, I did not invent this example. Before I taught my first operating systems course at Berkeley, Mike Powell gave me a copy of his notes and the “too much milk” example was there. So, it goes back at least to Mike Powell. I don’t know if he invented it, or if he got it from someone else.

So there you have it.

This post doesn’t have too much of a point, but I thought this was a nice story and also it provides a bit of insight into how teaching actually works: figuring out how to teach course material is hard and in practice, most of the time we borrow a framework and adapt it to fit our needs. Every now and then I’ll discuss this in class and invariably one or two undergraduates are so surprised that I’ll get dinged on the course evaluations with comments like “Lazy professor doesn’t even develop his own course materials.” However, I’ve done three or four courses the other way — starting from scratch — and have found that it takes multiple iterations before the material converges, so it’s not really clear to me that it’s better to start from scratch in cases where a good set of existing material can be found.
