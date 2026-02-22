# CKB Builder Track – Learning Journey

Building on the **Common Knowledge Base** with the Community Keeps Building programme.

---

## 👤 Participant

| Field | Value |
|-------|--------|
| **Name** | Nnadozie clara |
| **Track** | Builder |
| **Program** | CKB (Nervos Community Catalyst) |
| **Duration** | 3 months |

---

## 🎯 About This Repository

This repository is my **developer log** for the CKB Builder Track. It tracks:

- **Theoretical study** – CKB Academy, Handbook, and core docs  
- **Hands-on work** – tutorials, CCC Playground, and exercises  
- **Proof of completion** – screenshots and code  
- **Weekly progress** – status updates per programme requirements  
- **Capstone** – final application built on Nervos CKB  

The goal is to upskill in CKB development, complete programme deliverables on time, and build a small application that showcases what I’ve learned.

---

## 📋 Recent progress (Week 4)

**Introduction to Smart Contract (Smart Contract Basics)**  
Studied the full Intro to Script material: what Scripts are (binaries on CKB-VM), Lock vs Type Script, syscalls, cycle limits, VM selection (data vs type hash), Type ID for upgradeable scripts, and Spawn for cross-script calls (IPC, pipes). Also covered debugging (CKB-Debugger), script testing, fuzzing, and the recommended upgrade workflow.  
→ **[Smart Contract Basics – notes](notes/01-introduction/smart-contract-basics.md)**

**View and Transfer CKB**  
Ran the Transfer CKB tutorial dApp on OffCKB devnet: loaded an account from a private key, viewed total capacity (balance) and lock script, and sent CKB to another address. The note explains how `generateAccountFromPrivateKey`, `capacityOf`, and `transfer` work (building outputs, completing inputs and fee, signing and sending).  
→ **[View and Transfer CKB – notes](notes/02-beginner/view-and-transfer-ckb-balance.md)**

**Store Data on Cell**  
Ran the Store Data on Cell tutorial: wrote a text message (e.g. "Hello CKB!") into a Cell's data field and read it back using the transaction hash and output index. The note walks through each function in `lib.ts`: encoding/decoding (utf8ToHex, hexToUtf8), building and sending the message tx (buildMessageTx), and reading the on-chain message (readOnChainMessage).  
→ **[Store Data on Cell – notes](notes/02-beginner/store-data-on-cell.md)**

**Weekly report:** [Week 4](weekly-reports/week-4.md)

---

## 📚 Learning Resources

