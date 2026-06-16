---
title: 'Protobuf Tip #7: Scoping It Out'
url: /2025/06/03/protobuf-tip-7
published: "2025-06-03T00:00:00Z"
feed: mcyoung
guid: /2025/06/03/protobuf-tip-7
---

# Protobuf Tip #7: Scoping It Out

*You’d need a very specialized electron microscope to get down to the level to* *actually see a single strand of DNA. – Craig Venter*

TL;DR: `buf convert` is a powerful tool for examining wire format dumps, by converting them to JSON and using existing JSON analysis tooling. `protoscope` can be used for lower-level analysis, such debugging messages that have been corrupted.

> [note](#note:1)
>
> I’m editing a series of best practice pieces on Protobuf, a language that I work on which has lots of evil corner-cases.These are shorter than what I typically post here, but I think it fits with what you, dear reader, come to this blog for. These tips are also posted on the [buf.build blog](https://buf.build/blog/totw-7-scoping-it-out).

## [JSON from Protobuf?](\#json-from-protobuf)

JSON’s human-readable syntax is a big reason why it’s so popular, possibly second only to built-in support in browsers and many languages. It’s easy to examine any JSON document using tools like online prettifiers and the inimitable `jq`.

But Protobuf is a binary format! This means that you can’t easily use `jq` -like tools with it…or can you?

## [Transcoding with `buf convert`](\#transcoding-with-buf-convert)

The Buf CLI offers a utility for transcoding messages between the three Protobuf encoding formats: the wire format, JSON, and textproto; it also supports YAML. This is `buf convert`, and it’s very powerful.

To perform a conversion, we need four inputs:

1. A Protobuf source to get types out of. This can be a local `.proto` file, an encoded `FileDescriptorSet` , or a remote BSR module.
   - If not provided, but run in a directory that is within a local Buf module, that module will be used as the Protobuf type source.
2. The name of the top-level type for the message we want to transcode, via the `--type` flag.
3. The input message, via the `--from` flag.
4. A location to output to, via the `--to` flag.

`buf convert` supports input and output redirection, making it usable as part of a shell pipeline. For example, consider the following Protobuf code in our local Buf module:

```protobuf highlight
// my_api.proto
syntax = "proto3";
package my.api.v1;

message Cart {
  int32 user_id = 1;
  repeated Order orders = 2;
}

message Order {
  fixed64 sku = 1;
  string sku_name = 2;
  int64 count = 3;
}

```

[Protobuf](#code:1)

Then, let’s say we’ve dumped a message of type `my.api.v1.Cart` from a service to debug it. And let’s say…well—you can’t just `cat` it.

```console highlight
$ cat dump.pb | xxd -ps
08a946121b097ac8e80400000000120e76616375756d20636c65616e6572
18011220096709b519000000001213686570612066696c7465722c203220
7061636b1806122c093aa8188900000000121f69736f70726f70796c2061
6c636f686f6c203730252c20312067616c6c6f6e1802

```

[Console](#code:2)

However, we can use `buf convert` to turn it into some nice JSON. We can then pipe it into `jq` to format it.

```console highlight
$ buf convert --type my.api.v1.Cart --from dump.pb --to -#format=json | jq
{
  "userId": 9001,
  "orders": [
    {
      "sku": "82364538",
      "skuName": "vacuum cleaner",
      "count": "1"
    },
    {
      "sku": "431294823",
      "skuName": "hepa filter, 2 pack",
      "count": "6"
    },
    {
	    "sku": "2300094522",
      "skuName": "isopropyl alcohol 70%, 1 gallon",
      "count": "2"
    }
  ]
}

```

[Console](#code:3)

Now you have the full expressivity of `jq` at your disposal. For example, we could pull out the user ID for the cart:

```console highlight
$ function buf-jq() { buf convert --type $1 --from $2 --to -#format=json | jq $3 }
$ buf-jq my.api.v1.Cart dump.pb '.userId'
9001

```

[Console](#code:4)

Or we can extract all of the SKUs that appear in the cart:

```console highlight
$ buf-jq my.api.v1.Cart dump.pb '[.orders[].sku]'
[
  "82364538",
  "431294823",
  "2300094522"
]

```

[Console](#code:5)

Or we could try calculating how many items are in the cart, total:

```console highlight
$ buf-jq my.api.v1.Cart dump.pb '[.orders[].count] | add'
"162"

```

[Console](#code:6)

Wait. That’s wrong. The answer should be `9`. This illustrates one pitfall to keep in mind when using `jq` with Protobuf. Protobuf will *sometimes* serialize numbers as quoted strings (the C++ reference implementation only does this when they’re integers outside of the IEEE754 representable range, but Go is somewhat lazier, and does it for all 64-bit values).

> You can test if an `x int64` is in the representable float range with this very simple check: `int64(float64(x)) == x)`. See [https://go.dev/play/p/T81SbbFg3br](https://go.dev/play/p/T81SbbFg3br). The [equivalent version in C++](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/json/internal/unparser.cc#L96) is much more complicated.

This means we need to use the `tonumber` conversion function:

```console highlight
$ buf-jq my.api.v1.Cart dump.pb '[.orders[].count | tonumber] | add'
9

```

[Console](#code:7)

`jq` ’s whole deal is JSON, so it brings with it all of JSON’s pitfalls. This is notable for Protobuf when trying to do arithmetic on 64-bit values. As we saw above, Protobuf serializes integers outside of the 64-bit float representable range (and in some runtimes, some integers inside it).

For example, if you have a `repeated int64` that you want to sum over, it may produce incorrect answers due to floating-point rounding. For notes on conversions in `jq`, see [https://jqlang.org/manual/#identity](https://jqlang.org/manual/#identity).

## [Disassembling with `protoscope`](\#disassembling-with-protoscope)

[`protoscope`](https://github.com/protocolbuffers/protoscope) is a tool provided by the Protobuf team (which I originally wrote!) for decoding arbitrary data as if it were encoded in the Protobuf wire format. This process is called *disassembly*. It’s designed to work without a schema available, although it doesn’t produce especially clean output.

```console highlight
$ go install github.com/protocolbuffers/protoscope/cmd/protoscope...@latest
$ protoscope dump.pb
1: 9001
2: {
  1: 82364538i64
  2: {"vacuum cleaner"}
  3: 1
}
2: {
  1: 431294823i64
  2: {
    13: 101
    14: 97
    4: 102
    13: 1.3518748403899336e-153   # 0x2032202c7265746ci64
    14: 97
    12:SGROUP
    13:SGROUP
  }
  3: 6
}
2: {
  1: 2300094522i64
  2: {"isopropyl alcohol 70%, 1 gallon"}
  3: 2
}

```

[Console](#code:8)

The field names are gone; only field numbers are shown. This example also reveals an especially glaring limitation of `protoscope`, which is that it can’t tell the difference between string and message fields, so it guesses according to some heuristics. For the first and third elements it was able to grok them as strings, but for `orders[1].sku_name`, it incorrectly guessed it was a message and produced garbage.

The tradeoff is that not only does `protoscope` not need a schema, it also tolerates almost any error, making it possible to analyze messages that have been partly corrupted. If we flip a random bit somewhere in `orders[0]`, disassembling the message still succeeds:

```console highlight
$ protoscope dump.pb
1: 9001
2: {`0f7ac8e80400000000120e76616375756d20636c65616e65721801`}
2: {
  1: 431294823i64
  2: {
    13: 101
    14: 97
    4: 102
    13: 1.3518748403899336e-153   # 0x2032202c7265746ci64
    14: 97
    12:SGROUP
    13:SGROUP
  }
  3: 6
}
2: {
  1: 2300094522i64
  2: {"isopropyl alcohol 70%, 1 gallon"}
  3: 2
}

```

[Console](#code:9)

Although `protoscope` did give up on disassembling the corrupted submessage, it still made it through the rest of the dump.

Like `buf convert`, we can give `protoscope` a `FileDescriptorSet` to make its heuristic a little smarter.

```console highlight
$ protoscope \
  --descriptor-set <(buf build -o -) \
  --message-type my.api.v1.Cart \
  --print-field-names \
  dump.pb
1: 9001                   # user_id
2: {                      # orders
  1: 82364538i64          # sku
  2: {"vacuum cleaner"}   # sku_name
  3: 1                    # count
}
2: {                          # orders
  1: 431294823i64             # sku
  2: {"hepa filter, 2 pack"}  # sku_name
  3: 6                        # count
}
2: {                                      # orders
  1: 2300094522i64                        # sku
  2: {"isopropyl alcohol 70%, 1 gallon"}  # sku_name
  3: 2                                    # count
}

```

[Console](#code:10)

Not only is the second order decoded correctly now, but `protoscope` shows the name of each field (via `--print-field-names` ). In this mode, `protoscope` still decodes partially-valid messages.

`protoscope` also provides a number of other flags for customizing its heuristic in the absence of a `FileDescriporSet`. This enables it to be used as a forensic tool for debugging messy data corruption bugs.
