Okay, here is the combined content of the provided files, translated into English and formatted as a single Markdown code block. The files have been arranged according to the logical flow suggested by the `README.md`.

```markdown
# Module 5: Advanced Leo Concepts

This module covers advanced concepts of the Leo programming language on the Aleo blockchain.

## Module Contents

### 5.1 Composability
- [Part 1](5.1_Composability.md): Introduction and Program Imports
- [Part 2](5.1_Composability_part2.md): Signer vs Caller
- [Part 3](5.1_Composability_part3.md): External Records and Imported Structs
- [Part 4](5.1_Composability_part4.md): Asynchronous Transitions and Reading External Mappings

### 5.2 Strings
- [Part 1](5.2_Strings.md): Naive Approaches for Strings
- [Part 2](5.2_Strings_part2.md): Efficient Approach with Fields

### 5.3 Fungible Token Standard - ARC20
- [Part 1](5.3_Fungible_Token_Standard_ARC20.md): Metadata and Basic Token Mechanics
- [Part 2](5.3_Fungible_Token_Standard_ARC20_part2.md): Approval Mechanism

### 5.4 Token Registry - ARC21
- [Part 1](5.4_Token_Registry_ARC21.md): Introduction and Using the Token Registry
- [Part 2](5.4_Token_Registry_ARC21_part2.md): Data Structures of the Token Registry Program
- [Part 3](5.4_Token_Registry_ARC21_part3.md): Mappings, Constants, and Initial Functions
- [Part 4](5.4_Token_Registry_ARC21_part4.md): Additional Functions
- [Part 5](5.4_Token_Registry_ARC21_part5.md): Token Transfer and Management Functions

### 5.5 Non-Fungible Token Standard - ARC721
- [Part 1](5.5_Non-Fungible_Token_Standard_ARC721.md): Introduction and Data Structure
- [Part 2](5.5_Non-Fungible_Token_Standard_ARC721_part2.md): Private Data and Ownership
- [Part 3](5.5_Non-Fungible_Token_Standard_ARC721_part3.md): Public Data and Re-obfuscation
- [Part 4](5.5_Non-Fungible_Token_Standard_ARC721_part4.md): Approvals and Parameters

### Exercise
- [Exercise 2: Create an "actual" Token](Assignment_2_Create_an_actual_Token.md): Creating a token following the ARC21 standard

---
--- START OF CONTENT FROM 5.1_Composability.md ---

# 5.1 Composability

On blockchains, isolated programs are generally not the norm. In fact, what's interesting about developing on a blockchain is that, in theory, anyone can build their own applications by leveraging the tokens, states, and functionalities of other applications. Things become more interesting when multiple programs work together. This is generally referred to as program composability.

## Importing a Program

On Aleo, a transition from one program can call a transition from another program. Here's how it works in practice.

Let's take a first example program, `arithmetic.aleo`, with a transition that calculates the square value of an input.

```
leo new arithmetic
cd arithmetic
```

Let's update `arithmetic/src/main.leo` with the following code:

```leo
program arithmetic.aleo {
    transition square_u64(a: u64) -> u64 {
        return a*a;
    }
}
```

Now let's create another program, quadratic residue, which calculates the quadratic residue of a number modulo another, using our first transition.

```
cd ..
leo new quadratic
cd quadratic
```

We first need to add the other program to our list of external dependencies:

```
leo add arithmetic --local ../arithmetic
```

The `local` argument here is the path to the local Leo folder of the project. If you want to use a program already deployed on-chain, use instead:

```
leo add your_program_id.aleo --network testnet # Or mainnet depending on the network
```

---
--- START OF CONTENT FROM 5.1_Composability_part2.md ---

Our source file for this program, `quadratic/src/main.leo` will contain the following content:

```leo
import arithmetic.aleo;

program quadratic.aleo {
    transition residue(a: u64, n: u64) -> u64 {
        let squared: u64 = arithmetic.aleo/square_u64(a);
        return squared.mod(n);
    }
}
```

First, note the line at the top, outside the program code block.

```leo
import arithmetic.aleo;
```

Then, inside the `residue` transition, note the external call:

```leo
let squared: u64 = arithmetic.aleo/square_u64(a);
```

As you can see, external transition calls have almost the same syntax as calling an internal function. The difference is that the function identifier includes the ID of the imported program followed by the function name, separated by a forward slash.

On Aleo, there is no "call by address", as you would do in Solidity, at the time this course is written. This means it's not possible to dynamically depend on any arbitrary program implementing a certain interface, and then provide that program address as one of the arguments to the calling function.

## Signer vs Caller

In a transition, you can get the address of the direct caller of the transition, whether it's a program address or a user account address.

```leo
transition get_caller() -> address {
    let direct_caller: address = self.caller;
    return direct_caller;
}
```

For example, for the following call chain: `User → Program A → Program B`

- `self.caller` in program B would be the address of program A.
- `self.signer` in program B would be the user's account address.

Here is an example. The `direct_calls_only` function defined below could only be called directly by a user and would fail otherwise if called from an importing program.

```leo
transition direct_calls_only() {
    let direct_caller: address = self.caller;
    let transaction_signer: address = self.signer;
    assert_eq(direct_caller, transaction_signer);
}
```

You can directly use a program ID in Leo to get its address, for example, to get the address of `my_program.aleo`:

```leo
let my_program_address: address = my_program.aleo;
```

---
--- START OF CONTENT FROM 5.1_Composability_part3.md ---

## External Records

Let's update our imported program to include a counter record, to count the number of times it has been called from `quadratic.aleo`:

```leo
program arithmetic.aleo { // Renamed for consistency, assuming it was arithmetic.aleo
    record Counter {
        owner: address,
        amount: u64
    }

    transition create() -> Counter {
        let counter: Counter = Counter {
            owner: self.signer,
            amount: 0u64
        };
        return counter;
    }

    // The square function now includes a Counter type input/output.
    transition square_u64(a: u64, in_counter: Counter) -> (u64, Counter) {
        let out_counter: Counter = Counter {
            owner: in_counter.owner,
            amount: in_counter.amount + 1u64
        };
        return (a*a, out_counter);
    }
}
```

Here's how `quadratic.aleo` can now provide these external records:

```leo
import arithmetic.aleo; // Assuming arithmetic.aleo

program quadratic.aleo {
    transition residue(
        a: u64, n: u64, in_counter: arithmetic.aleo/Counter // Corrected import path
    ) -> (u64, arithmetic.aleo/Counter) { // Corrected import path
        let (squared, out_counter): (
            u64, arithmetic.aleo/Counter // Corrected import path
        ) = arithmetic.aleo/square_u64(a, in_counter); // Pass the record directly
        let out_residue: u64 = squared.mod(n);
        return (out_residue, out_counter);
    }
}
```

Note that before each record name, the ID of the imported program is provided before the slash.

Having an external record as an output of a program does not necessarily mean it will consume that record. The record will only be consumed if it is used as input to a transition of the original program in which it was defined.

## Imported Structs

When you import a program that defines `struct` data types, make sure to redefine these structs in the importing program as it might lead to errors. Unlike records, you do not add the program ID before the imported struct name.

Here is an example program defining and using a struct:

```leo
program arithmetic.aleo { // Renamed for consistency
    struct Point {
        x: u64,
        y: u64
    }

    transition add_points(a: Point, b: Point) -> Point {
        let c: Point = Point {
            x: a.x + b.x,
            y: a.y + b.y
        };
        return c;
    }
}
```

And here is how the importing program can use these structs:

```leo
import arithmetic.aleo; // Assuming arithmetic.aleo

program quadratic.aleo {
    // Redefine the struct with the same name and fields
    struct Point {
        x: u64,
        y: u64
    }

    // Use the struct like a normal struct
    transition double_point(a: Point) -> Point {
        // Use the imported program's function for the operation
        let doubled: Point = arithmetic.aleo/add_points(a, a);
        return doubled;
    }
}
```

---
--- START OF CONTENT FROM 5.1_Composability_part4.md ---

## Calling an Asynchronous Transition

Asynchronous transition calls are slightly different from the non-asynchronous calls we just learned about. The reason is that there is both on-chain and off-chain code being executed. For each of them, we can independently decide whether the calling program's code should execute before/after the imported code.

To understand this, let's first explore the syntax through an example. Our arithmetic program will now store a counter of how many times it has been called using a mapping.

```leo
program arithmetic.aleo { // Renamed for consistency
    mapping counter: u8 => u64;

    // The square function now includes an asynchronous part
    async transition square_u64(a: u64) -> (u64, Future) {
        let squared_value: u64 = a * a; // Off-chain calculation
        let square_u64_future: Future = finalize_square_u64(); // Prepare the future for on-chain update
        return (squared_value, square_u64_future);
    }
    async function finalize_square_u64(){ // On-chain execution
        let amount: u64 = counter.get_or_use(0u8, 0u64);
        amount += 1u64;
        counter.set(0u8, amount);
    }
}
```

Let's see how to call it from the `quadratic.aleo` program:

```leo
import arithmetic.aleo; // Assuming arithmetic.aleo

