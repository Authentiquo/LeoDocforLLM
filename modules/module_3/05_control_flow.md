# 3.5 Control Flow

## Conditions

Conditional statements allow you to execute different blocks of code based on specific conditions.

The following example demonstrates an `if` statement that checks the value of a variable `a` and updates it accordingly:

```rust
// Assume 'a' is declared mutable beforehand, e.g., let mut a: u8 = 1u8;
let mut a: u8 = 1u8; // Variable needs to be mutable to be reassigned

if a == 1u8 { // Parentheses around condition are optional in modern Leo/Rust syntax
    a += 1u8;
} else if a == 2u8 {
    a += 2u8;
} else {
    a += 3u8;
}
// After this block, 'a' will be 2u8 because the first condition was true.
```
*(Self-correction: Added `mut` for variable `a`, removed unnecessary parentheses around conditions as per modern Leo/Rust style.)*

In this case:
- If `a` is `1`, it increments by `1`.
- If `a` is `2`, it increments by `2`.
- Otherwise, it increments by `3`.

## Assert

Assertions are special statements that allow halting the execution of a transition under a specific condition.

They can be used simply as follows:

```rust
let condition: bool = false;
assert(condition); // This will always fail because the condition is false
```

This code will always fail when the condition is false, and execute correctly when the condition is true.

Here is a more useful example:

```rust
program assert_example.aleo {
    // Only the specified address can successfully execute this transition.
    transition protected() {
        // Ensure the transaction signer is the allowed address.
        assert(self.signer == aleo1hn9z4zt6awd6ts9zn8ppkwwvjwrg04ax48krd0fgnnzu6ezg9qxst3v26j);
        // Do something that only the address above should be allowed to do...
    }
}
```

This transition can never be successfully called by anyone other than `aleo1hn9z4zt6awd6ts9zn8ppkwwvjwrg04ax48krd0fgnnzu6ezg9qxst3v26j`. Execution will halt if the `assert` condition is false.

`assert` statements (and `assert_eq`, `assert_neq`) are extremely useful, especially for validating user inputs and enforcing access control.

## Loops

A `for` loop is used to iterate over a range of values. The loop executes a specific number of times, making it ideal for iterating over sequences or performing repeated operations.

```rust
// General syntax
for variable: type in lower_bound..upper_bound {
    // Loop body
}
```

- The loop variable is declared with a type.
- The loop iterates from `lower_bound` up to (but not including) `upper_bound`.
- The bounds must be compile-time constants of the same integer type.
- The `lower_bound` must always be less than or equal to the `upper_bound`. If `lower_bound == upper_bound`, the loop does not execute.

This example demonstrates a simple `for` loop that counts from `0` to `4` and keeps track of the count:

```rust
// Assume this code is within a transition or function
let mut count: u32 = 0u32; // Needs to be mutable

for i: u32 in 0u32..5u32 {
    count += 1u32;
}

// return count; // If inside a function/transition that returns u32
// At this point, count will be 5u32
```
*(Self-correction: Added `mut` for `count`)*

- The loop executes for `i` values `0, 1, 2, 3, 4`, iterating `5` times.
- `count` is incremented during each iteration.
- If returned, the function would return `5u32`.

In Leo, loops must always have a finite number of iterations that are known **at compile time**. For example, the upper or lower bound of the loop cannot be an input to the transition directly controlling the loop range.

So while this code is invalid:

```rust
// INVALID: Loop bounds cannot be runtime variables like transition inputs
program invalid_loop.aleo {
    transition variable_loop(iterations: u32) -> u32 {
        let mut output: u32 = 0u32;
        // ERROR: 'iterations' is not a compile-time constant
        for i: u32 in 0u32..iterations {
            output += 1u32;
        }
        return output;
    }
}
```

Something like this could be used instead, by looping up to a fixed maximum and conditionally performing the work inside:

```rust
program fixed_loop.aleo {
    // Define a compile-time constant for the maximum loop iterations
    const MAX_ITERATIONS: u32 = 100u32;

    transition variable_loop(iterations: u32) -> u32 {
        // Ensure the requested number of iterations does not exceed the maximum
        assert(iterations <= MAX_ITERATIONS);

        let mut output: u32 = 0u32; // Needs to be mutable
        // Loop up to the *compile-time constant* maximum
        for i: u32 in 0u32..MAX_ITERATIONS {
            // Conditionally execute the loop body based on the runtime input
            if i < iterations {
                output += 1u32;
            }
        }
        return output; // Returns the value of 'iterations' (capped by MAX_ITERATIONS)
    }
}
```
*(Self-correction: Added `mut` for `output`)*

This pattern allows handling variable iteration counts up to a predefined limit, satisfying the compile-time bounds requirement for loops. 