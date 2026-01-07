# 5.1. Defining and Instantiating Structs

Structs let you create custom types by grouping related data together with meaningful names. Let's explore how to define and use them.

## Defining a Struct

To define a struct, use the `struct` keyword followed by field definitions:

```zig
const User = struct {
    active: bool,
    username: []const u8,
    email: []const u8,
    sign_in_count: u64,
};
```

This creates a new type called `User` with four fields. Each field has:
- A name (like `username`)
- A type (like `[]const u8`)

## Instantiating a Struct

Create an instance by specifying values for each field:

```zig
const user1 = User{
    .active = true,
    .username = "someusername123",
    .email = "someone@example.com",
    .sign_in_count = 1,
};
```

**Important syntax notes:**
- Use `.field_name = value` syntax
- Fields can be in any order
- All fields must be initialized (unless they have defaults)

## Accessing Struct Fields

Use dot notation to access fields:

```zig
const std = @import("std");

pub fn main() void {
    const user1 = User{
        .active = true,
        .username = "alice",
        .email = "alice@example.com",
        .sign_in_count = 1,
    };
    
    std.debug.print("Email: {s}\n", .{user1.email});
    std.debug.print("Sign in count: {d}\n", .{user1.sign_in_count});
}
```

## Mutable Struct Instances

To modify fields, the instance must be mutable:

```zig
var user1 = User{
    .active = true,
    .username = "alice",
    .email = "alice@example.com",
    .sign_in_count = 1,
};

user1.email = "newemail@example.com";
user1.sign_in_count += 1;
```

**Note:** In Zig, the entire instance must be mutable - you can't mark individual fields as mutable or immutable.

## Functions That Return Structs

Functions can create and return struct instances:

```zig
fn buildUser(email: []const u8, username: []const u8) User {
    return User{
        .active = true,
        .username = username,
        .email = email,
        .sign_in_count = 1,
    };
}
```

Use it like this:

```zig
const user = buildUser("bob@example.com", "bob123");
```

## Field Init Shorthand

When parameter names match field names, you can use shorthand:

```zig
fn buildUser(email: []const u8, username: []const u8) User {
    return User{
        .active = true,
        .username = username,    // Same as: .username = username
        .email = email,          // Same as: .email = email
        .sign_in_count = 1,
    };
}
```

Actually, Zig doesn't require the repetition - just write the field name:

```zig
fn buildUser(email: []const u8, username: []const u8) User {
    return .{  // Type inferred from return type
        .active = true,
        .username = username,
        .email = email,
        .sign_in_count = 1,
    };
}
```

## Struct Update Syntax

Create a new instance based on an existing one:

```zig
const user2 = User{
    .email = "another@example.com",
    .active = user1.active,
    .username = user1.username,
    .sign_in_count = user1.sign_in_count,
};
```

Zig doesn't have spread syntax like Rust's `..user1`, but you can create helper functions for this pattern.

## Default Field Values

Structs can have default values for fields:

```zig
const Config = struct {
    port: u16 = 8080,
    host: []const u8 = "localhost",
    debug: bool = false,
};

const config1 = Config{};  // Uses all defaults

const config2 = Config{
    .port = 3000,  // Override just the port
};
```

## Tuple Structs

Zig doesn't have named tuple structs, but you can use anonymous structs:

```zig
const Point = struct { f32, f32, f32 };  // Error - fields need names

// Instead, use meaningful names or tuples
const Point = struct {
    x: f32,
    y: f32,
    z: f32,
};

// Or for simple cases, use tuples
const point = .{ 0.0, 1.0, 2.0 };
```

## Anonymous Structs

Zig has powerful anonymous struct support:

```zig
const point = .{
    .x = 10,
    .y = 20,
};

// Type is inferred: struct { x: comptime_int, y: comptime_int }
```

Anonymous structs are useful for:
- Returning multiple values
- Temporary groupings
- Test data

```zig
fn divMod(a: i32, b: i32) struct { quotient: i32, remainder: i32 } {
    return .{
        .quotient = @divTrunc(a, b),
        .remainder = @mod(a, b),
    };
}

const result = divMod(17, 5);
std.debug.print("Q: {d}, R: {d}\n", .{ result.quotient, result.remainder });
```

## Nested Structs

Structs can contain other structs:

```zig
const Address = struct {
    street: []const u8,
    city: []const u8,
    zip: []const u8,
};

const User = struct {
    name: []const u8,
    address: Address,
};

const user = User{
    .name = "Alice",
    .address = .{
        .street = "123 Main St",
        .city = "Springfield",
        .zip = "12345",
    },
};

std.debug.print("City: {s}\n", .{user.address.city});
```

## Packed Structs

Control memory layout with `packed struct`:

```zig
const Flags = packed struct {
    read: bool,
    write: bool,
    execute: bool,
    _padding: u5 = 0,  // Pad to byte boundary
};

// Guaranteed to be exactly 1 byte
const flags = Flags{
    .read = true,
    .write = false,
    .execute = true,
};
```

**When to use packed structs:**
- Bit flags and hardware registers
- Binary protocol compatibility  
- Memory-constrained scenarios

## Extern Structs

For C interoperability, use `extern struct`:

```zig
const CPoint = extern struct {
    x: c_int,
    y: c_int,
};

// Memory layout matches C's struct layout
```

## Complete Example

```zig title="user_struct.zig"
const std = @import("std");

const User = struct {
    name: []const u8,
    age: u32,
    email: []const u8,
    active: bool = true,
    
    // We'll add methods in the next section
};

fn createUser(name: []const u8, age: u32, email: []const u8) User {
    return .{
        .name = name,
        .age = age,
        .email = email,
        // .active uses default value of true
    };
}

pub fn main() void {
    const user1 = createUser("Alice", 30, "alice@example.com");
    
    std.debug.print("User: {s}\n", .{user1.name});
    std.debug.print("Age: {d}\n", .{user1.age});
    std.debug.print("Email: {s}\n", .{user1.email});
    std.debug.print("Active: {}\n", .{user1.active});
    
    var user2 = User{
        .name = "Bob",
        .age = 25,
        .email = "bob@example.com",
    };
    
    user2.age += 1;
    std.debug.print("\nBob's new age: {d}\n", .{user2.age});
}
```

## Memory Ownership

Unlike Rust, Zig structs don't have ownership rules enforced by the compiler. You're responsible for memory management:

```zig
const User = struct {
    name: []const u8,  // Doesn't own the string
    // vs
    // name: std.ArrayList(u8),  // Would own allocated memory
};
```

For structs with allocated data, you typically:
1. Pass an allocator to creation functions
2. Implement a `deinit()` method to free resources
3. Document ownership clearly

We'll explore this more in later chapters.

## Key Takeaways

- **Define structs** with `const TypeName = struct { ... }`
- **Instantiate** with `.field = value` syntax
- **Access fields** with dot notation
- **Defaults** can be specified for fields
- **Type inference** works with `.{ }` syntax
- **Packed structs** control exact memory layout
- **Anonymous structs** are useful for temporary groupings
- **You manage memory** - Zig doesn't enforce ownership

## Next Steps

Now that you know how to define structs, let's build a complete program using them in [An Example Program Using Structs](02-example-program-using-structs.md).