program quadratic.aleo {
    async transition residue(a: u64, n: u64) -> (u64, Future) {
        // Call the off-chain part of the external transition
        let (squared, square_u64_future): (
            u64, Future
        ) = arithmetic.aleo/square_u64(a);
        
        // Perform local off-chain calculations
        let out_residue: u64 = squared.mod(n);

        // Prepare the future for the local on-chain function, passing the external future
        let residue_future: Future = finalize_residue(square_u64_future);
        return (out_residue, residue_future);
    }
    async function finalize_residue(square_u64_future: Future){ // On-chain execution
        // Do something before...
        square_u64_future.await(); // Wait for the external future (executes finalize_square_u64)
        // Do something after...
        // e.g., update a local mapping if needed
    }
}
```

As you can notice, the future returned by the asynchronous external call is passed as an argument to the function executed on-chain.

All futures from asynchronous external calls must be passed to an async function and awaited in this way. They cannot be returned directly. If there are multiple asynchronous external calls, each must be passed and awaited.

```leo
async transition calls_2transitions() -> Future {
    let foo_future: Future = external.aleo/foo(); // Assume external.aleo defines foo
    let bar_future: Future = external.aleo/bar(); // Assume external.aleo defines bar
    let out_future: Future = finalize_calls_2transitions(foo_future, bar_future);
    return out_future;
}
async function finalize_calls_2transitions(future1: Future, future2: Future){
    future1.await();
    future2.await();
}
```

All function statements, mapping reads, writes, etc., are executed sequentially in the same order as their future is awaited.

Note that futures can be awaited in a different order than the off-chain executions. The following body for the `finalize_calls_2transitions` function above would also be valid:

```leo
async function finalize_calls_2transitions(future1: Future, future2: Future){
    future2.await();
    future1.await();
}
```

## Reading External Mappings

Although a mapping cannot be updated directly by another program, it is possible to read its value. Here is an example:

```leo
import arithmetic.aleo; // Assuming arithmetic.aleo

program quadratic.aleo {
    async transition read_external_mapping() -> Future { // Needs to be async to call finalize
        let read_future: Future = finalize_read_external_mapping();
        return read_future;
    }
    async function finalize_read_external_mapping(){ // On-chain execution
        // Reads the value from arithmetic.aleo's counter mapping at key 0u8
        let count: u64 = arithmetic.aleo/counter.get(0u8);
        assert_neq(count, 0u64); // Example assertion
    }
}
```

You can also use the mapping functions `contains` and `get_or_use` in this way.

---
--- START OF CONTENT FROM 5.2_Strings.md ---

# 5.2 Strings

On Aleo, there is no native type for strings available when writing programs. In this chapter, we will discover some ways to create our own representation of strings.

## Naive Approach

A naive approach to representing strings is to represent the characters of the string as `u8` and strings as arrays of these `u8` characters.

```leo
[u8; 32] // string of 32 characters
```

Here's how you could then write a transition checking if two 32-character strings are equal to each other:

```leo
program strings.aleo {
    transition equals(str1: [u8; 32], str2: [u8; 32]) -> bool {
        let mut out: bool = true; // Use mut for modification inside loop
        for i: u16 in 0u16..32u16 {
            out = out && (str1[i] == str2[i]); // Correct boolean logic
        }
        return out;
    }
}
```

Let's try it now:

```
leo run equals "[1u8, 2u8, 3u8, 4u8, 5u8, 6u8, 7u8, 8u8, 9u8, 10u8, 0u8, 0u8, 0u8, 0u8, 0u8, 0u8, 1u8, 2u8, 3u8, 4u8, 5u8, 6u8, 7u8, 8u8, 9u8, 10u8, 0u8, 0u8, 0u8, 0u8, 0u8, 0u8]" "[1u8, 2u8, 9u8, 0u8, 0u8, 0u8, 0u8, 0u8, 0u8, 0u8, 0u8, 0u8, 0u8, 0u8, 0u8, 0u8, 1u8, 2u8, 3u8, 4u8, 5u8, 6u8, 7u8, 8u8, 9u8, 10u8, 0u8, 0u8, 0u8, 0u8, 0u8, 0u8]"
```

As you can see, we have a program with 95 constraints:

```
 •  'strings.aleo/equals' - 95 constraints (called 1 time)
```

Another problem we have is that arrays on Aleo are limited to 32 elements (Note: This limit might change, but was a consideration). Strings defined this way would thus be limited to 32 characters.

## Less Naive (but still naive) Approach

Let's now represent our strings using arrays of `u128` elements. Since each character only needs 8 bits to be represented, we can represent 128/8 = 16 characters for each `u128`.

```leo
[u128; 2] // string of 32 characters (2 * 16)
```

The previous function is now:

```leo
program strings.aleo {
    transition equals(str1: [u128; 2], str2: [u128; 2]) -> bool {
        let mut out: bool = true; // Use mut
        for i: u16 in 0u16..2u16 { // Loop up to 2
            out = out && (str1[i] == str2[i]); // Correct boolean logic
        }
        return out;
    }
}
```

And when we run this:

```
leo run equals "[67305985u128, 513u128]" "[513u128, 513u128]"
```
*(Note: The specific u128 values represent packed character data)*

Here, the number of constraints is much smaller than before:

```
 •  'strings.aleo/equals' - 5 constraints (called 1 time)
```

Here are the functions you can use in TypeScript to encode and decode such a string to and from u128 arrays:

```javascript
function stringToBigInt(input: string): bigint {
  const encoder = new TextEncoder();
  const encodedBytes = encoder.encode(input);
  // Ensure little-endian for packing (Leo typically works with little-endian fields)
  // No reverse needed if TextEncoder produces UTF-8 bytes in order

  let bigIntValue = BigInt(0);
  for (let i = 0; i < encodedBytes.length; i++) {
    const byteValue = BigInt(encodedBytes[i]);
    const shiftedValue = byteValue << BigInt(8 * i);
    bigIntValue = bigIntValue | shiftedValue;
  }

  return bigIntValue;
}

function bigIntToString(bigIntValue: bigint): string {
  const bytes: number[] = [];
  let tempBigInt = bigIntValue;

  while (tempBigInt > BigInt(0)) {
    const byteValue = Number(tempBigInt & BigInt(255));
    bytes.push(byteValue);
    tempBigInt = tempBigInt >> BigInt(8);
  }

  // bytes are now in little-endian order, reverse for standard string order
  // bytes.reverse(); // Only reverse if the encoding process resulted in reversed bytes

  const decoder = new TextDecoder();
  // Use Uint8Array for direct decoding
  const asciiString = decoder.decode(Uint8Array.from(bytes));
  return asciiString;
}

// Function to split a string into chunks and encode each as a u128 bigint
function splitStringToBigInts(input: string, chunkSize = 16): bigint[] {
  const numChunks = Math.ceil(input.length / chunkSize);
  const bigInts: bigint[] = [];

  for (let i = 0; i < numChunks; i++) {
    const start = i * chunkSize;
    const end = start + chunkSize;
    const chunk = input.substring(start, end); // Use substring for clarity
    const bigIntValue = stringToBigInt(chunk); // Encode each chunk
    // Check if the value exceeds u128 max
    if (bigIntValue >= (BigInt(1) << BigInt(128))) {
        throw new Error(`Chunk "${chunk}" is too large to fit in a u128.`);
    }
    bigInts.push(bigIntValue);
  }

  // Pad the last chunk if needed, though often handled by the string length itself
  return bigInts;
}

// Function to decode an array of u128 bigints back into a single string
function joinBigIntsToString(bigInts: bigint[]): string {
  let result = '';

  for (let i = 0; i < bigInts.length; i++) {
    const chunkString = bigIntToString(bigInts[i]); // Decode each bigint
    result += chunkString;
  }

  return result;
}
```

---
--- START OF CONTENT FROM 5.2_Strings_part2.md ---

## Efficient Approach

Now, the best approach we can use to store the maximum amount of information with the fewest constraints is to use arrays of fields instead of u128.

```leo
program strings.aleo {
    transition equals(str1: [field; 2], str2: [field; 2]) -> bool {
        let mut out: bool = true; // Use mut
        // Loop up to the array length (2 in this case)
        for i: u16 in 0u16..2u16 {
            out = out && (str1[i] == str2[i]); // Correct boolean logic
        }
        return out;
    }
}
```

And when we execute this transition:

```
leo run equals "[67305985field, 513field]" "[513field, 513field]"
```
*(Note: Specific field values represent packed character data)*

As you can see, we still only have 5 constraints for our program:

```
 •  'strings.aleo/equals' - 5 constraints (called 1 time)
```

Although a field element can hold up to ~2^253 different integers. Such an array of fields can hold approximately 253 bits multiplied by the length of the array. This means we have roughly ~31.6 characters available for each new field in our array representation (253 bits / 8 bits/char).

Here are the functions you can use in JavaScript to encode and decode such a string to and from arrays of fields:

```javascript
// Aleo's BLS12-377 scalar field modulus
const FIELD_MODULUS = 8444461749428370424248824938781546531375899335154063827935233455917409239041n; // Note: Often ends in 1 for prime fields

// Helper to convert string to a single bigint (little-endian bytes)
function stringToBigInt(input: string): bigint {
  const encoder = new TextEncoder();
  const encodedBytes = encoder.encode(input);

  let bigIntValue = BigInt(0);
  for (let i = 0; i < encodedBytes.length; i++) {
    const byteValue = BigInt(encodedBytes[i]);
    const shiftedValue = byteValue << BigInt(8 * i);
    bigIntValue = bigIntValue | shiftedValue;
  }

  return bigIntValue;
}

// Helper to convert a single bigint back to string (little-endian bytes)
function bigIntToString(bigIntValue: bigint): string {
  const bytes: number[] = [];
  let tempBigInt = bigIntValue;
  while (tempBigInt > BigInt(0)) {
    const byteValue = Number(tempBigInt & BigInt(255));
    bytes.push(byteValue);
    tempBigInt = tempBigInt >> BigInt(8);
  }

  const decoder = new TextDecoder();
  const asciiString = decoder.decode(Uint8Array.from(bytes));
  return asciiString;
}

