# 3.5. Control Flow

Control flow determines the order in which code executes. Zig provides several constructs for controlling program flow.

## If Expressions

The `if` expression evaluates a condition and executes code based on the result:

```zig
const temperature = 25;

if (temperature > 30) {
    std.debug.print("It's hot!\n", .{});
} else if (temperature > 20) {
    std.debug.print("It's warm.\n", .{});
} else {
    std.debug.print("It's cool.\n", .{});
}
```

### If as an Expression

In Zig, `if` is an expression that returns a value:

```zig
const max = if (a > b) a else b;

const status = if (score >= 60) "pass" else "fail";
```

### If with Optionals

Use `if` to unwrap optional values:

```zig
const maybe_value: ?i32 = 42;

if (maybe_value) |value| {
    std.debug.print("Value is {d}\n", .{value});
} else {
    std.debug.print("No value\n", .{});
}
```

### If with Error Unions

Similarly, `if` can unwrap error unions:

```zig
if (divide(10, 2)) |result| {
    std.debug.print("Result: {d}\n", .{result});
} else |err| {
    std.debug.print("Error: {}\n", .{err});
}
```

## While Loops

The `while` loop repeats code while a condition is true:

```zig
var count: u32 = 0;

while (count < 5) {
    std.debug.print("Count: {d}\n", .{count});
    count += 1;
}
```

### While with Continue Expression

Execute code after each iteration:

```zig
var i: u32 = 0;

while (i < 10) : (i += 1) {
    if (i % 2 == 0) {
        std.debug.print("{d} is even\n", .{i});
    }
}
```

### While with Break and Continue

Control loop execution:

```zig
var i: u32 = 0;

while (i < 10) : (i += 1) {
    if (i == 3) continue; // Skip 3
    if (i == 7) break;    // Stop at 7
    
    std.debug.print("{d}\n", .{i});
}
// Prints: 0, 1, 2, 4, 5, 6
```

### While with Optionals

Loop while unwrapping optionals:

```zig
var value: ?u32 = 10;

while (value) |v| {
    std.debug.print("{d}\n", .{v});
    value = if (v > 0) v - 1 else null;
}
```

## For Loops

Use `for` to iterate over arrays and slices:

```zig
const numbers = [_]i32{ 1, 2, 3, 4, 5 };

for (numbers) |num| {
    std.debug.print("{d}\n", .{num});
}
```

### For with Index

Get the index and value:

```zig
for (numbers, 0..) |num, i| {
    std.debug.print("numbers[{d}] = {d}\n", .{ i, num });
}
```

### For with Range

Iterate over a range:

```zig
for (0..5) |i| {
    std.debug.print("{d}\n", .{i});
}
// Prints: 0, 1, 2, 3, 4
```

### For with Multiple Arrays

Iterate over multiple arrays in parallel:

```zig
const a = [_]i32{ 1, 2, 3 };
const b = [_]i32{ 4, 5, 6 };

for (a, b) |x, y| {
    std.debug.print("{d} + {d} = {d}\n", .{ x, y, x + y });
}
```

## Switch Expressions

Use `switch` for multiple conditions:

```zig
const day = 3;

const day_name = switch (day) {
    1 => "Monday",
    2 => "Tuesday",
    3 => "Wednesday",
    4 => "Thursday",
    5 => "Friday",
    6, 7 => "Weekend",
    else => "Invalid",
};
```

### Switch Must Be Exhaustive

Switch expressions must handle all possible cases:

```zig
const color = enum { red, green, blue };

const value = switch (color.red) {
    .red => 0xFF0000,
    .green => 0x00FF00,
    .blue => 0x0000FF,
    // No 'else' needed - all cases covered
};
```

### Switch with Ranges

```zig
const score = 85;

const grade = switch (score) {
    90...100 => 'A',
    80...89 => 'B',
    70...79 => 'C',
    60...69 => 'D',
    else => 'F',
};
```

### Switch with Multiple Values

```zig
const result = switch (value) {
    1, 2, 3 => "low",
    4, 5, 6 => "medium",
    7, 8, 9 => "high",
    else => "unknown",
};
```

## Defer

Execute code when leaving the current scope:

```zig
{
    defer std.debug.print("Goodbye!\n", .{});
    std.debug.print("Hello!\n", .{});
    // "Goodbye!" prints here when scope exits
}
```

### Multiple Defers

Defers execute in reverse order (LIFO):

```zig
{
    defer std.debug.print("1\n", .{});
    defer std.debug.print("2\n", .{});
    defer std.debug.print("3\n", .{});
}
// Prints: 3, 2, 1
```

### Defer for Cleanup

Perfect for resource cleanup:

```zig
fn processFile(path: []const u8) !void {
    const file = try std.fs.cwd().openFile(path, .{});
    defer file.close(); // Always close, even on error
    
    // Use file...
}
```

## Errdefer

Like `defer`, but only executes on error:

```zig
fn createResource() !Resource {
    const resource = try allocateResource();
    errdefer freeResource(resource); // Only free if an error occurs
    
    try initializeResource(resource);
    return resource;
}
```

## Example: Control Flow in Action

```zig title="control_flow.zig"
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    
    // If expression
    const age = 25;
    const status = if (age >= 18) "adult" else "minor";
    try stdout.print("Status: {s}\n", .{status});
    
    // For loop with range
    try stdout.print("Counting: ", .{});
    for (0..5) |i| {
        try stdout.print("{d} ", .{i});
    }
    try stdout.print("\n", .{});
    
    // While loop with continue expression
    var sum: u32 = 0;
    var i: u32 = 1;
    while (i <= 10) : (i += 1) {
        sum += i;
    }
    try stdout.print("Sum of 1-10: {d}\n", .{sum});
    
    // Switch expression
    const day = 3;
    const day_type = switch (day) {
        1...5 => "weekday",
        6, 7 => "weekend",
        else => "invalid",
    };
    try stdout.print("Day {d} is a {s}\n", .{ day, day_type });
    
    // Defer example
    {
        defer try stdout.print("Cleaning up!\n", .{});
        try stdout.print("Doing work...\n", .{});
    }
}
```

## Labeled Blocks and Loops

Break or continue outer loops using labels:

```zig
outer: for (0..5) |i| {
    for (0..5) |j| {
        if (i * j > 6) {
            break :outer; // Break from outer loop
        }
        std.debug.print("{d} ", .{i * j});
    }
}
```

## Key Takeaways

- `if` is an expression that can return values
- `while` loops with optional continue expressions
- `for` loops for iterating over arrays, slices, and ranges
- `switch` must be exhaustive and handle all cases
- `defer` runs code when leaving scope
- `errdefer` runs cleanup code only on errors
- `break` and `continue` control loop flow

## Next Steps

Congratulations! You've completed Chapter 3 and learned the fundamental building blocks of Zig programming. You now understand:

- Variables and mutability
- Data types
- Functions
- Comments
- Control flow

You're ready to explore more advanced topics as the book continues. Keep practicing these concepts - they're the foundation of everything you'll build in Zig!
