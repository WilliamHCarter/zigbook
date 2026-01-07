# Chapter 6: Enums and Pattern Matching

In this chapter, we'll explore **enums** (enumerations) - a way to define a type by listing its possible values. Combined with Zig's powerful `switch` statement, enums let you write safe, expressive code.

## What You'll Learn

We'll cover:

1. **Defining Enums** - Creating types with a fixed set of values
2. **Switch and Pattern Matching** - Exhaustively handling all cases
3. **Optional Types** - Zig's approach to nullable values

## Why Enums Matter

Enums help you:

- **Model state** - Represent values that can only be one of several options
- **Ensure safety** - The compiler checks you handle all cases
- **Express intent** - Make impossible states unrepresentable
- **Write clean code** - `switch` is clearer than chains of `if`/`else`

## A Quick Preview

Here's what enums look like in Zig:

```zig
const IpAddrKind = enum {
    v4,
    v6,
};

const addr = IpAddrKind.v4;

const version = switch (addr) {
    .v4 => 4,
    .v6 => 6,
};
```

And with tagged unions (enums with data):

```zig
const IpAddr = union(enum) {
    v4: [4]u8,
    v6: [16]u8,
};

const localhost = IpAddr{ .v4 = .{ 127, 0, 0, 1 } };

switch (localhost) {
    .v4 => |octets| std.debug.print("IPv4: {any}\n", .{octets}),
    .v6 => |segments| std.debug.print("IPv6: {any}\n", .{segments}),
}
```

## Enums vs Structs

**Structs** group different data together:
```zig
const Rectangle = struct { width: u32, height: u32 };
```

**Enums** define a value that can be one of several variants:
```zig
const Shape = enum { rectangle, circle, triangle };
```

**Tagged unions** combine both - one of several variants, each with different data:
```zig
const Shape = union(enum) {
    rectangle: struct { width: u32, height: u32 },
    circle: struct { radius: u32 },
    triangle: struct { base: u32, height: u32 },
};
```

## Zig's Approach vs Other Languages

- **Rust**: Has powerful enums with pattern matching via `match`
- **C**: Has basic enums (just named integers)
- **Zig**: Combines simple enums with tagged unions for flexibility

Zig gives you:
- Simple enums when you just need named values
- Tagged unions when you need data with variants
- Exhaustive `switch` checking for safety
- Optional types (`?T`) for nullable values

## Getting Started

Ready to make impossible states impossible? Let's begin with [Defining an Enum](01-defining-an-enum.md).
