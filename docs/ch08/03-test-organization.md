# Test Organization

As your Zig project grows, organizing your tests becomes increasingly important. A well-organized test suite is easier to maintain, understand, and extend. In this section, we'll explore different strategies for organizing tests and discuss when to use each approach.

## Tests in the Same File

The most common pattern in Zig is to write tests directly in the same file as the code they test. This keeps tests close to the implementation and makes it easy to verify behavior while writing code:

```zig title="math.zig"
const std = @import("std");

pub fn add(a: i32, b: i32) i32 {
    return a + b;
}

pub fn multiply(a: i32, b: i32) i32 {
    return a * b;
}

test "add positive numbers" {
    try std.testing.expectEqual(5, add(2, 3));
}

test "add negative numbers" {
    try std.testing.expectEqual(-5, add(-2, -3));
}

test "multiply positive numbers" {
    try std.testing.expectEqual(6, multiply(2, 3));
}

test "multiply by zero" {
    try std.testing.expectEqual(0, multiply(5, 0));
}
```

This approach works well for:
- Unit tests that test individual functions
- Tests that don't require complex setup
- Library code where you want tests to document API usage

## Tests in Separate Files

For more complex testing scenarios, you might want to separate tests into their own files. This is particularly useful for integration tests that test multiple modules together:

```zig title="src/user.zig"
const std = @import("std");

pub const User = struct {
    id: u32,
    name: []const u8,
    email: []const u8,
    
    pub fn init(id: u32, name: []const u8, email: []const u8) User {
        return .{
            .id = id,
            .name = name,
            .email = email,
        };
    }
};
```

```zig title="src/database.zig"
const std = @import("std");
const User = @import("user.zig").User;

pub const Database = struct {
    users: std.ArrayList(User),
    allocator: std.mem.Allocator,
    
    pub fn init(allocator: std.mem.Allocator) Database {
        return .{
            .users = std.ArrayList(User).init(allocator),
            .allocator = allocator,
        };
    }
    
    pub fn deinit(self: *Database) void {
        self.users.deinit();
    }
    
    pub fn addUser(self: *Database, user: User) !void {
        try self.users.append(user);
    }
    
    pub fn findUser(self: *Database, id: u32) ?User {
        for (self.users.items) |user| {
            if (user.id == id) return user;
        }
        return null;
    }
};
```

```zig title="test/integration_test.zig"
const std = @import("std");
const User = @import("../src/user.zig").User;
const Database = @import("../src/database.zig").Database;

test "database integration - add and find user" {
    var db = Database.init(std.testing.allocator);
    defer db.deinit();
    
    const user = User.init(1, "Alice", "alice@example.com");
    try db.addUser(user);
    
    const found = db.findUser(1);
    try std.testing.expect(found != null);
    try std.testing.expectEqual(user.id, found.?.id);
    try std.testing.expectEqualStrings(user.name, found.?.name);
}

test "database integration - find nonexistent user" {
    var db = Database.init(std.testing.allocator);
    defer db.deinit();
    
    const found = db.findUser(999);
    try std.testing.expect(found == null);
}
```

To run tests from a separate test file:

```bash
zig test test/integration_test.zig
```

## Organizing Tests by Category

For larger projects, you might organize tests into multiple directories:

```
my_project/
├── src/
│   ├── main.zig
│   ├── user.zig
│   ├── database.zig
│   └── api.zig
├── test/
│   ├── unit/
│   │   ├── user_test.zig
│   │   └── database_test.zig
│   └── integration/
│       ├── api_test.zig
│       └── end_to_end_test.zig
└── build.zig
```

In your `build.zig`, you can set up different test steps:

```zig title="build.zig"
const std = @import("std");

pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    // Unit tests
    const unit_tests = b.addTest(.{
        .root_source_file = b.path("test/unit/user_test.zig"),
        .target = target,
        .optimize = optimize,
    });
    
    const unit_tests_db = b.addTest(.{
        .root_source_file = b.path("test/unit/database_test.zig"),
        .target = target,
        .optimize = optimize,
    });

    // Integration tests
    const integration_tests = b.addTest(.{
        .root_source_file = b.path("test/integration/api_test.zig"),
        .target = target,
        .optimize = optimize,
    });

    // Unit test step
    const run_unit_tests = b.addRunArtifact(unit_tests);
    const run_unit_tests_db = b.addRunArtifact(unit_tests_db);
    
    const unit_test_step = b.step("test-unit", "Run unit tests");
    unit_test_step.dependOn(&run_unit_tests.step);
    unit_test_step.dependOn(&run_unit_tests_db.step);

    // Integration test step
    const run_integration_tests = b.addRunArtifact(integration_tests);
    
    const integration_test_step = b.step("test-integration", "Run integration tests");
    integration_test_step.dependOn(&run_integration_tests.step);

    // All tests step
    const test_step = b.step("test", "Run all tests");
    test_step.dependOn(unit_test_step);
    test_step.dependOn(integration_test_step);
}
```

This allows you to run different test categories:

```bash
zig build test-unit          # Run only unit tests
zig build test-integration   # Run only integration tests
zig build test               # Run all tests
```

## Test Helpers and Utilities

As your test suite grows, you'll likely want to create helper functions to reduce duplication:

