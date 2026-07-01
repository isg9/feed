---
title: All Package Management Functionality Moved from Compiler to Build System
url: https://ziglang.org/devlog/2026/#2026-06-30
published: "2026-06-30T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2026/#2026-06-30
---

# [All Package Management Functionality Moved from Compiler to Build System](\#2026-06-30)

Author: Andrew Kelley

Now that there is a separate process for users’ build.zig scripts and the build system itself, it makes sense for that to be the place that package management logic lives.

I moved these subcommands to the maker process:

- `zig build`
- `zig fetch`
- `zig init`
- `zig libc`

This means that large parts of what used to be included in the compiler executable are now shipped in source form instead, including:

- package fetching logic
- HTTP client and networking
- TLS (Transport Layer Security) and associated crypto
- Git protocol
- xz, gzip, zstd, flate, zip
- parsing, validation, and otherwise dealing with `build.zig.zon` files

Consequently, this functionality can now be patched without rebuilding the compiler, making it easier for users and contributors to tinker.

Furthermore, it means that package management in zig now has safety checks enabled when doing networking, since the maker executable is compiled in `ReleaseSafe` mode. Plus, all the crypto used for networking and file hashing can now take advantage of special CPU instructions available on the host, even the ones that are too rare to normally depend on when distributing software. *We can have AOT cake and eat JIT, too!*

My original motivation for doing this was in relation to exposing a [build server protocol](https://codeberg.org/ziglang/zig/issues/35538) in order to unblock [ZLS](https://zigtools.org/zls/) after [maker/configurer process separation](#2026-05-26) made breaking changes to the `--build-runner` override flag.

Originally, the process tree looked like this:

```
zig build  (the zig compiler + package manager)
└─ builder (the user's build.zig logic + build system implementation)

```

The process separation changeset made it look like this instead:

```
zig build     (the zig compiler + package manager)
├─ configurer (the user's build.zig logic)
└─ maker      (build system)

```

At this point, consider a long-running `zig build --watch` process, watching files and rebuilding on source code changes. If any changes to `build.zig` are detected, or any files observed during execution of that logic, it means `configurer` needs to be rerun, meaning that `maker` process must exit to give `zig build` a chance to repeat the package management logic.

Now, after the changes described in this devlog entry, it looks like this:

```
zig build        (the zig compiler)
└─ maker         (build system + package manager)
   └─ configurer (the user's build.zig logic)

```

Thus, when configuration needs to be rerun, `maker` process can continue to live because it is the parent process rather than a sibling. In terms of the upcoming build server, it means avoiding an awkward situation where the server has to exit and the client has to reconnect, rather than simply informing the client of a configuration change.

This is almost entirely a non-breaking change, but there are some observable differences:

- Zig executable binary size: shrinks 4% from 14.1 to 13.5 MiB (no LLVM, ReleaseSmall)
- `--maker-opt` flag is replaced by `ZIG_DEBUG_MAKER` environment variable
- `--zig-lib-dir` flag is replaced by `ZIG_LIB_DIR` environment variable

The follow-up issues to this changeset are the main blockers until we tag Zig 0.17.0:

- [build server protocol MVP](https://codeberg.org/ziglang/zig/issues/35538) (needed to unblock ZLS)
- [introduce the concept of adding path dependencies of the build script itself](https://codeberg.org/ziglang/zig/issues/35473)
- [make `zig build --watch` detect modifications to the build script and rerun itself](https://github.com/ziglang/zig/issues/20602)
- [different cwd causes build script cache miss](https://codeberg.org/ziglang/zig/issues/35500)

I have two conferences coming up in July and I need to work on my talks, so being realistic, I don’t think I will have time to wrap these up until early August. Contributions welcome, of course.

Big thanks to Techatrix from the ZLS team for reaching out and working with me on the build server protocol! [They are seeking sponsorship](https://github.com/sponsors/zigtools), by the way.
