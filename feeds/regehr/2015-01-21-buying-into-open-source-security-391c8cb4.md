---
title: Buying Into Open Source Security
url: https://blog.regehr.org/archives/1211
published: "2015-01-21T00:42:57Z"
feed: regehr
guid: http://blog.regehr.org/?p=1211
---

# Buying Into Open Source Security

If you were given the opportunity to spend USD 100 million over five years to maximally improve the security of open source software, what would you do? Let’s just assume that the money comes with adequate administrative staff to manage awards and contracts so you can focus on technical issues. A few ideas:

- Bug bounties, skewed towards remotely exploitable bugs and towards ubiquitous infrastructure such as OpenSSL and the Linux kernel. To get rid of USD 100 M in five years, we’ll probably need to make the bounties very large by current standards, or else give out a lot of them.

- Contracts for compatible rewrites of crufty-but-important software in safe languages.

- Contracts for aggressive cleanup and refactoring of things like OpenSSL.

- Contracts for convincing demonstrations of the security of existing codes, in cases where it seems clear that rewrites are undesirable or impractical. These demonstrations might include formal verification, high-coverage test suites, and thorough code inspections.

- Research grants and seed funding towards technologies such as unikernels, static analyzers, fuzzers, homomorphic encryption, Qubes/Bromium-kinda things, etc. (just listing some of my favorites here).

- Contracts and grants for high-performance, open-source hardware platforms.

This post is motivated by the fact that there seems to be some under-investment in security-critical open source components like Bash and OpenSSL. I’ll admit that it is possible (and worrying) that USD 100 M isn’t enough to make much of a dent in our current and upcoming problems.
