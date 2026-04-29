## Builder Track Weekly Report — Week 13

**Name:** Nnadozie Clara  
**Week Ending:** 04-25-2026

---

### Courses Completed

- **CKB public testnet (Pudge) deployment**
  - OffCKB `--network testnet`, deploy costs, and keeping **`deployment/scripts.json`** in sync with **`testnet.index.bc`**
  - [OffCKB docs](https://docs.nervos.org/docs/sdk-and-devtool/offckb)
- **Testnet system scripts for CCC**
  - Exporting **`deployment/system-scripts.testnet.json`** with **`ckb_js_vm`** deps aligned to testnet genesis (not devnet outpoints)
  - [CCC overview](https://docs.ckbccc.com/docs/ccc/)
- **Integration runner — multi-network**
  - **`CKB_NETWORK=devnet|testnet`**, RPC via **`CKB_RPC_URL`**, explorer evidence on **Pudge**

---

### Key Learnings

- **Deploy on testnet needs real CKB**
  - Default OffCKB keys may be unfunded for **`pnpm deploy:testnet`**; the machine run failed once with **`Insufficient CKB`** until a **funded testnet wallet** was used.
- **Never paste doc placeholders as secrets**
  - **`0xYOUR_FUNDED_KEY`** (or angle-bracket placeholders) produces **`Invalid bytes`**; use a real **`0x…`** key or **`pnpm prep:testnet:env`** so **`DEPLOYER_PRIVATE_KEY`** comes from **`backend/.env.local`** without putting the key on the CLI.
- **Fund vs spend are two transactions**
  - Explorer shows **inputs and outputs for one tx hash**; **locking** escrow and **releasing** it are **two hashes** → two **`[explorer]`** lines in the integration output.
- **Testnet metadata must match the chain**
  - If **`deployment/scripts.json`** lacks **`testnet["index.bc"]`**, integration fails until **`pnpm prep:testnet`** (or **`prep:testnet:env`**) completes after a successful deploy.
- **Per-network script resolution**
  - Devnet-only secp tweaks do not apply on public testnet; **`always_success`** may need **`getKnownScript`** when the testnet export omits it under **`testnet`**.

---

### Practical Progress

- **Phase A — testnet deploy + artifacts**
  - **`pnpm prep:testnet:env`** (or equivalent): **`build:contracts`** → **`deploy:testnet:env`** (reads **`DEPLOYER_PRIVATE_KEY`** from **`backend/.env.local`**) → **`system-scripts:testnet`**.
  - **`deployment/system-scripts.testnet.json`** generated for CCC.
  - **`deployment/scripts.json`** includes a live **`testnet.index.bc`** cell dep after deploy.

- **Phase B — backend integration on testnet**
  - **`CKB_NETWORK=testnet`** path in **`backend/src/integration/runIntegration.ts`**: **`testnet`** section of **`scripts.json`**, **`system-scripts.testnet.json`**, no devnet secp patch on testnet.
  - **`pnpm test:integration:testnet`** forces testnet via **`backend/scripts/integration-testnet.mjs`**.
  - **`artifacts/integration-tx-history.jsonl`** records runs with **`"network":"ckb-testnet"`** and **`rpcHost`** reflecting **`https://testnet.ckb.dev`** (not local **`127.0.0.1:28114`**).

- **Phase C — verified end-to-end on Pudge (release mode)**
  - Successful integration line ( **`mode":"release"` ):
    - **Fund tx:** `0x93a7ce7b1dd1e6067f30a98e9c745ad1f26616872608f972196e9b936cb57233`
    - **Spend tx:** `0x4b80cb51036af4f530bade9a66763fb6e876a1cc4852ff4fc05d02b5ff513f5b`
  - Earlier failed run documented **`missing testnet["index.bc"]`** until prep/deploy was complete (**`INTEGRATION_SPEND_MODE`** **`timeout`** in that failure record).

---

### Current Blocker

- **None for core Week 4 release-path demo** — **release** succeeded on **ckb-testnet** with explorer-verifiable hashes.
- **Follow-up (optional):** Run **`INTEGRATION_SPEND_MODE=dispute`** and **`timeout`** against testnet and paste hashes into **`artifacts/WEEK4_TESTNET_STATUS.md`** so all four scenarios match the Week 4 spec checklist.

---

### Screenshots / Evidence

https://github.com/user-attachments/assets/a2702598-2821-4e80-9b5b-31f4d78a30cf

https://github.com/user-attachments/assets/a9b75e4c-f49f-4298-9f7c-c96b60a701c7

---

### Environment

- **Workspace:** pnpm monorepo (`contracts`, `backend`, `frontend`)  
- **Testnet RPC:** **`https://testnet.ckb.dev/rpc`** (recorded in **`backend/.env.local`** for integration)  
- **Backend:** `@ckb-ccc/core`, `@ckb-ccc/ccc`, `tsx`, `dotenv`

---

### Plan for next week (Week 14)

- Complete **dispute** and **timeout** integration runs on **testnet** (same env + **`INTEGRATION_SPEND_MODE`**) and capture hashes in **`WEEK4_TESTNET_STATUS.md`**.
- **Week 5 (frontend):** connect UI to backend / testnet config as scoped by the project plan.