| Resource | Purpose |
|----------|---------|
| [CKB Builder Handbook](https://github.com/nervosnetwork/ckb-builder-handbook) | Main roadmap: Introduction → Beginner → (optional) Intermediate/Advanced |
| [CKB Academy](https://academy.nervos.org) | 8 modules + interactive tutorial; L1 concepts |
| [CCC (Common Chain Connector)](https://ckbccc.com) | Build on CKB with JS/TS; App, Playground, examples, API |
| [CCC Playground](https://live.ckbccc.com) | Run and test code in the browser |
| [Nervos CKB Docs](https://docs.nervos.org) | Official documentation, Quick Start (OffCKB), Introduction to Script |
| [OffCKB](https://github.com/ckb-devrel/offckb) | Local dev environment and project setup |
| [Testnet Faucet](https://faucet.nervos.org) | Testnet CKB for development |
| [CKB Explorer](https://explorer.nervos.org) | Inspect transactions and chain state |

---

## 🗺️ Learning Journey (Roadmap)

### Phase 1: Introduction  
*Concepts and environment*

| # | Milestone | Status |
|---|-----------|--------|
| 1.1 | Read *Introduction to Nervos CKB* (Cell Model, terminology) | ⏳ |
| 1.2 | *Getting started on CKB* – Quick Start with **OffCKB**; networks & RPCs | ⏳ |
| 1.3 | CKB Academy – Lessons 1 & 2 | ⏳ |
| 1.4 | Read *Introduction to Script* (smart contracts on CKB) | ✅ |
| 1.5 | Create and maintain **GitHub dev log** (this repo) | ✅ |

---

### Phase 2: Beginner – Hands-on  
*Tutorials and first app*

| # | Milestone | Status |
|---|-----------|--------|
| 2.1 | **Transfer CKB** – tutorial + screenshots + dev log | ✅ |
| 2.2 | **Store Data on Cell** – tutorial + screenshots + dev log | ✅ |
| 2.3 | **Create Fungible Token** – tutorial + screenshots + dev log | ⏳ |
| 2.4 | **Create DOB (Digital Object)** – tutorial + screenshots + dev log | ⏳ |
| 2.5 | **Build a Simple Lock** – tutorial + screenshots + dev log | ⏳ |
| 2.6 | **Learn building on CKB (JavaScript & TypeScript) using CCC** | ⏳ |
| 2.7 | Explore **CCC App** | ⏳ |
| 2.8 | Run code in **CCC Playground**; complete examples | ⏳ |
| 2.9 | Try **CCC code examples**; skim **API** docs | ⏳ |
| 2.10 | **Task: Create your own basic application** (e.g. tokens, Spore/DOBs); discuss with programme lead (Neon) & CKB DevRel | ⏳ |

---

### Phase 3: Further Reading

- Nervos Nation videos (UTXO, RISC-V, Cell Model, RGB++, etc.)  
- Handbook *Reading list: Understanding CKB*  

---

### Phase 4: Intermediate 

| # | Milestone | Status |
|---|-----------|--------|
| 4.1 | Script development course (Classes 1–10) | ⏳ |
| 4.2 | sUDT – standard + tutorial | ⏳ |
| 4.3 | Nervos DAO – deposit & withdraw | ⏳ |
| 4.4 | Spore Protocol / DOBs – docs & cookbook | ⏳ |

---

### Phase 5: Advanced 

| # | Milestone | Status |
|---|-----------|--------|
| 5.1 | SSRI (Script-Sourced Rich Information) | ⏳ |
| 5.2 | RGB++ Protocol | ⏳ |
| 5.3 | xUDT standard deep dive | ⏳ |
| 5.4 | iCKB | ⏳ |

---

### Capstone (Month 3)

| # | Milestone | Status |
|---|-----------|--------|
| C.1 | Design and build **capstone application** on CKB | ⏳ |
| C.2 | Showcase skills; submit per programme requirements | ⏳ |

---

## 📁 Repository Structure

This layout keeps **what you learned** (notes), **what you delivered** (reports, proof), and **what you built** (code) clearly separated—a common pattern for bootcamp logs and professional dev logs.

```
ckbuilders-weekly-report/
├── README.md                 # This file – overview and roadmap
├── notes/                    # Topic-based learning notes (one file per topic/module)
│   ├── README.md             # Index of notes + how to use this folder
│   ├── 01-introduction/      # Phase 1: CKB basics, scripts intro, Academy 1 & 2
│   ├── 02-beginner/          # Phase 2: exercises, CCC, first app
│   ├── 03-intermediate/      # Optional: sUDT, Nervos DAO, Spore
│   └── 04-advanced/          # Optional: SSRI, RGB++, xUDT, iCKB
├── weekly-reports/           # Weekly status updates (programme requirement)
├── exercises/                # Tutorial code and practice projects
├── screenshots/              # Proof of completion for exercises
└── capstone-project/         # Final capstone application
```

**Why this works well**

- **`notes/`** – One note per topic (e.g. Cell Model, Scripts, CCC basics). Use it as your single place for concepts, definitions, and “how X works” so you can review and link from weekly reports.
- **`weekly-reports/`** – Programme deliverables only; you can reference your notes here (e.g. “See `notes/01-introduction/scripts-intro.md`”).
- **`exercises/`** – Code and small projects; screenshots go in **`screenshots/`** so evidence stays in one place.
- **`capstone-project/`** – Isolated final project; keeps the main repo tidy and easy to share.

See **`notes/README.md`** for a suggested list of note files per phase so you know what to create as you go.

---

## 🚀 Capstone Project

| Field | Value |
|-------|--------|
| **Title** | [To Be Announced] |
| **Description** | A small application on Nervos CKB demonstrating skills from the track. |
| **Status** | ⏳ Planning / 🔄 In Progress / ✅ Completed |

---

## 🔄 Update Policy

This repository is updated **at least weekly** with:

- New weekly report  
- New code or screenshots as exercises are completed  
- Roadmap status changes (e.g. ⏳ → ✅)  

Updates align with the Community Keeps Building programme schedule and review cycle.

---

## 📄 License & Attribution

This repository is for **educational purposes** as part of the **Community Keeps Building** initiative (Nervos Community Catalyst).  
CKBuilder Terms and Conditions of Participation apply to programme participation and deliverables.
# ckbuilders-weekly-report
