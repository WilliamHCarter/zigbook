# Chapter 5: Using Structs to Structure Related Data

A **struct** (structure) is a custom data type that lets you package together and name multiple related values that make up a meaningful group. If you're familiar with object-oriented programming, a struct is like an object's data attributes.

In this chapter, we'll explore how to use structs to create custom types that are meaningful for your domain. Structs are one of the fundamental building blocks for creating well-organized Zig programs.

## What You'll Learn

We'll cover:

1. **Defining and Instantiating Structs** - How to create custom types
2. **An Example Program** - Building a rectangle area calculator
3. **Methods** - Adding behavior to your structs

## Why Structs Matter

Structs help you:

- **Group related data** - Keep associated values together
- **Add meaning** - Name your data to make code clearer
- **Build abstractions** - Create types that model your domain
- **Organize code** - Keep data and behavior together

## A Quick Preview

Here's a taste of what we'll build:

```zig
const Rectangle = struct {
    width: u32,
    height: u32,
    
    pub fn area(self: Rectangle) u32 {
        return self.width * self.height;
    }
};

const rect = Rectangle{
    .width = 30,
    .height = 50,
};

const area_value = rect.area(); // 1500
```

Notice how we:
- Define fields with names and types
- Create instances with named field syntax
- Add methods that operate on the struct

## Structs vs Other Types

You've already seen several ways to group data:

**Arrays** - Same type, fixed or dynamic size:
```zig
const numbers = [_]i32{ 1, 2, 3 };
```

**Tuples** - Different types, anonymous fields:
```zig
const tuple = .{ "Alice", 25, true };
```

**Structs** - Different types, named fields:
```zig
const User = struct {
    name: []const u8,
    age: u32,
    active: bool,
};
```

Structs give you the best of both: different types AND meaningful names.

## Getting Started

Ready to create your own types? Let's begin with [Defining and Instantiating Structs](01-defining-and-instantiating-structs.md).
