# Module 1: The Aleo Blockchain

The Aleo blockchain represents the next generation of blockchain technology, focusing on privacy and programmability. After exploring the fundamentals of cryptography and blockchain in the previous module, we now turn our attention to Aleo's unique architecture and features.

This module covers:

- [1.1 Aleo Fundamentals](https://zlearn.gitbook.io/zlearn/the-aleo-blockchain/1.1-aleo-fundamentals) - Core concepts and principles behind Aleo's design
- [1.2 Accounts](https://zlearn.gitbook.io/zlearn/the-aleo-blockchain/1.2-accounts) - Understanding Aleo's account system
- [1.3 Concepts](https://zlearn.gitbook.io/zlearn/the-aleo-blockchain/1.3-concepts) - Key concepts specific to the Aleo ecosystem
- [1.4 Consensus - AleoBFT](https://zlearn.gitbook.io/zlearn/the-aleo-blockchain/1.4-consensus-aleobft) - The consensus mechanism powering Aleo

Through this module, you'll gain a comprehensive understanding of how Aleo leverages zero-knowledge proofs to enable private, programmable transactions on a decentralized blockchain.

---

# 1.1 Aleo Fundamentals

Aleo is about building a ledger with permissionless private programmability, which really just means you can execute arbitrary programs on the blockchain without asking for permission and include private data.

Aleo includes two key pieces to achieve this: SnarkVM and SnarkOS.

SnarkVM is where the magic happens. It's Aleo's virtual machine, letting you run programs and generate zero-knowledge proofs off-chain of the execution of those programs on some inputs. Think of it as a private execution sandbox: you do all your computations here, produce a proof that you did them correctly, and then share just that proof instead of all the details. That's how Aleo keeps things private while still making sure everything checks out.

SnarkOS is what keeps the network running. It's the blockchain node software that powers Aleo's consensus mechanism, called AleoBFT. This ensures that all network participants agree on the state of the chain securely and efficiently. Once a program has been executed offchain by a user, its proof is shared to the network and verified by peers running SnarkOS (that calls SnarkVM proof verification under the hood).

Put these two together, and you've got a system where private transactions and smart contracts execute freely, without central oversight.

## From Programs to ZKPs

So, how do we take a regular program, some inputs and turn it into a ZK proof of the correct execution? Here's how it works, from 5000kms:

- First, we write some code that represents the statement we want to prove. For example, proving someone is over 18 without revealing their actual age. The code takes an input, runs a check, and gives a true or false result.
    
- The program is transformed into a polynomial represented as its linear factors and the quotient polynomial, following a procedure not explained in detail here but in the vein of:
    
  - The program gets broken down into basic logic gates (AND, OR, etc.).
      
  - Each logic gate becomes a constraint that must be met for the proof to be valid. Each constraint in an R1CS is an equation, in the base field, of the form:
      
    (a₀+a₁x₁+...+aₙxₙ)(b₀+b₁y₁+...+bₘyₘ)=(c₀+c₁z₁+...+cₗzₗ)
      
  - These constraints get simplified into one unique mathematical equation, that can be verified very quickly.
      

Following this process, Aleo lets you run private programs that anyone can verify - without exposing sensitive data. That's how we enable powerful, scalable, and private blockchain applications.

So what makes Aleo permissionless, private, and programmable? It all comes down to architecture. The system is built so that anyone can join, execute logic privately, and broadcast results to the network. No one needs approval, and no one has to expose sensitive data.

Let's break this down further.

## Components in Play

**User** Users interact with the network. They can check their on-chain state, whether public or private, by querying an Aleo client node. When executing a transaction, they generate a zero-knowledge proof that confirms correctness without revealing sensitive data. For efficiency, they can even delegate proof generation to a third-party proving provider (some wallets offer this service) to speed up processing.

**Third-Party Prover** This is a specialized service that handles proof generation on behalf of users. It doesn't touch authorization or signing, just the heavy lifting of computing zero-knowledge proofs. Users outsource this part for better performance, while still maintaining security.

![User and Third-Party Prover](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252FKuUTi2uWby11pbEmQFgR%252Fimage.png%3Falt%3Dmedia%26token%3D1e353df3-60d2-4acb-8205-a450b8804721&width=768&dpr=4&quality=100&sign=a5c90dec&sv=2)

## The Aleo Network Architecture

The Aleo blockchain ecosystem is built on two core network layers: the consensus network and the peer-to-peer (P2P) network. Within this framework, three key types of nodes operate: Validators, Provers, and Clients.

**Validator:** Validators are critical participants in both the consensus and P2P networks. They typically run on port 5000 for consensus and port 4130 for P2P operations. Validators communicate with each other over port 5000 to maintain network consensus. Clients and Provers, however, are not permitted to interact directly with the consensus network.

The primary job of a Validator node is to follow AleoBFT rules to generate new blocks and maintain network integrity. Within the Aleo P2P network, Clients and Provers interact with each other and connect to select Validators to stay updated with the latest blocks.

**Client:** Clients play a key role by syncing blocks from the consensus network and updating the ledger. Through the Client's RPC, users can access real-time ledger data and broadcast transactions. Once transactions are included in a new block, they are officially confirmed and finalized on the network.

**Prover:** Prover nodes sync CoinbasePuzzles created by the consensus network. They run algorithms to solve these puzzles according to set criteria. Once a Prover finds a valid solution, it broadcasts it across the P2P network, where it is eventually submitted to the consensus network. If the solution is accepted and added to a block, the Prover receives a CoinbaseReward as an incentive for contributing to network security and computation.

![Aleo Network Architecture](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252FS0lWjmpMm25kpY5LrX8F%252Fimage.png%3Falt%3Dmedia%26token%3D878131df-795f-4a62-b04d-780704a7de8b&width=768&dpr=4&quality=100&sign=5b969cdc&sv=2)

## Executing a Private Transaction

1. The user prepares the required inputs.
    
2. Fetches the relevant program from a node.
    
3. Runs the necessary function locally using SnarkVM.
    
4. Generates an output proof and broadcasts it, along with encrypted inputs and outputs, to a client node.
    
5. The validator verifies the proof, and consensus is reached through AleoBFT.
    
6. If all checks pass, the transaction is included in a new block on the ledger.

And that's it, the transaction is finalized, and privacy remains intact.

This process is what allows Aleo to offer true private programmability at scale. The heavy computations happen off-chain, the network only sees and validates proofs, and users stay in control of their data. It's a system designed for both security and usability, and as we move forward, we'll explore how to build and deploy real-world applications using these principles.

---

# 1.2 Accounts

This is where the previous chapter 0.3 on Elliptic Curve Cryptography will be super useful, make sure you have it in mind.

## Aleo Fundamental Objects

Let's start with defining some useful mathematical objects.

### Fields

We'll start with the most fundamental object. Since the underlying constraint system of Aleo is R1CS, programs are essentially represented as sets of linear equations over a field. This makes elements of this field very special. They are simply called fields, for short, and represent elements of base field of the elliptic curve that Aleo accounts are based upon.

Every other types that are available on Aleo are derived from this one, in order for the instructions that involve those extra types to be reduced to linear equations over the base field.

This means that using field directly is generally the most optimal way to write a program constraint-wise.

Fields are the elements of Z/pZ with:

p=8444461749428370424248824938781546531375899335154063827935233455917409239041

### Group

For reference here are the elliptic curves used in Aleo:

**Edwards BLS12**

**BLS12-377**

**Curve Type**

Twisted Edwards

Barreto-Lynn-Scott

**Scalar Field Size**

251 bits

253 bits

**Base Field Size**

253 bits

377 bits

**G1 Compressed Size***

32 bytes

48 bytes

**G2 Compressed Size***

N/A

96 bytes

Group elements represent points from a specific sub-group of the group of points on the **Edwards BLS12** elliptic curve. This sub-group is generated by a generator point noted **G1.**

### Scalar

The sub-group described above has order:

q=2111115437357092606062206234695386632838870926408408195193685246394721360383

This means that when adding the generator G with itself successively: G+G+...+G, by definition, we end up reaching every element of the group, 1⋅G,2⋅G,... until reaching q⋅G=0. This set of integer by which we can multiply G is hence simply the field Z/qZ, called the set of scalar.

## Accounts

### Traditional Accounts

On traditional blockchains accounts can be simply made the following way, once an appropriate elliptic curve has been chosen, with fast scalar multiplication and hard ECDLP:

- User generates a uniformly random 256 bits number s.
    
- It can be encoded in a human friendly way, by representing it a sequence of 24 words taken from a list of 2048 possible words. Since 2048^24=(2^11)^24=2^264 we can store 264 bits of information using that sentence of words (called seed phrase).
    
- User then carefully saves that sequence. Any number of accounts (a_i,A_i) can be generated with it:
    
  - Generate private key a_i=hash_to_salar(s|i)
      
  - Generate address A_i=a_i⋅G
      

### Aleo Accounts

On Aleo things are done a bit differently in order to include privacy features. Instead of having just a private key and an address, accounts also have two extra keys called the View Key and Compute Key.

**Private Key:** The private key's, as with traditional chains, is used to authenticate transactions.

**Address:** The address serves as an identifier for users accounts. If you want to a user to send you some assets, you simply give him you address. Although, with just your address, this user will not be able to see all your past private transactions and the private state that involves your account.

**View Key:** The view key is used to decrypt all the private state on chain owned by your account. With it you can discover all your private holdings and state.

**Compute Key:** The compute key is used for proving a zkVM execution. It can be provided when delegating a transaction proof for instance.

### Account Generation

- User generates a uniformly random field element s (the seed that can be encoded as words as above).
    
- **Private Key:**
    
  User computes a signature secret key (scalar):
    
  - sk_sig = hash_to_scalar(C_1 | s)
      
  and a signature randomizer (scalar):
    
  - r_sig = hash_to_scalar(C_2|s)
      
  Where C_1 and C_2 are constants.
    
- **Compute Key:**
    
  User computes signature public key (group):
    
  - pk_sig = sk_sig ⋅ G 
      
  and a signature public randomizer (group):
    
  - pr_sig = r_sig ⋅ G
      
  and a pseudo random secret key (scalar):
    
  - sk_prf = hash_to_scalar(pk_sig|pr_sig)
      
- **View Key** (scalar):
    
  - vk = k_sig + r_sig + sk_prf
      
- **Address** (group):
    
  - A = vk ⋅ G

![Aleo Account Structure 1](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252FnhgV5kvhUB0sHw0nXwki%252Fimage.png%3Falt%3Dmedia%26token%3D0dfc15ad-2034-41c7-93fd-dc72ccdf1584&width=768&dpr=4&quality=100&sign=4bdc058b&sv=2)

![Aleo Account Structure 2](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252FlHEouuBFKDzNnwQS691z%252Fimage.png%3Falt%3Dmedia%26token%3D9725414f-4b44-491e-b2d8-c492da550c98&width=768&dpr=4&quality=100&sign=730e8656&sv=2)

---

# 1.3 Concepts

Programs, transactions, transitions, blocks. These are the core structures of Aleo, and you'll run into them constantly. So let's break them down.

Programs first. A program is the backbone of Aleo applications. It's both the logic and the state. Think of it like a contract that lives on-chain, defining rules that must be followed. But unlike traditional smart contracts, Aleo programs use zero-knowledge proofs. This means they can execute logic privately while still proving correctness to the network. Aleo uses its own language called Aleo instructions, which is statically typed and designed for writing privacy-preserving applications. You'll write a lot of Aleo programs, so get familiar.

Next, transactions. Transactions move things forward. They publish new programs, execute program functions, and handle fees. There are three types:

**Deploy Transaction**: This is how you publish an Aleo program. Once deployed, it's available on the network for execution.

**Execute Transaction**: This calls a function in an Aleo program. If you want to interact with a deployed contract, you send one of these.

**Fee Transaction**: This pays for rejected transactions. If a transaction fails, the network still needs compensation for the work done.

Every transaction has an ID, computed from its transitions using a Merkle Tree Digest. The ledger keeps track of all these transactions, making sure they follow the rules.

Now, transitions. A transition is what actually changes state on Aleo. You can think of it as a state update inside a transaction. Every transition has inputs and outputs, just like a function. It takes in some data, does something with it, and produces a new state. Transitions can be public or private, and validators process them to update the blockchain. Important fields include:

- **id**: The transition ID, derived from its inputs and outputs.
    
- **program_id**: The program this transition belongs to.
    
- **function_name**: The function being executed.
    
- **inputs & outputs**: Data going in and out of the transition.
    
- **tpk**: A public key to verify the digital signature of the transaction.
    
- **tcm**: The transition commitment, ensuring correctness.

Finally, blocks. A block is just a container for transactions. It groups transactions together, adds cryptographic proofs, and advances the ledger. Every block has a unique hash and points to the previous block, forming the Aleo blockchain. Inside each block, you'll find:

- **Transactions**: A list of included transactions.
    
- **Block Header**: Metadata summarizing the block's state.
    
- **Ratifications**: Prover rewards.
    
- **Coinbase**: The accumulated solution for the coinbase puzzle.
    
- **Signature**: A digital signature proving validity.

The block header itself tracks important things like the Merkle root of transactions, finalize state, ratifications, and proof targets. It also holds metadata like the block's height, network ID, and timestamp. Every new block extends the chain, ensuring Aleo remains secure and up to date.

---

# 1.4 Consensus - AleoBFT

In decentralized ledgers, the consensus mechanism is the fundamental component that enforces trustless verifiable operations. The Aleo network employs the **Aleo Byzantine Fault Tolerance (AleoBFT)** algorithm, a consensus mechanism based on a **Directed Acyclic Graph (DAG)** structure, a directed graph with no directed cycles. AleoBFT takes inspiration from **Bullshark** and **Narwhal**, incorporating modifications that enable dynamic committees of validator and staking.

## Structure of AleoBFT

AleoBFT achieves high transaction throughput and security by integrating **Narwhal's efficient data dissemination** with **Bullshark's streamlined ordering process**. This hybrid architecture enables Aleo to scale effectively while preserving decentralization and preventing blockchain forks, making it a suitable foundation for **privacy-preserving blockchain applications**.

### Narwhal: DAG-Based Mempool Abstraction

Narwhal serves as a **DAG-based mempool abstraction layer** that can function alongside any consensus mechanism. AleoBFT relies on a combination of Narwhal and a partially synchronous implementation of Bullshark. The primary role of Narwhal is to facilitate the transmission of transactions between network nodes while assembling certificates and constructing the DAG.

**DAG Construction**

A DAG consists of:

- **Vertices**, representing transaction certificates.
    
- **Edges**, representing references to certificates from the previous round.

Each certificate has a unique author per round, and validators contribute to the construction of the DAG by forming references across rounds. However, Narwhal alone provides only a causal order of events; final sequencing is performed by Bullshark.

### Bullshark: Total Order Determination

Bullshark determines a **definitive sequence of transactions** by interpreting the DAG structure generated by Narwhal. This ordering process occurs with **zero additional message overhead**, meaning that no extra message exchanges are required between validators. The finalized sequence forms the **blockchain ledger** for the network.

## Operation of Narwhal

Each validator node comprises **multiple worker instances** and a **single primary instance**. The workers distribute transaction batches among validators while generating a digest for the primary instance. The digest serves as a **certificate of availability** for the underlying transaction data. The consensus mechanism operates solely on ordering **block digest certificates**, which are significantly smaller than raw transaction data, thereby improving efficiency.

### DAG Formation Process

The DAG is constructed over multiple rounds. Each round requires **(n - f)** certificates from the previous round, where:

- **n** is the total number of validators,
    
- **f** is the maximum number of Byzantine (faulty) validators.

This ensures that the network commits to values agreed upon by at least **two-thirds** of all validators. The steps involved are:

1. Validators **propose** transactions and broadcast them to the network.
    
2. Other validators **endorse** proposals by signing them and returning the endorsements to the sender.
    
3. Once a validator receives **(n - f)** endorsements, it forms a **certificate of availability** and broadcasts it.
    
4. The validators incorporate this certificate in their next round.
    
5. Validators transition to the subsequent round upon collecting the required number of certificates.

This mechanism ensures that block propagation remains **resilient to network delays** and Byzantine faults.

## Bullshark: Ordering the DAG

Once the DAG is established, Bullshark determines the **total order of transactions** without additional communication. The properties of Narwhal guarantee **non-equivocation**, meaning that:

- If two honest validators obtain a certificate for the same round from the same author, the certificates are **identical**.
    
- The combination of signature verification and graph structure ensures **consistency** across different validator DAGs.

### Leader Selection and Commit Rules

Bullshark employs **deterministic leader selection** every **even-numbered round**. If a leader authors a certificate, it is designated as an **anchor**. The commit rule is as follows:

- If an anchor receives at least **(1 + f)** incoming edges (votes) from odd-round certificates, it is **committed**.
    
- If an anchor lacks sufficient votes, it remains **uncommitted** and may be skipped.
    
- Due to network asynchrony, some validators may recognize an anchor while others do not. However, if an anchor is committed by one validator, **all future anchors will have a reference to it**, ensuring eventual consistency.

## Advantages of AleoBFT over Traditional Consensus

AleoBFT provides several advantages over traditional **leader-based consensus protocols**:

1. **High Throughput**: Traditional consensus mechanisms rely on a single leader, creating a bottleneck. AleoBFT, by contrast, distributes workload across multiple validators.
    
2. **Reduced Overhead**: The use of **block digest certificates** minimizes the amount of data processed during consensus.
    
3. **Resilience to Failures**: The DAG structure encodes all necessary information, eliminating the need for **view-change mechanisms** in case of leader failure.

AleoBFT is a **formally verified, DAG-based consensus protocol** that ensures high security, decentralization, and scalability. By integrating Narwhal and Bullshark, it achieves a balance between **efficient data dissemination** and **optimal transaction sequencing**. These properties make AleoBFT a suitable foundation for **next-generation blockchain applications**, particularly those requiring **privacy-preserving computation**.

![AleoBFT Diagram 1](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252Fp5g3uj2L31JEIneYBGMc%252Fimage.png%3Falt%3Dmedia%26token%3D2af50154-fac3-41ef-9019-546f647263f3&width=768&dpr=4&quality=100&sign=d73a7d57&sv=2)

![AleoBFT Diagram 2](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252Fx58dKMWyM3B2kOtQSX05%252Fimage.png%3Falt%3Dmedia%26token%3D4838cc68-9dc3-4fe4-9b5f-0582d78da08e&width=768&dpr=4&quality=100&sign=ffa77bcb&sv=2)

![AleoBFT Diagram 3](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252FUSvGj4CEGpQbu5y6Zyx9%252Fimage.png%3Falt%3Dmedia%26token%3Df2722b17-2067-418f-9e9e-09c577dd760d&width=768&dpr=4&quality=100&sign=7bfa8367&sv=2)

![AleoBFT Diagram 4](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252FqOwnqF8RDWLOuSBPKYmk%252Fimage.png%3Falt%3Dmedia%26token%3D6a08e0a6-1149-45e7-8389-9a8b716d8c63&width=768&dpr=4&quality=100&sign=3e58afd9&sv=2)