## Builder Track Weekly Report — Week 10

**Name:** Nnadozie Clara
**Week Ending:** 04-04-2026

---

### Courses Completed

- **Self-directed study (no formal course completion):** Read and applied official material on **CCC** ([CCC overview](https://docs.ckbccc.com/docs/ccc/)), **CKB JS VM** ([Mechanism & capabilities](https://docs.nervos.org/docs/script/js/js-vm)), **JS quick start** ([docs](https://docs.nervos.org/docs/script/js/js-quick-start)), **Simple Lock** full-stack example ([Build a Simple Lock](https://docs.nervos.org/docs/dapp/simple-lock)), **CCC wallet / connector** ([Nervos CCC wallet](https://docs.nervos.org/docs/integrate-wallets/ccc-wallet)), and **ecosystem scripts / deployment status** ([Ecosystem scripts](https://docs.nervos.org/docs/ecosystem-scripts/introduction)).
- **Project documentation:** Wrote and iterated `**Project_Documentation.md`** for a **TypeScript + ckb-js-vm** escrow (phases, week roadmap, folder layout, day-to-day `pnpm` commands, git safety notes).

---

### Key Learnings

- `**ckb-js-vm` vs `@ckb-js-std`:** `**ckb-js-vm`** is the **on-chain QuickJS runtime** (the engine nodes run). `**@ckb-js-std`** is the **library you import** when authoring the lock script (syscalls, script args, witness, hashing, etc.).
- **Escrow on CKB:** Enforcement is **cells + lock scripts** (release / refund / timeout paths and witness layout); **CCC** is for **building and sending** txs that satisfy those rules.
- **TypeScript on-chain:** Viable via **ckb-js-vm** for learning and simpler scripts; **Rust** remains the usual recommendation for **heavy production** scripts (cycles, hardening). **Mainnet** support for JS VM must be checked against **ecosystem scripts** docs.
- **Tooling:** **OffCKB** for local devnet; `**pnpm create ckb-js-vm-app`** for on-chain TS layout; `**ckb-debugger**` required for **build bytecode** and **ckb-testtool** tests.
- **Windows / shell:** `**NODE_OPTIONS='...' jest`** breaks on **cmd** — fixed with `**cross-env`**. `**ckb-debugger**` may be on **Git Bash PATH** but not **PowerShell**. `**../../node_modules/ckb-testtool`** was wrong after monorepo move — `**node_modules/ckb-testtool**` relative to `**on-chain-script**` matches **pnpm**’s per-package links.

---

### Practical Progress

- **Repository layout (pnpm monorepo at root):** `**contracts/`** (`on-chain-script`, `on-chain-script-tests`), `**backend/**` (`escrow-backend`, `@ckb-ccc/ccc` for headless integration later), `**frontend/**` (**Vite + React**, package `**escrow-web`**, `**@ckb-ccc/connector-react**` with provider + connect wallet + basic marketing/role cards).
- **Root workspace:** `**pnpm-workspace.yaml`**, `**package.json**`, `**pnpm-lock.yaml**`, `**.gitignore**` hardened for safe `**git add .**`.
- **Docs / README:** Root `**README.md`** and `**Project_Documentation.md**` updated (tech stack, phases, folder tree, four main commands: `**pnpm install**`, `**pnpm run build:contracts**`, `**pnpm run test**`, `**pnpm run dev**`, plus optional `**pnpm run build**` / backend typecheck).
- **Environment checks:** Confirmed **Node** and `**offckb`** versions; `**ckb-debugger**` available in user’s shell for builds/tests when PATH is correct.

---

### Screenshots / Evidence

- **Capstone escrow (source code):** [Brielle28/escrow on GitHub](https://github.com/Brielle28/escrow)

---

### Environment

- **CCC Docs** — [docs.ckbccc.com](https://docs.ckbccc.com/docs/ccc/): overview, `create-ccc-app`, manual installs (`@ckb-ccc/ccc`, connector packages), transaction examples, FAQs (TypeScript `moduleResolution`, RSC `"use client"`).
- **CCC API Reference** — [api.ckbccc.com](https://api.ckbccc.com/): TypeDoc-style reference for CCC monorepo packages.
- **Nervos CKB JS VM** — [docs.nervos.org — JS VM](https://docs.nervos.org/docs/script/js/js-vm): QuickJS, args layout, when to use JS vs Rust, testnet/mainnet deployment notes.
- **Nervos — CCC wallet / React** — [CCC wallet](https://docs.nervos.org/docs/integrate-wallets/ccc-wallet): `**Provider`**, `**useCcc**`, `**useSigner**`, testnet/mainnet clients.
- **OffCKB** — [github.com/ckb-devrel/offckb](https://github.com/ckb-devrel/offckb): local devnet CLI (`offckb node`, accounts, deploy).
- **ckb-debugger** — [ckb-standalone-debugger](https://github.com/nervosnetwork/ckb-standalone-debugger): required for local script compile / **ckb-testtool** verification.

---

