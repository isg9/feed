---
title: 'Protobuf Tip #2: Compress Your Protos!'
url: /2025/04/15/protobuf-tip-2
published: "2025-04-15T00:00:00Z"
feed: mcyoung
guid: /2025/04/15/protobuf-tip-2
---

# Protobuf Tip #2: Compress Your Protos!

*As a matter of fact, when compression technology came along, we thought the* *future in 1996 was about voice. We got it wrong. It is about voice, video, and* *data, and that is what we have today on these cell phones. –Steve Buyer*

TL;DR: Compression is everywhere: CDNs, HTTP servers, even in RPC frameworks like Connect. This pervasiveness means that wire size tradeoffs matter less than they used to twenty years ago, when Protobuf was designed.

> [note](#note:1)
>
> I’m editing a series of best practice pieces on Protobuf, a language that I work on which has lots of evil corner-cases.These are shorter than what I typically post here, but I think it fits with what you, dear reader, come to this blog for. These tips are also posted on the [buf.build blog](https://buf.build/blog/totw-2-compress-protos).

## [Varints from 1998](\#varints-from-1998)

Protobuf’s wire format is intended to be relatively small. It makes use of *variable-width integers* so that smaller values take up less space on the wire. Fixed width integers might be larger on the wire, but often have faster decoding times.

But what if I told you that doesn’t matter?

See, most internet traffic is compressed. Bandwidth is precious, and CDN operators don’t want to waste time sending big blobs full of zeros. There are many compression algorithms available, but the state of the art for HTTP requests (which dominates much of global internet traffic) is [Brotli](https://en.wikipedia.org/wiki/Brotli), an algorithm developed at Google in 2013 and standardized in IETF [RFC7932](https://datatracker.ietf.org/doc/html/rfc7932) in 2016. There is a very good chance that this article was delivered to your web browser as a Brotli-compressed blob.

## [Using Compression](\#using-compression)

How compression is applied in your case will vary, but both Connect RPC and gRPC support native compression. For example, Connect has an API for injecting compression providers: [https://pkg.go.dev/connectrpc.com/connect#WithCompression](https://pkg.go.dev/connectrpc.com/connect#WithCompression).

Connect uses `gzip` by default, which uses the DEFLATE compression algorithm. Providing your own compression algorithm (such as Brotli) is pretty simple, as shown by [this third-party package](https://github.com/mattrobenolt/connect-brotli/blob/921ee0236bcd2d66827590c6890bb850e56516ad/connect_brotli.go).

Other services may compress for you transparently. Any competent CDN will likely use Brotli (or `gzip` or `zlib`, but probably Brotli) to compress any files it serves for you. (In fact, JavaScript and HTML minimization can often be rendered irrelevant by HTTP compression, too.)

It’s important to remember that Protobuf predates pervasive compression: if it didn’t, it would almost certainly not use variable-width integers for anything. It only uses them because they offer a primitive form of compression in exchange for being slower to decode. If that tradeoff was eliminated, Protobuf would almost certainly only use fixed-width integers on the wire.

## [How Good Is It Really?](\#how-good-is-it-really)

Let’s do some apples-to-apples comparisons. Consider the following Protobuf type.

```protobuf highlight
message Foo {
	repeated int32 varints = 1;
	repeated sfixed32 fixeds = 2;
}

```

[Protobuf](#code:1)

There are two fields that contain essentially the same data, which can be encoded in four different ways: as old-style repeated fields, as packed fields, and the integers can be encoded as varints or fixed32 values.

Using [Protoscope](https://github.com/protocolbuffers/protoscope), we can create some data that exercises these four cases:

```protoscope highlight
# a.pb, repeated varint
1: 1
1: 2
1: 3
# ...

# b.pb, packed varint
1: {
  1 2 3
  # ...
}

# c.pb, repeated fixed32
2: 1i32
2: 2i32
2: 3i32

# d.pb, packed fixed32
2: {
  1i32 2i32 3i32
  # ...
}

```

[Protoscope](#code:2)

Each blob contains the integers from 1 to 1000 encoded in different ways. I’ll compress each one using gzip, zlib, and Brotli, using their default compression levels, and arrange their sizes, in bytes, in the table below.FileUncompressed`gzip` (DEFLATE)`zlib`Brotlia.pb2875189918781094b.pb187715341524885c.pb5005157715671140d.pb4007144019161140

Compression achieves incredible results: Brotli manages to get all of the files down to around 1.1 kB, except for the packed varints, which it gets about 250 bytes smaller! Of course, that’s only because most of the values in that repeated field are small. If the values range from 100000 to 101000, b.pb and d.pb are 3006 and 4007 bytes respectively (see that d.pb’s size is unchanged!), but when compressed with brotli, the lead for b.pb starts to disappear: 1039 bytes vs. 1163 bytes. Now it’s only 120 bytes smaller.

## [Are Varints Still Better?](\#are-varints-still-better)

Applying compression can often have similar results to replacing everything with varints, but not exactly: using a varint will likely always be slightly smaller, at least when using state-of-the-art compression like Brotli. But you can pretty much always assume you *will* be using compression, such as to compress HTTP headers and other ancillary content in your request. Compression is generic and highly optimized—it applies to all data, regardless of schema, and is often far more optimized than application-level codecs like those in a Protobuf library.

Not to mention, you should definitely be compressing any large data blobs you’re storing on disk, too!

As a result, you can usually disregard many encoded size concerns when making tradeoffs in designing a Protobuf type. Fixed integer types will decode faster, so if decoding speed is important to you, and you’re worried about the size on the wire, don’t. It’s almost certainly already taken care of at a different layer of the stack.
