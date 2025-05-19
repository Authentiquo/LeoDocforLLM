# Module 0: Blockchain Fundamentals and Cryptography

In 2009, for the first time, a community of developers came up with a solution to enable peer-to-peer payments without the need for a central authority to validate transactions: Bitcoin.

This new decentralized system allowed permissionless, verifiable exchanges of value. It did so by introducing a unique consensus mechanism to add new transactions to an existing immutable public ledger. Everyone can then verify the validity of these transactions since they are all public.

Bitcoin paved the way for other ledgers, each with its own extra features. In particular, one caught the world's attention - Ethereum - by allowing users to deploy smart contracts: arbitrary sets of instructions executed by network peers. These programs enabled a wide range of use cases, such as creating custom currencies or developing applications where users can exchange those currencies without needing to trust a centralized authority.

Since then, the cryptocurrency and blockchain industry has been at the center of a significant transformation in finance, payments, identity, gaming... The list of disrupted sectors goes on. However, an important limitation has remained. Because transaction verification relies on their content being public, traditional blockchain data is visible to everyone. Give someone your address to receive funds, and they will be able to retrace all your past transactions as well as your current balance. This limitation, along with the problem of scalability of decentralized ledgers, are major issues that need to be solved for crypto to practically transform real-world industries.

While some blockchains, such as Zcash, proposed a solution for making transaction data private while maintaining verifiability, they still lacked programmability. But in 2018, a group of researchers published a paper describing how to enable decentralized private execution: ZEXE. The implementation of this paper led to the creation of a new blockchain: Aleo.

This course will guide you on how to build private decentralized applications on Aleo. From understanding blockchain core concepts to writing smart contracts, debugging and deploying them, and acquiring the right tools for connecting your app to a frontend, you should have everything you need to begin your builder journey on Aleo.

This course requires no prior knowledge of blockchain. It is aimed at engineers with a mathematical background and experience in programming.

---

# 0.1 Hash Functions

Let's start with understanding the most important tool in the cryptographer's tool box: hash functions. Hash functions are used everywhere: from building quickly searchable data structures, to storing passwords or encrypting messages and verifying data integrity.

Let's say you download a huge file on the internet, that could be a list of transactions in a ledger for instance. Maybe your internet connection is not so stable and some packets might have been tempered with. To ensure your file is not corrupted, you could use a function that takes the file data as an input and computes a fingerprint identifier for the file, and compare that with the identifier posted by the person who shared that file to you, or even the one from other people to ensure you all have the same version of the file. Such a function should be deterministic, fast to compute and collision resistant we'll describe what that means exactly later in that chapter.

Now let's take another example, let's say you run a website where you have to store your users accounts, including an email and password to authenticate them. If you were to store users password directly you would risk them being stolen on the first hack of your database. You could instead store a fingerprint of that password on user registration. Whenever logging in, users would just give their password, you would compute the fingerprint and compare it with the version you stored to ensure it is the right password. Such a function to compute fingerprint should then be pre-image resistance, deterministic, collision resistance as we'll, again, see later.

Both those use cases require a hash function. A **cryptographic hash function** takes an input (a message, a transaction, a block of data, or an entire ledger, whatever sequence of bytes really) and produces a fixed-size element, which is its **hash**. It could be a string of characters, or an element of Z/pZ where Z/pZ is a prime number. This hash is unique to the input data - any change to the input, even a single character, will produce a completely different hash. In more formal terms, if h:{0,1}*→Z/pZ:

- **Injection**: for two keys k1 ≠ k2, the hash function should give different results h(k1) ≠ h(k2), with probability p-1/p.
    
- **Diffusion** (stronger than injection): if k1 ≠ k2, knowing h(k1) gives _no information_ about h(k2). For example, if k2 is exactly the same as k1, except for one bit, then every bit in h(k2) should change with 1/2 probability compared to h(k1). Knowing the bits of h(k1) does not give any information about the bits of h(k2).

A good cryptographic hash function should have the following properties:

- **Deterministic** – The same input always produces the same output.
    
- **Fast computation** – The hash can be quickly computed for any given input.
    
- **Pre-image resistance** – Given a hash, it is nearly impossible to determine the original input.
    
- **Small changes cause big differences** – A tiny modification to the input drastically changes the output.
    
- **Collision resistance** – It is infeasible to find two different inputs that produce the same hash.

## Hashing in Blockchain

