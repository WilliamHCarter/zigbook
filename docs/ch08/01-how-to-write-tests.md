# 8.1. How to Write Tests

Writing tests in Zig is straightforward. Use the `test` keyword to define a test, and the `std.testing` module provides assertions to verify behavior.

## Basic Test Syntax

Define a test with the `test` keyword followed by a name:

```zig
const std = @import("std");

test "simple test" {
    const result = 2 + 2;
    try std.testing.expectEqual(4, result);
}
```

Run it:
```bash
$ zig test example.zig
All 1 tests passed.
```

## Test Names

Test names should describe what they're testing:

```zig
test "addition of positive numbers" {
    try std.testing.expectEqual(5, add(2, 3));
}

test "addition of negative numbers" {
    try std.testing.expectEqual(-5, add(-2, -3));
}

test "addition with zero" {
    try std.testing.expectEqual(5, add(5, 0));
}
```

Good test names make failures easier to understand.

## Common Assertions

The `std.testing` module provides many assertions:

### expectEqual

Check two values are equal:

```zig
test "expectEqual" {
    try std.testing.expectEqual(42, 42);
    try std.testing.expectEqual(@as(i32, 10), calculate());
}
```

**Important:** Types must match exactly! Use `@as` to cast if needed.

### expect

Check a boolean condition:

```zig
test "expect" {
    try std.testing.expect(5 > 3);
    try std.testing.expect(isEven(4));
}
```

### expectError

Verify a function returns a specific error:

```zig
test "division by zero returns error" {
    try std.testing.expectError(error.DivisionByZero, divide(10, 0));
}
```

### expectEqualStrings

Compare strings:

```zig
test "string comparison" {
    try std.testing.expectEqualStrings("hello", "hello");
    
    const name = getName();
    try std.testing.expectEqualStrings("Alice", name);
}
```

### expectEqualSlices

Compare slices:

```zig
test "slice comparison" {
    const expected = [_]i32{ 1, 2, 3 };
    const actual = [_]i32{ 1, 2, 3 };
    try std.testing.expectEqualSlices(i32, &expected, &actual);
}
```

### expectApproxEqAbs

Compare floating-point numbers with tolerance:

```zig
test "floating point comparison" {
    const result = 0.1 + 0.2;
    try std.testing.expectApproxEqAbs(0.3, result, 0.0001);
}
```

## Testing Functions

Let's test a real function:

```zig title="math.zig"
const std = @import("std");

pub fn add(a: i32, b: i32) i32 {
    return a + b;
}

pub fn multiply(a: i32, b: i32) i32 {
    return a * b;
}

test "add function" {
    try std.testing.expectEqual(5, add(2, 3));
    try std.testing.expectEqual(0, add(-5, 5));
    try std.testing.expectEqual(-10, add(-5, -5));
}

test "multiply function" {
    try std.testing.expectEqual(6, multiply(2, 3));
    try std.testing.expectEqual(0, multiply(5, 0));
    try std.testing.expectEqual(25, multiply(-5, -5));
}
```

## Testing Error Handling

Verify error cases thoroughly:

```zig
const DivisionError = error{
    DivisionByZero,
    Overflow,
};

fn divide(a: i32, b: i32) DivisionError!i32 {
    if (b == 0) return error.DivisionByZero;
    if (a == std.math.minInt(i32) and b == -1) return error.Overflow;
    return @divTrunc(a, b);
}

test "divide success cases" {
    try std.testing.expectEqual(5, try divide(10, 2));
    try std.testing.expectEqual(-3, try divide(9, -3));
}

test "divide by zero" {
    try std.testing.expectError(error.DivisionByZero, divide(10, 0));
}

test "divide overflow" {
    try std.testing.expectError(
        error.Overflow,
        divide(std.math.minInt(i32), -1)
    );
}
```

## Testing Allocations

Use `std.testing.allocator` which detects memory leaks:

```zig
test "string duplication" {
    const allocator = std.testing.allocator;
    
    const original = "hello";
    const copy = try allocator.dupe(u8, original);
    defer allocator.free(copy);
    
    try std.testing.expectEqualStrings(original, copy);
}

test "forgot to free - will fail!" {
    const allocator = std.testing.allocator;
    
    // Bug: not freeing memory!
    const data = try allocator.alloc(u8, 100);
    _ = data;
    
    // Test will fail with: "memory leak detected"
}
```

The testing allocator automatically fails tests that leak memory!

## Testing with Setup and Teardown

Use regular Zig code for setup/teardown:

