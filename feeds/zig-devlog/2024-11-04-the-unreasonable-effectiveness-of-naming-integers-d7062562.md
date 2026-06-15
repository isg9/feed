---
title: The Unreasonable Effectiveness of Naming Integers
url: https://ziglang.org/devlog/2024/#2024-11-04
published: "2024-11-04T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2024/#2024-11-04
---

# The Unreasonable Effectiveness of Naming Integers

November 04, 2024

# [The Unreasonable Effectiveness of Naming Integers](\#2024-11-04)

Author: Andrew Kelley

Lately my Zig code has started to look like this in all my projects:

```zig
/// Uniquely identifies a section index across all objects. Each Object has a section_start field.
/// By subtracting that value from this one, the Object section index is obtained.
const SectionIndex = enum(u32) {
    _,
};

/// Index into object_function_imports
pub const ObjectFunctionImportIndex = enum(u32) {
    _,
};

/// Index into `object_global_imports`
pub const ObjectGlobalImportIndex = enum(u32) {
    _,
};

/// Index into object_table_imports.
pub const ObjectTableImportIndex = enum(u32) {
    _,
};

/// Index into `function_imports`.
pub const FunctionImportIndex = enum(u32) {
    _,
};

/// Index into `global_imports`.
pub const GlobalImportIndex = enum(u32) {
    _,
};

/// Index into `table_imports`.
pub const TableImportIndex = enum(u32) {
    _,
};

/// Index into `memory_imports`.
pub const MemoryImportIndex = enum(u32) {
    _,
};

/// Index into `output_globals`.
pub const GlobalIndex = enum(u32) {
    _,
};

/// Index into `functions`.
pub const FunctionIndex = enum(u32) {
    _,
};

/// Index into object_functions
pub const ObjectFunctionIndex = enum(u32) {
    _,

    pub fn toOptional(i: ObjectFunctionIndex) OptionalObjectFunctionIndex {
        const result: OptionalObjectFunctionIndex = @enumFromInt(@intFromEnum(i));
        assert(result != .none);
        return result;
    }
};

/// Index into object_functions, or null.
pub const OptionalObjectFunctionIndex = enum(u32) {
    none = std.math.maxInt(u32),
    _,

    pub fn unwrap(i: OptionalObjectFunctionIndex) ?ObjectFunctionIndex {
        if (i == .none) return null;
        return @enumFromInt(@intFromEnum(i));
    }
};

```

It’s uncanny how effective it is to avoid bugs by simply naming integers and thereby giving Zig’s type system a chance to catch mistakes. It’s extremely annoying and time consuming to troubleshoot this kind of mistake at runtime, but it’s *trivial* to solve when you get a compile error from a type mismatch.
