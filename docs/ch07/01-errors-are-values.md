# 7.1. Errors as Values

In Zig, errors aren't exceptions that get thrown. They're **values** that get returned, just like any other value. This makes error handling explicit and predictable.

## Defining Errors

Define an error set with specific error values:

```zig
const FileError = error{
    FileNotFound,
    PermissionDenied,
    DiskFull,
};
```

Each name is a distinct error value. Error names must be in PascalCase.

## Error Union Types

A function that can return an error uses the `!` operator:

```zig
fn openFile(path: []const u8) !File {
    // Can return File OR an error
}
```

The type `!File` is an **error union type** - it's either an error or a `File`.

## Returning Errors

Return errors like any other value:

```zig
fn divide(a: i32, b: i32) !i32 {
    if (b == 0) {
        return error.DivisionByZero;
    }
    return @divTrunc(a, b);
}
```

Notice:
- Return type is `!i32` (can error)
- We return `error.DivisionByZero` for errors
- We return a normal `i32` for success

## Inferred Error Sets

You don't have to declare error sets explicitly. Zig can infer them:

```zig
fn divide(a: i32, b: i32) !i32 {
    if (b == 0) {
        return error.DivisionByZero;  // Error inferred
    }
    return @divTrunc(a, b);
}
```

The error set is `error{DivisionByZero}`, inferred from the return statement.

## Explicit Error Sets

For better documentation, specify the error set:

```zig
const MathError = error{
    DivisionByZero,
    Overflow,
    Underflow,
};

fn divide(a: i32, b: i32) MathError!i32 {
    if (b == 0) {
        return error.DivisionByZero;
    }
    return @divTrunc(a, b);
}
```

Now the function signature documents which errors are possible.

## Catching Errors

Use `catch` to handle errors:

```zig
const result = divide(10, 0) catch |err| {
    std.debug.print("Error: {}\n", .{err});
    return 0;
};
```

The `|err|` syntax captures the error value.

## Providing Default Values

For simple cases, provide a default:

```zig
const result = divide(10, 0) catch 0;
```

This returns `0` if an error occurs.

## Checking Which Error

Use `switch` to handle different errors:

```zig
const result = divide(10, 0) catch |err| switch (err) {
    error.DivisionByZero => {
        std.debug.print("Cannot divide by zero!\n", .{});
        return 0;
    },
    error.Overflow => {
        std.debug.print("Result too large!\n", .{});
        return maxInt(i32);
    },
};
```

## Errors are Enums

Under the hood, errors are just enum values:

```zig
const err = error.FileNotFound;
const name = @errorName(err);  // "FileNotFound"
```

This makes them lightweight - they're just integers at runtime.

## Anyerror Type

The `anyerror` type can hold any error:

```zig
fn handleError(err: anyerror) void {
    std.debug.print("Got error: {}\n", .{err});
}
```

Use this when you don't know which errors are possible.

## Error Return Traces

In debug mode, Zig captures stack traces for errors:

```zig
fn inner() !void {
    return error.SomethingWrong;
}

fn outer() !void {
    try inner();  // Stack trace captured here
}
```

This helps debug where errors originated.

## Complete Example

```zig title="errors_basic.zig"
const std = @import("std");

const ParseError = error{
    InvalidCharacter,
    Overflow,
};

fn parseUnsigned(str: []const u8) ParseError!u32 {
    if (str.len == 0) {
        return error.InvalidCharacter;
    }
    
    var result: u32 = 0;
    for (str) |c| {
        if (c < '0' or c > '9') {
            return error.InvalidCharacter;
        }
        
        const digit = c - '0';
        
        // Check for overflow
        if (result > (@as(u32, std.math.maxInt(u32)) - digit) / 10) {
            return error.Overflow;
        }
        
        result = result * 10 + digit;
    }
    
    return result;
}

pub fn main() void {
    // Success case
    const num1 = parseUnsigned("123") catch |err| {
        std.debug.print("Error parsing '123': {}\n", .{err});
        return;
    };
    std.debug.print("Parsed: {d}\n", .{num1});
    
    // Error case - invalid character
    const num2 = parseUnsigned("12a3") catch |err| {
        std.debug.print("Error parsing '12a3': {}\n", .{err});
        return;
    };
    std.debug.print("Parsed: {d}\n", .{num2});
}
```

Output:
```
Parsed: 123
Error parsing '12a3': error.InvalidCharacter
```

## Error Sets are Types

Error sets are first-class types:

```zig
const NetworkError = error{
    ConnectionRefused,
    Timeout,
    HostUnreachable,
};

const FileError = error{
    FileNotFound,
    PermissionDenied,
};

// Union of error sets
const AppError = NetworkError || FileError;
```

The `||` operator combines error sets.

## Error Set Subsets

One error set can be a subset of another:

```zig
const BigErrorSet = error{
    ErrorA,
    ErrorB,
    ErrorC,
};

const SmallErrorSet = error{
    ErrorA,
    ErrorB,
};

fn process() BigErrorSet!void {
    try mayFail();  // SmallErrorSet coerces to BigErrorSet
}

fn mayFail() SmallErrorSet!void {
    return error.ErrorA;
}
```

## Unused Error Values

The compiler warns about unused error union values:

```zig
divide(10, 2);  // Warning: error union ignored
```

You must either:
- Use `try` to propagate
- Use `catch` to handle
- Explicitly discard with `_ =`

```zig
_ = divide(10, 2);  // Explicitly ignore errors (not recommended)
```

## Key Takeaways

- **Errors are values** - Returned, not thrown
- **!T syntax** - Error union type
- **catch** - Handle errors with default or logic
- **Inferred or explicit** - Error sets can be both
- **Lightweight** - Just integers at runtime
- **Compile-time checked** - Can't ignore errors
- **Stack traces** - Captured in debug mode

## Next Steps

Now that you understand error basics, learn how to propagate errors through your call stack efficiently with [Propagating Errors with try](02-propagating-errors-with-try.md).
