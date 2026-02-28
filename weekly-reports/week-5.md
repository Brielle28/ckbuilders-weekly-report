## Builder Track Weekly Report — Week 5

**Name:** Nnadozie Clara  
**Week Ending:** 28-02-2026

---

### Courses Completed

- **Create a Fungible Token** ([Nervos CKB Docs – Create a Fungible Token](https://docs.nervos.org/docs/build-dapp/create-fungible-token)):
  - Issued a custom token using the pre-deployed **xUDT (extensible User-Defined Token)** script on OffCKB devnet; the token is represented by a Cell whose Type Script is xUDT and whose data field stores the token amount; the **issuer’s Lock Script hash** (plus placeholder) forms the **xUDT args** and acts as the unique token ID
  - Viewed issued tokens by querying Live Cells with the same xUDT Type Script (same xUDT args); identified holders by the Lock Script of each Cell
  - Transferred custom tokens to another address by building a transaction that consumes the sender’s xUDT Cells and creates new xUDT Cells locked to the receiver (and a change Cell back to the sender); used the xUDT dApp UI for Issue, View, and Transfer steps
- **Create a DOB (Digital Object)** ([Nervos CKB Docs – Create a DOB](https://docs.nervos.org/docs/build-dapp/create-dob) / Spore Protocol):
  - Learned that **Spore** stores digital objects **on-chain**: the actual content (e.g. image bytes) lives in a **Spore Cell**, not as an off-chain link; the Cell’s data holds content-type and raw content; type script enforces Spore rules and carries a unique Spore ID in args; lock script defines ownership
  - Created a digital object by uploading an image, converting it to bytes (e.g. FileReader → Uint8Array), and building a transaction that produces a Spore Cell with that content; the Spore SDK (or tutorial code) handles transaction construction, signing, and sending
  - Viewed the DOB by fetching the Spore Cell from the chain using its **OutPoint** (transaction hash + output index), unpacking the Cell data, reconstructing a Blob from the bytes, and rendering it in the browser (e.g. in an `<img>` tag)

---

### Key Learnings

- **Create a Fungible Token (xUDT)**

  On CKB, custom tokens are **User-Defined Tokens (UDTs)**. The standard used in this tutorial is **xUDT**: a minimal, extensible UDT whose script is already deployed. Unlike ERC20 or BRC20, there is no single “contract balance”; each token balance is a **Cell**. When you **issue** a token, you create a Cell with: (1) your **Lock Script** (you are the first holder), (2) the **xUDT Type Script** (args = issuer’s Lock Script hash + `"00000000"` — that hash is the **unique token ID**), and (3) **data** = token amount (e.g. 16-byte little-endian). So one Cell = one balance of one token type; the same token type is identified by the same xUDT args everywhere.

  **Issue flow:** The app takes the issuer’s private key and amount, derives the Lock Script, computes `xudtArgs = lockScript.hash() + "00000000"`, builds a Type Script with `ccc.Script.fromKnownScript(client, ccc.KnownScript.XUdt, xudtArgs)`, and creates a transaction with one output: `{ lock: issuerLock, type: xudtType }` and `outputsData: [ccc.numLeToBytes(amount, 16)]`. It adds cell deps for xUDT, completes inputs and fee, then signs and sends. After confirmation, the token exists on-chain.

  **Query (view) flow:** To see who holds the token, you build the same xUDT Type Script from the same **xUDT args** (the token ID). You then search for Live Cells with that Type Script (e.g. `cccClient.findCellsByType(typeScript, true)`). Each Cell’s Lock Script tells you who owns that balance.

  **Transfer flow:** The sender’s private key, the **xUDT args**, the amount to send, and the receiver’s address are the inputs. The app finds the sender’s Cells that match the xUDT Type Script and the sender’s Lock Script, builds outputs: one xUDT Cell to the receiver (same Type Script, receiver’s Lock Script) and, if there is leftover balance, a change Cell back to the sender. It uses helpers like `completeInputsByUdt` and then completes capacity and fee, signs and sends. So transferring = changing the **Lock Script** of the token Cell to the receiver’s Lock Script (same token type, new owner).

  → **Full note (Cell model, issue/query/transfer, xUDT args, NETWORK):** [Create a Fungible Token – my notes](../notes/02-beginner/create-fungible-token.md)

- **Create a DOB (Spore / Digital Object)**

  **Spore** is an on-chain digital object protocol on CKB. “On-chain” means the **content itself** (e.g. image bytes) is stored in the blockchain, not a URL or hash pointing elsewhere. Many NFTs elsewhere only store a link; if the server dies, the “asset” is gone. With Spore, the **content is the asset**: it lives in a **Spore Cell**.

  A **Spore Cell** has: (1) **data** — content-type (e.g. `image/jpeg`) and the raw content bytes; optionally a cluster_id; (2) **type script** — identifies the Cell as a Spore object and includes a unique **Spore ID** in its args; (3) **lock script** — who owns and can transfer the object. Once created, the Cell is **immutable**.

  **Create flow:** You provide a private key (to sign and pay) and the content (e.g. image as `Uint8Array`). The app reads the file (e.g. via FileReader), converts to bytes, then builds a transaction that creates a Spore Cell containing that content and content-type, with the user’s Lock Script. The Spore SDK (or tutorial helpers) handle building the transaction; you sign and send. After confirmation, the image (or any binary) is permanently on-chain with a unique Spore ID.

  **View flow:** To display the DOB, you need the **OutPoint** (transaction hash and output index) of the Spore Cell. You fetch the Live Cell from the chain, read its data, unpack content-type and content bytes, create a **Blob** from the bytes, generate an object URL, and use it in an `<img>` (or other element) to render the image. So: **fetch by OutPoint → decode Cell data → Blob → render**.

  → **Full note (Spore Cell structure, create/view, immutability, NETWORK):** [Create a DOB – my notes](../notes/02-beginner/create-DOB.MD)

---

### Practical Progress

- **Create a Fungible Token**
  - Started OffCKB devnet (`offckb node`) and listed pre-funded accounts (`offckb accounts`). Opened the xUDT tutorial dApp from `capstone-project/xudt` with `npm install && NETWORK=devnet npm start` and accessed it in the browser (e.g. `http://localhost:1234`).
  - **Step 1 – Issue:** Pasted an issuer private key from `offckb accounts`, entered a token amount (e.g. 42), and clicked **Issue Token**. Confirmed the transaction and copied the **Token xUDT args** from the Result section (the unique token ID for later steps).
  - **Step 2 – View:** Pasted the same xUDT args into the View section and clicked **Query Issued Token**. Verified that the issued token appeared and that the holder was the issuer’s Lock Script.
  - **Step 3 – Transfer:** Used the same issuer private key, the same xUDT args, a transfer amount (e.g. 10), and a receiver address from another `offckb accounts` entry. Clicked **Transfer Custom Token** and confirmed the transaction. Queried again in Step 2 to confirm two holders: issuer (change) and receiver.
- **Create a DOB**
  - Ran the DOB tutorial dApp from `capstone-project/create-dob` with the same devnet and `NETWORK=devnet npm start`. Used a pre-funded account private key.
  - **Create:** Selected an image file in the app; the app read it as bytes and created a Spore Cell containing that content. Signed and sent the transaction; noted the transaction hash and/or Spore ID.
  - **View:** Entered the transaction hash (and output index if required) to fetch the Spore Cell. Confirmed that the image was decoded from the Cell data and displayed in the browser, proving the content is stored on-chain.
- Wrote and updated notes for both tutorials with clear summaries and links to the official docs: [Create a Fungible Token](../notes/02-beginner/create-fungible-token.md), [Create a DOB](../notes/02-beginner/create-DOB.MD).

---

### Screenshots / Evidence

https://github.com/user-attachments/assets/904a9801-5320-4fdc-a0dc-1704e5bc7020

https://github.com/user-attachments/assets/4756902b-f55e-4e08-90eb-b25bfb8a2693

https://github.com/user-attachments/assets/f9107d4c-fcb5-4943-94a3-18296eec371a

---

### Environment

- OffCKB devnet (`offckb node`); pre-funded accounts via `offckb accounts`
- CCC SDK; `capstone-project/xudt` and `capstone-project/create-dob` with `NETWORK=devnet npm start`
- Notes: `notes/02-beginner/create-fungible-token.md`, `notes/02-beginner/create-DOB.MD`
