## Builder Track Weekly Report — Week 14

**Name:** Nnadozie Clara  
**Week Ending:** 05-02-2026  

**Repository (all code & evidence lives here):** [https://github.com/Brielle28/escrow](https://github.com/Brielle28/escrow)

---

### This week

We’re still on the Nervos CKB escrow: funds stay on-chain until someone releases, the clock runs out and a refund happens, or a dispute path opens—same idea as before, with the rules in the on-chain script.

The big move was **dropping the JS (ckb-js-vm) contract and porting the escrow logic to Rust**. The old contract packages are gone. We got the full loop working locally: build the contract, deploy to OffCKB’s devnet, then run the headless integration tests that fire real txs through CCC from Node (same stack you’d use from a wallet, just without a UI).

There was a gnarly bug in the Rust verifier: we were verifying signatures the wrong way at first (see below). After fixing that, redeploying, and re-running tests, **fund + spend** worked for all three automated paths—release, dispute, and timeout.

---

### How the pieces fit

| Piece | In practice |
|-------|-------------|
| Rust on-chain | Escrow lives in `contracts/escrow-rust`, built for RISC-V so CKB-VM can run it. |
| No JS contract in repo anymore | Removed `contracts/on-chain-script` and `contracts/on-chain-script-tests`. One source of truth: Rust. |
| CCC / backend | `backend/src/integration/runIntegration.ts` uses `@ckb-ccc/core` to build and send txs. The Rust doesn’t run on your machine for that—it runs on the nodes when the tx hits the chain. |
| After deploy | `offckb deploy` drops script hashes and cell deps under `deployment/`, mainly `deployment/scripts.json`. OffCKB names the entry `escrow-rust`; the backend accepts `escrow` or `escrow-rust`. |
| Proof it works | `pnpm prep:devnet` (with `offckb node`) and `pnpm test:integration` both passed here after the signature fix. |

---

### What changed in the repo

- **On-chain:** `contracts/escrow-rust/` — type script, `cargo build --target riscv64imac-unknown-none-elf`, binary name `escrow-rust`. Root `pnpm run build:contracts` now builds this instead of the old JS bundle.
- **Cleanup:** Pulled out the old on-chain script packages. Root `package.json` now points deploy at the Rust binary; `pnpm test` runs `cargo test` for `escrow-rust`.
- **Backend:** `runIntegration.ts` loads the deployed script from `deployment/scripts.json` and only pulls the cell deps the contract needs (no JS VM layer). `deployment.ts` has `escrowEntryForChain()` so `escrow` and `escrow-rust` both resolve.
- **The signature bug:** `k256`’s default `verify()` was **hashing the message again** with SHA-256. CKB signs the raw 32-byte tx hash, so we were double-hashing. Fix was prehash verification plus low-S normalization; after that, spends started passing.

---

### Devnet evidence (local, after fix + redeploy)

From `pnpm test:integration` (default `INTEGRATION_SPEND_MODE=release`) plus two more runs for `dispute` and `timeout`. These are **local devnet** txs, not public testnet, but they show the full path working on-chain.

| Mode | Fund (lock) | Spend |
|------|-------------|--------|
| release | `0x9111e27d8296931f612e032d3e879ad31967455461143522274bb9406d04b82e` | `0x5803cf01e09800afb1ac1140e3fa418aa76f90f8b83c1973f01309855cb2b593` |
| dispute | `0x9f3c8010a27737a030c3c1470b6f5fef0b3ec383741d4633281036a2df625c78` | `0xa7148a636e49584a0ff6106e2d9cdbd57d840a188b88cd8e95bad166f20c9c32` |
| timeout | `0xd1f2336c1885a2cefe52dfefcaf88e724c8459aec8d69b9a6c11ee6aa61f592a` | `0x6ca115688c06e28bc8ebcb22973c3d988118f37e9673e231b93f3ca4e6a8c831` |

Successful runs also append to `artifacts/integration-tx-history.jsonl` so we have a simple audit trail.

---

### Notes

- On-chain code and the code that *sends* txs don’t have to be the same language: Rust on CKB, TypeScript + CCC on the machine that drives txs.
- **Signature rules have to match CKB’s hashing.** If you hash twice by accident, verification fails even when the rest of the story looks fine.
- Deploy names matter: we accept both `escrow` and `escrow-rust` so deploy output and the backend don’t fight.

---

### Next week (Week 15) — UI, chain, wiring

Hook up what already works in scripts to something people can actually use.

**`frontend/` is the main target:** deposit, release, refund, dispute with `@ckb-ccc/connector-react`, clear states, and explorer links where it helps. Same script metadata and spend flow as the headless runner so we don’t ship two different behaviors. If duplication gets painful, we can pull a small shared module or API later.

No extra Rust or a dedicated backend API unless the browser path shows a gap the current integration can’t cover.

**Goal for the next report:** escrow actions from the browser against devnet or testnet, with tx hashes captured the same way we did for the integration runs.

---

### Environment

- Monorepo with pnpm: `contracts/escrow-rust`, `backend`, `frontend`.
- Local chain: OffCKB devnet RPC often at `http://127.0.0.1:28114`.
- Integration: `backend/.env.local` (gitignored) — `CKB_RPC_URL`, `DEPLOYER_PRIVATE_KEY` (devnet genesis keys, local only).
