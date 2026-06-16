---
title: 'Protobuf Tip #6: The Subtle Dangers of Enum Aliases'
url: /2025/05/20/protobuf-tip-6
published: "2025-05-20T00:00:00Z"
feed: mcyoung
guid: /2025/05/20/protobuf-tip-6
---

# Protobuf Tip #6: The Subtle Dangers of Enum Aliases

*I’ve been very fortunate to dodge a nickname throughout my entire career. I’ve* *never had one. – Jimmie Johnson*

TL;DR: Enum values can have aliases. This feature is poorly designed and shouldn’t be used. The [`ENUM_NO_ALLOW_ALIAS`](https://buf.build/docs/lint/rules/#enum_no_allow_alias) Buf lint rule prevents you from using them by default.

> [note](#note:1)
>
> I’m editing a series of best practice pieces on Protobuf, a language that I work on which has lots of evil corner-cases.These are shorter than what I typically post here, but I think it fits with what you, dear reader, come to this blog for. These tips are also posted on the [buf.build blog](https://buf.build/blog/totw-5-avoid-import-public-weak).

## [Confusion and Breakage](\#confusion-and-breakage)

Protobuf permits multiple enum values to have the same number. Such enum values are said to be *aliases* of each other. Protobuf used to allow this by default, but now you have to set a special option, `allow_alias`, for the compiler to not reject it.

This can be used to effectively rename values without breaking existing code:

```protobuf highlight
package myapi.v1;

enum MyEnum {
  option allow_alias = true;
  MY_ENUM_UNSPECIFIED = 0;
  MY_ENUM_BAD = 1 [deprecated = true];
  MY_ENUM_MORE_SPECIFIC = 1;
}

```

[Protobuf](#code:1)

This works perfectly fine, and is fully wire-compatible! And unlike renaming a field (see [TotW #1](/2025/04/08/protobuf-tip-1)), it won’t result in source code breakages.

But if you use either reflection or JSON, or a runtime like Java that doesn’t cleanly allow enums with multiple names, you’ll be in for a nasty surprise.

For example, if you request an enum value from an enum using reflection, such as with `protoreflect.EnumValueDescriptors.ByNumber()`, the value you’ll get is the one that appears in the file lexically. In fact, both `myapipb.MyEnum_MY_ENUM_BAD.String()` and `myapipb.MyEnum_MY_ENUM_MORE_SPECIFIC.String()` return the same value, leading to potential confusion, as the old “bad” value will be used in printed output like logs.

You might think, “oh, I’ll switch the order of the aliases”. But that would be an *actual* wire format break. Not for the binary format, but for JSON. That’s because JSON preferentially stringifies enum values by using their declared name (if the value is in range). So, reordering the values means that what once serialized as `{"my_field": "MY_ENUM_BAD"}` now serializes as `{"my_field": "MY_ENUM_MORE_SPECIFIC"}` .

If an old binary that hasn’t had the new enum value added sees this JSON document, it won’t parse correctly, and you’ll be in for a bad time.

You can argue that this is a language bug, and it kind of is. Protobuf should include an equivalent of `json_name` for enum values, or mandate that JSON should serialize enum values with multiple names as a number, rather than an arbitrarily chosen enum name. The feature is intended to allow renaming of enum values, but unfortunately Protobuf hobbled it enough that it’s pretty dangerous.

# [What To Do](\#what-to-do)

Instead, if you *really* need to rename an enum value for usability or compliance reasons (ideally, not just aesthetics) you’re better off making a new enum type in a new version of your API. As long as the enum value numbers are the same, it’ll be binary-compatible, but it will *somewhat* reduce the risk of the above JSON confusion.

Buf provides a lint rule against this feature, [`ENUM_NO_ALLOW_ALIAS`](https://buf.build/docs/lint/rules/#enum_no_allow_alias) , and Protobuf requires that you specify a magic option to enable this behavior, so in practice you don’t need to worry about this. But remember, the consequences of enum aliases go much further than JSON—they affect anything that uses reflection. So even if you don’t use JSON, you can still get burned.
