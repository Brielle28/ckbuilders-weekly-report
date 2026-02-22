# Store Data on Cell

**Source:** [Nervos CKB Docs – Store Data on Cell](https://docs.nervos.org/docs/build-dapp/store-data-on-cell)

This tutorial teaches how to store arbitrary data (e.g. a text message like "Hello CKB!") in the **data field** of a CKB Cell and then read it back. The Cell structure has a `data` field that can hold any bytes; here we encode text as UTF-8, convert to hex for on-chain storage, and decode when reading. The same setup as Transfer CKB is used: OffCKB devnet, CCC SDK, and a small dApp that writes a message and reads it via the transaction hash and output index.

---

## Overview

On CKB, each Cell has **capacity** (CKB balance + storage space) and can store **data** in its `data` field. This tutorial builds a transaction that **creates one new Cell** whose lock is the sender’s (so they own it) and whose **outputsData** is the message in hex. After the transaction is confirmed, we **read** that Cell by its OutPoint (txHash + output index), read the `outputData`, and decode it back to text. Encoding/decoding is up to you; the example uses the browser’s `TextEncoder` / `TextDecoder` for UTF-8.

---

## What each function in `lib.ts` does

### 1. `generateAccountFromPrivateKey(privKey: string): Promise<Account>`

**Purpose:** Turns a private key into an account object the app can use (lock script, address string, public key).

**What it does:** It creates a CCC signer from the private key (`new ccc.SignerCkbPrivateKey(cccClient, privKey)`), then gets the SECP256K1 address object with `signer.getAddressObjSecp256k1()`. That address carries the **Lock Script** (codeHash, hashType, args) used to lock cells for this account. The function returns an object with: `lockScript` (the script), `address` (the `ckt1...` string from `lock.toString()`), and `pubKey` (the signer’s public key). The app uses this to show the current account’s address and lock script and to build transactions that spend or create cells locked to this account.

---

### 2. `capacityOf(address: string): Promise<bigint>`

**Purpose:** Returns the total CKB balance (in Shannons) for a given address.

**What it does:** It parses the address string into a Lock Script with `ccc.Address.fromString(address, cccClient)`, then calls `cccClient.getBalance([addr.script])`. The client queries the node for all Live Cells whose lock matches that script and sums their capacities. The result is in **Shannons** (1 CKB = 10^8 Shannons). The app uses this to display “Total capacity” for the selected account.

---

### 3. `utf8ToHex(utf8String: string): string`

**Purpose:** Converts a plain text string (UTF-8) into a hex string suitable for the Cell `data` field.

**What it does:** It uses the browser’s `TextEncoder` to encode the string as UTF-8 bytes (`encoder.encode(utf8String)`), which gives a `Uint8Array`. It then maps each byte to a two-character hex string (e.g. `72` → `"48"` for the letter "H"), joins them, and prefixes `"0x"`. So `"Hello CKB!"` becomes a hex string like `0x48656c6c6f20434b4221`. This format is what CKB expects in `outputsData`: raw bytes represented as a hex string. Used **before** writing to chain so we store bytes, not a JavaScript string.

---

### 4. `hexToUtf8(hexString: string): string`

**Purpose:** Converts a hex string (as returned from the Cell’s data) back into a plain text string (UTF-8).

**What it does:** It takes a hex string (e.g. `0x48656c6c6f20434b4221` or without the `0x`). The regex `hexString.match(/[\da-f]{2}/gi)` splits it into pairs of hex digits; each pair is parsed with `parseInt(h, 16)` and placed in a `Uint8Array`. Then `TextDecoder("utf-8").decode(uint8Array)` turns those bytes back into a JavaScript string. Used **after** reading the Cell’s `outputData` from the chain so we can show the original message. Together with `utf8ToHex`, this defines the encoding/decoding scheme for “message in a cell.”

---

### 5. `buildMessageTx(onChainMemo: string, privateKey: string): Promise<string>`

**Purpose:** Builds, signs, and sends a transaction that creates a **new Cell** whose data field contains the given message. Returns the transaction hash so the app (or user) can later read the message from that cell.

**What it does step by step:**

1. **Encode the message:** `utf8ToHex(onChainMemo)` turns the text (e.g. `"Hello CKB!"`) into a hex string for the cell data.

2. **Create signer and lock:** It builds a signer from `privateKey` and gets the signer’s address (lock script). The new cell will be **locked to the same account** (the sender), so only they can spend it later.

3. **Build the transaction:** `ccc.Transaction.from({ outputs: [{ lock: signerAddress.script }], outputsData: [onChainMemoHex] })` creates a transaction with **one output**: a cell with the sender’s lock and **data** equal to `onChainMemoHex`. So the “bottle” (cell) is owned by the sender and contains the message bytes.

4. **Complete inputs and fee:** `tx.completeInputsByCapacity(signer)` selects Live Cells belonging to the signer so their total capacity covers the new output and fee. `tx.completeFeeBy(signer, 1000)` reserves 1000 Shannons as the transaction fee. After this, the transaction is fully built.

5. **Sign and send:** `signer.sendTransaction(tx)` signs the transaction and submits it to the CKB node (devnet). The function shows an alert with the transaction hash and returns that hash. The app can store or display this hash so the user can call **Read** later with the same tx hash.

So this function is the “Write” path: **message + private key → one new on-chain cell containing that message → tx hash**.

---

### 6. `readOnChainMessage(txHash: string, index = "0x0"): Promise<string | void>`

**Purpose:** Fetches a specific Live Cell by its OutPoint (transaction hash + output index), reads its `data` field, decodes it from hex to UTF-8 text, and shows it (e.g. in an alert). Returns the decoded message string.

**What it does:**

1. **Locate the cell:** It calls `cccClient.getCellLive({ txHash, index }, true)`. The **OutPoint** is the pair (txHash, index): the transaction that created the cell and the position of the output in that transaction (0 = first output). In this tutorial we only create one output (the message cell), so that cell is always at index `0x0`. The second parameter (`true`) typically means “include output data” so we get the stored bytes.

2. **Handle missing cell:** If the cell is not found (e.g. tx not confirmed yet or wrong hash), the client returns `null`. The function then shows an alert (“cell not found, please retry later”) and returns.

3. **Read and decode:** If the cell exists, `cell.outputData` is the raw data as stored on-chain (hex string). The function passes it to `hexToUtf8(data)` to get back the original text and shows it in an alert (`"read msg: " + msg`) and returns the message string.

So this function is the “Read” path: **tx hash (and optionally index) → fetch cell → decode data → show message**. The user must have the tx hash from the earlier **Write** (buildMessageTx) to read the message they just stored.

---

### 7. `shannonToCKB(amount: bigint): bigint`

**Purpose:** Converts an amount from **Shannons** to **CKB** for display (1 CKB = 10^8 Shannons).

**What it does:** It divides the amount by `100_000_000` (10^8) and returns the result. So for example `4199993700000000n` Shannons becomes `41999937` CKB. The app uses this to show “Total capacity” in CKB instead of raw Shannons. Purely a display helper; it does not touch the chain.

---

## Flow summary

**Write:** User enters message and private key → `buildMessageTx` encodes message with `utf8ToHex`, builds a tx with one output (lock = sender, data = hex message), completes inputs and fee, signs and sends → returns tx hash → user can copy or the app can store it.

**Read:** User provides tx hash (and optionally index, default `0x0`) → `readOnChainMessage` calls `getCellLive` with that OutPoint, gets `outputData`, decodes with `hexToUtf8`, and shows the message.

**Helpers:** `generateAccountFromPrivateKey` and `capacityOf` support the UI (show address, lock script, balance). `utf8ToHex` and `hexToUtf8` define the on-chain format for the message. `shannonToCKB` is for displaying balance in CKB.

---

## Recap

- The **data field** of a Cell can hold any bytes; we store text as UTF-8 bytes in hex form.
- **Encoding/decoding** is our choice; here we use `TextEncoder`/`TextDecoder` and hex.
- To **write:** build a transaction whose **outputsData** is the encoded message; complete inputs and fee, sign and send; keep the **tx hash**.
- To **read:** get the Live Cell by **OutPoint** (txHash + index) via RPC, then read and decode **outputData**.
- The functions in `lib.ts` implement this: account/balance helpers, encode/decode, buildMessageTx (write), readOnChainMessage (read), and a display helper for CKB units.
