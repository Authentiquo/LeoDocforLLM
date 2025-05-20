---
id: fundamentals
sidebar_label: Aleo Fundamentals
---

# 1.1 Aleo Fundamentals

Aleo is a blockchain focused on privacy and programmability. It is based on two main components: SnarkVM (a virtual machine for private execution and ZK proof generation) and SnarkOS (node software and AleoBFT consensus).

## From Logic to ZK Proof

- You write a program representing the assertion to be proven (e.g., proving you are over 18 without revealing your age).
- The program is transformed into logical constraints (R1CS), then into a single equation that can be verified quickly.
- You obtain a ZK proof that can be publicly verified without revealing sensitive data.

## Aleo Network Architecture

- **User**: Generates the ZK proof locally or via a third-party prover service, then submits it to the network.
- **Third-Party Prover**: Specialized service for generating proofs on behalf of the user.
- **Validator**: Validates proofs and maintains consensus via AleoBFT.
- **Client**: Synchronizes blocks and provides access to the ledger state.
- **Prover**: Solves coinbase puzzles and receives rewards.

![User and Third-Party Prover](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252FKuUTi2uWby11pbEmQFgR%252Fimage.png%3Falt%3Dmedia%26token%3D1e353df3-60d2-4acb-8205-a450b8804721&width=768&dpr=4&quality=100&sign=a5c90dec&sv=2)

## Executing a Private Transaction

1. The user prepares the inputs.
2. Retrieves the program.
3. Executes the function locally (SnarkVM).
4. Generates the proof and broadcasts it with encrypted inputs/outputs.
5. The validator verifies the proof and consensus is reached.
6. The transaction is added to the ledger.

This model enables scalable private programmability, with off-chain computation and on-chain validation. 