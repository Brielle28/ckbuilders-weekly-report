## Builder Track Weekly Report — Week 1

**Name:** Nnadozie Clara  
**Week Ending:** 30-01-2001

---

### Courses Completed

- **How CKB Works** ([Nervos CKB Docs](https://docs.nervos.org/docs/getting-started/how-ckb-works)):
  - Intro to CKB – blockchain as decentralized immutable ledger; CKB as Common Knowledge Base; Proof of Work (POW) consensus
  - How Transaction Works – Cell Model; Cells as storage for CKBytes (value + capacity); minimum 61 CKBytes per Cell (62+ recommended); immutability and consume → modify → create new Cell; Live Cells vs Dead Cells; transaction flow (Create → Sign → Broadcast → Validate → Propagate → Confirm)
  - Scripts – what a Script is (binary executable on CKB-VM); `code_hash`, `hash_type`, `args`; Lock Script (ownership, spending) vs Type Script (how Cell can be used/modified); CKB system script `secp256k1_blake160_sighash_all` and signature verification
  - CKB-VM – RISC-V instruction set; script execution (syscalls, loading code and data, return 0 = success); Cycles and `max_block_cycles` (e.g. 3.5B on Mainnet MIRANA)
- **Quick Start (5 min)** ([Nervos CKB Docs](https://docs.nervos.org/docs/getting-started/quick-start)):
  - Installed OffCKB globally: `npm install -g @offckb/cli`
  - Started the local CKB Devnet with `offckb node`; proxy RPC at `localhost:28114`; noted `offckb clean` for a fresh chain
- **CKB Builder Handbook** (brief):
  - Skimmed structure: Introduction → Beginner (exercises, CCC, first app) → optional Intermediate/Advanced; confirmed learning path and deliverables for the bootcamp
- **CKB Academy** (brief):
  - Opened and browsed to see how lessons are structured and how they align with the Handbook
- **CCC Playground** (brief):
  - Explored the CCC Playground to understand how to run and test CKB code in the browser as part of the upcoming hands-on phase

---

### Key Learnings

- **Cell Model**
  - **Concept:** Cells are immutable storage units; each holds CKBytes (native token = value + storage capacity). Minimum 61 CKBytes per Cell; 62+ recommended for fees.
  - **Concept:** To change data you consume a Cell and create a new one; unconsumed Cells are “Live Cells” available for future transactions.
- **Transaction flow**
  - **Concept:** Select Live Cell(s) as input → define new Live Cell(s) as output (e.g. to recipient + change) → sign with private key → broadcast → node validates → propagates to mempool → miners include in block → confirmation.
  - **Concept:** Transaction fee is the small CKB difference between total input and total output capacity.
- **Scripts**
  - **Concept:** Scripts are programs run on CKB-VM (RISC-V); they guard and protect Cells. Identified by `code_hash`, `hash_type`, and `args`.
  - **Concept:** Lock Script (required) = who can spend the Cell; Type Script (optional) = how the Cell can be used or modified.
  - **Concept:** Default lock `secp256k1_blake160_sighash_all`: sign with private key; Blake2b hash of public key compared to `script_args`; secp256k1 verifies signature.
- **CKB-VM and cycles**
  - **Concept:** Script execution uses syscalls to load code and data; return 0 = valid, non-zero = invalid.
  - **Concept:** Cycles measure computational cost; block-level limit `max_block_cycles` (no per-transaction cycle cap).
- **OffCKB**
  - **Concept:** One-line setup for local CKB Devnet with pre-funded accounts and built-in Scripts (e.g. Omnilock, Spore); RPC at `localhost:28114`.

---

### Practical Progress

- Installed OffCKB CLI globally (`npm install -g @offckb/cli`).
- Started the local CKB Devnet with `offckb node` and confirmed proxy RPC at `localhost:28114`.
- Briefly reviewed the CKB Builder Handbook to map the bootcamp journey (Introduction → Beginner → Capstone).
- Browsed CKB Academy to see lesson structure.
- Explored CCC Playground to understand the in-browser testing environment for later exercises.

---

### Screenshots 
![install-running-offckb](https://github.com/user-attachments/assets/e9c5b796-93b4-4c3d-b497-a04ade94f52d)
<img width="977" height="502" alt="image" src="https://github.com/user-attachments/assets/c01cb6f9-f79e-4ec6-99ce-922271177485" />



---

### Environment

- OffCKB CLI installed (version ≥0.4.0).
- Local CKB Devnet running; RPC endpoint: `localhost:28114`.
- Node.js/npm available for global installs.
- Dev log (this repository) set up; Week 1 report and notes in place.
