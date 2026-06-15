---
title: missing the point of webassembly — wingolog
url: https://wingolog.org/archives/2024/01/08/missing-the-point-of-webassembly
published: "2024-01-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2024/01/08/missing-the-point-of-webassembly
---

# missing the point of webassembly — wingolog

## [missing the point of webassembly](/archives/2024/01/08/missing-the-point-of-webassembly)

8 January 2024 11:45 AM

- [webassembly](/tags/webassembly)
- [wasm](/tags/wasm)
- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [conway's law](/tags/conway%27s%20law)
- [hoot](/tags/hoot)
- [linux](/tags/linux)
- [ebpf](/tags/ebpf)
- [containers](/tags/containers)
- [virtualization](/tags/virtualization)
- [igalia](/tags/igalia)

I find most descriptions of WebAssembly to be uninspiring: if you start with a phrase like “assembly-like language” or a “virtual machine”, we have already lost the plot. That’s not to say that these descriptions are incorrect, but it’s like explaining what a dog is by starting with its circulatory system. You’re not wrong, but you should probably lead with the bark.

I have a different preferred starting point which is less descriptive but more operational: *WebAssembly is a new fundamental abstraction* *boundary*. WebAssembly is a new way of dividing computing systems into pieces and of composing systems from parts.

This all may sound high-falutin´, but it’s for real: this is the actually interesting thing about Wasm.

### fundamental & abstract

It’s probably easiest to explain what I mean by example. Consider the Linux ABI: Linux doesn’t care what code it’s running; Linux just handles system calls and schedules process time. Programs that run against the x86-64 Linux ABI don’t care whether they are in a container or a virtual machine or “bare metal” or whether the processor is AMD or Intel or even a Mac M3 with Docker and Rosetta 2. The Linux ABI interface is *fundamental* in the sense that either side can implement any logic, subject to the restrictions of the interface, and *abstract* in the sense that the universe of possible behaviors has been simplified to a limited language, in this case that of system calls.

Or take HTTP: when you visit [`wingolog.org`](https://wingolog.org/), you don’t have to know (but surely would be delighted to learn) that it’s Scheme code that handles the request. I don’t have to care if the other side of the line is curl or Firefox or [Wolvic](https://www.wolvic.com/en/). HTTP is such a successful fundamental abstraction boundary that at this point it is the default for network endpoints; whether you are a database or a golang microservice, if you don’t know that you need a custom protocol, you use HTTP.

Or, to rotate our metaphorical compound microscope to high-power magnification, consider the SYS-V amd64 C ABI: almost every programming language supports some form of `extern C {}` to *access* external libraries, and the best language implementations can produce artifacts that *implement* the C ABI as well. The standard C ABI splits programs into parts, and allows works from separate teams to be composed into a whole. Indeed, one litmus test of a fundamental abstraction boundary is, could I reasonably define an interface and have an implementation of it be in Scheme or OCaml or what-not: if the answer is yes, we are in business.

It is in this sense that WebAssembly is a new fundamental abstraction boundary.

WebAssembly shares many of the concrete characteristics of other abstractions. Like the Linux syscall interface, WebAssembly defines an interface language in which programs rely on host capabilities to access system features. Like the C ABI, calling into WebAssembly code has a predictable low cost. Like HTTP, you can arrange for WebAssembly code to have no shared state with its host, by construction.

But WebAssembly is a new point in this space. Unlike the Linux ABI, there is no fixed set of syscalls: WebAssembly imports are named, typed, and without pre-defined meaning, more like the C ABI. Unlike the C ABI, WebAssembly modules have only the shared state that they are given; neither side has a license to access all of the memory in the “process”. And unlike HTTP, WebAssembly modules are “in the room” with their hosts: close enough that hosts can allow themselves the luxury of synchronous function calls, and to allow WebAssembly modules to synchronously call back into their hosts.

### applied teleology

At this point, you are probably nodding along, but also asking yourself, *what is it for*? If you arrive at this question from the “WebAssembly is a virtual machine” perspective, I don’t think you’re well-equipped to answer. But starting as we did by the interface, I think we are better positioned to appreciate how WebAssembly fits into the computing landscape: the narrative is generative, in that you can explore potential niches by identifying existing abstraction boundaries.

Again, let’s take a few examples. Say you ship some “smart cities” IoT device, consisting of a microcontroller that runs some non-Linux operating system. The system doesn’t have an MMU, so you don’t have hardware memory protections, but you would like to be able to enforce some invariants on the software that this device runs; and you would also like to be able to update that software over the air. WebAssembly is getting used in these environments; I wish I had a list of deployments at hand, but perhaps we can at least take [this article last
year from a WebAssembly IoT
vendor](https://thenewstack.io/why-webassembly-is-perfect-for-tiny-iot-devices/) as proof of commercial interest.

Or, say you run a function-as-a-service cloud, meaning that you run customer code in response to individual API requests. You need to limit the allowable set of behaviors from the guest code, so you choose some abstraction boundary. You could use virtual machines, but that would be quite expensive in terms of memory. You could use containers, but you would like more control over the guest code. You could have these functions written in JavaScript, but that means that your abstraction is no longer fundamental; you limit your applicability. WebAssembly fills an interesting niche here, and there are a number of products in this space, for example [Fastly
Compute](https://docs.fastly.com/products/compute) or [Fermyon
Spin](https://www.fermyon.com/spin).

Or to go smaller, consider extensible software, like the [GIMP image
editor](https://www.gimp.org/) or [VS Code](https://vscodium.com/): in the past you would use loadable plug-in modules via the C ABI, which can be quite gnarly, or you lean into a particular scripting language, which can be slow, inexpressive, and limit the set of developers that can write extensions. It’s not a silver bullet, but WebAssembly can have a role here. For example, the [Harfbuzz](https://github.com/harfbuzz/harfbuzz) text shaping library supports [fonts with an embedded (em-behdad?) WebAssembly
extension](https://github.com/harfbuzz/harfbuzz/blob/main/docs/wasm-shaper.md) to control how strings of characters are mapped to positioned glyphs.

### aside: what boundaries do

They say that good fences make good neighbors, and though I am not quite sure it is true—since my neighbor put up a fence a few months ago, our kids don’t play together any more—boundaries certainly facilitate separation of functionality. [Conway’s
law](https://www.wingolog.org/archives/2015/11/09/embracing-conways-law) is sometimes applied as a descriptive observation—ha-ha, isn’t that funny, they just shipped their org chart—but this again misses the point, in that boundaries facilitate separation, but also composition: if I know that I can fearlessly allow a font to run code because I have an appropriate abstraction boundary between host application and extension, I have gained in power. I no longer need to be responsible for every part of the product, and my software can scale up to solve harder problems by composing work from multiple teams.

There is little point in using WebAssembly if you control both sides of a boundary, just as (unless you have chickens) there is little point in putting up a fence that runs through the middle of your garden. But where you *want* to compose work from separate teams, the boundaries imposed by WebAssembly can be a useful tool.

### narrative generation

WebAssembly is enjoying a tail-wind of hype, so I think it’s fair to say that wherever you find a fundamental abstraction boundary, someone is going to try to implement it with WebAssembly.

Again, some examples: back in 2022 I [speculated that someone would
“compile” Docker containers to WebAssembly
modules](https://wingolog.org/archives/2022/09/01/new-month-new-brainworm), and now [that is a thing](https://github.com/ktock/container2wasm).

I think at some point someone will attempt to replace [eBPF](https://ebpf.io/) with Wasm in the Linux kernel; eBPF is just not as good a language as Wasm, and the toolchains that produce it are worse. eBPF has clunky calling-conventions about what registers are saved and spilled at call sites, a decision that can be made more efficiently for the program and architecture at hand when register-allocating WebAssembly locals. (Sometimes people lean on the provably-terminating aspect of eBPF as its virtue, but that could apply just as well to Wasm if you prohibit the [`loop`
opcode](https://webassembly.github.io/spec/core/exec/instructions.html#exec-loop) (and the tail-call instructions) at verification-time.) And why don’t people write whole device drivers in eBPF? Or rather, targetting eBPF from C or what-have-you. It’s because eBPF is just not good enough. WebAssembly is, though! Anyway I think Linux people are too chauvinistic to pick this idea up but I bet Microsoft could do it.

I was thinking today, you know, it actually makes sense to run a WebAssembly operating system, one which runs WebAssembly binaries. If the operating system includes the Wasm run-time, it can interpose itself at syscall boundaries, sometimes allowing it to avoid context switches. You could start with something like the Linux ABI, perhaps via [WALI](https://arxiv.org/abs/2312.03858), but for a subset of guest processes that conform to particular conventions, you could build purpose-built composition that can allocate multiple WebAssembly modules to a single process, eliding inter-process context switches and data copies for streaming computations. Or, focussing on more restricted use-cases, you could make a microkernel; googling around I found [this article from a
couple days ago where someone is giving this a
go](https://medium.com/@r1ru/wasmos-a-proof-of-concept-microkernel-that-runs-webassembly-natively-850043cad121).

### wwwhat about the wwweb

But let’s go back to the web, where you are reading this. In one sense, WebAssembly is a massive web success, being deployed to literally billions of user agents. In another, it is marginal: people do not write web front-ends in WebAssembly. Partly this is because the kind of abstraction supported by linear-memory WebAssembly 1.0 isn’t a good match for the garbage-collected DOM API exposed by web browsers. As a corrolary, languages that are most happy targetting this linear-memory model (C, Rust, and so on) aren’t good for writing DOM applications either. WebAssembly is used in auxiliary modules where you want to run legacy C++ code on user devices, or to speed up a hot leaf function, but isn’t a huge success.

This will change with the recent addition of managed data types to WebAssembly, but not necessarily in the way that you might think. Like, now that it will be cheaper and more natural to pass data back and forth with JavaScript, are we likely to see Wasm/GC progressively occupying more space in web applications? For me, I doubt that *progressive* is the word. In the same way that you wouldn’t run a fence through the middle of your front lawn, you wouldn’t want to divide your front-end team into JavaScript and WebAssembly sub-teams. Instead I think that we will see more phase transitions, in which whole web applications switch from JavaScript to Wasm/GC, compiled from Dart or Elm or what have you. The natural fundamental abstraction boundary in a web browser is between the user agent and the site’s code, not within the site’s code itself.

### conclusion

So, friends, if you are talking to a compiler engineer, by all means: keep describing WebAssembly as an virtual machine. It will keep them interested. But for everyone else, the value of WebAssembly is what it does, which is to be a different way of breaking a system into pieces. Armed with this observation, we can look at current WebAssembly uses to understand the nature of those boundaries, and to look at new boundaries to see if WebAssembly can have a niche there. Happy hacking, and may your components always compose!

## related articles

- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [wastrel milestone: full hoot support, with generational gc as a treat](/archives/2026/04/09/wastrel-milestone-full-hoot-support-with-generational-gc-as-a-treat)
- [ahead-of-time wasm gc in wastrel](/archives/2026/02/06/ahead-of-time-wasm-gc-in-wastrel)
- [a world to win: webassembly for the rest of us](/archives/2023/03/20/a-world-to-win-webassembly-for-the-rest-of-us)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [the last couple years in v8's garbage collector](/archives/2025/11/13/the-last-couple-years-in-v8s-garbage-collector)

### 10 responses

1. Otto Jung says:[8 January 2024 4:14 PM](#a9306db5cd29c47cd056937a7e29c887ee19c288)

   What I understood reading this is that every virtual machine with an FFI is a fundamental abstraction boundary.

   And what’s important in the case of WebAssembly is that its abstraction boundary is expected to be as influential as the C ones.

   Do you agree?

2. [jfred](https://jfred.dreamwidth.org) says:[8 January 2024 6:02 PM](#d59a365857238ef2930781509fc9e5358f39edc9)

   Thanks for this, was a fun read. One thing I’d like to touch on a bit though:

   The natural fundamental abstraction boundary in a web browser is between the user agent and the site’s code, not within the site’s code itself.

   This is certainly true of the parts of the site’s code written by the site’s author, but many/most webapps these days are made up largely of other people’s code in the form of JS dependencies like React, analytics libraries, etc. A better abstraction boundary between modules within a site’s code would also be welcome for many of the same reasons you mentioned.

3. [jfred](https://jfred.dreamwidth.org) says:[8 January 2024 6:03 PM](#5029ac47f059117be3d2e4c962ce13c19aaf8780)

   Oops, the comment system seems to have stripped angle brackets. The second paragraph in my comment above was meant to be a blockquote.

4. [Steve Manuel](https://dylibso.com) says:[8 January 2024 7:13 PM](#8ff9d180c984ff84948cde4e9f3982732ab0f1fc)

   I no longer need to be responsible for every part of the product, and my software can scale up to solve harder problems by composing work from multiple teams.

   This, to me, is the most exciting (concrete) opportunity to change the way we build software. WebAssembly brings the portable, secure sandbox that every application should use.

   I’m working to make this eventual future happen faster with https://github.com/extism/extism, which was initially designed to be the plugin system for everyone, and has since been used for many other things (code generators, FaaS, web apps & more)

   I think you’re dead-on with the opener here... \> I find most descriptions of WebAssembly to be uninspiring: if you start with a phrase like “assembly-like language” or a “virtual machine”, we have already lost the plot.

5. Shai says:[9 January 2024 7:13 PM](#351720c17f3829193aea2b9d44a6467af4c658ae)

   WebAssembly is like a JVM from the late 90s only not as good... These abstractions aren’t new. The trending direction is wrong. E.g. we moved from VMs to Containers to get better performance from the bare metal. Why go in the exact opposite way for no technical benefit?

   Engineers always wanted stuff like that because it’s cool. Polyglot and abstractions are cool. But they make bad engineering. Java makes sense since it built this HUGE ecosystem. The enterprise capabilities alone are amazing and will take decades for anyone to approach. WASM makes absolutely no sense for anyone other than hackers.

6. Great says:[9 January 2024 8:24 PM](#36184bea8e4f22aeb9dfb6d50c8593e3fe2bc1b8)

   Oh good because we need more of those.

   I’ll stick with a languages primitives and focus on the boundaries that solve the problem at hand.

   IT is deep into pseudoscience territory these days. Still just managed electron state in a machine.

   Also 34.9999999999999999… is between 34 and 42. The form rejected it. Not really coming off as authoritative voice in computer science. Maybe focus more on the science less on contemporary machine abstractions

7. Mike Bentley says:[9 January 2024 10:31 PM](#61e11b53b2454cdbbf26ce3468bac261debca552)

   “I have a different preferred starting point which is less descriptive but more operational: WebAssembly is a new fundamental abstraction boundary. WebAssembly is a new way of dividing computing systems into pieces and of composing systems from parts.”

   I think if you rearrange this paragraph, you’ll improve the article by having a clear and explicit definition of fundamental abstraction boundary. You have a bunch of implicit descriptions scattered about but you don’t come out and say what it is.

8. Bob Young says:[10 January 2024 2:16 PM](#679d30bb045f32ddd9082024e426471b3a4e2dd7)

   With containers, any language can work, including complex compositions of scripts, applications creating and reading files from other applications, databases, etc.

   Webassembly is limited to running a single application per instance, and only a handful programming language have good support for Webassembly (Go, Zig, C, C#, AssemblyScript and that’s about it).

9. Hubert says:[12 January 2024 4:52 AM](#27e583d39fa3b19a7ca28e2a3acfaf8b36d7f905)

   WasmGC is a great development, a solid step in the direction of intermediate-level abstract machines that could simultaneously provide a pseudo-compilation target for high-level languages, and a target for hardware ISAs in CPUs. Still it is debatable whether a stack-based machine (Java, Python, InterLISP) is best, as compared to register-based (LLVM, Parrot, Lua), or even memory-to-memory based machine (eg. Inferno’s Dis as suggested by the Register’s Liam Proven: https://forums.theregister.com/forum/all/2023/12/20/firefox *121* released/). In the end, we might find a mixed abstract machine to be best IMHO.

   This brings me to France’s fantastic plan to revive the declining rats population by getting citizens to dispose of their “compostable” biotrash in the wild, on sidewalks (as we’re not allowed to merge them with regular household trash anymore). Wouldn’t it have been more sensible to request folks to use bags of two different colors instead (and still put them in their one garbage bin)?

10. [Adam Shostack](https://shostack.org) says:[15 January 2024 4:37 PM](#e84a3c4c7629cbe9f0e5ba668d87c979c3db2b41)

    Thanks - this is an interesting article and perspective. I’m struggling to understand what the boundary is. I think it’s “you can export an API” and memory on the other side is inaccessible? Other than that, the API creator has to think through all of the security of that API and implementation except direct memory access?

Comments are closed.
