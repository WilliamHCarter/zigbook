# 7.2. Propagating Errors with try

Often, you don't want to handle an error where it occurs. You want to pass it up to the caller. The `try` keyword makes this effortless.

## The try Keyword

`try` is shorthand for propagating errors:

```zig
fn readConfig() !Config {
    const file = try openFile("config.txt");  // If error, return it
    defer file.close();
    
    const contents = try file.readAll();  // If error, return it
    return try parseConfig(contents);     // If error, return it
}
```

## What try Does

The code above is equivalent to:

```zig
fn readConfig() !Config {
    const file = openFile("config.txt") catch |err| return err;
    defer file.close();
    
    const contents = file.readAll() catch |err| return err;
    return parseConfig(contents) catch |err| return err;
}
```

`try` saves you from writing this boilerplate!

## try Requires Error Union Return

You can only use `try` in functions that return error unions:

```zig
// This works
fn mayFail() !void {
    try doSomething();
}

// This doesn't compile!
fn cannotUse() void {
    try doSomething();  // Error: function doesn't return error union
}
```

## Combining try with Other Operations

`try` works in expressions:

```zig
fn calculate() !i32 {
    const a = try getValue();
    const b = try getValue();
    return (try divide(a, b)) * 2;
}
```

## Error Propagation Chain

Errors bubble up the call stack:

```zig
fn level3() !void {
    return error.Problem;
}

fn level2() !void {
    try level3();  // Error propagates up
}

fn level1() !void {
    try level2();  // Error propagates up
}

pub fn main() !void {
    level1() catch |err| {
        std.debug.print("Caught error: {}\n", .{err});
    };
}
```

Output:
```
Caught error: error.Problem
```

## Early Returns

Use `try` with expressions that might need early returns:

```zig
fn processData(data: []const u8) !Result {
    const parsed = try parse(data);
    if (parsed.len == 0) return error.EmptyData;
    
    const validated = try validate(parsed);
    return try transform(validated);
}
```

## try vs catch

Choose based on intent:

**Use `try` when:**
- You want to propagate errors to the caller
- The error should be handled at a higher level
- You're building a chain of operations

**Use `catch` when:**
- You can handle the error locally
- You want to provide a default value
- The error shouldn't propagate

## Mixing try and catch

You can combine both:

```zig
fn smartOperation() !i32 {
    // Propagate most errors
    const data = try fetchData();
    
    // But handle specific ones locally
    const parsed = parseData(data) catch |err| switch (err) {
        error.InvalidFormat => {
            // Handle locally
            std.debug.print("Using default format\n", .{});
            return try parseDataDefault(data);
        },
        else => return err,  // Propagate others
    };
    
    return try processData(parsed);
}
```

## Error Return Trace

When errors propagate, Zig tracks the path:

```zig
const std = @import("std");

fn inner() !void {
    return error.Failed;
}

fn middle() !void {
    try inner();
}

fn outer() !void {
    try middle();
}

pub fn main() !void {
    try outer();
}
```

In debug mode, you'll see where the error originated and how it propagated.

## Complete Example: File Processing

```zig title="file_processing.zig"
const std = @import("std");

const ProcessError = error{
    FileNotFound,
    InvalidFormat,
    TooLarge,
};

fn readFile(allocator: std.mem.Allocator, path: []const u8) ![]u8 {
    const file = std.fs.cwd().openFile(path, .{}) catch {
        return error.FileNotFound;
    };
    defer file.close();
    
    const stat = try file.stat();
    if (stat.size > 1024 * 1024) {  // 1 MB limit
        return error.TooLarge;
    }
    
    return try file.readToEndAlloc(allocator, stat.size);
}

fn validateContent(content: []const u8) !void {
    if (content.len < 10) {
        return error.InvalidFormat;
    }
    
    for (content[0..10]) |c| {
        if (c < 32 or c > 126) {
            return error.InvalidFormat;
        }
    }
}

fn processFile(allocator: std.mem.Allocator, path: []const u8) !void {
    // All errors propagate with try
    const content = try readFile(allocator, path);
    defer allocator.free(content);
    
    try validateContent(content);
    
    std.debug.print("File processed successfully: {s}\n", .{path});
    std.debug.print("Size: {d} bytes\n", .{content.len});
}

pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    const allocator = gpa.allocator();
    
    processFile(allocator, "test.txt") catch |err| {
        std.debug.print("Failed to process file: {}\n", .{err});
        return;
    };
}
```

## Chaining Operations

Build pipelines of fallible operations:

```zig
fn pipeline(input: []const u8) !Result {
    return try transform(
        try validate(
            try parse(
                try normalize(input)
            )
        )
    );
}
```

Or more readably:

```zig
fn pipeline(input: []const u8) !Result {
    const normalized = try normalize(input);
    const parsed = try parse(normalized);
    const validated = try validate(parsed);
    return try transform(validated);
}
```

## Conditional try

Use with if expressions:

```zig
fn maybeProcess(should_process: bool, data: []const u8) !void {
    if (should_process) {
        try processData(data);
    }
}
```

## try in Loops

Handle errors in iterations:

```zig
fn processAll(items: [][]const u8) !void {
    for (items) |item| {
        try processItem(item);  // Stops on first error
    }
}
```

To collect errors instead:

```zig
fn processAllCollectErrors(items: [][]const u8) []Error {
    var errors = std.ArrayList(Error).init(allocator);
    
    for (items) |item| {
        processItem(item) catch |err| {
            try errors.append(err);
            continue;  // Keep processing
        };
    }
    
    return errors.toOwnedSlice();
}
```

## errdefer

Clean up on error returns:

```zig
fn createResource() !*Resource {
    const resource = try allocateResource();
    errdefer freeResource(resource);  // Only runs on error
    
    try initializeResource(resource);
    try setupResource(resource);
    
    return resource;  // Success - errdefer doesn't run
}
```

Compare with `defer`:
- `defer` - Always runs when scope exits
- `errdefer` - Only runs if an error is returned

## Multiple errdefer

They execute in reverse order (LIFO):

```zig
fn complexSetup() !void {
    const a = try allocA();
    errdefer freeA(a);
    
    const b = try allocB();
    errdefer freeB(b);  // Runs first if error
    
    try setupAB(a, b);
}
```

If `setupAB` fails: `freeB` runs, then `freeA`.

## Key Takeaways

- **try** - Propagates errors automatically
- **Requires error union return** - Function must return `!T`
- **Early returns** - Stops execution on error
- **Chainable** - Build operation pipelines
- **Stack traces** - Preserved through propagation
- **errdefer** - Cleanup that only runs on errors
- **Clean code** - Eliminates boilerplate

## Next Steps

You now know how to define errors and propagate them. Learn when to catch, when to propagate, and how to design error-aware APIs in [Error Handling Strategies](03-error-handling-strategies.md).
