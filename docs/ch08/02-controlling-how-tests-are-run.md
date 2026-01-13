# Controlling How Tests Are Run

Now that you know how to write tests in Zig, let's explore the various ways you can control how your test suite runs. The Zig compiler provides several command-line options that allow you to run specific tests, control test output, and optimize test execution.

## Running All Tests

The simplest way to run tests is with the `zig test` command:

```bash
zig test src/main.zig
```

This command compiles the specified file and all its dependencies, runs every test it finds, and reports the results. If all tests pass, you'll see output similar to:

```
All 12 tests passed.
```

## Running Specific Tests with Filters

As your test suite grows, you may want to run only specific tests. Zig provides a `--test-filter` option that lets you run tests whose names contain a specific substring:

```bash
zig test src/main.zig --test-filter "addition"
```

This command will run only tests with "addition" in their name. For example:

```zig title="math.zig"
const std = @import("std");

test "addition with positive numbers" {
    try std.testing.expectEqual(5, 2 + 3);
}

test "addition with negative numbers" {
    try std.testing.expectEqual(-5, -2 + -3);
}

test "subtraction" {
    try std.testing.expectEqual(1, 3 - 2);
}
```

Running `zig test math.zig --test-filter "addition"` would execute both addition tests but skip the subtraction test.

You can also use multiple filters by specifying the option multiple times:

```bash
zig test src/main.zig --test-filter "addition" --test-filter "multiplication"
```

## Controlling Test Output

By default, Zig's test runner provides minimal output. For more detailed information about which tests are running, you can increase verbosity using build options or by examining test output more carefully.

When a test fails, Zig provides helpful information including:

- The test name
- The file and line number where the failure occurred
- The specific assertion that failed
- Expected vs. actual values (when applicable)

Here's an example of a test failure:

```zig title="example_failure.zig"
const std = @import("std");

test "this will fail" {
    try std.testing.expectEqual(10, 5 + 4);
}
```

Running this test produces:

```
Test [0/1] test.this will fail... FAIL (TestExpectedEqual)
/path/to/example_failure.zig:4:5: 0x1034a8b77 in test.this will fail (test)
    try std.testing.expectEqual(10, 5 + 4);
    ^
expected: 10
found: 9
```

## Using the Build System

For larger projects, you typically integrate tests into your `build.zig` file. This allows you to configure test execution more precisely:

```zig title="build.zig"
const std = @import("std");

pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    // Create the main executable
    const exe = b.addExecutable(.{
        .name = "myapp",
        .root_source_file = b.path("src/main.zig"),
        .target = target,
        .optimize = optimize,
    });
    b.installArtifact(exe);

    // Create a test step
    const unit_tests = b.addTest(.{
        .root_source_file = b.path("src/main.zig"),
        .target = target,
        .optimize = optimize,
    });

    const run_unit_tests = b.addRunArtifact(unit_tests);

    // Create a test step that can be invoked with `zig build test`
    const test_step = b.step("test", "Run unit tests");
    test_step.dependOn(&run_unit_tests.step);
}
```

With this build configuration, you can run tests using:

```bash
zig build test
```

This approach integrates nicely into continuous integration systems and development workflows.

## Testing with Different Optimization Modes

Tests can behave differently depending on the optimization level. By default, tests run in Debug mode, but you can test with different optimization settings:

```bash
zig test src/main.zig -O ReleaseFast
```

The optimization modes are:

- `Debug` (default): No optimizations, includes safety checks
- `ReleaseSafe`: Optimized but keeps safety checks
- `ReleaseFast`: Optimized without safety checks
- `ReleaseSmall`: Optimized for small binary size

It's good practice to run your tests in both Debug and ReleaseSafe modes to catch issues that might only appear with optimizations enabled.

## Testing for Specific Targets

You can also run tests for specific target architectures:

```bash
zig test src/main.zig -target x86_64-linux
```

This is useful when writing cross-platform code to ensure your tests pass on different architectures, though keep in mind you typically can't run tests for foreign architectures unless you're using an emulator or cross-compilation setup.

## Test Execution Model

Zig tests run sequentially by default in the order they appear in your code. This means:

1. Tests have a predictable execution order
2. You don't need to worry about race conditions between tests
3. Test failures stop execution at that test

However, this also means that test execution is not parallelized. For large test suites, this can impact test runtime. The Zig team is exploring ways to enable parallel test execution while maintaining safety.

## Performance Considerations

When running tests, consider these performance factors:

**Compilation Time**: Each `zig test` invocation triggers a full compilation. Use `zig build test` in projects where you've set up build caching—this can significantly speed up repeated test runs.

**Test Isolation**: Because tests run in the same process, they share memory. Be careful about:
- Global state that persists between tests
- File system modifications that might affect other tests
- Resource exhaustion (memory, file handles)

**Example of Proper Resource Management**:

```zig title="resource_test.zig"
const std = @import("std");

test "first test with allocator" {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    const allocator = gpa.allocator();
    
    const data = try allocator.alloc(u8, 100);
    defer allocator.free(data);
    
    try std.testing.expect(data.len == 100);
}

test "second test with allocator" {
    // Completely independent allocator
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    const allocator = gpa.allocator();
    
    const data = try allocator.alloc(u8, 50);
    defer allocator.free(data);
    
    try std.testing.expect(data.len == 50);
}
```

Each test creates and destroys its own allocator, ensuring complete isolation.

## Continuous Integration

In CI environments, you typically want to:

1. Run all tests
2. Ensure no memory leaks
3. Test multiple optimization modes
4. Fail the build on any test failure

A typical CI script might look like:

```bash
#!/bin/bash
set -e  # Exit on any error

# Run tests in debug mode
zig build test

# Run tests with safety checks and optimizations
zig build test -Doptimize=ReleaseSafe

echo "All tests passed!"
```

## Key Takeaways

- Use `zig test` for quick test runs and `zig build test` for project-integrated testing
- Filter tests with `--test-filter` to run specific subsets
- Test with different optimization modes to catch mode-specific bugs
- Tests run sequentially in the order they appear
- Properly manage resources in each test to maintain isolation
- Integrate tests into your build system for better caching and CI integration

## Next Steps

In the next section, we'll explore test organization strategies—how to structure your tests as your codebase grows, separate unit tests from integration tests, and maintain a clean, maintainable test suite.
