---
title: encyclopedia snabb and the case of the foreign drivers — wingolog
url: https://wingolog.org/archives/2017/02/24/encyclopedia-snabb-and-the-case-of-the-foreign-drivers
published: "2017-02-24T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2017/02/24/encyclopedia-snabb-and-the-case-of-the-foreign-drivers
---

# encyclopedia snabb and the case of the foreign drivers — wingolog

## [encyclopedia snabb and the case of the foreign drivers](/archives/2017/02/24/encyclopedia-snabb-and-the-case-of-the-foreign-drivers)

24 February 2017 5:37 PM

- [snabb](/tags/snabb)
- [igalia](/tags/igalia)
- [networking](/tags/networking)
- [dpdk](/tags/dpdk)
- [drivers](/tags/drivers)
- [lca](/tags/lca)
- [lwn](/tags/lwn)
- [conway's law](/tags/conway%27s%20law)

Peoples of the blogosphere, welcome back to the solipsism! Happy 2017 and all that. Today's missive is about [Snabb](https://github.com/SnabbCo/snabb) (formerly Snabb Switch), a high-speed networking project we've been working on at [work](https://www.igalia.com/networking) for some years now.

What's Snabb all about you say? Good question and I have a nice answer for you in video and third-party textual form! This year I managed to make it to [linux.conf.au](http://lca2017.linux.org.au/) in lovely Tasmania. Tasmania is amazing, with wild wombats and pademelons and devils and wallabies and all kinds of things, *and* they let me talk about Snabb.

[Click to download video](//wingolog.org/pub/lca2017-wingo-snabb.webm)

You can check that video [on the youtube](https://www.youtube.com/watch?v=E9VI9bUkvIM) if the link above doesn't work; [slides here](https://wingolog.org/pub/lca-2017-slides.pdf).

Jonathan Corbet from [LWN](https://lwn.net/) wrote up the talk in an [article here](https://lwn.net/Articles/713918/), which besides being flattering is a real windfall as I don't have to write it up myself :)

In that talk I mentioned that Snabb uses its own drivers. We were recently approached by a customer with a simple and honest question: does this really make sense? Is it really a win? Why wouldn't we just use the work that the NIC vendors have already put into their drivers for the [Data Plane Development Kit (DPDK)](http://dpdk.org)? After all, part of the attraction of a switch to open source is that you will be able to take advantage of the work that others have produced.

Our answer is that while it is indeed possible to use drivers from DPDK, there are costs and benefits on both sides and we think that when we weigh it all up, it makes both technical and economic sense for Snabb to have its own driver implementations. It might sound counterintuitive on the face of things, so I wrote this long article to discuss some perhaps under-appreciated points about the tradeoff.

Technically speaking there are generally two ways you can imagine incorporating DPDK drivers into Snabb:

1. Bundle a snapshot of the DPDK into Snabb itself.

2. Somehow make it so that Snabb could (perhaps optionally) compile against a built DPDK SDK.

As part of a software-producing organization that ships solutions based on Snabb, I need to be able to ship a "known thing" to customers. When we ship the lwAFTR, we ship it in source and in binary form. For both of those deliverables, we need to know exactly what code we are shipping. We achieve that by having a minimal set of dependencies in Snabb -- only LuaJIT and three Lua libraries (DynASM, ljsyscall, and pflua) -- and we [include those dependencies directly in the source tree](https://github.com/snabbco/snabb/tree/master/lib). This requirement of ours rules out (2), so the option under consideration is only (1): importing the DPDK (or some part of it) directly into Snabb.

So let's start by looking at Snabb and the DPDK from the top down, comparing some metrics, seeing how we could make this combination.

snabbdpdkCode lines61K583KContributors (all-time)60370Contributors (since Jan 2016)32240Non-merge commits (since Jan 2016)1.4K3.2K

These numbers aren't directly comparable, of course; in Snabb our unit of code change is the merge rather than the commit, and in Snabb we include a number of production-ready applications like the lwAFTR and the NFV, but they are fine enough numbers to start with. What seems clear is that the DPDK project is significantly larger than Snabb, so adding it to Snabb would fundamentally change the nature of the Snabb project.

So depending on the DPDK makes it so that suddenly Snabb jumps from being a project that compiles in a minute to being a much more heavy-weight thing. That could be OK if the benefits were high enough and if there weren't other costs, but there are indeed other costs to including the DPDK:

- Data-plane control. Right now when I ship a product, I can be responsible for the whole data plane: everything that happens on the CPU when packets are being processed. This includes the driver, naturally; it's part of Snabb and if I need to change it or if I need to understand it in some deep way, I can do that. But if I switch to third-party drivers, this is now out of my domain; there's a wall between me and something that running on my CPU. And if there is a performance problem, I now have someone to blame that's not myself! From the customer perspective this is terrible, as you want the responsibility for software to rest in one entity.

- Impedance-matching development costs. Snabb is written in Lua; the DPDK is written in C. I will have to build a bridge, *and keep it up to date* as both Snabb and the DPDK evolve. This impedance-matching layer is also another source of bugs; either we make a local impedance matcher in C or we bind everything using LuaJIT's FFI. In the former case, it's a lot of duplicate code, and in the latter we lose compile-time type checking, which is a no-go given that the DPDK can and does change API and ABI.

- Communication costs. The DPDK development list had [3K messages in January](http://dpdk.org/ml/archives/dev/2017-January/). Keeping up with DPDK development would become necessary, as the DPDK is now in your dataplane, but it costs significant amounts of time.

- Costs relating to mismatched goals. Snabb tries to win development and run-time speed by searching for simple solutions. The DPDK tries to be a showcase for NIC features from vendors, placing less of a priority on simplicity. This is a very real cost in the form of the way network packets are represented in the DPDK, with support for such features as scatter/gather and indirect buffers. In Snabb we were able to do away with this complexity by having simple linear buffers, and our speed did not suffer; adding the DPDK again would either force us to marshal and unmarshal these buffers into and out of the DPDK's format, or otherwise to reintroduce this particular complexity into Snabb.

- Abstraction costs. A network function written against the DPDK typically uses at least three abstraction layers: the "EAL" environment abstraction layer, the "PMD" poll-mode driver layer, and often an internal hardware abstraction layer from the network card vendor. (And some of those abstraction layers are actually external dependencies of the DPDK, as with Mellanox's ConnectX-4 drivers!) Any discrepancy between the goals and/or implementation of these layers and the goals of a Snabb network function is a cost in developer time and in run-time. Note that those low-level HAL facilities aren't considered acceptable in upstream Linux kernels, for all of these reasons!

- Stay-on-the-train costs. The DPDK is big and sometimes its abstractions change. As a minor player just riding the DPDK train, we would have to invest a continuous amount of effort into just staying aboard.

- Fork costs. The Snabb project has a number of contributors but is really run by Luke Gorrie. Because Snabb is so small and understandable, if Luke decided to stop working on Snabb or take it in a radically different direction, I would feel comfortable continuing to maintain (a fork of) Snabb for as long as is necessary. If the DPDK changed goals for whatever reason, I don't think I would want to continue to maintain a stale fork.

- Overkill costs. Drivers written against the DPDK have many considerations that simply aren't relevant in a Snabb world: kernel drivers (KNI), special NIC features that we don't use in Snabb (RDMA, offload), non-x86 architectures with different barrier semantics, threads, complicated buffer layouts (chained and indirect), interaction with specific kernel modules (uio-pci-generic / igb-uio / ...), and so on. We don't need all of that, but we would have to bring it along for the ride, and any changes we might want to make would have to take these use cases into account so that other users won't get mad.

So there are lots of costs if we were to try to hop on the DPDK train. But what about the benefits? The goal of relying on the DPDK would be that we "automatically" get drivers, and ultimately that a network function would be driver-agnostic. But this is not necessarily the case. Each driver has its own set of quirks and tuning parameters; in order for a software development team to be able to support a new platform, the team would need to validate the platform, discover the right tuning parameters, and modify the software to configure the platform for good performance. Sadly this is not a trivial amount of work.

Furthermore, using a different vendor's driver isn't always easy. Consider Mellanox's DPDK ConnectX-4 / ConnectX-5 support: the ["Quick Start"](http://www.mellanox.com/related-docs/prod_software/MLNX_DPDK_Quick_Start_Guide_v2%201_1%201.pdf) guide has you first install [MLNX\_OFED](http://www.mellanox.com/page/products_dyn?product_family=26) in order to build the DPDK drivers. What is this thing exactly? You go to download the tarball and it's 55 megabytes. What's in it? 30 other tarballs! If you build it somehow from source instead of using the vendor binaries, then what do you get? All that code, running as root, with kernel modules, and implementing systemd/sysvinit services!!! And this is just step one!!!! Worse yet, this enormous amount of code powering a DPDK driver is mostly driver-specific; what we hear from colleagues whose organizations decided to bet on the DPDK is that you don't get to amortize much knowledge or validation when you switch between an Intel and a Mellanox card.

In the end when we ship a solution, it's going to be tested against a specific NIC or set of NICs. Each NIC will add to the validation effort. So if we were to rely on the DPDK's drivers, we would have payed all the costs but we wouldn't save very much in the end.

There is another way. Instead of relying on so much third-party code that it is impossible for any one person to grasp the entirety of a network function, much less be responsible for it, we can build systems small enough to understand. In Snabb we just read the data sheet and write a driver. (Of course we also benefit by looking at DPDK and other open source drivers as well to see how they structure things.) By only including what is needed, Snabb drivers are typically only a [thousand or two thousand lines of Lua](https://github.com/snabbco/snabb/tree/master/src/apps/intel). With a driver of that size, it's possible for even a small ISV or in-house developer to "own" the entire data plane of whatever network function you need.

Of course Snabb drivers have costs too. What are they? Are customers going to be stuck forever paying for drivers for every new card that comes out? It's a very good question and one that I know is in the minds of many.

Obviously I don't have the whole answer, as my role in this market is a software developer, not an end user. But having talked with other people in the Snabb community, I see it like this: Snabb is still in relatively early days. What we need are about three good drivers. One of them should be for a standard workhorse commodity 10Gbps NIC, which we have in the Intel 82599 driver. That chipset has been out for a while so we probably need to update it to the current commodities being sold. Additionally we need a couple cards that are going to compete in the 100Gbps space. We have the Mellanox ConnectX-4 and presumably ConnectX-5 drivers on the way, but there's room for another one. We've found that it's hard to actually get good performance out of 100Gbps cards, so this is a space in which NIC vendors can differentiate their offerings.

We budget somewhere between 3 and 9 months of developer time to create a completely new Snabb driver. Of course it usually takes less time to develop Snabb support for a NIC that is only incrementally different from others in the same family that already have drivers.

We see this driver development work to be similar to the work needed to validate a new NIC for a network function, with the additional advantage that it gives us up-front knowledge instead of the best-effort testing later in the game that we would get with the DPDK. When you add all the additional costs of riding the DPDK train, we expect that the cost of Snabb-native drivers competes favorably against the cost of relying on third-party DPDK drivers.

In the beginning it's natural that early adopters of Snabb make investments in this base set of Snabb network drivers, as they would to validate a network function on a new platform. However over time as Snabb applications start to be deployed over more ports in the field, network vendors will also see that it's in their interests to have solid Snabb drivers, just as they now see with the Linux kernel and with the DPDK, and given that the investment is relatively low compared to their already existing efforts in Linux and the DPDK, it is quite feasible that we will see the NIC vendors of the world start to value Snabb for the performance that it can squeeze out of their cards.

So in summary, in Snabb we are convinced that writing minimal drivers that are adapted to our needs is an overall win compared to relying on third-party code. It lets us ship solutions that we can feel responsible for: both for their operational characteristics as well as their maintainability over time. Still, we are happy to learn and share with our colleagues all across the open source high-performance networking space, from the DPDK to VPP and beyond.

## related articles

- [notes from the fosdem 2018 networking devroom](/archives/2018/02/05/notes-from-the-fosdem-2018-networking-devroom)
- [an early look at p4 for software networking](/archives/2017/06/26/an-early-look-at-p4-for-software-networking)
- [missing the point of webassembly](/archives/2024/01/08/missing-the-point-of-webassembly)
- [lightweight concurrency in lua](/archives/2018/05/16/lightweight-concurrency-in-lua)
- [embracing conway's law](/archives/2015/11/09/embracing-conways-law)
- [Pfmatch, a packet filtering language embedded in Lua](/archives/2015/07/03/pfmatch-a-packet-filtering-language-embedded-in-lua)

### 2 responses

1. fd says:[25 February 2017 8:47 PM](#93cb7e27ba520ddedcbd7b23b5757ed05bdb4e36)

   I enjoyed the youtube video of your presentation.

   I presume snabb doesn't copy packets between apps, so I'm wondering how packet buffer memory is managed along an entire chain of apps. Is ownership of a buffer transferred from one app to the next during link receive/transmit operations? Are buffers reference-counted so they can be shared among apps (think fan-outs)? If buffers are reference-counted, are they read-only? If an app desires to modify a buffer, can it obtain exclusive access? Or does it need to make a copy? etc.

2. [Blake](http://l33.fr/) says:[21 March 2017 7:36 AM](#beda8785b88149e52be240a5f69ae8430a17fa0f)

   Thanks for keeping the code clean and simple, Andy!

   Hopefully the driver code in VPP will help you accelerate development of new Snabb drivers as well.

Comments are closed.
