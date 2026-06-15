---
title: The Problem with Friendly C
url: https://blog.regehr.org/archives/1287
published: "2015-12-23T23:34:32Z"
feed: regehr
guid: http://blog.regehr.org/?p=1287
---

# The Problem with Friendly C

I’ll assume you’re familiar with the [Proposal for Friendly C](https://blog.regehr.org/archives/1180) and perhaps also Dan Bernstein’s recent [call for a Boring C compiler](https://groups.google.com/forum/m/#!msg/boring-crypto/48qa1kWignU/o8GGp2K1DAAJ). Both proposals are reactions to creeping exploitation of undefined behaviors as C/C++ compilers get better optimizers. In contrast, we want old code to just keep working, with latent bugs remaining latent.

After publishing the Friendly C Proposal, I spent some time discussing its design with people, and eventually I came to the depressing conclusion that there’s no way to get a group of C experts — even if they are knowledgable, intelligent, and otherwise reasonable — to agree on the Friendly C dialect. There are just too many variations, each with its own set of performance tradeoffs, for consensus to be possible. To get a taste of this, notice that in the comments for the Friendly C post, several people disagree with what I would consider an extremely non-controversial design choice for Friendly C: memcpy() should have memmove() semantics. Another example is what should be done when a 32-bit integer is shifted by 32 places (this is undefined behavior in C and C++). Stephen Canon pointed out on twitter that there are many programs typically compiled for ARM that would fail if this produced something besides 0, and there are also many programs typically compiled for x86 that would fail when this evaluates to something other than the original value. So what is the friendly thing to do here? I’m not even sure– perhaps the baseline should be “do what gcc 3 would do on that target platform.” The situation gets worse when we start talking about a friendly semantics for races and OOB array accesses.

I don’t even want to entertain the idea of a big family of Friendly C dialects, each making some niche audience happy– that is not really an improvement over our current situation.

Luckily there’s an easy away forward, which is to skip the step where we try to get consensus. Rather, an influential group such as the Android team could create a friendly C dialect and use it to build the C code (or at least the security-sensitive C code) in their project. My guess is that if they did a good job choosing the dialect, others would start to use it, and at some point it becomes important enough that the broader compiler community can start to help figure out how to better optimize Friendly C without breaking its guarantees, and maybe eventually the thing even gets standardized. There’s precedent for organizations providing friendly semantics; Microsoft, for example, provides [stronger-than-specified semantics for volatile variables](https://msdn.microsoft.com/en-us/library/jj204392.aspx) by default on platforms other than ARM.

Since we published the Friendly C proposal, people have been asking me how it’s going. This post is a long-winded way of saying that I lost faith in my ability to push the work forward. However, I still think it’s a great idea and that there are people besides me who can make it happen.
