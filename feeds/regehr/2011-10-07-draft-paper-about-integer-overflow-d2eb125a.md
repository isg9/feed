---
title: Draft Paper about Integer Overflow
url: https://blog.regehr.org/archives/598
published: "2011-10-07T16:12:18Z"
feed: regehr
guid: http://blog.regehr.org/?p=598
---

# Draft Paper about Integer Overflow

[![](https://blog.regehr.org/wp-content/uploads/2011/10/pacman.png)](http://en.wikipedia.org/wiki/Pac-Man#Split-screen) Result of the infamous Pac-Man integer overflow

Last Spring I had a lucky conversation. I was chatting with Vikram Adve, while visiting the University of Illinois, and we realized that we working on very similar projects — figuring out what to do about integer overflow bugs in C and C++ programs. Additionally, Vikram’s student Will and my student Peng had independently created very similar LLVM-based dynamic checking tools for finding these bugs. As a researcher I find duplicated effort to be bad at several levels. First, it’s a waste of time and grant money. Second, as soon as one of the competing groups wins the race to publish their results, the other group is left with a lot of unpublishable work. However, after talking things through, we agreed to collaborate instead of compete. This was definitely a good outcome since [the resulting paper](http://www.cs.utah.edu/~regehr/papers/overflow12.pdf) — submitted last week — is almost certainly better than what either of the groups would have produced on its own. The point is to take a closer look at integer overflow than had been taken in previous work. This required looking for integer overflows in a lot of real applications and then studying these overflows. It turns out they come in many varieties, and the distinctions between them are very subtle. The paper contains all the gory details. The [IOC (integer overflow checker) tool is here](http://embed.cs.utah.edu/ioc/). We hope to convince the LLVM developers that IOC should be part of the default LLVM build.

We would be happy to receive feedback about the draft.
