# Chapter 2: Programming a Guessing Game

Let's learn Zig by building a fun, interactive program: a guessing game!

## What We'll Build

In this chapter, we'll build a program that:

1. Generates a random number between 1 and 100
2. Prompts the user to guess the number
3. Tells the user if their guess is too high or too low
4. Congratulates the user when they guess correctly

## What You'll Learn

Along the way, you'll learn about:

- Reading user input
- Working with random numbers
- Using comparison operators
- Control flow with loops
- Error handling
- String parsing

## The Complete Program

Here's what our finished program will look like:

```zig title="src/main.zig"
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    const stdin = std.io.getStdIn().reader();

    var prng = std.Random.DefaultPrng.init(@intCast(std.time.timestamp()));
    const random = prng.random();
    const secret_number = random.intRangeAtMost(u8, 1, 100);

    try stdout.print("Guess the number!\n", .{});

    var buf: [10]u8 = undefined;
    
    while (true) {
        try stdout.print("Please input your guess (1-100): ", .{});
        
        const input = try stdin.readUntilDelimiter(&buf, '\n');
        const trimmed = std.mem.trim(u8, input, &std.ascii.whitespace);
        
        const guess = std.fmt.parseInt(u8, trimmed, 10) catch {
            try stdout.print("Please enter a valid number!\n", .{});
            continue;
        };

        try stdout.print("You guessed: {d}\n", .{guess});

        if (guess < secret_number) {
            try stdout.print("Too small!\n", .{});
        } else if (guess > secret_number) {
            try stdout.print("Too big!\n", .{});
        } else {
            try stdout.print("You win!\n", .{});
            break;
        }
    }
}
```

## Let's Build It!

We'll build this program step by step, explaining each concept as we go.

### Setting Up

Create a new project:

```bash
mkdir guessing_game
cd guessing_game
zig init
```

Now open `src/main.zig` and let's start coding!

### Step 1: Getting User Input

First, let's read input from the user:

```zig
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    const stdin = std.io.getStdIn().reader();

    try stdout.print("Please input your guess: ", .{});
    
    var buf: [10]u8 = undefined;
    const input = try stdin.readUntilDelimiter(&buf, '\n');
    
    try stdout.print("You guessed: {s}\n", .{input});
}
```

Run it with `zig build run` and try entering a number!

### Step 2: Generating a Random Number

Now let's generate a secret number to guess:

```zig
var prng = std.Random.DefaultPrng.init(@intCast(std.time.timestamp()));
const random = prng.random();
const secret_number = random.intRangeAtMost(u8, 1, 100);
```

This creates a random number between 1 and 100 using the current timestamp as a seed.

### Step 3: Comparing the Guess

We need to parse the user's input and compare it:

```zig
const trimmed = std.mem.trim(u8, input, &std.ascii.whitespace);
const guess = std.fmt.parseInt(u8, trimmed, 10) catch {
    try stdout.print("Please enter a valid number!\n", .{});
    return;
};

if (guess < secret_number) {
    try stdout.print("Too small!\n", .{});
} else if (guess > secret_number) {
    try stdout.print("Too big!\n", .{});
} else {
    try stdout.print("You win!\n", .{});
}
```

### Step 4: Looping Until Correct

Finally, wrap the guessing logic in a loop:

```zig
while (true) {
    // ... guessing code here ...
    
    if (guess == secret_number) {
        try stdout.print("You win!\n", .{});
        break;  // Exit the loop
    }
}
```

## Try It Out!

Build and run your guessing game:

```bash
zig build run
```

Play the game a few times. Try entering invalid input to see how error handling works!

## What We Learned

In this chapter, we covered:

- ✓ Reading user input with `stdin.readUntilDelimiter`
- ✓ Random number generation with `std.Random`
- ✓ String parsing with `std.fmt.parseInt`
- ✓ Error handling with `catch`
- ✓ Control flow with `if`/`else` and `while` loops
- ✓ The `break` statement to exit loops

## Next Steps

Now that you've built a complete program, you're ready to learn the fundamental concepts of Zig in [Chapter 3: Common Programming Concepts](../ch03/index.md).
