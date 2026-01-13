# Chapter 8: Writing Automated Tests

Tests are an essential part of software development. They give you confidence that your code works correctly and continues to work as you make changes.

In this chapter, we'll explore Zig's built-in testing framework. Unlike many languages that require external test frameworks, Zig includes testing as a first-class language feature.

## What You'll Learn

We'll cover:

1. **How to Write Tests** - Test syntax and assertions
2. **Controlling How Tests Are Run** - Filtering and running specific tests
3. **Test Organization** - Structuring tests in your codebase

## Why Test?

Automated tests provide:

- **Confidence** - Know your code works correctly
- **Documentation** - Tests show how code should be used
- **Regression prevention** - Catch bugs when refactoring
- **Design feedback** - Hard-to-test code is often poorly designed
- **Faster development** - Catch bugs early, not in production

## Zig's Testing Philosophy

Zig's approach to testing:

- **Built into the language** - No external frameworks needed
- **Compile-time execution** - Tests can run at compile time
- **Fast** - Parallel test execution by default
- **Simple** - Just write functions with `test` keyword
- **Integrated** - Tests live alongside your code

## A Quick Preview

Here's what a Zig test looks like:

```zig
const std = @import("std");

fn add(a: i32, b: i32) i32 {
    return a + b;
}

test "addition works" {
    const result = add(2, 3);
    try std.testing.expectEqual(5, result);
}

test "addition is commutative" {
    try std.testing.expectEqual(add(2, 3), add(3, 2));
}
```

Run tests with:
```bash
zig test myfile.zig
```

Output:
```
All 2 tests passed.
```

## Test-Driven Development

A common workflow:

1. **Write a failing test** - Define the expected behavior
2. **Write minimal code** - Make the test pass
3. **Refactor** - Clean up while tests ensure correctness
4. **Repeat** - Build features incrementally

This "red, green, refactor" cycle leads to well-tested, maintainable code.

## Types of Tests

**Unit tests** - Test individual functions:
```zig
test "divide by two" {
    try std.testing.expectEqual(5, divide(10, 2));
}
```

**Integration tests** - Test components working together:
```zig
test "user authentication flow" {
    const user = try createUser("alice", "password123");
    try authenticateUser("alice", "password123");
}
```

**Property-based tests** - Test properties hold for many inputs:
```zig
test "addition is commutative for many values" {
    var i: i32 = -100;
    while (i < 100) : (i += 1) {
        var j: i32 = -100;
        while (j < 100) : (j += 1) {
            try std.testing.expectEqual(add(i, j), add(j, i));
        }
    }
}
```

## Getting Started

Ready to write bulletproof code with tests? Let's begin with [How to Write Tests](01-how-to-write-tests.md).
