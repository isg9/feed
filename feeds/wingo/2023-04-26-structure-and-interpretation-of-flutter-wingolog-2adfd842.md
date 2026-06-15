---
title: structure and interpretation of flutter — wingolog
url: https://wingolog.org/archives/2023/04/26/structure-and-interpretation-of-flutter
published: "2023-04-26T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/04/26/structure-and-interpretation-of-flutter
---

# structure and interpretation of flutter — wingolog

## [structure and interpretation of flutter](/archives/2023/04/26/structure-and-interpretation-of-flutter)

26 April 2023 1:50 PM

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

Good day, gentle hackfolk. Like an [old-time](https://en.wikipedia.org/wiki/Old-time_music) fiddler I would appear to be deep in the groove, playing endless variations on a theme, in this case mobile application frameworks. But one can only recognize novelty in relation to the familiar, and today’s note is a departure: we are going to look at [Flutter](https://flutter.dev/), a UI toolkit based not on JavaScript but on the [Dart](https://dart.dev/) language.

### The present, from the past

Where to start, even? The problem is big enough that I’ll approach it from three different angles: from the past, from the top, and from the bottom.

With the other frameworks we looked at, we didn’t have to say much about their use of JavaScript. JavaScript is an obvious choice, in 2023 at least: it is ubiquitous, has high quality implementations, and as a language it is quite OK and progressively getting better. Up to now, “always bet on JS” has had an uninterrupted winning streak.

But winning is not the same as unanimity, and Flutter and Dart represent an interesting pole of contestation. To understand how we got here, we have to go back in time. Ten years ago, JavaScript just wasn’t a great language: there were no modules, no async functions, no destructuring, no classes, no extensible iteration, no optional arguments to functions. In addition it was hobbled with a significant degree of what can only be called accidental sloppiness: `with` which can dynamically alter a lexical scope, direct `eval` that can define new local variables, `Function.caller`, and so on. Finally, larger teams were starting to feel the need for more rigorous language tooling that could use types to prohibit some classes of invalid programs.

All of these problems in JavaScript have been addressed over the last decade, mostly successfully. But in 2010 or so if you were a virtual machine engineer, you might look at JavaScript and think that in some other world, things could be a lot better. That’s effectively what happened: the team that originally built V8 broke off and started to work on what became Dart.

Initially, Dart was targetted for inclusion in the Chrome web browser as an alternate “native” browser language. This didn’t work, for various reasons, but since then Dart grew the Flutter UI toolkit, which has breathed new life into the language. And this is a review of Flutter, not a review of Dart, not really anyway; to my eyes, Dart is spiritually another JavaScript, different but in the same family. Dart’s implementation has many interesting aspects as well that we’ll get into later on, but all of these differences are incidental: they could just as well be implemented on top of JavaScript, TypeScript, or another source language in that family. Even if Flutter isn’t strictly part of the JavaScript-based mobile application development frameworks that we are comparing, it is valuable to the extent that it shows what is possible, and in that regard there is much to say.

### Flutter, from the top

At its most basic, Flutter is a UI toolkit for Dart. In many ways it is like React. Like React, its interface follows the functional-reactive paradigm: programmers describe the “what”, and Flutter takes care of the “how”. Also, like the phenomenon in which new developers can learn React without really knowing JavaScript, Flutter is the killer app for Dart: Flutter developers mostly learn Dart at the same time that they pick up Flutter.

In some other ways, Flutter is the logical progression of React, going in the same direction but farther along. Whereas React-on-the-web takes the user’s declarative specifications of what the UI should look like and lowers them into DOM trees, and [React Native lowers them to
platform-native UI
widgets](https://wingolog.org/archives/2023/04/21/structure-and-interpretation-of-react-native), Flutter has its own built-in layout, rasterization, and compositing engine: Flutter draws all the pixels.

This has the predictable challenge that Flutter has to make significant investments so that its applications don’t feel out-of-place on their platform, but on the other hand it opens up a huge space for experimentation and potential optimization: Flutter has the potential to beat native at its own game. Recall that with React Native, the result of the [render-commit-mount
process](https://reactnative.dev/architecture/render-pipeline) is a tree of native widgets. The native platform will surely then perform a kind of layout on those widgets, divide them into layers that correspond to GPU textures, paint those layers, then composite them to the screen – basically, [what a web engine will
do](https://developer.chrome.com/articles/layoutng/).

What if we could instead skip the native tree and go directly to the lower GPU layer? That is the promise of Flutter. Flutter has the potential to create much more smooth and complex animations than the other application development frameworks we have mentioned, with lower overhead and energy consumption.

In practice... that’s always the question, isn’t it? Again, please accept my usual caveat that I am a compilers guy moonlighting in the user interface domain, but my understanding is that Flutter mostly lives up to its promise, but with one significant qualification which we’ll get to in a minute. But before that, let’s traverse Flutter from the other direction, coming up from Dart.

### Dart, from the bottom

To explain some aspects of Dart I’m going to tell a just-so story that may or may not be true. I know and like many of the Dart developers, and we have similar instincts, so it’s probably not too far from the truth.

Let’s say you are the team that originally developed V8, and you decide to create a new language. You write a new virtual machine that looks like V8, taking Dart source code as input and applying advanced adaptive compilation techniques to get good performance. You can even be faster than JS because your language is just a bit more rigid than JavaScript is: you have traded off expressivity for performance. (Recall from [our
discussion of
NativeScript](https://wingolog.org/archives/2023/04/24/structure-and-interpretation-of-nativescript) that expressivity isn’t a value judgment: there can be reasons to pay for more “mathematically appealing operational equivalences”, in Felleisen’s language, in exchange for applying more constraints on a language.)

But, you fail to [ship the VM in a
browser](https://lists.webkit.org/pipermail/webkit-dev/2011-December/018775.html); what do you do? The project could die; that would be annoying, but you work for Google, so it happens all the time. However, a few interesting things happen around the same time that will cause you to pivot. One is a concurrent experiment by Chrome developers to pare the web platform down to its foundations and rebuild it. This effort will eventually become Flutter; while it was originally [based on
JS](https://github.com/flutter/flutter/commit/00882d626a478a3ce391b736234a768b762c853a), eventually they will choose to [switch to
Dart](https://bugs.chromium.org/p/chromium/issues/detail?id=454613).

The second thing that happens is that recently-invented smart phones become ubiquitous. Most people have one, and the two platforms are iOS and Android. Flutter wants to target them. You are looking for your niche, and you see that mobile application development might be it. As the Flutter people continue to experiment, you start to think about what it would mean to target mobile devices with Dart.

[The initial Dart VM was made to
JIT](https://mrale.ph/talks/vmil2020/#/3), but as we know, Apple doesn’t let people do this on iOS. So instead you look to write a quick-and-dirty ahead-of-time compiler, based on your JIT compiler that takes your program as input, parses and baseline-compiles it, and generates an image that can be loaded at runtime. It ships on iOS. Funnily enough, it ships on Android too, because AOT compilation allows you to avoid some startup costs; forced to choose between peak performance via JIT and fast startup via AOT, you choose fast startup.

It’s a success, you hit your product objectives, and you start to look further to a proper ahead-of-time compiler native code that can stand alone without the full Dart run-time. After all, if you have to compile at build-time, you might as well [take the time to do some proper
optimizations](https://github.com/dart-lang/sdk/issues/30480). You actually [change the language to have a sound typing
system](https://medium.com/dartlang/dart-and-the-performance-benefits-of-sound-types-6ceedd5b6cdc) so that the compiler can make program transformations that are valid as long as it can rely on the program’s types.

Fun fact: I am told that the shift to a sound type system actually started before Flutter and thus before AOT, because of a Dart-to-JavaScript compiler that you inherited from the time in which you thought the web would be the main target. The Dart-to-JS compiler used to be a whole-program compiler; this enabled it to do flow-sensitive type inference, resulting in faster and smaller emitted JavaScript. But whole-program compilation doesn’t scale well in terms of compilation time, so Dart-to-JS switched to separate per-module compilation. But then you lose lots of types! The way to recover the fast-and-small-emitted-JS property was through a stronger, sound type system for Dart.

At this point, you still have your virtual machine, plus your ahead-of-time compiler, plus your Dart-to-JS compiler. Such riches, such bounty! It is not a bad situation to be in, in 2023: you can offer a good development experience via the just-in-time compiled virtual machine. Apparently you can even use the JIT on iOS in developer mode, because attaching `ptrace` to a binary allows for native code generation. Then when you go to deploy, you make a native binary that includes everything.

For the web, you also have your nice story, even nicer than with JavaScript in some ways: because the type checker and ahead-of-time compiler are integrated in Dart, you don’t have to worry about WebPack or Vite or minifiers or uglifiers or TypeScript or JSX or Babel or any of the other things that JavaScript people are used to. Granted, the tradeoff is that innovation is mostly centralized with the Dart maintainers, but currently Google seems to be investing enough so that’s OK.

Stepping back, this story is not unique to Dart; many of its scenes also played out in the world of JavaScript over the last 5 or 10 years as well. [Hermes](https://hermesengine.dev/) (and [QuickJS](https://bellard.org/quickjs/), for that matter) does ahead-of-time compilation, albeit only to bytecode, and V8’s snapshot facility is a form of native AOT compilation. But the tooling in the JavaScript world is more diffuse than with Dart. With the perspective of developing a new JavaScript-based mobile operating system in mind, the advantages that Dart (and thus Flutter) has won over the years are also on the table for JavaScript to win. Perhaps even TypeScript could eventually migrate to have a sound type system, over time; it would take a significant investment but the JS ecosystem does evolve, if slowly.

(Full disclosure: while the other articles in this series were written without input from the authors of the frameworks under review, through what I can only think was good URL guesswork, a draft copy of this article leaked to Flutter developers. Dart hacker Slava Egorov kindly sent me a mail correcting a number of misconceptions I had about Dart’s history. Fair play on whoever guessed the URL, and many thanks to Slava for the corrections; any remaining errors are wholly mine, of course!)

### Evaluation

So how do we expect Flutter applications to perform? If we were writing a new mobile OS based on JavaScript, what would it mean in terms of performance to adopt a Flutter-like architecture?

### Startup latency

Flutter applications are well-positioned to start fast, with ahead-of-time compilation. However they have had [problems realizing
this potential](https://github.com/flutter/flutter/projects/188), with many users seeing a big stutter when they launch a Flutter app.

To explain this situation, consider the structure of a typical low-end Android mobile device: you have a small number of not-terribly-powerful CPU cores, but attached to the same memory you also have a decent GPU with many cores. For example, the [SoC in the low-end Moto E7
Plus](https://www.cpu-monkey.com/en/cpu-qualcomm_snapdragon_460) has 8 CPU cores and 128 GPU cores (texture shader units). You could paint widget pixels into memory from either the CPU or the GPU, but you’d rather do it in the GPU because it has so many more cores: in the time it takes to compute the color of a single pixel on the CPU, on the GPU you could do, like, 128 times as many, given that the comparison is often between multi-threaded rasterization on the GPU versus single-threaded rasterization on the CPU.

Flutter has always tried to paint on the GPU. Historically it has done so via a GPU back-end to the Skia graphics library, notably used by Chrome among other projects. But, Skia’s API is a drawing API, not a GPU API; Skia is the one responsible for configuring the GPU to draw what we want. And here’s the problem: configuring the GPU takes time. Skia generates shader code at run-time for rasterizing the specific widgets used by the Flutter programmer. That shader code then needs to be compiled to the language the GPU driver wants, which looks more like [Vulkan](https://www.vulkan.org/) or [Metal](https://developer.apple.com/metal/). The process of compilation and linking takes time, potentially seconds, even.

The solution to “too much startup shader compilation” is much like the solution to “too much startup JavaScript compilation”: move this phase to build time. The new [Impeller](https://docs.flutter.dev/perf/impeller) rendering library does just that. However to do that, it had to change the way that Flutter renders: instead of having Skia generate specialized shaders at run-time, Impeller instead lowers the shapes that it draws to a fixed set of primitives, and then renders those primitives using a [smaller,
fixed set of
shaders](https://github.com/flutter/engine/tree/main/impeller/entity/shaders). These primitive shaders are pre-compiled at build time and included in the binary. By switching to this new renderer, Flutter should be able to avoid startup jank.

### Jank

Of all the application development frameworks we have considered, to my mind Flutter is the best positioned to avoid jank. It has the React-like asynchronous functional layout model, but “closer to the metal”; by skipping the tree of native UI widgets, it can potentially spend less time for each frame render.

When you start up a Flutter app on iOS, the shell of the application is actually written in Objective C++. On Android it’s the same, except that it’s Java. That shell then creates a FlutterView widget and spawns a new thread to actually run Flutter (and the user’s Dart code). Mostly, Flutter runs on its own, rendering frames to the GPU resources backing the FlutterView directly.

If a Flutter app needs to communicate with the platform, it [passes
messages across an asynchronous channel back to the main
thread](https://docs.flutter.dev/development/platform-integration/platform-channels). Although these messages are asynchronous, this is probably the largest potential source of jank in a Flutter app, outside the initial frame paint: any graphical update which depends on the answer to an asynchronous call may lag.

### Peak performance

Dart’s type system and ahead-of-time compiler optimize for predictable good performance rather than the more variable but potentially higher peak performance that could be provided by just-in-time compilation.

This story should probably serve as a lesson to any future platform. The people that developed the original Dart virtual machine had a built-in bias towards just-in-time compilation, because it allows the VM to generate code that is specialized not just to the program but also to the problem at hand. A given system with ahead-of-time compilation can always be made to perform better via the addition of a just-in-time compiler, so the initial focus was on JIT compilation. On iOS of course this was not possible, but on Android and other platforms where this was available it was the default deployment model.

However, even Android switched to ahead-of-time compilation instead of the JIT model in order to reduce startup latency: doing any machine code generation at all at program startup was more work than was needed to get to the first frame. One could add JIT back again on top of AOT but it does not appear to be a high priority.

I would expect that Capacitor could beat Dart in some raw throughput benchmarks, given that [Capacitor’s JavaScript implementation can take
advantage of the platform’s native JIT capability](https://wingolog.org/archives/2023/04/20/structure-and-interpretation-of-capacitor-programs). Does it matter, though, as long as you are hitting your frame budget? I do not know.

### Aside: An escape hatch to the platform

What happens if you want to embed a web view into a Flutter app?

If you think on the problem for a moment I suspect you will arrive at the unsatisfactory answer, which is that for better or for worse, at this point it is too expensive even for Google to make a new web engine. Therefore Flutter will have to embed the native WebView. However Flutter runs on its own threads; the native WebView has its own process and threads but its interface to the app is tied to the main UI thread.

Therefore either you need to make the native WebView (or indeed any other native widget) render itself to (a region of) Flutter’s GPU backing buffer, or you need to copy the native widget’s pixels into their own texture and then composite them in Flutter-land. It’s not so nice! The [Android](https://docs.flutter.dev/development/platform-integration/android/platform-views) and [iOS](https://docs.flutter.dev/development/platform-integration/ios/platform-views) platform view documentation discuss some of the tradeoffs and mitigations.

### Aside: For want of a canvas

There is a very funny situation in the React Native world in which, if the application programmer wants to draw to a canvas, they have to [embed a whole WebView into the React Native
app](https://github.com/react-native-webview/react-native-webview/blob/master/docs/Guide.md#communicating-between-js-and-native) and then [proxy the canvas calls into the
WebView](https://github.com/iddan/react-native-canvas#readme). Flutter is happily able to avoid this problem, because it includes its own drawing library with a canvas-like API. Of course, Flutter also has the luxury of defining its own set of standard libraries instead of necessarily inheriting them from the web, so when and if they want to provide equivalent but differently-shaped interfaces, they can do so.

Flutter manages to be more expressive than React Native in this case, without losing much in the way of understandability. Few people will have to reach to the canvas layer, but it is nice to know it is there.

### Conclusion

Dart and Flutter are terribly attractive from an engineering perspective. They offer a delightful API and a high-performance, flexible runtime with a built-in toolchain. Could this experience be brought to a new mobile operating system as its primary programming interface, based on JavaScript? React Native is giving it a try, but I think there may be room to take things further to own the application from the program all the way down to the pixels.

Well, that’s all from me on Flutter and Dart for the time being. Next up, a mystery guest; see you then!

## related articles

- [parallel futures in mobile application development](/archives/2023/06/15/parallel-futures-in-mobile-application-development)
- [structure and interpretation of ark](/archives/2023/05/02/structure-and-interpretation-of-ark)
- [structure and interpretation of nativescript](/archives/2023/04/24/structure-and-interpretation-of-nativescript)
- [structure and interpretation of react native](/archives/2023/04/21/structure-and-interpretation-of-react-native)
- [structure and interpretation of capacitor programs](/archives/2023/04/20/structure-and-interpretation-of-capacitor-programs)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)

Comments are closed.
