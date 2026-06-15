---
title: Upgrading Precursor&#8217;s TRNG
url: https://www.bunniestudios.com/blog/2021/upgrading-precursors-trng/
published: "2021-06-14T14:14:35Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=6094
---

# Upgrading Precursor&#8217;s TRNG

It was pointed out that we’re missing a step-by-step guide on how to go from an idea, to hardware, to a fully implemented feature in Xous for Precursor. So, here it is.

Because Precursor uses an FPGA for its SoC, we can add new features to the hardware “on the fly”. In this case, we’re going to add some improvements to our basic managed TRNG block. To review, the existing TRNG consists of an [avalanche noise source](https://betrusted.io/avalanche-noise) and a [ring oscillator noise source](https://github.com/betrusted-io/betrusted-wiki/wiki/TRNG-characterization#ring-oscillator) as hardware-based sources of “true” entropy. Two generators are used for the following reasons:

- An external discrete generator is easy to check (just put an oscilloscope on the avalanche noise source), but harder to protect against physical access attacks
- An integrated, on-chip generator is harder to hack (more robust against a pair of tweezers executing a short-to-ground attack, or RF interference attacks), but harder to check (is the data from the TRNG, or merely a decoy CSPRNG implant?)
- All hardware mechanisms are fallible; having two sources improves robustness against transient drop-outs or aging failures

We’ve already done extensive, months-long characterization of both of the TRNG sources and found them to produce passable raw entropy. However, the system is still missing two features that are generally considered to be best practice:

1. Independent, on-line health monitors of the raw TRNG outputs. It’s important that the health monitoring happens before any conditioning or mixing of the raw data happens, and significantly, there is no one-size-fits-all health monitor for a TRNG: it’s advised ( [NIST SP 800-90B sec 4.4](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-90B.pdf)) to have tests that are tailored to the noise source.
2. Conditioning of the raw data. Despite best efforts to make TRNGs unbiased and flawless, they are really hard to get right. Furthermore, they are only capable of producing high-quality entropy at a limited data rate. Thus, most practical systems take a TRNG output and run it through a cryptographic stream cipher to generate a final datastream; this simultaneously protects against minor flaws in the TRNG while improving the availability of random numbers.

The following lengthy posts walk step-by-step through the thought process, implementation and debugging process of adding these features. Few people would even notice these features, and if everything is doing its job right (that is, the TRNG’s raw data is working correctly) is indistinguishable from the state before all this effort. However, we take TRNGs seriously here; so much rides on the quality of these random numbers that it’s probably worth the effort to harden them against failures, be they unintentional, malicious, or just design bugs.

I have to be honest, I spent a lot of time to check a box that few people care about, but I’ve come to realize that’s mainly what writing OS code and firmware is about. You can get more fame and dopamine from creating a cool UI theme with an afternoon of work. It’s also really hard to explain to everyday people what you’re doing exactly with all this time and effort; but without the underlying frameworks that make things durable and reliable, we all might as well be drawing chalk pictures on the side walk.

Without further ado, here are the two guides for adding features, there’s some repetition between the posts so they can be read independently.

- [On-line health monitors](https://github.com/betrusted-io/betrusted-wiki/wiki/TRNG-Online-Health-Monitors)
- [Raw data conditioning](https://github.com/betrusted-io/betrusted-wiki/wiki/TRNG-Data-Conditioning)

I’ll also take highlights from these wiki articles and repost them to the blog here, creating a “TL;DR” version that is also neatly delivered to the inboxes of my blog’s email subscribers.
