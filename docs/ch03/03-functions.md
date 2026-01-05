# 3.3. Functions

Functions are the primary way to organize and reuse code in Zig. Let's explore how they work.

## Declaring Functions

Here's the basic syntax for a function:

```zig
fn functionName(parameter: Type) ReturnType {
    // function body
    return value;
}
```

## A Simple Function

```zig
fn add(a: i32, b: i32) i32 {
    return a + b;
}
```

Call it like this:

```zig
const result = add(5, 3); // result is 8
```

## Functions with No Return Value

Use `void` when a function doesn't return anything:

```zig
fn greet(name: []const u8) void {
    std.debug.print("Hello, {s}!\n", .{name});
}
```

## Functions That Can Error

Use `!` before the return type for functions that might fail:

```zig
fn divide(a: i32, b: i32) !i32 {
    if (b == 0) {
        return error.DivisionByZero;
    }
    return @divTrunc(a, b);
}
```

Call with `try` to propagate errors:

```zig
const result = try divide(10, 2);
```

Or handle the error:

```zig
const result = divide(10, 0) catch |err| {
    std.debug.print("Error: {}\n", .{err});
    return;
};
```

## Multiple Parameters

Functions can take multiple parameters:

```zig
fn calculateArea(width: f64, height: f64) f64 {
    return width * height;
}

const area = calculateArea(5.0, 3.0);
```

## Function Visibility

Use `pub` to make functions accessible from other files:

```zig
pub fn publicFunction() void {
    // Can be called from other files
}

fn privateFunction() void {
    // Only accessible in this file
}
```

## The Main Function

Every Zig program needs a `main` function:

```zig
pub fn main() !void {
    // Program starts here
}
```

The `!void` return type means it can return an error or nothing.

## Returning Multiple Values

Use structs or tuples to return multiple values:

```zig
fn divMod(a: i32, b: i32) struct { quotient: i32, remainder: i32 } {
    return .{
        .quotient = @divTrunc(a, b),
        .remainder = @mod(a, b),
    };
}

const result = divMod(17, 5);
// result.quotient is 3
// result.remainder is 2
```

## Anonymous Functions (Closures)

Zig supports anonymous functions for simple cases:

```zig
const double = struct {
    fn call(x: i32) i32 {
        return x * 2;
    }
}.call;

const result = double(5); // 10
```

## Generic Functions

Use `anytype` for generic parameters:

```zig
fn max(a: anytype, b: @TypeOf(a)) @TypeOf(a) {
    return if (a > b) a else b;
}

const result1 = max(5, 10);      // Works with i32
const result2 = max(3.14, 2.71); // Works with f64
```

## Inline Functions

Mark functions as `inline` to hint at inlining:

```zig
inline fn square(x: i32) i32 {
    return x * x;
}
```

## Example Program

```zig title="functions.zig"
const std = @import("std");

fn celsius_to_fahrenheit(celsius: f64) f64 {
    return celsius * 9.0 / 5.0 + 32.0;
}

fn fahrenheit_to_celsius(fahrenheit: f64) f64 {
    return (fahrenheit - 32.0) * 5.0 / 9.0;
}

fn print_temperature(temp: f64, scale: []const u8) !void {
    const stdout = std.io.getStdOut().writer();
    try stdout.print("{d:.1}°{s}\n", .{ temp, scale });
}

pub fn main() !void {
    const celsius = 25.0;
    const fahrenheit = celsius_to_fahrenheit(celsius);
    
    try print_temperature(celsius, "C");
    try print_temperature(fahrenheit, "F");
    
    const back_to_celsius = fahrenheit_to_celsius(fahrenheit);
    try print_temperature(back_to_celsius, "C");
}
```

## Function Pointers

You can store functions in variables:

```zig
const MathOp = *const fn (i32, i32) i32;

fn add(a: i32, b: i32) i32 {
    return a + b;
}

fn multiply(a: i32, b: i32) i32 {
    return a * b;
}

pub fn main() void {
    var op: MathOp = add;
    const result1 = op(5, 3); // 8
    
    op = multiply;
    const result2 = op(5, 3); // 15
}
```

## Key Takeaways

- Functions are declared with `fn` keyword
- Use `pub` for public functions
- `!` in return type indicates possible errors
- Use `try` to propagate errors, `catch` to handle them
- Functions can be generic with `anytype`
- Return multiple values using structs

## Next Steps

Learn how to document your code with [Comments](04-comments.md).
