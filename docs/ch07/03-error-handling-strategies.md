# 7.3. Error Handling Strategies

Now that you know the mechanics of error handling, let's explore strategies for when to catch, when to propagate, and how to design error-aware programs.

## The Golden Rule

**Handle errors at the right level** - where you have enough context to make a meaningful decision.

## Strategy 1: Let It Crash

For unrecoverable errors, don't catch them - let them propagate to main:

```zig
pub fn main() !void {
    try criticalSetup();
    try run();
}
```

If initialization fails, the program exits with an error message. This is often the right choice for:
- Missing configuration files
- Required resources unavailable
- Invalid arguments

## Strategy 2: Retry with Backoff

For transient failures, retry the operation:

```zig
fn fetchWithRetry(url: []const u8, max_attempts: u32) ![]u8 {
    var attempts: u32 = 0;
    while (attempts < max_attempts) : (attempts += 1) {
        const result = fetch(url) catch |err| {
            if (attempts + 1 >= max_attempts) {
                return err;  // Final attempt failed
            }
            
            std.debug.print("Attempt {d} failed, retrying...\n", .{attempts + 1});
            std.time.sleep(std.time.ns_per_s * (attempts + 1));
            continue;
        };
        
        return result;  // Success
    }
    
    return error.MaxRetriesExceeded;
}
```

## Strategy 3: Fallback Values

Provide sensible defaults when operations fail:

```zig
fn loadConfig(allocator: std.mem.Allocator) Config {
    const config = loadConfigFromFile(allocator) catch |err| {
        std.debug.print("Using default config: {}\n", .{err});
        return Config.default();
    };
    
    return config;
}
```

Good for:
- Optional features
- User preferences
- Cache hits/misses

## Strategy 4: Transform Errors

Convert low-level errors to domain-specific errors:

```zig
const AppError = error{
    ConfigurationError,
    DatabaseError,
    NetworkError,
};

fn loadUser(id: u32) AppError!User {
    const conn = database.connect() catch {
        return error.DatabaseError;
    };
    defer conn.close();
    
    const user = conn.query("SELECT * FROM users WHERE id = ?", .{id}) catch {
        return error.DatabaseError;
    };
    
    return user;
}
```

This hides implementation details from callers.

## Strategy 5: Error Context

Add context to errors as they propagate:

```zig
const Error = error{
    FileError,
    ParseError,
    ProcessError,
};

fn processFile(path: []const u8) Error!void {
    const content = std.fs.cwd().readFileAlloc(
        allocator,
        path,
        1024 * 1024,
    ) catch {
        std.debug.print("Failed to read file: {s}\n", .{path});
        return error.FileError;
    };
    defer allocator.free(content);
    
    const data = parseContent(content) catch {
        std.debug.print("Failed to parse file: {s}\n", .{path});
        return error.ParseError;
    };
    
    process(data) catch {
        std.debug.print("Failed to process data from: {s}\n", .{path});
        return error.ProcessError;
    };
}
```

## Strategy 6: Collect All Errors

For validation, collect all errors instead of failing fast:

```zig
const ValidationError = struct {
    field: []const u8,
    message: []const u8,
};

fn validateUser(user: User) ![]ValidationError {
    var errors = std.ArrayList(ValidationError).init(allocator);
    
    if (user.name.len == 0) {
        try errors.append(.{
            .field = "name",
            .message = "Name cannot be empty",
        });
    }
    
    if (user.age < 0 or user.age > 150) {
        try errors.append(.{
            .field = "age",
            .message = "Age must be between 0 and 150",
        });
    }
    
    if (user.email.len == 0) {
        try errors.append(.{
            .field = "email",
            .message = "Email is required",
        });
    }
    
    return errors.toOwnedSlice();
}
```

## Strategy 7: Early Validation

Check preconditions early:

```zig
fn divide(a: i32, b: i32) !i32 {
    if (b == 0) return error.DivisionByZero;
    if (a == std.math.minInt(i32) and b == -1) return error.Overflow;
    
    return @divTrunc(a, b);
}
```

## Strategy 8: Panic for Programming Errors

Use `unreachable` or `@panic` for "should never happen" cases:

```zig
fn getElement(array: []const i32, index: usize) i32 {
    if (index >= array.len) {
        @panic("Index out of bounds");
    }
    return array[index];
}
```

Reserve this for:
- Broken invariants
- Logic errors
- Contract violations

Not for expected errors like user input or network failures.

## Complete Example: Robust File Processor

