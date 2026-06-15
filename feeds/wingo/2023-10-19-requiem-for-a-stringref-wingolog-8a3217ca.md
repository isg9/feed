---
title: requiem for a stringref — wingolog
url: https://wingolog.org/archives/2023/10/19/requiem-for-a-stringref
published: "2023-10-19T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/10/19/requiem-for-a-stringref
---

# requiem for a stringref — wingolog

## [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)

19 October 2023 10:33 AM

- [webassembly](/tags/webassembly)
- [wasm](/tags/wasm)
- [igalia](/tags/igalia)
- [gc](/tags/gc)
- [stringref](/tags/stringref)
- [strings](/tags/strings)
- [unicode](/tags/unicode)
- [standards](/tags/standards)
- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [swift](/tags/swift)
- [python](/tags/python)
- [java](/tags/java)

Good day, comrades. Today’s missive is about strings!

### a problem for java

Imagine you want to compile a program to WebAssembly, with the new [GC
support for
WebAssembly](https://github.com/WebAssembly/gc/blob/master/proposals/gc/MVP.md). Your WebAssembly program will run on web browsers and render its contents using the DOM API: [`Document.createElement`](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElement), [`Document.createTextNode`](https://developer.mozilla.org/en-US/docs/Web/API/Document/createTextNode), and so on. It will also use DOM interfaces to read parts of the page and read input from the user.

How do you go about representing your program in WebAssembly? The GC support gives you the ability to define a number of different kinds of [aggregate data
types](https://webassembly.github.io/gc/core/syntax/types.html#aggregate-types): structs (records), arrays, and functions-as-values. Earlier versions of WebAssembly gave you 32- and 64-bit integers, floating-point numbers, and opaque references to host values ( `externref`). This is what you have in your toolbox. But what about strings?

WebAssembly’s historical answer has been to throw its hands in the air and punt the problem to its user. This isn’t so bad: the direct user of WebAssembly is a compiler developer and can fend for themself. Using the primitives above, it’s clear we should represent strings as some kind of array.

The source language may impose specific requirements regarding string representations: for example, in Java, you will want to use an `(array i16)`, because Java’s strings are specified as sequences of UTF-16¹ code units, and Java programs are written assuming that random access to a code unit is constant-time.

Let’s roll with the Java example for a while. It so happens that JavaScript, the main language of the web, also specifies strings in terms of 16-bit code units. The DOM interfaces are optimized for JavaScript strings, so at some point, our WebAssembly program is going to need to convert its `(array i16)` buffer to a JavaScript string. You can imagine that a high-throughput interface between WebAssembly and the DOM is going to involve a significant amount of copying; could there be a way to avoid this?

Similarly, Java is going to need to perform a number of gnarly operations on its strings, for example, locale-specific collation. This is a hard problem whose solution basically amounts to shipping a copy of [`libICU`](https://unicode-org.github.io/icu/userguide/icu/) in their WebAssembly module; that’s a lot of binary size, and it’s not even clear how to compile `libICU` in such a way that works on GC-managed arrays rather than linear memory.

Thinking about it more, there’s also the problem of regular expressions. A high-performance regular expression engine is a lot of investment, and not really portable from the native world to WebAssembly, as the main techniques require just-in-time code generation, which is unavailable on Wasm.

This is starting to sound like a terrible system: big binaries, lots of copying, suboptimal algorithms, and a likely ongoing functionality gap. What to do?

### a solution for java

One observation is that in the specific case of Java, we could just use JavaScript strings in a web browser, instead of implementing our own string library. We may need to make some shims here and there, but the basic functionality from JavaScript gets us what we need: constant-time UTF-16¹ code unit access from within WebAssembly, and efficient access to browser regular expression, internationalization, and DOM capabilities that doesn’t require copying.

A sort of minimum viable product for improving the performance of Java compiled to Wasm/GC would be to represent strings as [`externref`](https://webassembly.github.io/gc/core/syntax/types.html#syntax-heaptype), which is WebAssembly’s way of making an opaque reference to a host value. You would operate on those values by importing the equivalent of `String.prototype.charCodeAt` and friends; to get the receivers right you’d need to run them through [`Function.call.bind`](https://github.com/WebAssembly/stringref/issues/5#issuecomment-1243487288). It’s a somewhat convoluted system, but a WebAssembly engine could be taught to recognize such a function and compile it specially, using the same code that JavaScript compiles to.

(Does this sound too complicated or too distasteful to implement? Disabuse yourself of the notion: it’s happening already. V8 does this and other JS/Wasm engines will be forced to follow, as users file bug reports that such-and-such an app is slow on e.g. Firefox but fast on Chrome, and so on and so on. It’s the same dynamic that led `asm.js` adoption.)

Getting properly good performance will require a bit more, though. String literals, for example, would have to be loaded from e.g. UTF-8 in a WebAssembly data section, then transcoded to a JavaScript string. You need a function that can convert UTF-8 to JS string in the first place; let’s call it `fromUtf8Array`. An engine can now optimize the `array.new_data` \+ `fromUtf8Array` sequence to avoid the intermediate array creation. It would also be nice to tighten up the typing on the WebAssembly side: having everything be `externref` imposes a dynamic type-check on each operation, which is something that can’t always be elided.

### beyond the web?

“JavaScript strings for Java” has two main limitations: JavaScript and Java. On the first side, this MVP doesn’t give you anything if your WebAssembly host doesn’t do JavaScript. Although it’s a bit of a failure for a universal virtual machine, to an extent, the WebAssembly ecosystem is OK with this distinction: there are different compiler and toolchain options when targetting the web versus, say, Fastly’s edge compute platform.

But does that mean you can’t run Java on Fastly’s cloud? Does the Java compiler have to actually implement all of those things that we were trying to avoid? Will Java actually implement those things? I think the answers to all of those questions is “no”, but also that I expect a pretty crappy outcome.

First of all, it’s not technically required that Java implement its own strings in terms of `(array i16)`. A Java-to-Wasm/GC compiler can keep the strings-as-opaque-host-values paradigm, and instead have these string routines provided by an auxiliary WebAssembly module that itself probably uses `(array i16)`, effectively polyfilling what the browser would give you. The effort of creating this module can be shared between e.g. Java and C#, and the run-time costs for instantiating the module can be amortized over a number of Java users within a process.

However, I don’t expect such a module to be of good quality. It doesn’t seem possible to implement a good regular expression engine that way, for example. And, absent a very good run-time system with an adaptive compiler, I don’t expect the low-level per-codepoint operations to be as efficient with a polyfill as they are on the browser.

Instead, I could see non-web WebAssembly hosts being pressured into implementing their own built-in UTF-16¹ module which has accelerated compilation, a native regular expression engine, and so on. It’s nice to have a portable fallback but in the long run, first-class UTF-16¹ will be everywhere.

### beyond java?

The other drawback is Java, by which I mean, Java (and JavaScript) is outdated: if you were designing them today, their strings would not be UTF-16¹.

I keep this little “¹” sigil when I mention UTF-16 because Java (and JavaScript) don’t actually use UTF-16 to represent their strings. UTF-16 is standard *Unicode encoding form*. A Unicode encoding form encodes a sequence of *Unicode scalar values* (USVs), using one or two 16-bit *code units* to encode each USV. A USV is a *codepoint*: an integer in the range \[0,0x10FFFF\], but excluding *surrogate* *codepoints*: codepoints in the range \[0xD800,0xDFFF\].

Surrogate codepoints are an accident of history, and occur either when accidentally slicing a two-code-unit UTF-16-encoded-USV in the middle, or when treating an arbitrary `i16` array as if it were valid UTF-16. They are annoying to detect, but in practice are here to stay: no amount of wishing will make them go away from Java, JavaScript, C#, or other similar languages from those heady days of the mid-90s. Believe me, I have engaged in some serious wishing, but if you, the virtual machine implementor, want to support Java as a source language, your strings have to be accessible as 16-bit code units, which opens the door (eventually) to surrogate codepoints.

So when I say UTF-16¹, I really mean [WTF-16](https://simonsapin.github.io/wtf-8/): sequences of any 16-bit code units, without the UTF-16 requirement that surrogate code units be properly paired. In this way, WTF-16 encodes a larger language than UTF-16: not just USV codepoints, but also surrogate codepoints.

The existence of WTF-16 is a consequence of a kind of original sin, originating in the choice to expose 16-bit code unit access to the Java programmer, and which everyone agrees should be somehow firewalled off from the rest of the world. The usual way to do this is to prohibit WTF-16 from being transferred over the network or stored to disk: a message sent via an HTTP `POST`, for example, will never include a surrogate codepoint, and will either replace it with the U+FFFD replacement codepoint or throw an error.

But within a Java program, and indeed within a JavaScript program, there is no attempt to maintain the UTF-16 requirements regarding surrogates, because any change from the current behavior would break programs. (How many? Probably very, very few. But productively deprecating web behavior is hard to do.)

If it were just Java and JavaScript, that would be one thing, but WTF-16 poses challenges for using JS strings from non-Java languages. Consider that any JavaScript string can be invalid UTF-16: if your language defines strings as sequences of USVs, which excludes surrogates, what do you do when you get a fresh string from JS? Passing your string *to* JS is fine, because WTF-16 encodes a superset of USVs, but when you receive a string, you need to have a plan.

You only have a few options. You can eagerly check that a string is valid UTF-16; this might be a potentially expensive O(n) check, but perhaps this is acceptable. (This check may be [faster in the
future](https://github.com/tc39/proposal-is-usv-string).) Or, you can replace surrogate codepoints with U+FFFD, when accessing string contents; lossy, but preserves your language’s semantic domain. Or, you can extend your language’s semantics to somehow deal with surrogate codepoints.

My point is that if you want to use JS strings in a non-Java-like language, your language will need to define what to do with invalid UTF-16. Ideally the browser will give you a way to put your policy into practice: replace with U+FFFD, error, or pass through.

### beyond java? (reprise) (feat: snakes)

With that detail out of the way, say you are compiling Python to Wasm/GC. [Python’s language
reference](https://docs.python.org/3/reference/datamodel.html#immutable-sequences) says: “A string is a sequence of values that represent Unicode code points. All the code points in the range U+0000 - U+10FFFF can be represented in a string.” This corresponds to the domain of JavaScript’s strings; great!

On second thought, how do you actually access the contents of the string? Surely not via the equivalent of JavaScript’s `String.prototype.charCodeAt`; Python strings are sequences of codepoints, not 16-bit code units.

Here we arrive to the second, thornier problem, which is less about domain and more about idiom: in Python, we expect to be able to access strings by codepoint index. This is the case not only to access string contents, but also to refer to positions in strings, for example when extracting a substring. These operations need to be fast (or fast enough anyway; CPython doesn’t have a very high performance baseline to meet).

However, the web platform doesn’t give us O(1) access to string codepoints. Usually a codepoint just takes up one 16-bit code unit, so the (zero-indexed) 5th codepoint of JS string `s` may indeed be at `s.codePointAt(5)`, but it may also be at offset 6, 7, 8, 9, or 10. You get the point: finding the *n* th codepoint in a JS string requires a linear scan from the beginning.

More generally, *all* languages will want to expose O(1) access to *some* primitive subdivision of strings. For Rust, this is bytes; 8-bit bytes are the *code units* of UTF-8. For others like Java or C#, it’s 16-bit code units. For Python, it’s codepoints. When targetting JavaScript strings, there may be a performance impedance mismatch between what the platform offers and what the language requires.

Languages also generally offer some kind of string iteration facility, which doesn’t need to correspond to how a JavaScript host sees strings. In the case of Python, one can implement `for char in s: print(char)` just fine on top of JavaScript strings, by decoding WTF-16 on the fly. Iterators can also map between, say, UTF-8 offsets and WTF-16 offsets, allowing e.g. Rust to preserve its preferred “strings are composed of bytes that are UTF-8 code units” abstraction.

Our O(1) random access problem remains, though. Are we stuck?

### what does the good world look like

How should a language represent its strings, anyway? Here we depart from a precise gathering of requirements for WebAssembly strings, but in a useful way, I think: we should build abstractions not only for what is, but also for what should be. We should favor a better future; imagining the ideal helps us design the real.

I keep returning to Henri Sivonen’s authoritative article, [It’s Not
Wrong that “🤦🏼‍♂️”.length == 7, But It’s Better that “🤦🏼‍♂️”.len() == 17
and Rather Useless that len(“🤦🏼‍♂️”) ==
5](https://hsivonen.fi/string-length/). It is *so* good and if you have reached this point, pop it open in a tab and go through it when you can. In it, Sivonen argues (among other things) that random access to codepoints in a string is not actually important; he thinks that if you were designing Python today, you wouldn’t include this interface in its standard library. Users would prefer extended grapheme clusters, which is variable-length anyway and a bit gnarly to compute; storage wants bytes; array-of-codepoints is just a bad place in the middle. Given that UTF-8 is more space-efficient than either UTF-16 or array-of-codepoints, and that it embraces the variable-length nature of encoding, programming languages should just use that.

As a model for how strings are represented, array-of-codepoints is outdated, as indeed is UTF-16. Outdated doesn’t mean irrelevant, of course; there is lots of Python code out there and we have to support it somehow. But, if we are designing for the future, we should nudge our users towards other interfaces.

There is even a case that a JavaScript engine should represent its strings as UTF-8 internally, despite the fact that JS exposes a UTF-16 view on strings in its API. The pitch is that UTF-8 takes less memory, is probably what we get over the network anyway, and is probably what many of the low-level APIs that a browser uses will want; it would be faster and lighter-weight to pass UTF-8 to text shaping libraries, for example, compared to passing UTF-16 or having to copy when going to JS and when going back. JavaScript engines already have a dozen internal string representations or so (narrow or wide, cons or slice or flat, inline or external, interned or not, and the product of many of those); adding another is just a Small Matter Of Programming that could show benefits, even if some strings have to be later transcoded to UTF-16 because JS accesses them in that way. I have talked with JS engine people in all the browsers and everyone thinks that UTF-8 has a chance at being a win; the drawback is that actually implementing it would take a lot of effort for uncertain payoff.

I have two final data-points to indicate that UTF-8 is the way. One is that [Swift](https://swift.org/) used to use UTF-16 to represent its strings, but was able to [switch to
UTF-8](https://www.swift.org/blog/utf8-string/). To adapt to the newer performance model of UTF-8, Swift maintainers designed new APIs to allow users to request a *view* on a string: treat this string as UTF-8, or UTF-16, or a sequence of codepoints, or even a sequence of extended grapheme clusters. Their users appear to be happy, and I expect that many languages will follow Swift’s lead.

Secondly, as a maintainer of the [Guile
Scheme](https://gnu.org/s/guile/) implementation, I also want to switch to UTF-8. Guile has long used Python’s representation strategy: array of codepoints, with an optimization if all codepoints are “narrow” (less than 256). The Scheme language exposes codepoint-at-offset ( `string-ref`) as one of its fundamental string access primitives, and array-of-codepoints maps well to this idiom. However, we do plan to move to UTF-8, with a Swift-like [breadcrumbs](https://www.swift.org/blog/utf8-string/#breadcrumbs) strategy for accelerating per-codepoint access. We hope to lower memory consumption, simplify the implementation, and have general (but not uniform) speedups; some things will be slower but most should be faster. Over time, users will learn the performance model and adapt to prefer string builders / iterators (“string ports”) instead of `string-ref`.

### a solution for webassembly in the browser?

Let’s try to summarize: it definitely makes sense for Java to use JavaScript strings when compiled to WebAssembly/GC, when running on the browser. There is an OK-ish compilation strategy for this use case involving `externref`, `String.prototype.charCodeAt` imports, and so on, along with some engine heroics to specially recognize these operations. There is an [early
proposal](https://github.com/WebAssembly/js-string-builtins/blob/main/proposals/js-string-builtins/Overview.md) to sand off some of the rough edges, to make this use-case a bit more predictable. However, there are two limitations:

1. Focussing on providing JS strings to Wasm/GC is only really good for Java and friends; the cost of mapping `charCodeAt` semantics to, say, Python’s strings is likely too high.

2. JS strings are only present on browsers (and Node and such).

I see the outcome being that Java will have to keep its implementation that uses `(array i16)` when targetting the edge, and use JS strings on the browser. I think that polyfills will not have acceptable performance. On the edge there will be a binary size penalty and a performance and functionality gap, relative to the browser. Some edge Wasm implementations will be pushed to implement fast JS strings by their users, even though they don’t have JS on the host.

If the JS string builtins proposal were a local maximum, I could see putting some energy into it; it does make the Java case a bit better. However I think it’s likely to be an unstable saddle point; if you are going to infect the edge with WTF-16 anyway, you might as well step back and try to solve a problem that is a bit more general than Java on JS.

### stringref: a solution for webassembly?

I think WebAssembly should just bite the bullet and try to define a string data type, for languages that use GC. It should support UTF-8 and UTF-16 views, like Swift’s strings, and support some kind of iterator API that decodes codepoints.

It should be abstract as regards the concrete representation of strings, to allow JavaScript strings to stand in for WebAssembly strings, in the context of the browser. JS hosts will use UTF-16 as their internal representation. Non-JS hosts will likely prefer UTF-8, and indeed an abstract API favors migration of JS engines away from UTF-16 over the longer term. And, such an abstraction should give the user control over what to do for surrogates: allow them, throw an error, or replace with U+FFFD.

What I describe is what the [stringref](https://github.com/WebAssembly/stringref/blob/main/proposals/stringref/Overview.md) proposal gives you. We don’t yet have consensus on this proposal in the Wasm standardization group, and we may never reach there, although I think it’s still possible. As I understand them, the objections are two-fold:

1. WebAssembly is an instruction set, like AArch64 or x86. Strings are too high-level, and should be built on top, for example with `(array i8)`.

2. The requirement to support fast WTF-16 code unit access will mean that we are effectively standardizing JavaScript strings.

I think the first objection is a bit easier to overcome. Firstly, WebAssembly now defines quite a number of components that don’t map to machine ISAs: typed and extensible locals, `memory.copy`, and so on. You could have defined `memory.copy` in terms of primitive operations, or required that all local variables be represented on an explicit stack or in a fixed set of registers, but WebAssembly defines higher-level interfaces that instead allow for more efficient lowering to machine primitives, in this case SIMD-accelerated copies or machine-specific sets of registers.

Similarly with garbage collection, there was a very interesting [“continuation marks”
proposal](https://github.com/WebAssembly/exception-handling/issues/105) by Ross Tate that would give a low-level primitive on top of which users could implement root-finding of stack values. However when choosing what to include in the standard, the group preferred a more high-level facility in which a Wasm module declares managed data types and allows the WebAssembly implementation to do as it sees fit. This will likely result in more efficient systems, as a Wasm implementation can more easily use concurrency and parallelism in the GC implementation than a guest WebAssembly module could do.

So, the criteria of what to include in the Wasm standard is not “what is the most minimal primitive that can express this abstraction”, or even “what looks like an ARMv8 instruction”, but rather “what makes Wasm a good compilation target”. Wasm is designed for its compiler-users, not for the machines that it runs on, and if we manage to find an abstract definition of strings that works for Wasm-targetting toolchains, we should think about adding it.

The second objection is trickier. When you compile to Wasm, you need a good model of what the performance of the Wasm code that you emit will be. Different Wasm implementations may use different stringref representations; requesting a UTF-16 view on a string that is already UTF-16 will be cheaper than doing so on a string that is UTF-8. In the worst case, requesting a UTF-16 view on a UTF-8 string is a linear operation on one system but constant-time on another, which in a loop over string contents makes the former system quadratic: a real performance failure that we need to design around.

The stringref proposal tries to reify as much of the cost model as possible with its “view” abstraction; the compiler can reason that any cost may incur then rather than when accessing a view. But, this abstraction can leak, from a performance perspective. What to do?

I think that if we look back on what the expected outcome of the JS-strings-for-Java proposal is, I believe that if Wasm succeeds as a target for Java, we will probably already end up with WTF-16 everywhere. We might as well admit this, I think, and if we do, then this objection goes away. Likewise on the Web I see UTF-8 as being potentially advantageous in the medium-long term for JavaScript, and certainly better for other languages, and so I expect JS implementations to also grow support for fast UTF-8.

### i’m on a horse

I may be off in some of my predictions about where things will go, so who knows. In the meantime, in the time that it takes other people to reach the same conclusions, stringref is in a kind of hiatus.

The [Scheme-to-Wasm compiler](https://gitlab.com/spritely/guile-hoot/) that I work on does still emit stringref, but it is purely a toolchain concept now: we have a [post-pass that lowers stringref to
WTF-8](https://gitlab.com/spritely/guile-hoot/-/blob/main/module/wasm/lower-stringrefs.scm) via `(array i8)`, and which emits calls to host-supplied conversion routines when passing these strings to and from the host. When compiling to Hoot’s built-in Wasm virtual machine, we can leave stringref in, instead of lowering it down, resulting in more efficient interoperation with the host Guile than if we had to bounce through byte arrays.

So, we wait for now. Not such a bad situation, at least we have GC coming soon to all the browsers. appy hacking to all my stringfolk, and until next time!

## related articles

- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [wastrel, a profligate implementation of webassembly](/archives/2025/10/30/wastrel-a-profligate-implementation-of-webassembly)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [a world to win: webassembly for the rest of us](/archives/2023/03/20/a-world-to-win-webassembly-for-the-rest-of-us)

### 4 responses

1. Hubert says:[19 October 2023 1:42 PM](#0a7586157856508170a1963dea8c0ec0533e3a1a)

   Maybe we should differentiate between strings and hieroglyphs?

   A string would be a sequence of 16-bit characters (chars being a base data type, of fixed size).

   Hieroglyphs would be sequences of grapheme clusters of arbitrary individual sizes.

2. Arthur A. Gleckler says:[19 October 2023 3:35 PM](#0e4c3a1976e1bd4404a577fa2a9215da75f50582)

3. Max says:[19 October 2023 11:40 PM](#07c3e160f302c6468d445ab786ec4b6d6ff09df4)

   With that detail out of the way, say you are compiling Python to Wasm/GC. Python’s language reference says: “A string is a sequence of values that represent Unicode code points. All the code points in the range U+0000 - U+10FFFF can be represented in a string.” This corresponds to the domain of JavaScript’s strings; great!

   This isn’t quite correct. JS strings don’t provide a way to encode arbitrary code points as Python 3 does.

   In Python “ud83dudca9” represents an ill-formed string distinct from “U0001f4a9” or “💩” (since the denoted code points are not USVs, and the string units are not interpreted as UTF-16 code units).

   In JS, “ud83dudca9” is the same as “u{1f4a9}” or “💩”, which consists of 2 UTF-16 code units.

4. Max says:[19 October 2023 11:46 PM](#91e2ca4dc58770f5851e0ce845b820384d798fd8)

   Corrections to above message, since some characters are removed during submission:

   The first line is meant to be a quote within the page (“greater-than sign” removed).

   “ud83dudca9”, “U0001f4a9” and “u{1f4a9}” are all meant to have backslashes before the ‘u’ or ‘U’ characters.

Comments are closed.
