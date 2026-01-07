# 6.3. Optional Types and Null Safety

One of the most common bugs in programming is the "null pointer exception." Zig eliminates this entire class of errors through **optional types**.

## The Problem with Null

Tony Hoare, the inventor of null references, called it his "billion-dollar mistake." The problem: in many languages, any pointer can be null, but you don't know which ones actually are.

```c
// C code - which of these can be NULL?
char* getName();
int* getAge();
// You have to read documentation or source code to know!
```

## Zig's Solution: Optional Types

In Zig, values are **non-null by default**. If something might be null, you mark it with `?`:

```zig
const name: []const u8 = "Alice";  // Cannot be null
const maybe_name: ?[]const u8 = null;  // Can be null
```

The `?` makes the type **optional**.

## Defining Optional Values

```zig
// These work
const some_number: ?i32 = 5;
const no_number: ?i32 = null;

// This doesn't compile!
const regular_number: i32 = null;  // Error!
```

Without the `?`, you **cannot** assign null. This is enforced at compile time.

## Using Optional Values

You can't use an optional value directly:

```zig
const x: i32 = 5;
const y: ?i32 = 5;

const sum = x + y;  // Error: can't add i32 and ?i32
```

This forces you to explicitly handle the null case!

## Unwrapping with if

Use `if` to check and unwrap:

```zig
const maybe_number: ?i32 = 5;

if (maybe_number) |number| {
    // number is i32 here, not ?i32
    std.debug.print("Got number: {d}\n", .{number});
} else {
    std.debug.print("No number\n", .{});
}
```

The `|number|` syntax **unwraps** the optional.

## Unwrapping with .?

If you're **certain** a value isn't null, use `.?`:

```zig
const maybe_number: ?i32 = 5;
const number = maybe_number.?;  // Unwraps to i32
```

**Warning:** If the value is null, this causes a runtime panic in debug mode and undefined behavior in release mode!

```zig
const nothing: ?i32 = null;
const oops = nothing.?;  // Panic!
```

## The orelse Operator

Provide a default value with `orelse`:

```zig
const maybe_number: ?i32 = null;
const number = maybe_number orelse 0;  // Returns 0 if null
```

This is like the `??` operator in other languages.

## Optional with Switch

You can switch on optionals:

```zig
const maybe_number: ?i32 = 5;

const result = switch (maybe_number) {
    null => 0,
    else => |num| num * 2,
};
```

## Optional Pointers

Optionals are commonly used with pointers:

```zig
fn findUser(id: u32) ?*User {
    // Return pointer to user, or null if not found
    if (id == 0) return null;
    // ... search logic
}

if (findUser(42)) |user| {
    std.debug.print("Found: {s}\n", .{user.name});
} else {
    std.debug.print("User not found\n", .{});
}
```

## Chaining Optionals

You can chain optional operations:

```zig
const value = parseNumber(input) orelse return error.InvalidInput;
```

Or with `if`:

```zig
if (parseNumber(input)) |num| {
    if (validate(num)) |valid_num| {
        return processNumber(valid_num);
    }
}
return error.InvalidNumber;
```

## Example: Safe Array Access

```zig
fn getElement(array: []const i32, index: usize) ?i32 {
    if (index >= array.len) return null;
    return array[index];
}

const numbers = [_]i32{ 10, 20, 30 };

if (getElement(&numbers, 1)) |value| {
    std.debug.print("Found: {d}\n", .{value});  // Prints 20
}

if (getElement(&numbers, 10)) |value| {
    std.debug.print("Found: {d}\n", .{value});
} else {
    std.debug.print("Index out of bounds\n", .{});  // Prints this
}
```

## Optional in Structs

Structs can have optional fields:

```zig
const User = struct {
    name: []const u8,
    email: ?[]const u8,  // Email is optional
    age: ?u32,           // Age is optional
};

const user = User{
    .name = "Alice",
    .email = "alice@example.com",
    .age = null,  // Age not provided
};

if (user.age) |age| {
    std.debug.print("Age: {d}\n", .{age});
} else {
    std.debug.print("Age not provided\n", .{});
}
```

## Error Unions vs Optionals

Optionals say "might be null."
Error unions say "might be an error."

```zig
// Optional - value or null
const maybe: ?i32 = null;

// Error union - value or error
const result: !i32 = error.Failed;

// Both! - value, error, or null
const complex: !?i32 = null;
```

## Complete Example: Linked List

```zig title="optional_example.zig"
const std = @import("std");

const Node = struct {
    value: i32,
    next: ?*Node,
};

fn findValue(head: ?*Node, target: i32) ?*Node {
    var current = head;
    while (current) |node| {
        if (node.value == target) {
            return node;
        }
        current = node.next;
    }
    return null;
}

fn printList(head: ?*Node) void {
    var current = head;
    while (current) |node| {
        std.debug.print("{d} -> ", .{node.value});
        current = node.next;
    }
    std.debug.print("null\n", .{});
}

pub fn main() void {
    var node3 = Node{ .value = 30, .next = null };
    var node2 = Node{ .value = 20, .next = &node3 };
    var node1 = Node{ .value = 10, .next = &node2 };
    
    printList(&node1);
    
    if (findValue(&node1, 20)) |found| {
        std.debug.print("Found node with value: {d}\n", .{found.value});
    } else {
        std.debug.print("Not found\n", .{});
    }
    
    if (findValue(&node1, 99)) |found| {
        std.debug.print("Found node with value: {d}\n", .{found.value});
    } else {
        std.debug.print("Value 99 not found\n", .{});
    }
}
```

## When to Use Optional Types

**Use ?T when:**
- A value might legitimately not exist
- You want compile-time null safety
- The absence of a value is normal, not an error

**Use !T when:**
- A value might fail to be produced
- You need to communicate why it failed
- The absence of a value is exceptional

**Use !?T when:**
- An operation can fail (error) OR succeed with no value (null)

## Benefits Over Null Pointers

1. **Explicit** - You know which values can be null
2. **Safe** - Can't use null without checking
3. **Clear** - Compiler guides you to handle null cases
4. **No surprises** - Null pointer exceptions impossible

## Key Takeaways

- **?T** - Optional type (value or null)
- **Non-null by default** - Must explicitly mark optionals
- **if and orelse** - Safe ways to unwrap
- **.?** - Unwrap when certain (use sparingly)
- **switch** - Pattern match on optionals
- **Compile-time safety** - No null pointer exceptions

## Summary

You've now mastered:
1. **Enums** - Defining types with fixed variants
2. **Tagged unions** - Enums with different data per variant
3. **Switch** - Exhaustive pattern matching
4. **Optional types** - Null safety built into the type system

Together, these features let you write code that's both expressive and safe. The compiler ensures you handle all cases, preventing entire classes of bugs.

## Next Steps

With enums, structs, and optionals under your belt, you're ready to build robust Zig programs. These building blocks are the foundation for creating custom types that accurately model your domain and prevent invalid states.
