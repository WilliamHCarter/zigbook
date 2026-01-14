# 1.1. Installation

The first step to learning Zig is to install it on your system. We'll download Zig from the official website and get it set up so you can start writing Zig programs. You'll need an internet connection for the download.

!!! note "About Zig Versions"
    Zig is still in active development and has not yet reached version 1.0. This means new versions may include breaking changes. The examples in this book are written for Zig 0.13.0 or later. As Zig approaches 1.0, the language is stabilizing, but you may encounter small differences between versions. When in doubt, consult the official Zig documentation at [ziglang.org/documentation](https://ziglang.org/documentation).

## Command Line Notation

Throughout this book, we'll show commands you can use in the terminal. Lines that you should enter in a terminal start with `$`. You don't need to type the `$` character; it's the command line prompt shown to indicate the start of each command. Lines that don't start with `$` typically show the output of the previous command. Additionally, PowerShell-specific examples will use `>` rather than `$`.

## Downloading Zig

Unlike languages that use an installer tool, Zig is distributed as a simple archive file that you extract and add to your system's PATH. This simplicity means you can have multiple versions of Zig installed side-by-side if needed, and there's no complex installation process.

You can download Zig from the official website at [ziglang.org/download](https://ziglang.org/download/). The download page provides builds for all major platforms and architectures.

### Installing Zig on macOS

If you're using macOS, the easiest way to install Zig is through Homebrew:

```bash
$ brew install zig
```

This command downloads and installs the latest stable release of Zig. Homebrew will automatically add Zig to your PATH, so you can use it immediately.

If you prefer not to use Homebrew, you can download the macOS archive directly from the download page and extract it to a location like `/usr/local/zig`, then add it to your PATH manually.

### Installing Zig on Linux

On Linux, you'll download an archive file and extract it to a suitable location. Open a terminal and run these commands:

```bash
$ wget https://ziglang.org/download/0.13.0/zig-linux-x86_64-0.13.0.tar.xz
$ tar -xf zig-linux-x86_64-0.13.0.tar.xz
$ sudo mv zig-linux-x86_64-0.13.0 /usr/local/zig
```

Next, add Zig to your PATH. You can do this temporarily for your current terminal session:

```bash
$ export PATH=$PATH:/usr/local/zig
```

To make this permanent, add that line to your shell configuration file (`~/.bashrc`, `~/.zshrc`, or similar):

```bash
$ echo 'export PATH=$PATH:/usr/local/zig' >> ~/.bashrc
$ source ~/.bashrc
```

!!! tip "Package Managers"
    Some Linux distributions include Zig in their package repositories, though these versions may be outdated. For the latest version, downloading directly from ziglang.org is recommended.

### Installing Zig on Windows

On Windows, you have several options. The simplest is to use a package manager:

**Using Scoop:**

```powershell
> scoop install zig
```

**Using Chocolatey:**

```powershell
> choco install zig
```

Alternatively, you can download the Windows archive from [ziglang.org/download](https://ziglang.org/download/), extract it to a location like `C:\zig`, and add that directory to your PATH:

1. Right-click on "This PC" or "My Computer" and select "Properties"
2. Click "Advanced system settings"
3. Click "Environment Variables"
4. Under "System variables", find and select "Path", then click "Edit"
5. Click "New" and add the path to your Zig installation (e.g., `C:\zig`)
6. Click "OK" to close all dialogs

The rest of this book uses commands that work in both PowerShell and CMD. If there are specific differences, we'll explain which to use.

## Verifying Your Installation

To check whether you have Zig installed correctly, open a terminal (or Command Prompt/PowerShell on Windows) and enter this command:

```bash
$ zig version
```

You should see the version number in a format like this:

```
0.13.0
```

If you see the version number, you have successfully installed Zig! If you don't see this information, check that Zig is in your PATH:

**On Windows CMD:**

```cmd
> echo %PATH%
```

**On PowerShell:**

```powershell
> echo $env:Path
```

**On Linux and macOS:**

```bash
$ echo $PATH
```

Look for the directory where you installed Zig in the output. If it's not there, revisit the installation steps above.

## Updating Zig

Because Zig is under active development and hasn't reached version 1.0 yet, you'll likely want to update it from time to time to get the latest features and improvements. To update Zig, simply repeat the installation process with a newer version.

If you installed via a package manager (Homebrew, Scoop, or Chocolatey), use the package manager's update command:

```bash
$ brew upgrade zig       # macOS with Homebrew
$ scoop update zig       # Windows with Scoop
$ choco upgrade zig      # Windows with Chocolatey
```

## Reading the Official Documentation

Zig includes excellent documentation that you can access online at [ziglang.org/documentation](https://ziglang.org/documentation). This includes:

- The Zig Language Reference - comprehensive documentation of all language features
- The Standard Library documentation - details on every function and type in `std`

Any time you encounter a standard library function and you're not sure what it does or how to use it, the standard library documentation is your best resource!

## Working with Text Editors and IDEs

This book makes no assumptions about what tools you use to write Zig code. Any text editor will work! However, many editors and integrated development environments (IDEs) have support for Zig through plugins:

- **VSCode** - Install the "Zig Language" extension
- **Vim/Neovim** - Use plugins like `ziglang/zig.vim`
- **Emacs** - Use `zig-mode`
- **Sublime Text** - Install the Zig package
- **IntelliJ/CLion** - Install the Zig plugin

Most Zig editor support is powered by `zls` (Zig Language Server), which provides features like autocompletion, go-to-definition, and error highlighting. Visit [github.com/zigtools/zls](https://github.com/zigtools/zls) for installation instructions.

## What About a C Compiler?

One of Zig's unique features is that it can act as a C compiler itself! Zig bundles clang and can compile C and C++ code. This means you don't need a separate C compiler installed to work with Zig, even when interfacing with C libraries. This is particularly convenient on Windows, where setting up a C toolchain can be complex.

However, on macOS, you may still want to install Xcode Command Line Tools for system libraries:

```bash
$ xcode-select --install
```

On Linux, having GCC or Clang installed can be helpful for certain development tasks, though it's not strictly necessary for Zig itself.

## Summary

You now have Zig installed and ready to use! You've also learned:

- Where to download Zig for your platform
- How to verify your installation
- How to update Zig when new versions are released
- Where to find official documentation
- How to set up editor support (optional)

Now that we have Zig installed, let's write your first program in [Hello, World!](02-hello-world.md).
