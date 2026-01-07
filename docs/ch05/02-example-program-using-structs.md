# 5.2. An Example Program Using Structs

To understand when structs are useful, let's write a program that calculates the area of a rectangle. We'll start with simple variables and refactor toward using structs.

## Version 1: Separate Variables

Let's start with the most basic approach:

```zig title="rectangles_v1.zig"
const std = @import("std");

fn area(width: u32, height: u32) u32 {
    return width * height;
}

pub fn main() void {
    const width = 30;
    const height = 50;
    
    std.debug.print(
        "The area of the rectangle is {} square pixels.\n",
        .{area(width, height)},
    );
}
```

Run it:
```bash
$ zig build-exe rectangles_v1.zig
$ ./rectangles_v1
The area of the rectangle is 1500 square pixels.
```

**The problem:** The `area` function takes two parameters, but nothing indicates they're related. The function signature doesn't communicate that we're calculating the area of a rectangle.

## Version 2: Using Tuples

Let's group the dimensions with a tuple:

```zig title="rectangles_v2.zig"
const std = @import("std");

fn area(dimensions: struct { u32, u32 }) u32 {
    return dimensions[0] * dimensions[1];
}

pub fn main() void {
    const rect = .{ 30, 50 };
    
    std.debug.print(
        "The area of the rectangle is {} square pixels.\n",
        .{area(rect)},
    );
}
```

**Better, but:** While we're now passing a single value, we've lost clarity. Which index is width and which is height? The code doesn't say.

## Version 3: Using an Anonymous Struct

Let's add names to our fields:

```zig title="rectangles_v3.zig"
const std = @import("std");

fn area(rectangle: anytype) u32 {
    return rectangle.width * rectangle.height;
}

pub fn main() void {
    const rect = .{
        .width = 30,
        .height = 50,
    };
    
    std.debug.print(
        "The area of the rectangle is {} square pixels.\n",
        .{area(rect)},
    );
}
```

**Much better!** Named fields make the code self-documenting. But we can do even better by defining a proper type.

## Version 4: Using a Defined Struct

Let's create a dedicated `Rectangle` type:

```zig title="rectangles_v4.zig"
const std = @import("std");

const Rectangle = struct {
    width: u32,
    height: u32,
};

fn area(rectangle: Rectangle) u32 {
    return rectangle.width * rectangle.height;
}

pub fn main() void {
    const rect = Rectangle{
        .width = 30,
        .height = 50,
    };
    
    std.debug.print(
        "The area of the rectangle is {} square pixels.\n",
        .{area(rect)},
    );
}
```

**Perfect!** Now:
- The type name documents what we're working with
- Field names clarify the meaning of each value
- The function signature is crystal clear

## Adding Debug Printing

Let's say we want to print the rectangle while debugging. We can implement custom formatting:

```zig
const std = @import("std");

const Rectangle = struct {
    width: u32,
    height: u32,
    
    pub fn format(
        self: Rectangle,
        comptime fmt: []const u8,
        options: std.fmt.FormatOptions,
        writer: anytype,
    ) !void {
        _ = fmt;
        _ = options;
        try writer.print("Rectangle {{ width: {d}, height: {d} }}", .{
            self.width,
            self.height,
        });
    }
};

pub fn main() !void {
    const rect = Rectangle{
        .width = 30,
        .height = 50,
    };
    
    std.debug.print("rect is {}\n", .{rect});
}
```

Output:
```
rect is Rectangle { width: 30, height: 50 }
```

## Simpler Debug Printing

For quick debugging, use `std.debug.print` with `{any}`:

```zig
const rect = Rectangle{
    .width = 30,
    .height = 50,
};

std.debug.print("rect is {any}\n", .{rect});
```

Output:
```
rect is main.Rectangle{ .width = 30, .height = 50 }
```

The `{any}` format specifier automatically prints all struct fields - perfect for debugging!

## Pretty Printing

For more readable output, you can write your own print function:

```zig
fn printRectangle(rect: Rectangle) void {
    std.debug.print(
        \\Rectangle:
        \\  width: {d}
        \\  height: {d}
        \\
    , .{ rect.width, rect.height });
}

pub fn main() void {
    const rect = Rectangle{
        .width = 30,
        .height = 50,
    };
    
    printRectangle(rect);
}
```

Output:
```
Rectangle:
  width: 30
  height: 50
```

## Multiple Rectangles

Let's work with multiple rectangles:

