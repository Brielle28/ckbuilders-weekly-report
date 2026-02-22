# CKB Academy - Lesson 1: What is CKB?

**Source:** [CKB Academy - Lesson 1](https://academy.nervos.org)

---

## Core Concept: Cells and Transformation

To understand **CKB**, we must capture the very essence: **it's all about cells and the transformation of cells**.

### What is a Cell?

- **Cell** is the basic unit of CKB, similar to a cell in the human body
- All cells constitute the general state of the entire CKB blockchain
- When we initiate a transaction (state change), it's nothing more than **spending some cells while creating some new ones**
- This process is the same as the **Bitcoin UTXO** model

### Live Cells vs Dead Cells

- **Live Cells:** Unspent cells (available for use)
- **Dead Cells:** Spent cells (consumed in transactions)

A CKB chain keeps on spending and creating cells through transactions, just like the renewal and division of cells throughout the body.

---

## What Does a Cell Contain?

### Cell Data Structure

```javascript
Cell: {
  capacity: HexString;  // Space size (number of CKBytes)
  lock: Script;        // Required script (ownership lock)
  type: Script;        // Optional script (transformation rules)
  data: HexString;     // Arbitrary bytes (any type of data)
}
```

### Field Definitions

1. **`capacity`**
   - The space size of the cell
   - Integer number of native tokens (CKBytes) represented by this cell
   - Usually expressed in hexadecimal
   - Basic unit: **shannon** (1 CKB = 10^8 shannons)

2. **`lock`**
   - A script, essentially the equivalent of a lock
   - Required field
   - Controls ownership and access to the cell

3. **`type`**
   - A script, same as lock but for a different purpose
   - Optional field
   - Dictates how a cell can be used or modified

4. **`data`**
   - Can store arbitrary bytes (any type of data)
   - Can be a hash, text, date, or even binary code that can be referenced by other cells and run on-chain through CKB-VM
   - This is how **smart contracts** work on CKB

### Capacity Constraint

**Important rule:** A cell's total size for all four fields must be **less than or equal to** the capacity of the cell.

```
capacity = Cell's total space
         >= Sum of the 4 fields' byte length
```

### Storage Economics

- **1 CKB = 1 byte of storage space**
- Example: 100 CKB = 100 bytes of on-chain space
- Example: One Chinese character = 2 bytes (GBK encoding)
  - With 100 CKB, you can store less than 50 Chinese characters
- Example: Dream of the Red Chamber (~780,000 words) ≈ 1.56 million CKB tokens needed

**On-chain storage is a precious asset.**

### Why "Common Knowledge Base"?

CKB stores consensus data on-chain, enabling anyone to upload valuable and necessary data to the consensus. It is comparable to a **knowledge base owned by all of humanity**—hence the name **Common Knowledge Base**.

---

## How to Tell That You Own a Cell?

## What is Script 
a script is a small program (code) that the blockchain runs to decide if something is allowed . in simple terms it ia a rule written in code that ckb checks befoore accepting a transaction.

### Lock and Type Scripts

If the cell is a **box**, the lock and type scripts are the **two locks on the box**:

- **Lock script:** The default lock (required) and it determins who can spend the cell that is why it is required. in simple words it controls ownership, spending , provides security and it is always required.
- **Type script:** An optional lock. this script defines rules for the cell. example what kind of cell is this and how should it behave.

### Script Structure

```javascript
Script: {
  code_hash: HexString;  // Hash of a certain piece of code
  args: HexString;       // Arguments transferred to the code
  hash_type: Uint8;      // Three allowed values: {0: "data", 1: "type", 2: "data1"}
}
```

### Script Execution

- Scripts are a piece of **code and parameters**
- When we try to consume a cell, scripts run automatically
- They take the parameters and proof we submit (such as signature) to determine if the locks can be unlocked
- **Return value:**
  - **0** = lock can be opened (success) ✅
  - **Non-zero** = unlocking failed ❌

### Ownership Verification

We can introduce an **asymmetric encryption algorithm** via `code_hash` and place our own **public key** on the `args` as an argument. When spending the cell:

1. Use the **private key** to sign the transaction
2. The cryptographic algorithm inputs the public key and signature
3. It figures out whether the corresponding private key initiated the transaction
4. This identifies whether it was the real cell owner

**Warning:** If you create a cell with a lock that can be unlocked by anyone, anyone can spend your money! Locks are vital to the cell.

---

## Where Is the Code Actually Located?

### Code Storage in Dep Cells

The `code_hash` is not the actual code, but an **index** that allows us to retrieve the code. The code is stored in **another cell's `data` field**!

This dependency cell is called **CellDep** (dep cell).

### Code Locating via hash_type

Depending on the value of `hash_type`, `code_hash` has different interpretations:

- **If `hash_type` is "data" or "data1":**
  - `code_hash` should match `blake2b_ckbhash(data)` of a dep cell
  
- **If `hash_type` is "type":**
  - `code_hash` should match `blake2b_ckbhash(type script)` of a dep cell

### Code Locating Workflow

When unlocking a cell, a transaction imports the dep cell, and CKB follows the rules above to find and execute the corresponding code.

**Why use indexing instead of putting real code directly?**

- **Major advantage:** If everyone needs the same type of lock, the lock code will be identical, and so will the `code_hash` value
- Then it's just a matter of introducing the same dep cell rather than deploying the same code all over again for each case
- **Efficiency:** Code reuse across many cells

### Example: Finding SUDT Type Script

**Simple User Defined Token (SUDT)** is a widely used type script. To find its real code on CKB testnet:

| Parameter | Value |
|-----------|-------|
| `code_hash` | `0xc5e5dcf215925f7ef4dfaf5f4b4f105bc321c02776d6e7d52a1db3fcd9d011a4` |
| `hash_type` | `type` |
| `tx_hash` | `0xe12877ebd2c3c364dc46c5c992bcfaf4fee33fa13eebdf82c591fc9825aab769` |
| `index` | `0x0` |
| `dep_type` | `code` |

Steps:
1. Click the `tx_hash` link → jump to transaction view on CKB testnet explorer
2. Click the **Cell Info** button of #0 Output
3. View the real code of SUDT type script

### What If the Lock Code Is Lost?

**Theoretically:** If someone destroys the dep cell (spends it), the lock's code will be gone, and cells using that lock can no longer be unlocked.

**For CKB's built-in scripts:**
- The cells containing built-in lock script code are inherently inaccessible by anyone
- They use a **never-unlockable lock:**
  ```javascript
  {
    code_hash: 0x0000000000000000000000000000000000000000000000000000000000000000
    hash_type: data
    args: 0x
  }
  ```
- This means no one will ever be able to unlock it, and the code will always be there

**For other scripts:**
- We can still unlock our own cell if the dep cell was destroyed
- We can **redeploy the same lock code** to a new cell
- Then reference the new cell as a dep cell to retrieve the lock code

---

## What Is a Transaction?

### Transaction Essence

Constructing a transaction is to **destroy some cells and create some more**.

```
transaction: inputs -> outputs
```

The essence of inputs and outputs are still some cells:

```
inputs:
    some cells (live cells)...
    |
    |
    |
    \/
outputs:
    some new cells (become live cells)...
```

- Cells in **inputs** must all be **live cells**
- Input cells will be **spent** and become **dead cells** after transaction is committed
- Newly created **output cells** will become new **live cells**

### Transaction Rules

**Capacity Conservation:**
```
sum(cell's capacity for each cell in inputs)
> sum(cell's capacity for each cell in outputs)
```

A transaction **cannot mint capacities from the air**.

**Transaction Fee:**
```
sum(cell's capacity for each cell in inputs)
- sum(cell's capacity for each cell in outputs)
= fee
```

Miners collect the difference as a fee.

### OutPoint Structure

For storage optimization, we don't put the complete cell in an input. Instead, we put the **cell's index** that leads us to the real input cell. This index structure is called **OutPoint**:

```javascript
OutPoint: {
  tx_hash: HexString;  // Hash value of the transaction to which the target cell belongs
  index: Uint32;       // Cell position in the transaction to which the target cell belongs
}
```

---

## Difference Between Lock and Type Script

### Purpose

- **Lock Script:** Usually used to **protect the ownership** of the box, indicating **who can unlock** the box
- **Type Script:** Used to ensure that the cell **follows certain application logic** (transformation rules)

### Execution Mechanism

**Lock Script:**
- In a transaction, lock scripts run for **all inputs by group**

**Type Script:**
- In a transaction, type scripts run for **all inputs and outputs by group**

**Note:** CKB does not run the script one by one. It first **groups the inputs or outputs by script** and runs the same script only once.

### Examples

**Lock Script Example:**
- Protects ownership: only the owner (or authorized person) can unlock and spend the cell

**Type Script Examples:**
- Rule: A cell can produce only **one new cell** at a time in a transaction
- Rule: A cell must **never show the word "carrot"** in its data field during a transaction

### Analogy

- **Lock script** = The **gatekeeper** (protects ownership)
- **Type script** = The **guardian** (secures transformation rules)

Due to the variations in execution mechanisms, different suitable uses are derived. These are the recommended official usages, but you are perfectly free to have your own ideas.

---

## Summary

### Key Concepts

1. ✅ CKB is essentially a **chain of cells** that are constantly being created and destroyed
2. ✅ A cell is a **box** that can be used to store **all types of data**
3. ✅ To own a cell and store data on-chain, you need tokens: **1 CKB = 1 Byte**
4. ✅ The byte size of the entire cell **cannot exceed** the value of the `capacity` field
5. ✅ To protect your cell, you must put a **lock** on the cell so that only you or someone you authorize can open it
6. ✅ A lock is essentially a piece of **code** that checks if the cell can be unlocked using arguments and user-provided signatures or proofs
7. ✅ The return value of **0** means that the lock was unlocked successfully, while any other value means the unlock attempt failed
8. ✅ The lock's `code_hash` and `hash_type` fields are used to **locate code**, which is stored in the `data` field of a dep cell
9. ✅ Each cell can carry **two scripts**: one is called **lock script** (default) and the other, **type script** (optional)
10. ✅ In one transaction, **lock scripts run for all inputs by group**, while **type scripts run for all inputs and outputs by group**
11. ✅ The differences in the execution mechanism result in different uses: Lock scripts protect ownership; Type scripts handle transformation rules
12. ✅ Constructing a transaction is fundamentally about **destroying some cells and creating some new ones**

---
