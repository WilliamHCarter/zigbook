# 4.3. Slices and Pointers

Now that you understand allocators, let's explore how to reference data without owning it. Zig provides slices and pointers for this purpose.

## Understanding Pointers

A **pointer** is a variable that stores a memory address. It "points to" data stored elsewhere.

### Single-Item Pointers

Create a pointer with `*`:

```zig
const std = @import("std");

pub fn main() void {
    var x: i32 = 5;
    const ptr: *i32 = &x;  // Pointer to x
    
    std.debug.print("Value: {d}\n", .{ptr.*}); // Dereference with .*
    
    ptr.* = 10;  // Modify through pointer
    std.debug.print("New value: {d}\n", .{x}); // x is now 10
}
```

**Key syntax:**
- `*T` - Type of a pointer to T
- `&x` - Get address of x (create pointer)
- `ptr.*` - Dereference pointer (access value)

### Const Pointers

Pointers can be const, preventing modification:

```zig
var x: i32 = 5;
const ptr: *const i32 = &x;  // Cannot modify through ptr

// ptr.* = 10;  // Error! Cannot modify const pointer
std.debug.print("{d}\n", .{ptr.*}); // Reading is OK
```

### Pointers to Const vs Const Pointers

```zig
var x: i32 = 5;
var y: i32 = 10;

// Pointer to const - cannot modify value
const ptr1: *const i32 = &x;
// ptr1.* = 6;  // Error!

// Const pointer - cannot change what it points to (less common)
const ptr2: *i32 = &x;
ptr2.* = 6;  // OK - can modify value
// ptr2 = &y;  // Error if ptr2 was const - cannot reassign
```

## Slices - Pointers with Length

A **slice** is a pointer to an array plus a length. It's one of Zig's most important types.

### Slice Basics

```zig
const std = @import("std");

pub fn main() void {
    var array = [_]i32{ 1, 2, 3, 4, 5 };
    
    // A slice is a view into the array
    const slice: []i32 = array[0..];
    
    std.debug.print("Length: {d}\n", .{slice.len});
    std.debug.print("First: {d}\n", .{slice[0]});
    std.debug.print("All: {any}\n", .{slice});
}
```

**Slice characteristics:**
- Contains a pointer and a length
- Doesn't own the data
- Can be a subset of an array
- Runtime-known length

### Creating Slices

```zig
var array = [_]i32{ 1, 2, 3, 4, 5 };

const full: []i32 = array[0..];        // Whole array
const part: []i32 = array[1..4];       // Elements 1, 2, 3
const from_start: []i32 = array[..3];  // Elements 0, 1, 2
const to_end: []i32 = array[2..];      // Elements 2, 3, 4
```

### Const Slices

Most slices should be const slices:

```zig
const array = [_]i32{ 1, 2, 3, 4, 5 };
const slice: []const i32 = &array;

// Can read but not modify
std.debug.print("{d}\n", .{slice[0]});
// slice[0] = 10;  // Error!
```

String literals are const slices:

```zig
const message: []const u8 = "Hello, Zig!";
// Type is []const u8 - slice of const bytes
```

## Arrays vs Slices

Understanding the difference is crucial:

```zig
// Array - fixed size, known at compile time
const array: [5]i32 = [_]i32{ 1, 2, 3, 4, 5 };

// Slice - runtime size, points to array data
const slice: []const i32 = &array;

std.debug.print("Array size: {d}\n", .{array.len});  // Compile-time constant
std.debug.print("Slice size: {d}\n", .{slice.len});  // Runtime value
```

**Arrays:**
- Fixed size: `[5]i32`
- Value type (copied when assigned)
- Live on stack if size is known
- Can't pass different sizes to same function

**Slices:**
- Dynamic size: `[]i32`
- Reference type (pointer + length)
- Point to array data
- Same function can handle any size

## Slices and Functions

Functions typically take slices, not arrays:

```zig
const std = @import("std");

// Good - works with any size
fn sum(numbers: []const i32) i32 {
    var total: i32 = 0;
    for (numbers) |n| {
        total += n;
    }
    return total;
}

pub fn main() void {
    const arr1 = [_]i32{ 1, 2, 3 };
    const arr2 = [_]i32{ 1, 2, 3, 4, 5 };
    
    std.debug.print("Sum 1: {d}\n", .{sum(&arr1)});
    std.debug.print("Sum 2: {d}\n", .{sum(&arr2)});
}
```

## Allocated Slices

When you allocate memory, you get a slice:

```zig
const std = @import("std");

pub fn main() !void {
    const allocator = std.heap.page_allocator;
    
    // alloc returns a slice
    const numbers: []i32 = try allocator.alloc(i32, 10);
    defer allocator.free(numbers);
    
    // Initialize
    for (numbers, 0..) |*num, i| {
        num.* = @intCast(i * 2);
    }
    
    std.debug.print("Numbers: {any}\n", .{numbers});
}
```

