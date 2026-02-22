## Builder Track Weekly Report — Week 2

**Name:** Nnadozie Clara  
**Week Ending:** 06-02-2026

---

### Courses Completed

- **CKB Academy - Lesson 1: What is CKB?** ([CKB Academy](https://academy.nervos.org)):
  - What is CKB? – Cells as the basic unit; transformation of cells (spending/creating); similarity to Bitcoin UTXO; Live Cells vs Dead Cells
  - What does a cell contain? – Cell data structure: `capacity`, `lock`, `type`, `data`; 1 CKB = 1 byte of storage; capacity calculation and constraints; example with 80 CKB cell
  - How to tell that you own a cell? – Lock and type scripts as locks on the box; Script structure (`code_hash`, `args`, `hash_type`); script execution and return value (0 = success); asymmetric encryption and public/private key verification
  - Where is the code actually located? – Code stored in dep cells (CellDep); `hash_type` values (0: "data", 1: "type", 2: "data1"); code locating workflow via data hash vs type hash; example finding SUDT type script on testnet; built-in scripts with never-unlockable lock
  - What is a transaction? – Transaction essence: inputs → outputs (destroy cells, create new ones); transaction rules (capacity conservation, fee calculation); OutPoint structure (`tx_hash`, `index`)
  - Difference between lock and type script – Lock script protects ownership (gatekeeper); Type script secures transformation rules (guardian); execution mechanism differences (lock runs for inputs by group, type runs for inputs and outputs by group)
- **CKB Academy - Lesson 2** (brief preview):
  - Opened Lesson 2 to preview upcoming content structure

---

### Key Learnings

- **Cell Model (Deep Dive)**
  - **Concept:** CKB is essentially a chain of cells constantly being created and destroyed; cells are like boxes that can store any type of data
  - **Concept:** Unspent cells = Live Cells; spent cells = Dead Cells; transaction process mirrors Bitcoin UTXO model
  - **Concept:** Cell data structure has four fields: `capacity` (space size in CKBytes), `lock` (Script for ownership), `type` (optional Script for transformation rules), `data` (arbitrary bytes)
- **Capacity and Storage**
  - **Concept:** 1 CKB = 1 byte of storage space; total cell size (all 4 fields) must be ≤ capacity
  - **Concept:** Capacity is measured in shannons (1 CKB = 10^8 shannons); example: 100 CKB = 100 bytes, can store ~50 Chinese characters (2 bytes each)
  - **Concept:** On-chain storage is precious; storing large data (e.g., entire novel) requires significant CKB tokens
- **Scripts and Ownership**
  - **Concept:** Scripts are code + parameters; `code_hash` locates code, `args` provides arguments, `hash_type` indicates locating method
  - **Concept:** Script execution returns 0 = unlock successful, non-zero = unlock failed; uses asymmetric encryption (public key in args, private key signs transaction)
  - **Concept:** Lock script protects ownership (who can unlock); Type script ensures transformation rules (how cell can be transformed)
- **Code Location (Dep Cells)**
  - **Concept:** Code is stored in another cell's `data` field; referenced via `code_hash` and `hash_type` through dep cells (CellDep)
  - **Concept:** `hash_type` determines how code is located: "data"/"data1" → match `blake2b_ckbhash(data)` of dep cell; "type" → match `blake2b_ckbhash(type script)` of dep cell
  - **Concept:** Built-in scripts use never-unlockable lock (`code_hash: 0x00...00`) so code always exists; other scripts can be redeployed if dep cell is destroyed
- **Transactions**
  - **Concept:** Transaction = inputs (live cells) → outputs (new live cells); input cells become dead, output cells become live
  - **Concept:** Capacity conservation: sum(input capacities) > sum(output capacities); difference = transaction fee paid to miners
  - **Concept:** Inputs use OutPoint (`tx_hash` + `index`) to reference cells, not full cell data (storage optimization)
- **Lock vs Type Script Execution**
  - **Concept:** Lock scripts run for all inputs by group (protect ownership); Type scripts run for all inputs and outputs by group (ensure transformation rules)
  - **Concept:** CKB groups inputs/outputs by script and runs same script only once per group (efficiency)

---

### Practical Progress

- Completed **CKB Academy Lesson 1** ("What is CKB?") with interactive examples
- Worked through capacity calculation example (Hex 0x12a05f200 → decimal)
- Explored interactive cell visualization tool to understand capacity vs actual occupancy
- Found SUDT type script code on CKB testnet explorer using provided parameters (code_hash, hash_type, tx_hash, index)
- Reviewed Lesson 2 structure to prepare for next week

---

### Screenshots / Evidence

<img width="1697" height="1318" alt="image" src="https://github.com/user-attachments/assets/8fbb687e-af43-4cef-9c7a-37e60a0591d4" />
<img width="1646" height="1276" alt="image" src="https://github.com/user-attachments/assets/090eafe7-57e7-4c3b-9b56-79b2565e0840" />

---

### Environment

- CKB Academy account and access set up
- CKB testnet explorer access for code location exercises
- Dev log updated with Lesson 1 notes
