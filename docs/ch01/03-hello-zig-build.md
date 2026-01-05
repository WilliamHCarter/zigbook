# 1.3. Hello, Zig Build!

While `zig build-exe` is useful for simple programs, real-world projects use the Zig build system. Let's learn how to use it!

## Initializing a Zig Project

The `zig init` command creates a new project with a proper build configuration:

```bash
mkdir hello_zig_build
cd hello_zig_build
zig init
```

This creates several files:

- `build.zig` - The build configuration
- `build.zig.zon` - The package manifest
- `src/main.zig` - Your main source file
- `src/root.zig` - Library root (if building a library)

## Exploring the Generated Files

Let's look at `src/main.zig`:

```zig title="src/main.zig"
const std = @import("std");

pub fn main() !void {
    std.debug.print("All your {s} are belong to us.\n", .{"codebase"});
}
```

This is similar to our earlier Hello World, but uses `std.debug.print` instead.

## Building Your Project

To build the project:

```bash
zig build
```

This compiles your program and places the executable in `zig-out/bin/`.

## Running Your Project

To build and run in one command:

```bash
zig build run
```

You should see:

```
All your codebase are belong to us.
```

## Understanding build.zig

The `build.zig` file is a Zig program that describes how to build your project. Here's a simplified version:

```zig title="build.zig"
const std = @import("std");

pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    const exe = b.addExecutable(.{
        .name = "hello_zig_build",
        .root_source_file = b.path("src/main.zig"),
        .target = target,
        .optimize = optimize,
    });

    b.installArtifact(exe);

    const run_cmd = b.addRunArtifact(exe);
    run_cmd.step.dependOn(b.getInstallStep());

    const run_step = b.step("run", "Run the app");
    run_step.dependOn(&run_cmd.step);
}
```

## Build Options

You can customize your build:

```bash
# Build in release mode with optimizations
zig build -Doptimize=ReleaseFast

# Build for a specific target
zig build -Dtarget=x86_64-windows

# Run tests
zig build test
```

## Summary

You now know how to:

- Initialize a Zig project with `zig init`
- Build projects with `zig build`
- Run projects with `zig build run`
- Understand the structure of a Zig project

## Next Steps

Ready to build something more interactive? Continue to [Chapter 2: Programming a Guessing Game](../ch02/index.md).
