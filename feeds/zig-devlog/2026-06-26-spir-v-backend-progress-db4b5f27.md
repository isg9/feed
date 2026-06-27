---
title: SPIR-V Backend Progress
url: https://ziglang.org/devlog/2026/#2026-06-26
published: "2026-06-26T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2026/#2026-06-26
---

# [SPIR-V Backend Progress](\#2026-06-26)

Author: Ali Cheraghi

There’s quite a bit to cover. The SPIR-V backend had bitrotted in a number of places after the recent compiler changes, so I spent the past several weeks dragging it into a better state.

## @SpirvType

SPIR-V has a handful of types that couldn’t be expressed in Zig’s type system. The new `@SpirvType` builtin has been introduced to address the longest-standing blocker for writing shaders. See [#20550](https://github.com/ziglang/zig/issues/20550), [#23326](https://github.com/ziglang/zig/pull/23326) and [#35461](https://codeberg.org/ziglang/zig/pulls/35461) to trace the background.

```zig
const Sampler = @SpirvType(.sampler);
const Image = @SpirvType(.{ .image = .{
    .usage = .{ .sampled = u32 },
    .format = .unknown,
    .dim = .@"2d",
    .depth = .unknown,
    .arrayed = false,
    .multisampled = false,
    .access = .unknown,
} });
const SampledImage = @SpirvType(.{ .sampled_image = Image });
const RuntimeArray = @SpirvType(.{ .runtime_array = u32 });
const sampled_image = @extern(*addrspace(.constant) const SampledImage, .{
    .name = "sampled_image",
    .decoration = .{ .descriptor = .{ .set = 0, .binding = 1 } },
});

```

## Execution Mode on the Calling Convention

Execution mode info (workgroup size, fragment origin, etc.) is now carried by the calling convention instead of being emitted via inline assembly `OpExecutionMode`. The old `std.gpu.executionMode()` helper is gone, and the SPIR-V assembler now rejects manual `OpExecutionMode` instructions. Two new calling conventions, `spirv_task` and `spirv_mesh`, were also added for mesh shading pipelines.

```zig
export fn vert() callconv(.spirv_vertex) void {}
export fn frag() callconv(.{ .spirv_fragment = .{ .depth_assumption = .greater } }) void {}
export fn comp() callconv(.{ .spirv_kernel = .{ .x = 8, .y = 8, .z = 1 } }) void {}
export fn task() callconv(.{ .spirv_task = .{ .x = 1, .y = 1, .z = 1 } }) void {}
export fn mesh() callconv(.{ .spirv_mesh = .{ .stage_output = .output_lines, .max_primitives = 1, .max_vertices = 2 } }) void {}

```

## Capabilities and Extensions from CPU Features

Capabilities and extensions used to be emitted ad hoc by codegen or via inline assembly. They’re now driven entirely by the CPU feature set like other targets, with dependency chains extracted from [SPIRV-Headers](https://github.com/KhronosGroup/SPIRV-Headers) (excluding external vendors for now), and the assembler now rejects any attempt to emit `OpCapability` or `OpExtension` directly.

## Multi-Threaded Codegen

From day one, the SPIR-V backend ran codegen single-threaded inside the linker thread. Each codegen job now produces an `Mir` value just like every other self-hosted backend, and gets scheduled on the compiler’s thread pool.

The same change brought back two ISel passes that had been removed during earlier refactors: `dedup_types` (which merges equivalent type instructions) and `prune_unused` (which strips dead code from the final module). These had originally been deleted back when codegen was single-threaded.

## Object File Linking

`.spv` files are now recognised as object files. You can compile multiple `.zig` files (or external `.spv` objects) and have the SPIR-V linker stitch them into a single module.

Tens of bugs have also been fixed along the way with a nearly 10% increase in total passing behavior tests (49% now) on the `spirv64-vulkan` target, `std.gpu` was renamed to `std.spirv` and the SPIR-V backend is meaningfully more useful than it was a month ago, but there’s still a long way to go. Plenty of behavior tests remain skipped on SPIR-V. That said, if you’ve been on the fence about trying Zig for shaders or compute kernels, this is a good time to give it a shot. Bug reports are very welcome on [Codeberg](https://codeberg.org/ziglang/zig/issues). Happy hacking!
