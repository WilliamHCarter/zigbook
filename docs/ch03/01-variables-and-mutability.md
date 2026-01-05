# 3.1. Variables and Mutability

Variables are fundamental to any programming language. In Zig, variables are immutable by default, which helps prevent bugs.

## Declaring Variables

Use `const` to declare an immutable variable:

```zig
const x = 5;
```

Once set, the value of `x` cannot be changed. Trying to reassign it will cause a compile error:

```zig
const x = 5;
x = 6; // Error: cannot assign to constant
```

## Mutable Variables

To create a mutable variable, use `var`:

```zig
var x = 5;
x = 6; // This is fine!
```

## Type Inference and Explicit Types

Zig can infer types from the value you assign:

```zig
const x = 5; // Zig infers this is an integer
const name = "Alice"; // Zig infers this is a string
```

You can also explicitly specify the type:

```zig
const x: i32 = 5;
const y: f64 = 3.14;
```

## Undefined Values

Variables can be declared without an initial value using `undefined`:

```zig
var x: i32 = undefined;
x = 5; // Must assign before use
```

!!! warning
    Using an undefined variable before assigning it leads to undefined behavior. Zig's safety features try to catch this at compile time when possible.

## Constants vs Variables

Choose between `const` and `var` based on whether the value needs to change:

- Use `const` by default - it's safer and can enable optimizations
- Use `var` only when you need to modify the value

```zig
const maximum_points = 100; // Won't change
var current_points = 0;     // Will change as player scores
```

## Shadowing

Zig allows variable shadowing - declaring a new variable with the same name:

```zig
const x = 5;
{
    const x = 10; // Different variable in inner scope
    // x is 10 here
}
// x is 5 here
```

## Example Program

Here's a complete example:

```zig title="variables.zig"
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    
    // Immutable variable
    const max_health = 100;
    try stdout.print("Max health: {d}\n", .{max_health});
    
    // Mutable variable
    var current_health = 100;
    try stdout.print("Current health: {d}\n", .{current_health});
    
    // Taking damage
    current_health -= 25;
    try stdout.print("Health after damage: {d}\n", .{current_health});
    
    // Healing
    current_health += 10;
    try stdout.print("Health after healing: {d}\n", .{current_health});
}
```

## Key Takeaways

- Variables are immutable by default with `const`
- Use `var` for mutable variables
- Zig encourages immutability for safer code
- Type inference is available but you can specify types explicitly

## Next Steps

Now that you understand variables, let's explore [Data Types](02-data-types.md) in depth.
