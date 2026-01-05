# 3.2. Data Types

Zig is a statically typed language, meaning every value has a type known at compile time. Let's explore the built-in types.

## Integer Types

Zig provides integer types of various sizes, both signed and unsigned:

| Type   | Size    | Range                    |
|--------|---------|--------------------------|
| `i8`   | 8 bits  | -128 to 127              |
| `u8`   | 8 bits  | 0 to 255                 |
| `i16`  | 16 bits | -32,768 to 32,767        |
| `u16`  | 16 bits | 0 to 65,535              |
| `i32`  | 32 bits | -2,147,483,648 to 2,147,483,647 |
| `u32`  | 32 bits | 0 to 4,294,967,295       |
| `i64`  | 64 bits | Large negative to positive |
| `u64`  | 64 bits | 0 to very large number   |

You can also use arbitrary bit-width integers:

```zig
const a: i3 = -4;   // 3-bit signed integer
const b: u7 = 127;  // 7-bit unsigned integer
```

### Special Integer Types

- `isize` / `usize` - Pointer-sized integers (match the target architecture)
- `comptime_int` - Arbitrary precision integer available at compile time

## Floating-Point Types

Zig supports IEEE-754 floating-point numbers:

```zig
const pi: f32 = 3.14;       // 32-bit float
const e: f64 = 2.71828;     // 64-bit float
const big: f128 = 1.23;     // 128-bit float
```

## Boolean Type

The `bool` type has two values: `true` and `false`:

```zig
const is_zig_awesome: bool = true;
const is_complete: bool = false;
```

## Character and String Types

Zig doesn't have a separate character type. Single characters are just integers (usually `u8`):

```zig
const letter: u8 = 'A'; // ASCII value 65
```

String literals are arrays of `u8`:

```zig
const greeting: []const u8 = "Hello, Zig!";
```

## Arrays

Arrays have a fixed size known at compile time:

```zig
const numbers: [5]i32 = [5]i32{ 1, 2, 3, 4, 5 };
const letters = [_]u8{ 'a', 'b', 'c' }; // Size inferred
```

Access elements with indexing:

```zig
const first = numbers[0]; // 1
const second = numbers[1]; // 2
```

## Slices

Slices are views into arrays or memory:

```zig
const array = [_]i32{ 1, 2, 3, 4, 5 };
const slice: []const i32 = array[1..4]; // Elements 1, 2, 3
```

## Pointers

Pointers store memory addresses:

```zig
var x: i32 = 5;
const ptr: *i32 = &x;  // Pointer to x
ptr.* = 10;             // Dereference and modify
```

## Optional Types

Optional types can be `null` or a value:

```zig
const maybe_number: ?i32 = 42;
const nothing: ?i32 = null;
```

Test for null with `if`:

```zig
if (maybe_number) |value| {
    // Use value here
} else {
    // Handle null case
}
```

## Error Union Types

Combine errors with values:

```zig
const result: !i32 = error.SomethingWentWrong;
const ok: !i32 = 42;
```

## Structs

Group related data:

```zig
const Point = struct {
    x: f64,
    y: f64,
};

const origin = Point{ .x = 0.0, .y = 0.0 };
```

## Enums

Define a type with a fixed set of values:

```zig
const Color = enum {
    red,
    green,
    blue,
};

const favorite = Color.blue;
```

## Type Coercion

Zig is strict about types but allows safe coercions:

```zig
const a: u8 = 250;
const b: u16 = a; // OK: u8 fits in u16

const c: i32 = 100;
const d: i64 = c; // OK: i32 fits in i64
```

Unsafe coercions require explicit casting:

```zig
const x: i32 = -1;
const y: u32 = @intCast(x); // Explicit cast required
```

## Example: Working with Types

```zig title="types.zig"
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    
    // Integers
    const age: u8 = 25;
    try stdout.print("Age: {d}\n", .{age});
    
    // Floats
    const temperature: f32 = 23.5;
    try stdout.print("Temperature: {d:.1}°C\n", .{temperature});
    
    // Booleans
    const is_sunny: bool = true;
    try stdout.print("Is sunny: {}\n", .{is_sunny});
    
    // Arrays
    const scores = [_]i32{ 95, 87, 92 };
    try stdout.print("First score: {d}\n", .{scores[0]});
    
    // Optionals
    const maybe: ?i32 = 42;
    if (maybe) |value| {
        try stdout.print("Value: {d}\n", .{value});
    }
}
```

## Key Takeaways

- Zig has explicit integer types with specific bit widths
- Arrays have fixed sizes, slices are flexible views
- Optional and error types make handling edge cases explicit
- Type safety prevents many common bugs

## Next Steps

Learn how to organize code with [Functions](03-functions.md).