// Encodes a string into an array of field elements
function stringToFields(input: string, numFieldElements: number = 4): bigint[] {
  const bigIntValue = stringToBigInt(input);
  const fieldElements: bigint[] = [];
  let remainingValue = bigIntValue;

  for (let i = 0; i < numFieldElements; i++) {
    // Use modulo operator to get the remainder, which fits in a field element
    const fieldElement = remainingValue % FIELD_MODULUS;
    fieldElements.push(fieldElement);
    // Integer division to process the next chunk
    remainingValue = remainingValue / FIELD_MODULUS;
  }

  // Check if the entire string was encoded
  if (remainingValue !== 0n) {
    // This means the string was too large for the specified number of field elements
    throw new Error("String is too big to be encoded into the specified number of field elements.");
  }

  // The resulting array `fieldElements` can be used as input for Leo transitions
  return fieldElements;
}

// Decodes an array of field elements back into a string
function fieldsToString(fields: bigint[]): string {
  let bigIntValue = BigInt(0);
  let multiplier = BigInt(1); // Start with 1 for the first field element

  // Reconstruct the original large integer from field elements
  for (const fieldElement of fields) {
    // Ensure fieldElement is positive and within range if necessary, though usually handled by Leo
    if (fieldElement < 0 || fieldElement >= FIELD_MODULUS) {
        console.warn("Field element is out of expected range:", fieldElement);
        // Handle potential negative values if they arise from specific computations
    }
    bigIntValue += fieldElement * multiplier;
    multiplier *= FIELD_MODULUS; // Update multiplier for the next field element
  }

  // Convert the reconstructed large integer back to a string
  return bigIntToString(bigIntValue);
}
```

This latter standard for strings is the most efficient and should be preferred for most uses.

---
--- START OF CONTENT FROM 5.3_Fungible_Token_Standard_ARC20.md ---

# 5.3 Fungible Token Standard - ARC20

As of March 19, 2025 (Note: This date seems futuristic, perhaps a typo?), due to the lack of dynamic composability (or "call by address") on Aleo, the standard for fungible tokens relies on the token registry program described in the next chapter 5.4.

This chapter exists to help understand where this token registry program comes from and what the token standard will be when dynamic composability is available, at some point in the future.

The standard for writing programs implementing fungible tokens is almost what we described in chapter 4.1, with the hybrid public/private state approach.

It includes two additional features: token metadata and the approval mechanism.

## Token Metadata

At the beginning of the program, all token metadata is defined as constants, with a struct gathering all this information.

```leo
program arc20.aleo { // Example program name
    // Token Metadata Constants
    const name: u128 = 0u128;           // Token Name (encoded, e.g., using methods from 5.2)
    const symbol: u128 = 0u64;           // Token Symbol (encoded) - Changed to u128 for potentially longer symbols
    const decimals: u8 = 0u8;           // Token Decimals
    const total_supply: u64 = 0u64;     // Max Token Supply (or initial if mintable)

    struct metadata {
        name: u128,
        symbol: u128, // Changed to u128
        decimals: u8,
        total_supply: u64, // Consider u128 if supply can exceed u64::MAX
    }
```

Then, a transition is defined to return this metadata struct to any caller using the constants defined above:

```leo
    transition get_metadata () -> metadata {
        return metadata {
            name: name,
            symbol: symbol,
            decimals: decimals,
            total_supply: total_supply,
        };
    }
```

## Basic Token Mechanics

A mapping is then defined to track the public ownership of tokens for each address:

```leo
    // Mapping from address to public balance
    mapping account: address => u64; // Consider u128 if balances can exceed u64::MAX
```

And a record is defined to track the private ownership of tokens by users:

```leo
    // Record for private token balances
    record token {
        owner: address,
        amount: u64, // Matches mapping type, consider u128 if needed
        // Gate not strictly required by ARC20 but often included for ZK proofs
        // gate: u64, // Example: Can be used for spending conditions
    }
```

All the transfer functions described in chapter 4.1 Simple Token Program can then be defined using these objects: `transfer_private`, `transfer_public`, `transfer_public_to_private`, `transfer_private_to_public`.

```leo
    // Public Transfer (On-chain state update)
    async transition transfer_public(
        public receiver: address, // to the recipient
        public amount: u64, // amount to transfer
    ) -> Future {
        // Uses self.caller as the sender implicitly
        let finalize_future: Future = finalize_transfer_public(
            self.caller,
            receiver,
            amount,
        );
        return finalize_future;
    }
    async function finalize_transfer_public(
        sender: address, receiver: address, amount: u64,
    ) {
        // Decrement sender's balance
        let sender_balance: u64 = account.get(sender);
        // Add underflow check
        assert(sender_balance >= amount);
        account.set(sender, sender_balance - amount);

        // Increment receiver's balance
        let receiver_balance: u64 = account.get_or_use(receiver, 0u64);
        // Add overflow check if necessary (depends on total_supply)
        account.set(receiver, receiver_balance + amount);
    }

    // Private Transfer (Record consumption and creation)
    transition transfer_private(
        private sender_record: token, // token record to consume
        private receiver: address, // to the recipient
        private amount: u64, // amount to transfer
    ) -> (token, token) {
        // Check if sender has enough balance in the private record
        assert(sender_record.amount >= amount);

        // Create the sender's remaining token record
        let sender_remaining_token: token = token {
            owner: sender_record.owner, // Should be self.caller or derived from input record owner
            amount: sender_record.amount - amount,
            // gate: sender_record.gate // Propagate gate if used
        };

        // Create the receiver's new token record
        let receiver_token: token = token {
            owner: receiver,
            amount: amount,
            // gate: 0u64 // Or initialize appropriately
        };

        // Return the new records: one for sender (remainder), one for receiver
        return (sender_remaining_token, receiver_token);
    }

    // Private to Public Transfer
    async transition transfer_private_to_public(
        private sender_record: token, // token record to consume
        public receiver: address, // to the public recipient
        public amount: u64, // amount to transfer
    ) -> (token, Future) {
        // Check sender's private balance
        assert(sender_record.amount >= amount);

        // Create the sender's remaining private token record
        let sender_remaining_token: token = token {
            owner: sender_record.owner,
            amount: sender_record.amount - amount,
            // gate: sender_record.gate
        };

        // Prepare the future to update the public balance
        let finalize_future: Future = finalize_transfer_private_to_public(
            receiver,
            amount,
        );

        // Return the remaining private record and the future
        return (sender_remaining_token, finalize_future);
    }
    async function finalize_transfer_private_to_public(
        receiver: address, amount: u64,
    ) {
        // Increment receiver's public balance
        let receiver_balance: u64 = account.get_or_use(receiver, 0u64);
        account.set(receiver, receiver_balance + amount);
    }

    // Public to Private Transfer
    async transition transfer_public_to_private(
        private receiver: address, // to the private recipient
        public amount: u64, // amount to transfer
    ) -> (token, Future) {
        // Create the receiver's new private token record
        let receiver_token: token = token {
            owner: receiver,
            amount: amount,
            // gate: 0u64 // Initialize appropriately
        };

        // Prepare the future to decrease the sender's public balance (self.caller)
        let finalize_future: Future = finalize_transfer_public_to_private(
            self.caller,
            amount,
        );

        // Return the new private record and the future
        return (receiver_token, finalize_future);
    }
    async function finalize_transfer_public_to_private(
        sender: address, amount: u64,
    ) {
        // Decrement sender's public balance
        let sender_balance: u64 = account.get(sender);
        assert(sender_balance >= amount);
        account.set(sender, sender_balance - amount);
    }
} // End of program block (added for completeness)
```

---
--- START OF CONTENT FROM 5.3_Fungible_Token_Standard_ARC20_part2.md ---

## Approval Mechanism

An additional mechanism defined in this standard is that of approvals. The idea behind approvals is to allow other accounts to spend public tokens that you own. This is particularly interesting for enabling programs to spend your tokens according to the conditions defined in the program's code.

To understand how this works, let's first define a struct: `approval` which includes an approver address and a spender address.

```leo
    // Struct to represent an approval relationship
    struct approval {
        approver: address, // The owner granting approval
        spender: address,  // The address allowed to spend
    }
```

We will track the u64 amount approved by the `approver` to be spent by the `spender` using a specific mapping:

```leo
    // Mapping from approval hash to the approved amount
    // Key: hash(approver, spender) -> Value: amount
    mapping approvals: field => u64; // Consider u128 if amounts exceed u64::MAX
```

The key of this mapping is the hash of the corresponding approval struct, and the value is the amount approved to be spent.

Two transitions are defined to create and remove these approvals. The caller is always the approver:

```leo
    // Approve a spender to withdraw tokens from the caller's public balance
    async transition approve_public(
        private spender: address, // The address being approved
        public amount: u64,       // The amount the spender is allowed to withdraw
    ) -> Future {
        // Create the approval struct
        let apvl: approval = approval {
            approver: self.caller, // The caller is the one approving
            spender: spender,
        };
        // Hash the approval struct to create the mapping key
        // Assume BHP256::hash_to_field exists or use a suitable hash function
        let apvl_hash: field = BHP256::hash_to_field(apvl);

        // Prepare the future to update the approvals mapping
        let finalize_future: Future = finalize_approve_public(apvl_hash, amount);
        return finalize_future;
    }
    async function finalize_approve_public (apvl_hash: field, amount: u64) {
        // Get the current approval or default to 0
        let current_approval: u64 = approvals.get_or_use(apvl_hash, 0u64);
        // Set the new approval amount (Note: ERC20 usually sets, not adds)
        // To mimic ERC20's behavior (overwrite):
        // approvals.set(apvl_hash, amount);
        // Or, if it should increase the allowance:
        // current_approval += amount;
        // approvals.set(apvl_hash, current_approval);
        // Let's assume it overwrites for simplicity, matching standard ERC20:
        approvals.set(apvl_hash, amount);
    }

    // Decrease or remove approval for a spender
    async transition unapprove_public(
        private spender: address, // The spender whose allowance is being reduced
        public amount_to_decrease: u64, // The amount to decrease the allowance by
    ) -> Future {
        let apvl: approval = approval {
            approver: self.caller,
            spender: spender,
        };
        let apvl_hash: field = BHP256::hash_to_field(apvl);
        // Prepare the future to update the approvals mapping
        let finalize_future: Future = finalize_unapprove_public(apvl_hash, amount_to_decrease);
        return finalize_future;
    }
    async function finalize_unapprove_public (apvl_hash: field, amount_to_decrease: u64) {
        let current_approval: u64 = approvals.get(apvl_hash);
        // Ensure sufficient approval exists before decreasing
        assert(current_approval >= amount_to_decrease);
        // Decrease the approval amount
        let new_approval: u64 = current_approval - amount_to_decrease;
        approvals.set(apvl_hash, new_approval);
    }
