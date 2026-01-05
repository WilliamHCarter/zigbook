# 3.4. Comments

Comments are notes in your code that the compiler ignores. They help you and others understand what your code does.

## Line Comments

Use `//` for single-line comments:

```zig
// This is a comment
const x = 5; // This is also a comment
```

## Doc Comments

Use `///` for documentation comments that describe code for users:

```zig
/// Calculates the area of a circle given its radius.
/// Returns the area as a 64-bit float.
pub fn circleArea(radius: f64) f64 {
    return 3.14159 * radius * radius;
}
```

These doc comments are extracted by Zig's documentation generator.

## Top-Level Doc Comments

Use `//!` at the top of a file to document the entire module:

```zig
//! This module provides mathematical utilities for geometry.
//! It includes functions for calculating areas and perimeters.

const std = @import("std");

pub fn circleArea(radius: f64) f64 {
    return 3.14159 * radius * radius;
}
```

## Commenting Best Practices

### Good Comments Explain Why

```zig
// We multiply by 2 because the radius is doubled in the formula
const diameter = radius * 2;
```

### Avoid Obvious Comments

```zig
// Bad: States the obvious
const x = 5; // Set x to 5

// Good: No comment needed - code is clear
const max_retries = 5;
```

### Use Comments for Complex Logic

```zig
// Apply the Luhn algorithm to validate the credit card number
// 1. Double every second digit from right to left
// 2. If doubling results in a two-digit number, add the digits
// 3. Sum all the digits
// 4. If sum is divisible by 10, the number is valid
fn validateCreditCard(number: []const u8) bool {
    // Implementation...
}
```

### Document Public APIs

```zig
/// Reads a line from standard input into the provided buffer.
/// 
/// Returns a slice of the buffer containing the line read (excluding newline).
/// Returns an error if the line is longer than the buffer.
/// 
/// Example:
/// ```zig
/// var buf: [100]u8 = undefined;
/// const line = try readLine(&buf);
/// ```
pub fn readLine(buffer: []u8) ![]u8 {
    // Implementation...
}
```

## Comments vs Self-Documenting Code

Often, good naming makes comments unnecessary:

```zig
// Bad: Needs comment to explain
const d = 86400; // Seconds in a day

// Good: Name explains itself
const seconds_per_day = 86400;
```

## Temporarily Disabling Code

Comments can temporarily disable code during development:

```zig
fn debug() void {
    // std.debug.print("Debug info\n", .{});
    // TODO: Remove this before production
}
```

!!! tip
    Use version control instead of leaving commented-out code. It clutters the codebase and confuses readers.

## TODO and FIXME

Mark areas that need work:

```zig
// TODO: Add input validation
// FIXME: This doesn't handle negative numbers correctly
fn processInput(value: i32) void {
    // Implementation...
}
```

## Multi-Line Explanations

For longer explanations, use multiple line comments:

```zig
// This function implements the A* pathfinding algorithm.
// It finds the shortest path from start to goal using a heuristic
// function to guide the search. The heuristic must be admissible
// (never overestimate the actual cost) for the algorithm to
// guarantee an optimal solution.
fn findPath(start: Point, goal: Point) ![]Point {
    // Implementation...
}
```

## Example: Well-Commented Code

```zig title="temperature.zig"
//! Temperature conversion utilities.
//! Provides functions for converting between Celsius and Fahrenheit.

const std = @import("std");

/// Converts Celsius to Fahrenheit.
/// Formula: F = C × 9/5 + 32
pub fn celsiusToFahrenheit(celsius: f64) f64 {
    return celsius * 9.0 / 5.0 + 32.0;
}

/// Converts Fahrenheit to Celsius.
/// Formula: C = (F - 32) × 5/9
pub fn fahrenheitToCelsius(fahrenheit: f64) f64 {
    return (fahrenheit - 32.0) * 5.0 / 9.0;
}

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    
    // Test with water's freezing point
    const freezing_c = 0.0;
    const freezing_f = celsiusToFahrenheit(freezing_c);
    
    try stdout.print("{d}°C = {d}°F\n", .{ freezing_c, freezing_f });
    
    // Test with water's boiling point
    const boiling_c = 100.0;
    const boiling_f = celsiusToFahrenheit(boiling_c);
    
    try stdout.print("{d}°C = {d}°F\n", .{ boiling_c, boiling_f });
}
```

## When to Comment

**Do comment:**

- Complex algorithms or business logic
- Non-obvious workarounds or optimizations
- Public API documentation
- Why you made a particular decision

**Don't comment:**

- What the code obviously does
- Repeating the code in English
- Leaving old code commented out
- Adding noise without value

## Key Takeaways

- `//` for regular comments
- `///` for doc comments
- `//!` for module-level documentation
- Good names often eliminate the need for comments
- Comment the "why," not the "what"
- Document all public APIs

## Next Steps

Learn about making decisions in your code with [Control Flow](05-control-flow.md).
