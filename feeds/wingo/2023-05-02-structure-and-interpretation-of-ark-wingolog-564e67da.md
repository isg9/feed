---
title: structure and interpretation of ark — wingolog
url: https://wingolog.org/archives/2023/05/02/structure-and-interpretation-of-ark
published: "2023-05-02T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/05/02/structure-and-interpretation-of-ark
---

# structure and interpretation of ark — wingolog

## [structure and interpretation of ark](/archives/2023/05/02/structure-and-interpretation-of-ark)

2 May 2023 9:13 AM

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

Hello, dear readers! Today’s article describes Ark, a new JavaScript-based mobile development platform. If you haven’t read them yet, you might want to start by having a look at my past articles on [Capacitor](https://wingolog.org/archives/2023/04/20/structure-and-interpretation-of-capacitor-programs), [React
Native](https://wingolog.org/archives/2023/04/21/structure-and-interpretation-of-react-native), [NativeScript](https://wingolog.org/archives/2023/04/24/structure-and-interpretation-of-nativescript), and [Flutter](https://wingolog.org/archives/2023/04/26/structure-and-interpretation-of-flutter); having a common understanding of the design space will help us understand where Ark is similar and where it differs.

### Ark, what it is

If I had to bet, I would guess that you have not heard of Ark. (I certainly hadn’t either, when commissioned to do this research series.) To a first approximation, Ark—or rather, what I am calling Ark; I don’t actually know the name for the whole architecture—is a loosely Flutter-like UI library implemented on top of a dialect of JavaScript, with build-time compilation to bytecode (like Hermes) but also with support for just-in-time and ahead-of-time compilation of bytecode to native code. It is made by Huawei.

At this point if you are already interested in this research series, I am sure this description raises more questions than it answers. Flutter-like? A dialect? Native compilation? Targetting what platforms? *From Huawei?* We’ll get to all of these, but I think we need to start with the last question.

### How did we get here?

In my last article on Flutter, I told a kind of just-so history of how Dart and Flutter came to their point in the design space. Thanks to corrections from a kind reader, it happened to also be more or less correct. In this article, though, I haven’t talked with Ark developers at all; I don’t have the benefit of a true claim on history. And yet, the only way I can understand Ark is by inventing a narrative, so here we go. It might even be true!

Recall that in 2018, Huawei was a dominant presence in the smartphone market. They were shipping excellent hardware at good prices both to the Chinese and to the global markets. Like most non-Apple, non-Google manufacturers, they shipped Android, and like most Android OEMs, they shipped [Google’s proprietary apps (mail, maps,
etc.)](https://en.wikipedia.org/wiki/Google_Mobile_Services).

But then, over the next couple years, the US decided that allowing Huawei to continue on as before was, like, against national security interests or something. Huawei was barred from American markets, a number of suppliers were forbidden from selling hardware components to Huawei, and even Google was prohibited from shipping its mobile apps on Huawei devices. The effect on Huawei’s market share for mobile devices was enormous: its revenue was cut in half over a period of a couple years.

In this position, as Huawei, what do you do? I can’t even imagine, but specifically looking at smartphones, I think I would probably do about what they did. I’d fork Android, for starters, because that’s what you already know and ship, and Android is mostly open source. I’d probably plan on continuing to use its lower-level operating system pieces indefinitely (kernel and so on) because that’s not a value differentiator. I’d probably ship the same apps on top at first, because otherwise you slip all the release schedules and lose revenue entirely.

But, gosh, there is the risk that your product will be perceived as just a worse version of Android: that’s not a good position to be in. You need to be different, and ideally better. That will take time. In the meantime, you *claim* that you’re different, without actually being different yet. It’s a somewhat ridiculous position to be in, but I can understand how you get here; [Ars Technica published a scathing
review](https://arstechnica.com/gadgets/2021/02/harmonyos-hands-on-huaweis-android-killer-is-just-android/) poking fun at the situation. But, you are big enough to ride it out, knowing that somehow eventually you will be different.

Up to now, this part of the story is relatively well-known; the part that follows is more speculative on my part. Firstly, I would note that Huawei had been working for a while on a compiler and language run-time called [Ark
Compiler](https://www.huaweicentral.com/ark-compiler-huaweis-self-developed-android-application-compiler-explained/), with the goal of getting better performance out of Android applications. If I understand correctly, this compiler took the Java / Dalvik / Android Run Time bytecodes as its input, and outputted native binaries along with a new run-time implementation.

As I can attest from personal experience, having a compiler leads to hubris: you start to consider source languages like a hungry person looks at a restaurant menu. “Wouldn’t it be nice to ingest that?” That’s what we say at restaurants, right, fellow humans? So in 2019 and 2020 when the Android rug was pulled out from underneath Huawei, I think having in-house compiler expertise allowed them to consider whether they wanted to stick with Java at all, or whether it might be better to choose a more fashionable language.

Like black, JavaScript is always in fashion. What would it mean, then, to retool Huawei’s operating system – by then known by the name “HarmonyOS” – to expose a JavaScript-based API as its primary app development framework? You could use your Ark compiler somehow to implement JavaScript (hubris!) and then you need a UI framework. Having ditched Java, it is now thinkable to ditch all the other Android standard libraries, including the UI toolkit: you start anew, in a way. So are you going to build a Capacitor, a React Native, a NativeScript, a Flutter? Surely not precisely any of these, but what will it be like, and how will it differ?

Incidentally, I don’t know the origin story for the name Ark, but to me it brings to mind tragedy and rebuilding: in the midst of being cut off from your rich Android ecosystem, you launch a boat into the sea, holding a promise of a new future built differently. Hope and hubris in one vessel.

### Two programming interfaces

In the end, Huawei builds two things: something web-like and something like Flutter. (I don’t mean to suggest copying or degeneracy here; it’s rather that I can only understand things in relation to other things, and these are my closest points of comparison for what they built.)

The web-like programming interface specifies UIs using an XML dialect, [HML](https://developer.harmonyos.com/en/docs/documentation/doc-guides-V3/js-framework-syntax-hml-0000001477981005-V3), and styles the resulting node tree with CSS. You augment these nodes with JavaScript behavior; the main app is a set of [DOM-like event
handlers](https://developer.harmonyos.com/en/docs/documentation/doc-guides-V3/js-framework-syntax-js-0000001428061552-V3). There is an API to [dynamically create DOM
nodes](https://developer.harmonyos.com/en/docs/documentation/doc-references-V3/js-components-create-elements-0000001478181509-V3?catalogVersion=V3), but unlike the other systems we have examined, the HarmonyOS documentation doesn’t really sell you on using a high-level framework like Angular.

If this were it, I think Ark would not be so compelling: the programming model is more like what was available back in the [DHTML
days](https://en.wikipedia.org/wiki/Dynamic_HTML). I wouldn’t expect people to be able to make rich applications that delight users, given these primitives, though CSS animation and the HML [loop and conditional
rendering](https://developer.harmonyos.com/en/docs/documentation/doc-references-V3/js-components-container-list-0000001427584924-V3#EN-US_TOPIC_0000001544695117__example) from the template system might be just expressive enough for simple applications.

The more interesting side is the so-called “declarative” UI programming model which exposes a Flutter/React-like interface. The programmer describes the “what” of the UI by providing a tree of UI nodes in its `build` function, and the framework takes care of calling `build` when necessary and of rendering that tree to the screen.

Here I need to show some example code, because it is... weird. Well, I find it weird, but it’s not too far from [SwiftUI](https://developer.apple.com/xcode/swiftui/) in flavor. A small example from [the fine
manual](https://developer.harmonyos.com/en/docs/documentation/doc-guides-V3/ui-ts-creating-simple-page-0000001493575800-V3):

```
@Entry
@Component
struct MyComponent {
  build() {
    Stack() {
        Image($rawfile('Tomato.png'))
        Text('Tomato')
            .fontSize(26)
            .fontWeight(500)
    }
  }
}

```

The `@Entry` decorator (\*) marks this `struct` (\*\*) as being the main entry point for the app. `@Component` marks it as being a component, like a React *functional component*. Components conform to an interface (\*\*\*) which defines them as having a `build` method which takes no arguments and returns no values: it creates the tree in a somewhat imperative way.

But as you see the flavor is somewhat declarative, so how does that work? Also, `build() { ... }` looks syntactically a lot like `Stack() { ... }`; what’s the deal, are they the same?

Before going on to answer this, note my asterisks above: these are concepts that aren’t in JavaScript. Indeed, programs written for HarmonyOS’s declarative framework aren’t JavaScript; they are in a dialect of TypeScript that Huawei calls ArkTS. In this case, an `interface` is [a TypeScript
concept](https://www.typescriptlang.org/docs/handbook/interfaces.html). Decorators would appear to correspond to [an experimental TypeScript
feature](https://www.typescriptlang.org/docs/handbook/decorators.html), looking at the source code.

But `struct` is an [ArkTS-specific
extension](https://gitee.com/openharmony/third_party_typescript#changes), and Huawei has actually extended the TypeScript compiler to specifically recognize the `@Component` decorator, such that when you “call” a struct, for example as above in `Stack() { ... }`, TypeScript will parse that as a new expression type [`EtsComponentExpression`](https://gitee.com/openharmony/third_party_typescript/blob/master/src/compiler/types.ts#L2065), which may optionally be followed by a block. When `Stack()` is invoked, its children (instances of `Image` and `Text`, in this case) will be populated via running the block.

Now, though TypeScript isn’t everyone’s bag, it’s quite normalized in the JavaScript community and not a hard sell. Language extensions like the handling of `@Component` pose a more challenging problem. Still, Facebook managed to sell people on [JSX](https://legacy.reactjs.org/docs/introducing-jsx.html), so perhaps Huawei can do the same for their dialect. More on that later.

Under the hood, it would seem that we have a similar architecture to Flutter: invoking the [components](https://gitee.com/openharmony/arkui_ace_engine/blob/master/frameworks/core/pipeline/base/component.h) creates a corresponding tree of [*elements*](https://gitee.com/openharmony/arkui_ace_engine/blob/master/frameworks/core/pipeline/base/element.h) (as with React Native’s shadow tree), which then are lowered to [*render nodes*](https://gitee.com/openharmony/arkui_ace_engine/blob/master/frameworks/core/pipeline/base/render_node.h), which draw themselves onto layers using Skia, in a multi-threaded [rendering pipeline](https://gitee.com/openharmony/arkui_ace_engine/blob/master/frameworks/core/pipeline/pipeline_context.h). Underneath, the UI code actually re-uses some parts of Flutter, though from what I can tell HarmonyOS developers are replacing those over time.

### Restrictions and extensions

So we see that the source language for the declarative UI framework is TypeScript, but with some extensions. It also has its restrictions, and to explain these, we have to talk about implementation.

Of the JavaScript mobile application development frameworks we discussed, Capacitor and NativeScript used “normal” JS engines from web browsers, while React Native built their own Hermes implementation. Hermes is also restricted, in a way, but mostly inasmuch as it lags the browser JS implementations; it relies on source-to-source transpilers to get access to new language features. ArkTS—that’s the name of HarmonyOS’s “extended TypeScript” implementation—has more fundamental restrictions.

Recall that the Ark compiler was originally built for Android apps. There you don’t really have the ability to load new Java or Kotlin source code at run-time; in Java you have class loaders, but those load bytecode. On an Android device, you don’t have to deal with the Java source language. If we use a similar architecture for JavaScript, though, what do we do about `eval`?

ArkTS’s answer is: don’t. As in, `eval` is not supported on HarmonyOS. In this way the implementation of ArkTS can be divided into two parts, a frontend that produces bytecode and a runtime that runs the bytecode, and you never have to deal with the source language on the device where the runtime is running. Like Hermes, the developer produces bytecode when building the application and ships it to the device for the runtime to handle.

Incidentally, before we move on to discuss the runtime, there are actually two front-ends that generate ArkTS bytecode: [one written in
C++ that seems to only handle standard TypeScript and
JavaScript](https://gitee.com/openharmony/arkcompiler_ets_frontend/tree/master/es2panda), and [one written in TypeScript that also handles “extended
TypeScript”](https://gitee.com/openharmony/arkcompiler_ets_frontend/tree/master/ts2panda). The former has a [test262 runner with about 10k skipped
tests](https://gitee.com/openharmony/arkcompiler_ets_frontend/blob/master/es2panda/test/test262skiplist.txt), and the latter doesn’t appear to have a test262 runner. Note, I haven’t actually built either one of these (or any of the other frameworks, for that matter).

The [ArkTS
runtime](https://gitee.com/openharmony/arkcompiler_ets_runtime) is itself built on a non-language-specific [common Ark
runtime](https://gitee.com/openharmony/arkcompiler_runtime_core), and the set of supported instructions is the union of the [core
ISA](https://gitee.com/openharmony/arkcompiler_runtime_core/blob/master/isa/isa.yaml) and the [JavaScript-specific
instructions](https://gitee.com/openharmony/arkcompiler_ets_runtime/blob/master/ecmascript/ecma_isa.yaml). Bytecode can be [interpreted](https://gitee.com/openharmony/arkcompiler_ets_runtime/blob/master/ecmascript/interpreter/interpreter-inl.h), JIT-compiled, or AOT-compiled.

On the side of design documentation, it’s somewhat sparse. There are some [core design
docs](https://gitee.com/openharmony/arkcompiler_runtime_core/tree/master/compiler/docs); readers may be interested in the [rationale to use a bytecode
interface](https://gitee.com/openharmony/arkcompiler_runtime_core/blob/master/docs/rationale-for-bytecode.md) for Ark as a whole, or the [optimization
overview](https://gitee.com/openharmony/arkcompiler_runtime_core/blob/master/docs/ir_format.md).

Indeed ArkTS as a whole has a surfeit of optimizations, to an extent that makes me wonder which ones are actually needed. There are [source-to-source optimizations on
bytecode](https://gitee.com/openharmony/arkcompiler_runtime_core/tree/master/bytecode_optimizer), which I expect are useful if you are generating ArkTS bytecode from JavaScript, where you probably don’t have a full compiler implementation. There is a [completely separate
optimizer](https://gitee.com/openharmony/arkcompiler_ets_runtime/blob/master/ecmascript/compiler/pass.h) in the eTS part of the run-time, based on what would appear to be a novel [“circuit-based”
IR](https://gitee.com/openharmony/arkcompiler_ets_runtime/blob/master/ecmascript/compiler/circuit_ir_specification.md) that bears some similarity to sea-of-nodes. Finally [the whole thing
appears to bottom out in
LLVM](https://gitee.com/openharmony/arkcompiler_ets_runtime/blob/master/ecmascript/compiler/llvm_ir_builder.h), which of course has its own optimizer. I can only assume that this situation is somewhat transitory. Also, ArkTS does appear to generate its own native code sometimes, notably for inline cache stubs.

Of course, when it comes to JavaScript, there are some fundamental language semantics and there is also a large and growing standard library. In the case of ArkTS, this standard library is [part of the
run-time](https://gitee.com/openharmony/arkcompiler_ets_runtime/tree/master/ecmascript), like the interpreter, compilers, and the garbage collector ( [generational concurrent mark-sweep with optional
compaction](https://gitee.com/openharmony/arkcompiler_ets_runtime/tree/master/ecmascript/mem)).

All in all, when I step back from it, it’s a huge undertaking. Implementing JavaScript is no joke. It appears that ArkTS has done the first 90% though; the proverbial second 90% should only take a few more years :)

### Evaluation

If you told a younger me that a major smartphone vendor switched from Java to JavaScript for their UI, you would probably hear me react in terms of the relative virtues of the programming languages in question. At this point in my career, though, the only thing that comes to mind is what an *expensive* proposition it is to change everything about an application development framework. 200 people over 5 years would be my estimate, though of course teams are variable. So what is it that we can imagine that Huawei bought with a thousand person-years of investment? Towards what other local maximum are we heading?

### Startup latency

I didn’t mention it before, but it would seem that one of the goals of HarmonyOS is in the name: Huawei wants to harmonize development across the different range of deployment targets. To the extent possible, it would be nice to be able to write the same kinds of programs for IoT devices as you do for feature-rich smartphones and tablets and the like. In that regard one can see through all the source code how there is a culture of doing work ahead-of-time and preventing work at run-time; for example see the [design doc for the
interpreter](https://gitee.com/openharmony/arkcompiler_runtime_core/blob/master/docs/design-of-interpreter.md), or for the [file
format](https://gitee.com/openharmony/arkcompiler_runtime_core/blob/master/docs/file_format.md), or indeed the lack of JavaScript `eval`.

Of course, this wide range of targets also means that the HarmonyOS platform bears the burden of a high degree of abstraction; not only can you change the kernel, but also the JavaScript engine (using [JerryScript](https://jerryscript.net/) on “lite” targets).

I mention this background because sometimes in news articles and indeed official communication from recent years there would seem to be some confusion that HarmonyOS is just for IoT, or aimed to be super-small, or something. In this evaluation I am mostly focussed on the feature-rich side of things, and there my understanding is that the developer will generate bytecode ahead-of-time. When an app is installed on-device, the AOT compiler will turn it into a single ELF image. This should generally lead to fast start-up.

However it would seem that [the rendering
library](https://gitee.com/openharmony/arkui_ace_engine/tree/master/frameworks/core/pipeline/base) that paints UI nodes into layers and then composits those layers uses Skia in the way that Flutter did pre-Impeller, which to be fair is a quite recent change to Flutter. I expect therefore that Ark (ArkTS + ArkUI) applications also experience shader compilation jank at startup, and that they may be well-served by tesellating their shapes into primitives like Impeller does so that they can precompile a fixed, smaller set of shaders.

### Jank

Maybe it’s just that apparently I think Flutter is great, but ArkUI’s fundamental architectural similarity to Flutter makes me think that jank will not be a big issue. There is a render thread that is separate from the ArkTS thread, so like with Flutter, async communication with main-thread interfaces is the main jank worry. And on the ArkTS side, ArkTS even has a number of extensions to be able to share objects between threads without copying, should that be needed. I am not sure how well-developed and well-supported these extensions are, though.

I am hedging my words, of course, because I am missing a bit of social proof; HarmonyOS is still in infant days, and it doesn’t have much in the way of a user base outside China, from what I can tell, and my ability to read Chinese is limited to what Google Translate can do for me :) Unlike other frameworks, therefore, I haven’t been as able to catch a feel of the pulse of the ArkUI user community: what people are happy about, what the pain points are.

It’s also interesting that unlike iOS or Android, HarmonyOS is *only* exposing these “web-like” and “declarative” UI frameworks for app development. This makes it so that the same organization is responsible for the software from top to bottom, which can lead to interesting cross-cutting optimizations: functional reactive programming isn’t just a developer-experience narrative, but it can directly affect the shape of the rendering pipeline. If there is jank, someone in the building is responsible for it and should be able to fix it, whether it is in the GPU driver, the kernel, the ArkTS compiler, or the application itself.

### Peak performance

I don’t know how to evaluate ArkTS for peak performance. Although there is a JIT compiler, I don’t have the feeling that it is as tuned for adaptive optimization as V8 is.

At the same time, I find it interesting that HarmonyOS has chosen to modify JavaScript. While it is doing that, could they switch to a sound type system, to allow the kinds of AOT optimizations that Dart can do? It would be an interesting experiment.

As it is, though, if I had to guess, I would say that ArkTS is well-positioned for predictably good performance with AOT compilation, although I would be interested in seeing the results of actually running it.

### Aside: On the importance of storytelling

In this series I have tried to be charitable towards the frameworks that I review, to give credit to what they are trying to do, even while noting where they aren’t currently there yet. That’s part of why I need a plausible narrative for how the frameworks got where they are, because that lets me have an idea of where they are going.

In that sense I think that Ark is at an interesting inflection point. When I started reading documentation about ArkUI and HarmonyOS and all that, I bounced out—there were too many architectural box diagrams, too many generic descriptions of components, too many promises with buzzwords. It felt to me like the project was trying to justify itself to a kind of clueless management chain. Was there actually anything here?

But now when I see the adoption of a modern rendering architecture and a bold new implementation of JavaScript, along with the willingness to experiment with the language, I think that there is an interesting story to be told, but this time not to management but to app developers.

Of course you wouldn’t want to market to app developers when your system is still a mess because you haven’t finished rebuilding an MVP yet. Retaking my charitable approach, then, I can only think that all the architectural box diagrams were a clever blind to avoid piquing outside interest while the app development kit wasn’t ready yet :) As and when the system starts working well, presumably over the next year or so, I would expect HarmonyOS to invest much more heavily in marketing and developer advocacy; the story is interesting, but you have to actually tell it.

### Aside: O platform, my platform

All of the previous app development frameworks that we looked at were cross-platform; Ark is not. It could be, of course: it does appear to be thoroughly open source. But HarmonyOS devices are the main target. What implications does this have?

A similar question arises in perhaps a more concrete way if we start with the mature Flutter framework: what would it mean to make a Flutter phone?

The first thought that comes to mind is that having a Flutter OS would allow for the potential for more cross-cutting optimizations that cross abstraction layers. But then I think, what does Flutter really need? It has the GPU drivers, and we aren’t going to re-implement those. It has the bridge to the platform-native SDK, which is not such a large and important part of the app. You get input from the platform, but that’s also not so specific. So maybe optimization is not the answer.

On the other hand, a Flutter OS would not have to solve the make-it-look-native problem; because there would be no other “native” toolkit, your apps won’t look out of place. That’s nice. It’s not something that would make the *platform* compelling, though.

HarmonyOS does have this embryonic concept of app mobility, where like you could put an app from your phone on your fridge, or something. Clearly I am not doing it justice here, but let’s assume it’s a compelling use case. In that situation it would be nice for all devices to present similar abstractions, so you could somehow install the same app on two different kinds of devices, and they could communicate to transfer data. As you can see here though, I am straying far from my domain of expertise.

One reasonable way to “move” an app is to have it stay running on your phone and the phone just communicates pixels with your fridge (or whatever); this is the low-level solution. I think HarmonyOS appears to be going for the higher-level solution where the app actually runs logic on the device. In that case it would make sense to ship UI assets and JavaScript / extended TypeScript bytecode to the device, which would run the app with an interpreter (for low-powered devices) or use JIT/AOT compilation. The Ark runtime itself would live on all devices, specialized to their capabilities.

In a way this is the Apple WatchOS solution (as I understand it); developers publish their apps as LLVM bitcode, and Apple compiles it for the specific devices. A FlutterOS with a Flutter run-time on all devices could do something similar. As with WatchOS you wouldn’t have to ship the framework itself in the app bundle; it would be on the device already.

Finally, publishing apps as some kind of intermediate representation also has security benefits: as the OS developer, you can ensure some invariants via the toolchain that you control. Of course, you would have to ensure that the Flutter API is sufficiently expressive for high-performance applications, while also not having arbitrary machine code execution vulnerabilities; there is a question of language and framework design as well as toolchain and runtime quality of implementation. HarmonyOS could be headed in this direction.

### Conclusion

Ark is a fascinating effort that holds much promise. It’s also still in motion; where will it be when it anneals to its own local optimum? It would appear that the system is approaching usability, but I expect a degree of churn in the near-term as Ark designers decide which language abstractions work for them and how to, well, *harmonize* them with the rest of JavaScript.

For me, the biggest open question is whether developers will love Ark in the way they love, say, React. In a market where Huawei is still a dominant vendor, I think the material conditions are there for a good developer experience: people tend to like Flutter and React, and Ark is similar. Huawei “just” needs to explain their framework well (and where it’s hard to explain, to go back and change it so that it is explainable).

But in a more heterogeneous market, to succeed Ark would need to make a cross-platform runtime like the one Flutter has and engage in some serious marketing efforts, so that developers don’t have to limit themselves to targetting the currently-marginal HarmonyOS. Selling extensions to JavaScript will be much more difficult in a context where the competition is already established, but perhaps Ark will be able to productively engage with TypeScript maintainers to move the language so it captures some of the benefits of Dart that facilitate ahead-of-time compilation.

Well, that’s it for my review round-up; hope you have enjoyed the series. I have one more pending article, speculating about some future technologies. Until then, happy hacking, and see you next time.

## related articles

- [parallel futures in mobile application development](/archives/2023/06/15/parallel-futures-in-mobile-application-development)
- [structure and interpretation of flutter](/archives/2023/04/26/structure-and-interpretation-of-flutter)
- [structure and interpretation of nativescript](/archives/2023/04/24/structure-and-interpretation-of-nativescript)
- [structure and interpretation of react native](/archives/2023/04/21/structure-and-interpretation-of-react-native)
- [structure and interpretation of capacitor programs](/archives/2023/04/20/structure-and-interpretation-of-capacitor-programs)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)

Comments are closed.