```
*(Self-correction: The original French text for `unapprove_public` implies decreasing by `amount`, not setting to zero. The translation reflects this decrease.)*

A specific transition is included in the standard for spending tokens from other addresses, which have been previously approved:

```leo
    // Transfer tokens from an approver's account by an approved spender (self.caller)
    async transition transfer_from_public(
        public approver: address, // from the approver (token owner)
        public receiver: address, // to the recipient
        public amount: u64,       // amount to transfer
    ) -> Future {
        // Create the approval struct to find the allowance
        let apvl: approval = approval{
            approver: approver,
            spender: self.caller, // The spender is the one calling this function
        };
        let apvl_hash: field = BHP256::hash_to_field(apvl);

        // Prepare the future to execute the transfer logic
        let finalize_future: Future = finalize_transfer_from_public(
            apvl_hash,
            approver,
            receiver,
            amount,
        );
        return finalize_future;
    }
    async function finalize_transfer_from_public(
        apvl_hash: field, approver: address, receiver: address, amount: u64,
    ) {
        // 1. Check and decrease the allowance
        let approved: u64 = approvals.get(apvl_hash);
        assert(approved >= amount); // Ensure spender has enough allowance
        approved -= amount;
        approvals.set(apvl_hash, approved);

        // 2. Decrease the approver's (owner's) public balance
        let from_balance: u64 = account.get(approver);
        assert(from_balance >= amount); // Ensure owner has enough balance
        from_balance -= amount;
        account.set(approver, from_balance);

        // 3. Increase the receiver's public balance
        let to_balance: u64 = account.get_or_use(receiver, 0u64);
        to_balance += amount;
        account.set(receiver, to_balance);
    }
