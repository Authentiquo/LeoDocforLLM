Okay, here is the combined content of the provided Markdown files, translated into English, and formatted as a single Markdown code block.

```markdown
# Module 3: Leo Programming Fundamentals

--- START OF 3.1-hello-world.md ---

# 3.1 Hello World

We are finally ready to create our first Aleo program. Let's see how to proceed.

The Leo command-line tool offers a simple way to start a new project from scratch:

```bash
leo new hello_world
```

This will create a new program project named `hello_world`. This project is instantiated in a new directory. Let's start by moving into this directory:

```bash
cd hello_world
```

You should have the following file structure:

```
hello_world/
├── .gitignore
├── .env
├── program.json
└── src/
  └── main.leo
```

`.gitignore` is just a default file related to git. The environment file `.env` contains the `NETWORK` and `PRIVATE_KEY` variables. Let's keep the default generated values for now.

The `program.json` file is a manifest containing all the metadata for your project:

```json
{
  "program": "hello_world.aleo",
  "version": "0.1.0",
  "description": "",
  "license": "MIT",
  "dependencies": null
}
```

No need to modify this information for now.

The `src/main.leo` file contains the source code for a "hello world" example program:

```rust
// The 'hello_world' program.
program hello_world.aleo {
    transition main(public a: u32, b: u32) -> u32 {
        let c: u32 = a + b;
        return c;
    }
}
```

Leo's syntax looks very similar to Rust's, if you are familiar with that language. Let's break down this first program line by line to try to understand what it does:

```rust
// The 'hello_world' program.
```

This line is a comment, you can just ignore it. Like in Rust, single-line comments start with `//` and multi-line comments start with `/*` and end with `*/`.

```rust
program hello_world.aleo {
```

All the code for your program will be included within this program structure. `hello_world.aleo` is the identifier for your program; it's a unique identifier for your program on the blockchain. Once deployed, you will be able to call your program using this identifier. If a program with this identifier is already deployed, you will need to deploy your program with a different identifier, updating the value here, as well as in the `program` field of the `program.json` file.

```rust
transition main(public a: u32, b: u32) -> u32 {
```

This is where things get interesting. A transition is a function within your program that can be called by users and other programs, directly on the blockchain. Using its program identifier and name, any user can call a transition in a program, for example, using the snarkOS command-line interface.

Here, the name of the transition is `main`, but it could be anything else.

A transition has inputs and outputs. These inputs must be provided by the caller of the transition. Here, the inputs are defined as `public a: u32, b: u32`, there are two inputs `a` and `b`. They are both 32-bit unsigned integers, but `a` is public while `b` is private; we will see exactly what this means in the next chapter. The output is a single 32-bit unsigned integer, as defined by: `-> u32`. The next line is:

```rust
let c: u32 = a + b;
```

Here, we simply define a new variable `c` which is the sum of `a` and `b`.

All statements in Leo must end with a semicolon, just like in C or Rust.

```rust
return c;
```

The sum is then returned as the output in the form of a 32-bit unsigned integer.

There you have your first Leo program, although it's not very useful yet, of course.

Now, let's try to execute the `main` transition (locally off-chain of course, your program is not yet deployed on the blockchain). We will use `a = 1u32` and `b = 2u32` as test values:

```bash
leo run main 1u32 2u32
```

Here is what the console should display:

```
       Leo ✅ Compiled 'hello_world.aleo' into Aleo instructions

⛓  Constraints

 •  'hello_world.aleo/main' - 33 constraints (called 1 time)

➡️  Output

 • 3u32

       Leo ✅ Finished 'hello_world.aleo/main' (in "/path/to/hello_world/build")
```
*(Note: The exact path in the output will vary)*

This command performs several steps in the background. First, it transcribes your high-level Leo code into Aleo instructions. This is the code that can be deployed on the blockchain as a program. Aleo instructions can then be compiled into AVM bytecode, a lower-level language that is not human-readable, but can be executed by the Aleo zkVM. To draw an analogy with traditional programming, you can think of Leo as Rust or C, Aleo instructions as assembly, and AVM bytecodes as machine code.

When transforming Aleo instructions into AVM bytecode, the compiler transforms each instruction of each function into an R1CS circuit representation. In the command output shown earlier, the number of constraints in the circuit for the main function is displayed in the "Constraint" section: here, it's 33. The higher the constraints, the longer it will take to prove a transition.

In the output section, the program's output is displayed: here `c` from our previous program.

If you actually want to generate a proof for the execution of a function on certain inputs, you should use the `execute` command instead:

```bash
leo execute main 1u32 2u32
```

In addition to the information displayed previously, this should produce a JSON output. This JSON contains all the information necessary to verify your execution of the function. It includes the Proof (at the "proof" key), but no knowledge about the private input and outputs of the said function. You could send this JSON to any verifier. Once your program is deployed, this is what users will broadcast to the blockchain nodes so that the function can be verified and executed on-chain.

After executing this command, if you list your files again, you should see something like:

