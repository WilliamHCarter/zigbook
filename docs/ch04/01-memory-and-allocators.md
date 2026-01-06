# 4.1. Memory and Allocators

To understand how Zig manages memory, we need to understand where memory lives and how allocators provide access to it.

## The Stack and the Heap

Like other systems programming languages, Zig programs use two regions of memory: the stack and the heap.

### The Stack

The **stack** stores values in last-in, first-out (LIFO) order. Think of a stack of plates: you add plates on top and remove from the top.

```zig
fn example() void {
    const x: i32 = 5;        // Pushed onto stack
    const y: i32 = 10;       // Pushed onto stack
    const z: i32 = x + y;    // Pushed onto stack
}  // All values popped off stack
```

**Stack characteristics:**
- Very fast - allocation is just moving a pointer
- Fixed size known at compile time
- Automatically cleaned up when scope ends
- Limited in size (typically a few megabytes)

### The Heap

The **heap** is less organized but more flexible. When you need memory on the heap, you request it from an allocator.

```zig
const allocator = std.heap.page_allocator;
const ptr = try allocator.create(i32);
ptr.* = 42;
// Later: must manually free
allocator.destroy(ptr);
```

**Heap characteristics:**
- Slower - requires finding available space
- Can grow dynamically at runtime
- You control when memory is freed
- Much larger than the stack

### When to Use Each

**Use the stack when:**
- Size is known at compile time
- Data is small and short-lived
- You want maximum performance

**Use the heap when:**
- Size is determined at runtime
- Data must outlive the current function
- You need large allocations

## The Allocator Interface

Zig doesn't have a global `malloc` or `new` keyword. Instead, you explicitly pass an allocator to anything that needs to allocate memory.

### The `std.mem.Allocator` Interface

All allocators in Zig implement this interface:

```zig
pub const Allocator = struct {
    ptr: *anyopaque,
    vtable: *const VTable,
    
    // Main allocation methods
    pub fn alloc(self: Allocator, comptime T: type, n: usize) ![]T
    pub fn free(self: Allocator, memory: anytype) void
    pub fn create(self: Allocator, comptime T: type) !*T
    pub fn destroy(self: Allocator, ptr: anytype) void
    // ... more methods
};
```

### Basic Allocation Example

```zig
const std = @import("std");

pub fn main() !void {
    // Get an allocator (we'll explore different types later)
    const allocator = std.heap.page_allocator;
    
    // Allocate a single integer
    const num = try allocator.create(i32);
    defer allocator.destroy(num); // Clean up when done
    
    num.* = 42;
    std.debug.print("Number: {d}\n", .{num.*});
}
```

### Allocating Slices

More commonly, you'll allocate arrays or slices:

```zig
const std = @import("std");

pub fn main() !void {
    const allocator = std.heap.page_allocator;
    
    // Allocate an array of 10 integers
    const numbers = try allocator.alloc(i32, 10);
    defer allocator.free(numbers);
    
    // Use the array
    for (numbers, 0..) |*num, i| {
        num.* = @intCast(i * 2);
    }
    
    std.debug.print("Numbers: {any}\n", .{numbers});
}
```

## Why Pass Allocators Explicitly?

You might wonder: why pass allocators around instead of using a global allocator?

**Benefits of explicit allocators:**

1. **Flexibility** - Different parts of your program can use different allocation strategies
2. **Testing** - Easy to inject a testing allocator that tracks allocations
3. **Control** - You decide the allocation strategy, not the language
4. **Clarity** - It's obvious when allocations happen

```zig
fn processData(allocator: std.mem.Allocator, size: usize) ![]u8 {
    // Caller controls the allocation strategy
    return try allocator.alloc(u8, size);
}
```

## Memory Leaks and Defer

In Zig, **you are responsible for freeing memory**. Forget to free, and you have a leak.

### The Problem

```zig
// BAD: Memory leak!
pub fn leaky() !void {
    const allocator = std.heap.page_allocator;
    const data = try allocator.alloc(u8, 100);
    // Oops! Never freed
}
```

### The Solution: defer

Use `defer` to ensure cleanup happens:

```zig
// GOOD: Memory is freed
pub fn clean() !void {
    const allocator = std.heap.page_allocator;
    const data = try allocator.alloc(u8, 100);
    defer allocator.free(data); // Runs when scope exits
    
    // Use data...
}
```

### Multiple Allocations

With multiple allocations, `defer` in reverse order:

```zig
pub fn example() !void {
    const allocator = std.heap.page_allocator;
    
    const a = try allocator.create(i32);
    defer allocator.destroy(a);
    
    const b = try allocator.create(i32);
    defer allocator.destroy(b);
    
    // b freed first, then a (LIFO)
}
```

## Errors and Cleanup

What if an error occurs after allocating?

```zig
pub fn mayFail(allocator: std.mem.Allocator) !void {
    const data = try allocator.alloc(u8, 100);
    defer allocator.free(data); // Still runs on error!
    
    try somethingThatMightFail();
    // If error occurs, defer still executes
}
```

The `defer` statement ensures cleanup even when errors are returned.

### errdefer for Conditional Cleanup

Sometimes you only want cleanup on error:

```zig
pub fn createResource(allocator: std.mem.Allocator) !*Resource {
    const resource = try allocator.create(Resource);
    errdefer allocator.destroy(resource); // Only on error
    
    try resource.init(); // If this fails, resource is freed
    return resource;     // Success: caller owns resource
}
```

## Example: Building a Dynamic Array

Let's put it all together:

```zig title="dynamic_array.zig"
const std = @import("std");

pub fn main() !void {
    const allocator = std.heap.page_allocator;
    
    // Allocate initial array
    var capacity: usize = 4;
    var len: usize = 0;
    var items = try allocator.alloc(i32, capacity);
    defer allocator.free(items);
    
    // Add items
    items[len] = 10;
    len += 1;
    items[len] = 20;
    len += 1;
    items[len] = 30;
    len += 1;
    
    // Print items
    std.debug.print("Items: ", .{});
    for (items[0..len]) |item| {
        std.debug.print("{d} ", .{item});
    }
    std.debug.print("\n", .{});
}
```

## Common Pitfalls

### Double Free

```zig
// BAD: Double free!
const data = try allocator.alloc(u8, 10);
allocator.free(data);
allocator.free(data); // Crash!
```

### Use After Free

```zig
// BAD: Use after free!
var data = try allocator.alloc(u8, 10);
allocator.free(data);
data[0] = 5; // Undefined behavior!
```

### Forgetting to Free

```zig
// BAD: Memory leak!
fn leak() !void {
    const data = try allocator.alloc(u8, 10);
    // Forgot defer allocator.free(data);
}
```

## Key Takeaways

- **Stack** - Fast, automatic, limited size, compile-time known
- **Heap** - Flexible, manual management, runtime size
- **Allocators** - Explicit interfaces for memory allocation
- **defer** - Ensures cleanup happens
- **errdefer** - Cleanup only on errors
- **You own memory** - Must free what you allocate

## Next Steps

Now that you understand allocators, let's explore different allocation strategies in [Allocator Patterns](02-allocator-patterns.md).