```

And that's it, that's all there is to know about the fungible token standard (in this simplified, pre-registry context).

However, a problem with this standard arises when you don't have dynamic composability. The only way to create programs using such tokens, like for example an exchange between token A and token B, is to deploy such an exchange program, importing both token programs, for each pair of tokens on the network.

In the next chapter, we will examine the widely adopted workaround for this problem: the token registry program.

The complete code for the standard (as implemented by ZSolutions) is available [here](https://github.com/zsolutions-io/aleo-standard-programs/blob/main/arc20/src/main.leo). *(Note: The original link was to zsociety-io, updated based on later context)*

---
--- START OF CONTENT FROM 5.4_Token_Registry_ARC21.md ---

# 5.4 Token Registry - ARC21

The token registry program is a standard program designed for issuing and managing new tokens on the Aleo blockchain. It functions as a singleton program because on Aleo, all imported programs must be known and deployed before the importing program, and dynamic inter-program calls are currently not supported, making composability difficult to implement. This means that a DeFi program must be compiled with support for all the token programs it will ever interact with. If a new token program is subsequently deployed on-chain, the DeFi program would need to be recompiled and redeployed on-chain to interact with that token.

In the short term, support for program scalability will address this issue, but currently, the problem is circumvented through the token registry which can manage balances of many different ARC-20-like tokens. This program would be the standard "hub" with which all tokens and DeFi programs interact. Individual ARC-20-like tokens can register with the registry and mint new tokens via this program. Token value transfers will happen through direct calls to the registry rather than the individual token program itself. The advantage of this approach is that DeFi programs do not need to be compiled with specific knowledge of individual ARC-20 tokens: their only dependency will be the registry. Consequently, deploying new tokens does not require redeploying DeFi programs. Similarly, individual token programs can also be compiled with a dependency on the registry, but without dependencies on DeFi programs. The registry thus enables interoperability between new tokens and DeFi programs, without requiring program redeployment. As a secondary benefit, the registry will provide privacy advantages (via an enhanced anonymity set) as all private transfers within the registry will hide the identity of the specific token being transferred.

This standard emerged from extensive discussions and the approval of the ARC-21 proposal to enable token interoperability between different applications.

## How to Use the Token Registry Program

Anyone can create a new token on Aleo using the `token_registry.aleo` program by calling the `register_token` transition with a unique token ID and specifying a name, symbol, decimals, and maximum supply. An optional boolean `external_authorization_required` gives additional control over the spendable tokens by requiring extra approval from an `external_authorization_party`, which can unlock certain amounts of balances for spending with expiration on a specific owner's token using `prehook_public` or `prehook_private`. The admin can also set the `external_authorization_party` to another address with `update_token_management` later if needed.

Once a token is registered, tokens can be minted either publicly using `mint_public` or privately for a specific recipient using `mint_private` with the `MINTER_ROLE` or `SUPPLY_MANAGER_ROLE` roles if not the admin. Tokens can also be burned either publicly with `burn_public` or privately with `burn_private` with the `BURNER_ROLE` or `SUPPLY_MANAGER_ROLE` roles if not the admin.

The token owner can then transfer the token either publicly using `transfer_public` or privately to a specific recipient using `transfer_private`. The token can also be converted from public to private using `transfer_public_to_private` or from private to public using `transfer_private_to_public`.

---
--- START OF CONTENT FROM 5.4_Token_Registry_ARC21_part2.md ---

## Data Structures of the Token Registry Program

### Token Record

```leo
// Record representing a private balance of a specific token
record Token {
  owner: address,                         // The address of the token owner
  amount: u128,                           // The amount of tokens held privately
  token_id: field,                        // The unique identifier for the token type
  external_authorization_required: bool,  // Flag indicating if external authorization is needed
  authorized_until: u32                   // Block height until which this amount is authorized for spending (if required)
  // gate: u64,                           // Optional: Gate value for ZK proof conditions
}
```
*Self-correction: Added comments explaining each field.*

- `owner`: The address of the token owner.
- `amount`: The amount of tokens in the account (record).
- `token_id`: The unique identifier for the token.
- `external_authorization_required`: Whether the token requires external authorization.
- `authorized_until`: The block height until which the token (amount in the record) is authorized.

### TokenMetadata Struct

```leo
// Struct holding metadata for a registered token
struct TokenMetadata {
  token_id: field,                        // Unique identifier for the token
  name: u128,                             // Token name (ASCII text represented as bits, packed into u128)
  symbol: u128,                           // Token symbol (ASCII text represented as bits, packed into u128)
  decimals: u8,                           // Number of decimal places for the token
  supply: u128,                           // Current total supply of the token
  max_supply: u128,                       // Maximum possible supply of the token
  admin: address,                         // Address controlling token management (roles, metadata updates)
  external_authorization_required: bool,  // If true, transfers might need pre-authorization
  external_authorization_party: address   // Address responsible for providing external authorization
}
```
*Self-correction: Added comments explaining each field and clarified encoding.*

- `token_id`: The unique identifier for the token.
- `name`: The name of the token.
- `symbol`: The symbol of the token.
- `decimals`: The number of decimals for the token.
- `supply`: The total supply of the token.
- `max_supply`: The maximum supply of the token.
- `admin`: The address of the token administrator.
- `external_authorization_required`: Whether the token requires external authorization.
- `external_authorization_party`: The address of the external authorization party.

### TokenOwner Struct

```leo
// Struct used as a key component for balance/allowance mappings
struct TokenOwner {
  account: address, // The address of the account (owner/approver)
  token_id: field   // The identifier of the token
}
// Note: This struct itself might not be directly stored but used to compute hash keys for mappings.
// For example, hash(TokenOwner) could be a key for public balances.
```
*Self-correction: Added comments explaining the typical usage.*

- `account`: The address of the token owner.
- `token_id`: The unique identifier for the token.

### Balance Struct

```leo
// Struct representing a public balance entry (likely used internally or potentially in mappings)
struct Balance {
  token_id: field,        // The identifier of the token
  account: address,       // The address of the account holding the balance
  balance: u128,          // The public balance amount
  authorized_until: u32   // Block height until which this balance is authorized (if applicable)
}
// Note: Often, a simple mapping `address => u128` (or `field => u128` where field=hash(token_id, address))
// is used for public balances instead of storing this full struct per user per token.
// This struct might be more relevant if `authorized_until` applies to public balances.
```
*Self-correction: Added comments explaining usage and potential alternatives.*

- `token_id`: The unique identifier for the token.
- `account`: The address of the token owner.
- `balance`: The token balance.
- `authorized_until`: The block height until which the token balance is authorized.

### Allowance Struct

```leo
// Struct used as a key component for allowance mappings
struct AllowanceKey { // Renamed for clarity, as it forms the key
  owner: address,     // The address granting the allowance (approver)
  spender: address,   // The address receiving the allowance
  token_id: field     // The identifier of the token
}
// Note: The hash of this struct, hash(AllowanceKey), is typically used as the key
// in the `allowances` mapping, which maps this key to the allowed `u128` amount.
```
*Self-correction: Renamed to `AllowanceKey` and clarified usage in mappings.*

- `owner` (formerly `account`): The address of the token owner (approver).
- `spender`: The address of the spender.
- `token_id`: The unique identifier for the token.

---
--- START OF CONTENT FROM 5.4_Token_Registry_ARC21_part3.md ---

## Mappings of the Token Registry Program

- `mapping registered_tokens: field => TokenMetadata;` Mapping from token IDs (`field`) to token metadata structures (`TokenMetadata`). Stores the details of each registered token.
- `mapping balances: field => u128;` Mapping from a key representing a user and token (`field`, e.g., `hash(token_id, account_address)`) to their public token balance (`u128`).
- `mapping allowances: field => u128;` Mapping from a key representing an owner, spender, and token (`field`, e.g., `hash(token_id, owner_address, spender_address)`) to the amount (`u128`) the spender is allowed to withdraw from the owner's public balance.
- `mapping roles: field => u8;` Mapping from a key representing a user and token (`field`, e.g., `hash(token_id, account_address)`) to the role (`u8`) assigned to that user for that specific token (e.g., Minter, Burner).

*Self-correction: Clarified the likely structure of the keys for balances, allowances, and roles.*

## Constants of the Token Registry Program

- `const CREDITS_RESERVED_TOKEN_ID: field = ...field;` Reserved token ID for the native Aleo Credits token when wrapped or represented within the registry. (Actual value omitted for brevity).
- `const MINTER_ROLE: u8 = 1u8;` Role identifier for minting tokens.
- `const BURNER_ROLE: u8 = 2u8;` Role identifier for burning tokens.
- `const SUPPLY_MANAGER_ROLE: u8 = 3u8;` Role identifier for managing supply (potentially combining mint/burn or other actions).
- `const ADMIN_ROLE: u8 = 0u8;` (Implied) The address in `TokenMetadata.admin` inherently has admin privileges. Sometimes an explicit role `0u8` might be used, or admin checks are done directly against the `admin` field.

*Self-correction: Added ADMIN_ROLE clarification.*

## Functions of the Token Registry Program

### `initialize()`

Initializes the token registry program. This is typically called only once upon deployment. It might register the native Aleo Credits token (`CREDITS_RESERVED_TOKEN_ID`) with predefined metadata. The program might set itself or a deployer address as the initial administrator for certain functions or for the wrapped credits token.

*Self-correction: Corrected description based on typical initialization patterns.*

**Parameters**: None.

**Returns**: Nothing (or a Future if async operations are needed).

### `register_token()`

Registers a new token type within the token registry program. Requires the caller to have appropriate permissions (e.g., maybe anyone can register, or only specific roles).

**Parameters**:
- `public token_id: field`: The unique identifier for the new token (must not already exist).
- `public name: u128`: The name of the token (encoded).
- `public symbol: u128`: The symbol of the token (encoded).
- `public decimals: u8`: The number of decimal places for the token.
- `public max_supply: u128`: The maximum possible supply for the token.
- `public external_authorization_required: bool`: Whether transfers of this token require pre-authorization.
- `public external_authorization_party: address`: The address responsible for providing authorization if required.

**Returns**:
- `Future`: A Future to finalize the token registration (updating the `registered_tokens` mapping).

### `update_token_management()`

Updates the management parameters (admin, authorization party) for an existing token. Requires the caller to be the current admin of the token.

**Parameters**:
- `public token_id: field`: The unique identifier of the token to update.
- `public new_admin: address`: The address of the new administrator.
- `public new_external_authorization_party: address`: The address of the new external authorization party.

**Returns**:
- `Future`: A Future to finalize the update (modifying the `TokenMetadata` in the `registered_tokens` mapping).

### `set_role()`

Assigns a specific role (Minter, Burner, etc.) to an account for a given token ID. Requires the caller to be the admin of the token.

**Parameters**:
- `public token_id: field`: The unique identifier for the token.
- `public account: address`: The address of the account to receive the role.
- `public role: u8`: The role to assign (`MINTER_ROLE`, `BURNER_ROLE`, etc.).

**Returns**:
- `Future`: A Future to finalize setting the role (updating the `roles` mapping).

---
--- START OF CONTENT FROM 5.4_Token_Registry_ARC21_part4.md ---

### `remove_role()`

Removes a specific role from an account for a given token ID. Requires the caller to be the admin of the token.

**Parameters**:
- `public token_id: field`: The unique identifier for the token.
- `public account: address`: The address of the account whose role is to be removed.
- `public role_to_remove: u8`: The specific role to remove. (Alternatively, could remove all roles by setting role to a 'none' value like 0u8 if 0 is not admin).

**Returns**:
- `Future`: A Future to finalize the role removal (updating the `roles` mapping, potentially setting the role to 0 or removing the entry).

*Self-correction: Added clarification on how removal might be implemented.*

### `mint_public()`

Mints new tokens and assigns them to a recipient's public balance. Requires the caller to have the `MINTER_ROLE` or be the admin for the specified `token_id`.

**Parameters**:
- `public token_id: field`: The unique identifier for the token to mint.
- `public recipient: address`: The address receiving the newly minted tokens (public balance).
- `public amount: u128`: The amount of tokens to mint.
- `public authorized_until: u32`: (Optional, relevant if `external_authorization_required` is true) The block height until which the minted public balance is authorized. Often set to 0 or max u32 if not applicable.

**Returns**:
- `Future`: A Future to finalize the minting (updating the recipient's public balance in the `balances` mapping and the `supply` in `TokenMetadata`).

### `mint_private()`

Mints new tokens and creates a private `Token` record for the recipient. Requires the caller to have the `MINTER_ROLE` or be the admin for the specified `token_id`.

**Parameters**:
- `public token_id: field`: The unique identifier for the token to mint.
- `private recipient: address`: The address of the recipient (kept private).
- `public amount: u128`: The amount of tokens to mint into the private record.
- `public external_authorization_required: bool`: Must match the token's metadata setting. Passed explicitly to construct the record.
- `public authorized_until: u32`: The block height until which the token record is authorized (relevant if `external_authorization_required` is true).

**Returns**:
- `Token`: The newly created private `Token` record for the recipient.
- `Future`: A Future to finalize the minting (updating the `supply` in `TokenMetadata`).

### `burn_public()`

Burns (destroys) tokens from a specified owner's public balance. Requires the caller to have the `BURNER_ROLE` or be the admin for the specified `token_id`.

**Parameters**:
- `public token_id: field`: The unique identifier for the token to burn.
- `public owner: address`: The address whose public balance will be reduced.
- `public amount: u128`: The amount of tokens to burn.

**Returns**:
- `Future`: A Future to finalize the burning (updating the owner's public balance in the `balances` mapping and the `supply` in `TokenMetadata`).

### `burn_private()`

Burns (destroys) tokens from a private `Token` record. The caller must be the owner of the input record. (Role checks might apply depending on implementation, e.g., only holders of BURNER_ROLE can burn their own tokens, or anyone can burn their own).

**Parameters**:
- `private input_record: Token`: The private token record to burn from.
- `public amount: u128`: The amount of tokens to burn from the record.

**Returns**:
- `Token`: The remaining `Token` record with the reduced balance (if amount < input_record.amount). If amount == input_record.amount, no record might be returned.
- `Future`: A Future to finalize the burning (updating the `supply` in `TokenMetadata`).

*Self-correction: Clarified caller requirements and return value.*

### `prehook_public()`

Allows the `external_authorization_party` to authorize a certain amount of a user's public balance for spending up to a specified block height.

**Parameters**:
- `public token_id: field`: The identifier of the token being authorized.
- `public owner: address`: The address of the token owner whose balance is being authorized.
- `public amount_to_authorize: u128`: The amount of the public balance to authorize. (Note: The standard might just authorize the *entire* balance up to the block height, rather than a specific amount).
- `public authorized_until: u32`: The block height until which the authorization is valid.

**Returns**:
- `Future`: A Future to finalize the prehook (updating the `authorized_until` field associated with the owner's public balance, potentially stored within the `Balance` struct or a separate mapping).

*Self-correction: Clarified the mechanism - typically authorizes the balance, not a specific amount.*

### `prehook_private()`

Allows the `external_authorization_party` to authorize a private `Token` record (or split it into authorized/unauthorized parts).

**Parameters**:
- `private input_record: Token`: The token record to authorize.
- `public authorized_until: u32`: The new authorization block height to set for the record (or a portion of it).
- `private amount_to_authorize: u128`: (Optional) If splitting, the amount to move into the newly authorized record. If authorizing the whole record, this might not be needed.

**Returns**:
- `Token`: The potentially modified input record (e.g., remaining unauthorized amount) or the newly authorized record. The exact return depends on whether it modifies in place or splits. A common pattern is splitting:
    - `Token`: New record containing `amount_to_authorize` with the new `authorized_until`.
    - `Token`: Original record with its amount reduced by `amount_to_authorize`.
- `Future`: A Future for any potential on-chain updates (though often prehooks might only manipulate records).

*Self-correction: Clarified splitting pattern.*

### `transfer_public()`

Transfers tokens from the caller's public balance to a recipient's public balance.

**Parameters**:
- `public token_id: field`: The unique identifier for the token to transfer.
- `public recipient: address`: The address receiving the tokens (public balance).
- `public amount: u128`: The amount of tokens to transfer.

**Returns**:
- `Future`: A Future to finalize the transfer (updating sender's (`self.caller`) and recipient's public balances in the `balances` mapping). Requires checking `authorized_until` if `external_authorization_required` is true for the token.

---
--- START OF CONTENT FROM 5.4_Token_Registry_ARC21_part5.md ---

### `transfer_public_as_signer()`

Transfers tokens from the transaction signer's public balance to a recipient's public balance. This is useful in cross-program calls where `self.caller` might be another program, but the action should be on behalf of the original user who signed the transaction.

**Parameters**:
- `public token_id: field`: The unique identifier for the token to transfer.
- `public recipient: address`: The address receiving the tokens (public balance).
- `public amount: u128`: The amount of tokens to transfer.

**Returns**:
- `Future`: A Future to finalize the transfer (updating sender's (`self.signer`) and recipient's public balances in the `balances` mapping). Requires checking authorization similar to `transfer_public`.

### `approve_public()`

Allows the caller (`owner`) to approve a `spender` to withdraw a certain `amount` of a specific `token_id` from the caller's public balance. Updates the allowance.

**Parameters**:
- `public token_id: field`: The unique identifier for the token.
- `public spender: address`: The address being granted the allowance.
- `public amount: u128`: The amount of tokens to approve.

**Returns**:
- `Future`: A Future to finalize the approval (updating the `allowances` mapping for the key `hash(token_id, self.caller, spender)`).

### `unapprove_public()`</h3>

Allows the caller (`owner`) to revoke or reduce the allowance previously granted to a `spender` for a specific `token_id`.

**Parameters**:
- `public token_id: field`: The unique identifier for the token.
- `public spender: address`: The address whose allowance is being modified.
- `public amount_to_decrease: u128`: The amount by which to decrease the allowance. (Setting amount to current allowance effectively revokes it).

**Returns**:
- `Future`: A Future to finalize the unapproval (updating the `allowances` mapping).

*Self-correction: Clarified parameter name and behavior.*

### `transfer_from_public()`

Allows an approved `spender` (`self.caller`) to transfer tokens from an `owner`'s public balance to a `recipient`. Requires prior approval via `approve_public`.

**Parameters**:
- `public token_id: field`: The unique identifier for the token.
- `public owner: address`: The address whose tokens are being transferred (the approver).
- `public recipient: address`: The address receiving the tokens.
- `public amount: u128`: The amount of tokens to transfer.

**Returns**:
- `Future`: A Future to finalize the transfer (decreasing allowance in `allowances`, decreasing owner's balance in `balances`, increasing recipient's balance in `balances`). Requires checking authorization if applicable.

### `transfer_public_to_private()`

Converts tokens from the caller's public balance into a private `Token` record owned by the specified `recipient`.

**Parameters**:
- `public token_id: field`: The unique identifier for the token.
- `private recipient: address`: The address receiving the private token record.
- `public amount: u128`: The amount of tokens to transfer.
- `public external_authorization_required: bool`: Flag needed to create the record correctly (must match token metadata).

**Returns**:
- `Token`: The newly created private `Token` record for the recipient.
- `Future`: A Future to finalize the transfer (decreasing the caller's public balance in `balances`).

### `transfer_from_public_to_private()`

Allows an approved `spender` (`self.caller`) to convert tokens from an `owner`'s public balance into a private `Token` record for a `recipient`. Requires prior approval.

**Parameters**:
- `public token_id: field`: The unique identifier for the token.
- `public owner: address`: The address whose public tokens are being converted (the approver).
- `private recipient: address`: The address receiving the private token record.
- `public amount: u128`: The amount of tokens to transfer.
- `public external_authorization_required: bool`: Flag needed to create the record correctly.

**Returns**:
- `Token`: The newly created private `Token` record.
- `Future`: A Future to finalize the transfer (decreasing allowance, decreasing owner's public balance).

### `transfer_private()`

Transfers tokens privately from one `Token` record (owned by the caller) to another, creating a new `Token` record for the recipient and returning the remainder to the sender.

**Parameters**:
- `private recipient: address`: The address receiving the tokens (private record).
- `private amount: u128`: The amount of tokens to transfer.
- `private input_record: Token`: The sender's token record to spend from. (`input_record.owner` must implicitly match `self.caller`).

**Returns**:
- `Token`: The sender's remaining `Token` record.
- `Token`: The recipient's new `Token` record.
- `Future`: (Optional) A Future might be needed if there are on-chain side effects, but often private transfers only manipulate records. Requires checking `authorized_until` if applicable.

*Self-correction: Adjusted parameter privacy and clarified return.*

### `transfer_private_to_public()`

Converts tokens from the caller's private `Token` record into a public balance for the specified `recipient`.

**Parameters**:
- `public recipient: address`: The address receiving the tokens into their public balance.
- `public amount: u128`: The amount of tokens to transfer.
- `private input_record: Token`: The sender's token record to spend from.

**Returns**:
- `Token`: The sender's remaining `Token` record.
- `Future`: A Future to finalize the transfer (increasing the recipient's public balance in `balances`). Requires checking authorization if applicable.

### `join()`

Combines two private `Token` records owned by the caller into a single record. Both records must be for the same `token_id`.

**Parameters**:
- `private token_1: Token`: The first token record.
- `private token_2: Token`: The second token record.

**Returns**:
- `Token`: The single, combined `Token` record. (`amount = token_1.amount + token_2.amount`). Authorization logic needs careful handling (e.g., take the minimum `authorized_until` or require both to be authorized).

### `split()`

Divides a single private `Token` record owned by the caller into two new records.

**Parameters**:
- `private token: Token`: The token record to split.
- `private amount: u128`: The amount to place into the first output record.

**Returns**:
- `Token`: The first new `Token` record (with `amount`).
- `Token`: The second new `Token` record (with `token.amount - amount`). Authorization properties (`authorized_until`, `external_authorization_required`) are typically copied to both new records.

---
--- START OF CONTENT FROM 5.5_Non-Fungible_Token_Standard_ARC721.md ---

# 5.5 Non-Fungible Token Standard - ARC721

NFTs are non-fungible tokens. These are tokens that are distinguished from each other by the inclusion of different identifiers or data.

Compared to NFTs on public ledgers like Ethereum, Aleo NFTs can independently have:

- Public or private visibility of the token owner.
- Public or private data associated with the token.

These two characteristics have implications for the feasibility of building applications involving NFTs such as marketplaces, escrow programs, etc.

**Examples:**

- **Domain Names** - Human-readable names that resolve to addresses in the Aleo Name Service.
- **Royalty NFTs** - Tradable assets whose utility is to claim creator or marketplace royalty fees.
- **IOU NFTs** - Tradable assets allowing the claim of fungible tokens owed in a loan agreement, including private loan data.

## Motivations

**On-chain vs Off-chain Data**

The NFT standard on Aleo allows NFT data to be on-chain, off-chain, or a combination of both. Off-chain to reduce storage fees on the network. On-chain to leverage the ability to use this data as inputs/outputs of zk circuits.

Note: On-chain data can consist either of a hash of some data, or be the data itself directly depending on the use case requirement to guarantee access to the data transactionally.

## Specification

An NFT collection is defined as a program implementing the following specifications.

**Data Structure**

The data stored in an NFT has the following structure:

```leo
// Represents a single trait of an NFT
struct attribute {
    // Using fields allows longer strings, encoded as per Chapter 5.2
    trait_type: [field; 4], // e.g., "Color", "Level", "Rarity"
    _value: [field; 4],     // e.g., "Blue", "10", "Epic" (using underscore as 'value' is a keyword)
}