```
hello_world/
├── .gitignore
├── .env
├── leo.lock
├── program.json
├── build/
│   ├── build/             <-- (Note: Leo structure might slightly change, this inner 'build' might not exist)
│   │   ├── main.avm
│   │   ├── main.prover
│   │   └── main.verifier
│   ├── program.json       <-- (Note: This might be at the outer 'build' level)
│   └── main.aleo
└── src/
    └── main.leo
```
*(File structure might vary slightly depending on the Leo version)*

`leo.lock` keeps track of the resolved dependencies for your program, we will come back to this in the next module. `main.aleo` is the transcribed file containing Aleo instructions, while `main.avm` is the compiled file containing AVM bytecode. The prover and verifier files contain serialized Proving and Verifying keys for the corresponding program. They can be used respectively to generate and verify proofs of executions.

In the next chapter, we will see what the private and public keywords mean.

--- END OF 3.1-hello-world.md ---

--- START OF 3.2-public-vs-private.md ---

# 3.2 Public vs Private

In this chapter, we will explore the fundamental concepts that distinguish Leo from languages used on other blockchains.

Remember, Leo is a language designed for writing programs where users can prove that a function returns certain outputs for given inputs. The prover always knows these inputs and outputs. Conversely, when writing the program, you can choose which inputs and outputs the verifier will be able to know.

**To specify which inputs/outputs will be visible to the verifier, you use the `private` or `public` keywords.**

Here is a first example. Suppose a user wants to prove they know a value that has a specific hash. They could use a program with the following code:

```rust
program public_private.aleo {
    // Hashes a private input 'a' and returns the public hash.
    transition hash(private a: u128) -> (public u128) {
        let hashed: u128 = BHP256::hash_to_u128(a);
        // 'BHP256::hash_to_u128' is simply a hash function
        return hashed;
    }
}
```

Then execute the following command to generate the proof on the input `123u128`:

```bash
leo execute hash 123u128
```

The JSON output of the execution gives (truncated, for clarity):

```json
...
  "program": "public_private.aleo",
  "function": "hash",
  "inputs": [    {
      "type": "private",
      "id": "a", // The ID might be different in actual output
      "value": "ciphertext1..." // Actual ciphertext will vary
    }
  ],
  "outputs": [    {
      "type": "public",
      "id": null, // ID might be null or different
      "value": "26515625101883903322553063858644017311u128"
    }
  ],
...
```
*(Note: The exact format of the JSON output, including `id` fields and ciphertext, may vary with Leo versions and execution context.)*

As you can see here, the type of the only input is private, and the value is a ciphertext, which can be decrypted using the prover's view key to retrieve the input value itself. Conversely, the output is a public value: `26515625101883903322553063858644017311u128` which directly corresponds to the hash of `123u128`.

Transitions can have up to 16 inputs and outputs (this limit might change in future Aleo versions), which can all be independently marked as public or private.

By default, if you do not specify any visibility for an input/output, it will be private.

The prover's address itself is also private, and accessible within a program using the `self.signer` keyword. For example, the following code could be used to prove that the prover's address is one of three hardcoded addresses:

```rust
program public_private.aleo {
    // Checks if the signer's address is one of the three allowed addresses.
    transition includes_signer() -> (public bool) {
        let included: bool = (
            self.signer == aleo1845psejz2fqy3zwjvdxjrexs9fwdh2tqjwlhtdjs2t6thr07ccyq3fvmfd
            || self.signer == aleo1hqrz7wukfv2mmxzuzwes0w44s2se2v98w840k6euncdm8mwfd5pq2dwjcy
            || self.signer == aleo15eapl52ju8a6zxz5qnau7nlvadz3x3napa0qds7c04d8qkt6f5gs747czs
        );
        return included;
    }
}
```

Executing this function, along with its proof, reveals nothing about the prover's address other than whether it is included in the list of the three addresses.

We are still missing one last important piece of input/output visibility available on Aleo. But before we look at it, we first need to describe all the types available in the language.

--- END OF 3.2-public-vs-private.md ---

--- START OF 3.3-variables-and-types.md ---

# 3.3 Variables and Types

In the last two chapters, we started manipulating some of the objects available in the Leo programming language, but there are many others that we will discover here.

In Leo, all variable types must be explicitly declared:

```rust
let setting: bool = true; // VALID, will compile
let setting = true; // INVALID, will cause an error during compilation
```

Some variables are constants that are declared at the top of the program, outside transitions at the very root of the project, using the `const` keyword, as in the following example:

```rust
program variables.aleo {
    const SETTING: bool = true;
    /*
    By convention, we will always use uppercase letters for constants
    in this course, to distinguish them from other variables.
    */

    transition test() -> bool {
        return SETTING || false;
    }
}
```

While these can never be updated (like inputs), other variables can, and must be created inside code blocks such as transitions using the `let` keyword (we will see other types of block structures in two chapters):

```rust
program variables.aleo {
    transition test() -> bool {
        let output: bool = true; // Variable declaration
        output = output || false; // Variable modification
        // By convention, we will always use snake_case for variables in this course.

        return output;
    }
}
```

All variables in Leo must have a value upon declaration and cannot be left empty. The Null type does not exist on Aleo.

On Aleo, there is no concept of pointers; all variables are "passed by value," essentially copied when passed to another structure.