Hash function can be used to ensure the integrity and immutability of the ledger itself. A decentralized system must prevent anyone from altering past transactions while still allowing new transactions to be added. This is where **cryptographic hashing** comes into play.

In a blockchain, every block, a new page of transaction in the ledger, contains:

- A list of transactions
    
- A timestamp
    
- A **hash** of the previous block
    
- A **nonce**

By including the hash of the previous block, each block is cryptographically linked to the one before it. This creates a **chain of blocks**, forming an immutable ledger. If someone tries to alter a past transaction, they would need to change all following blocks hash as well, making tampering practically impossible.

## Example: SHA-256

A widely used cryptographic hash function in blockchain systems is **SHA-256 (Secure Hash Algorithm 256-bit)**. When a transaction is processed, it is hashed using SHA-256, producing a unique 256-bit hash. This hash is then stored in the blockchain, and represents the transaction, ensuring it cannot be altered.

For example, hashing the message:

`A sends 3 coins to B`

might produce the hash:

`464982b640260c16e9c5938e6f799fd655bc29ae415135bda51021632148e7a4`

Even the slightest change in the message (e.g., `A sends 3.1 coins to B`) would generate a completely different hash:

`569d1670830b7cd00968c52ebc678574974eb2bc6a7e304fb786231cfb2d1a3f`

Cryptographic hashing ensures that transactions recorded in the ledger remain immutable.

## Commitment Schemes: Locking Secrets Before Revealing

Alright, so now that we've got a solid grasp of cryptographic hash functions and how they ensure data integrity, let's talk about another interesting cryptographic trick: **commitment schemes**. These are like digital envelopes - you lock something inside them, and no one can peek until you decide to open it.

### What's a Commitment Scheme?

Imagine you and a friend want to make a bet about the outcome of a sports game. You want to write down your prediction before the game starts so that you can prove you were right later - but you also don't want your friend to see it in advance. This is where a commitment scheme comes in handy!

A commitment scheme lets you:

1. **Commit** to a value: You choose a value (e.g., "Team A will win"), and you create a cryptographic commitment to it, essentially locking it in a secure way.
    
2. **Reveal** the value later: Once the game is over, you can show your original prediction, and your friend can verify that you didn't change it.

### The Two Key Properties of a Commitment Scheme

For this to be useful, a commitment scheme needs two important properties:

- **Binding:** Once you commit to a value, you can't change it later. It's like sealing a prediction in an envelope - if you try to swap out the paper inside, people will know something's up.
    
- **Hiding:** No one should be able to figure out what you committed to until you reveal it. The envelope is opaque until you decide to open it.

### How Does This Work in Cryptography?

Commitment schemes often use hash functions to ensure both binding and hiding. A simple way to commit to a value is to hash it along with some extra secret randomness (called a _nonce_ or _salt_). This randomness ensures that even if two people commit to the same value, their commitments look completely different.

For example:

- You commit to "Team A wins" by hashing:
    
  `commitment = hash("Team A wins" + random_value)`
    
- Later, when you reveal your prediction, you show both the original message ("Team A wins") and the random value you used.
    
- Your friend can now verify your commitment by hashing your revealed value and checking that it matches your original commitment.

In the next chapter, we will explore the different type of methods that can be used to encrypt and decrypt messages, as well as how it allows to identify users and to authenticate them when building a blockchain.

---

# 0.2 Symmetric vs Asymmetric Encryption

Now that we have presented hash functions, we can begin understanding where the crypto from cryptocurrency comes from.

Let's suppose we are a group of people that do not know each other, and we want to create a ledger to track transactions between one another. First, we need a way to identify one another: at least the ledger should contain the basic informations for each transaction:

`A sends 3 coins to B`

Here "A" and "B" are identifiers for our users. But we also want a way to authenticate our messages: such a transaction should only be submittable by A and only by A.

One way we could do this is simply by having a central actor keeping the ledger. He could have a list of usernames and password in a database. A user would then be able make a transaction by sending its username and password along with the transaction to the central actor, to authenticate the transaction. For instance:

`A sends to central actor the message: "A sends 3 coins to B; Password of A".`

This is a (over-)simplification of how banks actually work. The problem with this strategy is it introduces a central actor that you need to trust: not to introduce forged transaction to the ledger, not to censor users he does not want to be able to transact with others. To understand how we came up with a trust-less approach, we need to turn to cryptography.

