---
title: new month, new brainworm — wingolog
url: https://wingolog.org/archives/2022/09/01/new-month-new-brainworm
published: "2022-09-01T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2022/09/01/new-month-new-brainworm
---

# new month, new brainworm — wingolog

## [new month, new brainworm](/archives/2022/09/01/new-month-new-brainworm)

1 September 2022 10:12 AM

- [webassembly](/tags/webassembly)
- [wasi](/tags/wasi)
- [component model](/tags/component%20model)
- [kubernetes](/tags/kubernetes)
- [igalia](/tags/igalia)
- [compilers](/tags/compilers)
- [containers](/tags/containers)

Today, a brainworm! I had a thought a few days ago and can't get it out of my head, so I need to pass it on to another host.

So, imagine a world in which [there is a a drive to build a kind of Kubernetes on top of WebAssembly](https://wingolog.org/archives/2021/12/13/webassembly-the-new-kubernetes). Kubernetes nodes are generally [containers](https://github.com/opencontainers/image-spec/blob/main/spec.md), associated with additional metadata indicating their place in overall system topology (network connections and so on). (I am not a Kubernetes specialist, as you can see; corrections welcome.) Now in a WebAssembly cloud, the nodes would be [components](https://github.com/WebAssembly/component-model), probably also with additional topological metadata. VC-backed companies will duke it out for dominance of the WebAssembly cloud space, and in a couple years we will probably emerge with an open source project that has become a de-facto standard (though it might be dominated by one or two players).

In this world, Kubernetes and Spiffy-Wasm-Cloud will coexist. One of the success factors for Kubernetes was that you can just put your old database binary inside a container: it's the same ABI as when you run your database in a virtual machine, or on (so-called!) bare metal. The means of composition are TCP and UDP network connections between containers, possibly facilitated by some kind of network fabric. In contrast, in Spiffy-Wasm-Cloud we aren't starting from the kernel ABI, with processes and such: instead there's [WASI](https://github.com/WebAssembly/WASI/blob/main/README.md), which is more of a kind of specialized and limited `libc`. You can't just drop in your database binary, you have to write code to get it to conform to the new interfaces.

One consequence of this situation is that I expect WASI and the component model to develop a rich network API, to allow WebAssembly components to interoperate not just with end-users but also other (micro-)services running in the same cloud. Likewise there is room here for a company to develop some complicated network fabrics for linking these things together.

However, WebAssembly-to-WebAssembly links are better expressed via typed functional interfaces; it's more expressive and can be faster. Not only can you end up having [fine-grained composition that looks more like lightweight Erlang processes](https://github.com/WebAssembly/meetings/blob/main/wasi/2022/presentations/2022-06-16-luke-async.pdf), you can also string together components in a pipeline with communications overhead approaching that of a simple function call. Relative to Kubernetes, there are potential 10x-100x improvements to be had, in throughput and in memory footprint, at least in some cases. It's the promise of this kind of improvement that can drive investment in this area, and eventually adoption.

But, you still have some legacy things running in containers. What to do? Well... Maybe recompile them to WebAssembly? That's my brain-worm.

A container is a file system image containing executable files and data. Starting with the executable files, they are in machine code, generally x64, and interoperate with system libraries and the run-time via an ABI. You could compile them to WebAssembly instead. You could interpret them as data, or JIT-compile them as [webvm](https://leaningtech.com/webvm-server-less-x86-virtual-machines-in-the-browser/) does, or directly compile them to WebAssembly. This is the sort of thing you hire [Fabrice Bellard](https://bellard.org/) to do ;) Then you have the filesystem. Let's assume it is stateless: any change to the filesystem at runtime doesn't need to be preserved. (I understand this is a goal, though I could be wrong.) So you could put the filesystem in memory, as some kind of addressable data structure, and you make the libc interface access that data structure. It's something like the microkernel approach. And then you translate whatever topological connectivity metadata you had for Kubernetes to your Spiffy-Wasm-Cloud's format.

Anyway in the end you have a WebAssembly module and some metadata, and you can run it in your WebAssembly cloud. Or on the more basic level, you have a container and you can now run it on any machine with a WebAssembly implementation, even on other architectures (coucou RISC-V!).

Anyway, that's the tweet. Have fun, whoever gets to work on this :)

## related articles

- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)

Comments are closed.