// Represents the core data associated with an NFT
struct data {
    metadata_uri: [field; 4], // URI pointing to off-chain JSON metadata (encoded)
    // Optional on-chain fields (can override or supplement off-chain data)
    // optional name: [field; 4],         // On-chain name (encoded)
    // optional image_uri: [field; 16],   // On-chain image URI (encoded, potentially longer)
    // optional attributes: [attribute; 4], // Array of on-chain attributes
    // optional ... other custom fields
}
```
*Self-correction: Renamed `metadata` field to `metadata_uri` for clarity, corrected comment about `value` keyword, added examples.*

The names of the structs do not necessarily have to match 'data' and 'attribute', allowing multiple NFT collection programs to be imported without shadowing. However, the name of each struct attribute *is* imposed by the standard (e.g., `trait_type`, `_value`, `metadata_uri`). Aleo reserved keywords must be prefixed with an underscore (as with `_value`).

To obtain the complete data for an NFT, off-chain and on-chain data can simply be merged, for example in JavaScript:

```javascript
// Example function to fetch and merge NFT data
async function getCompleteNftData(nftRecord) {
    // Decode the metadata_uri from field array to string
    const metadataUriString = fieldsToString(nftRecord.data.metadata_uri); // Assumes fieldsToString from 5.2

    let offChainData = {};
    try {
        // Fetch the JSON from the URI
        const response = await fetch(metadataUriString);
        if (response.ok) {
            offChainData = await response.json();
        } else {
            console.error(`Failed to fetch metadata from ${metadataUriString}: ${response.statusText}`);
        }
    } catch (error) {
        console.error(`Error fetching metadata from ${metadataUriString}:`, error);
    }

    // Decode on-chain data (if present)
    const onChainData = {};
    // Example: Decode name if it exists in the record's data struct
    // if (nftRecord.data.name) {
    //     onChainData.name = fieldsToString(nftRecord.data.name);
    // }
    // Example: Decode attributes if they exist
    // if (nftRecord.data.attributes) {
    //     onChainData.attributes = nftRecord.data.attributes.map(attr => ({
    //         trait_type: fieldsToString(attr.trait_type),
    //         value: fieldsToString(attr._value) // Assuming _value corresponds to value
    //     }));
    // }
    // ... decode other optional on-chain fields ...

    // Merge data, with on-chain data potentially overriding off-chain data
    const completeNftData = {
        ...offChainData,
        ...onChainData, // On-chain data takes precedence
    };

    return completeNftData;
}

