---
title: bigint shipping in firefox! — wingolog
url: https://wingolog.org/archives/2019/05/23/bigint-shipping-in-firefox
published: "2019-05-23T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2019/05/23/bigint-shipping-in-firefox
---

# bigint shipping in firefox! — wingolog

## [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)

23 May 2019 12:13 PM

- [compilers](/tags/compilers)
- [bigint](/tags/bigint)
- [bignums](/tags/bignums)
- [javascript](/tags/javascript)
- [firefox](/tags/firefox)
- [spidermonkey](/tags/spidermonkey)
- [v8](/tags/v8)
- [igalia](/tags/igalia)
- [bloomberg](/tags/bloomberg)

I am delighted to share with folks the results of a project I have been helping out on for the last few months: implementation of "BigInt" in Firefox, which is finally shipping in Firefox 68 (beta).

**what's a bigint?**

BigInts are a new kind of JavaScript primitive value, like numbers or strings. A BigInt is a true integer: it can take on the value of any finite integer (subject to some arbitrarily large implementation-defined limits, such as the amount of memory in your machine). This contrasts with JavaScript number values, which have the well-known property of [only being able to precisely represent integers between -253 and 253](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER).

BigInts are written like "normal" integers, but with an `n` suffix:

```
var a = 1n;
var b = a + 42n;
b << 64n
// result: 793209995169510719488n

```

With the bigint proposal, the usual mathematical operations ( `+`, `-`, `*`, `/`, `%`, `<<`, `>>`, `**`, and the comparison operators) are extended to operate on bigint values. As a new kind of primitive value, bigint values have their own `typeof`:

```
typeof 1n
// result: 'bigint'

```

Besides allowing for more kinds of math to be easily and efficiently expressed, BigInt also allows for better interoperability with systems that use 64-bit numbers, such as ["inodes" in file systems](https://github.com/nodejs/node/pull/20220), [WebAssembly i64 values](https://sauleau.com/notes/wasm-bigint.html), [high-precision timers](https://nodejs.org/api/process.html#process_process_hrtime_bigint), and so on.

You can [read more about the BigInt feature over on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt), as usual. You might also like this [short article on BigInt basics](https://developers.google.com/web/updates/2018/05/bigint) that V8 engineer Mathias Bynens wrote when Chrome shipped support for BigInt last year. There is an accompanying [language implementation](https://v8.dev/blog/bigint) article as well, for those of y'all that enjoy the nitties and the gritties.

**can i ship it?**

To try out BigInt in Firefox, simply download a copy of [Firefox Beta](https://www.mozilla.org/en-US/firefox/channel/desktop/). This version of Firefox will be fully released to the public in a few weeks, on July 9th. If you're reading this in the future, I'm talking about Firefox 68.

BigInt is also shipping already in V8 and Chrome, and my colleague Caio Lima has an project in progress to implement it in JavaScriptCore / WebKit / Safari. Depending on your target audience, BigInt might be deployable already!

**thanks**

I must mention that my role in the BigInt work was relatively small; my Igalia colleague Robin Templeton did the bulk of the BigInt implementation work in Firefox, so large ups to them. Hearty thanks also to Mozilla's Jan de Mooij and Jeff Walden for their patient and detailed code reviews.

Thanks as well to the V8 engineers for their open source implementation of BigInt fundamental algorithms, as we used many of them in Firefox.

Finally, I need to make one big thank-you, and I hope that you will join me in expressing it. The road to ship anything in a web browser is long; besides the "simple matter of programming" that it is to implement a feature, you need a [specification with buy-in from implementors and web standards people](https://github.com/tc39/proposal-bigint/blob/master/README.md), you need a good working relationship with a browser vendor, you need willing technical reviewers, you need to follow up on the inevitable security bugs that any browser change causes, and all of this takes time. It's all predicated on having the backing of an organization that's foresighted enough to invest in this kind of long-term, high-reward platform engineering.

In that regard I think all people that work on the web platform should send a big shout-out to [Tech at Bloomberg](https://techatbloomberg.com) for making BigInt possible by underwriting all of [Igalia](https://igalia.com/)'s work in this area. Thank you, Bloomberg, and happy hacking!

## related articles

- [understanding webassembly code generation throughput](/archives/2020/04/14/understanding-webassembly-code-generation-throughput)
- [heap object representation in spidermonkey](/archives/2018/10/11/heap-object-representation-in-spidermonkey)
- [generators in firefox now twenty-two times faster](/archives/2014/11/14/generators-in-firefox-now-twenty-two-times-faster)
- [es6 generator and array comprehensions in spidermonkey](/archives/2014/03/07/es6-generator-and-array-comprehensions-in-spidermonkey)
- [multi-value webassembly in firefox: a binary interface](/archives/2020/04/08/multi-value-webassembly-in-firefox-a-binary-interface)
- [multi-value webassembly in firefox: from 1 to n](/archives/2020/04/03/multi-value-webassembly-in-firefox-from-1-to-n)

### 2 responses

1. Marc says:[26 May 2019 8:58 AM](#f807b4f7cad6e624322fdd6596366f5d211b4334)

   Why is it 2^53? I would expect 2^63 (64 bit and one bit signed/unsigned).

2. Craig says:[26 May 2019 12:46 PM](#e7421568616f418e7aab48c8559238c0a3cbbce2)

   @Marc because JavaScript numbers are represented as doubles (64 bit floating-point numbers), which only have 52 bits of precision. The exponent and the sign bit occupy the other 12.

   It was the same way in Lua, until they added a split float/integer representation in 5.3. A lot of times all you really need is the full 64 bits of an integer instead of BigInts.

Comments are closed.
