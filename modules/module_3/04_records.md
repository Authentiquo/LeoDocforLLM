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