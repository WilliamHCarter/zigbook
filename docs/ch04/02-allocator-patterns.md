# 4.2. Allocator Patterns

Zig provides several built-in allocators, each optimized for different use cases. Choosing the right allocator can dramatically improve your program's performance and simplify memory management.

## Common Allocator Types

### PageAllocator - Direct OS Allocation

The `page_allocator` requests memory directly from the operating system:

```zig
const std = @import("std");

pub fn main() !void {
    const allocator = std.heap.page_allocator;
    
    const data = try allocator.alloc(u8, 1000);
    defer allocator.free(data);
    
    // Use data...
}
```

**When to use:**
- Large, long-lived allocations
- When you need OS-backed memory
- Testing (easier to spot leaks)

**Drawbacks:**
- Slow - every allocation is a system call
- No safety checks in release mode
- Page-aligned (may waste memory for small allocations)

### GeneralPurposeAllocator (GPA) - Safe Default

The GPA is a general-purpose allocator with safety checks:

```zig
const std = @import("std");

pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer {
        const leaked = gpa.deinit();
        if (leaked == .leak) {
            std.debug.print("Memory leaked!\n", .{});
        }
    }
    
    const allocator = gpa.allocator();
    
    const data = try allocator.alloc(i32, 100);
    defer allocator.free(data);
    
    // Use data...
}
```

**Features:**
- Detects memory leaks
- Detects double frees
- Detects use-after-free (in debug mode)
- Thread-safe
- Reasonable performance

**When to use:**
- Development and debugging
- General-purpose allocation
- When you want safety checks

**Configuration:**

```zig
var gpa = std.heap.GeneralPurposeAllocator(.{
    .safety = true,           // Enable safety checks
    .thread_safe = true,      // Thread-safe allocation
    .verbose_log = false,     // Log allocations
}){};
```

### ArenaAllocator - Batch Free

An arena allocator lets you allocate many times but free everything at once:

```zig
const std = @import("std");

pub fn main() !void {
    var arena = std.heap.ArenaAllocator.init(std.heap.page_allocator);
    defer arena.deinit(); // Frees everything!
    
    const allocator = arena.allocator();
    
    // Allocate many things
    const a = try allocator.alloc(u8, 100);
    const b = try allocator.alloc(u8, 200);
    const c = try allocator.alloc(u8, 300);
    
    // No need for individual frees!
    // arena.deinit() cleans up everything
}
```

**When to use:**
- Request/response handling (HTTP servers)
- Parsing operations
- Short-lived computations
- When many allocations have the same lifetime

**Benefits:**
- Extremely fast allocation
- Simple cleanup - one `deinit()` for everything
- No individual `free` calls needed
- Reduces memory fragmentation

**Example: Processing a Request**

```zig
fn handleRequest(backing_allocator: std.mem.Allocator, request: Request) !Response {
    // Create arena for this request
    var arena = std.heap.ArenaAllocator.init(backing_allocator);
    defer arena.deinit(); // All request memory freed here
    
    const allocator = arena.allocator();
    
    // Parse request (allocates memory)
    const parsed = try parseRequest(allocator, request);
    
    // Process data (allocates more memory)
    const result = try processData(allocator, parsed);
    
    // Build response (allocates even more)
    return buildResponse(allocator, result);
    
    // All memory automatically freed by arena.deinit()!
}
```

### FixedBufferAllocator - Pre-Allocated Buffer

Allocate from a fixed buffer - no system calls:

```zig
const std = @import("std");

pub fn main() !void {
    // Pre-allocate a buffer
    var buffer: [1024]u8 = undefined;
    var fba = std.heap.FixedBufferAllocator.init(&buffer);
    const allocator = fba.allocator();
    
    const data = try allocator.alloc(u8, 100);
    // Allocated from buffer, not heap!
    
    // Can free to reuse space
    allocator.free(data);
}
```

**When to use:**
- Embedded systems (no heap)
- Performance-critical paths
- When maximum memory is known
- Avoiding system calls

**Characteristics:**
- No system calls
- Very fast
- Fixed maximum size
- Returns `OutOfMemory` when buffer is full

### StackFallbackAllocator - Stack Then Heap

Try stack first, fall back to heap if needed:

```zig
const std = @import("std");

pub fn main() !void {
    var buffer: [256]u8 = undefined;
    var fba = std.heap.FixedBufferAllocator.init(&buffer);
    var fallback = std.heap.stackFallback(256, std.heap.page_allocator);
    const allocator = fallback.get();
    
    // Small allocations use stack buffer
    const small = try allocator.alloc(u8, 100);
    defer allocator.free(small);
    
    // Large allocations use fallback
    const large = try allocator.alloc(u8, 1000);
    defer allocator.free(large);
}
```

**When to use:**
- Optimization - avoid heap for small sizes
- When size is usually small but might be large

## Choosing the Right Allocator

Here's a decision guide:

```zig
// Development/Debugging - Use GPA
var gpa = std.heap.GeneralPurposeAllocator(.{}){};
const allocator = gpa.allocator();

// Many allocations, same lifetime - Use Arena
var arena = std.heap.ArenaAllocator.init(parent_allocator);
const allocator = arena.allocator();

// Fixed maximum size - Use FixedBuffer
var buffer: [4096]u8 = undefined;
var fba = std.heap.FixedBufferAllocator.init(&buffer);
const allocator = fba.allocator();

// Simple/Testing - Use Page
const allocator = std.heap.page_allocator;
```

## Composing Allocators

Allocators can wrap other allocators:

```zig
const std = @import("std");

pub fn main() !void {
    // Base allocator with leak detection
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    
    // Arena wraps GPA
    var arena = std.heap.ArenaAllocator.init(gpa.allocator());
    defer arena.deinit();
    
    // Use arena for batch operations
    const allocator = arena.allocator();
    // ...
}
```

## Real-World Example: HTTP Server

```zig
const std = @import("std");

const Server = struct {
    gpa: std.heap.GeneralPurposeAllocator(.{}),
    
    pub fn init() Server {
        return .{
            .gpa = std.heap.GeneralPurposeAllocator(.{}){},
        };
    }
    
    pub fn deinit(self: *Server) void {
        _ = self.gpa.deinit();
    }
    
    pub fn handleRequest(self: *Server, request: []const u8) ![]u8 {
        // Create arena for this request
        var arena = std.heap.ArenaAllocator.init(self.gpa.allocator());
        defer arena.deinit();
        
        const allocator = arena.allocator();
        
        // All request processing uses arena
        const parsed = try parseRequest(allocator, request);
        const processed = try processRequest(allocator, parsed);
        
        // Response is allocated on GPA (outlives request)
        return try buildResponse(self.gpa.allocator(), processed);
    }
};

fn parseRequest(allocator: std.mem.Allocator, request: []const u8) !ParsedRequest {
    // Allocations use arena, automatically freed
    _ = request;
    return .{};
}

fn processRequest(allocator: std.mem.Allocator, parsed: ParsedRequest) !ProcessedData {
    _ = allocator;
    _ = parsed;
    return .{};
}

fn buildResponse(allocator: std.mem.Allocator, data: ProcessedData) ![]u8 {
    _ = data;
    return try allocator.dupe(u8, "HTTP/1.1 200 OK\r\n\r\n");
}

const ParsedRequest = struct {};
const ProcessedData = struct {};
```

## Performance Comparison

Different allocators have different performance characteristics:

| Allocator | Alloc Speed | Free Speed | Memory Overhead | Use Case |
|-----------|-------------|------------|----------------|----------|
| Page | Slow | Slow | High | Simple, large allocations |
| GPA | Medium | Medium | Medium | General purpose, safety |
| Arena | Very Fast | Very Fast* | Low-Medium | Batch operations |
| FixedBuffer | Very Fast | Very Fast | None | Bounded memory |

*Arena frees everything at once, not individual items

## Testing Allocators

For tests, use `std.testing.allocator`:

```zig
const std = @import("std");

test "allocation test" {
    const allocator = std.testing.allocator;
    
    const data = try allocator.alloc(u8, 100);
    defer allocator.free(data);
    
    // Test will fail if memory leaks!
}
```

This allocator:
- Detects leaks automatically
- Fails tests on memory errors
- Provides helpful error messages

## Key Takeaways

- **PageAllocator** - Direct OS calls, simple
- **GPA** - Safe, general purpose, good default
- **ArenaAllocator** - Fast batch allocation/deallocation
- **FixedBufferAllocator** - No heap, fixed size
- **Choose based on use case** - Different patterns for different needs
- **Compose allocators** - Wrap for combined benefits
- **Test with testing.allocator** - Automatic leak detection

## Next Steps

Now that you understand allocators, let's explore how to work with references to data in [Slices and Pointers](03-slices-and-pointers.md).
