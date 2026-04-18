## Builder Track Weekly Report — Week 12

**Name:** Nnadozie Clara
**Week Ending:** 04-19-2026

---

### Courses Completed

- **OffCKB local devnet operations**
  - Starting local chain and validating RPC (`offckb node`, tip checks)
  - [OffCKB docs](https://docs.nervos.org/docs/sdk-and-devtool/offckb)
- **Nervos JS VM deployment flow**
  - Build/deploy lifecycle for JS bytecode contracts (`index.bc`)
  - [CKB JS VM — mechanism & capabilities](https://docs.nervos.org/docs/script/js/js-vm)
  - [JavaScript / TypeScript quick start](https://docs.nervos.org/docs/script/js/js-quick-start)
- **CCC integration prep**
  - Local env + deployment metadata + script dependency wiring
  - [CCC overview](https://docs.ckbccc.com/docs/ccc/)

---

### Key Learnings

- **Deploy means “publish contract bytes to chain state”**
  - Building `index.bc` is local compilation; deploying creates an on-chain cell reference used by transactions.
- **JS VM contracts need structured dependencies**
  - Execution requires `ckb_js_vm` script info + deployed bytecode reference + correct args layout.
- **Metadata drift can break integration quickly**
  - If devnet state changes/restarts, stale outpoints in `scripts.json` can cause `Unknown(OutPoint(...))`.
- **CLI placeholders and shell syntax matter on Windows**
  - Angle brackets (`<...>`) in commands are placeholders in docs, not literal input in cmd/bash usage.
- **Fee-rate fetch on local RPC can be null**
  - CCC `getFeeRate` path may fail on local setup; using explicit fee rate in `completeFee...` avoids this path.

---

### Practical Progress

- **Phase A completed (local chain + account + RPC)**
  - OffCKB node running locally and RPC verified via curl.
  - Environment captured in `backend/.env.local` / template in `backend/.env.example`.
- **Phase B completed (build artifacts)**
  - `pnpm run build:contracts` generates `contracts/on-chain-script/dist/index.bc`.
- **Phase C completed (deploy + metadata)**
  - Deployed `index.bc` to devnet and generated `deployment/scripts.json`.
  - Current bytecode dep outpoint recorded in `deployment/scripts.json`:
    - `txHash`: `0x1e8c4834c17f98d3db7d5842191b2e3dcb8617194bfe7d88d0ce83ad7b288e98`
    - `index`: `0`
  - Exported system scripts for CCC:
    - `deployment/system-scripts.devnet.json`
    - includes `ckb_js_vm` script metadata and dep.
- **Phase D implementation created under `backend/src/integration/`**
  - `runIntegration.ts` runner added with mode switch:
    - `INTEGRATION_SPEND_MODE=release|dispute|timeout`
  - Helpers added:
    - `deployment.ts`, `cellDeps.ts`, `escrowPayload.ts`, `loadEnv.ts`, `paths.ts`
  - Package scripts added:
    - root: `pnpm test:integration`
    - backend: `pnpm integration:devnet`
- **Fix applied during debugging**
  - `runIntegration.ts` funding path now uses explicit fee rate:
    - `completeFeeChangeToLock(..., 1000n)` (line near 224)
  - This addressed the earlier `Cannot destructure property 'mean' of 'object null'` fee-rate error.

---

### Current Blocker

- **Integration run still failing on live send with unknown outpoint resolution**
  - Error observed:
    - `TransactionFailedToResolve: Resolve failed Unknown(OutPoint(...))`
  - Failure point:
    - `backend/src/integration/runIntegration.ts` around sending funded tx (`sendTransactionNoCache`, around line 232).
  - Most likely cause:
    - Transaction references an outpoint not resolvable by current chain state (stale/mismatched dep or changed local chain state).

---

### Screenshots / Evidence

https://github.com/user-attachments/assets/aac6daae-af1b-4bfa-b9cf-a4b05c1b809d

https://github.com/user-attachments/assets/c78d813c-daf1-45aa-b23c-7ac696f57bfb
| What                                      | Evidence                                                                            |
| ----------------------------------------- | ----------------------------------------------------------------------------------- |
| OffCKB deploy success                     | terminal output + generated `deployment/scripts.json`                               |
| System scripts export                     | `offckb system-scripts --export-style ccc -o deployment/system-scripts.devnet.json` |
| Integration failure #1 (fee stats null)   | terminal stack trace at `runIntegration.ts:224`                                     |
| Integration failure #2 (unknown outpoint) | terminal stack trace at `runIntegration.ts:232`                                     |


---

### Environment

- **OS/Shell:** Windows 10 + cmd / git-bash
- **Workspace:** pnpm monorepo (`contracts`, `backend`, `frontend`)
- **Backend integration stack:** `@ckb-ccc/core`, `@ckb-ccc/ccc`, `tsx`, `dotenv`, `@noble/secp256k1`
- **Node toolchain:** OffCKB CLI, local devnet RPC (`http://127.0.0.1:28114`)

---

### Plan for next week (Week 13)

- Stabilize Phase D transaction assembly by validating dep/outpoint consistency against current devnet tip.
- Add preflight checks in runner:
  - verify all `cellDeps` outpoints are live before sending tx.
- Complete all four scenarios end-to-end and record tx hashes:
  - D1 lock, D2 release, D3 timeout refund, D4 dispute refund.
- Add regression output file (JSON) for tx hashes and mode result.
- Optional stretch: repeat same scenarios on public testnet once local flow is green.