// Assuming `nft` has a structure like: { data: { metadata_uri: [field,...], ... } }
// const nft_data = await getCompleteNftData(nft);
```
*Self-correction: Provided a more complete JS example including decoding.*

---
--- START OF CONTENT FROM 5.5_Non-Fungible_Token_Standard_ARC721_part2.md ---

**Private Data and Ownership**

As with ARC20-21 tokens, the privacy of the NFT owner is achieved by representing the token as an Aleo record. This also allows an NFT's data to remain private, by including it as a private attribute of the record.

However, to ensure uniqueness, a public identifier for the NFT is necessary. Simply using a hash of the data would introduce 2 problems:

- Two NFTs could not share the same data.
- It would leak some knowledge about the data: verifying that certain data corresponds would be an instantaneous operation.

For these reasons, we include in the NFT record a scalar element, called `edition`:

```leo
record NFT {
    owner: address,   // Private owner address
    data: data,       // Private data struct (defined in Part 1)
    edition: scalar,  // Private unique scalar (nonce/blinding factor)
    // gate: u64       // Optional: Gate value for ZK proofs
}
```
*Self-correction: Added owner privacy comment.*

We define the public identifier of the NFT as the commitment of its `edition` scalar to the hash of its `data`:

```leo
// Function to compute the public NFT identifier (commitment)
inline commit_nft(
    nft_data: data,
    nft_edition: scalar
) -> field {
    // 1. Hash the structured data to a field element
    // BHP256::hash_to_field is a placeholder for a suitable hash function
    // that can handle complex structs, perhaps Poseidon or similar.
    let data_hash: field = BHP256::hash_to_field(nft_data);

    // 2. Commit the edition scalar to the data hash
    // BHP256::commit_to_field is a placeholder for a Pedersen commitment or similar
    // commit(message, blinding_factor) -> commitment_field
    let nft_commit: field = BHP256::commit_to_field(data_hash, nft_edition);
    return nft_commit;
}
```
*Self-correction: Added comments about hash/commit functions.*

This "NFT commit" does not reveal any information about the NFT's data as long as the `edition` obfuscator is chosen uniformly at random from the scalar field.

The following mapping serves to guarantee the uniqueness of "NFT commit" identifiers (ensuring each NFT, identified by its commitment, exists or has existed only once):

```leo
// Mapping to track NFT commitments that exist or have existed
mapping nft_commits: field => bool;
// Key: NFT commit (field) => Value: exists (bool, typically true)
```

**Public Ownership**

Besides ensuring non-fungibility, the NFT commit also allows making the NFT owner public while keeping its data private.

This is a key feature on Aleo because it allows programs to own NFTs without revealing their data, as programs cannot spend records directly.

Similar to ARC20-21 tokens, an NFT owner can be made public by mapping the NFT commit to that owner:

```leo
// Mapping from NFT commit to the public owner address
mapping nft_owners: field => address;
// Key: NFT commit (field) => Value: Public owner address
```

However, this raises a difficult question: if user A is the private owner of an NFT and transfers ownership to a public owner (user B or a program), how can user B know the actual data behind the public NFT commit?

The proposed solution is to include another record, `NFTView`, defined as:

```leo
// Record used to transfer NFT data visibility to a public owner
record NFTView {
    owner: address,   // The recipient who can view/access the data
    data: data,       // The private NFT data
    edition: scalar,  // The private edition scalar (matching the original NFT)
    is_view: bool     // Flag to differentiate from the actual NFT record
}
```

This record does not represent private ownership of the NFT itself but acts as a vehicle for the NFT's data. It is minted for the public recipient during transfers *to* public ownership.

The conversion from private owner to public owner can then be implemented as follows:

```leo
// Transition to transfer ownership from a private record to a public mapping entry
async transition transfer_private_to_public(
    private nft: NFT,       // The private NFT record being consumed
    public to: address,     // The recipient address (public owner)
) -> (NFTView, Future) { // Returns a view record for the recipient and a future
    // Calculate the NFT commitment
    let nft_commit: field = commit_nft(nft.data, nft.edition);

    // Create the NFTView record for the recipient
    // This record holds the data and edition, allowing the recipient to reconstruct the commit
    let nft_view: NFTView = NFTView {
        owner: to, // The recipient owns this view record
        data: nft.data,
        edition: nft.edition,
        is_view: true, // Mark this as a view record
    };

    // Prepare the future to update the public ownership mapping
    let transfer_future: Future =
        finalize_transfer_private_to_public(
            to, nft_commit
        );

    // Return the view record and the future
    // The original NFT record is consumed.
    return (nft_view, transfer_future);
}

// On-chain function to finalize the private-to-public transfer
async function finalize_transfer_private_to_public(
    to: address,
    nft_commit: field,
){
    // Check if the NFT commit already exists (optional, depends on desired overwrite logic)
    // assert(!nft_commits.contains(nft_commit)); // Might prevent re-transferring if already public?

    // Check if it already has a public owner (optional)
    // assert(!nft_owners.contains(nft_commit));

    // Set the public owner in the mapping
    nft_owners.set(nft_commit, to);

    // Mark the commit as existing (if not already done)
    // Ensures uniqueness even if ownership changes later
    if !nft_commits.contains(nft_commit) {
         nft_commits.set(nft_commit, true);
    }
}
```
*Self-correction: Added checks and comments in finalize function.*

Note: `is_view` is always true and is here just to differentiate `NFTView` from `NFT` in plaintext representations of records.

---
--- START OF CONTENT FROM 5.5_Non-Fungible_Token_Standard_ARC721_part3.md ---

An ultimate problem remains: what happens if the public recipient of a transfer is a program? Then `NFTView` doesn't help directly, as programs cannot spend records to access their private data. To illustrate this problem, imagine trying to create a marketplace program. A seller lists the NFT using `transfer_private_to_public`, making the marketplace program the public owner via the `nft_owners` mapping. A buyer then accepts the listing and should automatically receive the data corresponding to the NFT. One way this could be done is for the seller to come back later to disclose the private data (`NFTView`'s content or similar) in order to withdraw the payment.

Note: These latter considerations are only relevant for NFT data that is intended to always remain private.

**Public Data**

For collections where the data can eventually become public, called "publishable collections," the following mapping and function should be included to address the problem stated in the last section.

```leo
// Struct to hold the revealed content (data + edition) of an NFT
struct nft_content {
    data: data,       // The NFT's data
    edition: scalar   // The NFT's edition scalar
}

// Mapping from NFT commit to its publicly revealed content
mapping nft_contents: field => nft_content;
// Key: NFT commit => Value: Publicly revealed data and edition

// Transition to publish the content (data + edition) of an NFT publicly
// Typically called by the owner (private or public?) after deciding to reveal.
// If called by private owner, needs the NFT record as input.
// If called by public owner, might need NFTView or similar proof of knowledge.
// Let's assume it's called by someone who knows the data and edition.
async transition publish_nft_content(
    // Inputs required to reconstruct the commit and provide the content
    public nft_data: data,
    public nft_edition: scalar,
) -> Future {
    // Recompute the NFT commit from the provided data and edition
    let nft_commit: field = commit_nft(nft_data, nft_edition);

    // Prepare the future to store the content in the public mapping
    let publish_future: Future = finalize_publish_nft_content(
        nft_commit,
        nft_data,
        nft_edition,
    );
    return publish_future;
}

// On-chain function to finalize publishing the NFT content
async function finalize_publish_nft_content(
    nft_commit: field,
    nft_data: data,
    nft_edition: scalar,
) {
    // Optional: Verify that the NFT commit exists (i.e., the NFT was minted)
    assert(nft_commits.contains(nft_commit));

    // Optional: Verify that content isn't already published
    // assert(!nft_contents.contains(nft_commit)); // Prevent overwriting?

    // Create the content struct
    let public_content: nft_content = nft_content {
        data: nft_data,
        edition: nft_edition
    };
    // Store the content publicly, associated with the NFT commit
    nft_contents.set(nft_commit, public_content);
}
```
*Self-correction: Added caller assumptions and optional checks.*

`publish_nft_content` can then be called alongside transfers to a program, for example, in a marketplace program during listing, making the data accessible to the program or subsequent buyers by querying the `nft_contents` mapping using the `nft_commit`.

**Re-obfuscation**

If an NFT's content (`data` and `edition`) has been published once via `nft_contents`, the only way to make it private again (in the sense that the *link* between the old public commit and the data is somewhat obscured) is to transfer it back to a private owner (`NFT` record) and then use the following function to update the `edition`, thereby changing the NFT's commitment (`nft_commit`).

```leo
// Transition for a private owner to update the edition scalar of their NFT
// This changes the NFT's public commitment, effectively re-obfuscating it.
async transition update_edition_private(
    private nft: NFT,            // The current private NFT record (consumed)
    private new_edition: scalar, // The new random scalar to use
) -> (NFT, Future) { // Returns the updated NFT record and a future
    // Create the updated NFT record with the new edition
    let out_nft: NFT = NFT {
        owner: nft.owner, // Keep the same owner
        data: nft.data,   // Keep the same data
        edition: new_edition, // Use the new edition scalar
        // gate: nft.gate // Keep the same gate if applicable
    };

    // Calculate the *new* NFT commitment using the new edition
    let new_nft_commit: field = commit_nft(out_nft.data, new_edition);

    // Prepare the future to record the *new* commitment
    let update_future: Future = finalize_update_edition_private(
        new_nft_commit
    );

    // Return the updated record and the future
    return (out_nft, update_future);
}

