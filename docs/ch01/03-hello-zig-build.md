# 1.3. Hello, Zig Build!

While compiling with `zig build-exe` works fine for simple, single-file programs, real-world projects quickly become more complex. You'll want to manage multiple source files, link external libraries, configure different build modes, and make it easy to share your project with others. That's where Zig's build system comes in.

The build system is Zig's answer to Makefiles, CMake, or other build tools you might have used in other languages. The interesting twist? Your build configuration is written in Zig itself, making it type-safe, cross-platform, and as powerful as you need it to be.

## Creating a Project with `zig init`

Let's create a new project using Zig's project initialization command. Navigate to your `zig-projects` directory and run:

```bash
$ cd ~/zig-projects
$ mkdir hello_zig_build
$ cd hello_zig_build
$ zig init
info: created build.zig
info: created build.zig.zon
info: created src/main.zig
info: created src/root.zig
info: see `zig build --help` for a menu of options
```

The `zig init` command creates a project structure with several files. Let's explore what each one does.

## Project Structure

Your new project has this structure:

```
hello_zig_build/
├── build.zig          # Build configuration
├── build.zig.zon      # Package dependencies
└── src/
    ├── main.zig       # Application entry point
    └── root.zig       # Library root
```

Let's examine each file:

### src/main.zig

This is the main entry point for your executable:

```zig title="src/main.zig"
const std = @import("std");

pub fn main() !void {
    std.debug.print("All your {s} are belong to us.\n", .{"codebase"});
}
```

This looks similar to our earlier Hello World program, but with a few differences:

- It uses `std.debug.print` instead of getting a stdout writer
- It demonstrates string formatting with `{s}` and a tuple argument `.{"codebase"}`

The `{s}` is a format specifier for strings, and the tuple `.{"codebase"}` provides the value to format into that position.

### build.zig

This file is a Zig program that describes how to build your project. Here's a simplified look at what `zig init` generates:

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

Don't worry if you don't understand all of this yet! The key points are:

- The `build` function is called when you run `zig build`
- It creates an executable from `src/main.zig`
- It sets up a "run" step so you can execute your program
- It's just Zig code, so you can use the full power of the language

### build.zig.zon

This file describes your package and its dependencies:

```zig title="build.zig.zon"
.{
    .name = "hello_zig_build",
    .version = "0.0.0",
    .dependencies = .{},
    .paths = .{
        "build.zig",
        "build.zig.zon",
        "src",
    },
}
```

For now, we don't have any dependencies, but when you add external packages, they'll be listed here. This file uses ZON (Zig Object Notation), which is similar to JSON but follows Zig's syntax.

## Building Your Project

Now let's build the project. From your `hello_zig_build` directory, run:

```bash
$ zig build
```

That's it! The build system:

1. Reads your `build.zig` configuration
2. Compiles all necessary source files
3. Produces an executable in `zig-out/bin/`

You won't see much output unless there are errors. You can check the result:

```bash
$ ls zig-out/bin/
hello_zig_build
```

## Running Your Project

You can run the compiled executable directly:

```bash
$ ./zig-out/bin/hello_zig_build
All your codebase are belong to us.
```

But there's a more convenient way! Remember the "run" step in `build.zig`? You can use it:

```bash
$ zig build run
All your codebase are belong to us.
```

The `zig build run` command builds your project (if needed) and immediately runs it. This is the command you'll use most often during development.

## Exploring Build Options

The Zig build system offers many options. You can see them all with:

```bash
$ zig build --help
```

Here are some useful ones:

### Optimization Modes

By default, projects build in Debug mode, which includes safety checks and makes debugging easier. You can choose different optimization levels:

```bash
$ zig build -Doptimize=Debug        # Default: no optimizations, all safety checks
$ zig build -Doptimize=ReleaseSafe  # Optimized but keeps safety checks
$ zig build -Doptimize=ReleaseFast  # Fully optimized, removes safety checks
$ zig build -Doptimize=ReleaseSmall # Optimized for size
```

Try building in release mode:

```bash
$ zig build -Doptimize=ReleaseFast
$ ./zig-out/bin/hello_zig_build
All your codebase are belong to us.
```

The program works the same but runs faster and produces a smaller binary.

### Cross-Compilation

One of Zig's superpowers is trivial cross-compilation. Want to build for Windows while on Linux? Just specify the target:

```bash
$ zig build -Dtarget=x86_64-windows
```

Zig can target any supported platform without installing separate toolchains. The built-in C compiler means you don't need platform-specific SDKs for most builds.

Common targets include:

- `x86_64-linux` - 64-bit Linux
- `x86_64-windows` - 64-bit Windows
- `x86_64-macos` - 64-bit macOS
- `aarch64-linux` - 64-bit ARM Linux (like Raspberry Pi)
- `wasm32-wasi` - WebAssembly

## Running Tests

The `zig init` command also sets up a test step. You'll see tests in `src/root.zig`:

```bash
$ zig build test
All 1 tests passed.
```

We'll explore testing in detail in Chapter 8.

## Modifying Your Project

Let's modify `src/main.zig` to print something different:

```zig title="src/main.zig"
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    try stdout.print("Hello from the Zig build system!\n", .{});
}
```

Now rebuild and run:

```bash
$ zig build run
Hello from the Zig build system!
```

The build system detected your change and recompiled automatically. This incremental compilation makes the build system fast even for larger projects.

## Understanding the Workflow

Here's the typical development workflow with the Zig build system:

1. **Initialize**: `zig init` creates your project structure
2. **Edit**: Modify source files in `src/`
3. **Build and Run**: `zig build run` compiles and executes
4. **Test**: `zig build test` runs your test suite
5. **Release**: `zig build -Doptimize=ReleaseFast` creates optimized binaries

## Why Use the Build System?

You might wonder: why not just use `zig build-exe` for everything? Here's what the build system gives you:

**Organization**: Clear project structure with sensible defaults

**Incremental builds**: Only recompiles what changed

**Dependencies**: Easy integration of external packages

**Testing**: Built-in test runner

**Cross-compilation**: Target any platform with simple flags

**Customization**: Full power of Zig to script your build

**Shareable**: Others can build your project with just `zig build`

As your projects grow beyond a single file, these benefits become essential.

## Summary

You've learned how to use Zig's build system! You now know how to:

- Create a new project with `zig init`
- Understand the project structure
- Build projects with `zig build`
- Run projects with `zig build run`
- Use different optimization modes
- Cross-compile to different platforms
- Run tests with `zig build test`

The build system might seem like extra complexity for a simple Hello World, but it scales beautifully as your projects grow. Every real Zig project you encounter will use this build system, so understanding it early will serve you well.

## Next Steps

You now have all the tools you need to write Zig programs! You can create projects, build them, and run them. In the next chapter, we'll put these tools to use by building something more interesting: a guessing game that demonstrates Zig's fundamental features. Continue to [Chapter 2: Programming a Guessing Game](../ch02/index.md).
