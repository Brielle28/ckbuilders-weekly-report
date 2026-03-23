## Builder Track Weekly Report — Week 7

**Name:** Nnadozie Clara
**Week Ending:** 21-03-2026

---

### Courses Completed
- **CCC (CKBers' Codebase) Official Documentation — Full Review:**
    - CCC Overview & Architecture
    - CCC App — Feature Walkthrough
    - Code Examples — Transaction Building
    - Playground — In-Browser Testing Environment
    - API Reference — Module Index & Package Breakdown


### Key Learnings

- **What CCC is**
    - Concept: CCC (CKBers' Codebase) is a one-stop JS/TS SDK for building on the CKB (Nervos Network) blockchain. It abstracts away the low-level complexity of CKB's UTXO-based cell model and provides clean, chainable APIs for building and sending transactions.
    - Concept: CCC is wallet-agnostic and supports interoperability across multiple chain ecosystems — meaning it can work with wallets from different blockchains connecting into CKB's ecosystem.
    - Concept: The library fully enables CKB's Turing completeness and cryptographic freedom power, meaning developers can build complex programmable logic on top of CKB cells using CCC as the abstraction layer.

- **CCC App - Mini Toolset**
    - Concept: The CCC App is a publicly accessible mini-toolset that demonstrates CKB's core capabilities. It is usable even by non-developers, making it a great onboarding and demo tool.
    - Features include: signing and verifying messages, transferring native CKB tokens, transferring UDT tokens, issuing xUDT tokens (via Single-Use Lock or Type ID cell), managing Nervos DAO positions, transferring CKB with time locks, hashing messages, generating mnemonics and keypairs, encrypting/decrypting keystores, and interacting with the Spore Protocol (create cluster, mint spore, transfer/melt spore).
    -  The CCC App also supports the old Lumos SDK for backward-compatible CKB token transfers, which is important for developers migrating from older CKB tooling.

- **Transaction Building with CCC**
    - Concept: Building a CKB transaction with CCC follows a minimal and readable three-step pattern: (1) define transaction outputs using ccc.Transaction.from(), (2) complete inputs automatically by capacity using tx.completeInputsByCapacity(signer), (3) calculate and append fees with tx.completeFeeBy(signer), and (4) broadcast using signer.sendTransaction(tx).
    - Concept: The ccc.fixedPointFrom(amount) utility handles CKB's fixed-point arithmetic, abstracting the Shannon (smallest unit) conversion from human-readable CKB values.
    - Transactions are fee-aware by default — completeFeeBy automatically calculates the transaction fee rate without the developer needing to manually compute it, reducing a major source of errors in raw CKB development.
    - The signer abstraction means the same transaction-building code works regardless of which wallet or signing method is used — the signer handles key management and signing under the hood.

- **CCC Playground**
    - Concept: The CCC Playground is an integrated browser-based testing environment that supports live code execution, data visualization, and code sharing — no local setup required.
    - Cells are visualized graphically in the Playground using a three-layer ring model: the outermost ring represents the lock script (ownership), the middle ring represents the type script (asset type), and the filled center indicates whether the cell's capacity is fully used for data storage.
    - Color coding is used meaningfully — cells sharing the same outer ring color share the same lock script (same owner), and cells sharing the same inner ring color share the same type script (same asset type). This makes it easy to visually reason about cell ownership and asset classification at a glance.
    - Clicking any cell in the Playground reveals its full details, making it a powerful debugging and learning tool for understanding the cell model in practice.

- **CCC API — Module Architecture**
    - Concept: The CCC SDK is organized as a monorepo of scoped npm packages, each addressing a specific concern. The full list of modules includes: @ckb-ccc/ccc, @ckb-ccc/connector, @ckb-ccc/connector-react, @ckb-ccc/core, @ckb-ccc/eip6963, @ckb-ccc/joy-id, @ckb-ccc/lumos-patches, @ckb-ccc/nip07, @ckb-ccc/okx, @ckb-ccc/rei, @ckb-ccc/shell, @ckb-ccc/spore, @ckb-ccc/ssri, @ckb-ccc/udt, @ckb-ccc/uni-sat, @ckb-ccc/utxo-global, and @ckb-ccc/xverse.
    - The top-level @ckb-ccc/ccc module exposes a SignersController class and a WalletWithSigners type alias, acting as the main entry point for managing wallet connections and signer instances across the SDK.
    - Wallet connector packages (eip6963, joy-id, nip07, okx, uni-sat, xverse, rei) each correspond to a specific wallet ecosystem — EVM-compatible wallets (via EIP-6963), JoyID (biometric/passkey), Bitcoin wallets (UniSat, Xverse, OKX), and others — enabling CKB to function as a multi-ecosystem settlement layer.
    - @ckb-ccc/spore is a dedicated module for the Spore Protocol, CKB's on-chain digital object (DOB) standard, supporting cluster creation, spore minting, transferring, and melting operations.
    - @ckb-ccc/udt handles User Defined Token operations, specifically xUDT — CKB's extensible token standard — covering both issuance and transfer.
    - @ckb-ccc/shell and @ckb-ccc/ssri suggest support for script-level interactions and SSRI (Script-Sourced Rich Information), which are more advanced CKB scripting patterns.
    - @ckb-ccc/utxo-global and @ckb-ccc/uni-sat indicate support for UTXO-native wallets, reinforcing CKB's design as a UTXO chain that shares conceptual ground with Bitcoin.
    - @ckb-ccc/lumos-patches — Backward Compatibility,  This module provides patches specifically for using Lumos (the older CKB JS SDK) together with CCC, enabling gradual migration rather than a hard cutover. It exposes three key functions: generateScriptInfo, generateDefaultScriptInfos, and asserts — utilities for generating and validating CKB script metadata in a Lumos-compatible format while operating within the CCC ecosystem. The existence of this module reflects a maturity in the CKB tooling ecosystem — the team acknowledges that many existing projects are built on Lumos and provides a bridge rather than forcing a rewrite.

### Practical Progress

 - Reviewed the full CCC documentation site end-to-end at docs.ckbccc.com, covering all four major sections: CCC overview, CCC App, Code Examples, and Playground.
 - Studied the minimal CKB transfer example and traced the full transaction lifecycle: output definition → input completion by capacity → fee completion → broadcast.
 - Explored the CCC Playground at playground.ckbccc.com and understood the cell visualization model (three-layer ring: lock, type, data occupancy).
 - Reviewed the full API module index at api.ckbccc.com and catalogued all 17 scoped packages in the CCC monorepo.
 - Studied the @ckb-ccc/lumos-patches module page in detail, understanding its role as a migration and compatibility bridge between Lumos and CCC.
 - Identified the wallet ecosystem coverage of CCC: EVM (EIP-6963), Bitcoin-native (UniSat, Xverse, OKX), passkey-based (JoyID), and UTXO-global wallets.
 - Cross-referenced the CCC App feature list with the API module structure to understand how high-level app features map to underlying SDK packages.


### Screenshots / Evidence
https://github.com/user-attachments/assets/3a872e17-4ea3-42af-aa69-48e04114c0ed

### Environment
- CCC Docs — Accessed via docs.ckbccc.com. Navigation covers CCC, CCC App, Code Examples, Playground, and external API link.
- CCC API Reference — Accessed via api.ckbccc.com. Full TypeDoc-generated reference for all 17 monorepo packages.

