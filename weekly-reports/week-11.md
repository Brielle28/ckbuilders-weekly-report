## Builder Track Weekly Report — Week 11

**Name:** Nnadozie Clara
**Week Ending:** 04-11-2026

---

### Courses Completed

- **Nervos CKB — JavaScript / TypeScript on CKB-VM (`ckb-js-vm`):**
  - Lock script structure, `args` vs witness, and running scripts under the JS VM
  - [CKB JS VM — mechanism & capabilities](https://docs.nervos.org/docs/script/js/js-vm)
  - [JavaScript / TypeScript quick start](https://docs.nervos.org/docs/script/js/js-quick-start)
- **Project documentation (repo):**
  - Escrow roles (depositor, recipient, arbiter), three spend paths, and Week 2–3 roadmap
  - `Project_Documentation.md` (escrow design and workspace layout)
- **CCC (`@ckb-ccc/core`) & local script testing:**
  - Building transactions and computing `tx.hash()` for signing messages in tests
  - [CCC overview](https://docs.ckbccc.com/docs/ccc/)

---

### Key Learnings

- **How our escrow should behave**
  - We picked a simple rule set:
    - If everything goes well, money goes to the **seller**.
    - If there is a dispute or timeout, money goes back to the **buyer**.
  - In short: seller gets paid on success, buyer gets refunded on failed or expired deals.

- **How approvals work**
  - People must “approve” with digital signatures before money can move.
  - For two-person approval paths, both sign the same transaction data so the script can verify they both agreed.

- **How timeout works**
  - Timeout uses the transaction’s `since` value.
  - If the transaction is not old enough, refund-by-timeout is rejected.
  - If the time condition is met, the timeout refund path can proceed.

- **What we learned from testing setup**
  - In our current test setup, escrow logic runs inside the JS VM as a script in the transaction.
  - We must use the correct built-in CKB constants when reading transaction data; hardcoded numbers cause failures.

- **Windows setup lesson**
  - Tests/build need `ckb-debugger` available in PATH.
  - Keeping a local copy in `tools/` and calling it from scripts makes setup more reliable.


### Practical Progress

- **Escrow lock script implemented** in `contracts/on-chain-script/src/index.ts`:
  - `main()` — dispatches witness tags: release (1), dispute (2), timeout (3).
  - `parseEscrowPayload()` — reads trailing **169-byte** payload from `lock.args` (pubkey hashes, recipient/depositor destination lock hashes, `min_since`, version).
  - `escrowSigningMessage()` — BLAKE2b hash of raw transaction bytes for signing.
  - `verifySigAndPubkeyHash()` — ECDSA verify + pubkey hash match.
  - `lockScriptHashFromOutput0()` — hashes output 0 lock for payout/refund checks.
  - `Err` — numeric error codes for args / witness / signature / output / since failures.
- **Contract build fixed for Windows** in `contracts/on-chain-script/package.json`:
  - `build:release` / `start` invoke `..\..\tools\ckb-debugger.exe` to compile `dist/index.js` → `dist/index.bc`.
- **Local bytecode build verified** — `pnpm run build:contracts` succeeds and produces `contracts/on-chain-script/dist/index.bc`.
- **Unit / VM integration tests** in `contracts/on-chain-script-tests/src/index.test.ts`:
  - Release path: recipient + arbiter signatures; output 0 locked to **recipient** script.
  - Dispute path: depositor + arbiter signatures; output 0 locked to **depositor** script.
  - Timeout path: depositor signature; `since >= min_since`; output 0 locked to **depositor** script.
- **Test runner + debugger PATH** — `contracts/on-chain-script-tests/run-jest.cjs` prepends repo `tools/` to `PATH`; `contracts/on-chain-script-tests/package.json` test script runs it; `jest.config.cjs` uses standard `ts-jest` + Node.
- **Signing in tests** — `@noble/secp256k1` with `hmacSha256Sync` wired via Node `crypto` for `signSync`; **`{ der: false }`** for compact signatures.
- **Automated verification** — `pnpm run test` passes (3 tests).

---

### Screenshots / Evidence
- **Capstone escrow (source code):** [Brielle28/escrow on GitHub](https://github.com/Brielle28/escrow)


---

### Environment

- **pnpm** workspace (`contracts/*`, `frontend`, `backend`).
- **TypeScript** — `contracts/on-chain-script` builds with `tsc` + `esbuild` + `ckb-debugger`.
- **`ckb-debugger`** — Windows binary under `tools/ckb-debugger.exe` (see `contracts/on-chain-script/package.json`).
- **Jest + `ckb-testtool`** — `contracts/on-chain-script-tests`; tests shell out to `ckb-debugger` via PATH set in `run-jest.cjs`.
- **OffCKB / local node** — not required for Week 2 VM tests; planned for Week 3.

---

### Plan for next week (Week 3)

- **Run a local “real” chain (local testnet)** using **OffCKB** (`offckb node`) so blocks, RPC, and deployment behave like production devnet.
- **End-to-end CCC flows** in `backend/` (or small scripts):
- **Optional:** repeat the same four scenarios on **public testnet** once local devnet is stable.