```zig title="robust_processor.zig"
const std = @import("std");

const ProcessError = error{
    FileNotFound,
    FileTooLarge,
    InvalidFormat,
    ProcessingFailed,
};

const Config = struct {
    max_file_size: usize = 10 * 1024 * 1024,  // 10 MB
    retry_attempts: u32 = 3,
};

fn processFileWithRetry(
    allocator: std.mem.Allocator,
    path: []const u8,
    config: Config,
) ProcessError!void {
    var attempts: u32 = 0;
    
    while (attempts < config.retry_attempts) : (attempts += 1) {
        processFileSingle(allocator, path, config) catch |err| {
            std.debug.print(
                "Attempt {d}/{d} failed: {}\n",
                .{ attempts + 1, config.retry_attempts, err },
            );
            
            // Don't retry certain errors
            switch (err) {
                error.FileNotFound, error.InvalidFormat => return err,
                else => {},
            }
            
            if (attempts + 1 >= config.retry_attempts) {
                return err;
            }
            
            // Exponential backoff
            const backoff_ms = (@as(u64, 1) << @intCast(attempts)) * 100;
            std.time.sleep(backoff_ms * std.time.ns_per_ms);
            continue;
        };
        
        return;  // Success
    }
}

fn processFileSingle(
    allocator: std.mem.Allocator,
    path: []const u8,
    config: Config,
) ProcessError!void {
    // Open file with context
    const file = std.fs.cwd().openFile(path, .{}) catch {
        std.debug.print("File not found: {s}\n", .{path});
        return error.FileNotFound;
    };
    defer file.close();
    
    // Check size
    const stat = file.stat() catch |err| {
        std.debug.print("Cannot stat file {s}: {}\n", .{ path, err });
        return error.ProcessingFailed;
    };
    
    if (stat.size > config.max_file_size) {
        std.debug.print(
            "File too large: {d} bytes (max: {d})\n",
            .{ stat.size, config.max_file_size },
        );
        return error.FileTooLarge;
    }
    
    // Read content
    const content = file.readToEndAlloc(allocator, stat.size) catch |err| {
        std.debug.print("Failed to read {s}: {}\n", .{ path, err });
        return error.ProcessingFailed;
    };
    defer allocator.free(content);
    
    // Validate format
    if (!isValidFormat(content)) {
        std.debug.print("Invalid format in {s}\n", .{path});
        return error.InvalidFormat;
    }
    
    // Process
    processContent(content) catch {
        std.debug.print("Processing failed for {s}\n", .{path});
        return error.ProcessingFailed;
    };
    
    std.debug.print("Successfully processed: {s}\n", .{path});
}

fn isValidFormat(content: []const u8) bool {
    return content.len > 0 and content[0] == '#';
}

fn processContent(content: []const u8) !void {
    // Simulate processing
    if (content.len < 10) return error.TooShort;
}

pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    const allocator = gpa.allocator();
    
    const config = Config{};
    
    // Process multiple files, continuing on errors
    const files = [_][]const u8{ "file1.txt", "file2.txt", "file3.txt" };
    
    var succeeded: u32 = 0;
    var failed: u32 = 0;
    
    for (files) |file| {
        processFileWithRetry(allocator, file, config) catch {
            failed += 1;
            continue;
        };
        succeeded += 1;
    }
    
    std.debug.print(
        "\nResults: {d} succeeded, {d} failed\n",
        .{ succeeded, failed },
    );
}
```

## Designing Error-Aware APIs

### Be Specific

```zig
// Bad: Generic error
fn loadConfig() !Config

// Good: Specific errors
fn loadConfig() (error{FileNotFound,InvalidSyntax,PermissionDenied})!Config
```

### Document Errors

```zig
/// Loads user configuration from the standard location.
/// 
/// Errors:
/// - FileNotFound: Config file doesn't exist
/// - PermissionDenied: Cannot read config file  
/// - InvalidFormat: Config file is malformed
fn loadConfig() !Config
```

### Consider Error Recovery

Design APIs so callers can recover:

```zig
// Bad: Can't tell why it failed
fn connect() !Connection

// Good: Caller can handle specific errors
fn connect() (error{NetworkUnreachable,AuthFailed,Timeout})!Connection
```

## Testing Error Paths

Always test error cases:

```zig
test "divide by zero" {
    const result = divide(10, 0);
    try std.testing.expectError(error.DivisionByZero, result);
}

test "divide success" {
    const result = try divide(10, 2);
    try std.testing.expectEqual(@as(i32, 5), result);
}
```

## Key Takeaways

- **Right level** - Handle errors where you have context
- **Let it crash** - For unrecoverable errors
- **Retry** - For transient failures
- **Fallbacks** - For optional operations
- **Transform** - To hide implementation details
- **Context** - Add information as errors bubble up
- **Collect** - For validation scenarios
- **Document** - What errors are possible and why
- **Test** - Both success and error paths

## Summary

Zig's error handling gives you:
- **Explicit errors** - Visible in type signatures
- **Zero overhead** - No stack unwinding
- **Type safety** - Compiler ensures handling
- **Flexibility** - Handle or propagate as needed

The key is choosing the right strategy for each situation. With practice, error handling becomes natural and your code becomes more robust.

## Next Steps

You've now mastered error handling in Zig! Combined with the memory management, structs, enums, and control flow you learned earlier, you have all the tools to write safe, reliable programs.

Keep building, keep handling errors thoughtfully, and your code will thank you.
