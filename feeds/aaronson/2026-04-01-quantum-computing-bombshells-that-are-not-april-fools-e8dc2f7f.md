---
title: Quantum computing bombshells that are not April Fools
url: https://scottaaronson.blog/?p=9665
published: "2026-04-01T21:26:47Z"
feed: aaronson
guid: https://scottaaronson.blog/?p=9665
---

# Quantum computing bombshells that are not April Fools

For those of you who haven’t seen, there were actually two “bombshell” QC announcements this week. One, from Caltech, including friend-of-the-blog John Preskill, [showed how to do quantum fault-tolerance](https://scirate.com/arxiv/2603.28627) with lower overhead than was previously known, by using high-rate codes, which could work for example in neutral-atom architectures (or possibly other architectures that allow nonlocal operations, like trapped ions). The second bombshell, from Google, [gave a lower-overhead implementation of Shor’s algorithm](https://scirate.com/arxiv/2603.28846) to break 256-bit elliptic curve cryptography.

Notably, out of an abudance of caution, the Google team chose to “publish” its result via a cryptographic zero-knowledge proof that their circuit exists (so, without revealing the details to attackers). This is the first time I’ve ever seen a new mathematical result actually announced that way, although I understand that there’s precedent in the 1500’s, when mathematicians would (for example) prove their ability to solve quartic equations by challenging their rivals to duels. I’m not sure how much it will actually help, as once other groups know that a smaller circuit exists, it might be only a short time until they’re able to find it as well.

Neither of these results change the basic principles of QC that we’ve known for decades, but they do change the numbers.

When you put both of them together, Bitcoin signatures for example certainly look vulnerable to quantum attack earlier than was previously known!  In particular, the Caltech group estimates that a mere 25,000 physical qubits might suffice for this, where a year ago the best estimates were in the millions. How much time will this save — maybe a year?  Subtracting, of course, off a number of years that no one knows.

In any case, these results provide an even stronger impetus for people to upgrade now to quantum-resistant cryptography.  They—meaning you, if relevant—should really get on that!

When I got an early heads-up about these results—especially the Google team’s choice to “publish” via a zero-knowledge proof—I thought of Frisch and Peierls, calculating how much U-235 was needed for a chain reaction in 1940, but *not* publishing it, even though the latest results on nuclear fission had been openly published just the year prior. Will we, in quantum computing, also soon cross that threshold? But I got strong pushback on that analogy from the cryptography and cybersecurity people who I most respect. They said: we have decades of experience with this, and the answer is that you publish. And, they said, if publishing causes people still using quantum-vulnerable systems to crap their pants … well, maybe that’s what *needs* to happen right now.

Naturally, journalists have been hounding me for comments, though it was the worst possible week, when I needed to host like four separate visitors in Austin. I hope this post helps! Please feel free to ask questions or post further details in the comments.

And now, with no time for this blog post to leaven and rise, I need to go home for my family’s Seder. Happy Passover!
