## Builder Track Weekly Report — Week 7

**Name:** Nnadozie Clara
**Week Ending:** 14-03-2026

---

### Courses Completed

- **CKB SDK & Dev Tools — JavaScript/TypeScript (CCC)** ([Nervos CKB Docs – Playground](https://docs.nervos.org/docs/sdk-and-devtool/playground)):
  - Introduction to CCC as the recommended JS/TS SDK for CKB development
  - Overview of what CCC can do and the problems it solves
  - High-level look at available packages and installation options

- **CCC Official Docs — Overview** ([CCC Docs](https://docs.ckbccc.com)):
  - What CCC is and why it exists
  - The five core use cases CCC is designed to support
  - Getting started paths: Playground, CLI tool, and manual install

---

### Key Learnings

- **What CCC Is**
  - Concept: CCC (CKBers' Codebase) is a one-stop JavaScript/TypeScript SDK built specifically for CKB development. It is the officially recommended tool, positioned as a better alternative to older tools like Lumos and ckb-js-sdk.
  - Beyond being a dev SDK, CCC also functions as a wallet connector — it handles interoperability between wallets from different blockchain ecosystems and connects them into CKB.

- **What CCC Can Do — The Five Use Cases:** 
  - Learn CKB: CCC comes with numerous code examples and web demos to help new developers understand how CKB works in practice.
  - Analyze Data: CCC can interact with CKB nodes directly and process blockchain data programmatically, making it useful for data and indexing work.
  - Compose Transactions: CCC provides highly intuitive and customizable transaction composition helpers — this is its most central capability and will be the main focus of Week 8.
  - Sign Easily: CCC provides a unified signing interface with pre-built signing methods that work across multiple chains seamlessly.
  - Connect Wallets: CCC includes a connector component that can be integrated in minutes, or used as a base to build a fully custom wallet connection UI.

- **Getting Started Options**
  - CCC can be used immediately in the browser via the Playground at live.ckbccc.com — no installation needed.
  - For building real applications, the create-ccc-app CLI scaffolds a full project in one command. Manual installation is also available with targeted packages like @ckb-ccc/shell for Node.js or @ckb-ccc/connector-react for React apps or Custom UI (@ckb-ccc/ccc), Web Component (@ckb-ccc/connector) or depending on the developer need 
  - All CCC functionality is accessed through a single unified ccc object regardless of which package is installed, keeping the API surface consistent.
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