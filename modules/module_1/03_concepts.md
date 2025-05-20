---
id: concepts
sidebar_label: Concepts
---

# 1.3 Concepts

## Programs
An Aleo program is the logic and state of an application. It defines the rules to follow, runs with ZK proofs, and uses a statically typed language designed for privacy.

## Transactions
Three types:
- **Deploy Transaction**: Publishes a program on the network.
- **Execute Transaction**: Calls a function of a deployed program.
- **Fee Transaction**: Pays for rejected transactions.

Each transaction has an ID (Merkle Tree Digest) and the ledger ensures their validity.

## Transitions
A transition is a state update within a transaction. It has:
- **id**: Transition ID
- **program_id**: Concerned program
- **function_name**: Executed function
- **inputs/outputs**: Input/output data
- **tpk**: Public signing key
- **tcm**: Transition commitment

## Blocks
A block groups transactions, adds cryptographic proofs, and advances the ledger. It contains:
- Transactions
- Block Header (metadata)
- Ratifications (rewards)
- Coinbase
- Signature

The header tracks state, Merkle root, height, timestamp, etc. Each new block extends the chain and guarantees network security. 