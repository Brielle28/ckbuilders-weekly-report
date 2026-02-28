Spore is an on-chain digital object (DOB) protocol built on CKB. “On-chain” means the actual asset data is stored directly inside the blockchain, not just a reference or link. In Spore, each digital object is stored in a special Cell called a Spore Cell.
Most NFTs (like on Ethereum) are "fake" on-chain assets — they just store a link to an image hosted somewhere on the internet (IPFS, S3, etc.). If that server goes down, your NFT is gone.
Spore/CKB does it differently — the actual image bytes are stored directly on the blockchain itself. The content is the asset.

A Spore Cell contains three main parts: data, type script, and lock script. The data field includes a content-type (for example, image/jpeg) and the actual content (binary data). It can also optionally include a cluster_id. The type script identifies the Cell as a Spore object and includes a unique Spore ID in its args. The lock script defines who owns the object and who can unlock or transfer it. Once created, all fields in a Spore Cell are immutable, meaning the digital object cannot be changed.

### How It Works Step by Step
1. A "Cell" is CKB's version of a storage unit (like a UTXO but richer). A Spore Cell holds:

content-type — what kind of data it is (e.g. image/jpeg)
content — the raw bytes of the actual file
A type script that enforces the Spore rules
A lock script that controls who owns it

2. Creating a DOB — you take an image file, read it as raw bytes (ArrayBuffer → Uint8Array), then build a blockchain transaction that produces a Spore Cell with those bytes embedded in it. The Spore SDK handles the complexity of constructing that transaction.
3. Rendering it back — to display the image later, you fetch the live Cell from the blockchain using its transaction hash + output index, unpack the raw data, reconstruct a Blob from the bytes, and render it as a normal <img> tag.

To follow the tutorial, you set up a local development environment using OffCKB (devnet), Git, and the CCC JavaScript SDK. You start the local blockchain node with `offckb node`, check pre-funded accounts with `offckb accounts`, and run the example dApp with npm. The app runs locally and interacts with your devnet blockchain.

To create a digital object, the main function takes two inputs: a private key (used to sign the transaction) and the content (as a Uint8Array). In this example, the content is an image read from your computer using FileReader and converted into a byte array. The Spore SDK simplifies the process by building a transaction that creates a new Spore Cell. The transaction includes the image’s binary data and its content type, assigns ownership using the user’s lock script, and signs and sends the transaction to the blockchain. After the transaction is confirmed, the image officially exists on-chain as a digital object, and it has a unique Spore ID.

To view the digital object, you need to retrieve the Spore Cell from the blockchain. This is done by using the transaction hash and output index (together called an outpoint) to locate the live Cell. Once found, the app unpacks the raw data stored in the Cell. The binary content is then converted back into a format the browser understands by creating a Blob and generating a temporary URL. That URL is used in an <img> tag to render the image in the browser.

In summary, the Spore Protocol allows you to turn any binary data, such as a picture, into a permanent on-chain digital object on CKB. You create a Spore Cell containing the content and its type, sign and send a transaction to store it on-chain, and later retrieve and decode the data to display it. By changing the NETWORK environment variable, the same application can run on devnet, testnet, or mainnet.
