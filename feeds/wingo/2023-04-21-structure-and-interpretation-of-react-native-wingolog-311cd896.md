---
title: structure and interpretation of react native — wingolog
url: https://wingolog.org/archives/2023/04/21/structure-and-interpretation-of-react-native
published: "2023-04-21T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/04/21/structure-and-interpretation-of-react-native
---

# structure and interpretation of react native — wingolog

## [structure and interpretation of react native](/archives/2023/04/21/structure-and-interpretation-of-react-native)

21 April 2023 8:20 AM

- [igalia](/tags/igalia)
- [compilers](/tags/compilers)
- [mobile](/tags/mobile)
- [ionic](/tags/ionic)
- [capacitor](/tags/capacitor)
- [react](/tags/react)
- [react native](/tags/react%20native)
- [nativescript](/tags/nativescript)
- [flutter](/tags/flutter)
- [dart](/tags/dart)
- [javascript](/tags/javascript)

Hey hey! Today’s missive continues exploring the space of JavaScript and mobile application development.

Yesterday we looked into [Ionic / Capacitor](https://wingolog.org/archives/2023/04/20/structure-and-interpretation-of-capacitor-programs), giving a brief structural overview of what Capacitor apps look like under the hood and how this translates to three aspects of performance: startup latency, jank, and peak performance. Today we’ll apply that same approach to another popular development framework, [React Native](https://reactnative.dev/).

### Background: React

I don’t know about you, but I find that there is so much marketing smoke and lights around the whole phenomenon that is React and React Native that sometimes it’s hard to see what’s actually there. This is compounded by the fact that the programming paradigm espoused by React (and its “native” cousin that we are looking at here) is so effective at enabling JavaScript UI programmers to focus on the “what” and not the “how” that the machinery supporting React recedes into the background.

At its most basic, React is what they call a *functional reactive* programming model. It is *functional* in the sense that the user interface elements render as a *function* of the global application state. The *reactive* comes into how user input is handled, but I’m not going to focus on that here.

React’s rendering process starts with a root *element tree*, describing the root node of the user interface. An *element* is a JavaScript object with a `type` property. To render an element tree, if the value of the `type` property is a string, then the element is *terminal* and doesn’t need further lowering, though React will visit any node in the `children` property of the element to render them as needed.

Otherwise if the `type` property of an element is a function, then the element node is *functional*. In that case React invokes the node’s render function (the `type` property), passing the JavaScript element object as the argument. React will then recursively re-render the element tree produced as a result of rendering the component until all nodes are terminal. (Functional element nodes can instead have a class as their `type` property, but the concerns are pretty much the same.)

(In the [language of React
Native](https://reactnative.dev/architecture/glossary), a terminal node is a *React Host Component*, and a functional node is a *React Composite* *Component*, and both are *React Elements*. There are many imprecisely-used terms in React and I will continue this tradition by using the terms I mention above.)

The rendering phase of a React application is thus a function from an element tree to a terminal element tree. Nodes of element trees can be either functional or terminal. Terminal element trees are composed only of terminal elements. Rendering lowers all functional nodes to terminal nodes. This description applies both to React (targetting the web) and React Native (which we are reviewing here).

It’s probably useful to go deeper into what React does with a terminal element tree, before building to the more complex pipeline used in React Native, so here we go. The basic idea is that React-on-the-web does impedance matching between the functional description of what the UI should have, as described by a terminal element tree, and the stateful tree of DOM nodes that a web browser uses to actually paint and display the UI. When rendering yields a new terminal element tree, React will compute the difference between the new and old trees. From that difference React then computes the set of imperative actions needed to mutate the DOM tree to correspond to what the new terminal element tree describes, and finally applies those changes.

In this way, small changes to the leaves of a React element tree should correspond to small changes in the DOM. Additionally, since rendering is a pure function of the global application state, we can avoid rendering at all when the application state hasn’t changed. We’ll dive into performance more deeply later on in the article.

### React Native doesn’t use a WebView

React Native is similar to React-on-the-web in intent but different in structure. Instead of using a WebView on native platforms, as [Ionic /
Capacitor](https://wingolog.org/archives/2023/04/20/structure-and-interpretation-of-capacitor-programs) does, React Native renders the terminal element tree to platform-native UI widgets.

When a React Native functional element renders to a terminal element, it will create not just a JS object for the terminal node as React-on-the-web does, but also a corresponding C++ [*shadow*
*object*](https://reactnative.dev/architecture/render-pipeline). The fully lowered tree of terminal elements will thus have a corresponding tree of C++ shadow objects. React Native will then calculate the layout for each node in the shadow tree, and then *commit* the shadow tree: as on the web, React Native computes the set of imperative actions needed to change the current UI so that it corresponds to what the shadow tree describes. These changes are then applied on the main thread of the application.

### The twisty path that leads one to implement JavaScript

The description above of React Native’s rendering pipeline applies to the so-called [“new
architecture”](https://reactnative.dev/architecture/overview), which has been in the works for some years and is only now (April 2023) starting to be deployed. The key development that has allowed React Native to move over to this architecture is tighter integration and control over its JavaScript implementation. Instead of using the platform’s JavaScript engine (JavaScriptCore on iOS or V8 on Android), Facebook went and made their own whole new JavaScript implementation, [Hermes](https://hermesengine.dev/). Let’s step back a bit to see if we can imagine why anyone in their right mind would make a new JS implementation.

In the last article, I mentioned that the only way to get peak JS performance on iOS is to use the platform’s WkWebView, which enables JIT compilation of JavaScript code. React Native doesn’t want a WebView, though. I guess you could create an invisible WebView and just run your JavaScript in it, but the real issue is that the interface to the JavaScript engine is so narrow as to be insufficiently expressive. You can’t cheaply synchronously create a shadow tree of layout objects, for example, because every interaction with JavaScript has to cross a process boundary.

So, it may be that JIT is just not worth paying for, if it means having to keep JavaScript at arm’s distance from other parts of the application. How do you do JavaScript without a browser on mobile, though? Either you use the platform’s JavaScript engine, or you ship your own. It would be nice to use the same engine on iOS and Android, though. When React Native was first made, V8 wasn’t able to operate in a mode that didn’t JIT, so React Native went with JavaScriptCore on both platforms.

Bundling your own JavaScript engine has the nice effect that you can easily augment it with native extensions, for example to talk to the Swift or Java app that actually runs the main UI. That’s what I describe above with the creation of the shadow tree, but that’s not quite what the original React Native did; I can only speculate but I suspect that there was a fear that JavaScript rendering work (or garbage collection!) could be heavy enough to cause the main UI to drop frames. Phones were less powerful in 2016, and JavaScript engines were less good. So the original React Native instead ran JavaScript in a separate thread. When a render would complete, the resulting terminal element tree would be serialized as JSON and shipped over to the “native” side of the application, which would actually apply the changes.

This arrangement did work, but it ran into problems whenever the system needed synchronous communication between native and JavaScript subsystems. As I understand it, this was notably the case when React layout would need the dimensions of a native UI widget; to avoid a stall, React would assume something about the dimensions of the native UI, and then asynchronously re-layout once the actual dimensions were known. This was particularly gnarly with regards to text measurements, which depend on low-level platform-specific rendering details.

To recap: React Native had to interpret its JS on iOS and was using a “foreign” JS engine on Android, so they weren’t gaining anything by using a platform JS interpreter. They would sometimes have some annoying layout jank when measuring native components. And what’s more, React Native apps would still experience the same problem as Ionic / Capacitor apps, in that application startup time was dominated by parsing and compiling the JavaScript source files.

The solution to this problem was partly to switch to the so-called “new architecture”, which doesn’t serialize and parse so much data in the course of rendering. But the other side of it was to find a way to move parsing and compiling JavaScript to the build phase, instead of having to parse and compile JS every time the app was run. On V8, you would do this by generating a [snapshot](https://v8.dev/blog/custom-startup-snapshots). On JavaScriptCore, which React Native used, there was no such facility. Faced with this problem and armed with Facebook’s bank account, the React Native developers decided that the best solution would be to make a new JavaScript implementation optimized for ahead-of-time compilation.

The result is [Hermes](https://hermesengine.dev/). If you are familiar with JavaScript engines, it is what you might expect: a JavaScript parser, originally built to match the behavior of [Esprima](https://esprima.org/); an [SSA-based intermediate
representation](https://github.com/facebook/hermes/blob/main/include/hermes/IR/IR.h); a [set of basic
optimizations](https://github.com/facebook/hermes/blob/main/include/hermes/Optimizer/PassManager/Passes.def); a [custom bytecode
format](https://github.com/facebook/hermes/blob/main/include/hermes/BCGen/HBC/BytecodeList.def); an [interpreter to run that
bytecode](https://github.com/facebook/hermes/blob/main/lib/VM/Interpreter.cpp); a [GC to manage JS objects](https://hermesengine.dev/docs/hades/); and so on. Of course, given the presence of `eval`, Hermes needs to include the parser and compiler as part of the virtual machine, but the hope is that most user code will be parsed and compiled ahead-of-time.

If this were it, I would say that Hermes seems to me to be a dead end. V8 is complete; Hermes is not. For example, Hermes doesn’t have `with`, async function implementation has been lagging, and so on. Why Hermes when you can V8 (with snapshots), now that [V8 doesn’t require JIT code
generation](https://v8.dev/blog/jitless)?

I thought about this for a while and in the end, given that V8’s main target isn’t as an embedded library in a mobile app, perhaps the binary size question is the one differentiating factor (in theory) for Hermes. By focussing on lowering distribution size, perhaps Hermes will be a compelling JS engine in its own right. In any case, Facebook can afford to keep Hermes running for a while, regardless of whether it has a competitive advantage or not.

It sounds like I’m criticising Hermes here but that’s not really the point. If you can afford it, it’s good to have code you control. For example one benefit that I see React Native getting from Hermes is that they control the [threading
model](https://reactnative.dev/architecture/threading-model); they can mostly execute JS in its own thread, but interrupt that thread and switch to synchronous main-thread execution in response to high-priority events coming from the user. You might be able to do that with V8 at some point but the mobile-apps-with-JS domain is still in flux, so it’s nice to have a sandbox that React Native developers can use to explore the system design space.

### Evaluation

With that long overview out of the way, let’s take a look to what kinds of performance we can expect out of a React Native system.

### Startup latency

Because React Native apps have their JavaScript code pre-compiled to Hermes bytecode, we can expect that the latency imposed by JavaScript during application startup is lower than is the case with Ionic / Capacitor, which needs to parse and compile the JavaScript at run-time.

However, it must be said that as a framework, [React tends to result in
large application
sizes](https://timkadlec.com/remembers/2020-04-21-the-cost-of-javascript-frameworks/#javascript-bytes) and incurs [significant work at startup
time](https://timkadlec.com/remembers/2020-04-21-the-cost-of-javascript-frameworks/#javascript-main-thread-time). One of React’s strengths is that it allows development teams inside an organization to compose well: because rendering is a pure function, it’s easy to break down the task of making an app into subtasks to be handled by separate groups of people. Could this strength lead to a kind of weakness, in that there is less of a need for overall coordination on the project management level, such that in the end nobody feels responsible for overall application performance? I don’t know. I think the concrete differences between React Native and React (the C++ shadow object tree, the multithreading design, precompilation) could mean that React Native is closer to an optimum in the design space than React. It does seem to me though that whether a platform’s primary development toolkit shold be React-like remains an open question.

### Jank

In theory React Native is well-positioned to avoid jank. JavaScript execution is mostly off the main UI thread. The [threading
model](https://reactnative.dev/architecture/threading-model) changes to allow JavaScript rendering to be pre-empted onto the main thread do make me wonder, though: what if that work takes too much time, or what if there is a GC pause during that pre-emption? I would not be surprised to see an article in the next year or two from the Hermes team about efforts to avoid GC during high-priority event processing.

Another question I would have about jank relates to interactivity. Say the user is dragging around a UI element on the screen, and the UI needs to re-layout itself. If rendering is slow, then we might expect to see a lag between UI updates and the dragging motion; the app technically isn’t dropping frames, but the render can’t complete in the 16 milliseconds needed for a 60 frames-per-second update frequency.

### Peak perf

But why might rendering be slow? On the one side, there is the fact that Hermes is not a high-performance JavaScript implementation. It uses a simple bytecode interpreter, and will never be able to meet the performance of V8 with JIT compilation.

However the other side of this is the design of the application framework. In the limit, React suffers from the O(n) problem: any change to the application state requires the whole element tree to be recomputed. Rendering and layout work is proportional to the size of the application, which may have thousands of nodes.

Of course, React tries to minimize this work, by detecting subtrees whose layout does not change, by avoiding re-renders when state doesn’t change, by minimizing the set of mutations to the native widget tree. But the native widgets aren’t the problem: the programming model is, or it can be anyway.

### Aside: As good as native?

Again in theory, React Native can used to write apps that are as good as if they were written directly against platform-native APIs in Kotlin or Swift, because it uses the same platform UI toolkits as native applications. React Native can also do this at the same time as being cross-platform, targetting iOS and Android with the same code. In practice, besides the challenge of designing suitable cross-platform abstractions, React Native has to grapple with potential performance and memory use overheads of JavaScript, but the result has the potential to be quite satisfactory.

### Aside: Haven’t I seen that rendering model somewhere?

As I mentioned in the last article, I am a compiler engineer, not a UI specialist. In the course of my work I do interact with a number of colleagues working on graphics and user interfaces, notably in the context of browser engines. I was struck when reading about React Native’s rendering pipeline about how much it resembled what [a browser
itself will
do](https://developer.chrome.com/articles/renderingng-data-structures/#the-immutable-fragment-tree) as part of the layout, paint, and render pipeline: translate a tree of objects to a tree of immutable layout objects, clip those to the viewport, paint the ones that are dirty, and composite the resulting textures to the screen.

It’s funny to think about how many levels we have here: the element tree, the recursively expanded terminal element tree, the shadow object tree, the platform-native widget tree, surely a corresponding platform-native layout tree, and then the GPU backing buffers that are eventually composited together for the user to see. Could we do better? I could certainly imagine any of these mobile application development frameworks switching to their own Metal/Vulkan-based rendering architecture at some point, to flatten out these layers.

### Summary

By all accounts, React Native is a real delight to program for; it makes developers happy. The challenge is to make it perform well for users. With its new rendering architecture based on Hermes, React Native may well be on the path to addressing many of these problems. Bytecode pre-compilation should go a long way towards solving startup latency, provided that React’s expands-to-fit-all-available-space tendency is kept in check.

If you were designing a new mobile operating system from the ground up, though, I am not sure that you would necessarily end up with React Native as it is. At the very least, you would include Hermes and the base run-time as part of your standard library, so that every app doesn’t have to incur the space costs of shipping the run-time. Also, in the same way that Android [can ahead-of-time and just-in-time compile
its
bytecode](https://source.android.com/docs/core/runtime/jit-compiler), I would expect that a mobile operating system based on React Native would extend its compiler with on-device post-install compilation and possibly JIT compilation as well. And at that point, why not switch back to V8?

Well, that’s food for thought. Next up, [NativeScript](https://wingolog.org/archives/2023/04/24/structure-and-interpretation-of-nativescript). Until then, happy hacking!

## related articles

- [parallel futures in mobile application development](/archives/2023/06/15/parallel-futures-in-mobile-application-development)
- [structure and interpretation of ark](/archives/2023/05/02/structure-and-interpretation-of-ark)
- [structure and interpretation of flutter](/archives/2023/04/26/structure-and-interpretation-of-flutter)
- [structure and interpretation of nativescript](/archives/2023/04/24/structure-and-interpretation-of-nativescript)
- [structure and interpretation of capacitor programs](/archives/2023/04/20/structure-and-interpretation-of-capacitor-programs)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)

Comments are closed.
