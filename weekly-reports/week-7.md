## Builder Track Weekly Report — Week 7

**Name:** Nnadozie Clara
**Week Ending:** 14-03-2026

---

### Courses Completed

- **CKB SDK & Dev Tools — CCC Playground** ([Nervos CKB Docs – Playground](https://docs.nervos.org/docs/sdk-and-devtool/playground)):
  - What CCC Playground is and how it works as a browser-based CKB development environment
  - Core features: interactive code editor, transaction visualization, four-tab interface (Transaction, Scripts, Console, About)
  - Testnet vs Mainnet — how to switch networks and why it matters
  - Writing and loading code: direct editing, loading from GitHub via `?src=`, sharing via Nostr protocol
  - Debugging tools: `console.log()`, step execution, `playground.render()`, stack traces

- **CCC Docs — Introduction** ([CCC Docs](https://docs.ckbccc.com)):
  - What CCC (CKBers' Codebase) is and its role as a one-stop JS/TS SDK for CKB development
  - The six core use cases: Learn CKB, Analyze Data, Compose Transactions, Sign Easily, Connect Wallets, Interoperability
  - Getting started options: Playground, `create-ccc-app` CLI tool, and manual installation
  - Available packages: `@ckb-ccc/shell`, `@ckb-ccc/ccc`, `@ckb-ccc/connector`, `@ckb-ccc/connector-react`
  - The import system and how the `ccc` object works with tree-shaking
  - Basic transaction composition flow using CCC helpers

---

### Key Learnings

- **CCC Playground**
  - **What it is:** A browser-based IDE specifically for CKB — no local node or setup required to test transactions and scripts
  - **Transaction Visualization:** Instead of reading raw JSON, the playground renders a live visual diagram of inputs, outputs, cell dependencies, and witness data as you build
  - **Testnet vs Mainnet:** Switching is done from the bottom-left corner; always verify which network you are on before sending anything — mistakes on Mainnet involve real assets
  - **Code Sharing via Nostr:** Clicking Share packages your code into a decentralized Nostr event and generates a shareable link — no login or account needed
  - **Debugging:** Step execution lets you pause and inspect the transaction state at each point; `playground.render()` acts as a visual breakpoint

- **CCC Library**
  - **What CCC is:** A JavaScript/TypeScript library (not a framework) that abstracts away the complexity of building on CKB — handling inputs, fees, signing, and wallet connections
  - **`create-ccc-app` clarification:** This is a CLI scaffolding tool, not a framework itself — it just generates a project structure so you can start building faster
  - **Transaction helpers:** CCC handles a lot automatically — `completeInputsByCapacity(signer)` finds your inputs, `completeFeeBy(signer)` calculates the fee, and `signer.sendTransaction(tx)` sends it
  - **Tree-shaking:** All CCC features are accessed via the `ccc` object, and only what you actually use gets bundled into your production build
  - **Advanced entry point:** `/advanced` exports `cccA` for low-level customization, but these APIs are unstable and subject to change

---

### Practical Progress

- Read and took notes on the full CCC Playground documentation — understood the interface, debugging tools, network switching, and code-sharing features
- Read and took notes on the CCC introduction docs — understood the library structure, use cases, installation options, and the transaction composition model
- Studied the minimal transaction example end-to-end and understood what each step does and what CCC handles automatically
- Began familiarizing with the available CCC packages and when to use each one

---

### Screenshots / Evidence

None this week.

---

### Environment

- CCC Playground ([live.ckbccc.com](https://live.ckbccc.com)) — browser-based, no local setup required
- Notes: `notes/02-beginner/ccc.md`