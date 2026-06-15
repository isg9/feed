---
title: pictie, my c++-to-webassembly workbench — wingolog
url: https://wingolog.org/archives/2019/06/03/pictie-my-c-to-webassembly-workbench
published: "2019-06-03T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2019/06/03/pictie-my-c-to-webassembly-workbench
---

# pictie, my c++-to-webassembly workbench — wingolog

## [pictie, my c++-to-webassembly workbench](/archives/2019/06/03/pictie-my-c-to-webassembly-workbench)

3 June 2019 10:10 AM

- [igalia](/tags/igalia)
- [c++](/tags/c%2B%2B)
- [webassembly](/tags/webassembly)
- [emscripten](/tags/emscripten)
- [finalizers](/tags/finalizers)
- [weak-refs](/tags/weak-refs)
- [javascript](/tags/javascript)
- [webidl](/tags/webidl)
- [embind](/tags/embind)
- [sicp](/tags/sicp)

Hello, interwebs! Today I'd like to share a little skunkworks project with y'all: [Pictie](https://github.com/wingo/pictie), a workbench for WebAssembly C++ integration on the web.

**loading pictie...**



**wtf just happened????!?**

So! If everything went well, above you have some colors and a prompt that accepts Javascript expressions to evaluate. If the result of evaluating a JS expression is a painter, we paint it onto a canvas.

But allow me to back up a bit. These days everyone is talking about WebAssembly, and I think with good reason: just as many of the world's programs run on JavaScript today, tomorrow much of it will also be in languages compiled to WebAssembly. JavaScript isn't going anywhere, of course; it's around for the long term. It's the "also" aspect of WebAssembly that's interesting, that it appears to be a computing substrate that is compatible with JS and which can extend the range of the kinds of programs that can be written for the web.

And yet, it's early days. What are programs of the future going to look like? What elements of the web platform will be needed when we have systems composed of WebAssembly components combined with JavaScript components, combined with the browser? Is it all going to work? Are there missing pieces? What's the status of the toolchain? What's the developer experience? What's the user experience?

When you look at the current set of applications targetting WebAssembly in the browser, mostly it's games. While compelling, games don't provide a whole lot of insight into the shape of the future web platform, inasmuch as there doesn't have to be much JavaScript interaction when you have an already-working C++ game compiled to WebAssembly. (Indeed, much of the incidental interactions with JS that are currently necessary -- bouncing through JS in order to call WebGL -- people are actively working on removing all of that overhead, so that WebAssembly can call platform facilities (WebGL, etc) directly. But I digress!)

For WebAssembly to really succeed in the browser, there should also be incremental stories -- what does it look like when you start to add WebAssembly modules to a system that is currently written mostly in JavaScript?

To find out the answers to these questions and to evaluate potential platform modifications, I needed a small, standalone test case. So... I wrote one? It seemed like a good idea at the time.

**pictie is a test bed**

Pictie is a simple, standalone C++ graphics package implementing an algebra of painters. It was created not to be a great graphics package but rather to be a test-bed for compiling C++ libraries to WebAssembly. You can read more about it on [its github page](https://github.com/wingo/pictie).

Structurally, pictie is a modern C++ library with a functional-style interface, smart pointers, reference types, lambdas, and all the rest. We use emscripten to compile it to WebAssembly; you can see more information on how that's done in the repository, or check the [README](https://github.com/wingo/pictie/blob/master/README.md#webassembly).

Pictie is inspired by Peter Henderson's "Functional Geometry" ( [1982](http://pmh-systems.co.uk/phAcademic/papers/funcgeo.pdf), [2002](https://eprints.soton.ac.uk/257577/1/funcgeo2.pdf)). "Functional Geometry" inspired the [Picture language](https://sarabander.github.io/sicp/html/2_002e2.xhtml#g_t2_002e2_002e4) from the well-known *Structure and Interpretation of Computer Programs* computer science textbook.

**prototype in action**

So far it's been surprising how much stuff just works. There's still lots to do, but just getting a C++ library on the web is pretty easy! I advise you to take a look to see the details.

If you are thinking of dipping your toe into the WebAssembly water, maybe take a look also at Pictie when you're doing your back-of-the-envelope calculations. You can use it or a prototype like it to determine the effects of different compilation options on compile time, load time, throughput, and network trafic. You can check if the different binding strategies are appropriate for your C++ idioms; Pictie currently uses [embind](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html) [(source)](https://github.com/wingo/pictie/blob/master/pictie.bindings.cc), but [I would like to compare to WebIDL as well](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html). You might also use it if you're considering what shape your C++ library should have to have a minimal overhead in a WebAssembly context.

I use Pictie as a test-bed when working on the web platform; the [weakref proposal](https://github.com/tc39/proposal-weakrefs) which adds finalization, [leak detection](https://github.com/emscripten-core/emscripten/pull/8687), and working on the binding layers around Emscripten. Eventually I'll be able to use it in other contexts as well, with the [WebIDL bindings](https://github.com/WebAssembly/webidl-bindings/blob/master/proposals/webidl-bindings/Explainer.md) proposal, typed objects, and [GC](https://github.com/WebAssembly/webidl-bindings/blob/master/proposals/webidl-bindings/Explainer.md).

**prototype the web forward**

As the browser and adjacent environments have come to dominate programming in practice, we lost a bit of the delightful variety from computing. JS is a great language, but it shouldn't be the only medium for programs. WebAssembly is part of this future world, waiting in potentia, where applications for the web can be written in any of a number of languages. But, this future world will only arrive if it "works" -- if all of the various pieces, from standards to browsers to toolchains to virtual machines, only if all of these pieces fit together in some kind of sensible way. Now is the early phase of annealing, when the platform as a whole is actively searching for its new low-entropy state. We're going to need a lot of prototypes to get from here to there. In that spirit, may your prototypes be numerous and soon replaced. Happy annealing!

## related articles

- [the last couple years in v8's garbage collector](/archives/2025/11/13/the-last-couple-years-in-v8s-garbage-collector)
- [javascript weakmaps should be iterable](/archives/2024/08/19/javascript-weakmaps-should-be-iterable)
- [parallel futures in mobile application development](/archives/2023/06/15/parallel-futures-in-mobile-application-development)
- [accessing webassembly reference-typed arrays from c++](/archives/2022/08/23/accessing-webassembly-reference-typed-arrays-from-c)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [on "binary security of webassembly"](/archives/2020/10/15/on-binary-security-of-webassembly)

### One response

1. [civodul](http://web.fdn.fr/~lcourtes/) says:[13 June 2019 9:01 PM](#ba719c09a6fae093ec6a679f6066d03f94bd4575)

   Regarding bindings to JS, the original Hop (https://hop.inria.fr) had a rather good story for interfacing Scheme code with JS in the browser. You could interact with JS objects and Scheme objects in the same way, seamlessly; the "Code Reuse" section at https://queue.acm.org/detail.cfm?id=2330089 explains the approach. Of course interfacing C++ with JS is inherently more difficult than interfacing two dynamic languages.

Comments are closed.
