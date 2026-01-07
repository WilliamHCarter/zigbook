# 5.3. Methods

Methods are functions that are defined within a struct and operate on instances of that struct. They provide a clean way to associate behavior with your types.

## What Are Methods?

In Zig, methods are simply functions defined inside a struct. The first parameter is typically `self`, representing the instance the method is called on.

```zig
const Rectangle = struct {
    width: u32,
    height: u32,
    
    // This is a method
    pub fn area(self: Rectangle) u32 {
        return self.width * self.height;
    }
};
```

## Calling Methods

Use dot notation to call methods:

```zig
const rect = Rectangle{
    .width = 30,
    .height = 50,
};

const area_value = rect.area();
```

This is more intuitive than calling a free function:

```zig
// Free function style
const area_value = area(rect);

// Method style
const area_value = rect.area();
```

## Defining Methods

Let's convert our rectangle functions to methods:

```zig
const std = @import("std");

const Rectangle = struct {
    width: u32,
    height: u32,
    
    pub fn area(self: Rectangle) u32 {
        return self.width * self.height;
    }
    
    pub fn perimeter(self: Rectangle) u32 {
        return 2 * (self.width + self.height);
    }
    
    pub fn canHold(self: Rectangle, other: Rectangle) bool {
        return self.width > other.width and self.height > other.height;
    }
};

pub fn main() void {
    const rect1 = Rectangle{ .width = 30, .height = 50 };
    const rect2 = Rectangle{ .width = 10, .height = 40 };
    
    std.debug.print("Area: {d}\n", .{rect1.area()});
    std.debug.print("Perimeter: {d}\n", .{rect1.perimeter()});
    std.debug.print("Can hold rect2? {}\n", .{rect1.canHold(rect2)});
}
```

## The `self` Parameter

The `self` parameter can take different forms:

### By Value

```zig
pub fn area(self: Rectangle) u32 {
    return self.width * self.height;
}
```

Copies the struct (fine for small structs).

### By Reference (Immutable)

```zig
pub fn area(self: *const Rectangle) u32 {
    return self.width * self.height;
}
```

Passes a pointer to the struct - no copy.

### By Reference (Mutable)

```zig
pub fn grow(self: *Rectangle, amount: u32) void {
    self.width += amount;
    self.height += amount;
}
```

Allows modifying the struct.

## Choosing the Right Self Type

**Use `self: Type`** when:
- The struct is small (a few fields)
- You don't need to modify it
- You want simple semantics

**Use `self: *const Type`** when:
- The struct is large
- You don't need to modify it
- You want to avoid copying

**Use `self: *Type`** when:
- You need to modify the struct
- Size doesn't matter

### Example with All Three

```zig
const Counter = struct {
    count: u32,
    
    // By value - small struct, read-only
    pub fn value(self: Counter) u32 {
        return self.count;
    }
    
    // By mutable reference - modifies the struct
    pub fn increment(self: *Counter) void {
        self.count += 1;
    }
    
    // By const reference - large struct, read-only
    pub fn display(self: *const Counter) void {
        std.debug.print("Count: {d}\n", .{self.count});
    }
};

pub fn main() void {
    var counter = Counter{ .count = 0 };
    
    counter.increment();
    counter.increment();
    std.debug.print("Value: {d}\n", .{counter.value()});
}
```

## Methods with Multiple Parameters

Methods can take additional parameters after `self`:

```zig
const Rectangle = struct {
    width: u32,
    height: u32,
    
    pub fn canHold(self: Rectangle, other: Rectangle) bool {
        return self.width > other.width and self.height > other.height;
    }
    
    pub fn scaleBy(self: *Rectangle, factor: u32) void {
        self.width *= factor;
        self.height *= factor;
    }
};

pub fn main() void {
    const rect1 = Rectangle{ .width = 30, .height = 50 };
    const rect2 = Rectangle{ .width = 10, .height = 40 };
    
    std.debug.print("Can hold? {}\n", .{rect1.canHold(rect2)});
    
    var rect3 = Rectangle{ .width = 5, .height = 10 };
    rect3.scaleBy(3);
    std.debug.print("Scaled: {d}x{d}\n", .{ rect3.width, rect3.height });
}
```

## Associated Functions (Constructors)

Functions in a struct that don't take `self` are called **associated functions**. They're often used as constructors:

```zig
const Rectangle = struct {
    width: u32,
    height: u32,
    
    // Associated function - no self parameter
    pub fn init(width: u32, height: u32) Rectangle {
        return Rectangle{
            .width = width,
            .height = height,
        };
    }
    
    // Associated function for creating a square
    pub fn square(size: u32) Rectangle {
        return Rectangle{
            .width = size,
            .height = size,
        };
    }
    
    pub fn area(self: Rectangle) u32 {
        return self.width * self.height;
    }
};

pub fn main() void {
    const rect = Rectangle.init(30, 50);
    const sq = Rectangle.square(25);
    
    std.debug.print("Rectangle area: {d}\n", .{rect.area()});
    std.debug.print("Square area: {d}\n", .{sq.area()});
}
```

Call associated functions with `Type.function()` syntax, not on an instance.

## Public vs Private

Use `pub` to make methods public (accessible from other files):

