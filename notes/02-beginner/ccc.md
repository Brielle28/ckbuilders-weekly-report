---

# CCC 

## What is CKB?

**CKB (Common Knowledge Base)** is a blockchain. Developers on CKB need to write and test:

* **Scripts** — think smart contracts
* **Transactions** — moving assets around on the chain

Normally, this requires a local setup, running a node, and configuring everything manually. **CCC Playground** removes all that friction.

---

## What is CCC Playground?

**CCC Playground** is a browser-based development environment hosted at [live.ckbccc.com](https://live.ckbccc.com).

* Write **TypeScript** code directly in the browser
* Interact with the **CKB blockchain** without any local setup

It’s perfect for experimenting, learning, and testing blockchain transactions.

---

## Core Features

### 1. Code Editor

The editor provides:

* **Syntax highlighting** — color-coded, readable code
* **Real-time execution** — run code instantly
* **Error reporting** — tells you where and why something broke
* **TypeScript support** — type safety ensures mistakes are caught before running

---

### 2. Transaction Visualization

CKB transactions consist of:

* **Inputs** — cells being consumed
* **Outputs** — new cells being created
* **Cell dependencies** — external data your transaction reads
* **Witness data** — signatures and proofs

Instead of raw JSON, CCC Playground shows a **live visual diagram** of your transaction as you build it.

---

### 3. The Four Tabs

* **Transaction tab** — full details of your transaction
* **Scripts tab** — manage your CKB scripts
* **Console tab** — shows `console.log()` outputs, errors, and debug info
* **About tab** — documentation and info about the playground

---

### 4. Testnet vs Mainnet

* **Testnet** — fake money for testing; mistakes cost nothing
* **Mainnet** — real money; mistakes could result in asset loss

Switch between networks easily via the **bottom-left corner**. Always double-check which network you are on!

---

## How You Write and Share Code

### Direct Editing

Start typing in the editor using the **preloaded TypeScript template**.

### Loading from GitHub

Add `?src=` to the URL and point it to any raw GitHub file. The playground will load it automatically.

### Sharing via Nostr

* Click **Share** to package your code into a Nostr event (decentralized post)
* Generates a **shareable link**
* No login needed — fully decentralized sharing

---

## Debugging Experience

CCC Playground supports:

* `console.log()` to print values
* **Step execution** — pause and inspect code
* `playground.render()` — visualize the current transaction state
* **Stack traces** — see exactly which line caused an error

---

## Six Use Cases

1. **Learn CKB** — study working examples and live demos
2. **Analyze Data** — connect to CKB nodes and pull blockchain data programmatically
3. **Compose Transactions** — helper functions handle inputs, outputs, fees, and capacity calculations
4. **Sign Easily** — unified cryptographic signing interface
5. **Connect Wallets** — supports CKB, Bitcoin, Ethereum wallets
6. **Interoperability** — interact with different blockchain ecosystems

---

## How to Get Started

### Option 1: Use the Playground

* No installation or setup
* Ideal for learning and experimenting

### Option 2: `create-ccc-app` (Quick App Setup)

```bash
npx create-ccc-app@latest my-ccc-app
```

* Bootstraps a complete project structure
* Asks framework and configuration questions

### Option 3: Manual Installation

Install only the packages you need:

```bash
@ckb-ccc/shell        # Node.js backend
@ckb-ccc/ccc          # Custom UI
@ckb-ccc/connector    # Web Component for wallet connection
@ckb-ccc/connector-react # React Component for wallet connection
```

---

## The Import System

```javascript
import { ccc } from "@ckb-ccc/ccc";
```

* Access all CCC features through the `ccc` object
* Supports **tree-shaking** for small production builds
* Advanced entry point `/advanced` provides low-level APIs for heavy customization

---

## Transaction Example

```javascript
// Step 1: Define the output
const tx = ccc.Transaction.from({
  outputs: [{ lock: toLock, capacity: ccc.fixedPointFrom(amount) }],
});

// Step 2: CCC finds the inputs automatically
await tx.completeInputsByCapacity(signer);

// Step 3: CCC calculates and adds the fee
await tx.completeFeeBy(signer);

// Step 4: Send the transaction
const txHash = await signer.sendTransaction(tx);
```

### What CCC Handles for You

* Finds inputs automatically
* Calculates transaction fee
* Sends the transaction
* Reduces manual work significantly


The CCC App
Separate from the library itself, there's also a CCC App — a mini web tool built using CCC that demonstrates common CKB tasks. Even non-developers can use it to do basic things on CKB. It's both a useful tool and a living demonstration of what CCC can do.

---