```zig title="multiple_rectangles.zig"
const std = @import("std");

const Rectangle = struct {
    width: u32,
    height: u32,
};

fn area(rectangle: Rectangle) u32 {
    return rectangle.width * rectangle.height;
}

pub fn main() void {
    const rect1 = Rectangle{
        .width = 30,
        .height = 50,
    };
    
    const rect2 = Rectangle{
        .width = 10,
        .height = 40,
    };
    
    const rect3 = Rectangle{
        .width = 60,
        .height = 45,
    };
    
    std.debug.print("Area of rect1: {d}\n", .{area(rect1)});
    std.debug.print("Area of rect2: {d}\n", .{area(rect2)});
    std.debug.print("Area of rect3: {d}\n", .{area(rect3)});
}
```

## Adding More Functionality

Let's add more rectangle operations:

```zig
const Rectangle = struct {
    width: u32,
    height: u32,
};

fn area(rectangle: Rectangle) u32 {
    return rectangle.width * rectangle.height;
}

fn perimeter(rectangle: Rectangle) u32 {
    return 2 * (rectangle.width + rectangle.height);
}

fn canHold(self: Rectangle, other: Rectangle) bool {
    return self.width > other.width and self.height > other.height;
}

pub fn main() void {
    const rect1 = Rectangle{ .width = 30, .height = 50 };
    const rect2 = Rectangle{ .width = 10, .height = 40 };
    const rect3 = Rectangle{ .width = 60, .height = 45 };
    
    std.debug.print("Area: {d}\n", .{area(rect1)});
    std.debug.print("Perimeter: {d}\n", .{perimeter(rect1)});
    std.debug.print("Can rect1 hold rect2? {}\n", .{canHold(rect1, rect2)});
    std.debug.print("Can rect1 hold rect3? {}\n", .{canHold(rect1, rect3)});
}
```

Output:
```
Area: 1500
Perimeter: 160
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

## The Problem with Free Functions

Notice that all our functions are "free functions" - they're not associated with the `Rectangle` type in any special way. This means:

1. Functions could work on any type with `width` and `height` fields
2. No clear organization - which functions are for `Rectangle`?
3. Users must import functions separately from the type

What we want is to tie these functions directly to our `Rectangle` type. That's where methods come in!

## Preview: Methods

Here's a sneak peek at what we'll learn next:

```zig
const Rectangle = struct {
    width: u32,
    height: u32,
    
    pub fn area(self: Rectangle) u32 {
        return self.width * self.height;
    }
    
    pub fn canHold(self: Rectangle, other: Rectangle) bool {
        return self.width > other.width and self.height > other.height;
    }
};

pub fn main() void {
    const rect1 = Rectangle{ .width = 30, .height = 50 };
    const rect2 = Rectangle{ .width = 10, .height = 40 };
    
    // Call methods on the struct!
    std.debug.print("Area: {d}\n", .{rect1.area()});
    std.debug.print("Can hold? {}\n", .{rect1.canHold(rect2)});
}
```

Much better! The functions are now clearly part of the `Rectangle` type.

## Complete Example

Here's our complete rectangle program before adding methods:

```zig title="rectangles_complete.zig"
const std = @import("std");

const Rectangle = struct {
    width: u32,
    height: u32,
};

fn area(rectangle: Rectangle) u32 {
    return rectangle.width * rectangle.height;
}

fn perimeter(rectangle: Rectangle) u32 {
    return 2 * (rectangle.width + rectangle.height);
}

fn canHold(self: Rectangle, other: Rectangle) bool {
    return self.width > other.width and self.height > other.height;
}

pub fn main() void {
    const rect1 = Rectangle{
        .width = 30,
        .height = 50,
    };
    
    const rect2 = Rectangle{
        .width = 10,
        .height = 40,
    };
    
    const rect3 = Rectangle{
        .width = 60,
        .height = 45,
    };
    
    std.debug.print("Rectangle 1: {any}\n", .{rect1});
    std.debug.print("  Area: {d}\n", .{area(rect1)});
    std.debug.print("  Perimeter: {d}\n", .{perimeter(rect1)});
    std.debug.print("\n", .{});
    
    std.debug.print("Can rect1 hold rect2? {}\n", .{canHold(rect1, rect2)});
    std.debug.print("Can rect1 hold rect3? {}\n", .{canHold(rect1, rect3)});
}
```

## Key Takeaways

- **Start simple** - Variables work for basic cases
- **Add structure** - Group related data
- **Add meaning** - Use named fields
- **Create types** - Define structs for clarity
- **Debug easily** - Use `{any}` or custom formatting
- **Free functions work** - But methods are better (next section!)

## Next Steps

We've seen how structs improve code organization. Now let's learn how to add behavior to our types with [Methods](03-methods.md).
