---
title: Dispatches from the possibly last days of human relevance
url: https://scottaaronson.blog/?p=9782
published: "2026-05-28T04:36:07Z"
feed: aaronson
guid: https://scottaaronson.blog/?p=9782
---

# Dispatches from the possibly last days of human relevance

As most readers have presumably heard by now, Paul Erdös’s [Unit Distance Problem](https://en.wikipedia.org/wiki/Unit_distance_graph) from 1946—one of the central open problems from the field of discrete geometry— [has been solved](https://cdn.openai.com/pdf/74c24085-19b0-4534-9c90-465b8e29ad73/unit-distance-proof.pdf) by an internal OpenAI model. Erdös had conjectured that, given n points in the plane, at most n1+o(1) pairs of them could be unit distance apart. Using high-powered results from algebraic number theory, GPT refuted this, constructing a set with n1+ε unit-distance pairs, for ε ~ 10-38. Shortly afterward, Will Sawin, a human (!), [improved GPT’s construction](https://arxiv.org/abs/2605.20579) to get ~n1.014 pairs. There’s since been a [claim](https://mathoverflow.net/questions/511514/what-is-the-unit-distance-exponent) to improve this further, to ~n1.034. Meanwhile, the best known upper bound remains n4/3, improving Erdös’s n3/2.

The entire process seems have been one-shot: my former student [Lijie Chen](https://chen-lijie.github.io/) simply gave GPT the problem, then GPT thought for a while and output a several-page argument that, on analysis by human experts, turned out to be correct. *Of course* there’s selection bias here; we’re not hearing as much about the hundreds of other problems GPT was given that it *didn’t* solve (isn’t that the case with humans too?). Clearly, too, GPT was helped by the facts that human mathematicians had wasted most of their time trying to prove Erdös right rather than looking for a counterexample, and that, even if they *did* look for a counterexample, they’d need to be experts in algebraic number theory to find this one, which hardly any discrete geometers are. So, *maybe* that suggests that AI, right now, is “merely” picking various medium-hanging fruits that human mathematicians missed for contingent reasons? With emphasis on the “right now.”

In a [companion paper](https://cdn.openai.com/pdf/74c24085-19b0-4534-9c90-465b8e29ad73/unit-distance-remarks.pdf), OpenAI helpfully included commentary from Timothy Gowers, Noga Alon, Will Sawin, Daniel Litt, and many other experts, reflecting on the breakthrough, the path that GPT took to get to it (which can actually be seen by examining its chain-of-thought), and what this might mean for the future of mathematical research.

I heard the news maybe an hour after it broke, when some UT grad students came to my office to tell me. For what it’s worth: these students were morose, musing about how everything might soon be over for young scientists and mathematicians like themselves. I don’t know whether they’re right, but I feel like I should tell the truth about what their reaction was.

\[ **Update:** News has been coming faster than I can write about it, but today we learned that *another* important conjecture of Erdös [has been refuted](https://arxiv.org/abs/2605.28781v1). Erdös and Szemerédi’s strong sumset conjecture over R, from the 1970s, had said that, if A is a finite set of real numbers, then either \|A+A\| or \|A×A\| must be at least \|A\|2-o(1). In this case humans, including the aforementioned Sawin, did almost all of the work of constructing the counterexample, but they were directly inspired by GPT’s earlier refutation of the unit distance conjecture. It remains open whether such a counterexample exists where A is a set of integers.\]

Then, a few days later, a team at DeepMind, including my UT Austin colleague [Swarat Chaudhuri](https://www.cs.utexas.edu/~swarat/), announced that they were able to use a system called AlphaProof Nexus to [settle nine more (!) Erdös problems](https://arxiv.org/abs/2605.22763v1), many of them in additive combinatorics, along with miscellaneous other open math problems. Notably, in this case the AI also fully formalized its proofs in Lean.

And then, just today, Jelani Nelson alerted me to a [new CS theory paper](https://arxiv.org/pdf/2605.24130), which solves a longstanding open problem about electrical flows on graphs using a proof from GPT5.5Pro.

It seems to me that we’re now over the top of this particular rollercoaster, and it will keep accelerating until we reach the bottom, wherever that might be. I don’t know whether to hope or dread that solutions to P versus NP and all our other great problems will be included in the ride—that our role, as human mathematicians, will be reduced to (at most) deciding which questions we find interesting and then understanding AI models’ answers to those questions.

But *maybe* that won’t happen. Maybe the new AI mathematicians will soon hit a wall, because they lack the uncomputable quantum gravity microtubules of Penrose and Hameroff, or some other magic human ingredient. The fantastical thing is that, one way or the other, we’re going to find out empirically before very long.

---

Readers may have also seen the news that multiple prizewinning entries in a short fiction contest called the Commonwealth Prize, [give overwhelming indications of having been written by AIs](https://www.thefp.com/p/ai-generated-literature-controversy). As [Kelsey Piper puts it](https://www.theargumentmag.com/p/the-literary-world-is-sleepwalking):

> There are, let’s say, also some noticeable similarities in the prose style between the winning stories that were flagged for AI use. AI chatbots love metaphors and similes, and they often spit out ones that sound vaguely pleasing but are logically incoherent or ascribe properties to things that don’t make sense.
>
> “The Serpent in the Grove” gave us, “The girl smiled like sunrise over a sink.” “The Bastion’s Shadow” says, “She carried it now in her bag, heavy as a charm.” “Mehendi Nights” describes something as “swaying against plaster like a warning bell.”

The Commonwealth Foundation, whose judges chose these stories, hasn’t exactly covered itself in glory—saying, on the one hand, that it strictly forbids AI use but on the other, that it will continue taking authors at their word that they didn’t use AI, no matter the immensity of evidence to the contrary. As many others have pointed out, judges more versed in AI would’ve ironically been better placed to notice the signs of its use.

[If only](https://scottaaronson.blog/?p=9333) there were some sort of automated way to detect AI-generated text. Someone should really get on that problem, don’t you think?

---

But maybe we should just throw in the towel—as some of my colleagues have already done in the context of undergraduate projects? Maybe we should simply say that a good story is a good story, regardless of what manner of entity produced it?

As it happens, just last week I read my very first AI-written story that affected me *as* a story, to the extent that I wanted to read it more than once. This happened when I gave GPT5.5Pro the following simple prompt:

> Write me a story about the most ancient Israelites that’s riveting like the stories of the Bible but that’s also consistent with all of the archeological evidence.

You can [read the result here](https://chatgpt.com/s/t_6a0d10c5a6188191ae55c1ddf0ca8015?fbclid=IwY2xjawSEm_hleHRuA2FlbQIxMQBzcnRjBmFwcF9pZBAyMjIwMzkxNzg4MjAwODkyAAEefenoQRTNfSgls6rstt-4V6OSjRHGFrDif_KKDPaC_ODSeQrr8lsG79LwAkk_aem_2e1OvpSXaGY0S_rQEP8OFA). One of my Facebook friends called it “disturbingly good,” and whatever the problems with the piece, I share that feeling. Of course, I’m well aware that GPT could easily generate a thousand stories like it—sampled from the same probability distribution—and then I could even do statistics on which tropes were the most common. This makes it feel silly to overindex on the first story that happened to be output, and yet somehow I did.

---

I feel like at this point, both the prophets of AI utopia like Ray Kurzweil, and of AI doom like Eliezer Yudkowsky, could be forgiven for asking: *dude, will you listen to us YET?* Do you *still* find it prudent to call this new form of terrestrial intelligence a stochastic parrot, a laughable fraud, or a fad that’s about to go away? Fear it all you want, hate it even, but at least respect it!

Which brings me to the *other* big AI news from the past week, namely that Pope Leo released his first encyclical, which is entitled [“Safeguarding the Human Person in the Time of Artificial Intelligence.”](https://www.vatican.va/content/leo-xiv/en/encyclicals/documents/20260515-magnifica-humanitas.html) I read it and … well, I certainly agreed with the theme that such a world-changing technology needs to be developed for the common good (as the Pope would have it, like the walls of Jerusalem), rather than for the profit or vanity of any one individual or company (in his analogy, like the Tower of Babel). I had quibbles with some of the other parts. Zvi Mowshowitz, as he often does, had a [superb paragraph-by-paragraph analysis](https://thezvi.substack.com/p/rtmh-pope-leos-magnifica-humanitas). Amusingly, there are [indications that parts of the encyclical were written by AI](https://www.reddit.com/r/slatestarcodex/comments/1toa872/claude_author_of_the_humanitas_evidence_that_the/).

To me, though, maybe the most notable part was that Chris Olah, who leads Anthropic’s interpretability team, was [standing next to the Pope at the ceremony](https://www.forbes.com/sites/aliciapark/2026/05/25/anthropic-billionaire-cofounder-joins-pope-leo-warns-ai-job-losses-will-spark-moral-imperative-of-historic-proportions/), and [delivered his own remarks](https://www.anthropic.com/news/chris-olah-pope-leo-encyclical). I felt like Chris, who I met even before Anthropic existed, was a non-obvious yet inspired choice here, one of the rare figures in frontier AI whose technical *and* moral authority are both completely unimpeachable by anyone.

And so, at this momentous era for the human project, and on no less of an authority than that of the Vicar of Christ himself, the Supreme Pontiff and the Successor of Peter, I hereby throw myself on the wisdom and mercy of … uhh, I guess, Chris Olah and his team at Anthropic.

Chris, if I am soon to share the earth with entities that can prove the Riemann Hypothesis and solve quantum gravity after 30 seconds of thought, then may you understand those entities well enough to cause them to be nice.

---

**Endnote:** I should have foreseen, but didn’t, that the comments on this post would be dominated by people looking for ways to minimize whichever specific AI accomplishments I blogged about. Thus, it turns out, the ability of AI to solve Erdös problems just demonstrates that Erdös’s problems were never “serious” math in the first place—nothing like algebraic geometry or Grothendieck-style theory-building, which remain untouched. Likewise, the story I shared was obvious AI slop.

I had taken it as obvious that, when assessing AI’s impact on the world, one needs to look at least *somewhat* into the future: to remember where things were four years ago, compared to where they are today, and at least *try* to draw a straight line through the data, if not the exponential that seems to fit better.

Does anyone seriously doubt at this point that major open problems in algebraic geometry and other “Grothendieck-friendly” areas of math *will* fall to future AI models? Or that AI-written stories will improve, not only to win literary awards from AI-naïve judges, but to avoid the features that commenters here are complaining about? And that, whenever that happens, there will be *new* confident reasons not to care immediately offered up in comment sections like mine?

Apparently people *do* still doubt—hence the throwaway remark in my post about Penrose and Hameroff and microtubules. If not that or something like it, what exactly do they think the ceiling will be, and why?

Recently (I should have mentioned this before), I came across what I consider one of the greatest social experiments of all time, one that illuminates people’s reactions to every AI advance. A Twitter/X user named SHL0MS [displayed the following AI-generated fake “Monet painting,”](https://x.com/Jediwolf/status/2054776716770320631) and asked people to explain what made it worse than real Monet paintings:

[![](https://scottaaronson.blog/wp-content/uploads/2026/05/image-1-1024x759.png)](https://scottaaronson.blog/wp-content/uploads/2026/05/image-1-scaled.png)

If you haven’t seen this yet, I recommend that you try the exercise yourself before reading further.

As it was, numerous art aficionados responded at length, savaging the flat, lifeless, uncreative AI slop, the emotionless composition, the missing spark, the lack of tranquility, the harshness, the lack of depth and symbiosis, and on and on and on.

Only after they had all said their piece did SHL0MS reveal that this is an actual Monet painting.
