---
title: When Will Software Verification Matter?
url: https://blog.regehr.org/archives/710
published: "2012-05-18T21:59:57Z"
feed: regehr
guid: http://blog.regehr.org/?p=710
---

# When Will Software Verification Matter?

When will average people start to use formally verified software? Let me be clear right from the start that I’m not talking about software that has been subjected to static analysis or about software that implements proved-correct algorithms, but rather about software whose functional correctness has been verified at the implementation level in a way that might [pass the piano test](https://blog.regehr.org/archives/364). As far as I know, such software is not yet in widespread use — if anyone knows differently, please let me know.

The problem isn’t that we cannot formally verify software, the problem is that at present the cost/benefit ratio is too high. Let’s look at these in turn. First, how much does full formal verification (i.e., a machine-checked correctness proof of an actual implementation, using believable specifications) increase the cost of development? For many projects, the multiplier might as well be infinity since we don’t know how to verify something like MS Word. For other projects (most notably [L4.verified](http://ertos.nicta.com.au/research/l4.verified/) and [CompCert](http://compcert.inria.fr/), but there are plenty of other good examples) the cost multiplier is probably well in excess of 10x. Next, how much benefit is gained by formal verification? This is harder to estimate, but let me argue by analogy that the value implicitly assigned to software correctness is very low in most circumstances. For example, compilers (empirically) become more likely to miscompile code as the optimization level is increased. But how much of the software that we use in practice was compiled without optimization? Almost none of it. The 2x or 3x performance gain far outweighs the perceived risk of incorrectness due to compiler defects. The only domain that I am aware of where optimization is routinely disabled for correctness reasons is avionics.

The cost/benefit argument tells us, with a good degree of accuracy, where we will first see mass use of formally verified software. It will be codes that:

- have very stable interfaces, permitting specification and verification effort to be amortized over time
- are amenable to formal specification in the first place
- are relatively small

The codes that come immediately to mind are things like crypto libraries, compression and decompression utilities, and virtual machine monitors. I mean, why hasn’t anyone verified gzip yet? Is it really that hard? What about [xmonad](http://xmonad.org/), can that be verified? A verified virtual machine monitor is clearly more difficult but this is a very high-value domain, and there are several such projects underway.

In all cases above — hypervisor, gzip, OS kernel, compiler — the question I’m trying to get at isn’t whether the artifacts can be created, because they can. The question is: Will they see widespread use? For this to happen, the verified codes are going to have to be relatively fully-featured (right now, most projects would never choose CompCert over GCC due to missing features) and are going to have to perform on a par with the best non-verified implementations.

What I suspect will happen is that we’ll start to see little islands of verified programs floating in the larger ocean of non-verified code. For example, I’d be happy to drop a proved-correct gzip into my Linux machine as long as it was respectably fast. Ideally, these islands will spread out to encompass more and more security-critical code. I think we are on the cusp of this starting to happen, but progress is going to be slow. I worry that creating these fully-featured, high-performance verified programs is going to require a huge amount of work, but that it will not be particularly novel (so the research community won’t care about it) or commercially viable. These problems go away only if we can dramatically reduce the cost of implementing verified programs, or (less likely!) of verifying implemented programs.
