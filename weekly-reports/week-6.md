## Builder Track Weekly Report — Week 6

**Name:** Nnadozie Clara  
**Week Ending:** 07-03-2026

---

### Courses Completed

- **Build a Simple Lock** ([Nervos CKB Docs – Build a Simple Lock](https://docs.nervos.org/docs/build-dapp/build-a-simple-lock)):
  - Implemented a custom **hash_lock** lock script in TypeScript using `@ckb-js-std/core` and `@ckb-js-std/bindings`, and compiled it to bytecode that runs under `ckb-js-vm`
  - Built and deployed the script to OffCKB devnet using the project’s `pnpm build` and `pnpm run deploy --network devnet` workflow, producing `hash-lock.bc` and `scripts.json` deployment metadata
  - Ran the frontend dApp, generated a hash_lock-based CKB address from a chosen hash, deposited CKB into that address, and successfully **unlocked** / transferred CKB by providing the correct preimage (and observed failure when the preimage was wrong)

---

### Key Learnings

- **Hash-based custom lock (hash_lock)**  

  The hash_lock script is a simple example of a **custom lock**: instead of unlocking with a signature, it unlocks with knowledge of a **preimage**. The script logic (in `contracts/hash-lock/src/index.ts`) does the following:

  - Loads the current Script and reads its **args**; the expected hash is encoded in the tail of the args:
    - `const expect_hash = new Uint8Array(HighLevel.loadScript().args).slice(35);`
  - Loads the **witness** for the first group input (`HighLevel.loadWitnessArgs(0, bindings.SOURCE_GROUP_INPUT)`), and takes the `lock` field as the **preimage**
  - Hashes the preimage with CKB’s default hash (`hashCkb(preimage)`, blake2b-256) and compares it with `expect_hash` using `bytesEq`
  - Returns **0** if the hashes match (unlock success), or **11** and logs an error if they don’t

  This shows how Scripts on CKB read **script args**, access **witness data**, and implement their own unlocking rule — all inside TypeScript, compiled and run in `ckb-js-vm`.

- **Build and deployment pipeline**

  The `scripts/build-contract.js` script implements a simple build pipeline for JS/TS contracts:

  - Locates the contract source under `contracts/<contract-name>/src/index.ts` or `.js`
  - Bundles it with **esbuild** into a single JavaScript file in `dist/<contract-name>.js`
  - Calls `ckb-debugger` with the `ckb-js-vm` binary to compile that JS into **bytecode** (`dist/<contract-name>.bc`)

  Then `pnpm run deploy --network devnet` publishes the bytecode to devnet and writes deployment artifacts (including `hash-lock.bc` `codeHash`, `hashType`, and cellDeps) into `deployment/scripts.json`. These values are later consumed by the frontend to construct the correct lock script.

- **Using the hash_lock in a dApp (frontend)**  

  In `frontend/app/hash-lock.ts`:

  - `generateAccount(hash: string)` builds a **hash_lock Script** from a chosen hash:
    - Constructs `lockArgs` by packing:
      - a small prefix (`0x0000`)
      - the `codeHash` and `hashType` of the deployed `hash-lock.bc` script
      - the user-supplied hash that the preimage must match
    - Wraps this in a lock script whose `codeHash` and `hashType` point to `ckb_js_vm`, because the JS contract runs under `ckb-js-vm`
    - Derives a CKB **address** from the lock script via `ccc.Address.fromScript`, so you can use it like any other CKB address (deposit CKB, query capacity)

  - `capacityOf(address)` uses `cccClient.getBalance([addr.script])` to sum capacities of all Live Cells with that lock script — “how much CKB is in this hash_lock safe?”.

  - `unlock(fromAddr, toAddr, amountInCKB)` builds a transfer transaction:
    - Resolves `fromAddr` and `toAddr` into lock scripts
    - Creates a transaction with an output locked by the **receiver’s** script
    - Adds the `hash-lock` and `ckb_js_vm` cellDeps and completes inputs by capacity using a **read-only signer** tied to the hash_lock script
    - Prompts the user for the **preimage**, converts it to bytes, and sets it in the witness lock field (using `ccc.WitnessArgs` and `tx.setWitnessArgsAt(0, newWitnessArgs)`)
    - Sends the transaction; if the supplied preimage’s hash matches the hash embedded in the script args, the script returns 0 and the unlock succeeds

  This ties together **on-chain logic** (hash check) with **frontend UX** (prompt for preimage) over the same lock script definition.

- **Security limitations**

  The tutorial emphasizes that hash_lock is **not safe for real funds**:

  - The preimage is revealed in the **witness**, so a miner (or anyone) seeing your transaction can front‑run it and send the funds elsewhere
  - Once any transaction from that hash_lock address is on-chain, the preimage is public and all remaining funds locked with that hash can be stolen

  The purpose of this example is to practice building and wiring up a custom lock, not to design a production‑ready scheme.

---

### Practical Progress

- Set up and confirmed **OffCKB devnet** running with `offckb node`, and used `offckb accounts` to get pre‑funded accounts for testing.

- downloaded the ckb debugger 

- cloned the simple-lock repository

- In the `simple-lock` project:
  - Ran `pnpm install` at the root to install dependencies.
  - Built the **hash_lock** contract with the project’s scripts:
    - Verified the TypeScript implementation in `contracts/hash-lock/src/index.ts` (using `HighLevel.loadScript`, `HighLevel.loadWitnessArgs`, `hashCkb`, `bytesEq`).
    - Used the build script (via `pnpm build` or its underlying `build-contract.js`) to produce `dist/hash-lock.js` and `dist/hash-lock.bc`.
  - Deployed the compiled contract to devnet with:
    - `pnpm run deploy --network devnet`
    - Confirmed that deployment artifacts (including `hash-lock.bc` codeHash, hashType, cellDep) were written to the deployment folder.
---

### Screenshots / Evidence

https://github.com/user-attachments/assets/6c2703b4-4b2c-4a39-af2b-5ba205d2f7f3

https://github.com/user-attachments/assets/07c74f44-2a97-4fc6-bc70-97fac1261a7a

---

### Environment

- OffCKB devnet (`offckb node`); pre‑funded accounts via `offckb accounts`  
- `capstone-project/simple-lock`:
  - Contracts built with `pnpm install`, `pnpm build` (`build-contract.js` + `ckb-debugger`/`ckb-js-vm`)
  - Deployment via `pnpm run deploy --network devnet`  
  - Frontend under `simple-lock/frontend` with `pnpm i && pnpm run dev` (served at `http://localhost:3000`)
- JavaScript/TypeScript stack using `@ckb-js-std/core`, `@ckb-js-std/bindings`, and `@ckb-ccc/core` for on-chain logic and dApp integration.