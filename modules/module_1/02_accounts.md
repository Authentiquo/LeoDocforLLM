---
id: accounts
sidebar_label: Accounts
---

# 1.2 Accounts

## Fundamental Aleo Objects

### Fields

Fields are the base elements of the elliptic curve body used by Aleo. All types derive from this fundamental type.

p=8444461749428370424248824938781546531375899335154063827935233455917409239041

### Group

Points of the Edwards BLS12 (G1) subgroup. Used for addresses and public keys.

### Scalar

q=2111115437357092606062206234695386632838870926408408195193685246394721360383

## Traditional Accounts vs Aleo

- **PrivKey**: Authenticates transactions.
- **Address**: Public identifier.
- **View Key**: Decrypts private on-chain state.
- **Compute Key**: Used for zkVM execution proof.

### Generating an Aleo Account

- Generate a random field s (seed).
- Compute private/public keys, view key, and address via hashes and curve multiplications.

![Aleo Account Structure 1](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252FnhgV5kvhUB0sHw0nXwki%252Fimage.png%3Falt%3Dmedia%26token%3D0dfc15ad-2034-41c7-93fd-dc72ccdf1584&width=768&dpr=4&quality=100&sign=4bdc058b&sv=2)

![Aleo Account Structure 2](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252FlHEouuBFKDzNnwQS691z%252Fimage.png%3Falt%3Dmedia%26token%3D9725414f-4b44-491e-b2d8-c492da550c98&width=768&dpr=4&quality=100&sign=730e8656&sv=2) 