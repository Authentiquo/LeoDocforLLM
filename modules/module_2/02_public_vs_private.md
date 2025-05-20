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