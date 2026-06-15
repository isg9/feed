---
title: sticking point — wingolog
url: https://wingolog.org/archives/2023/04/18/sticking-point
published: "2023-04-18T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/04/18/sticking-point
---

# sticking point — wingolog

## [sticking point](/archives/2023/04/18/sticking-point)

18 April 2023 8:23 PM

- [meta](/tags/meta)
- [hackjam](/tags/hackjam)
- [guile](/tags/guile)
- [gc](/tags/gc)
- [hegel](/tags/hegel)

Good evening, gentle readers. A brief note tonight, on a sticky place.

See, I have too many projects right now.

In and of itself this is not so much of a problem as a condition. I know my limits; I keep myself from burning out by shedding load, and there is a kind of priority list of which projects keep adequate service levels.

First come the tiny humans that are in my care who need their butts wiped and bodies translated to and from school or daycare and who – well you know the old Hegelian trope, that the dialectic crank of history doesn’t turn itself, that it takes actions from people to synthesize the thesis and the antithesis, and that even History itself isn’t always monotonic; in the same way, bedtime is a reality, there are the material conditions of sleepiness and you’re-gonna-be-so-tired-tomorrow but without the World-Historical Actor which whose name is Dada ain’t nobody getting into pyjamas, not to mention getting in bed and going to sleep.

Lower in the priority queue there is work, and work is precious: a whole day, a day of just thinking about things and solving problems; yes, I wish I could choose all of the problems that I work on but the simple fact of being able to think and solve is a gift, or rather a reward, time stolen from the arbitrary chaotic preliterate interruptors. I love my kids – and here I have to choose my conjunction wisely – *and* also they love me. Which is nice! They express this through wanting to sit on my lap and demand that I read to them when I am thinking about things. We all have our love languages, I suppose.

And the house? There are 5 of us now, and that’s, you know, more than 5 times the work it was before because the 3 wee ones are more chaotic than the adults. There is so much laundry. We are now a cook-the-whole-500g-bag-of-pasta family, and we’re far from the teenager stage.

Finally there are the weird side projects, of which there are a few. I have some sporty things I do that I can’t avoid because I am socially coerced into doing them; all in all a fine configuration. I have some volunteer work, which has parts I like that take time, which I am fine with, and parts I don’t that I neglect. There’s the garden, also neglected but nice to muck about in sometimes. Sometimes I see friends? Rarely, so rarely, but sometimes.

But, netfriends, if I find a free couple hours, I hack. Not less than a couple hours, because otherwise I can’t get into things. More? Yes please! But usually no, because priorities.

I write this note because I realized that in the last month or so, hain’t been no hacktime; there’s barely 30 minutes between the end of the evening kitchen-clean and the onset of zzzz. That’s fine, really; ultimately I need to carve out work time for this sort of thing, somehow or other.

But I was also realizing that I painted myself into a corner with my [GC
work](https://wingolog.org/archives/2023/02/07/whippet-towards-a-new-local-maximum). See, I need some new benchmarks, and I can’t afford to use Guile itself as my benchmark yet, because I can’t afford to fix deep bugs right now. I need something small. Specifically I need a little language implementation with efficient GC integration. I was looking at porting the [little Scheme
implementation](https://wingolog.org/archives/2022/08/18/just-in-time-code-generation-within-webassembly) from my Wasm JIT work to C – because the rest of whippet is in C – but is that a good enough reason? I got mired in the swamps of precise GC roots in C and let me tell you, it is disgusting. You don’t have RAII like you do in C++, without GCC extensions. You don’t have stack maps, like you would like. I might be able to push through but it’s the sort of project you need a day for, and those days have not been forthcoming recently.

Anyway, just a little hack log tonight, not detailing a success, but rather a condition. Perhaps observing the position of this logjam will change its velocity. Here’s hoping, and until next time, happy hacking!

## related articles

- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [whippet hacklog: adding freelists to the no-freelist space](/archives/2025/08/07/whippet-hacklog-adding-freelists-to-the-no-freelist-space)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)

Comments are closed.