Cryptography, the science of securing communication, has been around for thousands of years. From ancient ciphers used by the Greeks and Romans to modern encryption algorithms that protect data on the internet, cryptography has always been about ensuring privacy of communications and authenticity of the origin of messages.

A simple method of securing messages is symmetric encryption. In this system, both the sender and the recipient share the same secret key. This key is used both to encrypt and decrypt messages, ensuring that only authorized parties can read them.

An example of symmetric encryption is the **Caesar cipher**, one of the earliest encryption methods. In the Caesar cipher, each letter in a message is shifted by a fixed number of places in the alphabet. For example, shifting by three places, "HELLO" becomes "KHOOR." While simple, this method is vulnerable to attacks as the key (the shift value) is easy to guess. Modern symmetric encryption methods, such as AES (Advanced Encryption Standard), are much more secure.

However, symmetric encryption has a major limitation: you have to send the key to the person who will be able to decrypt the message you want to send. If he's across the internet for instance, there is really no way to use asymmetric encryption exclusively to secure your communications.

To overcome the limitations of symmetric encryption, we use asymmetric encryption. Instead of a single shared key, each participant has a pair of keys: a public key and a private key. The public key can be freely shared, while the private key remains secret. If someone wants to send a secure message to a recipient, they encrypt it using the recipient's public key. Only the recipient, with their private key, can decrypt and read the message.

An example of asymmetric encryption is **RSA (Rivest-Shamir-Adleman)**. In RSA, a message encrypted with a recipient's public key can only be decrypted with their corresponding private key. This method is widely used in securing online communications, including SSL/TLS protocols for web security, and is not in the scope of this course.

In the next chapter, we'll describe another asymmetric cryptography scheme that will help solve the identification/authentication problem on decentralized ledger of transactions.

---

# 0.3 Elliptic Curve Cryptography

To create a decentralized ledger, we need a way for users to authenticate their transactions without relying on a central authority. This is where **Elliptic Curve Cryptography (ECC)** comes in.

Elliptic Curve Cryptography is a form of asymmetric cryptography that provides the same security as traditional methods like RSA but with much smaller key sizes. This efficiency makes it well-suited for use in blockchain and cryptocurrencies.

ECC is based on mathematical structures called **elliptic curves**, which follow an equation of the form:

y²=x³+a⋅x+b  (mod p)

