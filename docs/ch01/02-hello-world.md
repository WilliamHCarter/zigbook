# 1.2. Hello, World!

Now that you've installed Zig, it's time to write your first Zig program. It's traditional when learning a new language to write a little program that prints the text "Hello, world!" to the screen, so we'll do the same here!

!!! note "Command Line Familiarity"
    This book assumes basic familiarity with the command line. Zig makes no specific demands about your editing or tooling or where your code lives, so if you prefer to use an IDE instead of the command line, feel free to use your favorite IDE. Many IDEs now have some degree of Zig support; check the IDE's documentation for details. The Zig community has been working on great IDE support via the Zig Language Server (`zls`).

## Creating a Project Directory

You'll start by making a directory to store your Zig code. It doesn't matter to Zig where your code lives, but for the exercises and projects in this book, we suggest making a `zig-projects` directory in your home directory and keeping all your projects there.

Open a terminal and enter the following commands to make a `zig-projects` directory and a directory for the "Hello, world!" project within it.

**For Linux, macOS, and PowerShell on Windows:**

```bash
$ mkdir ~/zig-projects
$ cd ~/zig-projects
$ mkdir hello_world
$ cd hello_world
```

**For Windows CMD:**

```cmd
> mkdir "%USERPROFILE%\zig-projects"
> cd /d "%USERPROFILE%\zig-projects"
> mkdir hello_world
> cd hello_world
```

## Writing and Running a Zig Program

Next, make a new source file and call it `main.zig`. Zig files always end with the `.zig` extension. If you're using more than one word in your filename, the convention is to use an underscore to separate them. For example, use `hello_world.zig` rather than `helloworld.zig`.

Now open the `main.zig` file you just created and enter the code in Listing 1-1.

```zig title="main.zig"
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    try stdout.print("Hello, world!\n", .{});
}
```

*Listing 1-1: A program that prints "Hello, world!"*

Save the file and go back to your terminal window in the `~/zig-projects/hello_world` directory. On Linux or macOS, enter the following commands to compile and run the file:

```bash
$ zig build-exe main.zig
$ ./main
Hello, world!
```

On Windows, enter the command `.\main` instead of `./main`:

```cmd
> zig build-exe main.zig
> .\main
Hello, world!
```

Regardless of your operating system, the string "Hello, world!" should print to the terminal. If you don't see this output, refer back to the "Verifying Your Installation" part of the Installation section for ways to troubleshoot.

If "Hello, world!" did print, congratulations! You've officially written a Zig program. That makes you a Zig programmer—welcome!

## The Anatomy of a Zig Program

Let's review this "Hello, world!" program in detail. Here's the first piece of the puzzle:

```zig
const std = @import("std");
```

This line imports Zig's standard library. The `@import` function is a built-in function (indicated by the `@` prefix) that loads a package or module. The `"std"` argument tells Zig to import the standard library. We assign it to a constant called `std`, which we'll use to access standard library features.

In Zig, you must explicitly import what you want to use—there are no implicit imports. This makes it clear where functionality comes from.

Next, we define our main function:

```zig
pub fn main() !void {

}
```

These lines define a function named `main`. The `main` function is special: it is always the first code that runs in every executable Zig program. Here's what each part means:

- `pub` makes the function public, so it can be called from outside this file
- `fn` is the keyword for defining a function
- `main` is the name of the function
- `()` indicates the function takes no parameters
- `!void` is the return type, meaning the function might return an error or nothing (we'll discuss this more later)

The function body is wrapped in curly braces `{}`. Zig requires curly braces around all function bodies. It's good style to place the opening curly brace on the same line as the function declaration, adding one space in between.

!!! tip "Automatic Formatting"
    If you want to stick to a standard style across Zig projects, you can use `zig fmt` to automatically format your code. This built-in formatter ensures consistent style without debate. Try it: `zig fmt main.zig`

The body of the `main` function contains this code:

```zig
const stdout = std.io.getStdOut().writer();
try stdout.print("Hello, world!\n", .{});
```

These lines do all the work in this program. Let's break down what's happening:

**First line:** We get a writer to standard output. In Zig, input and output operations go through writers and readers. The `getStdOut()` function returns a `File` representing standard output, and calling `.writer()` on it gives us something we can write to.

**Second line:** We print our message. There are several important details here:

1. The `try` keyword is Zig's error handling mechanism. It says "if this operation returns an error, return that error from this function." Since our `main` function returns `!void`, it can propagate errors upward.

2. `stdout.print()` is a function that writes formatted text. It takes a format string (our message) and a tuple of arguments to format into that string. The `.{}` is an empty tuple because we're not formatting any values into the string—we're just printing the literal text.

3. The `\n` at the end of our string is an escape sequence that represents a newline character. This moves the cursor to the next line after printing.

4. We end the line with a semicolon (`;`), which indicates that this statement is over. Most lines of Zig code end with a semicolon.

## Compiling and Running Are Separate Steps

You've just run a newly created program, so let's examine each step in the process.

Before running a Zig program, you must compile it using the Zig compiler by entering the `zig build-exe` command and passing it the name of your source file, like this:

```bash
$ zig build-exe main.zig
```

If you have a C or C++ background, you'll notice that this is similar to `gcc` or `clang`. After compiling successfully, Zig outputs a binary executable.

On Linux, macOS, and PowerShell on Windows, you can see the executable by entering the `ls` command in your shell:

```bash
$ ls
main  main.zig
```

On Linux and macOS, you'll see two files: the source code file with the `.zig` extension and the executable file (just `main`). On Windows, you'll see three files:

```cmd
> dir /B
main.exe
main.pdb
main.zig
```

This shows the source code file with the `.zig` extension, the executable file (`main.exe` on Windows, `main` on other platforms), and when using Windows, a file containing debugging information with the `.pdb` extension.

From here, you run the `main` or `main.exe` file, like this:

```bash
$ ./main  # or .\main.exe on Windows
```

If your `main.zig` is your "Hello, world!" program, this line prints "Hello, world!" to your terminal.

If you're more familiar with a dynamic language such as Ruby, Python, or JavaScript, you might not be used to compiling and running a program as separate steps. Zig is an ahead-of-time compiled language, meaning you can compile a program and give the executable to someone else, and they can run it even without having Zig installed. If you give someone a `.rb`, `.py`, or `.js` file, they need to have a Ruby, Python, or JavaScript implementation installed (respectively). But in those languages, you only need one command to compile and run your program. Everything is a trade-off in language design.

## The `zig run` Command

For quick experimentation during development, compiling with `zig build-exe` and then running the executable as separate steps can feel cumbersome. Zig provides a shortcut command that does both in one step:

```bash
$ zig run main.zig
Hello, world!
```

The `zig run` command compiles your program and immediately runs it. This is convenient when you're iterating quickly and want to see results fast. Under the hood, it's doing the same compilation as `zig build-exe`, but it handles the execution for you and cleans up the temporary files.

Just compiling with `zig build-exe` is fine for simple programs, but as your project grows, you'll want to manage dependencies, configure build options, and make it easy to share your code. Next, we'll introduce you to Zig's build system, which will help you write real-world Zig programs.

## Summary

You've already accomplished quite a bit! You have:

- Created a project directory
- Written and compiled a Zig program
- Learned about Zig's main function
- Understood how imports work in Zig
- Seen Zig's error handling with `try`
- Explored both compilation and execution steps

These fundamentals will serve you well as you continue learning Zig.

## Next Steps

Hello, world! is fine for a first program, but what about managing larger projects? In the next section, we'll introduce the Zig build system in [Hello, Zig Build!](03-hello-zig-build.md).