```zig
const Database = struct {
    data: std.ArrayList(i32),
    
    fn init(allocator: std.mem.Allocator) !Database {
        return Database{
            .data = std.ArrayList(i32).init(allocator),
        };
    }
    
    fn deinit(self: *Database) void {
        self.data.deinit();
    }
    
    fn add(self: *Database, value: i32) !void {
        try self.data.append(value);
    }
};

test "database add" {
    // Setup
    var db = try Database.init(std.testing.allocator);
    defer db.deinit();  // Teardown
    
    // Test
    try db.add(42);
    try std.testing.expectEqual(@as(usize, 1), db.data.items.len);
    try std.testing.expectEqual(42, db.data.items[0]);
}
```

## Testing with Loops

Test multiple cases efficiently:

```zig
test "is_even for multiple values" {
    const test_cases = [_]struct { input: i32, expected: bool }{
        .{ .input = 0, .expected = true },
        .{ .input = 1, .expected = false },
        .{ .input = 2, .expected = true },
        .{ .input = -2, .expected = true },
        .{ .input = -1, .expected = false },
    };
    
    for (test_cases) |case| {
        try std.testing.expectEqual(case.expected, isEven(case.input));
    }
}
```

## Complete Example: Stack Implementation

```zig title="stack_test.zig"
const std = @import("std");

fn Stack(comptime T: type) type {
    return struct {
        items: std.ArrayList(T),
        
        const Self = @This();
        
        pub fn init(allocator: std.mem.Allocator) Self {
            return .{
                .items = std.ArrayList(T).init(allocator),
            };
        }
        
        pub fn deinit(self: *Self) void {
            self.items.deinit();
        }
        
        pub fn push(self: *Self, item: T) !void {
            try self.items.append(item);
        }
        
        pub fn pop(self: *Self) ?T {
            return self.items.popOrNull();
        }
        
        pub fn peek(self: *Self) ?T {
            if (self.items.items.len == 0) return null;
            return self.items.items[self.items.items.len - 1];
        }
        
        pub fn isEmpty(self: Self) bool {
            return self.items.items.len == 0;
        }
    };
}

test "Stack - push and pop" {
    var stack = Stack(i32).init(std.testing.allocator);
    defer stack.deinit();
    
    try stack.push(1);
    try stack.push(2);
    try stack.push(3);
    
    try std.testing.expectEqual(@as(?i32, 3), stack.pop());
    try std.testing.expectEqual(@as(?i32, 2), stack.pop());
    try std.testing.expectEqual(@as(?i32, 1), stack.pop());
    try std.testing.expectEqual(@as(?i32, null), stack.pop());
}

test "Stack - peek doesn't remove" {
    var stack = Stack(i32).init(std.testing.allocator);
    defer stack.deinit();
    
    try stack.push(42);
    
    try std.testing.expectEqual(@as(?i32, 42), stack.peek());
    try std.testing.expectEqual(@as(?i32, 42), stack.peek());
    try std.testing.expectEqual(@as(?i32, 42), stack.pop());
}

test "Stack - isEmpty" {
    var stack = Stack(i32).init(std.testing.allocator);
    defer stack.deinit();
    
    try std.testing.expect(stack.isEmpty());
    
    try stack.push(1);
    try std.testing.expect(!stack.isEmpty());
    
    _ = stack.pop();
    try std.testing.expect(stack.isEmpty());
}

test "Stack - different types" {
    var int_stack = Stack(i32).init(std.testing.allocator);
    defer int_stack.deinit();
    
    var string_stack = Stack([]const u8).init(std.testing.allocator);
    defer string_stack.deinit();
    
    try int_stack.push(42);
    try string_stack.push("hello");
    
    try std.testing.expectEqual(@as(?i32, 42), int_stack.pop());
    try std.testing.expectEqualStrings("hello", string_stack.pop().?);
}
```

## Skipping Tests

Temporarily disable a test by returning `error.SkipZigTest`:

```zig
test "not ready yet" {
    return error.SkipZigTest;
}

test "only on linux" {
    if (@import("builtin").os.tag != .linux) {
        return error.SkipZigTest;
    }
    
    // Linux-specific test...
}
```

## Key Takeaways

- **test keyword** - Define tests with descriptive names
- **std.testing** - Built-in assertions module
- **expectEqual** - Most common assertion
- **testing.allocator** - Detects memory leaks automatically
- **try in tests** - Propagate assertion failures
- **defer** - Clean up resources
- **Table-driven tests** - Test multiple cases with loops
- **SkipZigTest** - Conditionally skip tests

## Next Steps

Now that you can write tests, learn how to run and filter them efficiently in [Controlling How Tests Are Run](02-controlling-how-tests-are-run.md).
