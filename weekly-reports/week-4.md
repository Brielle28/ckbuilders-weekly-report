## Builder Track Weekly Report — Week 4

**Name:** Nnadozie Clara  
**Week Ending:** 21-02-2026

---

### Courses Completed

- **Continuation and completion of Introduction to Smart Contract (Smart Contract Basics)** ([Nervos CKB Docs – Intro to Script](https://docs.nervos.org/docs/script/intro-to-script)):
  - **Inter-Process Communication (IPC) in Scripts: A Deep Dive into ckb-script-ipc Library** –
  - **Debug, Testing, Fuzzing, common error codes** – CKB-Debugger (tx file, cell index, GDB, profiling); script testing guide (Lock vs Type, inputs/outputs); fuzzing with sanitizers and corpuses
  - **Recommended Workflow** – Type ID + upgradable lock → iterate → freeze with unlockable lock; common error codes
- **View and Transfer CKB** ([Nervos CKB Docs – Transfer CKB](https://docs.nervos.org/docs/build-dapp/transfer-ckb)):
  - Ran the tutorial dApp on OffCKB devnet; viewed balance (capacityOf) and lock script (generateAccountFromPrivateKey); transferred CKB to another address (transfer: outputs, completeInputsByCapacity, completeFeeBy, sign and send)
- **Store Data on Cell** ([Nervos CKB Docs – Store Data on Cell](https://docs.nervos.org/docs/build-dapp/store-data-on-cell)):
  - Ran the tutorial dApp; wrote a message (“Hello CKB!”) into a Cell’s data field (utf8ToHex → buildMessageTx with outputsData); read the message back via tx hash and output index (readOnChainMessage, getCellLive, hexToUtf8)

---

### Key Learnings

- **Smart contract continuation (summary)**  
  **IPC:** Spawn lets scripts run child processes and talk over pipes in one tx; **ckb-script-ipc** provides a packet-based wire format (Request/Response, VLQ) and client–server model (Rust/C/JS). **Debug:** CKB-Debugger runs a script off-chain with a tx file (return code, cycles, GDB); Native Simulator for faster host-side iteration. **Upgrade workflow:** Type ID + upgradable lock (e.g. multisig) → iterate → freeze with unlockable lock; script vs dApp developer roles. **Error codes:** System (e.g. index out of bounds, item missing), Spawn/IPC (invalid FD, resource limits), Molecule serialization, ckb-script-error-codes. **Testing:** Lock tests on inputs; Type tests on burn/transfer/split/mint; cover failure cases and size limits. **Fuzzing:** Coverage-guided, corpuses from unit tests/mock tx, sanitizers (ASan, UBSan); CI or 24/7 for mature scripts.  
  → **Full detail:** [Smart Contract Basics – my notes](../notes/01-introduction/smart-contract-basics.md)

- **View and Transfer CKB (hands-on)**

  **Setup on my PC:** I have the tutorial projects inside this dev log repo. The capstone/tutorial code lives under `capstone-project/` (e.g. `simple-transfer` and `store-data-on-cell`). I did not need to clone a separate repo—these folders are part of the same repository I use for weekly reports and notes. If starting from scratch, the Handbook points to the official tutorial repos; you would clone those and then open the relevant project folder.

  **Running OffCKB and the devnet locally:** I installed the OffCKB CLI globally with `npm install -g @offckb/cli` so I could start a local CKB node and RPC from my machine. In a terminal I run `offckb node` and leave it running. OffCKB downloads and runs the CKB binary (e.g. the version required by the CLI), and the devnet RPC proxy is exposed at `http://127.0.0.1:28114`. So the “blockchain” is running locally on my PC—no testnet or mainnet needed for these tutorials. In a second terminal I run `offckb accounts` to list pre-funded accounts; each entry has a private key, public key, lock script, and address. I copied one private key (to use as sender) and one address (to use as recipient) for the Transfer CKB app.

  **Running the Transfer CKB dApp:** I navigated to the `capstone-project/simple-transfer` directory (or the equivalent path in my repo), ran `npm install` to install dependencies (including the CCC SDK), then started the app with `NETWORK=devnet npm start`. The app bound to a local port (e.g. `http://localhost:1234`). I opened that URL in my browser, pasted the sender’s private key, and triggered the action to load the account. The UI showed the account’s **address** (`ckt1...`), **lock script** (code_hash, hash_type, args), and **total capacity** (balance in CKB). That balance is the sum of capacities of all Live Cells locked by this account’s Lock Script—the CCC client queries the local devnet and sums them. I then entered the recipient address and an amount in CKB, clicked Transfer, and the app built the transaction (one output to the recipient), called `completeInputsByCapacity(signer)` to select my cells, `completeFeeBy(signer, 1000)` to reserve the fee, and `signer.sendTransaction(tx)` to sign and send to the local node. I opened the browser console to get the transaction hash and the explorer link, and verified the transaction on the devnet explorer so I could see the inputs, outputs, and that the recipient received the sent capacity.

  → **Setup, account/lock, balance, transfer flow, completing inputs/fee:** [View and Transfer CKB – my notes](../notes/02-beginner/view-and-transfer-ckb-balance.md)

- **Store Data on Cell (hands-on)**

  **Same local setup:** I used the same OffCKB devnet as above—one terminal with `offckb node` and the RPC at `http://127.0.0.1:28114`. The tutorial code is in `capstone-project/store-data-on-cell` (same repo, no extra clone). I had a pre-funded account private key from `offckb accounts`.

  **Running the Store Data on Cell dApp:** I went to the `store-data-on-cell` folder, ran `npm install && NETWORK=devnet npm start`, and opened the app in the browser (e.g. `http://localhost:1234`). I pasted my private key and loaded the account so the UI showed my address and balance. In the Write section I entered a message (e.g. `Hello CKB!`). On Write, the app encodes the text to UTF-8 bytes then to a hex string (e.g. via `utf8ToHex`), builds a transaction with one output whose lock is my address and whose `outputsData` is that hex (so the Cell’s **data** field holds arbitrary bytes—here, the message). It completes inputs and fee the same way as the transfer app, signs and sends to the local devnet, and shows the **transaction hash** in an alert; I copied and saved it. To read the message back, I went to the Read section, pasted that tx hash, left the output index as `0x0` (first output of that transaction), and clicked Read. The app called the node to fetch the Live Cell by OutPoint (txHash + index), read the Cell’s `outputData`, decoded the hex back to UTF-8 (e.g. via `hexToUtf8`), and displayed the original message in an alert. So the full loop—encode, store in a Cell on the local devnet, retrieve by OutPoint, decode—ran entirely on my PC.

  → **Function-by-function breakdown of lib.ts:** [Store Data on Cell – my notes](../notes/02-beginner/store-data-on-cell.md)

---

### Practical Progress

- Completed **Introduction to Smart Contract** reading and note-taking; summarized Intro to Script, Syscalls, Cycle Limits, VM Selection/History, Type ID, Spawn, IPC, Debug Scripts, Recommended Workflow, Error Codes, Script Testing, and Fuzzing → [Smart Contract Basics](../notes/01-introduction/smart-contract-basics.md)
- Completed **View and Transfer CKB** tutorial: started OffCKB devnet, ran `simple-transfer` dApp, loaded account with private key, viewed balance and lock script, transferred CKB to another address, verified tx in explorer
- Completed **Store Data on Cell** tutorial: ran `store-data-on-cell` dApp, wrote a message into a Cell, received tx hash, read the message back using tx hash and default index `0x0`
- Wrote detailed notes for both beginner tutorials with per-function explanations and redirects from this report

---

### Screenshots / Evidence

---

### Environment

- OffCKB devnet (`offckb node`); pre-funded accounts via `offckb accounts`
- CCC SDK; `simple-transfer` and `store-data-on-cell` capstone-project examples (`NETWORK=devnet npm start`)
- Notes: `notes/01-introduction/smart-contract-basics.md`, `notes/02-beginner/view-and-transfer-ckb-balance.md`, `notes/02-beginner/store-data-on-cell.md`
