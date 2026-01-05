# 1.2. Hello, World!

It's tradition when learning a new programming language to write a program that prints "Hello, World!" to the screen. Let's follow that tradition!

## Creating a Project Directory

First, create a directory for your Zig projects:

```bash
mkdir ~/zig-projects
cd ~/zig-projects
mkdir hello_world
cd hello_world
```

## Writing Your First Zig Program

Create a new file called `main.zig` and add the following code:

```zig title="main.zig"
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    try stdout.print("Hello, World!\n", .{});
}
```

Let's break down what this code does:

- `const std = @import("std");` imports Zig's standard library
- `pub fn main() !void` defines the main function, which is the entry point of the program
- The `!void` return type indicates the function may return an error
- We get a writer to standard output
- `try stdout.print(...)` prints our message, propagating any errors

## Compiling and Running

Now compile and run your program:

```bash
zig build-exe main.zig
./main
```

You should see:

```
Hello, World!
```

Congratulations! You've written and run your first Zig program.

## Alternative: Compile and Run in One Step

You can also use `zig run` to compile and execute in one command:

```bash
zig run main.zig
```

This is convenient during development when you want quick feedback.

## Next Steps

In the next section, we'll learn about the Zig build system: [Hello, Zig Build!](03-hello-zig-build.md).
