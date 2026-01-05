# 1.1. Installation

The first step to using Zig is to install it on your system.

## Downloading Zig

You can download Zig from the official website at [ziglang.org/download](https://ziglang.org/download/).

### Installation Options

=== "macOS"

    ```bash
    # Using Homebrew
    brew install zig
    ```

=== "Linux"

    ```bash
    # Download and extract the latest version
    # Visit https://ziglang.org/download/ for the latest URL
    wget https://ziglang.org/download/latest/zig-linux-x86_64.tar.xz
    tar -xf zig-linux-x86_64.tar.xz
    sudo mv zig-linux-x86_64 /usr/local/zig
    export PATH=$PATH:/usr/local/zig
    ```

=== "Windows"

    Download the Windows installer from [ziglang.org/download](https://ziglang.org/download/) and run it.
    
    Alternatively, use a package manager:
    
    ```powershell
    # Using Scoop
    scoop install zig
    
    # Using Chocolatey
    choco install zig
    ```

## Verifying Installation

After installation, verify that Zig is installed correctly:

```bash
zig version
```

You should see output showing the version of Zig you installed.

## Updating Zig

Zig is under active development. To stay current:

- Check the [Zig download page](https://ziglang.org/download/) regularly
- Follow the same installation process to upgrade to a newer version

## Next Steps

Now that you have Zig installed, let's write your first program in [Hello, World!](02-hello-world.md).
