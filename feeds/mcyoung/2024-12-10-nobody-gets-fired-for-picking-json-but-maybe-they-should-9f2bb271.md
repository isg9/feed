---
title: Nobody Gets Fired for Picking JSON, but Maybe They Should?
url: /2024/12/10/json-sucks
published: "2024-12-10T00:00:00Z"
feed: mcyoung
guid: /2024/12/10/json-sucks
---

# Nobody Gets Fired for Picking JSON, but Maybe They Should?

JSON is extremely popular but deeply flawed. This article discusses the details of JSON’s design, how it’s used (and misused), and how seemingly helpful “human readability” features cause headaches instead. Crucially, you rarely find JSON-based tools (except dedicated tools like `jq`) that can safely handle arbitrary JSON documents without a schema—common corner cases can lead to data corruption!

## [What is JSON?](\#what-is-json)

JSON is famously simple. In fact, you can [fit the entire grammar on the back of a business card](https://www.flickr.com/photos/equanimity/3763158824/in/photostream/). It’s so omnipresent in REST APIs that you might assume you already know JSON quite well. It has decimal numbers, quoted strings, arrays with square brackets, and key-value maps (called “objects”) with curly braces. A JSON document consists of any of these constructs: `null`, `42`, and `{"foo":"bar"}` are all valid JSON documents.

However, the formal definition of JSON is quite complicated. JSON is defined by the IETF document [RFC8259](https://datatracker.ietf.org/doc/html/rfc8259) (if you don’t know what the IETF is, it’s the standards body for Internet protocols). However, it’s *also* normatively defined by [ECMA-404](https://ecma-international.org/publications-and-standards/standards/ecma-404/), which is from ECMA, the standards body that defines JavaScript[1](#fn:1).

JavaScript? Yes, JSON (JavaScript Object Notation) is closely linked with JavaScript and is, in fact, (almost) a subset of it. While JSON’s JavaScript ancestry is the main source of its quirks, several other poor design decisions add additional unforced errors.

However, the biggest problem with JSON isn’t any specific design decision but rather the incredible diversity of parser behavior and non-conformance across and within language ecosystems. RFC8259 goes out of its way to call this out:

> [reference](#ref:1)
>
> Note, however, that ECMA-404 allows several practices that this specification recommends avoiding in the interests of maximal interoperability.

The RFC makes many observations regarding interoperability elsewhere in the document. Probably the most glaring—and terrifying—is how numbers work.

## [Everything is Implementation-Defined](\#everything-is-implementation-defined)

JSON numbers are encoded in decimal, with an optional minus sign, a fractional part after a decimal point, and a scientific notation exponent. This is similar to how many programming languages define their own numeric literals.

Presumably, JSON numbers are meant to be floats, right?

Wrong.

RFC8259 reveals that the answer is, unfortunately, “whatever you want."

> This specification allows implementations to set limits on the range and precision of numbers accepted. Since software that implements IEEE 754 binary64 (double precision) numbers is generally available and widely used, good interoperability can be achieved by implementations that expect no more precision or range than these provide, in the sense that implementations will approximate JSON numbers within the expected precision.

`binary64` is the “standards-ese” name for the type usually known as `double` or `float64`. Floats have great dynamic range but often can’t represent exact values. For example, `1.1` isn’t representable as a float because all floats are fractions of the form `n / 2^m` for integers `n` and `m`, but `1.1 = 11/10`, which has a factor of 5 in its denominator. The closest `float64` value is

```highlight
2476979795053773 / 2^51 = 1.100000000000000088817841970012523233890533447265625

```

[Plaintext](#code:1)

Of course, you might think to declare “all JSON values map to their closest `float64` value”. Unfortunately, this value might not be unique. For example, the value `900000000000.00006103515625` isn’t representable as a `float64`, and it’s precisely between two exact `float64` values. Depending on the rounding mode, this rounds to either or `900000000000` or `900000000000.0001220703125` .

IEEE 754 recommends “round ties to even” as the default rounding mode, so for almost all software, the result is `900000000000`. But remember, floating-point state is a global variable implemented in hardware, and might just happen to be clobbered by some dependency that calls `fesetround()` or a similar system function.

## [Data Loss! Data Loss!](\#data-loss-data-loss)

You’re probably thinking, “I don’t care about such fussy precision stuff. None of my numbers have any fractional parts—and there is where you would be wrong. The `n` part of `n / 2^m` only has 53 bits available, but `int64` values fall outside of that range. This means that for very large 64-bit integers, such as randomly generated IDs, a JSON parser that converts integers into floats results in *data loss.* Go’s `encoding/json` package does this, for example.

How often does this actually happen for randomly-generated numbers? We can do a little Monte Carlo simulation to find out.

```go highlight
package main

import (
	"fmt"
	"math"
	"math/big"
	"math/rand"
)

const trials = 5_000_000
func main() {
	var misses int
	var err big.Float
	for range trials {
		x := int64(rand.Uint64())
		y := int64(float64(x)) // Round-trip through binary64.
		if x != y {
			misses++
			err.Add(&err, big.NewFloat(math.Abs(float64(x - y))))
		}
	}

	err.Quo(&err, big.NewFloat(trials))
	fmt.Printf("misses: %d/%d, avg: %f", misses, trials, &err)
}

// Output:
// misses: 4970572/5000000, avg: 170.638499

```

[Go](#code:2)

It turns out that almost all randomly distributed `int64` values are affected by round-trip data loss. Roughly, the only numbers that are safe are those with at most 16 digits (although not exactly: 9,999,999,999,999,999, for example, gets rounded up to a nice round 10 quadrillion).

How does this affect you? Suppose you have a JSON document somewhere that includes a user ID and a transcript of their private messages with another user. Data loss due to rounding would result in the wrong user ID being associated with the private messages, which could result in leaking PII or incorrect management of privacy consent (such as GDPR requirements).

This isn’t just about *your* user IDs, mind you. Plenty of other vendors’ IDs are nice big integers, which the JSON grammar can technically accommodate and which random tools will mangle. Some examples:

- License keys: for example, Adobe uses 24 digits for [their serial numbers](https://helpx.adobe.com/x-productkb/global/invalid-revoked-serial-numbers.html), which may be tempting to store as an integer.

- Barcode IDs like the unique serial numbers of medical devices, [which are tightly regulated](https://www.fda.gov/medical-devices/unique-device-identification-system-udi-system/udi-basics).

- Visa and Mastercard credit card numbers *happen* to fit in the “safe” range for `binary64` , which may lull you into a false sense of security, since they’re so common. But not all credit cards have 16 digit numbers: [some now support 19](https://en.wikipedia.org/wiki/Payment_card_number#Structure).

These are pretty bad compliance consequences purely due to a data serialization format.

This problem is avoidable with care. After all, Go can parse JSON into any arbitrary type using reflection. For example, if we replace the inner loop of the Monte Carlo simulation with something like the following:

```go highlight
for range trials {
	x := int64(rand.Uint64())
	var v struct{ N int64 }
	json.Unmarshal([]byte(fmt.Sprintf(`{"N":%d}`, x)), &v)
	y := v.N
	if x != y {
		// ...
	}
}

```

[Go](#code:3)

We suddenly see that `x == y` in every trial. This is because with type information, Go’s JSON library knows exactly what the target precision is. If we were parsing to an `any` instead of to a `struct { N int64 }`, we’d be in deep trouble: the outer object would be parsed into a `map[string]any`, and the `N` field would become a `float64`.

This means that your system probably can’t safely handle JSON documents with unknown fields. Tools like `jq` must be extremely careful about number handling to avoid data loss. This is an easy mistake for third-party tools to make.

But again, `float64` isn’t the standard—there is no standard. Some implementations might only have 32-bit floats available, making the problem worse. Some implementations might try to be clever, using a `float64` for fractional values and an `int64` for integer values; however, this still imposes arbitrary limits on the parsed values, potentially resulting in data loss.

Some implementations such as Python use bignums, so they appear not to have this problem. However, this can lead to a false sense of security where issues are not caught until it’s too late: some database now contains ostensibly valid but non-interoperable JSON.

Protobuf is forced to deal with this in a pretty non-portable way. To avoid data loss, large 64-bit integers are serialized as quoted strings when serializing to JSON. So, instead of writing `{"foo":6574404881820635023}`, it emits `{"foo":"6574404881820635023"}`. This solves the data loss issue but does not work with other JSON libraries such as Go’s, producing errors like this one:

```highlight
json: cannot unmarshal string into Go struct field .N of type int64

```

[Plaintext](#code:4)

### [Non-Finite Values](\#non-finite-values)

The special floating point values `Infinity`, `-Infinity`, and `NaN` are not representable: it’s the wild west as to what happens when you try to serialize the equivalent of `{x:1.0/0.0}`.

- Go refuses to serialize, citing `json: unsupported value: +Inf`.
- Protobuf serializes it as `{"x":"inf"}` (or should—it’s unclear which implementations get it right).
- JavaScript won’t even bother trying: `JSON.stringify({x:Infinity})` prints `{"x":null}.`
- Python is arguably the worst offender: `json.dumps({"x":float("inf")})` prints `{"x":Infinity}`, which isn’t even valid JSON per RFC8259.

NaN is arguably an even worse offender, because the NaN payload (yes, [NaNs have a special payload](https://doc.rust-lang.org/std/primitive.f32.html#nan-bit-patterns)) is discarded when converting to `"nan"` or however your library represents it.

Does this affect you? Well, if you’re doing anything with floats, you’re one division-by-zero or overflow away from triggering serialization errors. At best, it’s “benign” data corruption (JavaScript). At worst, when the data is partially user-controlled, it might result in crashes or unparseable output, which is the making of a DoS vector.

In comparison, Protobuf serialization can’t fail except due to non-UTF-8 `string` fields or cyclic message references, both of which are comparatively unlikely to a NaN popping up in a calculation.

The upshot is that all the parsers end up parsing a bunch of crazy things for the special floating-point values over time because of [Postel’s law](https://en.wikipedia.org/wiki/Robustness_principle). RFC8259 makes no effort to provide suggestions for dealing with such real-world situations beyond “tough luck, not interoperable.”

## [Text Encodings and Invalid Unicode](\#text-encodings-and-invalid-unicode)

JSON strings are relatively tame, with some marked (but good) divergence from JavaScript. Specifically, JavaScript, being a language of a certain age (along with Java), uses UTF-16 as its Unicode text encoding. Most of the world has realized this is a bad idea (it doubles the size of ASCII text, which makes up almost all of Internet traffic), so JSON uses UTF-8 instead. RFC8259 actually specifies that the whole document MUST be encoded in UTF-8.

But when we go to read about Unicode characters in §8.2, we are disappointed: it merely says that it’s *really great* when all quoted strings consist entirely of Unicode characters, which means that unpaired surrogates are allowed. In effect, the spec merely requires that JSON strings be [WTF-8](https://en.wikipedia.org/wiki/UTF-8#Surrogates): UTF-8 that permits unpaired surrogates.

What’s an unpaired surrogate? It’s any encoded Unicode 32-bit value in the range `U+D800` to `U+DFFF` , which form a gap in the Unicode codepoint range. UTF-8’s variable-length integer encoding can encode them, but their presence in a bytestream makes it invalid UTF-8. WTF-8 is UTF-8 but permitting the appearance of these values.

So, who actually supports parsing (or serializing) these? Consider the document `{"x":"\udead"}`, which contains an unpaired surrogate, `U+DEAD`.

- Go gladly deserializes AND serializes it (Go’s strings are arbitrary byte strings, not UTF-8). However, Go serializes a non-UTF-8 string such as `"\xff"` as `"\ufffd"`, having replaced the invalid byte with a `U+FFFD` replacement character (this thing: �).

- Most Java parsers seem to follow the same behavior as Go, but there are many different parsers available, and we’ve already learned that different JSON parsers may behave differently.

- JavaScript and Python similarly gladly parse unpaired surrogates, but they also serialize them back without converting them into `U+FFFD`.

- Different Protobuf runtimes may not handle this identically, but the reference C++ implementation (whose JSON codec I wrote!) refuses to parse unpaired surrogates.

There are other surprising pitfalls around strings: are `"x"` and `“\x78"` the same string? RFC8259 feels the need to call out that they are, for the purposes of checking that object keys are equal. The fact that they feel the need to call it out indicates that this is also a source of potential problems.

## [Byte Strings](\#byte-strings)

What if I don’t want to send text? A common type of byte blob to send is a cryptographic hash that identifies a document in a content-addressed blobstore, or perhaps a digital signature (an encrypted hash). JSON has no native way of representing byte strings.

You could send a quoted string full of ASCII and `\xNN` escapes (for bytes which are not in the ASCII range), but this is wasteful in terms of bandwidth, and has serious interoperability problems (as noted above, Go actively destroys data in this case). You could also encode it as an array of JSON numbers, which is much worse for bandwidth and serialization speed.

What everyone winds up doing, one way or another, is to rely on base64 encoding. Protobuf, for example, encodes `bytes` fields into base64 strings in JSON. This has the unfortunate side-effect of defeating JSON’s human-readable property: if the blob contains mostly ASCII, a human reader can’t tell.

Because this isn’t part of JSON, virtually no JSON codec does this decoding for you, particularly because in a schema-less context, there’s nothing to distinguish a byte blob encoded with base64 from an actual textual string that *happens* to contain valid base64, such as an alphanumeric username.

Compared to other problems, this is more like a paper cut, but it’s unnecessary and adds complexity and interop problems. [By the way, did you know there are multiple incompatible Base64 alphabets?](https://en.wikipedia.org/wiki/Base64#Variants_summary_table)

# [Streaming Doesn’t Work](\#streaming-doesnt-work)

A less obvious problem with JSON is that it can’t be streamed. Almost all JSON documents are objects or arrays and are therefore *incomplete* until they reach the closing `}` or `]`, respectively. This means you can’t send a stream of JSON documents that form a part of a larger document without some additional protocol for combining them in post-processing.

[JSONL](https://jsonlines.org/) is the world’s silliest spec that “solves” this problem in the simplest way possible: a JSONL document is a sequence of JSON documents separated by newlines. JSONL *is* streamable, but because it’s done in the simplest way possible, it only supports streaming a giant array. You can’t, for example, stream an object field-by-field or stream an array within that object.

Protobuf doesn’t have this problem: in a nutshell, the Protobuf wire format is as if we removed the braces and brackets from the top-level array or object of a document, and made it so that values with the same key get merged. In the wire format, the equivalent of the JSONL document

```json highlight
{"foo": {"x": 1}, "bar": [5, 6]}
{"foo": {"y": 2}, "bar": [7, 8]}

```

[JSON](#code:5)

is automatically “merged” into the single document

```json highlight
{ "foo": { "x": 1, "y": 2 }, "bar": [5, 6] }

```

[JSON](#code:6)

This forms the basis of the “message merge” operation, which is intimately connected to how the wire format was designed. We’ll dive into this fundamental operation in a future article.

# [Canonicalization Leads to Data Loss](\#canonicalization-leads-to-data-loss)

Thanks to [RFC7519](https://datatracker.ietf.org/doc/html/rfc7519) and [RFC7515](https://datatracker.ietf.org/doc/html/rfc7515), which define JSON Web Tokens (JWT) and JSON Web Signatures (JWS), digitally signing JSON documents is a very common operation. However, digital signatures can only sign specific byte blobs and are sensitive to things that JSON isn’t, such as whitespace and key ordering.

This results in specifications like [RFC8785](https://datatracker.ietf.org/doc/html/rfc8785) for *canonicalization* of JSON documents. This introduces a new avenue by which existing JSON documents, which accidentally happen to contain non-interoperable (or, thanks to non-conforming implementations such as Python’s) invalid JSON that must be manipulated and reformatted by third-party tools. RFC8785 itself references ECMA-262 (the JavaScript standard) for how to serialize numbers, meaning that it’s *required* to induce data loss for 64-bit numerical values!

# [Is JSON Fixable?](\#is-json-fixable)

Plainly? No. JSON can’t be fixed because of how extremely popular it is. Common mistakes are baked into the format. Are comments allowed? Trailing commas? Number formats? Nobody knows!

What tools are touching your JSON? Are they aware of all of the rakes they can step on? Do they emit invalid JSON (like Python does)? How do you even begin to audit that?

Thankfully, you don’t have to use JSON. There are alternatives—BSON, UBJSON, MessagePack, and CBOR are just a few binary formats that try to replicate JSON’s data model. Unfortunately, many of them have their own problems.

Protobuf, however, has none of these problems, because it was *designed* to fulfill needs JSON couldn’t meet. Using a strongly-typed schema system, like Protobuf, makes all of these problems go away.

---

1. Of course, some wise guy will probably want to cite <json.org>. I should underscore: <json.org> is **NOT** a standard. It is **NOT** normative. the documents produced by the IETF and by ECMA, which are international standards organizations that represent the industry **ARE** normative. When a browser implementer wants to implement JSON to the letter, they go to ECMA, not to some dude’s 90’s ass website. [↩︎](#fnref:1)
