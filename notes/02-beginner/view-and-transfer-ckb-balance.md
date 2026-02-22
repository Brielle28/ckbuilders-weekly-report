# View and Transfer a CKB Balance

**Source:** [Nervos CKB Docs – View and Transfer a CKB Balance](https://docs.nervos.org/docs/build-dapp/transfer-ckb)

This note summarizes the tutorial that teaches how to build a simple dApp to view balance and transfer CKB from one account to another using the CCC SDK and OffCKB devnet.

---

## Tutorial overview

CKB uses a UTXO-like Cell Model. Each Cell has a capacity limit: that capacity represents both the CKB balance held in the Cell and how much data can be stored in it at the same time.

Transferring balance on CKB means consuming some input Cells from the sender’s account and creating new output Cells that can be unlocked by the receiver’s account. The amount transferred is the total capacity of the output Cells created for the receiver (the rest may go to change and fee). So a transfer is “spend some of my cells, create new cells locked to the recipient (and possibly a change cell for me).”

The tutorial walks through writing a small dApp that does two things: show an account’s CKB balance (view) and send CKB to another address (transfer). You need Git, OffCKB (≥ v0.4.0) for a local CKB devnet, and the JavaScript SDK CCC (≥ v0.0.14-alpha.0).

---

## Setup: devnet and running the example

**Step 1 – Download the source code.** Clone or download the tutorial repository and open the project directory (e.g. the `simple-transfer` or equivalent folder that contains the dApp code).

**Step 2 – Start the devnet.** After installing the OffCKB CLI (`npm install -g @offckb/cli`), run `offckb node` in a terminal and leave it running. The devnet RPC proxy will be available at `http://127.0.0.1:28114`. In a second terminal you can run `offckb accounts` to list pre-funded accounts; each account has a private key, public key, lock script, address, and args. You can copy a private key and an address to use as sender and receiver in the dApp.

**Step 3 – Run the example.** In the project directory run `npm install && NETWORK=devnet npm start`. The app starts at `http://localhost:1234`. You can paste a private key to load an account, see its balance and lock script, then enter a recipient address and amount and click Transfer to send CKB. Opening the browser’s developer console (F12 → Console) lets you see the transaction hash and a link to the explorer (e.g. Pudge explorer for devnet) to verify the transaction.

---

## Behind the scene: account and lock script

The core logic lives in `lib.ts`. The first important piece is turning a private key into an account that the app can use (address, lock script, public key).

The function `generateAccountFromPrivateKey` takes a private key string and returns an `Account` object. It uses the CCC SDK to create a signer from that private key: `new ccc.SignerCkbPrivateKey(cccClient, privKey)`. From the signer it obtains the address object for the standard SECP256K1 format via `signer.getAddressObjSecp256k1()`. That address object carries the Lock Script (codeHash, hashType, args) and can be converted to the string form (the `ckt1...` address). The Lock Script defines who can spend the cells: only the owner who holds the matching private key can consume Live Cells locked by this script. The tutorial uses the CKB standard Lock Script template that combines the SECP256K1 signing algorithm with the BLAKE160 hashing algorithm. Different lock templates produce different address formats and different rules for spending.

The function returns the lock script, the address string, and the public key. Once we have the Lock Script for an account, we can both derive its address and query how much balance that account has.

---

## How balance is calculated

The function `capacityOf(address: string)` returns the balance for a given address. In CKB, an account’s balance is not stored in a single “account balance” field; it is the sum of the capacities of all Live Cells that are locked by that account’s Lock Script. So the calculation is: take the address, decode it into the Lock Script (using `ccc.Address.fromString(address, cccClient)`), then ask the client for the total balance for that script (e.g. `cccClient.getBalance([addr.script])`). The CCC client queries the node for all cells matching that lock script and sums their capacities. The result is in Shannons: 1 CKB = 10^8 Shannons, similar to 1 Bitcoin = 10^8 Satoshis. The CCC SDK usually works in Shannons internally; the UI may convert to CKB for display.

---

## How the transfer works

The `transfer` function takes three parameters: the recipient’s address (`toAddress`), the amount to send in CKB (`amountInCKB`), and the sender’s private key (`signerPrivateKey`). It returns the transaction hash once the transaction is sent.

Conceptually, the transfer transaction must do the following: collect enough Live Cells from the sender (input cells) so that their total capacity covers the amount to send plus the transaction fee (and possibly a change output back to the sender). It then creates one or more output Cells whose Lock Script is the receiver’s Lock Script and whose total capacity equals the amount being sent. Any leftover capacity from the inputs, after the receiver’s amount and the fee, can be sent back to the sender as a change output. The CCC SDK hides most of this cell selection and change logic behind high-level helpers.

In code, the transfer function first creates a signer from the sender’s private key and resolves the recipient’s address to a Lock Script (`toLock`). It then builds a transaction using `ccc.Transaction.from({ outputs: [{ lock: toLock }], outputsData: [] })`. This creates a transaction that has one output so far: a cell locked to the recipient, with no data. The capacity of that output is then set to the desired amount using `ccc.fixedPointFrom(amountInCKB)`, which converts the CKB amount (e.g. a string like `"62"`) into the internal representation (Shannons). There is a check: if the output’s capacity would exceed the requested amount, the code alerts and returns; otherwise it assigns the requested amount to the output.

At this point the transaction has its outputs defined but no inputs. The next step is to complete the inputs and the fee.

---

## Completing inputs and fee

The transaction must specify which existing cells (inputs) to spend. The CCC SDK provides `tx.completeInputsByCapacity(signer)`. This call looks at the transaction’s outputs (how much capacity is being sent and to whom) and, using the signer’s Lock Script, queries the node for Live Cells that the signer can spend. It then selects enough of those cells so that their total capacity is at least the sum of the output capacities (and leaves room for the fee). Those cells are added to the transaction as inputs. So “complete the inputs” means: automatically choose which of the sender’s cells to consume so that the transfer is funded.

After that, the fee must be accounted for. The call `tx.completeFeeBy(signer, 1000)` tells the SDK to reserve 1000 Shannons (0.00001 CKB) as the transaction fee. The SDK adjusts the transaction so that the signer pays this fee, typically by reducing the change output or consuming slightly more input capacity. Without this step, the transaction might not leave enough for the miner fee and could be rejected.

Once both `completeInputsByCapacity` and `completeFeeBy` have been called, the transaction has valid inputs and outputs and is ready to be signed and sent.

---

## Signing and sending

The signer is then used to sign and submit the transaction: `const txHash = await signer.sendTransaction(tx)`. The SDK signs the transaction with the sender’s private key (producing witnesses) and sends it to the CKB node (in this setup, the OffCKB devnet proxy at localhost:28114). The node validates the transaction, runs the lock scripts, and if everything passes, includes it in a block. The function returns the transaction hash. The code also logs a link to the explorer (e.g. Pudge explorer) so you can open it in the browser and confirm the transaction. In the tutorial, opening the browser console lets you see this log and the full transaction details if the app prints them.
