---
title: What the Hell Is a Target Triple?
url: /2025/04/14/target-triples
published: "2025-04-14T00:00:00Z"
feed: mcyoung
guid: /2025/04/14/target-triples
---

# What the Hell Is a Target Triple?

*Cross-compiling* is taking a computer program and compiling it for a machine that isn’t the one hosting the compilation. Although historically compilers would only compile for the host machine, this is considered an anachronism: all serious native compilers are now cross-compilers.

After all, you don’t want to be building your iPhone app on literal iPhone hardware.

Many different compilers have different mechanisms for classifying and identifying *targets*. A target is a platform that the compiler can produce executable code for. However, due to the runaway popularity of LLVM, virtually all compilers now use *target triples*. You may have already encountered one, such as the venerable `x86_64-unknown-linux`, or the evil `x86_64-pc-windows`. This system is convoluted and *almost* self-consistent.

But what is a target triple, and where did they come from?

## [Stupid GCC Conventions](\#stupid-gcc-conventions)

So if you go poking around the [Target Triplet](https://wiki.osdev.org/Target_Triplet) page on OSDev, you will learn both true and false things about target triples, because this page is about GCC, not native compilers in general.

> [warning](#warning:1)
>
> Generally, there is no “ground truth” for what a target triple is. There isn’t some standards body that assigns these names. But as we’ll see, LLVM is the trendsetter.

If you run the following command you can learn the target triple for your machine:

```console highlight
$ gcc -dumpmachine
x86_64-linux-gnu

```

[Console](#code:1)

Now if you’re at all familiar with any system that makes pervasive use of target triples, you will know that this is *not a target triple*, because this target’s name is `x86_64-unknown-linux-gnu`, which is what both `clang` and `rustc` call-

```console highlight
$ clang -dumpmachine
x86_64-pc-linux-gnu
$ rustc -vV | grep host
host: x86_64-unknown-linux-gnu

```

[Console](#code:2)

Oh no.

Well, GCC is missing the the `pc` or `unknown` component, and that’s specifically a GCC thing; it allows omitting parts of the triple in such a way that is unambiguous. And they are a GCC invention, so perhaps it’s best to start by assessing GCC’s beliefs.

According to GCC, a target triple is a string of the form `<machine>-<vendor>-<os>`. The “machine” part unambiguously identifies the architecture of the system. Practically speaking, this is the assembly language that the compiler will output at the end. The “vendor” part is essentially irrelevant, and mostly is of benefit for sorting related operating systems together. Finally, the “os” part identifies the operating system that this code is being compiled for. The main thing this identifies for a compiler is the executable format: COFF/PE for Windows, Mach-O for Apple’s operating systems, ELF for Linux and friends, and so on (this, however, is an oversimplification).

But you may notice that `x86_64-unknown-linux-gnu` has an extra, fourth entry[1](#fn:1), which plays many roles but is most often called the target’s “ABI”. For `linux`, it identifies the target’s libc, which has consequences for code generation of some language features, such as thread locals and unwinding. It is optional, since many targets only have one ABI.

### [Cross Compiling with GCC](\#cross-compiling-with-gcc)

A critical piece of history here is to understand the really stupid way in which GCC does cross compiling. Traditionally, each GCC binary would be built for *one* target triple. The full name of a GCC binary would include the triple, so when cross-compiling, you would compile with `x86_64-unknown-linux-gcc`, link with `x86_64-unknown-linux-ld`, and so on (here, `gcc` is not the fourth ABI component of a triple; it’s just one of the tools in the `x86_64-unknown-linux` toolchain).

Nobody with a brain does this[2](#fn:2). LLVM and all cross compilers that follow it instead put all of the backends in one binary, and use a compiler flag like `--target` to select the backend.

But regardless, this is where target triples come from, and why they look the way they look: they began as prefixes for the names of binaries in `autoconf` scripts.

But GCC is ancient technology. In the 21st century, LLVM rules all native compilers.

## [Names in the Ancient Language](\#names-in-the-ancient-language)

LLVM’s target triple list is the one that should be regarded as “most official”, for a few reasons:

1. Inertia. Everyone and their mother uses LLVM as a middleend and backend, so its naming conventions bubble up into language frontends like `clang`, `rustc` `swiftc`, `icc`, and `nvcc`.

2. Upstream work by silicon and operating system vendors. LLVM is what people get hired to work on for the most part, not GCC, so its platform-specific conventions often reflect the preferences of vendors.

These are in no small part because Apple, Google, and Nvidia have armies of compiler engineers contributing to LLVM.

The sources for “official” target triples are many. Generally, I would describe a target triple as “official” when:

1. A major compiler (so, `clang` or `rustc`) uses it. Rust does a way better job than LLVM of documenting their targets, so I prefer to give it deference. You can find Rust’s official triples [here](https://doc.rust-lang.org/rustc/platform-support.html).

2. A platform developer (e.g., a hardware manufacturer, OS vendor) distributes a toolchain with a target triple in the `arch-vendor-os` format.

So, what are the names in class (1)? LLVM does not really go out of its way to provide such a list. But we gotta start somewhere, so source-diving it is.

We can dig into [`Triple.cpp`](https://llvm.org/doxygen/Triple_8cpp_source.html) in LLVM’s target triple parser. It lists all of the names LLVM recognizes for each part of a triple. Looking at `Triple::parseArch()`, we have the following names, including many, many aliases. The first item on the right column is LLVM’s preferred name for the architecture, as indicated by `Triple::getArchTypeName()`.ArchitecturePossible Names[Intel x86](https://en.wikipedia.org/wiki/X86) (32-bit)`i386`, `i486`, `i586`, `i686`, `i786`, `i886`, `i986`[Intel x86](https://en.wikipedia.org/wiki/X86) (64-bit)`x86_64`, `amd64`, `x86_86h` [3](#fn:3)[ARM](https://en.wikipedia.org/wiki/ARM_architecture_family) (32-bit)`arm`, `xscale`, …[ARM](https://en.wikipedia.org/wiki/ARM_architecture_family) (32-bit, big-endian)`armeb`, `xscaleeb`, …[ARM](https://en.wikipedia.org/wiki/ARM_architecture_family) (64-bit)`aarch64`, `aarch64e`, `aarch64ec`, `arm64`, …[ARM](https://en.wikipedia.org/wiki/ARM_architecture_family) (64-bit, big-endian)`aarch64_be`, …[ARM](https://en.wikipedia.org/wiki/ARM_architecture_family) (64-bit, ILP32[4](#fn:4))`aarch64_32`, `arm64_32`, …[ARM Thumb](https://en.wikipedia.org/wiki/ARM_architecture_family#Thumb)`thumb`, …[ARM Thumb](https://en.wikipedia.org/wiki/ARM_architecture_family#Thumb) (big-endian)`thumbeb`, …[IBM PowerPC](https://en.wikipedia.org/wiki/PowerPC) [5](#fn:5) (32-bit)`powerpc`, `powerpcspe`, `ppc`, `ppc32`[IBM PowerPC](https://en.wikipedia.org/wiki/PowerPC) (little-endian)`powerpcle`, `ppcle`, `ppc32le`[IBM PowerPC](https://en.wikipedia.org/wiki/PowerPC) (64-bit)`powerpc64`, `ppu`, `ppc64`[IBM PowerPC](https://en.wikipedia.org/wiki/PowerPC) (64-bit, little-endian)`powerpc64le`, `ppc64le`[MIPS](https://en.wikipedia.org/wiki/MIPS_architecture) (32-bit)`mips`, `mipseb`, `mipsallegrex`, `mipsisa32r6`, `mipsr6`[MIPS](https://en.wikipedia.org/wiki/MIPS_architecture) (32-bit, little-endian)`mipsel`, `mipsallegrexel`, `mipsisa32r6el`, `mipsr6el`[MIPS](https://en.wikipedia.org/wiki/MIPS_architecture) (64-bit)`mips64`, `mips64eb`, `mipsn32`, `mipsisa64r6`, `mips64r6`, `mipsn32r6`[MIPS](https://en.wikipedia.org/wiki/MIPS_architecture) (64-bit, little-endian)`mips64el`, `mipsn32el`, `mipsisa64r6el`, `mips64r6el`, `mipsn32r6el`[RISC-V](https://en.wikipedia.org/wiki/RISC-V) (32-bit)`riscv32`[RISC-V](https://en.wikipedia.org/wiki/RISC-V) (64-bit)`riscv64`[IBM z/Architecture](https://en.wikipedia.org/wiki/Z/Architecture)`s390x` [6](#fn:6), `systemz`[SPARC](https://en.wikipedia.org/wiki/SPARC)`sparc`[SPARC](https://en.wikipedia.org/wiki/SPARC) (little-endian)`sparcel`[SPARC](https://en.wikipedia.org/wiki/SPARC) (64-bit)`sparcv6`, `sparc64`[WebAssembly](https://en.wikipedia.org/wiki/WebAssembly) (32-bit)`wasm32`[WebAssembly](https://en.wikipedia.org/wiki/WebAssembly) (64-bit)`wasm64`[Loongson](https://en.wikipedia.org/wiki/Loongson) (32-bit)`loongarch32`[Loongson](https://en.wikipedia.org/wiki/Loongson) (64-bit)`loongarch64`——————————————-———————————————————————–[Radeon R600](https://en.wikipedia.org/wiki/Radeon_HD_2000_series)`r600`[AMD GCN](https://en.wikipedia.org/wiki/Graphics_Core_Next)`amdgcn`[Qualcomm Hexagon](https://en.wikipedia.org/wiki/Qualcomm_Hexagon)`hexagon`[Nvidia PTX](https://en.wikipedia.org/wiki/Parallel_Thread_Execution) [7](#fn:7) (32-bit)`nvptx`[Nvidia PTX](https://en.wikipedia.org/wiki/Parallel_Thread_Execution) (64-bit)`nvptx64`AMD IL[8](#fn:8) (32-bit)`amdil`AMD IL (64-bit)`amdil64`Direct-X IL`dxil`, …[HSAIL](https://en.wikipedia.org/wiki/Heterogeneous_System_Architecture) (32-bit)`hsail`[HSAIL](https://en.wikipedia.org/wiki/Heterogeneous_System_Architecture) (64-bit)`hsail64`[Khronos SPIR](https://en.wikipedia.org/wiki/Standard_Portable_Intermediate_Representation) (32-bit)`spir`[Khronos SPIR](https://en.wikipedia.org/wiki/Standard_Portable_Intermediate_Representation) (64-bit)`spir64`[Khronos SPIR-V](https://en.wikipedia.org/wiki/Standard_Portable_Intermediate_Representation)`spirv`, …[Khronos SPIR-V](https://en.wikipedia.org/wiki/Standard_Portable_Intermediate_Representation) (32-bit)`spirv32`, …[Khronos SPIR-V](https://en.wikipedia.org/wiki/Standard_Portable_Intermediate_Representation) (64-bit)`spirv64`, …[Android RenderScript](https://developer.android.com/guide/topics/renderscript/compute) (32-bit)`renderscript32`[Android RenderScript](https://developer.android.com/guide/topics/renderscript/compute) (64-bit)`renderscript64`[Movidius SHAVE](https://en.wikichip.org/wiki/movidius/microarchitectures/shave_v2.0)`shave`——————————————-———————————————————————–[Atmel AVR](https://en.wikipedia.org/wiki/AVR_microcontrollers)`avr`[Motorola 68k](https://en.wikipedia.org/wiki/Motorola_68000_series)`m68k`[Argonaut ARC](https://en.wikipedia.org/wiki/ARC_(processor))`arc`[Texas Instruments MSP430](https://en.wikipedia.org/wiki/TI_MSP430)`msp430`[Tensilica Xtensa](https://en.wikipedia.org/wiki/Tensilica)`xtensa`[C-SKY](https://c-sky.github.io/)`csky`[OpenASIP](https://github.com/cpc/openasip)`tce`[OpenASIP](https://github.com/cpc/openasip) (little-endian)`tcele`[Myracom Lanai](https://q3k.org/lanai.html)`lanai`XMOS xCore`xcore`Kalimba[9](#fn:9)`kalimba`VE[9](#fn:9)`ve`

Here we begin to see that target triples are not a neat system. They are *hell*. Where a list of architecture names contains a “…”, it means that LLVM accepts many more names.

The problem is that architectures often have *versions* and *features*, which subtly change how the compiler generates code. For example, when compiling for an `x86_64`, we may want to specify that we want AVX512 instructions to be used. On LLVM, you might do that with `-mattr=+avx512`. Every architecture has a subtly-different way of doing this, because every architecture had a *different* *GCC*! Each variant of GCC would put different things behind `-mXXX` flags ( `-m` for “machine”), meaning that the interface is not actually that uniform. The meanings of `-march`, `-mcpu`, `-mtune`, and `-mattr` thus vary wildly for this reason.

Because LLVM is supposed to replace GCC (for the most part), it replicates a lot of this wacky behavior.

So uh, we gotta talk about 32-bit ARM architecture names.

### [`ARMTargetParser.cpp`](\#armtargetparsercpp)

There is a hellish file in LLVM dedicated to parsing ARM architecture names. Although members of the ARM family have many configurable features (which you can discover with `llc -march aarch64 -mattr help` [10](#fn:10)), the name of the architecture is somewhat meaningful, and can hav many options, mostly relating to the many versions of ARM that exist.

How bad is it? Well, we can look at all of the various ARM targets that `rustc` supports with `rustc --print target-list`:

```console highlight
$ rustc --print target-list | grep -P 'arm|aarch|thumb' \
  | cut -d- -f1 | sort | uniq
aarch64
aarch64_be
arm
arm64_32
arm64e
arm64ec
armeb
armebv7r
armv4t
armv5te
armv6
armv6k
armv7
armv7a
armv7k
armv7r
armv7s
armv8r
thumbv4t
thumbv5te
thumbv6m
thumbv7a
thumbv7em
thumbv7m
thumbv7neon
thumbv8m.base
thumbv8m.main

```

[Console](#code:3)

Most of these are 32-bit ARM versions, with profile information attached. These correspond to the names given [here](https://en.wikipedia.org/wiki/ARM_architecture_family#Cores). Why does ARM stick version numbers in the architecture name, instead of using `-mcpu` like you would on x86 (e.g. `-mcpu alderlake`)? I have no idea, because ARM is not my strong suit. It’s likely because of how early ARM support was added to GCC.

Internally, LLVM calls these “subarchitectures”, although ARM gets special handling because there’s so many variants. SPIR-V, Direct X, and MIPS all have subarchitectures, so you might see something like `dxilv1.7` if you’re having a bad day.

Of course, LLVM’s ARM support also sports some naughty subarchitectures not part of this system, with naughty made up names.

- `arm64e` is an Apple thing, which is an enhancement of `aarch64` present on some Apple hardware, which adds their own flavor of [pointer authentication](https://developer.apple.com/documentation/security/preparing-your-app-to-work-with-pointer-authentication) and some other features.

- `arm64ec` is a completely unrelated Microsoft invention that is essentially “ `aarch64` but with an `x86_64`-ey ABI” to make `x86_64` emulation on what would otherwise be `aarch64-pc-windows-msvc` target somewhat more amenable.

> [note](#note:1)
>
> Why the Windows people invented a whole other ABI instead of making things clean and simple like Apple did with Rosetta on ARM MacBooks? I have no idea, but [http://www.emulators.com/docs/abc\_arm64ec\_explained.htm](http://www.emulators.com/docs/abc_arm64ec_explained.htm) contains various excuses, none of which I am impressed by. My read is that their compiler org was just worse at life than Apple’s, which is not surprising, since Apple does compilers better than anyone else in the business.

Actually, since we’re on the topic of the names of architectures, I have a few things I need to straighten out.

### [Made Up Names of Architectures](\#made-up-names-of-architectures)

x86 and ARM both seem to attract a lot of people making up nicknames for them, which leads to a lot of confusion in:

1. What the “real” name is.

2. What name a particular toolchain wants.

3. What name you should use in your own cosmopolitan tooling.

Let’s talk about the incorrect names people like to make up for them. Please consider the following a relatively normative reference on what people call these architectures, based on my own experience with many tools.

When we say “x86” unqualified, in 2025, we almost always mean `x86_64`, because 32-bit x86 is dead. If you need to talk about 32-bit x86, you should either say “32-bit x86”, “protected mode”[11](#fn:11), or “i386” (the first Intel microarchitecture that implemented protected mode)[12](#fn:12). You should not call it `x86_32` or just `x86`.

You might also call it IA-32 for Intel Architecture 32, (or `ia32`), but nobody calls it that and you risk confusing people with `ia64`, or IA-64, the official name of Intel’s failed general-purpose VLIW architecture, [Itanium](https://en.wikipedia.org/wiki/Itanium), which is in no way compatible with x86. `ia64` was what GCC and LLVM named Itanium triples with. Itanium support was drowned in a bathtub during the Obama administration, so it’s not really relevant anymore. Rust has never had official Itanium support.

32-bit x86 is *extremely not* called “x32”; this is what Linux used to call its x86 ILP32[4](#fn:4) variant before it was removed (which, following the ARM names, would have been called `x86_6432`).

There are also many ficticious names for 64-bit x86, which you should avoid unless you want the younger generation to make fun of you. `amd64` refers to AMD’s original implementation of long mode in their K8 microarchitecture, first shipped in their [Athlon 64](https://en.wikipedia.org/wiki/Athlon_64) product. AMD still makes the best x86 chips (I am writing this on a machine socketed with a Zen2 Threadripper), sure, but calling it `amd64` is silly and also looks a lot like `arm64`, and I am honestly kinda annoyed at how much Go code I’ve seen with files named `fast_arm64.s` and `fast_amd64.s`. Debian also uses `amd64`/ `arm64`, which makes browsing packages kind of annoying.

On that topic, you should *absolutely not* call 64-bit mode `k8`, after the AMD K8. Nobody except for weird computer taxonomists like me know what that is. But Bazel calls it that, and it’s really irritating[13](#fn:13).

You should also not call it `x64`. Although LLVM does accept `amd64` for historical purposes, no one calls it `x64` except for Microsoft. And even though it is fairly prevalent on Windows, I absolutely give my gamedev friends a hard time when they write `x64`.

On the ARM side, well. Arm[14](#fn:14) has a bad habit of not using consistent naming for 64-bit ARM, since they used both AArch64 and ARM64 for it. However, in compiler land, `aarch64` appears to be somewhat more popular.

You should also probably stick to the LLVM names for the various architectures, instead of picking your favorite Arm Cortex name (like `cortex_m0`).

## [Vendors and Operating Systems](\#vendors-and-operating-systems)

The worst is over. Let’s now move onto examinining the rest of the triple: the platform vendor, and the operating system.

The vendor is intended to identify who is responsible for the ABI definition for that target. Although provides little to no value to the compiler itself, but it does help to sort related targets together. Sort of.

Returning to `llvm::Triple`, we can examine `Triple::VendorType`. Vendors almost always correspond to companies which develop operating systems or other platforms that code runs on, with some exceptions.

We can also get the vendors that `rustc` knows about with a handy dandy command:

```console highlight
rustc --print target-list | grep -P '\w+-\w+-' | cut -d- -f2 | sort | uniq

```

[Console](#code:4)

The result is this. This is just a representative list; I have left off a few that are not going to be especially recognizeable.VendorNameExample TripleVendor Unknown[15](#fn:15)`unknown``x86_64-unknown-linux`“PC”`pc``x86_64-pc-windows-msvc`[Advanced Micro Devices Inc.](https://en.wikipedia.org/wiki/AMD)`amd``amdgcn-amd-gfx906`[Apple Inc.](https://en.wikipedia.org/wiki/Apple_Inc.)`apple``aarch64-apple-ios-sim`[Intel Corporation](https://en.wikipedia.org/wiki/Intel)`intel``i386-intel-elfiamcu`[IBM Corporation](https://en.wikipedia.org/wiki/IBM)`ibm``powerpc64-ibm-aix`[Mesa3D Project](https://en.wikipedia.org/wiki/Mesa_(computer_graphics))`mesa``amdgcn-mesa-mesa3d`[MIPS Technologies LLC](https://en.wikipedia.org/wiki/MIPS_Technologies)`mti``mips-mti-none-elf`[Nintendo](https://en.wikipedia.org/wiki/Nintendo)`nintendo``armv6k-nintendo-3ds`[Nvidia Corporation](https://en.wikipedia.org/wiki/Nvidia)`nvidia``nvptx64-nvidia-cuda`[Sony Interactive Entertainment](https://en.wikipedia.org/wiki/Sony_Interactive_Entertainment)`scei`, `sie`, `sony``x86_64-sie-ps5`[Sun Microsystems](https://en.wikipedia.org/wiki/Sun_Microsystems)`sun``sparcv9-sun-solaris`[SUSE S. A.](https://en.wikipedia.org/wiki/SUSE_S.A.)`suse``aarch64-suse-linux`[Red Hat, Inc](https://en.wikipedia.org/wiki/Red_Hat)`redhat``x86_64-redhat-linux`[Universal Windows Platform](https://en.wikipedia.org/wiki/Universal_Windows_Platform)`uwp``aarch64-uwp-windows-msvc`

Most vendors are the names of organizations that produce hardware or operating systems. For example `suse` and `redhat` are used for those organizations’ Linux distributions, as a funny branding thing. Some vendors are projects, like the `mesa` vendor used with the Mesa3D OpenGL implementation’s triples.

The `unknown` vendor is used for cases where the vendor is not specified or just not important. For example, the canonical Linux triple is `x86_64-unknown-linux`… although one could argue it should be `x86_64-torvalds-linux`. It is not uncommon for companies that sell/distribute Linux distributions to have their own target triples, as do SUSE and sometimes RedHat. Notably, there are no triples with a `google` vendor, even though `aarch64-linux-android` and `aarch64-unknown-fuchsia` should really be called `aarch64-google-linux-android` and `aarch64-google-fuchsia`. The target triple system begins to show cracks here.

The `pc` vendor is a bit weirder, and is mostly used by Windows targets. The standard Windows target is `x86_64-pc-windows-msvc`, but really it should have been `x86_64-microsoft-windows-msvc`. This is likely complicated by the fact that there is also a `x86_64-pc-windows-gnu` triple, which is for [MinGW](https://en.wikipedia.org/wiki/MinGW) code. This platform, despite running on Windows, is not provided by Microsoft, so it would probably make more sense to be called `x86_64-unknown-windows-gnu`.

But not all Windows targets are `pc`! [UWP](https://en.wikipedia.org/wiki/Universal_Windows_Platform) apps use a different triple, that replaces the `pc` with `uwp`. `rustc` provides targets for Windows 7 backports that use a `win7` “vendor”.

### [Beyond Operating Systems](\#beyond-operating-systems)

The third (or sometimes second, ugh) component of a triple is the operating system, or just “system”, since it’s much more general than that. The main thing that compilers get from this component relates to generating code to interact with the operating system (e.g. SEH on Windows) and various details related to linking, such as object file format and relocations.

It’s also used for setting defines like `__linux__` in C, which user code can use to determine what to do based on the target.

We’ve seen `linux` and `windows`, but you may have also seen `x86_64-apple-darwin`. *Darwin?*

The operating system formerly known as Mac OS X (now macOS[16](#fn:16)) is a POSIX operating system. The POSIX substrate that all the Apple-specific things are built on top of is called Darwin. [Darwin](https://en.wikipedia.org/wiki/Darwin_(operating_system)) is a free and open source operating system based on Mach, a research kernel whose name survives in Mach-O, the object file format used by all Apple products.

All of the little doodads Apple sells use the actual official names of their OSes, like `aarch64-apple-ios`. For, you know, iOS. On your iPhone. Built with Xcode on your iMac.

`none` is a common value for this entry, which usually means a free-standing environment with no operating system. The object file format is usually specified in the fourth entry of the triple, so you might see something like `riscv32imc-unknown-none-elf`.

Sometimes the triple refers not to an operating system, but to a complete hardware product. This is common with game console triples, which have “operating system” names like `ps4`, `psvita`, `3ds`, and `switch`. (Both Sony and Nintendo use LLVM as the basis for their internal toolchains; the Xbox toolchain is just MSVC).

## [ABI! ABI!](\#abi-abi)

The fourth entry of the triple (and I repeat myself, yes, it’s still a triple) represents the binary interface for the target, when it is ambiguous.

For example, Apple targets never have this, because on an Apple platform, you just shut up and use `CoreFoundation.framework` as your libc. Except this isn’t true, because of things like `x86_64-apple-ios-sim`, the iOS simulator running on an x86 host.

On the other hand, Windows targets will usually specify `-msvc` or `-gnu`, to indicate whether they are built to match MSVC’s ABI or MinGW. Linux targets will usually specify the libc vendor in this position: `-gnu` for glibc, `-musl` for musl, `-newlib` for newlib, and so on.

This doesn’t just influence the calling convention; it also influences how language features, such as thread locals and dynamic linking, are handled. This usually requires coordination with the target libc.

On ARM free-standing ( `armxxx-unknown-none`) targets, `-eabi` specifies the ARM EABI, which is a standard embeded ABI for ARM. `-eabihf` is similar, but indicates that no soft float support is necessary ( `hf` stands for hardfloat). (Note that Rust does not include a vendor with these architectures, so they’re more like `armv7r-none-eabi`).

A lot of jankier targets use the ABI portion to specify the object file, such as the aforementioned `riscv32imc-unknown-none-elf`.

## [WASM Targets](\#wasm-targets)

One last thing to note are the various WebAssembly targets, which completely ignore all of the above conventions. Their triples often only have two components (they are still called triples, hopefully I’ve made that clear by now). Rust is a little bit more on the forefront here than `clang` (and anyways I don’t want to get into Emscripten) so I’ll stick to what’s going on in `rustc`.

There’s a few variants. `wasm32-unknown-unknown` (here using `unknown` instead of `none` as the system, oops) is a completely bare WebAssebly runtime where none of the standard library that needs to interact with the outside world works. This is essentially for building WebAssembly modules to deploy in a browser.

There are also the WASI targets, which provide a standard ABI for talking to the host operating system. These are less meant for browsers and more for people who are using WASI as a security boundary. These have names like `wasm32-wasip1`, which, unusually, lack a vendor! A “more correct” formulation would have been `wasm32-unknown-wasip1`.

## [Aside on Go](\#aside-on-go)

Go does the correct thing and distributes a cross compiler. This is well and good.

Unfortunately, they decided to be different and special and do not use the target triple system for naming their targets. Instead, you set the `GOARCH` and `GOOS` environment variables before invoking `gc`. This will sometimes be shown printed with a slash between, such as `linux/amd64`.

Thankfully, they at least provide documentation for a relevant internal package [here](https://pkg.go.dev/internal/platform), which offers the names of various `GOARCH` and `GOOS` values.

They use completely different names from everyone else for a few things, which is guaranteed to trip you up. They use call the 32- and 64-bit variants of x86 `386` (note the lack of leading `i`) and `amd64`. They call 64-bit ARM `arm64`, instead of `aarch64`. They call little-endian MIPSes `mipsle` instead of `mipsel`.

They also call 32-bit WebAssembly `wasm` instead of `wasm32`, which is a bit silly, and they use `js/wasm` as their equivalent of `wasm32-unknown-unknown`, which is *very* silly.

Android is treated as its own operating system, `android`, rather than being `linux` with a particular ABI; their system also can’t account for ABI variants in general, since Go originally wanted to not have to link any system libraries, something that does not actually work.

If you are building a new toolchain, don’t be clever by inventing a cute target triple convention. All you’ll do is annoy people who need to work with a lot of different toolchains by being different and special.

## [Inventing Your Own Triples](\#inventing-your-own-triples)

Realistically, you probably shouldn’t. But if you must, you should probably figure out what you want out of the triple.

Odds are there isn’t anything interesting to put in the vendor field, so you will avoid people a lot of pain by picking `unknown`. Just include a vendor to avoid pain for people in the future.

You should also avoid inventing a new name for an existing architecture. Don’t name your hobby operating system’s triple `amd64-unknown-whatever`, please. And you definitely don’t want to have an ABI component. One ABI is enough.

If you’re inventing a triple for a free-standing environment, but want to specify something about the hardware configuration, you’re probably gonna want to use `-none-<abi>` for your system. For some firmware use-cases, though, the system entry is a better place, such as for the UEFI triples. Although, I have unforunately seen both `x86_64-unknown-uefi` and `x86_64-pc-none-uefi` in the wild.

And most imporantly: this sytem was built up organically. Disabuse yourself now of the idea that the system is consistent and that target triples are easy to parse. Trying to parse them will make you very sad.

---

1. And no, a “target quadruple” is not a thing and if I catch you saying that I’m gonna bonk you with an Intel optimization manual. [↩︎](#fnref:1)

2. I’m not sure why GCC does this. I suspect that it’s because computer hard drives used to be small and a GCC with every target would have been too large to cram into every machine. Maybe it has some UNIX philosophy woo mixed into it.

   Regardless, it’s really annoying and thankfully no one else does this because cross compiling shouldn’t require hunting down a new toolchain for each platform. [↩︎](#fnref:2)

3. This is for Apple’s later-gen x86 machines, before they went all-in on ARM desktop. [↩︎](#fnref:3)

4. ILP32 means that the `int`, `long`, and pointer types in C are 32-bit, despite the architecture being 64-bit. This allows writing programs that are small enough top jive in a 32-bit address space, while taking advantage of fast 64-bit operations. It is a bit of a frankentarget. Also existed once as a process mode on `x86_64-unknown-linux` by the name of `x32`. [↩︎](#fnref:4) [↩︎](#fnref1:4)

5. Not to be confused with POWER, an older IBM CPU. [↩︎](#fnref:5)

6. This name is Linux’s name for IBM’s z/Architecture. See [https://en.wikipedia.org/wiki/Linux\_on\_IBM\_Z#Hardware](https://en.wikipedia.org/wiki/Linux_on_IBM_Z#Hardware). [↩︎](#fnref:6)

7. Not a real chip; refers to Nvidia’s PTX IR, which is what CUDA compiles to. [↩︎](#fnref:7)

8. Similar to PTX; an IR used by AMD for graphics. See [https://openwall.info/wiki/john/development/AMD-IL](https://openwall.info/wiki/john/development/AMD-IL). [↩︎](#fnref:8)

9. No idea what this is, and Google won’t help me. [↩︎](#fnref:9) [↩︎](#fnref1:9)

10. `llc` is the LLVM compiler, which takes LLVM IR as its input. Its interface is much more regular than `clang`’s because it’s not intended to be a substitute for GCC the way `clang` is. [↩︎](#fnref:10)

11. Very kernel-hacker-brained name. It references the three processor modes of an x86 machine: real mode, protected mode, long mode, which correspond to 16-, 32-, and 64-bit modes. There is also a secret fourth mode called [unreal mode](https://en.wikipedia.org/wiki/Unreal_mode), which is just what happens when you come down to real mode from protected mode after setting up a protected mode GDT.

    If you need to refer to real mode, call it “real mode”. Don’t try to be clever by calling it “8086” because you are almost certainly going to be using features that were not in the original Intel 8086. [↩︎](#fnref:11)

12. I actually don’t like this name, but it’s the one LLVM uses so I don’t really get to complain. [↩︎](#fnref:12)

13. Bazel also calls 32-bit x86 `piii`, which stands for, you guessed it, “Pentium III”. Extremely unserious. [↩︎](#fnref:13)

14. The intelectual property around ARM, the architecture famility, is owned by the British company Arm Holdings. Yes, the spelling difference is significant.

    Relatedly, ARM is not an acronym, and is sometimes styled in all-lowercase as arm. The distant predecesor of Arm Holdings is Acorn Computers. Their first compute, the Acorn Archimedes, contained a chip whose target triple name today might have been `armv1`. Here, ARM was an acronym, for Acorn RISC Machine. Wikipedia alleges without citation that the name was at once point changed to Advanced RISC Machine at the behest of Apple, but I am unable to find more details. [↩︎](#fnref:14)

15. “You are not cool enough for your company to be on the list.” [↩︎](#fnref:15)

16. Which I pronounce as one word, “macos”, to drive people crazy. [↩︎](#fnref:16)
