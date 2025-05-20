# 3.7 Operators

In this chapter, we will explore operators and expressions in the Leo programming language. Leo offers a long list of operators for performing arithmetic operations, logical comparisons, bitwise manipulations, and cryptographic functions.

### Basic Concepts

Operators in Leo compute a value based on one or more expressions (operands). Leo defaults to **checked arithmetic**. This means operations like addition, subtraction, and multiplication will generate an error at runtime if an overflow, underflow, or division by zero is detected during execution (specifically, during proof generation or finalize execution).

Let's see a simple example:

```rust
// Assume inside a transition or function
let mut a: u8 = 1u8 + 1u8; // Checked addition. a is 2u8. Needs 'mut'.
// a is equal to 2

a += 1u8; // Checked compound assignment. a is now 3u8.
// a is now equal to 3

// Using explicit method syntax (also checked)
a = a.add(1u8); // a becomes 4u8.
// a is now equal to 4
```
*(Self-correction: Added `mut` to `a` as it's reassigned.)*

This example demonstrates three different ways to perform addition in Leo:
- Using the `+` operator
- Using the compound assignment operator `+=`
- Using the `.add()` method

All these standard forms perform checked arithmetic. Unchecked (wrapping) versions are available via specific methods like `.add_wrapped()`.

### Operator Precedence

When an expression contains multiple operators, Leo follows a strict order of evaluation based on operator precedence. Here is the precedence table, from highest (evaluated first) to lowest:

| Precedence | Operator         | Description                       | Associativity  |
|------------|------------------|-----------------------------------|----------------|
| 1          | `()` `[]` `.`    | Parentheses, Indexing, Member Access | Left-to-right  |
| 2          | `!` `-(unary)`   | Logical NOT, Unary Minus          | Right-to-left  |
| 3          | `**`             | Exponentiation                    | Right-to-left  |
| 4          | `*` `/` `%`      | Multiplication, Division, Remainder | Left-to-right  |
| 5          | `+` `-(binary)`  | Addition, Subtraction             | Left-to-right  |
| 6          | `<<` `>>`        | Bitwise Shifts                    | Left-to-right  |
| 7          | `&`              | Bitwise AND                       | Left-to-right  |
| 8          | `^`              | Bitwise XOR                       | Left-to-right  |
| 9          | `|`              | Bitwise OR                        | Left-to-right  |
| 10         | `==` `!=` `<` `>` `<=` `>=` | Comparison Operators             | Left-to-right  |
| 11         | `&&`             | Logical AND                       | Left-to-right  |
| 12         | `||`             | Logical OR                        | Left-to-right  |
| 13         | `? :`            | Ternary Conditional               | Right-to-left  |
| 14         | `=` `+=` `-=` `*=` `/=` `%=` `**=` `<<=` `>>=` `&=` `|=` `^=` | Assignment Operators            | Right-to-left  |

#### Using Parentheses for Explicit Ordering

To override the default precedence, you can use parentheses `()` to explicitly control the order of evaluation:

```rust
// Assume 'a' is declared, e.g., let a: u8 = 5u8;
let a: u8 = 5u8;
let result: u8 = (a + 1u8) * 2u8; // result = (5 + 1) * 2 = 6 * 2 = 12u8
```

In this example, `(a + 1u8)` will be evaluated first, then the result will be multiplied by `2u8`.

### Context-dependent Expressions

Leo supports special expressions that provide information about the Aleo blockchain and the current transaction execution context.

#### self.caller

The `self.caller` expression returns the address of the account or program that directly invoked the current transition:

```rust
program caller_check.aleo {
    // Checks if the direct caller matches the provided address
    transition matches(addr: address) -> bool {
        return self.caller == addr;
    }
}
```
If transition `A` calls transition `B`, then inside `B`, `self.caller` will be the address of `A`.

#### self.signer

The `self.signer` expression returns the address of the account that initiated the top-level transaction - the account that signed the transaction:

```rust
program signer_check.aleo {
    // Checks if the original transaction signer matches the provided address
    transition matches(addr: address) -> bool {
        return self.signer == addr;
    }
}
```
If user account `U` signs a transaction that calls transition `A`, which then calls transition `B`, inside both `A` and `B`, `self.signer` will be the address of `U`.

#### block.height

The `block.height` expression returns the current block height. Note that this can **only** be used within a `finalize` block, as the block height is determined during on-chain execution by validators.

```rust
program block_check.aleo {
    mapping storage: u32 => u32; // Example mapping

    // Finalize block uses block.height
    finalize check_block_height(public height_key: u32) {
        let current_height: u32 = block.height;
        // Example: Store the current block height in the mapping
        Mapping::set(storage, height_key, current_height);
        // Example: Assert based on block height
        assert(current_height > 100u32); // Fails if block height is <= 100
    }

    transition trigger_finalize(key: u32) {
        // Off-chain logic...
        // Call finalize, passing the key
        finalize(key);
    }
}
```
*(Self-correction: Corrected the usage context of `block.height` to be within `finalize`.)*

### Core Functions (Often used like operators)

Leo provides several built-in functions for assertions and cryptographic operations that act like core language features.

#### Assertions

The `assert(condition)`, `assert_eq(left, right)`, and `assert_neq(left, right)` functions check conditions and halt program execution (aborting the transaction) if they evaluate to false:

```rust
program assertion_test.aleo {
    transition matches(a: u8, b: u8) {
        assert(a < 10u8);        // Continues if a is less than 10
        assert_eq(a, b);         // Continues if a equals b
        assert_neq(a, 0u8);      // Continues if a is not zero
        // assert(false);        // Would always halt execution
        // assert_eq(1u8, 2u8);  // Would always halt execution
    }
}
```

#### Hash Functions

Leo supports several hash algorithms, each with different input constraints and output types. They are accessed via `Algorithm::hash_to_{type}` methods:

```rust
// Example hashes (assuming inputs are valid for the chosen functions)
let input_field: field = 1field;
let input_bool: bool = true;
let input_u64: u64 = 123u64;

// Output type depends on the function used
let scalar_hash: scalar = BHP256::hash_to_scalar(input_bool);
let field_hash: field = Poseidon2::hash_to_field(input_field, input_field); // Poseidon2 takes 2 field inputs
let address_hash: address = Pedersen64::hash_to_address(input_u64); // Pedersen64 takes u64 input
let group_hash: group = Poseidon4::hash_to_group(input_field, input_field, input_field, input_field); // Poseidon4 takes 4 field inputs
```

Available hash algorithms include (check Aleo documentation for current list and exact function signatures):
- `BHP256`, `BHP512`, `BHP768`, `BHP1024`
- `Pedersen64`, `Pedersen128`
- `Poseidon2`, `Poseidon4`, `Poseidon8` (number indicates number of field inputs)
- `Keccak256`, `Keccak384`, `Keccak512`
- `SHA3-256`, `SHA3-384`, `SHA3-512`

#### Commitment Functions

Commitment schemes allow you to commit to a value while keeping it hidden, potentially revealing it later. They typically involve hashing the value with a secret salt (randomness).

```rust
// Generate a random scalar for the salt (typically done off-chain or provided as private input)
// let salt: scalar = ...;
let value_to_commit: u64 = 1000u64;
let salt: scalar = 123456789scalar; // Example salt

// Commit using a hash function
// The exact function depends on the desired output type and security properties
let commitment: field = BHP512::commit_to_field(value_to_commit, salt);
let group_commitment: group = Pedersen128::commit_to_group(value_to_commit, salt);
// let address_commitment: address = Pedersen64::commit_to_address(some_u64_value, salt);
```

Available commitment functions often mirror the hash functions: `Algorithm::commit_to_{type}`.

#### Random Number Generation

Leo currently does **not** provide a cryptographically secure pseudo-random number generator (CSPRNG) directly within transitions or finalize blocks due to the deterministic nature required for ZK circuits and consensus. Randomness (like salts or nonces) must typically be generated off-chain and provided as private inputs.

*(Self-correction: Removed mention of ChaCha as a directly usable RNG within Leo execution contexts. Randomness must come from external sources.)*

### Standard Operators Detailed

Let's explore the standard operators available in Leo in more detail. Remember that standard arithmetic operators (`+`, `-`, `*`, `/`, `**`) and their compound assignment versions perform **checked** arithmetic by default.

#### Arithmetic Operators

**Addition (`+`, `+=`, `add`, `add_wrapped`)**

```rust
let a: u8 = 1u8;
let b: u8 = 2u8;

// Checked addition
let sum1: u8 = a + b;       // sum1 = 3u8
let mut sum2: u8 = a;
sum2 += b;                  // sum2 becomes 3u8
let sum3: u8 = a.add(b);    // sum3 = 3u8

// Unchecked (wrapping) addition
let max_u8: u8 = 255u8;
let wrapped_sum: u8 = max_u8.add_wrapped(1u8);  // wrapped_sum = 0u8
```

**Subtraction (`-`, `-=`, `sub`, `sub_wrapped`)**

```rust
let a: u8 = 5u8;
let b: u8 = 2u8;

// Checked subtraction
let diff1: u8 = a - b;      // diff1 = 3u8
let mut diff2: u8 = a;
diff2 -= b;                 // diff2 becomes 3u8
let diff3: u8 = a.sub(b);   // diff3 = 3u8

// Unchecked (wrapping) subtraction
let zero_u8: u8 = 0u8;
let wrapped_diff: u8 = zero_u8.sub_wrapped(1u8);  // wrapped_diff = 255u8
```

**Multiplication (`*`, `*=`, `mul`, `mul_wrapped`)**

```rust
let a: u8 = 3u8;
let b: u8 = 4u8;

// Checked multiplication
let prod1: u8 = a * b;      // prod1 = 12u8
let mut prod2: u8 = a;
prod2 *= b;                 // prod2 becomes 12u8
let prod3: u8 = a.mul(b);   // prod3 = 12u8

// Unchecked (wrapping) multiplication
let high_u8: u8 = 128u8;
let wrapped_prod: u8 = high_u8.mul_wrapped(2u8);  // wrapped_prod = 0u8
```

**Division (`/`, `/=`, `div`, `div_wrapped`)**
*Note: Division is integer division (truncates towards zero).*

```rust
let a: u8 = 9u8;
let b: u8 = 2u8;

// Checked division (panics on division by zero or iN::MIN / -1 for signed types)
let quot1: u8 = a / b;      // quot1 = 4u8
let mut quot2: u8 = a;
quot2 /= b;                 // quot2 becomes 4u8
let quot3: u8 = a.div(b);   // quot3 = 4u8

// Unchecked (wrapping) division
// Primarily relevant for signed integers: iN::MIN.div_wrapped(-1) returns iN::MIN
// For unsigned, behavior is same as checked, except maybe division by zero (still likely panics)
let min_i8: i8 = -128i8;
// let wrapped_quot: i8 = min_i8.div_wrapped(-1i8); // wrapped_quot = -128i8 (avoids panic)
```

**Remainder (`%`, `%=`, `rem`)**
*Note: `rem` follows the sign of the dividend.*

```rust
let a: u8 = 9u8;
let b: u8 = 2u8;
let neg_a: i8 = -9i8;
let neg_b: i8 = 2i8;

// Remainder
let rem1: u8 = a % b;       // rem1 = 1u8
let mut rem2: u8 = a;
rem2 %= b;                  // rem2 becomes 1u8
let rem3: u8 = a.rem(b);    // rem3 = 1u8
let rem4: i8 = neg_a % neg_b;// rem4 = -1i8 (sign matches dividend -9)
let rem5: i8 = neg_a.rem(neg_b); // rem5 = -1i8
```

**Exponentiation (`**`, `pow`, `pow_wrapped`)**
*Note: Exponent must be an unsigned integer type (e.g., u8, u16, u32).*

```rust
let base: u8 = 2u8;
let exponent: u32 = 3u32; // Exponent type often u32

// Checked exponentiation
// Operator `**` might require specific setup or method call is preferred
// let pow1: u8 = base ** exponent; // Check Leo docs for current operator support
let pow2: u8 = base.pow(exponent); // pow2 = 8u8

// Unchecked (wrapping) exponentiation
let base_wrap: u8 = 16u8;
let exp_wrap: u32 = 2u32;
let wrapped_pow: u8 = base_wrap.pow_wrapped(exp_wrap); // wrapped_pow = 0u8 (16^2 = 256 wraps to 0 in u8)
```

#### Comparison Operators (`==`, `!=`, `<`, `>`, `<=`, `>=`)

These operators compare two operands and return a boolean (`true` or `false`). They work on integers, fields, scalars, addresses, booleans.

```rust
let a: u8 = 5u8;
let b: u8 = 10u8;

let eq: bool = a == b;      // false
let neq: bool = a != b;     // true
let lt: bool = a < b;       // true
let lte: bool = a <= b;     // true
let gt: bool = a > b;       // false
let gte: bool = a >= b;     // false
```

Method syntax is also available for comparisons, though less common: `a.eq(b)`, `a.neq(b)`, `a.lt(b)`, etc.

#### Logical Operators (`&&`, `||`, `!`)

These operate on boolean values. `&&` and `||` use short-circuiting evaluation.

```rust
let a: bool = true;
let b: bool = false;

let and_op: bool = a && b;     // false
let or_op: bool = a || b;      // true
let not_op: bool = !a;         // false
```

Boolean types also have methods like `a.and(b)`, `a.or(b)`, `a.xor(b)`, `a.not()`.

#### Bitwise Operators (`&`, `|`, `^`, `!`, `<<`, `>>`)

These operate on the binary representation of integer types. `!` performs bitwise NOT (inverting all bits).

```rust
let a: u8 = 0b1100_u8;  // 12 decimal
let b: u8 = 0b1010_u8;  // 10 decimal

let bit_and: u8 = a & b;   // 0b1000 (8 decimal)
let bit_or: u8 = a | b;    // 0b1110 (14 decimal)
let bit_xor: u8 = a ^ b;   // 0b0110 (6 decimal)
let bit_not: u8 = !a;      // 0b1111_0011 (243 decimal - depends on interpretation, usually requires casting)
                          // Note: Bitwise NOT on unsigned types can be tricky. Often used with masks.

// Shifts (Right operand is the number of bits to shift)
let shl: u8 = a << 1u32;   // 0b11000 (24 decimal) - Shift left by 1 (use u32 for shift amount)
let shr: u8 = a >> 1u32;   // 0b0110 (6 decimal) - Shift right by 1
```
*(Self-correction: Specified `u32` for shift amount, which is common practice. Clarified bitwise NOT behavior.)*

Checked and wrapping method syntax is available: `a.and(b)`, `a.shl(1u32)`, `a.shl_wrapped(1u32)`, etc.

#### Ternary Operator (`? :`)

The ternary conditional operator provides a concise way to express a simple `if-else` assignment.

```rust
let condition: bool = true;
let x: u8 = 5u8;
let y: u8 = 10u8;

// If condition is true, result is x, otherwise result is y.
let result: u8 = condition ? x : y;  // result = 5u8
```

### Type-Specific Operators and Methods

Some operations are specific to certain types:

#### Field Type

- Supports arithmetic: `+`, `-`, `*`, `/` (field division), `**` (exponentiation by scalar or integer)
- Methods: `.square()`, `.sqrt()` (square root), `.inverse()` (multiplicative inverse), `.double()`

#### Group Type

- Supports elliptic curve point addition: `+`
- Supports point subtraction (add the negation): `p - q` is `p + (-q)`
- Supports scalar multiplication: `scalar * group` or `group * scalar`
- Methods: `.double()` (point doubling), `-p` or `p.neg()` (point negation)
- Constant: `group::GEN` (the standard generator point `G`)

#### Signature Type

- Method: `signature.verify(address, message: field) -> bool` (verifies if the signature is valid for the message under the address's public key)

#### Boolean Type

- Methods: `.nand(b)`, `.nor(b)`, `.xor(b)` in addition to standard logical ops.

### Best Practices for Operators

1.  **Default to Checked Arithmetic:** Rely on Leo's default checked operations (`+`, `-`, `*`, `/`, `**`, assignments) for safety against overflows/underflows unless wrapping behavior is explicitly desired and understood.
2.  **Use Wrapping Methods Explicitly:** If wrapping is needed (e.g., in specific cryptographic contexts or hash functions), use the `.add_wrapped()`, `.sub_wrapped()`, etc., methods and document why.
3.  **Type Consistency:** Ensure operands in an expression have compatible types. Use `as` for explicit casting when necessary and safe.
4.  **Parentheses for Clarity:** Use parentheses `()` generously in complex expressions to make the order of operations unambiguous, even if it matches default precedence.
5.  **Beware Integer Division/Remainder:** Remember that `/` performs truncating integer division, and `%` gives the remainder whose sign matches the dividend.
6.  **Cryptographic Operations:** Use hash functions (`BHP::hash...`, `Poseidon::hash...`, etc.) and commitment functions (`BHP::commit...`) appropriately according to the security requirements of your application.
7.  **`finalize` Context:** Remember that `block.height` and mapping operations (`Mapping::get`, `Mapping::set`, etc.) are only available within `finalize` blocks.

### Common Patterns and Examples

#### Safe Increment with Overflow Check (Default)

```rust
// Inside a transition or function
function safe_increment(counter: u32) -> u32 {
    // This will cause the execution to fail if 'counter' is u32::MAX
    return counter + 1u32;
}
```

#### Wrapping Increment (Explicit)

```rust
function wrapping_increment(counter: u32) -> u32 {
    // Explicitly uses wrapping addition
    return counter.add_wrapped(1u32); // Wraps from u32::MAX back to 0
}
```

#### Calculating Average (Avoiding Intermediate Overflow)

```rust
// Calculate the average of two u32 numbers safely
function average(a: u32, b: u32) -> u32 {
    // Calculate average as (a/2) + (b/2) + ((a%2 + b%2) / 2)
    // This avoids potential overflow if a + b exceeds u32::MAX
    return a / 2u32 + b / 2u32 + (a % 2u32 + b % 2u32) / 2u32;
}
```

#### Simple Access Control Check

```rust
// Check if the transaction signer is either the owner or an admin
function check_permission(owner: address, admin: address) -> bool {
    return self.signer == owner || self.signer == admin;
}
```

#### Basic Hash-Based Commitment

```rust
// Commit to a u64 value using BHP256 hash function
function create_commitment(value: u64, salt: scalar) -> field {
    // Hash the tuple (value, salt) to a field element
    // Note: The exact hashing method might vary; ensure inputs match function signature.
    // Example assumes a compatible BHP variant exists or requires casting/serialization.
    // Placeholder: Actual implementation might need type casting or specific hash variants.
    // return BHP256::hash_to_field(value as field, salt as field); // Example if fields needed
    // A more direct hash might exist:
    return BHP256::hash_to_field((value, salt)); // Assuming direct tuple hashing is supported
}
```
*(Self-correction: Acknowledged that direct hashing of tuples or mixed types depends heavily on the specific hash function variant available in Leo's standard library. Casting or serialization might be required.)*

This comprehensive overview covers the essential operators and expressions in Leo, providing a foundation for building more complex logic within Aleo programs. Always refer to the official Leo documentation for the most current syntax and available functions. 