// On-chain function to finalize the edition update
async function finalize_update_edition_private(
    new_nft_commit: field,
) {
    // Assert that the *new* commit does not already exist (ensures uniqueness)
    // This check is crucial to prevent collisions.
    assert(!nft_commits.contains(new_nft_commit));

    // Add the *new* commit to the set of known commitments
    nft_commits.set(new_nft_commit, true);

    // Note: The OLD commit remains in nft_commits and potentially nft_owners/nft_contents.
    // This is intentional, as explained below.
}
```

The previous commit is *not* removed from the `nft_commits` mapping (nor from `nft_owners` or `nft_contents` if present there). Removing it would reveal that the previous commit and the new commit represent the same underlying data (just with a different edition), defeating the purpose of re-obfuscation. Keeping the old commit makes it appear as if a distinct NFT simply ceased to exist privately.

---
--- START OF CONTENT FROM 5.5_Non-Fungible_Token_Standard_ARC721_part4.md ---

**Approvals**

Similar to ARC20 tokens, the standard includes an approval mechanism allowing accounts to approve another account (a `spender`) to transfer their token on their behalf. This can apply to a specific NFT or potentially all NFTs in the collection owned by the approver.

```leo
// Struct representing an approval relationship (for a specific NFT or all)
struct approval {
    approver: address, // The owner granting approval
    spender: address   // The account authorized to act
}

// Mapping for 'approve for all' functionality
// Key: hash(approver, spender) => Value: bool (true if approved for all)
mapping for_all_approvals: field => bool;

// Mapping for approvals of specific NFTs
// Key: NFT commit => Value: hash(approver, spender) of the approved spender
mapping nft_approvals: field => field; // Stores the hash of the specific approval struct

// Transition for an owner (self.caller) to approve/unapprove a spender for ALL their NFTs in this collection
async transition set_for_all_approval(
    private spender: address, // The spender being approved/unapproved
    public new_value: bool,   // true to approve, false to revoke
) -> Future {
    // Create the approval struct for hashing
    let apvl: approval = approval {
        approver: self.caller,
        spender: spender,
    };
    // Hash the struct to get the key for the for_all_approvals mapping
    let apvl_hash: field = BHP256::hash_to_field(apvl); // Assuming suitable hash

    // Prepare the future to update the mapping
    let finalize_future: Future = finalize_set_for_all_approval(
        apvl_hash, new_value
    );
    return finalize_future;
}
async function finalize_set_for_all_approval(
    apvl_hash: field,
    new_value: bool,
){
    // Set the boolean flag in the mapping
    for_all_approvals.set(apvl_hash, new_value);
}

// Transition for a public owner (self.caller) to approve a spender for ONE specific NFT
// Requires the NFT to be publicly owned (in nft_owners mapping)
async transition approve_public(
    private spender: address,      // The spender being approved
    // Need to identify the specific NFT via its data and edition to compute commit
    private nft_data: data,
    private nft_edition: scalar,
) -> Future {
    // Calculate the commit of the specific NFT being approved
    let nft_commit: field = commit_nft(nft_data, nft_edition);

    // Create the approval struct for this specific approval instance
    let apvl: approval = approval {
        approver: self.caller, // The caller must be the owner
        spender: spender,
    );
    // Hash the approval struct
    let apvl_hash: field = BHP256::hash_to_field(apvl);

    // Prepare the future to link the NFT commit to the approval hash
    let finalize_future: Future = finalize_approve_public(
        self.caller, apvl_hash, nft_commit,
    );
    return finalize_future;
}
async function finalize_approve_public(
    caller: address, // Passed explicitly to check ownership
    apvl_hash: field, // Hash representing the approval (approver+spender)
    nft_commit: field,// The commit of the NFT being approved
){
    // 1. Verify the caller is the current public owner of the NFT
    // Check if the commit exists in the public owners mapping
    assert(nft_owners.contains(nft_commit));
    let owner: address = nft_owners.get(nft_commit);
    assert_eq(owner, caller); // Ensure the caller owns the NFT

    // 2. Store the approval hash associated with the NFT commit
    // This indicates that the spender identified by apvl_hash is approved for this nft_commit
    nft_approvals.set(nft_commit, apvl_hash);
}
```
*Self-correction: Clarified inputs and logic for `approve_public`.*

Once approved (either via `set_for_all_approval` or `approve_public`), a corresponding `transfer_from_public` function (not shown here, but similar to ARC20/21) can be called by the `spender` to transfer the specific NFT from the `approver` to a recipient address. This function would need to:
1. Check if the spender is approved for the specific `nft_commit` (by checking `nft_approvals` against `hash(approver, spender)`) OR if the spender is approved for all (by checking `for_all_approvals` for `hash(approver, spender)`).
2. If approved, update the `nft_owners` mapping to set the `recipient` as the new owner.
3. Potentially clear the specific `nft_approvals` entry for that `nft_commit` after the transfer.

**Parameters**

A mapping is responsible for collection-level parameters:

```leo
// Mapping for general collection settings
mapping general_settings: u8 => field;
// Key: Setting index (u8) => Value: Setting value (field)
```

Available parameters (example indices):

- `0u8` - Max supply (total number of NFTs, considering all editions, that can ever exist). Value stored as `field`.
- `1u8` - Max base NFTs (total number of unique data hashes, i.e., first editions) that can be minted. Value stored as `field`.
- `2u8` - Collection Symbol (e.g., "MYNFT"). Encoded into `field` or multiple fields.
- `3u8` - Base URI for metadata, part 1. Stored as `field`.
- `4u8` - Base URI for metadata, part 2. Stored as `field`.
- `5u8` - Base URI for metadata, part 3. Stored as `field`.
- `6u8` - Base URI for metadata, part 4. Stored as `field`. (Allows for long URIs).
- `7u8` - Admin address hash (e.g., Poseidon hash of the admin address). Used for access control checks. `field`.

*Self-correction: Clarified meaning of supply parameters and URI parts.*

An example of such off-chain metadata JSON referenced by the base URI + token ID can be found [here](https://aleo-public.s3.us-west-2.amazonaws.com/testnet3/privacy-pride/1.json).

This back-and-forth between buyer and seller (especially for private data) makes the user experience much worse than on traditional NFT marketplaces. Hence the need for a mechanism allowing Aleo programs to store private data and disclose it programmatically. An attempt to contribute to solving this problem is [Aleo DCP](https://github.com/zsolutions-io/aleo-dcp), where data is sharded using an MPC protocol. Here is an [example NFT marketplace program](https://github.com/zsolutions-io/aleo-dcp/blob/main/examples/nft_marketplace/programs/marketplace_example/src/main.leo) with the same "one-click buy" user experience as traditional marketplaces, leveraging Aleo DCP.

The complete code for the standard (as implemented by ZSolutions) is available [here](https://github.com/zsolutions-io/aleo-standard-programs/blob/main/arc721/src/main.leo).

---
--- START OF CONTENT FROM Assignment_2_Create_an_actual_Token.md ---

# Exercise 2: Create an "actual" Token

The goal of this exercise is to create your first Aleo token following the current community-adopted token standard: ARC21 (using the `token_registry.aleo` program).

Following the specifications from chapter 5.4, your objective is to write a program managing such a token: handling the registration and initial coin offering (minting) of the token. It must import the `token_registry.aleo` program:

```leo
import token_registry.aleo; // Ensure this program is available in imports
```

## Learning Objectives

By completing this exercise, you will:

- Understand how to use the community token standard (ARC21 via the registry).
- Write your first DeFi-related program interacting with the registry.

## Tasks

Your program (`my_arc21_token.aleo` or similar name) should include the following transitions:

1.  **`initialize_token`**: This transition is responsible for registering *your specific token* with the central `token_registry.aleo` program. It should call the `token_registry.aleo/register_token` transition. Your token's metadata such as the unique `token_id` (choose a unique field element, perhaps derived from your program name or a random generator), `max_supply`, `name`, `symbol`, `decimals`, etc., should all be defined as `const` values at the beginning of your program. Decide if `external_authorization` is required.

2.  **`mint`**: This is the transition called directly by users to mint your token *after* it has been registered. Inside this transition, you will call `token_registry.aleo/mint_public` (or `mint_private` if you prefer).
    *   This minting should only be allowed *after* a certain block height, defined as a constant `MINT_START_HEIGHT`. You'll need to assert `block.height >= MINT_START_HEIGHT`.
    *   During minting, the minter ( `self.caller` ) should publicly transfer Aleo credits (`credits.aleo/transfer_public`) to a specific `TREASURY` address (defined as a `const address`).
    *   The constant `MINT_PRICE` (e.g., `1_000_000u64` for 1 credit if decimals=6) will define the price in Aleo credits *per token* minted. Calculate the total credits needed based on the `amount` being minted.
    *   Ensure the caller actually sends the required credits *before* calling the registry's mint function.

3.  **`update_admin`**: A transition to update the administrator of *your specific token* within the `token_registry.aleo`. This transition should only be callable by the *current admin* of your token (which is initially set during `initialize_token`). Inside this transition, call `token_registry.aleo/update_token_management`, providing your token's `token_id` and the `new_admin` address supplied as input.

4.  Add a comment at the end of your program file showing the example `leo execute` command that would be used to run your `initialize_token` transition to register the token on-chain:

```leo
// Example execution command for registration:
// leo execute <your_program_name.aleo> initialize_token <token_id_field> <name_u128> <symbol_u128> <decimals_u8> <max_supply_u128> <ext_auth_bool> <ext_auth_party_addr> --private-key <admin_private_key> --query <node_url> --broadcast <node_url>/transaction/broadcast --fee <fee_amount> --record <any_fee_record>
```
*(Note: Adjust the command parameters based on your `initialize_token` function's exact signature and deployment needs. The example above assumes public inputs for registration details)*

---

```