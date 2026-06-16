---
title: Before we start on quantum
url: https://scottaaronson.blog/?p=9668
published: "2026-04-07T07:51:11Z"
feed: aaronson
guid: https://scottaaronson.blog/?p=9668
---

# Before we start on quantum

Imagine that every week for twenty years, people message you asking you to comment on the latest wolf sighting, and every week you have to tell them: I haven’t seen a wolf, I haven’t heard a wolf, I believe wolves exist but I don’t yet see evidence of them anywhere near our town.

Then one evening, you hear a howl in the distance, and sure enough, on a hill overlooking the town is the clear silhouette of a large wolf. So you point to it — and all the same people laugh and accuse you of “crying wolf.”

Now you know how it’s been for me with cryptographically relevant quantum computing.

---

I’ve been writing about QC on this blog for a while, and have done hundreds of public lectures and interviews and podcasts on the subject. By now, I can almost always predict where a non-expert’s QC question is going from its first few words, and have a well-rehearsed answer ready to go the moment they stop talking. Yet sometimes I feel like it’s all for naught.

Only today did it occur to me that I should write about something more basic. Not quantum computing itself, but *the habits of mind that seem to prevent some listeners from hearing whatever I or other researchers have to tell them about QC*. The stuff that we’re wasting our breath if we don’t get past.

Which habits of mind am I talking about?

1. **The Tyranny of Black and White.** Hundreds of times, I’ve answered someone’s request to explain QC, only to have them nod impatiently, then interrupt as soon as they can with: “So basically, the take-home message is that quantum is coming, and it’ll change everything?” Someone else might respond to *exactly the same words from me* with: “So basically, you’re saying it’s all hype and I shouldn’t take any of it seriously?” As in my wolf allegory, the same person might even jump from one reaction to the other. Seeing this, I’ve become a fervent believer in [horseshoe theory](https://en.wikipedia.org/wiki/Horseshoe_theory), in QC no less than in politics. Which sort of makes sense: if you think QCs are “the magic machines of the future that will revolutionize everything,” and then you learn that they’re *not*, why *wouldn’t* you jump to the opposite extreme and conclude you’ve been lied to and it’s all a scam?

2. **The Unidimensional Hype-Meter.** “So … *\[long, thoughtful pause\]* … you’re actually telling me that *some* of what I hear about QC is real … but *some* of it is hype? Or—yuk yuk, I bet no one ever told you this one before—it’s a *superposition* of real and hype?” OK, that’s better. But it’s *still* trying to project everything down onto a 1-dimensional subspace that loses almost all the information!

3. **Words As Seasoning.** I often get the sense that a listener is treating all the words of explanation—about amplitudes and interference, Shor versus Grover, physical versus logical qubits, etc.—as seasoning, filler, an annoying tic, a stalling tactic to put off answering the only questions that matter: “is Quantum real or not real? If it’s real, when is it coming? Which companies will own the Quantum space?” In reality, explanations are the entire substance of what I can offer. For my experience has consistently been that, if someone has no interest in learning what QC is, which classes of problems it helps for, etc., then *even if I answer their simplistic questions like “which QC companies are good or bad?,” they won’t believe my answers anyway.* Or they’ll believe my answers only until the next person comes along and tells them the opposite.

4. **Black-Boxing.** Sometimes these days, I’ll survey the spectacular recent progress in fault-tolerance, 2-qubit gate fidelities, programmable hundred-qubit systems, etc., only to be answered with a sneer: “What’s the biggest number that Shor’s algorithm has factored? Still 15 after all these years? Haha, apparently the emperor has no clothes!” I’ve commented that this is sort of like dismissing the Manhattan Project as hopelessly stalled in 1944, on the ground that so far it hasn’t produced even a *tiny* nuclear explosion. Or the Apollo program in 1967, on the ground that so far it hasn’t gotten any humans even 10% of the way to the moon. Or GPT in 2020, on the ground that so far it can’t even do elementary-school math. Yes, sometimes emperors are naked—but you can’t tell until you actually *look at* the emperor! Engage with the specifics of quantum error correction. If there’s a reason why you think it can’t work beyond a certain scale, say so. But don’t fixate on one external benchmark and ignore everything happening under the hood, if the experts are *telling you* that under the hood is where all the action now is, and your preferred benchmark is only relevant later.

5. **Questions with Confused Premises.** “When is Q-Day?” I confess that this question threw me for a loop the first few times I heard it, because I had no idea what “Q-Day” was. Apparently, it’s the single day when quantum computing becomes powerful enough to break all of cryptography? Or: “What differentiates quantum from binary?” “How will daily life be different once we all have quantum computers in our homes?” Try to minimize the number of presuppositions.

6. **Anchoring on Specific Marketing Claims.** “What do you make of D-Wave’s latest quantum annealing announcement?” “What about IonQ’s claim to recognize handwriting with a QC?” “What about Microsoft’s claim to have built a topological qubit?” These questions can be fine as part of a larger conversation. Again and again, though, someone who doesn’t know the basics will *lead* with them—with whichever specific, contentious thing they most recently read. Then the entire conversation gets stuck at a deep node within the concept tree, and it can’t progress until we backtrack about five levels.

---

Anyway—sorry for yet another post of venting and ranting. Maybe this will help:

The wise child asks, “what are the main classes of problems that are currently known to admit superpolynomial quantum speedups?” To this child, you can talk about quantum simulation and finding hidden structures in abelian and occasionally nonabelian groups, as well as Forrelation, glued trees, HHL, and DQI—explaining how the central challenge has been to find end-to-end speedups for non-oracular tasks.

The wicked child asks, “so can I buy a quantum computer right now to help me pick stocks and search for oil and turbocharge LLMs, or is this entire thing basically a fraud?” To this child you answer: “the quantum computing people who seek *you* as their audience are frauds.”

The simple child asks, “what is quantum computing?” You answer: “it’s a strange new way of harnessing nature to do computation, one that dramatically speeds up certain tasks, but doesn’t really help with others.”

And to the child who doesn’t know how to ask—well, to that child you don’t need to bring up quantum computing at all. That child is probably already fascinated to learn classical stuff.
