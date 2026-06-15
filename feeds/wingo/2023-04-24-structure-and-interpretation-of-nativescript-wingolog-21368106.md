---
title: structure and interpretation of nativescript — wingolog
url: https://wingolog.org/archives/2023/04/24/structure-and-interpretation-of-nativescript
published: "2023-04-24T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/04/24/structure-and-interpretation-of-nativescript
---

# structure and interpretation of nativescript — wingolog

## [structure and interpretation of nativescript](/archives/2023/04/24/structure-and-interpretation-of-nativescript)

24 April 2023 9:10 AM

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

Greetings, hackers tall and hackers small!

We’re only a few articles in to this series on mobile application development frameworks, but I feel like we are already well into our journey. We started our trip through the design space with a look at [Ionic /
Capacitor](https://wingolog.org/archives/2023/04/20/structure-and-interpretation-of-capacitor-programs), which defines its user interface in terms of the web platform, and only calls out to iOS or Android native features as needed. We proceeded on to [React
Native](https://wingolog.org/archives/2023/04/21/structure-and-interpretation-of-react-native), which moves closer to native by rendering to platform-provided UI widgets, layering a cross-platform development interface on top.

Today’s article takes an in-depth look at [NativeScript](https://nativescript.org/), whose point in the design space is further on the road towards the platform, unabashedly embracing the specificities of the API available on iOS and Android, exposing these interfaces directly to the application programmer.

In practice what this looks like is that a NativeScript app is a native app which simply happens to call JavaScript on the main UI thread. That JavaScript has access to *all* native APIs, directly, without the mediation of serialization or message-passing over a bridge or message queue.

The first time I heard this I thought that it couldn’t actually be *all* native APIs. After all, new versions of iOS and Android come out quite frequently, and surely it would take some effort on the part of NativeScript developers to expose the new APIs to JavaScript. But no, it really includes all of the various native APIs: the NativeScript developers wrote a build-time inspector that uses the platform’s native reflection capabilities to grovel through all available APIs and to automatically generate JavaScript bindings, with associated TypeScript type definitions so that the developer knows what is available.

Some of these generated files are checked into source, so you can get an idea of the range of interfaces that are accessible to programmers; for example, see [the iOS type definitions for
x86-64](https://github.com/NativeScript/NativeScript/tree/main/packages/types-ios/src/lib/ios/objc-x86_64). There are bindings for, like, everything.

Given access to all the native APIs, how do you go about making an app? You could write the same kind of programs that you would in Swift or Kotlin, but in JavaScript. But this would require more than just the *ability* to access native capabilities when needed: it needs a thorough knowledge of the platform interfaces, plus NativeScript itself on top. Most people don’t have this knowledge, and those that do are probably programming directly in Swift or Kotlin already.

On one level, NativeScript’s approach is to take refuge in that most ecumenical of adjectives, “unopinionated”. Whereas Ionic / Capacitor encourages use of web platform interfaces, and React Native only really supports React as a programming paradigm, NativeScript provides a low-level platform onto which you can layer a number of different high-level frameworks.

Now, most high-level JavaScript application development frameworks are oriented to targetting the web: they take descriptions of user interfaces and translate them to the DOM. When targetting NativeScript, you could make it so that they target native UI widgets instead. However given the baked-in assumptions of how widgets should be laid out (notably via CSS), there is some impedance-matching to do between DOM-like APIs and native toolkits.

NativeScript’s answer to this problem is a middle layer: a [cross-platform UI
library](https://github.com/NativeScript/NativeScript/tree/main/packages/core/ui) that provides DOM-like abstractions and CSS layout in a way that bridges the gap between web-like and native. You can even define parts of the UI using a [NativeScript-specific XML
vocabulary](https://github.com/NativeScript/NativeScript/blob/main/apps/automated/src/xml-declaration/tns.xsd), which NativeScript compiles to native UI widget calls [at
run-time](https://github.com/NativeScript/NativeScript/tree/main/packages/core/ui/builder). Of course, there is no CSS engine in UIKit or Android’s UI toolkit, so NativeScript includes its own, [implemented in JavaScript of
course](https://github.com/NativeScript/NativeScript/blob/main/packages/core/ui/styling/css-selector.ts).

You could program directly to this middle layer, but I suspect that its real purpose is in enabling Angular, Vue, Svelte, or the like. The pitch would be that NativeScript lets app developers use pleasant high-level abstractions, but while remaining close to the native APIs; you can always drop down for more power and expressiveness if needed.

Diving back down to the low level, as we mentioned all of the interactions between JavaScript and the native platform APIs happen on the main application UI thread. NativeScript does also allow programmers to create background threads, using an [implementation of
the Web Worker
API](https://docs.nativescript.org/advanced-concepts.html#multithreading-model). One could even in theory run a React-based UI in a worker thread and proxy native UI updates to the main thread; as an unopinionated platform, NativeScript can support many different frameworks and paradigms.

Finally, there is the question of how NativeScript runs the JavaScript in an application. Recall that Ionic / Capacitor uses the native JS engine, by virtue of using the native WebView, and that React Native used to use JavaScriptCore on both platforms but now uses its own Hermes implementation. NativeScript is another point in the design space, using V8 on both platforms. (They used to use JavaScriptCore on iOS but [switched to
V8](https://blog.nativescript.org/the-new-ios-runtime-powered-by-v8/) once V8 was able to run on iOS in “jitless” mode.) Besides the reduced maintenance burden of using a single implementation on all platforms, this also has the advantage of being able to use [V8
snapshots](https://v8.dev/blog/custom-startup-snapshots) to move JavaScript parse-and-compile work to build-time, even on iOS.

### Evaluation

NativeScript is fundamentally simple: it’s V8 running in an application’s main UI thread, with access to all platform native APIs. So how do we expect it to perform?

### Startup latency

In theory, applications with a NativeScript-like architecture should have no problem with startup time, because they can pre-compile all of their JavaScript into V8 snapshots. Snapshots are cheap to load up because they are already in a format that V8 is ready to consume.

In practice, it would seem that [V8 snapshots do not perform as expected
for
NativeScript](https://github.com/NativeScript/NativeScript/issues/8926). There are a number of aspects about this situation that I don’t understand, which I suspect relate to the state of the tooling around V8 rather than to the fundamental approach of ahead-of-time compilation. V8 is really made for Chrome, and it could be that not enough maintenance resources have been devoted to this snapshot facility.

In the meantime, NativeScript instead uses V8’s [code cache
feature](https://v8.dev/blog/code-caching), which caches the result of parsing and compiling JavaScript files on the device. In this way the first time an app is installed or updated, it might start up slowly, but subsequent runs are faster. If you were designing a new operating system, you’d probably want to move this work to app install-time.

As we mentioned above, NativeScript apps have access to all native APIs. That is a lot of APIs, and only some of those interfaces will actually be used by any given app. In an ideal world, we would expect the build process to only include JavaScript code for those APIs that are needed by the application. However in the presence of `eval` and dynamic property lookup, pruning the native API surface to the precise minimum is a hard problem for a bundler to perform on its own. The solution for the time being is to [manually allow and deny subsets of the platform
native
API](https://docs.nativescript.org/advanced-concepts.html#metadata-filtering). It’s not an automatic process though, so it can be error-prone.

Besides the work that the JavaScript engine has to do to load an application’s code, the other startup overhead involves whatever work that JavaScript might need to perform before the first frame is shown. In the case of NativeScript, more work is done before the initial layout than one would think: the main UI XML file is parsed by an XML parser written in JavaScript, any needed CSS files are parsed and loaded (again by JavaScript), and the tree of XML elements is translated to a [tree of
UI
elements](https://docs.nativescript.org/advanced-concepts.html#the-layout-process). The layout of the items in the view tree is then computed (in JavaScript, but calling into native code to measure text and so on), and then the app is ready.

At this point, I am again going to wave my “I am just a compiler engineer” flag: I am not a UI specialist, much less a NativeScript specialist. As in compilers, performance measurement and monitoring are key to UI development, but I suspect that also as in compilers there is a role for gut instinct. Incremental improvements are best driven by metrics, but qualitative leaps are often the result of somewhat ineffable hunches or even guesswork. In that spirit I can only surmise that React Native has an advantage over NativeScript in time-to-first-frame, because its layout is performed in C++ and because its element tree is computed directly from JavaScript instead of having JavaScript interpret XML and CSS files. In any case, I look forward to the forthcoming part 2 of the [NativeScript and React Native performance
investigations](https://blog.nativescript.org/perf-metrics-universal-javascript-part1/) that were started in November 2022.

If I were NativeScript and using NativeScript’s UI framework, and if startup latency proves to actually be a problem, I would lean into something in the shape of [Angular’s ahead-of-time compilation
mode](https://angular.io/guide/aot-compiler), but for the middle NativeScript UI layer.

### Jank

On the face of it, NativeScript is the most jank-prone of the three frameworks we have examined, because it runs JavaScript on the main application UI thread, interleaved with UI event handling and painting and all of that. If an app’s JavaScript takes too long to run, the app might miss frames or fail to promptly handle an event.

On the other hand, relative to React Native, the user’s code is much closer to the application’s behavior. There’s no asynchrony between the application’s logic and its main loop: in NativeScript it is easy to identify the code causing jank and eventually fix it.

The other classic JavaScript-on-the-main-thread worry relates to garbage collection pauses. V8’s garbage collector does try to minimize the stop-the-world phase by [tracing the heap concurrently and leveraging
parallelism during pauses](https://v8.dev/blog/trash-talk). Also, the user interface of a mobile app runs in an event loop, and typically spends most of its time idle; V8 exposes some API that can take advantage of this idle time to perform housekeeping tasks instead of needing to do them when handling high-priority events.

That said, having looked into the code of both the iOS and Android run-times, NativeScript does not currently take advantage of this facility. I dug deeper and it would seem that V8 itself is in flux, as the [IdleNotificationDeadline
API](https://chromium.googlesource.com/v8/v8/+/refs/heads/main/include/v8-isolate.h#1281) is on its way out; is the thought that concurrent tracing is largely sufficient? I would expect that if [conservative stack
scanning](https://groups.google.com/g/v8-reviews/c/5ptLig88soA?pli=1) lands, we will see a re-introduction of this kind of API, as it does make sense to synchronize with the event loop when scanning the main thread stack.

### Peak performance

As we have seen in our previous evaluations, this question boils down to “is the JavaScript engine state-of-the-art, and can it perform just-in-time compilation”. In the case of NativeScript, the answers are yes and maybe, respectively: V8 is state-of-the-art, and it can JIT on Android, but not on iOS.

Perhaps the mitigation here is that the hardware that iOS runs on tends to be significantly more powerful than median Android devices; if you had to pick a subset of users to penalize with an interpreter-only run-time, people with iPhones are the obvious choice, because they can afford it.

### Aside: Are markets wise?

Recall that our perspective in this series is that of the designer of a new JavaScript-based mobile development platform. We are trying to answer the question of what would it look like if a new platform offered a NativeScript-like experience. In this regard, only the structure of NativeScript is of interest, and notably its “market success” is not relevant, except perhaps in some Hayekian conception of the world in which markets are necessarily smarter than, well, me, or you, or any one of us.

It must be said, though, that React Native is the 800-pound gorilla of JavaScript mobile application development. The [2022 State of JS
survey](https://2022.stateofjs.com/en-US/libraries/mobile-desktop/) shows that among survey respondents, more people are aware of React Native than any other mobile framework, and people are generally more positive about React Native than other frameworks. Does NativeScript’s mitigated market share indicate something about its architecture, or does it speak speak more to the size of Facebook’s budget, both on the developer experience side and on marketing?

### Aside: On the expressive power of application framworks

Oddly, I think the answer to the market wisdom question might be found in a 35-year-old computer science paper, [“On the expressive power of
programming
languages”](https://dl.acm.org/doi/10.1016/0167-6423%2891%2990036-W) ( [PDF](https://www.sciencedirect.com/science/article/pii/016764239190036W/pdf?md5=b7dedd960214d9191929e6f41f5fd5be&pid=1-s2.0-016764239190036W-main.pdf&_valck=1)).

In this paper, Matthias Felleisen considers the notion of what it means for one programming language to be more expressive than another. For example, is a language with just `for` less expressive than a language with both `for` and `while`? Intuitively we would say no, these are similar things; you can make a simple local transformation of `while (x) {...}` to `for (;x;) {...}` and you have exactly the same program semantics. On the other hand a language with just `for` is less expressive than one which also has `goto`; there is no simple local rewrite that can turn `goto` into `for`.

In the same way, we can consider the question of what it would mean for one library to be more expressive than another. After all, the API of a library exposes a language in which its user can write programs; we should be able to reason about these languages. So between React Native and NativeScript, which one is more expressive?

By Felleisen’s definitions, NativeScript is clearly the more expressive language: there is no simple local transformation that can turn imperative operations on native UI widgets into equivalent functional-reactive programs. Yes, with enough glue code React Native can reach directly to native APIs in a similar way as NativeScript, but everything that touches the native UI tree is expressly under React Native’s control: there is no sanctioned escape hatch.

You might think that “more expressive” is always better, but Felleisen’s take is more nuanced than that. Yes, he says, more expressive languages do allow programmers to make more concise programs, because they allow programmers to define abstractions that encapsulate patterns, and this is a good thing. However he also concludes that “an increase in expressive power is related to a decrease of the set of ‘natural’ (mathematically appealing) operational equivalences.” Less expressive programming languages are easier to reason about, in general, and indeed that is one of the recognized strengths of React’s programming model: it is easy to compose components and have confidence that the result will work.

### Summary

A NativeScript-like architecture offers the possibility of performance: the developer has all the capabilities needed for writing pleasant-to-use applications that blend in with the platform-native experience. It is up to the developers to choose how to use the power at their disposal. In the wild, I expect that the low-level layer of NativeScript’s API is used mainly by expert developers, who know how to assemble well-functioning machines from the parts on offer.

As a primary programming interface for a new JavaScript-based mobile platform, though, just providing a low-level API would seem to be not enough. NativeScript rightly promotes the use of more well-known high-level frameworks on top: Angular, Vue, Svelte, and such. Less experienced developers should use an opinionated high-level UI framework; these developers don’t have good opinions yet and the API should lead them in the right direction.

That’s it for today. Thanks for reading these articles, by the way; I have enjoyed diving into this space. Next up, we’ll take a look beyond JavaScript, to [Flutter and Dart](https://wingolog.org/archives/2023/04/26/structure-and-interpretation-of-flutter). Until then, happy hacking!

## related articles

- [parallel futures in mobile application development](/archives/2023/06/15/parallel-futures-in-mobile-application-development)
- [structure and interpretation of ark](/archives/2023/05/02/structure-and-interpretation-of-ark)
- [structure and interpretation of flutter](/archives/2023/04/26/structure-and-interpretation-of-flutter)
- [structure and interpretation of react native](/archives/2023/04/21/structure-and-interpretation-of-react-native)
- [structure and interpretation of capacitor programs](/archives/2023/04/20/structure-and-interpretation-of-capacitor-programs)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)

Comments are closed.
