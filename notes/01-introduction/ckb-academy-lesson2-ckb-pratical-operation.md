# CKB Academy – Lesson 2: CKB Basic Practical Operation (my notes)
 [CKB Academy – CKB basic practical operation](https://academy.nervos.org)

---

## 1. Connecting Wallet

connect **MetaMask** to **CKB testnet** in the Academy. After connecting, the Academy assigned **10 Live Cells** to my wallet, each with **100 CKB** capacity. There was about a 10-second delay, so I had to click **Fetch Cells** to see them. My **CKB address** showed up (testnet addresses start with `ckt1...`).

---

## 2. Testnet chain config (Chain Info)

A testnet (short for test network) is a practice version of a blockchain where developers can test things without using real money. in the toolbox. **PREFIX: `ckt`** means testnet.
these are some of the built in script 
| Script | What it’s for |
|--------|----------------|
| SECP256K1_BLAKE160 | Default lock script, protects ownership |
| SECP256K1_BLAKE160_MULTISIG | Multi-sig version |
| DAO | Nervos DAO contract |
| SUDT | Simple UDT (type script) |
| ANYONE_CAN_PAY | Lock variant |
| OMNILOCK | What the Academy wallet uses |

**For “Send a Transaction”** the lesson said my cells use **Omnilock**, so I need **two** cellDeps. I copied these from Chain Info:

**1) OMNILOCK (code)**

```json
{
  "outPoint": {
    "txHash": "0xec18bf0d857c981c3d1f4e17999b9b90c484b303378e94de1a57b0872f5d4602",
    "index": "0x0"
  },
  "depType": "code"
}
```

**2) SECP256K1_BLAKE160 (depGroup)**

```json
{
  "outPoint": {
    "txHash": "0xf8de3bb47d055cdf460d93a2a6e1b05f7432f9777c8c474abf4eec1d4aee5d37",
    "index": "0x0"
  },
  "depType": "depGroup"
}
```

I also saved the SCRIPTS block for reference (OMNILOCK codeHash, etc.) so I can fill the output lock and cellDeps correctly.

---

## 3. What is a transaction? 

A CKB transaction is just **spending some existing Live Cells and creating some new ones**. Simple rule: **old cells (inputs) → transaction → new cells (outputs)**.

```
[ Old Cells ]  →  [ Transaction ]  →  [ New Cells ]
   (inputs)      (signed + verified)      (outputs)
```

**Off-chain computing, on-chain verifying:** I do the work off-chain (choose inputs, build outputs, add cell deps, sign with my private key). The chain only verifies (signatures, scripts, capacity rules). So I could even prepare a tx offline and submit it later.

**Steps Involves in Transaction**
1. Choose inputs (Live Cells via OutPoints: txHash + index)
2. Build outputs (capacity, lock, optional type, data)
3. Add cell deps (scripts the tx needs, e.g. OMNILOCK + SECP256K1)
4. Sign with my private key and put the signature in **witnesses**
5. Submit; network verifies; input cells become dead, output cells become live

**In the Academy:** I dragged a Live Cell into the input box. The generated input had `previousOutput` (txHash + index) and `since: "0x0"`. There was also a cellDeps list. For outputs I set capacity, lock (codeHash, hashType, args). Rule: sum of output capacities has to be **less** than sum of input capacities so the difference is the miner fee. Output data goes in **outputsData** (often `"0x"` for a simple transfer). Signing = putting my signature in **witnesses** (inside WitnessArgs); without that the tx is invalid.

---

## 4. Send a Transaction – how I filled the template

The Academy gave me a JSON template with “Fill in the blanks here”. Here’s what I put where.

**cellDeps:** I used the two deps above (OMNILOCK first, then SECP256K1_BLAKE160).

**inputs:** I took **one of my Live Cells** from the Live Cells toolbox. I used that cell’s **tx_hash** (of the tx that created it) and **index** (often 0x0) in `previousOutput`, and left `since` as `"0x0"`. Important: if I get “No alive cells found from OutPoints”, that input was already spent or wrong – I have to Fetch Cells again and pick a current live cell.

**outputs:** I set **capacity** to less than the input so there’s a fee. E.g. for 100 CKB input I used `0x2540be018` (leaves 1000 shannons fee) so I don’t get the “min fee rate” error. For **lock** I used OMNILOCK: codeHash `0xf329effd1c475a2978453c8600e1eaf0bc2087ee093c3ee64cc96ec6847752cb`, hashType `"type"`, and **args** = my wallet’s lock args from Chain Info / from when the Academy generated a tx for me.

**outputsData:** `["0x"]` for a simple transfer.

**witnesses:** I started with `["0x"]`. After signing (see below), I replaced that with the serialized WitnessArgs so the tx is valid.

---

## 5. WitnessArgs and signing (what I did)

**WitnessArgs** has:
- **lock** – lock script args (for a simple transfer, my signature goes here; Omnilock expects OmnilockWitnessLock here)
- **input_type** / **output_type** – for type scripts; I ignored these for this exercise

**Signing flow I followed:**
1. **Generate the message** – clicked the button so the Academy builds the message to sign
2. **Sign the Transaction** – my connected wallet (MetaMask) popped up; I signed and copied the signature
3. **Serialize witnessArgs** – pasted the signature into the Academy’s input and clicked “Serialize witnessArgs”; got a long hex string
4. **Fill witnesses** – I replaced `"0x"` in the transaction’s `witnesses` array with that full serialized hex (the whole string including 0x), then clicked Save
5. **Send the Transaction** – only after witnesses were filled
6. **Check** – used “Check the block where the transaction was packaged”; if tx_status is pending I waited and tried again

**Mistakes I ran into:** If I sent with `witnesses: ["0x"]` I got TransactionFailedToVerify (error 71 – lock script failed). If I didn’t leave enough fee (output capacity too high) I got PoolRejectedTransactionByMinFeeRate. If I used an old or already-spent cell as input I got “No alive cells found from OutPoints”.

**Why tx_hash matches:** The tx_hash is computed from the unsigned tx (with empty/placeholder witnesses). After I add the real signature to witnesses, the signed tx still has the same tx_hash – that’s how CKB is designed.

---

## 6. My takeaways

| Topic | What I’ll remember |
|-------|--------------------|
| Wallet | Balance = sum of capacities of Live Cells I can unlock |
| Transaction | Inputs → signed tx → Outputs; inputs become dead |
| Off-chain / on-chain | I build and sign off-chain; chain only verifies |
| cellDeps | List of script cells the tx needs (OMNILOCK + SECP256K1_BLAKE160 for Academy wallet) |
| inputs | OutPoints of cells I’m spending + since |
| outputs + outputsData | New cells (capacity, lock) and their data |
| witnesses | My signature in WitnessArgs; must be there for lock script to pass |
| Testnet | PREFIX ckt; Chain Info has script OutPoints and I use Live Cells for input OutPoints |

Once I connected the wallet, filled the template from Chain Info and Live Cells, signed with MetaMask, put the serialized witness in `witnesses`, and sent, I completed the CKB Basic Practical Operation lesson.
