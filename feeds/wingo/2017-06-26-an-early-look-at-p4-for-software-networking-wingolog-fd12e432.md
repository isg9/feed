---
title: an early look at p4 for software networking — wingolog
url: https://wingolog.org/archives/2017/06/26/an-early-look-at-p4-for-software-networking
published: "2017-06-26T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2017/06/26/an-early-look-at-p4-for-software-networking
---

# an early look at p4 for software networking — wingolog

## [an early look at p4 for software networking](/archives/2017/06/26/an-early-look-at-p4-for-software-networking)

26 June 2017 2:00 PM

- [igalia](/tags/igalia)
- [networking](/tags/networking)
- [p4](/tags/p4)
- [compilers](/tags/compilers)
- [snabb](/tags/snabb)
- [vpp](/tags/vpp)

Happy midsummer, hackfriends!

As you know [at work](https://www.igalia.com/networking) we have been trying to find ways to apply compilers technology to the networking space. We will compile [high-level configurations into low-level network processing graphs](https://github.com/snabbco/snabb/blob/master/src/program/config/README.md), [search algorithms into lookup routines optimized for the target data structures](https://github.com/snabbco/snabb/blob/master/src/lib/binary_search.dasl), [packet filters into code ready to be further trace-compiled](https://github.com/snabbco/snabb/tree/master/lib/pflua), or [hash functions into parallel AVX2 code](https://github.com/snabbco/snabb/pull/1155).

On one side, we try to provide fast implementations of existing "languages"; on the other side we can't help but try out new co-designed domain-specific languages that can be expressive and run fast. As an example, with [pfmatch](https://wingolog.org/archives/2015/07/03/pfmatch-a-packet-filtering-language-embedded-in-lua) we extended pflang, the tcpdump language, with a more match-action kind of semantics. [It worked fine but the embedding between pfmatch and the host language could have been more smooth](https://www.asumu.xyz/blog/2017/01/13/making-a-snabb-app-with-pfmatch/); in the end the abstractions it offers don't really apply to what we have needed to build. For a long time we have been wondering if indeed there is a better domain-specific programming language to apply to the networking domain.

[P4](http://p4.org/) claims to be this language, and I think it's worth a look. P4's goal is be able to define switches and other networking equipment in software, with the specific goal that they would like to be able for P4 programs to be synthesized to ASICs, or installed in the FPGA of a ["Smart NIC"](https://www.google.fr/search?q=smart+nic), or compiled to CPUs. It's a wide target domain and the silicon-bakery side of things definitely constrains what is possible. Indeed P4 explicitly disclaims any ambition to be a general-purpose programming language. Still, I think they manage to achieve an admirable balance between declarative programming and transparent low-level compilability.

The best, most current intro to P4 out there is probably [Vladimir Gurevich's slides from last month's P4 "developer day" in California](http://p4.org/wp-content/uploads/2017/05/p4_d2_2017_p4_16_tutorial.pdf). I think it does a good job linking the language's syntax and semantics with how they are intended to be applied to the target domain. For a more PL-friendly and abstract introduction, the [P416 specification](https://p4lang.github.io/p4-spec/docs/P4-16-v1.0.0-spec.html) is a true delight.

Like I said, at work we build software switches and other network functions, and our target is commodity hardware. We write most of our work in [Snabb](https://github.com/snabbco/snabb), a powerful network toolkit built on LuaJIT, though we are branching out now to [VPP/fd.io](https://wiki.fd.io/view/VPP) as well, just to broaden the offering a bit. Generally we try to build solutions that don't have any dependencies other than a commodity Xeon server and a commodity NIC like Intel's 82599. So how could P4 help us in what we're doing?

My first thought in this regard was that if there is a library of P4 building blocks out there, that it would be really convenient to be able to incorporate a functional block written in P4 within the graph of a Snabb program. For example, if we have an IPFIX collector written in Snabb (and we do!), it would be cool to stick that in the middle of a P4 traffic conditioner.

(Immediately I run into the problem that I am straining my mind to think of a network function that we wouldn't rather just write in Snabb -- something valuable enough that we wouldn't want to "own" it and instead we would import someone else's black box into our data-plane. Maybe this interesting [in-network key-value cache](http://p4.org/wp-content/uploads/2017/06/p4-ws-2017-netcache.pdf) counts? But I digress, let's assume that something exists here.)

One question is, why bother doing P4 in software? I can understand that if you have 1Tbps ports that you definitely need custom silicon pushing your packets around. You would like to be able to program that silicon, so P4 looks to be a compelling step forward. But if your needs are satisfied with 40Gbps ports and you have chosen a software networking solution for its low cost, low lock-in, high flexibility, and sufficient performance -- well does P4 buy you something?

Right now it would seem that the answer is "no". A Cisco group wrote [a custom P4 compiler to VPP](http://www.cs.princeton.edu/~mshahbaz/papers/sosr17demos-pvpp.pdf), which is architecturally pretty much the same as Snabb, and they had to do some work to get the performance within a couple percent of the hand-coded application. The only win I can see is if people start building up libraries of reusable P4 components that can be linked together -- but the language itself currently doesn't support any more global composition primitive than `#include` (yes, it uses CPP :).

Additionally, at least as things are now, it doesn't seem that there's a library of reusable, open source P4 components out there to take advantage of. If this changes, I'll have to have another look. And of course it's worth keeping an eye on what kinds of [cool](http://p4.org/wp-content/uploads/2017/06/p4-ws-2017-netcache.pdf) [things](http://p4.org/wp-content/uploads/2017/06/02-p4-to-wireshark.pdf) people are building :)

*Thanks to Luke Gorrie for conversations leading to this blog post. All opinions and errors mine, of course!*

## related articles

- [notes from the fosdem 2018 networking devroom](/archives/2018/02/05/notes-from-the-fosdem-2018-networking-devroom)
- [encyclopedia snabb and the case of the foreign drivers](/archives/2017/02/24/encyclopedia-snabb-and-the-case-of-the-foreign-drivers)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [Pfmatch, a packet filtering language embedded in Lua](/archives/2015/07/03/pfmatch-a-packet-filtering-language-embedded-in-lua)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [six thoughts on generating c](/archives/2026/02/09/six-thoughts-on-generating-c)

Comments are closed.
