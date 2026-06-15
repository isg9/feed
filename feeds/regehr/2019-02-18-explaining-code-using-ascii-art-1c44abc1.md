---
title: Explaining Code using ASCII Art
url: https://blog.regehr.org/archives/1653
published: "2019-02-18T16:53:43Z"
feed: regehr
guid: http://blog.regehr.org/?p=1653
---

# Explaining Code using ASCII Art

People tend to be visual: we use pictures to understand problems. Mainstream programming languages, on the other hand, operate in an almost completely different kind of abstract space, leaving a big gap between programs and pictures. This piece is about pictures drawn using a text character set and then embedded in source code. I love these! The other day I asked around on Twitter for more examples and [the responses](https://twitter.com/johnregehr/status/1095018518737637376) far exceeded expectations (thanks everyone!). There are a ton of great examples in the thread; here I’ve categorized a few of them. Click on images go to the repositories.

## Data Structures

One of the most common kinds of ASCII art in code is illustrating the shape of a data structure.

The example I started with comes from LLVM:

[![](../extra_files/ascii/llvm1.jpg)](https://t.co/FGDDGsTOsW)

The layout of a data structure in the Jikes RVM:

[![](../extra_files/ascii/jikes.jpg)](https://github.com/JikesRVM/JikesRVM/blob/master/rvm/src/org/jikesrvm/objectmodel/TIBLayoutConstants.java#L19)

A tree rotate in Musl:

[![](../extra_files/ascii/musl.jpg)](https://git.musl-libc.org/cgit/musl/tree/src/search/tsearch.c?id=v1.1.21)

Double-ended queue from the Rust library:

[![](../extra_files/ascii/rust-deq.jpg)](https://github.com/rust-lang/rust/blob/57d7cfc3cf50f0c427ad3043ff09eaef20671320/src/liballoc/collections/vec_deque.rs#L1399)

Swift compiler internals:

[![](../extra_files/ascii/swift.jpg)](https://github.com/apple/swift/blob/master/lib/AST/NameLookup.cpp#L1521)

Malloc header layout:

[![](../extra_files/ascii/malloc.jpg)](https://code.woboq.org/userspace/glibc/malloc/malloc.c.html#1075)

## State Machines

JavaScript profiling:

[![](../extra_files/ascii/js-profile.jpg)](https://searchfox.org/mozilla-central/rev/00c0d068ece99717bea7475f7dc07e61f7f35984/tools/profiler/core/RegisteredThread.h#293-301)

RPCs in Cloud Spanner:

[![](../extra_files/ascii/go-rpc.jpg)](https://github.com/googleapis/google-cloud-go/blob/master/spanner/read.go#L353)

I/O stream states:

[![](../extra_files/ascii/stream.jpg)](https://github.com/carllerche/h2/blob/8a0b7ff64f5fe26324ee9f540722549231ac1ad6/src/proto/streams/state.rs#L13-L50)

## Logical Structure in the Problem Domain

Control flow in an NWScript program being decompiled:

[![](../extra_files/ascii/decompile.png)](https://github.com/xoreos/xoreos-tools/blob/master/src/nwscript/controlflow.cpp#L328)

ECC internals:

[![](../extra_files/ascii/dalek.jpg)](https://github.com/dalek-cryptography/curve25519-dalek/blob/develop/src/backend/vector/avx2/edwards.rs#L123)

Formatting numbers:

[![](../extra_files/ascii/number.jpg)](https://searchfox.org/mozilla-central/source/js/src/builtin/intl/NumberFormat.cpp#592)

A quantum circuit:

[![](../extra_files/ascii/quantum.png)](https://github.com/Microsoft/QuantumKatas/blob/master/PhaseEstimation/Tasks.qs#L117)

Balancing memory management objectives in an OS kernel:

[![](../extra_files/ascii/paging.jpg)](https://github.com/torvalds/linux/blob/v4.20/mm/page-writeback.c#L828)

Subtyping relations (this is a very cool special case where the ASCII art is also code):

[![](../extra_files/ascii/racket.jpg)](https://docs.racket-lang.org/2d/index.html)

The format of a DBF file:

[![](../extra_files/ascii/dbf.jpg)](https://github.com/SocialExplorer/FastDBF/blob/master/FastDBF/DbfHeader.cs#L35)

A lookup table for image processing:

[![](../extra_files/ascii/sharpen.jpg)](https://github.com/libvips/libvips/blob/master/libvips/convolution/sharpen.c#L412)

Shape of a color function:

[![](../extra_files/ascii/color.jpg)](https://github.com/chriscalo/design-system-utils/blob/master/src/color-scales.js)

Structure of a URI:

[![](../extra_files/ascii/uri.jpg)](https://searchfox.org/mozilla-central/rev/00c0d068ece99717bea7475f7dc07e61f7f35984/netwerk/base/nsIURI.idl#9-36)

A very quick tutorial on undo systems from emacs:

[![](../extra_files/ascii/undo.jpg)](https://github.com/emacsmirror/undo-tree/blob/master/undo-tree.el#L242-L751)

## Geometry

Attitude control in the Apollo Guidance Computer (!!!):

[![](../extra_files/ascii/apollo.jpg)](https://github.com/chrislgarry/Apollo-11/blob/master/Comanche055/AUTOMATIC_MANEUVERS.agc#L193)

Image tiling:

[![](../extra_files/ascii/image1.jpg)](https://searchfox.org/mozilla-central/rev/00c0d068ece99717bea7475f7dc07e61f7f35984/gfx/wr/webrender/src/image.rs#247)

Boomerang trajectories in Nethack:

[![](../extra_files/ascii/boomerang.png)](https://github.com/NetHack/NetHack/blob/1a8a774719abe55b5a57f63cc43f183339dc6433/src/zap.c#L3461-L3472)

Rendering CSS borders:

[![](../extra_files/ascii/css.jpg)](https://searchfox.org/mozilla-central/rev/00c0d068ece99717bea7475f7dc07e61f7f35984/layout/painting/nsCSSRenderingBorders.cpp#918-956)

Quadtrees:

[![](../extra_files/ascii/quadtrees.jpg)](https://github.com/libfive/libfive/blob/master/libfive/src/render/brep/simplex/simplex_tree.cpp#L688-L717)

Speed control in a milling machine:

[![](../extra_files/ascii/milling.jpg)](https://github.com/grbl/grbl/blob/b237ad566ad25a1503e14fa5f487c34f88bec01f/grbl/stepper.c#L147)

Scrolling web pages:

[![](../extra_files/ascii/scrolling.jpg)](https://github.com/stipsan/compute-scroll-into-view/blob/d6447854d04031ee3942c6d5654a8477b6bb928f/src/index.ts#L124-L168)

I hope you enjoyed these as much as I did!
