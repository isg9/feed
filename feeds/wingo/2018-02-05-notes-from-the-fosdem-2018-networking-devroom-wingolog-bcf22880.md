---
title: notes from the fosdem 2018 networking devroom — wingolog
url: https://wingolog.org/archives/2018/02/05/notes-from-the-fosdem-2018-networking-devroom
published: "2018-02-05T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2018/02/05/notes-from-the-fosdem-2018-networking-devroom
---

# notes from the fosdem 2018 networking devroom — wingolog

## [notes from the fosdem 2018 networking devroom](/archives/2018/02/05/notes-from-the-fosdem-2018-networking-devroom)

5 February 2018 5:22 PM

- [fosdem](/tags/fosdem)
- [networking](/tags/networking)
- [userspace](/tags/userspace)
- [snabb](/tags/snabb)
- [vpp](/tags/vpp)
- [dpdk](/tags/dpdk)
- [igalia](/tags/igalia)

Greetings, internet!

I am on my way back from [FOSDEM](https://fosdem.org/) and thought I would share with yall some impressions from talks in the [Networking devroom](https://fosdem.org/2018/schedule/track/sdn_and_nfv/). I didn't get to go to all that many talks -- FOSDEM's hallway track is the hottest of them all -- but I did hit a select few. Thanks to Dave Neary at Red Hat for organizing the room.

**Ray Kinsella -- Intel -- [The path to data-plane micro-services](https://fosdem.org/2018/schedule/event/dpdk_microservices/)**

The day started with a drum-beating talk that was very light on technical information.

Essentially Ray was arguing for an evolution of network function virtualization -- that instead of running VNFs on bare metal as was done in the days of yore, that people started to run them in virtual machines, and now they run them in containers -- what's next? Ray is saying that "cloud-native VNFs" are the next step.

Cloud-native VNFs to move from "greedy" VNFs that take charge of the cores that are available to them, to some kind of resource sharing. "Maybe users value flexibility over performance", says Ray. It's the Care Bears approach to networking: (resource) sharing is caring.

In practice he proposed two ways that VNFs can map to cores and cards.

One was in-process sharing, which if I understood him properly was actually as nodes running within a VPP process. Basically in this case VPP or DPDK is the scheduler and multiplexes two or more network functions in one process.

The other was letting Linux schedule separate processes. In networking, we don't usually do it this way: [we run network functions on dedicated cores on which nothing else runs](https://github.com/snabbco/snabb/blob/master/src/program/lwaftr/doc/performance.md). Ray was suggesting that perhaps network functions could be more like "normal" Linux services. Ray doesn't know if Linux scheduling will work in practice. Also it might mean allowing DPDK to work with 4K pages instead of the 2M hugepages it currently requires. This obviously has the potential for more latency hazards and would need some tighter engineering, and ultimately would have fewer guarantees than the "greedy" approach.

Interesting side things I noticed:

- All the diagrams show Kubernetes managing CPU node allocation and interface assignment. I guess in marketing diagrams, Kubernetes has completely replaced OpenStack.

- One slide showed guest VNFs differentiated between "virtual network functions" and "socket-based applications", the latter ones being the legacy services that use kernel APIs. It's a useful terminology difference.

- The talk identifies user-space networking with DPDK (only!).

Finally, I note that [Conway's law](https://www.wingolog.org/archives/2015/11/09/embracing-conways-law) is obviously reflected in the performance overheads: because there are organizational isolations between dev teams, vendors, and users, there are big technical barriers between them too. The least-overhead forms of resource sharing are also those with the highest technical consistency and integration (nodes in a single VPP instance).

**Magnus Karlsson -- Intel -- [AF\_XDP](https://fosdem.org/2018/schedule/event/af_xdp/)**

This was a talk about getting good throughput from the NIC to userspace, but by using some kernel facilities. The idea is to get the kernel to set up the NIC and virtualize the transmit and receive ring buffers, but to let the NIC's DMA'd packets go directly to userspace.

The performance goal is 40Gbps for thousand-byte packets, or 25 Gbps for traffic with only the smallest packets (64 bytes). The fast path does "zero copy" on the packets if the hardware has the capability to steer the subset of traffic associated with the AF\_XDP socket to that particular process.

The AF\_XDP project builds on [XDP](https://www.iovisor.org/technology/xdp), a newish thing where a little kind of bytecode can run on the kernel or possibly on the NIC. One of the bytecode commands (REDIRECT) causes packets to be forwarded to user-space instead of handled by the kernel's otherwise heavyweight networking stack. AF\_XDP is the bridge between XDP on the kernel side and an interface to user-space using sockets (as opposed to e.g. AF\_INET). The performance goal was to be within 10% or so of DPDK's raw user-space-only performance.

The benefits of AF\_XDP over the current situation would be that you have just one device driver, in the kernel, rather than having to have one driver in the kernel (which you have to have anyway) and one in user-space (for speed). Also, with the kernel involved, there is a possibility for better isolation between different processes or containers, when compared with raw PCI access from user-space..

AF\_XDP is what was previously known as AF\_PACKET v4, and its numbers are looking somewhat OK. Though it's not upstream yet, it might be interesting to get a Snabb driver here.

I would note that kernel-userspace cooperation is a bit of a theme these days. There are other points of potential cooperation or common domain sharing, storage being an obvious one. However I heard more than once this weekend the kind of "I don't know, that area of the kernel has a different culture" sort of concern as that highlighted by Daniel Vetter in his [recent LCA talk](https://lwn.net/Articles/745817/).

**François-Frédéric Ozog -- Linaro -- [Userland Network I/O](https://fosdem.org/2018/schedule/event/netmdev/)**

This talk is hard to summarize. Like the previous one, it's again about getting packets to userspace with some support from the kernel, but the speaker went really deep and I'm not quite sure what in the talk is new and what is known.

François-Frédéric is working on a new set of abstractions for relating the kernel and user-space. He works on OpenDataPlane (ODP), which is kinda like DPDK in some ways. ARM seems to be a big target for his work; that x86-64 is also a target goes without saying.

His problem statement was, how should we enable fast userland network I/O, without duplicating drivers?

François-Frédéric was a bit negative on AF\_XDP because (he says) it is so focused on packets that it neglects other kinds of devices with similar needs, such as crypto accelerators. Apparently the challenge here is accelerating a single large IPsec tunnel -- because the cryptographic operations are serialized, you need good single-core performance, and making use of hardware accelerators seems necessary right now for even a single 10Gbps stream. (If you had many tunnels, you could parallelize, but that's not the case here.)

He was also a bit skeptical about standardizing on the "packet array I/O model" which AF\_XDP and most NICS use. What he means here is that most current NICs move packets to and from main memory with the help of a "descriptor array" ring buffer that holds pointers to packets. A transmit array stores packets ready to transmit; a receive array stores maximum-sized packet buffers ready to be filled by the NIC. The packet data itself is somewhere else in memory; the descriptor only points to it. When a new packet is received, the NIC fills the corresponding packet buffer and then updates the "descriptor array" to point to the newly available packet. This requires at least two memory writes from the NIC to memory: at least one to write the packet data (one per 64 bytes of packet data), and one to update the DMA descriptor with the packet length and possible other metadata.

Although these writes go directly to cache, there's a limit to the number of DMA operations that can happen per second, and with 100Gbps cards, we can't afford to make one such transaction per packet.

François-Frédéric promoted an alternative I/O model for high-throughput use cases: the "tape I/O model", where packets are just written back-to-back in a uniform array of memory. Every so often a block of memory containing some number of packets is made available to user-space. This has the advantage of packing in more packets per memory block, as there's no wasted space between packets. This increases cache density and decreases DMA transaction count for transferring packet data, as we can use each 64-byte DMA write to its fullest. Additionally there's no side table of descriptors to update, saving a DMA write there.

Apparently the only cards currently capable of 100 Gbps traffic, the Chelsio and Netcope cards, use the "tape I/O model".

Incidentally, the DMA transfer limit isn't the only constraint. Something I hadn't fully appreciated before was memory write bandwidth. Before, I had thought that because the NIC would transfer in packet data directly to cache, that this wouldn't necessarily cause any write traffic to RAM. Apparently that's not the case. Later over drinks (thanks to Red Hat's networking group for organizing), François-Frédéric asserted that the DMA transfers would eventually use up DDR4 bandwidth as well.

A NIC-to-RAM DMA transaction will write one cache line (usually 64 bytes) to the socket's last-level cache. This write will evict whatever was there before. As far as I can tell, there are three cases of interest here. The best case is where the evicted cache line is from a previous DMA transfer to the same address. In that case it's modified in the cache and not yet flushed to main memory, and we can just update the cache instead of flushing to RAM. (Do I misunderstand the way caches work here? Do let me know.)

However if the evicted cache line is from some other address, we might have to flush to RAM if the cache line is dirty. That causes a memory write traffic. But if the cache line is clean, that means it was probably loaded as part of a memory read operation, and then that means we're evicting part of the network function's working set, which will later cause memory read traffic as the data gets loaded in again, and write traffic to flush out the DMA'd packet data cache line.

François-Frédéric simplified the whole thing to equate packet bandwidth with memory write bandwidth, that yes, the packet goes directly to cache but it is also written to RAM. I can't convince myself that that's the case for all packets, but I need to look more into this.

Of course the cache pressure and the memory traffic is worse if the packet data is less compact in memory; and worse still if there is any need to copy data. Ultimately, processing small packets at 100Gbps is still a huge challenge for user-space networking, and it's no wonder that there are only a couple devices on the market that can do it reliably, not that I've seen either of them operate first-hand :)

Talking with Snabb's Luke Gorrie later on, he thought that it could be that we can still stretch the packet array I/O model for a while, given that PCIe gen4 is coming soon, which will increase the DMA transaction rate. So that's a possibility to keep in mind.

At the same time, apparently there are some ["coherent interconnects"](https://www.ccixconsortium.com/) coming too which will allow the NIC's memory to be mapped into the "normal" address space available to the CPU. In this model, instead of having the NIC transfer packets to the CPU, the NIC's memory will be directly addressable from the CPU, as if it were part of RAM. The latency to pull data in from the NIC to cache is expected to be slightly longer than a RAM access; for comparison, RAM access takes about 70 nanoseconds.

For a user-space networking workload, coherent interconnects don't change much. You still need to get the packet data into cache. True, you do avoid the writeback to main memory, as the packet is already in addressable memory before it's in cache. But, if it's possible to keep the packet on the NIC -- like maybe you are able to add some kind of inline classifier on the NIC that could directly shunt a packet towards an on-board IPSec accelerator -- in that case you could avoid a lot of memory transfer. That appears to be the driving factor for coherent interconnects.

At some point in François-Frédéric's talk, my brain just died. I didn't quite understand all the complexities that he was taking into account. Later, after he kindly took the time to dispell some more of my ignorance, I understand more of it, though not yet all :) The concrete "deliverable" of the talk was a model for kernel modules and user-space drivers that uses the paradigms he was promoting. It's a work in progress from Linaro's networking group, with some support from NIC vendors and CPU manufacturers.

**Luke Gorrie and Asumu Takikawa -- SnabbCo and Igalia -- [How to write your own NIC driver, and why](https://fosdem.org/2018/schedule/event/lua_snabb/)**

This talk had the most magnificent beginning: a sort of "repent now ye sinners" sermon from Luke Gorrie, a seasoned veteran of software networking. Luke started by describing the path of righteousness leading to "driver heaven", a world in which all vendors have publically accessible datasheets which parsimoniously describe what you need to get packets flowing. In this blessed land it's easy to write drivers, and for that reason there are many of them. Developers choose a driver based on their needs, or they write one themselves if their needs are quite specific.

But there is another path, says Luke, that of "driver hell": a world of wickedness and proprietary datasheets, where even when you buy the hardware, you can't program it unless you're buying a hundred thousand units, and even then you are smitten with the cursed non-disclosure agreements. In this inferno, only a vendor is practically empowered to write drivers, but their poor driver developers are only incentivized to get the driver out the door deployed on all nine architectural circles of driver hell. So they include some kind of circle-of-hell abstraction layer, resulting in a hundred thousand lines of code like a tangled frozen beard. We all saw the abyss and repented.

Luke described the process that led to Mellanox releasing the specification for its ConnectX line of cards, something that was warmly appreciated by the entire audience, users and driver developers included. Wonderful stuff.

My Igalia colleague Asumu Takikawa took the last half of the presentation, showing some code for the driver for the Intel i210, i350, and 82599 cards. For more on that, I recommend his recent [blog post on user-space driver development](https://www.asumu.xyz/blog/2018/01/15/supporting-both-vmdq-and-rss-in-snabb). It was truly a ray of sunshine in dark, dark Brussels.

**Ole Trøan -- Cisco -- [Fast dataplanes with VPP](https://fosdem.org/2018/schedule/event/vnf_vpp/)**

This talk was a delightful introduction to [VPP](https://wiki.fd.io/view/VPP), but without all of the marketing; the sort of talk that makes FOSDEM worthwhile. Usually at more commercial, vendory events, you can't really get close to the technical people unless you have a vendor relationship: they are surrounded by a phalanx of salesfolk. But in FOSDEM it is clear that we are all comrades out on the open source networking front.

The speaker expressed great personal pleasure on having being able to work on open source software; his relief was palpable. A nice moment.

He also had some kind words about Snabb, too, saying at one point that "of course you can do it on snabb as well -- Snabb and VPP are quite similar in their approach to life". He trolled the horrible complexity diagrams of many "NFV" stacks whose components reflect the org charts that produce them more than the needs of the network functions in question (service chaining anyone?).

He did get to drop some numbers as well, which I found interesting. One is that recently they have been working on carrier-grade NAT, aiming for 6 terabits per second. Those are pretty big boxes and I hope they are getting paid appropriately for that :) For context he said that for a 4-unit server, these days you can build one that does a little less than a terabit per second. I assume that's with ten dual-port 40Gbps cards, and I would guess to power that you'd need around 40 cores or so, split between two sockets.

Finally, he finished with a long example on lightweight 4-over-6. Incidentally this is the same network function my group at Igalia has been building in Snabb over the last couple years, so it was interesting to see the comparison. I enjoyed his commentary that although all of these technologies (carrier-grade NAT, MAP, lightweight 4-over-6) have the ostensible goal of keeping IPv4 running, in reality "we're day by day making IPv4 work worse", mainly by breaking the assumption that just because you get traffic from port P on IP M, doesn't mean you can send traffic to M from another port or another protocol and have it reach the target.

All of these technologies also have problems with IPv4 fragmentation. Getting it right is possible but expensive. Instead, Ole mentions that he and a cross-vendor cabal of dataplane people have a "dark RFC" in the works to deprecate IPv4 fragmentation entirely :)

OK that's it. If I get around to writing up the couple of interesting Java talks I went to (I know right?) I'll let yall know. Happy hacking!

## related articles

- [an early look at p4 for software networking](/archives/2017/06/26/an-early-look-at-p4-for-software-networking)
- [encyclopedia snabb and the case of the foreign drivers](/archives/2017/02/24/encyclopedia-snabb-and-the-case-of-the-foreign-drivers)
- [whippet at fosdem](/archives/2025/02/10/whippet-at-fosdem)
- [whippet: towards a new local maximum](/archives/2023/02/07/whippet-towards-a-new-local-maximum)
- [lightweight concurrency in lua](/archives/2018/05/16/lightweight-concurrency-in-lua)
- [Pfmatch, a packet filtering language embedded in Lua](/archives/2015/07/03/pfmatch-a-packet-filtering-language-embedded-in-lua)

### One response

1. [Max Rottenkolber](http://mr.gy) says:[6 February 2018 1:14 PM](#a7a5217b2008bd20fea292377bd323742930c776)

   Awesome write up, thanks!

Comments are closed.
