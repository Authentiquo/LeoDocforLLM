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