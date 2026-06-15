---
title: '[$] A loadable crypto module for FIPS certification'
url: https://lwn.net/Articles/1073759/
published: "2026-05-29T14:29:18Z"
feed: lwn
guid: https://lwn.net/Articles/1073759/
---

# [$] A loadable crypto module for FIPS certification

Many organizations require US [Federal Information Processing Standard](https://en.wikipedia.org/wiki/Federal_Information_Processing_Standards) (FIPS) certification of the crypto code they are running. The certification process is lengthy, but the bigger problem is that the way the crypto subsystem is built into the kernel makes the result unable to be reused across kernel updates. I have proposed a [patch
series](https://lwn.net/ml/all/20260418002032.2877-1-wanjay@amazon.com/) that decouples the crypto subsystem into a standalone loadable module, allowing a certified crypto module to be reused with multiple kernels and, thus, requiring fewer lengthy recertification delays.