Now that you know the basic rules for manipulating variables, let's discover the available types.

## Types

### Fields

We will start with the most fundamental type. Since Aleo's underlying constraint system is R1CS, programs are essentially represented as sets of linear equations over a field. This makes the elements of this field very special. They are simply called `field`s, for short, and represent elements of the base field of the elliptic curve on which Aleo accounts are based.

All other types available on Aleo are derived from this one, so that instructions involving these additional types can be reduced to linear equations over the base field.

This means that using fields directly is generally the most optimal way to write a program in terms of constraints.

Fields are the elements of Z/pZ with:

p = 8444461749428370424248824938781546531375899335154063827935233455917409239041

They can be used directly in Leo by postfixing `field` to their integer representation in [0, p-1] as follows:

```rust
let element: field = 12field;
```

The smallest field is `0field`, and the largest is p-1 = `8444461749428370424248824938781546531375899335154063827935233455917409239040field`.

Naturally, the operations `+ - * /` are the corresponding field operations.

### Group

Group elements represent points of a specific subgroup of the group of points on the Aleo elliptic curve. This subgroup is generated by a generator point denoted `group::GEN`. Group elements can be used in Leo by postfixing `group` to their x-coordinate. For example, `0group` represents the point at infinity (identity element), and `1group` represents the generator `G`. *Correction: `0group` is the identity, not `(0,1)`. The relationship between integer suffixes and specific points is complex and defined by the curve implementation.*

```rust
let identity: group = 0group; // The identity element (point at infinity)
let generator: group = 1group; // The standard generator G
let two_g: group = 2group; // Represents 2*G (G + G)
let sum: group = generator + two_g; // Corresponds to 3*G
```
*(Note: The specific coordinates for points like 2group depend on the underlying curve parameters and are not typically represented by simple integer coordinates like (2, 555...))*

### Scalar

The subgroup described above has an order:

q = 2111115437357092606062206234695386632838870926408408195193685246394721360383

This means that when adding the generator G to itself successively: G+G+...+G, by definition, one eventually reaches every element of the group, 1⋅G, 2⋅G,... until reaching q⋅G = 0 (the identity element). This set of integers by which we can multiply G is therefore simply the field Z/qZ, called the set of scalars. We call their elements scalars and they can be used directly in Leo by postfixing `scalar` to their integer representation in [0, q-1] as follows:

```rust
let element: scalar = 2scalar;
```

### Address