When we consider this equation in the domain of [finite fields](https://www.youtube.com/watch?v=uxZAZ4T05wQ&list=PLE8WtfrsTAinMMyQkK_CzXhXU_LHRNXy_&index=1) of the form Z/pZ where p is a prime number, the set of solutions forms a group, where addition is defined as the operation of adding points on the curve according to specific rules.

In ECC, addition of two points P and Q on the elliptic curve is defined geometrically:

- If P≠Q, draw a line through P and Q. This line will intersect the curve at a third point R. The reflection of R over the x-axis gives the sum P+Q.
    
- If P=Q, draw the tangent line at P. This line intersects the curve at a new point, which is reflected to get 2P.
    
- If the line is vertical, the result is the identity element (point at infinity).

![Elliptic Curve Point Addition](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252FizuVrOb5I6YDxdoMPX74%252Fimage.png%3Falt%3Dmedia%26token%3D6c62cb81-500b-47b9-99c3-4fcf525b87d7&width=768&dpr=4&quality=100&sign=6cd98436&sv=2)

A key operation in ECC is **scalar multiplication**, where a point P is added to itself repeatedly:

k⋅P=P+P+...+P   (k times)

These curves sometimes have unique properties that make cryptographic operations secure. One of the fundamental concepts behind ECC is the **Elliptic Curve Discrete Logarithm Problem (ECDLP).**

The idea is the following. Let's say we chose the elliptic curve parameters so the group of points on the curve has a cyclic sub-group of order q, a high prime, with generator G.

If someone takes a random scalar in [0, q-1] uniformly, and computes:

A=a⋅G

Then the ECDLP is said to be "hard" if it is difficult to find a given any A in a short amount of time. Basically the only solution would have to be to check every possible a. Although it remains "easy" to compute A from a. In fact, parameters of ECC curves are also chosen to make computing a multiplication operation as fast as possible.

This is exactly how an asymmetric key pair is generated:

- Generate a random scalar a∈[1,q−1]
    
- Compute A=a⋅G

a is the private key, and A the public key. On the chain, you are identified as A. You can broadcast your public key so other users can encrypt messages using A that only you can decrypt. Let's see how this work in practice:

## Encryption

B wants to send a message to A. The message is encoded as a single field element: m∈[0,p−1].

1. B generate a random r∈[1,q−1]. It should be new and never be used again.
    
2. B computes from r a "nonce" group element: N=r⋅G.
    
3. B then generates a view key group element: E=r⋅A.
    
4. B hashes the view key to get a field element: h=hash(E)∈[0,p−1].
    
5. B calculates the cipher text: c=m+h.

(c,N) is the encrypted message that can be shared with A through an insecure channel.

## Decryption

A receives (c,N), and recovers the original message m following those steps;

1. A computes a⋅N, which is the view key group element E because: a⋅N=a⋅(r⋅G)=(ar)⋅G=(ra)⋅G=r⋅(a⋅G)=r⋅A=E.
    
2. A hashes the view key to get back the field element: h=hash(E).
    
3. A computes the plain text message: m=c−h.

Let's see in next chapter how your private key k allows you to authenticate the transaction you want to broadcast to the network to spend your coins.

---

# 0.4 Cryptographic Signatures

Now that we understand how elliptic curve cryptography works, we can see how **cryptographic signatures** allow users to securely authenticate transactions on a decentralized ledger.

A cryptographic signature is a mathematical proof that a specific message was signed by the owner of a private key. Unlike a traditional signature, a cryptographic signature cannot be forged, and anyone can verify its authenticity.

Let's revisit our transaction example:

`A sends 3 coins to B`

Instead of sending a password or relying on a central authority, A generates a **digital signature** using their private key. The signed transaction might look like this:

`m = "A sends 3 coins to B"; Signature of m by A`

Anyone can verify this signature using A's **public key**. If the signature is valid, it proves that A authorized the transaction. If the signature is invalid, the transaction is rejected.

## Schnorr Signature

One of the most widely used applications of ECC in cryptocurrency is **Schnorr Signature**, which allows users to sign transactions securely. Here's how it works:

1. A user generates a private key a and computes the corresponding public key A=a⋅G.
    
2. When signing a message m (such as a transaction), the user creates a unique digital signature (R,s) using his private key a.
    
   1. A generates a random scalar r∈[1,q−1]. It should be new and never be used again.
       
   2. A computes R=r⋅G.
       
   3. A gets e=h(A|R|m), with (A|R|m) the concatenation of bit representation of those elements.
       
   4. A can then compute s=r+ea.
       
   5. The final signature is (R,s).
       
3. Anyone can verify the signature (R,s) of message m using the public key A:
    
   1. The verifier computes e=h(A|R|m),
       
   2. This enables the verifier to check the message by comparing s⋅G−R and e⋅A. If they are equal, the signature is valid, else it's invalid. Because if A chose s=r+ea at step 2.d, then: s⋅G−R=(r+ea)⋅G−R=r⋅G+e⋅(a⋅G)−R=e⋅A
       

This enforces that only the owner of the private key could have authorized the transaction, making Schnorr signature the key component of decentralized authentication. Of course a lot of extra security consideration are to be taken care of, but this gives a good general idea of how signing works.

In next chapter, we'll explain how to compute a field element from the message in a secured manner, by explaining how hash functions work.

---

# 0.5 Blockchain: Bitcoin

Now that we understand cryptographic signatures and hashing, we can explore how these concepts come together to form Bitcoin, the first decentralized digital currency.

Bitcoin was introduced in 2009 by an anonymous entity known as **Satoshi Nakamoto**. The idea was simple yet revolutionary: a decentralized, peer-to-peer electronic cash system that allows users to transfer value without relying on a trusted third party, such as a bank. To achieve this, Bitcoin uses cryptographic techniques to ensure the security and integrity of transactions.

## How Bitcoin Works

Bitcoin operates on a **blockchain**, a distributed ledger that records all transactions in a public and immutable way. Each transaction is broadcast to a network of computers (nodes), where it is verified, grouped into blocks, and added to the chain.

A Bitcoin transaction consists of the following:

1. **Sender and Receiver Addresses** – Bitcoin addresses are derived from public keys using a cryptographic hashing function. When sending Bitcoin, a user references their address (public key hash) and specifies the recipient's address.
    
2. **Transaction Inputs and Outputs** – As we'll see in next chapter, bitcoin transactions use an **input-output** model. Inputs reference previous transactions (unspent outputs), and outputs define new recipients.
    
3. **Digital Signatures** – The sender signs the transaction with their private key, proving ownership of the Bitcoin being spent. The signature is verified using the sender's public key.
    
4. **Transaction Fee** – A small fee incentivizes miners to include the transaction in a block.

## Mining and Proof of Work

Bitcoin relies on a **Proof of Work (PoW)** consensus mechanism to validate transactions and secure the network. Miners compete to solve complex mathematical problems using computational power. The first miner to solve the problem gets to add a new block to the blockchain and is rewarded with newly minted Bitcoin and transaction fees.

Here's how mining works:

1. Miners collect unconfirmed transactions from the network.
    
2. They bundle these transactions into a candidate block.
    
3. They attempt to find a **nonce** (a random number) that, when hashed with the block data using SHA-256, produces a hash below a predefined difficulty target, for instance ending with 30 zeros. which happens with probability p=2^(-30) since we can assimilate output from cryptographic hash functions to random oracles.
    
4. If a miner succeeds, the block is broadcast to the network and added to the blockchain.
    
5. The miner receives a **block reward**, which halves every four years in an event called the **halving**.

Unlike fiat currencies, which can be printed at will, Bitcoin has a fixed supply of **21 million coins**. This scarcity is enforced through the halving process, which reduces the mining reward every 210,000 blocks (approximately every four years). The predictable issuance schedule makes Bitcoin a deflationary asset.

Bitcoin's security relies on decentralization. No single entity controls the network. Instead, thousands of nodes maintain copies of the blockchain and validate transactions independently. This prevents fraud, such as double-spending, where the same Bitcoin is spent twice.

Bitcoin introduced the world to the first trustless, decentralized financial system. By combining **public-key cryptography, cryptographic hashing, and Proof of Work**, Bitcoin creates a secure and tamper-resistant ledger.

If you want to learn about bitcoin in more depth, I'd advise starting with [this video](https://www.youtube.com/watch?v=bBC-nXj3Ng4) from 3Blue1Brown. In the next chapter, we will explore Bitcoin's UTXO model.

---

# 0.6 UTXO based model

Now that we understand how Bitcoin transactions work, we need to explore the fundamental concept that powers Bitcoin's accounting system: **UTXOs (Unspent Transaction Outputs).** Unlike traditional banking systems that maintain account balances, Bitcoin operates using a **UTXO model**, which keeps track of spendable outputs from previous transactions.

## What Are UTXOs?

A UTXO is a **unit of Bitcoin that has been received but not yet spent**. Each Bitcoin transaction consists of **inputs and outputs**, and the outputs that remain unspent become UTXOs. These UTXOs act as the available balance for a Bitcoin address.

To understand UTXOs, consider the following example:

1. Alice receives **0.5 BTC** from Bob.
    
2. Alice receives **0.8 BTC** from Charlie.
    
3. Alice now has **two UTXOs** totaling **1.3 BTC**.

When Alice wants to send **0.7 BTC** to Dave:

- She must use one or more UTXOs as inputs.
    
- Suppose she chooses the **0.8 BTC UTXO**.
    
- The transaction will generate two outputs:
    
  - **0.7 BTC to Dave** (spent output).
      
  - **0.1 BTC in change** back to Alice (new UTXO).

This process ensures that no partial UTXOs exist - each transaction consumes full UTXOs and creates new ones as change.

## How UTXOs Prevent Double-Spending

Bitcoin nodes maintain a **UTXO set**, a global database of all unspent transaction outputs. Before confirming a transaction, nodes check whether the inputs being used are still unspent in the UTXO set. If an input has already been spent in a previous transaction, the transaction is rejected. This mechanism ensures **double-spending** is impossible.

## Challenges of the UTXO Model

Despite its advantages, the UTXO model also introduces challenges:

- Managing UTXOs requires users and wallets to track individual outputs instead of a single balance.
    
- Since each input references a previous UTXO, transactions can become large when multiple inputs are needed.
    
- Small leftover UTXOs, often called "dust," can accumulate over time and become uneconomical to spend due to transaction fees.

UTXOs are the foundation of Bitcoin's transaction model. Unlike traditional account-based systems, Bitcoin's UTXO model offers security, decentralization, and privacy benefits while preventing double-spending. However, it also requires careful management of outputs and transaction fees.

In the next chapter, we will explore how Bitcoin's **wallets** manage UTXOs and how fee optimization strategies help users minimize transaction costs.

![UTXO Model](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252FW166tfMvrNuHyoybU6zO%252Fimage.png%3Falt%3Dmedia%26token%3D7402cb5e-c898-41f4-b8ad-26aff69196ab&width=768&dpr=4&quality=100&sign=c762cb4c&sv=2)

---

# 0.7 Programmability: Ethereum

While Bitcoin introduced the world to decentralized digital currency, **Ethereum** expanded the concept by creating a platform for **programmable smart contracts** and **decentralized applications (dApps)**. Proposed by **Vitalik Buterin** in 2013 and launched in 2015, Ethereum goes beyond simple transactions, enabling complex computational logic on the blockchain.

## The Ethereum Virtual Machine (EVM)

Ethereum's core innovation is the **Ethereum Virtual Machine (EVM)**, a decentralized computational engine that allows developers to execute smart contracts. Smart contracts are self-executing programs stored on the blockchain that run when predefined conditions are met. These contracts eliminate intermediaries, making processes more transparent and efficient.

## Gas and Transaction Fees

Ethereum introduces gas as a unit of computational effort necessary for executing transactions and smart contracts. Users pay gas fees to incentivize network participants - whether miners in Proof of Work (PoW) or validators in Proof of Stake (PoS) - to process their transactions. The amount of gas required depends on multiple factors, including the complexity of computation. More intricate smart contract operations consume more gas. Additionally, network congestion impacts fees, as higher demand leads to increased costs due to users competing for limited block space.

## Ethereum's Transition to Proof of Stake (PoS)

Originally, Ethereum employed a Proof of Work (PoW) consensus mechanism similar to Bitcoin. However, to enhance scalability and reduce energy consumption, Ethereum transitioned to Proof of Stake (PoS) through the Ethereum 2.0 upgrade. Under PoS, validators replace miners. Their selection is based on the amount of ETH they stake, ensuring a more energy-efficient and secure network. Validators confirm transactions, propose new blocks, and earn rewards for their participation. This shift significantly decreases energy use while improving overall network performance.

## Smart Contracts and Decentralized Applications (dApps)

Ethereum's programmability supports the development of decentralized applications (dApps), which function autonomously on the blockchain without centralized oversight. These applications span multiple industries and offer diverse functionalities:

Decentralized Finance (DeFi) platforms, such as Uniswap and Aave, empower users to trade, lend, and borrow digital assets without intermediaries. The rise of Non-Fungible Tokens (NFTs) has transformed digital ownership, allowing unique representations of art, collectibles, and virtual goods on the blockchain. Additionally, Decentralized Autonomous Organizations (DAOs) redefine governance by enabling token holders to vote on proposals, ensuring transparent decision-making within communities.

Ethereum has revolutionized blockchain technology by introducing smart contracts and enabling a decentralized ecosystem. With its transition to Proof of Stake, Ethereum continues to change through specific gouvernance mechanisms, aiming to achieve greater scalability and sustainability.

In the next chapter we'll look further into the mechanics of account based model.

![Ethereum Architecture](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252F6B9I8ipQ7fnws4OJqG1O%252Fimage.png%3Falt%3Dmedia%26token%3Dd70e6722-574e-4e2c-a85f-3a1928698cce&width=768&dpr=4&quality=100&sign=f9b27c34&sv=2)

---

# 0.8 Account based model

## Ethereum's Account-Based Model

Ethereum operates on an account-based model, distinguishing itself from what we've seen before with Bitcoin's UTXO model. This system resembles traditional banking, where accounts hold balances and facilitate transactions. There are two main types of accounts:

Externally Owned Accounts (EOAs) function similarly to personal bank accounts and are controlled by private keys. Users rely on EOAs to send transactions and interact with smart contracts. Contract Accounts, on the other hand, store smart contract code and execute predefined logic when triggered by EOAs or other contracts.

Every account maintains a balance of Ether (ETH), Ethereum's native cryptocurrency. Transactions between accounts alter the blockchain state, enabling seamless interactions within the network.

Ethereum's account-based model operates by maintaining global account states that update upon every transaction. Unlike Bitcoin's UTXO model, which tracks individual unspent outputs, Ethereum records the latest state of each account, allowing for efficient transaction execution and smart contract interactions.

## How Accounts Store Data

Each Ethereum account consists of:

- A unique address that identifies the account on the network.
    
- A balance reflecting the amount of Ether held.
    
- A nonce, which counts the number of transactions sent to prevent replay attacks.
    
- Storage (for contract accounts only) containing smart contract data.

## Benefits of the Account-Based Model

One key advantage of Ethereum's model is its simplicity in handling transactions. Since accounts maintain a direct record of balances, smart contracts can operate more fluidly compared to Bitcoin's UTXO model, which requires tracking multiple inputs and outputs. This streamlined approach enhances programmability, making Ethereum the preferred platform for decentralized applications.

Additionally, Ethereum's account model enables better composability between smart contracts. Multiple contracts can interact within a single transaction, allowing for complex financial operations such as DeFi lending and multi-signature wallets.

## Security and Challenges

Despite its efficiency, the account-based model introduces certain security risks. Since balances are mutable, accounts are more vulnerable to state-based attacks, such as reentrancy attacks. Developers must implement rigorous security measures, such as checks-effects-interactions patterns, to mitigate these risks.

Another challenge is scalability. With every transaction modifying the global state, network congestion can increase, leading to higher gas fees and slower transaction times. Ethereum's ongoing upgrades, including Layer 2 scaling solutions, aim to address these limitations by optimizing state management and transaction throughput.

In the next chapter, we will get into the key tools mathematicians came up with to introduce privacy to decentralised ledgers.

---

# 0.9 ZKP and zkSNARK

Up until here in the course, all the blockchain we presented relied on the whole data of the ledger being public and accessible to anyone. It allows enforcing integrity of the data, enabling anyone to audit the blockchain and double check that every transaction is correct.

Modern cryptographic technologies allow to envision a different way, using methods allowing users to prove statements about their data without revealing the data itself. This need for privacy-preserving proofs is where Zero-Knowledge Proofs (ZKPs) come into play.

Zero-Knowledge Proofs allow one party, the prover, to convince another party, the verifier, that a certain statement is true without revealing any additional information beyond the validity of the statement. The theoretical foundation of ZKPs relies on three core properties:

1. **Completeness**: If the statement is true and both parties follow the protocol honestly, the verifier will be convinced.
    
2. **Soundness**: If the statement is false, no dishonest prover can convince the verifier that it is true, except with some small probability.
    
3. **Zero-Knowledge**: If the statement is true, the verifier learns nothing other than the fact that the statement is true.

## Interactive proofs

To understand how these proofs operate in practice, let's look at Interactive Proofs (IP). An interactive proof system involves multiple rounds of communication between the prover and verifier. In each round, the verifier sends a challenge to the prover, who responds with evidence supporting the claim. The process continues until the verifier is satisfied with the provided evidence.

### Example:

Let's see how a prover can show to a verifier he knows a x∈N such that: y=x² (mod n)

#### Protocol

Proof protocol would consist of multiple rounds. Here is how a single round of proving would go:

1. Prover chooses a random r∈[1,n] such that gcd(r,n)=1.
    
2. Prover sends s=r² (mod n) to V.
    
3. Verifier flips a coin, choosing a random b∈{0,1}, and sends b to Prover.
    
4. Prover computes z=rx^b (mod n), (which he can because he knows x) and sends it to Verifier. Meaning that, depending on the coin flip: z=rx or z=r (mod n).
    
5. Verifier then computes sy^b (mod n) and checks if it is equal to z².

This protocol is repeated for K consecutive rounds, if for every round the verifier's equality check is valid, then the verifiers accepts the proof.

#### Completeness

If both parties are honest, and execute the protocol correctly, then for every round: y=x² (mod n) ⟹ sy^b = r²x²^b=(rx^b)²=z² (mod n).

Therefore, if both parties are honest, execute the protocol correctly and the statement is true, then the verifier will accept all K rounds, hence the final proof. And this is exaclty how completeness is defined above.

#### Soundness

If a malicious prover doesn't have a valid x satisfying the statement, let's find out what would be his possibilities to try to cheat the protocol.

During a single round, instead of generating a random r∈[1,n], the malicious prover could dishonestly send a specific value as s, then depending on b, a malicious z, that would enable him to pass verifier's ultimate check: sy^b = z² (mod n).

- **Case** b=0: The final check would then be s = z² (mod n). Hence, to cheat the verifier in case he knows b will be 0, the prover would have to chose any r∈[1,n], send at step 1 s=r², and at step 4 z=r.
    
- **Case** b=1: The final check would then be sy = z² (mod n). Hence, to cheat the verifier in case he knows b will be 1, the prover would have to chose any r∈[1,n], send at step 1 s=r²y^(-1), and at step 4 z=r.

But since the prover doesn't know in advance b, in particular before sending a value for s at step 2, he can only prepare cheating for one of those two cases. He can either chose to:

- S**end** s=r²
    
  - Then if b=0, the prover can cheat the verifier by sending z=r.
      
  - But if b=1, the verifier will compute for final check sy^b = sy=r²y (mod n) and compare it to z². So to cheat him, the prover would have to send z such that z²=r²y=r²x²=(rx)². Hence necessarily, z=rx (mod n), which the prover could only compute by knowing x itself.
    
- **Send** s=r²y^(-1)
    
  - Then if b=1, the prover can cheat the verifier by sending z=r.
      
  - But if b=0, the verifier will compute for final check sy^b = s=r²y^(-1) (mod n) and compare it to z². So to cheat him, the prover would have to send z such that z²=r²y^(-1)=r²(x²)^(-1)=(rx^(-1))². Hence necessarily, z=rx^(-1) (mod n), which the prover could only compute by knowing x^(-1), and therefore x itself.

Hence, in any case, if the verifiers draws b∈{0,1} uniformly, after sending one of these two s value, the prover would only have a probability p=1/2 of successfully cheating the verifier during a single round.

Therefore the probability of cheating the verifier for K consecutive rounds would be 1/2^k. This means that by choosing a high enough amount of rounds K, the verifier can ensure with a probability as closed to 1 as he wants that the prover is not cheating, since lim_k (1-1/2^k)=1.

#### Zero knowledge

By choosing a uniformly random value r∈[1,n], a prover does not disclose any information about x by sending, r², rx nor r which are the only values disclosed if the protocol is correctly executed by the prover.

## Non-Interactive proofs

The problem with non-interactive proof is there lack practicality, in particular in a blockchain environment, where constant back-and-forth communication is infeasible as we cannot expect every users to remain live during proofs. It also causes a communication overhead which can be a bottleneck for scaling. Fortunately, a prominent class of zero-knowledge proofs, known as zk-SNARKs (Zero-Knowledge Succinct Non-Interactive Arguments of Knowledge), transforms the interactive process into a non-interactive one.

Usually turning an interactive proof protocol into a non-interactive one is done using a technique called Fiat-Shamir transform. The idea behind it is to make the prover to simulate the behavior of the verifier by generating challenges that are proven to be random, for instance by using a hash function to generate the challenges, with the data of the statement along with other data generated during a

A zk-SNARK consists of three key phases:

1. **Setup**: A trusted setup phase generates public parameters used for both proving and verification. This phase often involves complex cryptographic operations, including elliptic curve pairings.
    
2. **Proving**: The prover uses these public parameters and a private witness (the secret data) to generate a proof. This proof convinces the verifier of the statement's validity without revealing the witness.
    
3. **Verification**: The verifier, using the proof and the public parameters, checks the proof's validity in a computationally efficient manner.

The mathematical backbone of zk-SNARKs relies on quadratic arithmetic programs (QAPs), which encode the computation as a set of polynomial equations. These equations are then transformed into cryptographic commitments using homomorphic properties, ensuring that operations on encrypted data correspond to operations on the plaintext.

Consider a function f that we want to prove has a certain output y without revealing x. The prover constructs a polynomial that represents the computation of f(x)=y. Through a series of cryptographic transformations, this polynomial is committed to in a way that the verifier can check the correctness of y without gaining any information about x.

The succinctness of zk-SNARKs is another important feature. The proof size and verification time are both significantly smaller compared to traditional proof systems, making them ideal for applications with resource constraints, such as smart contracts on blockchains.

The implementation of zk-SNARKs also involves elliptic curve cryptography, particularly pairing-friendly curves like BN254. These curves support efficient bilinear pairings, which are critical for the construction of zk-SNARK proofs.

One real-world application of zk-SNARKs is in privacy-focused cryptocurrencies like Zcash. In Zcash, zk-SNARKs enable shielded transactions where the sender, receiver, and transaction amount remain confidential while still ensuring the transaction's validity and preventing double-spending.