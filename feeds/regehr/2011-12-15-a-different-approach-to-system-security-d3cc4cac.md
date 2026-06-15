---
title: A Different Approach to System Security
url: https://blog.regehr.org/archives/650
published: "2011-12-15T17:11:31Z"
feed: regehr
guid: http://blog.regehr.org/?p=650
---

# A Different Approach to System Security

I enjoy it when science fiction has something [useful to say about computer security](https://blog.regehr.org/archives/581). Towards the end of Iain M. Banks’ *[Matter](http://www.amazon.com/Matter-Culture-Iain-M-Banks/dp/0316005371)*, there’s a big space battle and we find this passage:

> “Compromised,” Hippinse told him. “Taken over by the other side. Persuaded by a sort of thought-infection.”
>
> “Does that happen a lot, sir?”
>
> “It happens.” Hippinse signed. “Not to Culture ships, as a rule; they write their own individual OS as they grow up, so it’s like every human in a population being slightly different, almost their own individual species despite appearances; bugs can’t spread. The Morthanveld like a degree more central control and predictability in their smart machines. That has its advantages too, but it’s still a potential weakness. This Iln machine seems to have exploited it.”

Monoculture is an obvious and serious danger. For example, about the [2003 Slammer/Sapphire worm](http://www.caida.org/publications/papers/2003/sapphire/sapphire.html):

> Propagation speed was Sapphire’s novel feature: in the first minute, the infected population doubled in size every 8.5 (±1) seconds. The worm achieved its full scanning rate (over 55 million scans per second) after approximately three minutes, after which the rate of growth slowed down somewhat because significant portions of the network did not have enough bandwidth to allow it to operate unhindered. Most vulnerable machines were infected within 10-minutes of the worm’s release.

Imagine the damage a similar worm could do today, or in 20 or 100 years, if properly weaponized.

I know there are automated approaches to diversity (ASLR, randomized instruction sets, etc.) but I found “they write their own individual OS as they grow up” to be a very charming idea, perhaps in part because it is so wildly impractical today.
