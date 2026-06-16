---
title: 'Protobuf Tip #5: Avoid import public/weak'
url: /2025/05/13/protobuf-tip-5
published: "2025-05-13T00:00:00Z"
feed: mcyoung
guid: /2025/05/13/protobuf-tip-5
---

# Protobuf Tip #5: Avoid import public/weak

*My dad had a guitar but it was acoustic, so I smashed a mirror and glued broken* *glass to it to make it look more metal. It looked ridiculous! –Max Cavalera*

TL;DR: Avoid `import public` and `import weak`. The Buf lint rules [`IMPORT_NO_PUBLIC`](https://buf.build/docs/lint/rules/#import_no_public) and [`IMPORT_NO_WEAK`](https://buf.build/docs/lint/rules/#import_no_weak) enforce this for you by default.

> [note](#note:1)
>
> I’m editing a series of best practice pieces on Protobuf, a language that I work on which has lots of evil corner-cases.These are shorter than what I typically post here, but I think it fits with what you, dear reader, come to this blog for. These tips are also posted on the [buf.build blog](https://buf.build/blog/totw-5-avoid-import-public-weak).

Protobuf `import` s allow you to specify two special modes: `import public` and `import weak`. The Buf CLI lints against these by default, but you might be tempted to try using them anyway, especially because [some GCP APIs use `import public`](https://github.com/googleapis/googleapis/blob/df58085901d8fb80c2c021e405923bb2351a6f29/google/spanner/v1/spanner.proto#L19). What are these modes, and why do they exist?

## [Import Visibility](\#import-visibility)

Protobuf imports are by file path, a fact that is very strongly baked into the language and its reflection model.

```protobuf highlight
import "my/other/api.proto";

```

[Protobuf](#code:1)

Importing a file dumps all of its symbols into the current file. For the purposes of name resolution, it’s as if all if the declarations in that file have been pasted into the current file. However, this isn’t transitive. If:

- `a.proto` imports `b.proto` …
- and `b.proto` imports `c.proto` …
- and `c.proto` defines `foo.Bar`…
- then, `a.proto` must import `c.proto` to refer to `foo.Bar`, even though `b.proto` imports it.

This is similar to how importing a package as `.` works in Go. When you write `import . "strings"`, it dumps all of the declarations from the `strings` package into the current file, but not those of any files that `"strings"` imports.

Now, what’s nice about Go is that packages can be broken up into files in a way that is transparent to users; users of a package import *the package*, not the files of that package. Unfortunately, Protobuf is not like that, so the file structure of a package leaks to its callers.

`import public` was intended as a mechanism for allowing API writers to break up files that were getting out of control. You can define a new file `new.proto` for some of the definitions in `big.proto`, move them to the new file, and then add `import public "new.proto";` to `big.proto`. Existing imports of `big.proto` won’t be broken, hooray!

Except this feature was designed for C++. In C++, each `.proto` file maps to a `.proto.h` header, which you `#include` in your application code. In C++, `#include` behaves like `import public`, so marking an import as `public` only changes name resolution in Protobuf—the C++ backend doesn’t have to do anything to maintain source compatibility when an import is changed to `public`.

But other backends, like Go, do not work this way: `import` in Go *doesn’t* pull in symbols transitively, so Go would need to explicitly add aliases for all of the symbols that come in through a public import. That is, if you had:

```protobuf highlight
// foo.proto
package myapi.v1;
message Foo { ... }

// bar.proto
package myotherapi.v1;
import public "foo.proto";

```

[Protobuf](#code:2)

Then the Go backend has to generate a `type Foo = foopb.Foo` in `bar.pb.go` to emulate this behavior (in fact, I was surprised to learn Go Protobuf implements this at all). Go *happens* to implement public imports correctly, but not all backends are as careful, because this feature is obscure.

The [`spanner.proto`](https://github.com/googleapis/googleapis/blob/df58085901d8fb80c2c021e405923bb2351a6f29/google/spanner/v1/spanner.proto#L19) example of an `import public` isn’t even used for breaking up an existing file; instead, it’s used to not make a huge file bigger and avoid making callers have to add an additional import. This is a *bad use* of a *bad feature!*

Using `import public` to effectively “hide” imports makes it harder to understand what a `.proto` file is pulling in. If Protobuf imports were at the package/symbol level, like Go or Java, this feature would not need to exist. Unfortunately, Protobuf is closely tailored for C++, and this is one of the consequences.

Instead of using `import public` to break up a file, simply plan to break up the file in the next version of the API.

The [`IMPORT_NO_PUBLIC`](https://buf.build/docs/lint/rules/#import_no_public) Buf lint rule enforces that no one uses this feature by default. It’s tempting, but the footguns aren’t worth it.

## [Weak Imports](\#weak-imports)

Public imports have a good, if flawed, reason to exist. Their implementation details are the main thing that kneecaps them.

Weak imports, however, simply should not exist. They were added to the language to make it easier for some of Google’s enormous binaries to avoid running out of linker memory, by making it so that message types could be dropped if they weren’t accessed. This means that weak imports are “optional”—if the corresponding descriptors are missing at runtime, the C++ runtime can handle it gracefully.

This leads to all kinds of implementation complexity and subtle behavior differences across runtimes. Most runtimes implement (or implemented, in the case of those that removed support) `import weak` in a buggy or inconsistent way. It’s unlikely the feature will ever be truly removed, even though Google has tried.

Don’t use `import weak`. It should be treated as completely non-functional. The [`IMPORT_NO_WEAK`](https://buf.build/docs/lint/rules/#import_no_weak) Buf lint rule takes care of this for you.
