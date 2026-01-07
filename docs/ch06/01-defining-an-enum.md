# 6.1. Defining an Enum

Enums allow you to define a type by enumerating its possible values. They're perfect for representing a value that can only be one of a fixed set of options.

## Basic Enum Definition

Define an enum with the `enum` keyword:

```zig
const IpAddrKind = enum {
    v4,
    v6,
};
```

This creates a type `IpAddrKind` with two possible values: `v4` and `v6`.

## Creating Enum Values

Access enum variants with dot notation:

```zig
const four = IpAddrKind.v4;
const six = IpAddrKind.v6;
```

Both values are of type `IpAddrKind`, so they can be used interchangeably where that type is expected.

## Using Enums in Functions

Enums make function signatures clearer:

```zig
fn route(ip_kind: IpAddrKind) void {
    // Handle routing based on IP version
}

route(IpAddrKind.v4);
route(IpAddrKind.v6);
```

## Enums with Integers

By default, enums are not integers, but you can specify an integer backing type:

```zig
const Status = enum(u8) {
    ok = 200,
    not_found = 404,
    server_error = 500,
};

const code: u8 = @intFromEnum(Status.ok);  // 200
```

## Enums vs Structs for Related Data

Initially, you might combine enums with structs:

```zig
const IpAddrKind = enum {
    v4,
    v6,
};

const IpAddr = struct {
    kind: IpAddrKind,
    address: []const u8,
};

const home = IpAddr{
    .kind = .v4,
    .address = "127.0.0.1",
};
```

But Zig has a better way: **tagged unions**.

## Tagged Unions - Enums with Data

A tagged union combines an enum with different data for each variant:

```zig
const IpAddr = union(enum) {
    v4: [4]u8,
    v6: [16]u8,
};

const localhost = IpAddr{ .v4 = .{ 127, 0, 0, 1 } };
const loopback = IpAddr{ .v6 = .{ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1 } };
```

Each variant can store different types and amounts of data!

## Tagged Union Syntax

The `union(enum)` syntax creates a tagged union:

```zig
const Message = union(enum) {
    quit,                           // No data
    move: struct { x: i32, y: i32 }, // Struct data
    write: []const u8,              // String data
    change_color: struct { r: u8, g: u8, b: u8 }, // RGB data
};
```

Create instances:

```zig
const msg1 = Message.quit;
const msg2 = Message{ .move = .{ .x = 10, .y = 20 } };
const msg3 = Message{ .write = "Hello" };
const msg4 = Message{ .change_color = .{ .r = 255, .g = 0, .b = 0 } };
```

## Why Tagged Unions?

Compare these approaches:

**Separate Structs:**
```zig
const QuitMessage = struct {};
const MoveMessage = struct { x: i32, y: i32 };
const WriteMessage = struct { data: []const u8 };
const ChangeColorMessage = struct { r: u8, g: u8, b: u8 };

// Can't write a single function to handle all types!
```

**Tagged Union:**
```zig
const Message = union(enum) {
    quit,
    move: struct { x: i32, y: i32 },
    write: []const u8,
    change_color: struct { r: u8, g: u8, b: u8 },
};

// One function handles all variants!
fn handleMessage(msg: Message) void {
    switch (msg) {
        .quit => std.debug.print("Quitting\n", .{}),
        .move => |coords| std.debug.print("Moving to {d},{d}\n", .{ coords.x, coords.y }),
        .write => |text| std.debug.print("Writing: {s}\n", .{text}),
        .change_color => |rgb| std.debug.print("Color: RGB({d},{d},{d})\n", .{ rgb.r, rgb.g, rgb.b }),
    }
}
```

## Methods on Enums and Tagged Unions

Like structs, you can define methods:

```zig
const Message = union(enum) {
    quit,
    write: []const u8,
    
    pub fn call(self: Message) void {
        switch (self) {
            .quit => std.debug.print("Quit called\n", .{}),
            .write => |text| std.debug.print("Write called: {s}\n", .{text}),
        }
    }
};

const msg = Message{ .write = "hello" };
msg.call();
```

## Complete Example

```zig title="enums_example.zig"
const std = @import("std");

const Coin = enum {
    penny,
    nickel,
    dime,
    quarter,
    
    pub fn valueInCents(self: Coin) u8 {
        return switch (self) {
            .penny => 1,
            .nickel => 5,
            .dime => 10,
            .quarter => 25,
        };
    }
};

const IpAddr = union(enum) {
    v4: [4]u8,
    v6: [16]u8,
    
    pub fn print(self: IpAddr) void {
        switch (self) {
            .v4 => |octets| {
                std.debug.print("IPv4: {d}.{d}.{d}.{d}\n", .{
                    octets[0], octets[1], octets[2], octets[3],
                });
            },
            .v6 => |segments| {
                std.debug.print("IPv6: {any}\n", .{segments});
            },
        }
    }
};

pub fn main() void {
    // Using simple enum
    const coin = Coin.quarter;
    std.debug.print("Value: {d} cents\n", .{coin.valueInCents()});
    
    // Using tagged union
    const localhost = IpAddr{ .v4 = .{ 127, 0, 0, 1 } };
    localhost.print();
    
    const loopback = IpAddr{ .v6 = .{ 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1 } };
    loopback.print();
}
```

## Enum Integer Values

Convert between enums and integers:

```zig
const Color = enum(u8) {
    red = 0,
    green = 1,
    blue = 2,
};

const red_value: u8 = @intFromEnum(Color.red);  // 0
const green: Color = @enumFromInt(1);            // Color.green
```

## Anonymous Tagged Unions

For one-off use, you can create anonymous tagged unions:

```zig
const value = union(enum) {
    int: i32,
    float: f64,
    string: []const u8,
}{ .int = 42 };
```

## Key Takeaways

- **Simple enums** - Just named values
- **Tagged unions** - `union(enum)` for variants with data
- **Each variant** - Can have different data types
- **Methods** - Work on both enums and tagged unions
- **Integer backing** - Optional for simple enums
- **Type safety** - Compiler ensures correct usage

## Next Steps

Now that you can define enums and tagged unions, learn how to work with them using Zig's powerful pattern matching in [Switch and Pattern Matching](02-switch-and-pattern-matching.md).
