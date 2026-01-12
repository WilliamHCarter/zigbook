# Chapter 7: Error Handling

Errors are a fact of programming life. Files might not exist, network connections fail, memory runs out. The question isn't whether errors will happen - it's how you'll handle them.

In this chapter, we'll explore Zig's approach to error handling. Unlike exceptions (which hide control flow) or error codes (which are easy to ignore), Zig treats errors as **values** that are handled explicitly in the type system.

## What You'll Learn

We'll cover:

1. **Errors are Values** - Defining and returning errors
2. **Propagating Errors with try** - Passing errors up the call stack
3. **Error Handling Strategies** - When to catch, when to propagate

## Zig's Error Philosophy

Zig's error handling is designed around these principles:

- **Explicit** - Errors are visible in function signatures
- **Lightweight** - No stack unwinding or hidden costs
- **Compile-time checked** - The type system ensures you handle errors
- **Composable** - Easy to propagate or transform errors

## A Quick Preview

Here's what error handling looks like in Zig:

```zig
// Function that can return an error
fn divide(a: i32, b: i32) !i32 {
    if (b == 0) {
        return error.DivisionByZero;
    }
    return @divTrunc(a, b);
}

// Propagating errors with try
fn calculate() !i32 {
    const result = try divide(10, 2);  // Returns error if divide fails
    return result * 2;
}

// Handling errors with catch
fn safeCalculate() i32 {
    const result = divide(10, 0) catch |err| {
        std.debug.print("Error: {}\n", .{err});
        return 0;
    };
    return result;
}
```

## Error Handling Approaches

Different languages take different approaches:

**Exceptions (Java, Python, C++):**
- Hidden control flow
- Stack unwinding overhead
- Easy to accidentally ignore

**Error codes (C, Go):**
- Must check manually
- Easy to forget to check
- Verbose error handling

**Result types (Rust):**
- Compile-time checked
- Explicit in type signatures
- Composable with combinators

**Zig's approach:**
- Error union types (`!T`)
- Explicit with `try` and `catch`
- No hidden control flow
- Zero runtime overhead

## Why This Matters

Good error handling:
- Makes your code more reliable
- Helps users understand what went wrong
- Prevents silent failures
- Guides you toward correct code

Bad error handling:
- Crashes your program
- Loses important information
- Makes debugging difficult
- Frustrates users

## Getting Started

Ready to write robust, error-free code? Let's begin with [Errors are Values](01-errors-are-values.md).
