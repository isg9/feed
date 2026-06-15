---
title: structure and interpretation of capacitor programs — wingolog
url: https://wingolog.org/archives/2023/04/20/structure-and-interpretation-of-capacitor-programs
published: "2023-04-20T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/04/20/structure-and-interpretation-of-capacitor-programs
---

# structure and interpretation of capacitor programs — wingolog

## [structure and interpretation of capacitor programs](/archives/2023/04/20/structure-and-interpretation-of-capacitor-programs)

20 April 2023 10:20 AM

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

Good day, hackers! Today’s note is a bit of a departure from compilers internals. A client at work recently asked me to look into cross-platform mobile application development and is happy for the results to be shared publically. This, then, is the first in a series of articles.

### Mobile apps and JavaScript: how does it work?

I’ll be starting by taking a look at [Ionic](https://ionicframework.com/)/ [Capacitor](https://capacitorjs.com/), [React Native](https://reactnative.dev/), [NativeScript](https://nativescript.org/), [Flutter](https://flutter.dev/)/ [Dart](https://dart.dev/), and then a mystery guest. This article will set the stage and then look into Ionic/Capacitor.

The angle I am taking is, if you were designing a new mobile operating system that uses JavaScript as its native application development language, what would it mean to adopt one of these as your primary app development toolkit? It’s a broad question but I hope we can come up with some useful conclusions.

I’m going to approach the problem from the perspective of a compiler engineer. Compilers translate high-level languages that people like to write into low-level languages that machines like to run. Compilation is a bridge between developers and users: developers experience the source language interface to the compiler, while users experience the result of compiling those source languages. Note, though, that my expertise is mostly on the language and compiler side of things; though I have worked on a few user-interface libraries in my time, I am certainly not a graphics specialist, nor a user-experience specialist.

I’m explicitly not including the native application development toolkits from Android and iOS, because these are most useful to me as a kind of “control” to the experiment. A cross-platform toolkit that compiles down to use native widgets can only be as good as the official SDK, but it might be worse; and a toolkit that routes around the official widgets by using alternate drawing primitives has the possibility to be better, but at the risk of being inconsistent with other platform apps, along with the more general risk of not being as good as the native UI libraries.

And, of course, there is the irritation that SwiftUI / UIKit / AppKit aren’t not open source, and that neither iOS nor Android’s native UI library is cross platform. If we are designing a new JavaScript-based mobile operating system—that’s the conceit of the original problem statement—then we might as well focus on the existing JS-based toolkits. (Flutter of course is the odd one out, but it’s interesting enough to include in the investigation.)

### Ionic / Capacitor

Let’s begin with the [Ionic Framework](https://ionicframework.com/) UI toolkit, based on the [Capacitor](https://capacitorjs.com/) web run-time.

#### Overview

Capacitor is like an alternate web browser development kit for phones. It uses the platform’s native WebView: [WKWebView](https://developer.apple.com/documentation/webkit/wkwebview) on iOS or [WebView](https://developer.android.com/reference/android/webkit/WebView) on Android. The JavaScript APIs available within that WebView are extended with [Capacitor plugins](https://capacitorjs.com/docs/apis) which can access native APIs; these plugins let JavaScript do the sort of things you wouldn’t be able to do directly in a web browser.

(Over time, most of the features provided by Capacitor end up in the web platform in some way. For example, there are web standards to allow JavaScript to detect or lock screen rotation. The [Fugu
project](https://fugu-tracker.web.app/) is particularly avant-garde in pushing these capabilities to the web. But, the wheel of web standards grinds very finely, and for an app to have access to, say, the user’s contact list now, it is pragmatic to use an extended-webview solution like Capacitor.)

I call Capacitor a “web browser *development kit*” because creating a Capacitor project effectively copies the shell of a specialized browser into your source tree, usually with an iOS and Android version. Building the app makes a web browser with your extensions that runs your JS, CSS, image, and other assets. Running the app creates a WebView and invokes your JavaScript; your JS should then create the user interface using the DOM APIs that are standard in a WebView.

As you can see, Capacitor is quite full-featured when it comes to native platform capabilities but is a bit bare-bones when it comes to UI. To provide a richer development environment, the [Ionic
Framework](https://ionicframework.com/) adds a set of widgets and some application life-cycle abstractions on top of the Capacitor WebView. Ionic is not like a “normal” app framework like [Vue](https://vuejs.org/) or [Angular](https://angular.io/) because it takes advantage of Capacitor APIs, and because it tries to mimic the presentation and behavior of the native plaform (iOS or Android, mainly), so that apps built with it don’t look out-of-place.

The Ionic Framework itself is built using [Web
Components](https://developer.mozilla.org/en-US/docs/Web/API/Web_components), which end up creating styled DOM nodes with JavaScript behavior. These components can compose with other app frameworks such as Vue or Angular, allowing developers to share some of their app code between web and mobile apps—you write the main app in Vue, and just implement the components one way on the web and using something like Ionic Framework on mobile.

To recap, an Ionic app uses Ionic Framework libraries on top of a Capacitor run-time. You could use other libraries on top of Capacitor instead of Ionic. In any case, I’m most interested in Capacitor, so let’s continue our focus there.

#### Evaluation

Performance has a direct effect on the experience that users have when interacting with an application, and here I would like to break down the performance in three ways: one, startup latency; two, jank; and three, peak throughput. As Capacitor is structurally much like a web browser, many of these concerns, pitfalls, and mitigation techniques are similar to those on the web.

#### Startup latency

Startup latency is how long the app makes you wait after you run it before you can interact with it. The app may hide some of this work behind a splash screen, but if there is too much work to do at startup-time, the risk is the user might get bored or frustrated and switch away. The goal, therefore, it to minimize startup latency. Most people consider time-to-interactive of less than than a second to be sufficient; there’s room to improve but for better or for worse, user expectations here are lower than they could be.

In the case of Capacitor, when a user launches an application, the app loads a minimal skeleton program that loads a WebView. On iOS and most Android systems, the rendering and JS parts of the WebView run in a separate process that has access to the app’s bundled assets: JavaScript files, images, CSS, and so on. The WebView then executes the JavaScript `main` function to create the UI.

Capacitor application startup time will be dominated by [parsing and
compiling the JavaScript source
files](https://infrequently.org/2021/03/the-performance-inequality-gap/), as well as any initial work needed to boot the app framework (which could require running a significant amount of JS), and will depend on how fast the user’s device is. The most effective performance mitigation here is to reduce the amount of JavaScript loaded, but this can be difficult for complex apps. The main techniques to reduce app size are toolchain-based. [Tree-shaking](https://rollupjs.org/faqs/#what-is-tree-shaking), [bundling](https://esbuild.github.io/api/#bundle), and [minification](https://esbuild.github.io/api/#minify) reduce the number of JS bytes that the engine needs to parse without changing application logic. [Code
splitting](https://developer.mozilla.org/en-US/docs/Glossary/Code_splitting) can defer some work until it is needed, but this might just result in jank later on.

Common Ionic application sizes would seem to be between 500 kB and 5 MB of JavaScript. In 2021, [Alex Russell suggested that the soon-to-be
standard Android performance baseline should be the Moto E7
Plus](https://infrequently.org/2021/03/the-performance-inequality-gap/) which, if its [performance relative to the mid-range Moto G4
phone](https://browser.geekbench.com/v4/cpu/compare/15987586?baseline=15667410) that he measured in 2019 translates to JavaScript engine speed, should be able to parse and compile uncompressed JavaScript at a rate of about 1 MB/s. That’s not very much, if you want to get startup latency under a second, and a JS payload size of 5 MB would lead to 5-second startup delays for many users. To my mind, this is the biggest challenge for a Capacitor-based app.

(Calculation detail: The Moto G4 JavaScript parse-and-compile throughput is measured at 170 kB/s, for compressed JS. Assuming a compression ratio of 4 and that the Moto E7 Plus is 50% faster than the Moto G4, that gets us to 1 MB/s for uncompressed JS, for what was projected to be a performance baseline in 2023.)

#### Jank

Jank is when an application’s animation and interaction aren’t smooth, because the application somehow missed rendering one or more frames. Technically startup time is a form of jank, but it is useful to treat startup separately.

Generally speaking, the question of whether a Capacitor application will show jank depends on who is doing the rendering: if application JavaScript is driving an animation, then this could cause jank, in the same way as it would on the web. The mitigation is to lean on the web platform so that the WebView is the one in charge of ensuring smooth interaction, for example by using CSS animations or the native scrolling capability.

There may always be an impetus to do some animation in JavaScript, though, for example if the design guidelines require a specific behavior that can’t be created using raw CSS.

#### Peak perf

Peak throughput is how much work an app can do per unit time. For an application written in JavaScript, this is a question of how good can the JavaScript engine make a user’s code run fast.

Here we need to make an important aside on the strategies that a JavaScript engine uses to run a user’s code. A standard technique that JS engines use to make JavaScript run fast is just-in-time (JIT) compilation, in which the engine emits specialized machine code at run-time and then runs that code. However, on iOS the platform prohibits JIT code generation for most applications. The usual justification for this restriction is that the [low-level capability
granted to a program that allows it to JIT-compile increases the
likelihood of security
exploits](https://wingolog.org/archives/2011/06/21/security-implications-of-jit-compilation). The only current exception to the no-JIT policy on iOS is for WebView (including the one in the system web browser), which iOS maintainers consider to be sufficiently sandboxed, so that any security exploit that uses JIT code generation won’t be able to access other capabilities possessed by the application process.

So much for the motivation, as I understand it. The upshot is, there is no JIT on iOS outside web browsers or web views. If a cross-platform application development toolkit wants peak JavaScript performance on iOS, it has to run that JS in a WebView. Capacitor does this, so it has fast JS on iOS. Note that these restrictions are not in place on Android; you can have fast JS on Android without using a WebView, by using the system’s JavaScript libraries.

Of course, actually getting peak throughput out of JavaScript is an art but the well-known techniques for JavaScript-on-the-web apply here; we won’t cover them in this series.

#### The bridge

All of these performance observations are common to all web browsers, but with Capacitor there is the additional wrinkle that a Capacitor application can access native capabilities, for example access to a user’s contacts. When application JavaScript goes to access the contacts API, Capacitor will send a message to the native side of the application over a *bridge*. The message will be serialized to JSON. Capacitor will include an implementation for the native side of the bridge into your app’s source code, written in Swift for iOS or Java for Android. The native end of the bridge parses the JSON message, performs the requested action, and sends a message back with the result. Some messages may need to be proxied to the main application thread, because some native APIs can only be processed there.

As you can imagine, the bridge has an overhead. Apps that have high-bandwidth access to native capabilities will also have JSON encoding overhead, as well general asynchronous coordination overhead. It may even be possible that encoding or decoding a large JSON message causes the WebView to miss a frame, for example when accessing a large file in local storage.

The bridge is a necessary component to the design, though; an iOS WebView can’t have direct in-process access to native capabilities. For Android, the WebView APIs do not appear to allow this either, though it is theoretically possible to ship your own WebView that could access native APIs directly. In any case, the Capacitor multi-process solution does allow for some parallelism, and the enforced asynchronous nature of the APIs should lead to less modal application programming.

### Aside: Can WebView act like native widgets?

Besides performance, one way that users experience application behavior is by way of expectations: users expect (but don’t require) a degree of uniformity between different apps on the same platform, for example the same scrolling behavior or the same kinds of widgets. This aspect isn’t the most important one to my investigation, because I’m more concerned with a new operating system that might come with its own design language with its own standard JavaScript-based component library, but it’s one to note; an Ionic app is starting at a disadvantage relative to a platform-native app. Sometimes ensuring a platform-native UI is just a question of CSS styling and using the right library (like Ionic Framework), but sometimes it might take significant engineering or possibly even new additions to the web platform.

### Aside: Is a WebView as good as native widgets?

As a final observation on user experience, it’s worth asking the question of whether you can actually make a user interface using the web platform that is “as good” as native widgets. After all, the API and widget set (DOM) available to a WebView is very different from that available to a native application; is a WebView able to use CPU and GPU resources efficiently to create a smooth interface? Are web standards, CSS, and the DOM API somehow a constraint that prevents efficient user interface construction? Here, somewhat embarrassingly, I can’t actually say. I am instead going to just play the “compiler engineer” card, that I’m not a UI specialist; to me it is an open question.

### Summary

Capacitor, with or without the Ionic Framework on top, is the always-bet-on-the-web solution to the cross-platform mobile application development problem. It uses stock system WebViews and JavaScript, augmented with native capabilities as needed. Consistency with the UX of the specific platform (iOS or Android) may require significant investment, though.

Performance-wise, my evaluation is that Capacitor apps are well-positioned for peak JS performance due to JIT code generation and low jank due to offloading animations to the web platform, but may have problems with startup latency, as Capacitor needs to parse and compile a possibly significant amount of JavaScript each time it is run.

Next time, a different beast: [React Native](https://wingolog.org/archives/2023/04/21/structure-and-interpretation-of-react-native). Until then, happy hacking!

## related articles

- [parallel futures in mobile application development](/archives/2023/06/15/parallel-futures-in-mobile-application-development)
- [structure and interpretation of ark](/archives/2023/05/02/structure-and-interpretation-of-ark)
- [structure and interpretation of flutter](/archives/2023/04/26/structure-and-interpretation-of-flutter)
- [structure and interpretation of nativescript](/archives/2023/04/24/structure-and-interpretation-of-nativescript)
- [structure and interpretation of react native](/archives/2023/04/21/structure-and-interpretation-of-react-native)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)

Comments are closed.