```zig
const Rectangle = struct {
    width: u32,
    height: u32,
    
    // Public - can be called from other files
    pub fn area(self: Rectangle) u32 {
        return self.width * self.height;
    }
    
    // Private - only usable within this file
    fn validate(self: Rectangle) bool {
        return self.width > 0 and self.height > 0;
    }
};
```

## Methods That Return Errors

Methods can return errors just like functions:

```zig
const Rectangle = struct {
    width: u32,
    height: u32,
    
    pub fn init(width: u32, height: u32) !Rectangle {
        if (width == 0 or height == 0) {
            return error.InvalidDimensions;
        }
        return Rectangle{
            .width = width,
            .height = height,
        };
    }
};

pub fn main() !void {
    const rect = try Rectangle.init(30, 50);
    // const bad = try Rectangle.init(0, 50);  // Would return error
}
```

## Getters and Setters

While not required, you can create getter/setter patterns:

```zig
const Rectangle = struct {
    width: u32,
    height: u32,
    
    // Getter
    pub fn getWidth(self: Rectangle) u32 {
        return self.width;
    }
    
    // Setter
    pub fn setWidth(self: *Rectangle, width: u32) void {
        self.width = width;
    }
};
```

However, in Zig, direct field access is common and preferred unless you need validation or computed values.

## Complete Example

Here's a comprehensive example with various method types:

```zig title="rectangle_methods.zig"
const std = @import("std");

const Rectangle = struct {
    width: u32,
    height: u32,
    
    // Constructor (associated function)
    pub fn init(width: u32, height: u32) Rectangle {
        return .{ .width = width, .height = height };
    }
    
    // Factory method for squares
    pub fn square(size: u32) Rectangle {
        return .{ .width = size, .height = size };
    }
    
    // Read-only method (by value)
    pub fn area(self: Rectangle) u32 {
        return self.width * self.height;
    }
    
    // Read-only method (by const reference)
    pub fn perimeter(self: *const Rectangle) u32 {
        return 2 * (self.width + self.height);
    }
    
    // Method that takes another instance
    pub fn canHold(self: Rectangle, other: Rectangle) bool {
        return self.width > other.width and self.height > other.height;
    }
    
    // Mutating method
    pub fn scale(self: *Rectangle, factor: u32) void {
        self.width *= factor;
        self.height *= factor;
    }
    
    // Method with validation
    pub fn setDimensions(self: *Rectangle, width: u32, height: u32) !void {
        if (width == 0 or height == 0) {
            return error.InvalidDimensions;
        }
        self.width = width;
        self.height = height;
    }
    
    // Computed property
    pub fn isSquare(self: Rectangle) bool {
        return self.width == self.height;
    }
};

pub fn main() !void {
    // Using constructors
    const rect1 = Rectangle.init(30, 50);
    const rect2 = Rectangle.square(20);
    
    // Calling methods
    std.debug.print("Rect1 area: {d}\n", .{rect1.area()});
    std.debug.print("Rect1 perimeter: {d}\n", .{rect1.perimeter()});
    std.debug.print("Is square? {}\n", .{rect1.isSquare()});
    
    std.debug.print("\nRect2 area: {d}\n", .{rect2.area()});
    std.debug.print("Is square? {}\n", .{rect2.isSquare()});
    
    // Comparing rectangles
    std.debug.print("\nCan rect1 hold rect2? {}\n", .{rect1.canHold(rect2)});
    
    // Mutating a rectangle
    var rect3 = Rectangle.init(10, 15);
    std.debug.print("\nBefore scale: {d}x{d}\n", .{ rect3.width, rect3.height });
    rect3.scale(2);
    std.debug.print("After scale: {d}x{d}\n", .{ rect3.width, rect3.height });
    
    // Using error-returning method
    try rect3.setDimensions(25, 25);
    std.debug.print("New dimensions: {d}x{d}\n", .{ rect3.width, rect3.height });
}
```

## Methods vs Free Functions

**Use methods when:**
- The function logically operates on the type
- You want to group behavior with data
- You want dot-notation calling syntax

**Use free functions when:**
- The function operates on multiple types equally
- The function is a utility that doesn't belong to a type
- You're writing generic code

Both are valid in Zig - choose based on what makes your code clearest.

## Organization

All methods and associated functions for a type go inside the struct definition:

```zig
const MyType = struct {
    field1: i32,
    field2: bool,
    
    pub fn init() MyType { ... }
    
    pub fn method1(self: MyType) void { ... }
    
    pub fn method2(self: *MyType) void { ... }
    
    fn privateHelper(self: MyType) i32 { ... }
};
```

This keeps everything together and makes your API clear.

## Key Takeaways

- **Methods** - Functions inside structs with `self` parameter
- **Call with dot notation** - `rect.area()` not `area(rect)`
- **Self types** - By value, const pointer, or mutable pointer
- **Associated functions** - No `self`, often used as constructors
- **pub keyword** - Makes methods public
- **Organize behavior** - Keep related functions with the type
- **Both styles valid** - Methods and free functions each have uses

## Summary

You've now learned how to:
1. Define custom types with structs
2. Build programs using structs
3. Add behavior with methods

Structs are fundamental building blocks in Zig. Combined with enums (which we'll cover later), they let you create rich, expressive types that model your problem domain clearly.

## Next Steps

With structs mastered, you're ready to explore more advanced topics. Keep building programs that use structs to organize your data and behavior!