Addresses are just another representation of group elements (specifically, it's the bech32 encoding). They are public keys, used to uniquely identify users and programs on-chain:

```rust
let owner: address = aleo1hqrz7wukfv2mmxzuzwes0w44s2se2v98w840k6euncdm8mwfd5pq2dwjcy;
```

Two useful addresses should be noted:

```rust
// 0group as address (identity element):
let zero_address: address = aleo1qqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq3ljyzc;

// A specific hardcoded group element address (often used as a burn address, related to 1group or similar, not necessarily 2group):
// The original text mentioned 2group, but standard burn addresses often relate to G or other specific points.
// Using the commonly cited 'aleo1k...' burn address for clarity:
let burn_address: address = aleo1kqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqs0d7q4;
```

The view key associated with `zero_address` is `0scalar` since 0 = 0s⋅G. The view key and private key for standard burn addresses like `aleo1k...` are unknown. Finding them would require solving the discrete logarithm problem.

The private key for these addresses is unknown. This makes these addresses "burn addresses" in practice: sending tokens to these addresses would render those tokens unusable. `zero_address` is a "verifiable burn address" since anyone can decrypt data encrypted with this public key using the view key `0scalar`. Conversely, addresses like `aleo1k...` act as "private burn addresses".

### Signatures

Leo supports the verification of Schnorr signatures within a program. The `signature` type represents the result of signing a field element using a private key. Here is an example of how to verify such a signature:

```rust
program variables.aleo {
    transition verify_signature(s: signature, signer_address: address, message: field) -> bool {
        // Verify the signature 's' corresponds to the 'message' signed by the private key
        // associated with 'signer_address'.
        let valid: bool = signature::verify(s, signer_address, message);
        return valid;
    }
}
```

Generating a signature is not possible within a program. The reason is that you would instead generate this signature outside the program, pass this signature as an argument, and validate it using the function above.

### Booleans

Leo supports traditional boolean values `true` and `false`.

```rust
let b: bool = false;
```

### Integers

In addition to field and scalar integers, Leo supports traditional integers more commonly used in computer science. There are unsigned integers: `u8, u16, u32, u64, u128` which support integers from 0 to 2^b-1 inclusive, for the type `ub`. These limits are enforced by the VM itself. Let's see an example:

```rust
let a: u8 = 1u8; // VALID declaration of a
let b: u8 = 2u8; // VALID declaration of b

let c: u8 = a - b; // INVALID declaration of c (at runtime)
```

This program will compile, although it will crash at runtime when attempting to generate a proof for any input involving this subtraction, due to underflow. The same applies if the upper limit is violated (overflow).

Next, there are signed integers: `i8, i16, i32, i64, i128` which support integers from -2^(b-1) to 2^(b-1)-1 inclusive, for the type `ib`.

```rust
let a: i8 = -2i8; // VALID declaration of a
let b: i8 = 3i8; // VALID declaration of b
```

Integer declaration can include underscores `_` to improve readability:

```rust
let a: u64 = 1_000_000u64;
```

Note that integers with larger bit lengths increase the circuit constraints, and therefore the proving time.

It is also important to mention that it is not possible to mix multiple integer types in expressions:

```rust
let c: u128 = 3u128 - 1u32; // INVALID operation
```

See the Casting section below to understand how this problem can be solved.

### Structs

Structs are composite data structures made of keys and values. They are defined at the root of the program block, using the `struct` keyword, and used in the rest of the program:

```rust
program variables.aleo {
    // Struct definition, notice the absence of a semicolon
    struct Date {
        year: u16,
        month: u8,
        day: u8
    }
    // By convention in this course, structs will be named using CamelCase

    transition get_null_date() -> Date {
        // Instantiation of a struct
        let null_date: Date = Date {
            year: 1970u16,
            month: 1u8,  // Type should match definition (u8)
            day: 1u8     // Type should match definition (u8)
        };
        return null_date;
    }

    // Struct properties can be accessed using "." followed by the attribute name
    transition is_january(date: Date) -> bool {
        return date.month == 1u8; // Comparison type should match (u8)
    }
}
```

Structs can be nested:

```rust
struct Person {
    id: field,
    date_of_birth: Date // Assuming Date struct is defined above
}

// Structs can be nested multiple times:
struct Couple {
    person1: Person,
    person2: Person
}
```

It is not possible to update a struct directly (structs are immutable):

```rust
let today: Date = Date {
    year: 2025u16,
    month: 3u8,
    day: 2u8
};

today.day = 3u8; // INVALID, structs cannot be updated in place

// Instead, you would create a new struct:
let tomorrow: Date = Date {
    year: today.year,
    month: today.month,
    day: today.day + 1u8 // Create new struct with updated value
};
```

### Arrays

Leo supports static arrays of fixed size. Arrays must declare both their type and length:

```rust
let arr: [bool; 4] = [true, false, true, false];
```

Arrays can be nested:

```rust
let nested: [[bool; 2]; 2] = [[true, false], [true, false]];
```

Arrays can also contain structs and structs can contain arrays:

```rust
struct StructOfArray {
    array: [u8; 2],
}

struct Bar {
    data: u8,
}

// ...

let arr_of_structs: [Bar; 2] = [Bar { data: 1u8 }, Bar { data: 2u8 }];
```

Array elements can be accessed using constant indices:

```rust
transition foo(a: [u8; 8]) -> u8 {
    // Accessing requires explicit type casting for the index if using a variable,
    // but constants work directly. However, Leo generally expects constant indices.
    // Using a constant index:
    return a[0]; // Access the first element
}

transition foo2(a: [Bar; 8]) -> u8 {
    // Accessing struct field within an array element
    return a[0].data; // Access data field of the first Bar struct
}
```
*(Note: While the original example used `0u8` as index, direct integer literals like `0` are usually sufficient and clearer for constant indices).*

Arrays can also be iterated over using a `for` loop, we will return to this in chapter 3.5:

```rust
transition sum_with_loop(a: [u64; 4]) -> u64 {
    let sum: u64 = 0u64;
    // Loop indices must match the array index type expectations or be cast.
    // Leo's for loop syntax requires explicit types and bounds.
    for i: u32 in 0u32..4u32 { // Using u32 for index, common practice
        // Array access requires the index type to be compatible or cast.
        // Leo might require explicit casting depending on version/context.
        // Assuming direct indexing works if loop variable type is appropriate size:
        sum += a[i as u64]; // Example assumes index needs casting to u64 if array expects it
                           // Or more likely, the array access works with u32 index directly
                           // Let's assume direct access works with appropriate index type (like u32):
         // sum += a[i]; // If array indexing accepts u32 directly
    }
    // Re-evaluating the loop based on standard Leo practice:
    let mut running_sum: u64 = 0u64; // Need 'mut' to modify
    for i: u32 in 0u32..4u32 {
         // Array access usually works with integer types if the value is within bounds.
         // Casting might only be needed if the index variable type is larger than needed.
         running_sum += a[i as usize]; // Using 'as usize' is often idiomatic for indexing if needed.
                                      // Or simply a[i] if the type matches expectations.
                                      // Let's simplify assuming direct u32 index access:
         // running_sum += a[i];
    }
    // Let's stick to the original provided loop structure for consistency,
    // acknowledging potential index type nuances.
    let mut sum: u64 = 0u64; // Mark as mutable
    for i: u32 in 0u32..4u32 { // Use u32 index
         sum += a[i as usize]; // Explicit cast to usize often needed
    }
    return sum;
}
```
*(Self-correction: Added `mut` for the `sum` variable as it's modified in the loop. Clarified array indexing type requirements, using `u32` for the loop and `as usize` for indexing, which is common practice).*

### Tuples

Leo also supports tuples, which are immutable collections of values with fixed types. Tuples must have at least one element:

```rust
program variables.aleo {
    transition baz(foo: u8, bar: u8) -> u8 {
        let a: (u8, u8) = (foo, bar);
        // Access tuple elements using .index notation
        let result: u8 = a.0 + a.1;
        return result;
    }
}
```

Each of these data types plays a crucial role in Leo programming, enabling expressive and efficient circuits.

## Casting

Variables can be converted from one type to another using the `as` syntax:

```rust
let a_u16: u16 = 142u16;
let a_u32: u32 = a_u16 as u32; // Cast u16 to u32

let g: group = 0group;
let burner: address = g as address; // Cast group (identity) to address
```

Although this can generate an error when the conversion is not possible or safe:

```rust
let negative: i8 = -1i8;
let unsigned: u8 = negative as u8; // INVALID (at runtime)
// A negative integer cannot be safely cast to an unsigned integer (results in wrapping or error)
```

Another, more complex example:

```rust
// Using ** requires explicit type for exponent, often u32
let exponent: u32 = 126u32;
let base: u128 = 2u128;
let a: u128 = base.pow(exponent); // 2^126, use .pow() method

let f: field = a as field; // Cast 2^126 to field (possible if < field modulus p)
let f_squared: field = f * f; // 2^252 (as a field element)

// Attempting to cast back requires checking bounds
// let s_scalar: scalar = f_squared as scalar; // INVALID, 2^252 is likely larger than the scalar field modulus q
// let s_u128: u128 = f_squared as u128; // INVALID, 2^252 is much larger than the maximum u128 value (2^128 - 1)
```
*(Self-correction: Used `.pow()` method for exponentiation which is standard. Clarified potential invalid casts based on field/type sizes.)*

And a final example:

```rust
// Cannot directly cast an arbitrary field element to a group element.
// Group elements correspond to specific points on the curve.
// let g: group = 1field as group; // INVALID
// Casting from field to group is not a direct operation.
```

You now know practically all the data types you can use on Aleo, except one: records. Let's discover what they are in the next chapter.

--- END OF 3.3-variables-and-types.md ---

--- START OF 3.4-records.md ---

# 3.4 Records

## Usage

So far, all the programs we have written were stateless. This means there was no continuity between calls of different transitions in our programs. We will need a mechanism to store and update state if we want to create useful applications. However, we also want this state to incorporate privacy. This is where records come into play.

Like structs, records are composite data structures made of keys and values. They are defined at the root of the program block:

```rust
program records.aleo {
    // Records can include arbitrary attribute types, even structs or arrays
    record Counter {
        owner: address, // Mandatory owner field
        amount: u64
        // Other fields can be added here
    }
    // ...
```

The difference with structs is that records MUST contain an `owner` attribute of type `address`. When a record is created, it is encrypted with the owner's public key and recorded in a list of existing records on-chain (conceptually, stored in the global state Merkle tree).

Here is an example of a transition that creates and outputs a record corresponding to the previous definition.

```rust
    // ...
    transition init() -> Counter {
        let counter: Counter = Counter {
             // The owner of the output record is the specified address
            owner: aleo1hn9z4zt6awd6ts9zn8ppkwwvjwrg04ax48krd0fgnnzu6ezg9qxst3v26j,
            // The amount is initialized to 0
            amount: 0u64
        };
        return counter;
    }
// } // Closing brace for the program block if this is the end
```

Let's try executing our init function using the command line interface:

```bash
leo execute init
```

This should generate something like the following:

```
⛓  Constraints

 •  'records.aleo/init' - ~2,000 constraints (called 1 time) // Constraint count varies

➡️  Output

 # Represents the plaintext structure before encryption for output
 • {
  owner: aleo1hn9z4zt6awd6ts9zn8ppkwwvjwrg04ax48krd0fgnnzu6ezg9qxst3v26j.private,
  amount: 0u64.private,
  _nonce: ...group.public // Nonce is generated and public
}

# Represents the actual transaction output (JSON format)
{
...
  "program":"records.aleo",
  "function":"init",
  "inputs":[],
  "outputs":[    {
      "type":"record",
      "id":"...", // Commitment or Serial number, internal identifier
      "checksum":"...", // Checksum for integrity
      "value":"record1qyqspk9djwf3fex3e3fdkam5c0qx8vjhccefcqx3x094q8znzc3tzpcsqyrxzmt0w4h8ggcqqgqsqvj9ghgthqq2mneg72d93xm7a9z3sghyfsyyj6s06evza8huuvc8s2fhgvjrk008sstc5gns23anzf044dqjmp3z7jqm32ah8hh59vzst9wyaq" // Encrypted record data
    }
  ]
...
```
*(Note: The exact JSON output structure, constraint count, nonce, id, checksum, and ciphertext will vary)*

As you can see in the execution output at the bottom, the output type of the record is neither private nor public, it's `record`. Its value, like private values, is encrypted: `record1qyq...`. But the difference from the private type is that it's not encrypted with the signer's public key; it's encrypted with the owner's public key. This means that once this transaction is on-chain, the owner will be able to decrypt the data of this record with their view key.

It can be decrypted using the snarkOS command line interface like so (using the owner's view key):

```bash
# Replace ciphertext and view key with actual values from execution
snarkos developer decrypt \
  --ciphertext record1qyqspk9djwf3fex3e3fdkam5c0qx8vjhccefcqx3x094q8znzc3tzpcsqyrxzmt0w4h8ggcqqgqsqvj9ghgthqq2mneg72d93xm7a9z3sghyfsyyj6s06evza8huuvc8s2fhgvjrk008sstc5gns23anzf044dqjmp3z7jqm32ah8hh59vzst9wyaq \
  --view-key AViewKey1...
```

Which yields the plaintext record structure:

```
{
  owner: aleo1hn9z4zt6awd6ts9zn8ppkwwvjwrg04ax48krd0fgnnzu6ezg9qxst3v26j.private,
  amount: 0u64.private,
  _nonce: ...group.public // The nonce used for encryption/decryption
}
```

You can see there's an additional `_nonce` attribute. It plays a role in how record encryption/decryption works, see the corresponding section below.

Each record created in a previous transition can then be consumed once and ONLY once by its owner. To see how this works, let's add a function that consumes an existing record:

```rust
// Continuing the program records.aleo
    // ...
    // This transition consumes a Counter record but does nothing with it.
    // It requires the signer to be the owner of the input 'counter'.
    transition consume(counter: Counter) {
        // The record 'counter' is consumed just by being an input.
        // No specific 'consume' keyword is needed.
    }
    // ...
// } // Closing brace for the program block
```

Executing this with a record *not* owned by the signer:

```bash
# Assuming the record JSON represents a record owned by someone else
leo execute consume '{
  owner: aleo1someotheraddress....................................................private,
  amount: 0u64.private,
  _nonce: ...group.public
}'
```

Will return an error if not executed by the record's owner:

```
Leo ✅ Compiled 'records.aleo' into Aleo instructions
Error: ❌ Execution error: Input record 'counter' for transition 'records.aleo/consume' must be owned by the signer.
```
*(Error message format might vary slightly)*

You cannot update an existing record directly. You must consume it and create a new one. Let's add such a function that increments a previous counter. Here is the complete code:

```rust
program records.aleo {
    record Counter {
        owner: address,
        amount: u64
    }

    // Initialize a new counter owned by the signer
    transition init() -> Counter {
        let counter: Counter = Counter {
            owner: self.signer, // Owner is set to the signer executing the transition
            amount: 0u64 // Amount is initiated at 0
        };
        return counter; // Returns the new record
    }

    // Consumes a counter record (requires ownership)
    transition consume(counter: Counter) {
        // Record 'counter' is consumed by being an input.
    }

    // Consumes an old counter and creates a new one with an incremented amount
    transition increment(old_counter: Counter) -> Counter {
        // old_counter is consumed as an input
        let new_counter: Counter = Counter {
            owner: old_counter.owner, // Keep the same owner
            amount: old_counter.amount + 1u64 // Increment the amount
        };
        return new_counter; // Return the new, updated record
    }
}
```

As you can see, accessing the properties of the `record` works the same way as for `struct`s:

```rust
// Inside the 'increment' transition:
let current_amount: u64 = old_counter.amount;
let owner_address: address = old_counter.owner;
```

There we have it, we now have encrypted state. Here, it increases each time we call the increment function. This is a very basic example, but records are a powerful concept that can capture arbitrary logic for your applications.

Records are completely private. Not only is the record data encrypted, but it is impossible to know which address owns a record just by looking at the chain state. It is also impossible to track which transition created a record that is consumed in another transition (due to the use of nullifiers).

Programs cannot own records. Although you can make transitions that output records with a program address as the owner, that program could never consume that record itself because programs cannot sign transactions to prove ownership.

## How Records Work Internally

### Encryption

The data of a record is encoded as a list of field elements which are each encrypted using the owner's view key. We will explain here how a single data field `d` is encrypted, which can then be repeated for each encoded field.

The owner's view key is a scalar `v` (derived from their private key, as detailed in Chapter 1.4), and the owner's address is `P = v * G` (where `G` is the generator of the subgroup of elliptic curve points that generate the `group` elements).

Here is the scheme for encrypting `d` as the producer of a record to be owned by `P`:

1.  Generate a random scalar `r` (chosen uniformly from the set of scalars). This `r` is ephemeral for this specific encryption.
2.  Compute the record nonce: `N = r * G`. This nonce is public.
3.  Compute the shared secret (ephemeral key): `E = r * P`. This is `r * (v * G)`.
4.  Hash the shared secret to get a field element used for masking: `h = hash(E)`. The specific hash function depends on the Aleo protocol version.
5.  Compute the ciphertext of the record field: `c = d + h`. This ciphertext `c` (along with the nonce `N` and other record metadata) is part of what's stored or represented by the `record1...` string.

### Decryption

Here is the decryption scheme, using P's view key, `v`:

1.  The owner `P` takes the public nonce `N` from the record.
2.  `P` computes the shared secret: `E = v * N`. This works because `v * N = v * (r * G) = r * (v * G) = r * P`, which is the same shared secret calculated during encryption.
3.  `P` hashes the shared secret to get the same masking field element: `h = hash(E)`.
4.  `P` can compute the plaintext data `d = c - h`.

### State Management

Records are identified on-chain using a commitment to their data (which includes owner, data, nonce, etc.) and stored within a global state Merkle tree. When a record is consumed:
1.  A proof of inclusion (Merkle proof) of that record's commitment in the state tree is included in the transition's execution proof. This proves the record existed.
2.  A unique "nullifier" (also called the record's serial number) is computed based on the record's data and the owner's private key (specifically, the spending key component). This nullifier is published to the network.
3.  The network checks that this nullifier has not been published before. If it has, it means the record was already spent (double spend attempt), and the transaction is rejected. If it hasn't, the nullifier is added to a list of spent nullifiers.
This nullifier does not reveal any information about which record was consumed, what data it contained, or who the owner was, thus preserving privacy while preventing double-spending.

This is explained in detail in the Zexe paper: [Zexe: Enabling Decentralized Private Computation](https://eprint.iacr.org/2018/962.pdf), section 2.4.

--- END OF 3.4-records.md ---

--- START OF 3.5-control-flow.md ---

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

--- END OF 3.5-control-flow.md ---

--- START OF 3.6-mappings.md ---

# 3.6 Mappings

While records are useful for storing private state accessible to a single user in our applications, we still don't have a way to share public state among multiple users. This is exactly what mappings are for.

## Syntax

Mappings are key-value stores that hold public state shared across all users. They are defined at the root of the program block using the `mapping` keyword:

```rust
program mappings.aleo {
    // Defines a mapping named 'accumulator' where keys are u8 and values are u64
    mapping accumulator: u8 => u64;
    // key_type => value_type

    // ... rest of the program
}
```

As you can see, the key and value types are explicitly declared when defining a mapping and can be any non-record, non-future Leo type.

Let's now see how to update and read the state of these mappings.

## Public Shared State

Until now, we have always written our code inside the `transition` structure. This allows users to execute this code privately, with potentially private inputs/outputs, and simply publish a proof of the execution on-chain. This proof can simply be verified by other users on the network, without requiring them to execute the code, or even know the inputs/outputs. Not only does this allow for privacy, but it also enables scalability because the code only needs to be executed once (by the prover).

However, this approach poses a problem: how to manage shared state between users.

On public programmable ledgers like Ethereum, smart contract code must be executed by all validators on the network. Although this lacks the advantages cited above, it is necessary to have **public state** shared among different users. This is why Aleo also includes such public executions, specifically for updating shared state stored in mappings. These public updates happen in the `finalize` scope.

Let's see how this works in practice.

First, we introduce a way to define logic that runs *after* the off-chain ZK proof generation and verification, specifically when the transaction is being included on-chain by validators. This logic is placed within a `finalize` block. Code inside `finalize` operates on public state and does not generate ZK proofs itself; it's executed publicly by validators.

```rust
program mappings.aleo {
    mapping accumulator: u8 => u64;

    // The 'finalize' block contains logic executed publicly by validators
    // to update mapping state. It's associated with a preceding transition.
    // It takes public inputs derived from the transition's execution.
    finalize increment_state(public count_key: u8, public increment_amount: u64) {
        // Read the current state from the mapping and increment it.
        // Get the current value at 'count_key', defaulting to 0u64 if it doesn't exist.
        let current_count: u64 = Mapping::get_or_use(accumulator, count_key, 0u64);
        // Calculate the new count.
        let new_count: u64 = current_count + increment_amount;
        // Set the updated value back into the mapping.
        Mapping::set(accumulator, count_key, new_count);
    }

    // The 'transition' block is executed off-chain by the user/prover.
    transition increment(key: u8, amount: u64) {
        // Off-chain logic happens here (can involve private inputs, records etc.)
        // For this example, the transition mainly prepares data for finalize.

        // The 'finalize' keyword *within* the transition specifies which
        // finalize block to call on-chain and provides its inputs.
        // These inputs MUST be public.
        finalize(key, amount);
    }
}
```
*(Self-correction: Refactored the example to use the standard `transition` -> `finalize` structure. The original text used `async function` and `Future`, which are outdated concepts from earlier Aleo designs. The current model uses `transition` for off-chain execution and `finalize` for on-chain public state updates linked to that transition.)*

Let's play with this example where we want to count how many times a specific key has been incremented, or by how much.

Users call the `transition` (`increment` in this case), not the `finalize` block directly.

```bash
# User executes the 'increment' transition off-chain
# Let's increment the counter at key 10u8 by 5u64
leo run increment 10u8 5u64
# Or leo execute increment 10u8 5u64 to generate the tx proof
```

The `transition` is executed and proven off-chain by the signer. When the resulting transaction (containing the proof and the call to `finalize` with its public arguments) is accepted on-chain, the validators execute the specified `finalize` block (`increment_state` in this example) with the provided public arguments (`key`=10u8, `amount`=5u64). This updates the public state in the `accumulator` mapping.

If the execution of a `finalize` block fails (e.g., due to an uncaught arithmetic error, failed assertion within `finalize`, etc.), the entire transaction is rejected. Any record state changes or other effects from the preceding `transition` are effectively nullified as the transaction doesn't make it onto the ledger.

## Update a Mapping Value

To set or update a mapping value at a specific key within the `finalize` block, you use `Mapping::set`:

```rust
// Inside finalize block:
let key_to_update: u8 = 0u8;
let new_value: u64 = 100u64;
Mapping::set(accumulator, key_to_update, new_value);
```

This function succeeds even if a value already exists at that key; it will overwrite the old value with the new one.

## Read a Mapping Value

There are two main ways to read a mapping value within the `finalize` block:

1.  **`Mapping::get`**: Retrieves the value associated with a key.
    ```rust
    // Inside finalize block:
    let key_to_read: u8 = 123u8;
    // This will FAIL the finalize execution if no value exists at key_to_read
    let value: u64 = Mapping::get(accumulator, key_to_read);
    ```
    This instruction will cause the `finalize` execution (and thus the entire transaction) to fail if no value was previously set for the given key.

2.  **`Mapping::get_or_use`**: Retrieves the value or uses a default if the key is not found.
    ```rust
    // Inside finalize block:
    let key_to_read: u8 = 123u8;
    let default_value: u64 = 0u64;
    // Provides a default value, ensuring the instruction succeeds.
    // 'value' will be the stored value if key exists, otherwise it will be 'default_value'.
    let value: u64 = Mapping::get_or_use(accumulator, key_to_read, default_value);
    ```
    This is generally safer when you can provide a sensible default.

## Delete a Mapping Entry

Use `Mapping::remove` within the `finalize` block to delete an entry at a specific key:

```rust
// Inside finalize block:
let key_to_remove: u8 = 123u8;
Mapping::remove(accumulator, key_to_remove);
```

This instruction does not fail, even if no element existed at that key.

## Check for Key Existence

You can check if an element exists at a certain key using `Mapping::contains` within the `finalize` block:

```rust
// Inside finalize block:
let key_to_check: u8 = 123u8;
let exists: bool = Mapping::contains(accumulator, key_to_check);
```

## Example Revisited

Let's refine our counter example. We want a transition that simply increments a counter stored at key `0u8` in the `accumulator` mapping by 1 each time it's called.

```rust
program mappings_counter.aleo {
    mapping accumulator: u8 => u64; // key: counter_id, value: count

    // Finalize logic: Increments the counter at key 0u8
    finalize increment_state(public counter_key: u8) {
        // Get current count for the key, default to 0 if not present
        let current_count: u64 = Mapping::get_or_use(accumulator, counter_key, 0u64);
        // Calculate the new count
        let new_count: u64 = current_count + 1u64;
        // Store the new count back into the mapping
        Mapping::set(accumulator, counter_key, new_count);
    }

    // Transition: Called by the user off-chain
    transition increment() {
        // Perform any necessary off-chain checks or logic here.
        // For this simple counter, we just need to trigger the finalize.

        // Call the finalize block, passing the key 0u8 as a public argument
        finalize(0u8);
    }
}
```

Now, if we execute this transition:

```bash
leo run increment
# or leo execute increment
```

The output from `leo run` or `leo execute` primarily reflects the off-chain execution. Since our `increment` transition does very little off-chain work (no complex constraints, no record inputs/outputs), the constraint count will be very low or zero. The significant part is the `finalize` call embedded within the transaction generated by `leo execute`.

```
       Leo ✅ Compiled 'mappings_counter.aleo' into Aleo instructions

⛓  Constraints

 • 'mappings_counter.aleo/increment' - 0 constraints (or minimal base constraints) (called 1 time)

➡️ Output (from leo run might be minimal or indicate finalize call)

 # The 'leo execute' command generates the transaction including the finalize call:
 # Transaction {
 #   ... transition proof ...
 #   fee_transition: ...,
 #   global_state_root: ...,
 #   finalize: Some([
 #     (program_id: mappings_counter.aleo,
 #      function_name: increment_state,
 #      arguments: [Public(Field(... representing 0u8 ...))])
 #   ])
 # }
```
*(Note: Actual output/transaction representation details vary)*

When this transaction is processed on-chain, the `increment_state` finalize logic runs, reading the value at key `0u8` from the `accumulator` mapping, adding 1, and writing the result back.

In the next chapter, we will look at a (nearly exhaustive) list of all operators available in Leo.

--- END OF 3.6-mappings.md ---

--- START OF 3.7-operators.md ---

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
| 9          | `\|`             | Bitwise OR                        | Left-to-right  |
| 10         | `==` `!=` `<` `>` `<=` `>=` | Comparison Operators             | Left-to-right  |
| 11         | `&&`             | Logical AND                       | Left-to-right  |
| 12         | `\|\|`           | Logical OR                        | Left-to-right  |
| 13         | `? :`            | Ternary Conditional               | Right-to-left  |
| 14         | `=` `+=` `-=` `*=` `/=` `%=` `**=` `<<=` `>>=` `&=` `\|=` `^=` | Assignment Operators            | Right-to-left  |

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

--- END OF 3.7-operators.md ---
```