```zig title="test/helpers.zig"
const std = @import("std");
const User = @import("../src/user.zig").User;
const Database = @import("../src/database.zig").Database;

pub fn createTestUser(id: u32) User {
    const name = std.fmt.allocPrint(
        std.testing.allocator,
        "User{d}",
        .{id},
    ) catch unreachable;
    
    const email = std.fmt.allocPrint(
        std.testing.allocator,
        "user{d}@example.com",
        .{id},
    ) catch unreachable;
    
    return User.init(id, name, email);
}

pub fn createTestDatabase() Database {
    return Database.init(std.testing.allocator);
}
```

Then use these helpers in your tests:

```zig title="test/integration_test.zig"
const std = @import("std");
const helpers = @import("helpers.zig");

test "using test helpers" {
    var db = helpers.createTestDatabase();
    defer db.deinit();
    
    const user = helpers.createTestUser(1);
    try db.addUser(user);
    
    try std.testing.expect(db.findUser(1) != null);
}
```

!!! warning "Helper Complexity"
    Keep test helpers simple. If a helper becomes too complex or tries to do too much, it can make tests harder to understand. Sometimes duplicating a bit of test setup code is preferable to having overly complex helpers.

## Testing Private Functions

Sometimes you want to test internal implementation details that aren't exported. In Zig, you can use the `@import` mechanism creatively:

```zig title="calculator.zig"
const std = @import("std");

// Private helper function
fn validateInput(x: i32) bool {
    return x >= 0 and x <= 1000;
}

pub fn calculate(x: i32, y: i32) !i32 {
    if (!validateInput(x) or !validateInput(y)) {
        return error.InvalidInput;
    }
    return x + y;
}

// Tests in the same file can access private functions
test "validateInput with valid values" {
    try std.testing.expect(validateInput(0));
    try std.testing.expect(validateInput(500));
    try std.testing.expect(validateInput(1000));
}

test "validateInput with invalid values" {
    try std.testing.expect(!validateInput(-1));
    try std.testing.expect(!validateInput(1001));
}

test "calculate with valid inputs" {
    try std.testing.expectEqual(7, calculate(3, 4));
}

test "calculate with invalid inputs" {
    try std.testing.expectError(error.InvalidInput, calculate(-1, 5));
    try std.testing.expectError(error.InvalidInput, calculate(5, 1001));
}
```

By keeping tests in the same file, they have access to private functions. This is one reason why in-file tests are so common in Zig.

## Documentation Tests

Tests also serve as documentation. Well-written tests show other developers how to use your code:

```zig title="parser.zig"
const std = @import("std");

pub fn parseInt(str: []const u8) !i32 {
    return std.fmt.parseInt(i32, str, 10);
}

/// This test demonstrates basic usage of parseInt
test "parseInt basic usage" {
    const result = try parseInt("42");
    try std.testing.expectEqual(42, result);
}

/// This test shows error handling
test "parseInt error handling" {
    try std.testing.expectError(error.InvalidCharacter, parseInt("not a number"));
}

/// This test demonstrates parsing negative numbers
test "parseInt with negative numbers" {
    const result = try parseInt("-42");
    try std.testing.expectEqual(-42, result);
}
```

## Performance Tests

While Zig doesn't have built-in benchmarking in the test framework, you can write performance tests using `std.time`:

```zig title="performance.zig"
const std = @import("std");

fn fibonacci(n: u32) u64 {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

test "fibonacci performance" {
    const start = std.time.nanoTimestamp();
    
    _ = fibonacci(20);
    
    const end = std.time.nanoTimestamp();
    const duration = end - start;
    
    // Log the duration (this will appear in test output)
    std.debug.print("fibonacci(20) took {}ns\n", .{duration});
    
    // Optionally assert a maximum duration
    try std.testing.expect(duration < 1_000_000_000); // Less than 1 second
}
```

!!! tip "Benchmarking"
    For serious performance testing, consider using dedicated benchmarking tools or writing custom benchmark harnesses. The built-in test system is better suited for correctness testing than performance measurement.

## Test Naming Conventions

Good test names make your test suite easier to navigate. Common conventions include:

```zig
// Describe what is being tested and the expected outcome
test "add returns sum of two positive numbers" { }
test "divide returns error when divisor is zero" { }

// Group related tests with prefixes
test "Stack: push adds item to top" { }
test "Stack: pop removes and returns top item" { }
test "Stack: pop on empty stack returns null" { }

// Use descriptive names for edge cases
test "parseInt handles maximum i32 value" { }
test "parseInt handles minimum i32 value" { }
test "parseInt rejects strings with only whitespace" { }
```

## Key Takeaways

- Keep unit tests in the same file as the code they test for simplicity
- Use separate test files for integration tests that span multiple modules
- Organize large test suites by category (unit, integration, end-to-end)
- Create test helpers to reduce duplication, but keep them simple
- Use tests as documentation to show how your code should be used
- Follow clear naming conventions to make tests easy to navigate
- Configure different test steps in build.zig for different test categories

## Summary

Testing is a fundamental part of Zig development. The language's built-in test framework makes it easy to write reliable tests without external dependencies. By organizing your tests well, writing clear test names, and leveraging the tools we've covered in this chapter, you can build a robust test suite that gives you confidence in your code.

Remember: tests are not just about catching bugs—they're also documentation, design feedback, and a safety net that enables confident refactoring. Invest time in your tests, and they'll pay dividends throughout your project's lifetime.