## Sentinel-Terminated Slices

Slices can have a sentinel value at the end:

```zig
// Null-terminated string (like C)
const c_string: [:0]const u8 = "Hello";

// Zig knows there's a \0 after the last character
// Useful for C interop
```

## Pointer Arithmetic - The Zig Way

Unlike C, Zig doesn't have pointer arithmetic. Instead, use slices:

```zig
// C style (not valid Zig):
// ptr++;  // Move to next element

// Zig style:
const array = [_]i32{ 1, 2, 3, 4, 5 };
var slice: []const i32 = &array;

// "Advance" by creating a new slice
slice = slice[1..];  // Skip first element
```

## Multi-Pointer

For advanced cases, Zig has multi-item pointers:

```zig
const array = [_]i32{ 1, 2, 3, 4, 5 };
const ptr: [*]const i32 = &array;

// Can index but no length info
const first = ptr[0];
const second = ptr[1];
```

**When to use:**
- C interop
- Performance-critical code
- When you track length separately

**Prefer slices** - they're safer because they carry length information.

## Practical Example: String Processing

```zig title="string_slices.zig"
const std = @import("std");

fn firstWord(s: []const u8) []const u8 {
    for (s, 0..) |char, i| {
        if (char == ' ') {
            return s[0..i];
        }
    }
    return s;
}

pub fn main() void {
    const sentence = "Hello Zig World";
    const word = firstWord(sentence);
    
    std.debug.print("First word: {s}\n", .{word});
    
    // Slice is still valid because sentence is in scope
}
```

This is safe because:
1. The slice references data that still exists
2. The slice can't outlive the original data (compile-time check)
3. Length is tracked automatically

## Dangling Pointers - Zig's Protection

Zig helps prevent dangling pointers at compile time:

```zig
fn dangle() []const u8 {
    const local = "hello";
    return local[0..];  // Error! Returning pointer to local data
}

// Zig: "function returns address of local variable"
```

To fix it, allocate or use a global:

```zig
fn notDangling(allocator: std.mem.Allocator) ![]const u8 {
    return try allocator.dupe(u8, "hello");
}
```

## Optional Pointers and Slices

Pointers and slices can be optional:

```zig
const maybe_ptr: ?*i32 = null;
const maybe_slice: ?[]const u8 = null;

if (maybe_slice) |slice| {
    std.debug.print("Got slice: {s}\n", .{slice});
} else {
    std.debug.print("No slice\n", .{});
}
```

## Memory Layout

Understanding how slices work internally:

```zig
const Slice = struct {
    ptr: [*]const u8,  // Pointer to first element
    len: usize,        // Number of elements
};

// A []const u8 is basically the above struct
```

When you pass a slice, you're passing:
- A pointer (8 bytes on 64-bit)
- A length (8 bytes on 64-bit)
- Total: 16 bytes, regardless of slice size

## Common Patterns

### Iterating with Index

```zig
const items = [_]i32{ 10, 20, 30, 40 };

for (items, 0..) |item, i| {
    std.debug.print("[{d}] = {d}\n", .{ i, item });
}
```

### Splitting Strings

```zig
fn split(s: []const u8, delimiter: u8) ?struct { []const u8, []const u8 } {
    for (s, 0..) |char, i| {
        if (char == delimiter) {
            return .{ s[0..i], s[i + 1 ..] };
        }
    }
    return null;
}
```

### Checking Prefixes

```zig
fn startsWith(s: []const u8, prefix: []const u8) bool {
    if (s.len < prefix.len) return false;
    return std.mem.eql(u8, s[0..prefix.len], prefix);
}
```

## Key Takeaways

- **Pointers** (`*T`) - Point to a single value
- **Slices** (`[]T`) - Point to a sequence with length
- **Const slices** (`[]const T`) - Most common, read-only
- **Arrays** vs **Slices** - Fixed vs dynamic size
- **Functions take slices** - More flexible than arrays
- **Slices are safe** - Carry length, prevent overruns
- **No pointer arithmetic** - Use slice operations instead
- **Zig prevents dangles** - Compile-time lifetime checks

## Summary

You've now learned the three pillars of Zig's memory management:

1. **Allocators** - Explicit memory allocation
2. **Allocator Patterns** - Different strategies for different needs
3. **Slices and Pointers** - Safe references to data

Together, these give you complete control over memory while maintaining safety and clarity. You decide when to allocate, which strategy to use, and how to reference data - all explicitly, with the compiler helping catch mistakes.

## Next Steps

With memory management mastered, you're ready to build more complex programs! The next chapters will explore structs, enums, and other ways to organize your data.
