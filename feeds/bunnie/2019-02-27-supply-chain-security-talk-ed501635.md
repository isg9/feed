---
title: Supply Chain Security Talk
url: https://www.bunniestudios.com/blog/2019/supply-chain-security-talk/
published: "2019-02-27T09:48:30Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=5519
---

# Supply Chain Security Talk

I recently gave an invited talk about supply chain security at BlueHat IL 2019. I was a bit surprised at the level of interest it received, so I thought I’d share it here for people who might have missed it.

In the talk, I relay some of my personal trials authenticating my supply chains, then I go into the why of the supply chain attacks to establish some scenarios for evaluating different approaches. The talk attempts to broadly categorize the space of possible attacks, ranging from attacks that cost a penny and a few seconds to pull off to hundreds of thousands of dollars and months. Finally, I try to outline the depth of the supply chain attack surface, highlighting the overall [TOCTOU (time of check, time of use)](https://en.wikipedia.org/wiki/Time_of_check_to_time_of_use) problem that is the supply chain.

The main insight is that transparency or openness of design by itself does little to secure a supply chain, because the entire situation is one huge TOCTOU problem. Checking hardware design files, locking down the assembly line, and Fedexing the product to your office is like hashing and signing your source code, running it through a trusted compiler, and then sending the binary unencrypted over the Internet and trusting it because it was “thoroughly checked”.

The inverse analysis is equally daunting: in software, one may copy each binary into RAM, hash and check its cryptographic signature, and run it only if it checks out. For hardware, there is no equivalent of “hash this instance of hardware and check its cryptographic signature” before use: “hashing” hardware involves taking it apart and inspecting every transistor and wire, which is both impractical and likely to render the hardware non-functional.

Thus while open source hardware does engender some benefits for security (such as [disclosing μ-state for Spectre side-channel analysis](https://arxiv.org/abs/1902.05178) and ensuring no backdoors due to design oversight), it addresses a separate problem domain from supply chain attacks. While an open source hardware phone is *arguably more trustable* than a closed source one, open source is necessary but not sufficient for it to *be trusted*.

I do have some ideas on the practical mitigation of supply chain attacks, but they are still a bit too green to blog about. Stay tuned…
