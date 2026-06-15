---
title: 'webassembly: the new kubernetes? — wingolog'
url: https://wingolog.org/archives/2021/12/13/webassembly-the-new-kubernetes
published: "2021-12-13T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2021/12/13/webassembly-the-new-kubernetes
---

# webassembly: the new kubernetes? — wingolog

## [webassembly: the new kubernetes?](/archives/2021/12/13/webassembly-the-new-kubernetes)

13 December 2021 3:50 PM

- [webassembly](/tags/webassembly)
- [kubernetes](/tags/kubernetes)
- [openstack](/tags/openstack)
- [containerization](/tags/containerization)
- [faas](/tags/faas)
- [lambda](/tags/lambda)
- [igalia](/tags/igalia)

I had an "oh, duh, of course" moment a few weeks ago that I wanted to share: is WebAssembly the next Kubernetes?

**katers gonna k8s**

[Kubernetes](https://kubernetes.io/) promises a software virtualization substrate that allows you to solve a number of problems at the same time:

- Compared to running services on bare metal, Kubernetes ("k8s") lets you use hardware more efficiently. K8s lets you run many containers on one hardware server, and lets you just add more servers to your cluster as you need them.

- The "cloud of containers" architecture efficiently divides up the work of building server-side applications. Your database team can ship database containers, your backend team ships java containers, and your product managers wire them all together using networking as the generic middle-layer. It cuts with the grain of [Conway's law](https://wingolog.org/archives/2015/11/09/embracing-conways-law): the software looks like the org chart.

- The container abstraction is generic enough to support lots of different kinds of services. Go, Java, C++, whatever -- it's not language-specific. Your dev teams can use what they like.

- The operations team responsible for the k8s servers that run containers don't have to trust the containers that they run. There is some sandboxing and security built-in.

K8s itself is an evolution on a previous architecture, [OpenStack](https://www.openstack.org). OpenStack had each container be a full virtual machine, with a whole kernel and operating system and everything. K8s instead generally uses containers, which don't generally require a kernel in the containers. The result is that they are lighter-weight -- think Docker versus VirtualBox.

In a Kubernetes deployment, you still have the kernel at a central place in your software architecture. The fundamental mechanism of containerization is the [Linux kernel process, with private namespaces](https://lwn.net/Articles/491310/). These containers are then glued together by TCP and UDP sockets. However, though one or more kernel process per container does scale better than full virtual machines, it doesn't generally scale to millions of containers. And processes do have some start-up time -- you can't spin up a container for each request to a high-performance web service. These technical contraints lead to certain kinds of system architectures, with generally long-lived components that keep some kind of state.

**k8s <=? w9y**

Server-side WebAssembly is in a similar space as Kubernetes -- or rather, WebAssembly is similar to processes plus private namespaces. WebAssembly gives you a good abstraction barrier and (can give) high security isolation. It's even better in some ways because WebAssembly provides "allowlist" security -- it has no capabilities to start with, requiring that the "host" that runs the WebAssembly explicitly delegate some of its own capabilities to the guest WebAssembly module. Compare to processes which by default start with every capability and then have to be restricted.

Like Kubernetes, WebAssembly also gets you Conway's-law-affine systems. Instead of shipping containers, you ship WebAssembly modules -- and some metadata about what kinds of things they need from their environment (the 'imports'). And WebAssembly is generic -- it's a [low level virtual machine](https://twitter.com/jckarter/status/1409896927718559744) that anything can compile to.

But, in WebAssembly you get a few more things. One is fast start. Because memory is data, you can arrange to create a [WebAssembly module that starts with its state pre-initialized in memory](https://fitzgeraldnick.com/2021/05/10/wasm-summit-2021.html). Such a module can start in microseconds -- fast enough to create one on every request, in some cases, just throwing away the state afterwards. You can run function-as-a-service architectures more effectively on WebAssembly than on containers. Another is that the virtualization is provided entirely in user-space. One process can multiplex between many different WebAssembly modules. This lets one server do more. And, you don't need to use networking to connect [WebAssembly components](https://www.infoq.com/podcasts/web-assembly-component-model/); they can transfer data in memory, sometimes even without copying.

(A digression: this lightweight in-process aspect of WebAssembly makes it so that other architectures are also possible, e.g. [this fun hack to sandbox a library linked into Firefox](https://hacks.mozilla.org/2021/12/webassembly-and-back-again-fine-grained-sandboxing-in-firefox-95/). They actually shipped that!)

I compare WebAssembly to K8s, but really it's more like processes and private namespaces. So one answer to the question as initially posed is that no, WebAssembly is not the next Kubernetes; that next thing is waiting to be built, though I know of a few organizations that have started already.

One thing does seem clear to me though: WebAssembly will be at the bottom of the new thing, and therefore that the near-term trajectory of WebAssembly is likely to follow that of Kubernetes, which means...

- Champagne time for analysts!

- The Gartner ✨✨Magic Quadrant✨✨™®© rides again

- IBM spins out a new WebAssembly division

- Accenture starts asking companies about their WebAssembly migration plan

- The Linux Foundation tastes blood in the waters

And so on. I see turbulent waters in the near-term future. So in that sense, in which Kubernetes is not essentially a technical piece of software but rather a nexus of frothy commercial jousting, then yes, certainly: we have a fun 5 years or so ahead of us.

## related articles

- [new month, new brainworm](/archives/2022/09/01/new-month-new-brainworm)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [nominal types in webassembly](/archives/2026/03/10/nominal-types-in-webassembly)
- [the last couple years in v8's garbage collector](/archives/2025/11/13/the-last-couple-years-in-v8s-garbage-collector)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)

### 2 responses

1. omkar says:[14 December 2021 7:50 AM](#a42ff4b0bc40e5745e3c9354c5f3b39381171002)

   check out https://github.com/krustlet/krustlet

2. [Liam](https://cosmonic.com) says:[15 December 2021 2:13 PM](#8d526230dda423346071c653bdd8e14668ebdc2c)

   WebAssembly in Dec of 2021 reminds me of where containers / kubernetes were in 2014 - the industry has realized that the current state of tech doesn't solve all of todays problems. Portability across edges/servers/browsers, a portable security model, and a tight coupling between business logic and libraries mean enterprises build/manage/operate multiple versions of an application. Technologies like wasmCloud are the future and in my view the next few years will be a better together story.

Comments are closed.
