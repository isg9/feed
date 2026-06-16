---
title: 'Protobuf Tip #3: Enum Names Need Prefixes'
url: /2025/04/22/protobuf-tip-3
published: "2025-04-22T00:00:00Z"
feed: mcyoung
guid: /2025/04/22/protobuf-tip-3
---

# Protobuf Tip #3: Enum Names Need Prefixes

*Smart people learn from their mistakes. But the real sharp ones learn from the* *mistakes of others. –Brandon Mull*

TL;DR: `enum` s inherit some unfortunate behaviors from C++. Use the Buf lint rules [`ENUM_VALUE_PREFIX`](https://buf.build/docs/lint/rules/#enum_value_prefix) and [`ENUM_ZERO_VALUE_SUFFIX`](https://buf.build/docs/lint/rules/#enum_zero_value_suffix) to avoid this problem (they’re part of the `DEFAULT` category).

> [note](#note:1)
>
> I’m editing a series of best practice pieces on Protobuf, a language that I work on which has lots of evil corner-cases.These are shorter than what I typically post here, but I think it fits with what you, dear reader, come to this blog for. These tips are also posted on the [buf.build blog](https://buf.build/blog/totw-3-enum-names-need-prefixes).

## [C++-Style Enums](\#c-style-enums)

Protobuf’s `enum` s define data types that represent a small set of valid values. For example, `google.rpc.Code` lists status codes used by various RPC frameworks, such as GRPC. Under the hood, every `enum` is just an `int32`  on the wire, although codegen backends will generate custom types and constants for the enum to make it easier to use.

Unfortunately, `enum` s were originally designed to match C++ enums exactly, and they inadvertently replicate many of those behaviors.

If you look at the source for `google.rpc.Code`, and compare it to, say, `google.protobuf.FieldDescriptorProto.Type`, you will notice a subtle difference:

```protobuf highlight
package google.rpc;
enum Code {
  OK = 0;
  CANCELLED = 1;
  UNKNOWN = 2;
  // ...
}

package google.protobuf;
message FieldDescriptorProto {
  enum Type {
    // 0 is reserved for errors.
    TYPE_DOUBLE = 1;
    TYPE_FLOAT = 2;
    TYPE_INT64 = 3;
    // ...
  }
}

```

[Protobuf](#code:1)

`FieldDescriptorProto.Type` has values starting with `TYPE_`, but `Code` ’s values don’t have a `CODE_` prefix.  This is because the fully-qualified names (FQN) of an enum value *don’t include the name of the enum.* That is, `TYPE_DOUBLE` actually refers to `google.protobuf.FieldDescriptorProto.TYPE_DOUBLE`. Thus, `OK` is not `google.rpc.Code.OK`, but `google.rpc.OK`.

This is because it matches the behavior of unscoped C++ enums. C++ is the “reference” implementation, so the language often bends for the sake of the C++ backend.

When generating code, `protoc`’s C++ backend emits the above as follows:

```cpp highlight
namespace google::rpc {
enum Code {
  OK = 0,
  CANCELLED = 1,
  UNSPECIFIED = 2,
  // ...
};
}

namespace google::protobuf {
class FieldDescriptorProto final {
 public:
  enum Type {
   TYPE_DOUBLE = 1;
   TYPE_FLOAT = 2;
   // ...
  };
};
}

```

[C++](#code:2)

And in C++, `enum` s don’t scope their enumerators: you write `google::rpc::OK`, NOT `google::rpc::Code::OK`.

If you know C++, you might be thinking, “why didn’t they use `enum class`?!”? Enums were added in `proto2`, which was developed around 2007-2008, but Google didn’t start using C++11, which introduced `enum class` , until much, much later.

Now, if you’re a Go or Java programmer, you’re probably wondering why you even care about C++. Both Go and Java do scope enum values to the enum type (although Go does it in a somewhat grody way: `rpcpb.Code_OK`).

Unfortunately, this affects name collision detection in Protobuf. You can’t write the following code:

```protobuf highlight
package myapi.v1;

enum Stoplight {
  UNSPECIFIED = 0;
  RED = 1;
  YELLOW = 2;
  GREEN = 3;
}

enum Speed {
  UNSPECIFIED = 0;
  SLOW = 1;
  FAST = 2;
}

```

[Protobuf](#code:3)

Because the enum name is not part of the FQN for an enum value, both `UNSPECIFIED` s here have the FQN `myapi.v1.UNSPECIFIED`, so Protobuf complains about duplicate symbols.

Thus, the convention we see in `FieldDescriptorProto.Type`:

```protobuf highlight
package myapi.v1;

enum Stoplight {
  STOPLIGHT_UNSPECIFIED = 0;
  STOPLIGHT_RED = 1;
  STOPLIGHT_YELLOW = 2;
  STOPLIGHT_GREEN = 3;
}

enum Speed {
  SPEED_UNSPECIFIED = 0;
  SPEED_SLOW = 1;
  SPEED_FAST = 2;
}

```

[Protobuf](#code:4)

Buf provides a lint rule to enforce this convention: [`ENUM_VALUE_PREFIX`](https://buf.build/docs/lint/rules/#enum_value_prefix). Even though you might think that an enum name will be unique, because top-level enums bleed their names into the containing package, the problem spreads across packages!

## [Zero Values](\#zero-values)

`proto3` relies heavily on the concept of “zero values” – all non-message fields that are neither `repeated` nor `optional` are implicitly zero if they are not present. Thus, `proto3` requires that enums specify a value equal to zero.

By convention, this value shouldn’t be a specific value of the enum, but rather a value representing that no value is specified. [`ENUM_ZERO_VALUE_SUFFIX`](https://buf.build/docs/lint/rules/#enum_zero_value_suffix) enforces this, with a default of `_UNSPECIFIED`. Of course, there are situations where this might not make sense for you, and a suffix like `_ZERO` or `_UNKNOWN` might make more sense.

It may be tempting to have a specific “good default” value for the zero value. Beware though, because that choice is forever. Picking a generic “unknown” as the default reduces the chance you’ll burn yourself.

## [Why Don’t All of Google’s Protobuf Files Do This?](\#why-dont-all-of-googles-protobuf-files-do-this)

Name prefixes and zero values also teach us an important lesson: because Protobuf names are forever, it’s really hard to fix style mistakes, especially as we collectively get better at using Protobuf.

`google.rpc.Code` is intended to be source-compatible with very old existing C++ code, so it throws caution to the wind. `FieldDescriptorProto.Type` doesn’t have a zero value because in `proto2` , which doesn’t have zero value footguns in its wire format, you don’t need to worry about that. The lesson isn’t just to use Buf’s linter to try to avoid some of the known pitfalls, but also to remember that even APIs designed by the authors of the language make unfixable mistakes, so unlike other programming languages, imitating “existing practice” isn’t always the best strategy.
