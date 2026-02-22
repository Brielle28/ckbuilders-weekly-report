## Builder Track Weekly Report — Week 3

**Name:** Nnadozie Clara  
**Week Ending:** [13-02-2026]

---

### Courses Completed

- **CKB Academy – Lesson 2: CKB Basic Practical Operation** ([CKB Academy](https://academy.nervos.org)):
  - Connecting Wallet – connected MetaMask to CKB testnet; received 10 Live Cells (100 CKB each); used Fetch Cells and viewed CKB address (`ckt1...`); understood that wallet balance = sum of Live Cell capacities
  - Testnet chain config – reviewed Chain Info (PREFIX `ckt`); built-in scripts (SECP256K1_BLAKE160, OMNILOCK, DAO, SUDT, etc.); noted cellDeps required for Omnilock (OMNILOCK + SECP256K1_BLAKE160)
  - What is Transaction? – transaction = spending Live Cells and creating new ones (inputs → outputs); off-chain computing, on-chain verifying; steps: choose inputs, build outputs, add cell deps, sign (signature in witnesses), submit
  - Send a Transaction – completed the **practical exercise**: filled the transaction template (cellDeps, inputs from Live Cells, outputs with capacity and lock); set output capacity to leave miner fee (e.g. `0x2540be018`); generated message, signed with wallet, serialized WitnessArgs, placed result in `witnesses`, saved and sent transaction to testnet
- **Introduction to Script (Smart Contract Basics)** ([Nervos CKB Docs](https://docs.nervos.org/docs/script/intro-to-script) / notes):
  - Intro to Script – Script = binary executable on-chain (smart contract); execution in CKB-VM; return 0 = success, non-zero = failure
  - Script Types – Lock Script (ownership and access to Cell); Type Script (how Cell is used in a transaction); execution difference: output Lock Scripts not run; input Lock Scripts and input/output Type Scripts are run
  - Script Structure – `code_hash`, `hash_type`, `args`; data/type hash for locating code; args differentiate Scripts (e.g. public key hash in lock args)
  - Invoke Scripts via Syscalls – scripts use syscalls to get data from the chain; return codes and sources (input/output/dep cells, headers)
  - Regulate Scripts via Cycle Limits – cycles as unit of cost; `max_block_cycles` (e.g. 3.5B on MIRANA); how cycles are measured (loading code, instructions, syscalls); optimizations (B extension, MOP Fusion)
  - VM Selection, VM History – VM versions (0, 1, 2); hash_type and data vs type hash for VM version; Type ID for upgradeable scripts; Spawn (cross-script calls, pipes, child processes)

---

### Key Learnings

- **Building and sending a CKB transaction (practical)**
  - **Transaction shape:** A tx is inputs (Live Cells, referenced by OutPoint: `txHash` + `index`) and outputs (new cells: capacity, lock, optional type). You also attach **cellDeps** (script cells the tx needs) and **witnesses** (proofs, e.g. signature). Fee = sum of input capacities minus sum of output capacities; the network requires a minimum fee (e.g. ≥309 shannons at 1000 shannons/KW), so output total must be strictly less than input total.
  - **Omnilock and cellDeps:** When spending cells locked by Omnilock, the tx must include two cellDeps: **OMNILOCK** (depType `code`) and **SECP256K1_BLAKE160** (depType `depGroup`). The output lock uses OMNILOCK’s `codeHash` and `hashType`, and your wallet’s lock **args** from Chain Info.
  - **Signing and witnesses:** You don’t type the signature manually. Flow: generate the message → sign with the connected wallet → paste the signature into the Academy → “Serialize witnessArgs” → put the **full** serialized hex into the tx’s `witnesses` array (replacing `"0x"`), then save and send. If `witnesses` stay empty or wrong, the lock script fails (e.g. TransactionFailedToVerify / error 71).
  - **Common errors:** “No alive cells found from OutPoints” means the input OutPoint is spent or wrong — use **Live Cells** in the toolbox and **Fetch Cells**, then pick a current Live Cell and use its `txHash` and `index` in `inputs`. “PoolRejectedTransactionByMinFeeRate” means fee is 0 or too low — reduce output capacity so enough is left as fee (e.g. `0x2540be018` for a 100 CKB input).
  - → **Detailed steps and values:** [CKB Academy Lesson 2 – my notes](../notes/01-introduction/ckb-academy-lesson2-ckb-pratical-operation.md)

- **Scripts and smart contract basics (Intro to Script)**
  - **What a Script is:** A Script is a binary executable run on-chain in **CKB-VM**; it’s the CKB notion of a smart contract. Success = return 0; non-zero = failure. Scripts use **syscalls** to read data (transaction, cells, headers) because they can’t access the outside world directly.
  - **Lock vs Type Script:** **Lock Script** controls who can spend the Cell (ownership). **Type Script** controls how the Cell may be used or changed in a transaction. Important: **output** cells’ Lock Scripts are **not** executed; only **input** Lock Scripts and **input/output** Type Scripts are run.
  - **Script structure:** A Script is identified by **code_hash**, **hash_type**, and **args**. `code_hash` + `hash_type` tell CKB how to find the code (in a dep cell): **data** hash = hash of the script binary; **type** hash = hash of a Cell’s Type Script. **args** are parameters (e.g. public key hash) so many users can share the same script code with different behavior.
  - **Cycles and VM:** Execution cost is measured in **cycles**; the chain has a **max_block_cycles** (e.g. 3.5B on MIRANA) per block. **VM version** can be fixed (data hash) or follow upgrades (type hash). **Type ID** is a pattern for upgradeable scripts; **Spawn** allows cross-script calls (child processes, pipes).
  - → **Full summary (syscalls, cycles, VM selection, Type ID, Spawn):** [Smart Contract Basics – my notes](../notes/01-introduction/smart-contract-basics.md)

---

### Practical Progress

- Completed **CKB Academy Lesson 2** end-to-end: connected MetaMask to CKB testnet, viewed Live Cells and blocks, reviewed Chain Info
- Completed the **Send a Transaction** exercise: filled transaction JSON (cellDeps, inputs from a Live Cell, outputs with fee); resolved min-fee error by reducing output capacity (e.g. `0x2540be018`); generated tx hash, signed with wallet, serialized WitnessArgs, updated `witnesses`, and sent transaction to testnet
- Took notes on transaction structure, WitnessArgs, and common errors → [CKB Academy Lesson 2 – my notes](../notes/01-introduction/ckb-academy-lesson2-ckb-pratical-operation.md)
- Studied **Introduction to Script** and Smart Contract Basics; summarized → [Smart Contract Basics – my notes](../notes/01-introduction/smart-contract-basics.md)

---

### Screenshots / Evidence

https://github.com/user-attachments/assets/b409d5a6-edc2-4ce9-926f-e5b1d8b33f64

click the link to watch the full video - [ckb-academy-transaction](https://github.com/user-attachments/assets/b409d5a6-edc2-4ce9-926f-e5b1d8b33f64)


---

### Environment

- MetaMask connected to CKB testnet; Academy-assigned Live Cells (100 CKB × 10)
- Chain Info and Live Cells toolbox used for cellDeps and input OutPoints
- Dev log and notes updated (Academy Lesson 2, Smart Contract Basics)
