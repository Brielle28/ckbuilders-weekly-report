# How CKB Works

**Source:** [Nervos CKB Docs - How CKB Works](https://docs.nervos.org/docs/getting-started/how-ckb-works)

---

## Intro to CKB

Blockchain is a **decentralized, immutable ledger** that records transactions across a network, enabling interactions without intermediaries.

**CKB (Common Knowledge Base)** is the foundational layer of the Nervos Network. It operates on **Proof of Work (POW) consensus**, where miners solve cryptographic puzzles to secure the network and validate transactions, ensuring strong security and decentralization.

---

## Cell Model

The CKB blockchain works like a **storage facility filled with boxes, called Cells**. These Cells can hold **CKBytes**, the native tokens of CKB, which represent both **value and storage capacity**—the more CKBytes you own, the more data you can store on the blockchain.

### Key Cell Properties

- **Minimum capacity:** Each Cell requires at least **61 CKBytes** to cover the essential data it needs to function
- **Recommended capacity:** In practice, allocate **62 CKBytes or more** to cover transaction fees and ensure smooth processing
- **Immutability:** Once added to the blockchain, Cells are **immutable**—they can't be changed
- **Updating Cells:** To update a Cell's data, you must:
  1. **Consume** it (extract the data)
  2. **Modify** the data
  3. **Create** a new Cell with the updated information

### Live Cells vs Dead Cells

- **Live Cells:** Cells that have not been consumed; available for use in future transactions
- **Dead Cells:** Cells that have been consumed in a transaction

---

## How Transactions Work

### Transaction Flow (6 Steps)

**Example:** Sending 100 CKBytes to Alice from a Live Cell containing 200 CKBytes

1. **Create**
   - Select a Live Cell with 200 CKBytes as input
   - Create two new Live Cells as output:
     - One Cell with **100 CKBytes** → locked to Alice
     - Another Cell with **99.999 CKBytes** → locked to yourself (change)
   - The **0.001 CKBytes** difference is used as the transaction fee

2. **Sign**
   - Sign the transaction with your **private key**
   - Proves you have authority to spend the CKBytes in this Cell

3. **Broadcast**
   - Transaction is broadcasted from your wallet to the **nearest CKB network node**

4. **Validate**
   - The receiving node performs several checks to validate your transaction
   - For more details, check out the [verification process](https://docs.nervos.org/docs/getting-started/how-ckb-works)

5. **Propagate**
   - Validated transaction is propagated to other nodes in the network
   - Enters the **mempool**—a temporary storage area for unconfirmed transactions, waiting for miners to pick up

6. **Confirm**
   - Miners select your transaction from the mempool and include it in a new block
   - Once the block is mined, it's added to the blockchain
   - Your transaction now has **1 confirmation**
   - With each subsequent block, your transaction gains additional confirmations

### Transaction Result

- Your original **200 CKBytes Live Cell is consumed** → marked as a Dead Cell
- The two **new Live Cells** (100 CKBytes for Alice + 99.999 CKBytes for you) are now **active on the blockchain**

---

## Scripts

### What is a Script?

A **Script** in CKB is a **binary executable** that can be executed on-chain. It is a program that runs on a virtual machine powered by the **RISC-V instruction set** (called the CKB-VM) and can perform arbitrary logic to guard and protect your Cells. You can think of it as a **smart contract**.

### Script Components

A Script consists of three parts:

- **`code_hash`:** Identifies the Script code to be loaded into the CKB-VM
- **`hash_type`:** Indicates the method CKB-VM uses to locate the Script code or Script
- **`args`:** Provides specific arguments that differentiate one Script from another, such as a user's public key hash

### Types of Scripts

There are two main types of Scripts:

1. **Lock Script** (required)
   - Controls the **ownership and access** to a Cell
   - Ensures only authorized users can **spend/consume** the Cell

2. **Type Script** (optional)
   - Dictates **how a Cell can be used or modified** in a transaction

---

## CKB System Script

A CKB System Script is a built-in set of rules or a "digital lock" that protects your assets on the network. The most common one is the Lock Script, which acts like a security guard for a storage unit (called a Cell). It uses complex math, specifically the secp256k1 elliptic curve and Blake2b hashing, to ensure that only the person who has the matching "digital key" (private key) can unlock and spend the funds inside. It does this by comparing a unique "fingerprint" (hash) from your transaction against the one stored on the lock; if they match, the lock opens

---

## CKB-VM
The CKB-VM is the virtual machine or the "computer engine" that does the actual work of running these scripts. It is built using RISC-V, which is a modern, open-source set of instructions that allows the software to talk very efficiently to the computer's physical brain (the CPU). You can think of it as the high-performance engine that powers the entire network’s logic and security checks

## Script Execution
Script Execution is the step-by-step process that happens when you submit a transaction.
1. First, the CKB-VM engine wakes up and uses special commands called syscalls to "load" the necessary security code and data into its memory.
2. Next, it retrieves evidence (called the Witness) from your transaction to verify you are who you say you are.
3. Finally, it runs the math checks. If the engine finishes and everything is correct, it returns a "0," which is the green light that your transaction is valid. If it returns any other number, the transaction is rejected as invalid.

---

## Cycles
Cycles are a way to measure the computational cost or "effort" required to run a script. Every tiny instruction the CKB-VM engine carries out consumes a certain number of cycles. You can think of cycles as "digital fuel." To prevent the network from getting bogged down by accidental or malicious infinite loops, there is a safety limit called max_block_cycles. On the main network, this limit is 3.5 billion cycles for an entire block of transactions. While a single transaction can use as much "fuel" as it needs, the total fuel used by all transactions in one block cannot exceed this limit.

**Reference:** [Regulate Scripts via Cycle Limits](https://docs.nervos.org/docs/basics/cycles)

---

## Key Takeaways

- **Cells** = immutable storage units; consume → modify → create new
- **CKBytes** = native token (value + storage capacity)
- **Scripts** = programs (smart contracts) that guard Cells
- **Lock Script** = who can spend; **Type Script** = how Cell can be used
- **CKB-VM** = RISC-V virtual machine that executes Scripts
- **Cycles** = measure of computational cost; block-level limit, no per-transaction limit
