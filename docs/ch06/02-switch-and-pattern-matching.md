# 6.2. Switch and Pattern Matching

Zig's `switch` statement is an extremely powerful control flow construct. It's perfect for working with enums and ensures you handle all possible cases.

## Basic Switch with Enums

Use `switch` to handle each enum variant:

```zig
const Coin = enum {
    penny,
    nickel,
    dime,
    quarter,
};

fn valueInCents(coin: Coin) u8 {
    return switch (coin) {
        .penny => 1,
        .nickel => 5,
        .dime => 10,
        .quarter => 25,
    };
}
```

The `switch` expression:
- Must handle **all** variants (exhaustive)
- Returns a value (it's an expression)
- Uses `.variant` shorthand inside

## Multi-line Arms

For complex logic, use blocks:

```zig
fn valueInCents(coin: Coin) u8 {
    return switch (coin) {
        .penny => {
            std.debug.print("Lucky penny!\n", .{});
            return 1;
        },
        .nickel => 5,
        .dime => 10,
        .quarter => 25,
    };
}
```

## Extracting Values from Tagged Unions

Use `|variable|` to capture the data:

```zig
const IpAddr = union(enum) {
    v4: [4]u8,
    v6: [16]u8,
};

fn printIp(addr: IpAddr) void {
    switch (addr) {
        .v4 => |octets| {
            std.debug.print("{d}.{d}.{d}.{d}\n", .{
                octets[0], octets[1], octets[2], octets[3],
            });
        },
        .v6 => |segments| {
            std.debug.print("IPv6: {any}\n", .{segments});
        },
    }
}
```

The `|octets|` syntax **captures** the data inside the variant.

## Exhaustiveness Checking

Zig requires you to handle **all** cases:

```zig
// This won't compile - missing .quarter!
fn valueInCents(coin: Coin) u8 {
    return switch (coin) {
        .penny => 1,
        .nickel => 5,
        .dime => 10,
        // Error: switch must handle all cases
    };
}
```

The compiler tells you exactly which cases are missing.

## The Else Clause

Use `else` to handle remaining cases:

```zig
fn isValuable(coin: Coin) bool {
    return switch (coin) {
        .quarter => true,
        else => false,
    };
}
```

But be careful - `else` can hide bugs if you add new variants later!

## Switching on Integers

`switch` works with any type:

```zig
fn describe(number: u8) []const u8 {
    return switch (number) {
        0 => "zero",
        1 => "one",
        2, 3, 5, 7 => "small prime",
        4, 6, 8, 9 => "small composite",
        else => "larger number",
    };
}
```

## Range Patterns

Use `...` for ranges:

```zig
fn categorize(score: u8) []const u8 {
    return switch (score) {
        0...59 => "F",
        60...69 => "D",
        70...79 => "C",
        80...89 => "B",
        90...100 => "A",
        else => "invalid",
    };
}
```

## Complex Example: State Machine

```zig
const State = union(enum) {
    idle,
    running: struct { speed: u32 },
    paused: struct { elapsed: u32 },
    error_state: []const u8,
};

fn handleState(state: State) void {
    switch (state) {
        .idle => {
            std.debug.print("System is idle\n", .{});
        },
        .running => |info| {
            std.debug.print("Running at speed {d}\n", .{info.speed});
        },
        .paused => |info| {
            std.debug.print("Paused after {d}ms\n", .{info.elapsed});
        },
        .error_state => |message| {
            std.debug.print("Error: {s}\n", .{message});
        },
    }
}
```

## Nested Patterns

You can match on nested data:

```zig
const Point = struct { x: i32, y: i32 };

const Location = union(enum) {
    origin,
    point: Point,
};

fn describe(loc: Location) []const u8 {
    return switch (loc) {
        .origin => "at origin",
        .point => |p| {
            if (p.x == 0 and p.y == 0) {
                return "also at origin";
            } else {
                return "somewhere else";
            }
        },
    };
}
```

## Complete Example: Expression Evaluator

```zig title="switch_example.zig"
const std = @import("std");

const BinaryOp = enum {
    add,
    subtract,
    multiply,
    divide,
};

const Expr = union(enum) {
    number: i32,
    binary: struct {
        op: BinaryOp,
        left: *const Expr,
        right: *const Expr,
    },
};

fn eval(expr: Expr) i32 {
    return switch (expr) {
        .number => |n| n,
        .binary => |bin| {
            const left = eval(bin.left.*);
            const right = eval(bin.right.*);
            return switch (bin.op) {
                .add => left + right,
                .subtract => left - right,
                .multiply => left * right,
                .divide => @divTrunc(left, right),
            };
        },
    };
}

pub fn main() void {
    // 5 + 3
    const five = Expr{ .number = 5 };
    const three = Expr{ .number = 3 };
    const add = Expr{
        .binary = .{
            .op = .add,
            .left = &five,
            .right = &three,
        },
    };
    
    std.debug.print("Result: {d}\n", .{eval(add)});
}
```

## Inline Switch

For performance-critical code, use `inline` switch:

```zig
fn handleInline(comptime T: type, value: T) void {
    inline switch (T) {
        i32 => std.debug.print("32-bit integer: {d}\n", .{value}),
        f64 => std.debug.print("64-bit float: {d}\n", .{value}),
        else => @compileError("Unsupported type"),
    }
}
```

## Pointer Capture

Capture by pointer to modify the data:

```zig
const Message = union(enum) {
    text: []const u8,
    number: i32,
};

fn modify(msg: *Message) void {
    switch (msg.*) {
        .number => |*n| {
            n.* += 10;  // Modify the number
        },
        .text => {},
    }
}
```

## Switch vs If-Else

**Use switch when:**
- Working with enums or fixed values
- You want exhaustiveness checking
- The code is clearer as a match

**Use if-else when:**
- Conditions are complex boolean expressions
- Cases aren't mutually exclusive
- You have open-ended conditions

## Key Takeaways

- **switch is exhaustive** - Must handle all cases
- **Expressions** - switch returns a value
- **Capture values** - Use `|variable|` syntax
- **Ranges** - Use `...` for range patterns
- **else clause** - Handles remaining cases
- **Type safe** - Compiler ensures correctness

## Next Steps

Switch is powerful for enums, but what about values that might not exist? Learn about Zig's approach to null safety in [Optional Types and Null Safety](03-optional-types-and-null-safety.md).
