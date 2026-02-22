# Smart Contract Basics (Intro to Script & Beyond)

---

## Intro to Script

A Script in Nervos CKB is a binary executable that can be executed on-chain. It can perform arbitrary logic to guard and protect your on-chain assets. You can think of it as smart contract.

### How a Script Works

When executing a Script, CKB takes the executables and runs them in a virtual machine environment called CKB-VM. After the execution, if the program returns a code of 0, we consider the Script successful; any non-zero return codes will be considered Script failures.

### Script Types

A Script can be one of two types:

- **Lock Script** – Used to control ownership and access to a Cell.
- **Type Script** – Used to control how a Cell is used in a transaction.

In most cases, Lock Script and Type Script function similarly. The only key difference is that, the output Cells' Lock Scripts will NOT be executed in a transaction, whereas the input Cells' Lock Scripts, the input Cells' Type Scripts, and the output Cells' Type Scripts will be executed.

### Script Structure

Script has the following structure:

```rust
pub struct Script {
    pub code_hash: H256,
    pub hash_type: ScriptHashType,
    pub args: JsonBytes,
}
```

Each Script consists of:

- **code_hash:** Identifies the Script code.
- **hash_type:** Defines how to interpret code_hash when locating the Script code.
- **args:** Parameters passed to the Script, used to customize behavior.

The combination of code_hash and hash_type tells CKB how to locate the Script code:

- **data, data1, data2** (data hash): Match the hash of the Script binary directly.
- **type** (type hash): Matches the hash of a Cell's Type Script.

For a deeper comparison, see Data Hash vs. Type Hash.

A Script also includes the args part, which differentiates one Script from another using the same Script code. The args can provide additional arguments for a CKB Script. For example, while multiple users might utilize the same default Lock Script code, each user can have their own public key hash stored in args. This setup allows each user to have a unique Lock Script while sharing the same Lock Script code.

### Programming language used for script

Script can be written in any language as long as you have the proper toolchain. However, at the current stage, we recommend that you use Rust to write CKB Scripts since it has a complete toolchain to work with CKB.

---

## Invoke Scripts via Syscalls

CKB scripts are small programs that run on the blockchain. They cannot access the outside world, so they use syscalls to ask the blockchain for information.

On CKB, every transaction is checked by running small programs called scripts. These scripts do not run on your computer or phone. They run inside a special virtual machine called CKB-VM, which is built into the blockchain. Every node in the network runs the same script in the same way to decide whether a transaction is valid.

Because scripts run in a restricted environment, they cannot freely access memory, files, or the internet. Instead, they use special system functions called syscalls. A syscall is like asking the blockchain: "Please give me this data" or "Please do this action for me."

There are so many types of syscalls. Some syscalls let a script stop running, check which VM version is being used, or see how much computing power (cycles) it has used. Others allow the script to read information from the transaction, such as the whole transaction, the current script, the transaction hash, or data inside cells. Some syscalls are used for debugging, so developers can print messages while testing.

There are also syscalls that allow advanced behavior, such as running another script inside the current script (spawn), creating communication pipes, and sending data between scripts. This makes it possible to build more complex smart contracts that behave like small programs with multiple parts.

There are also return codes, which are numbers that tell whether a syscall succeeded or failed. For example, "success," "index out of bounds," or "invalid data." Scripts use these codes to know if something went wrong.

Sources, which tell the syscall where to read data from. A script can read from input cells, output cells, dependency cells, or header dependencies. This controls which part of the transaction the script is allowed to inspect.

Then it describes cell fields, header fields, and input fields. These are specific pieces of data that a script can request, such as how much CKB a cell holds, the lock script, the type script, the data hash, or block information like epoch number. Instead of loading everything, the script can ask only for the exact field it needs, which saves computing power.

---

## Regulate Scripts via Cycle Limits

On CKB, every smart contract runs inside a virtual machine called CKB-VM. Because this VM supports loops, branches, and complex logic, a bad or poorly written script could run endlessly and slow down the network. To prevent this, CKB uses something called cycles. This is done so that scripts cannot abuse the system by running forever or using too much power.

A cycle is like a unit of "computer effort." Every instruction and every system call used by a script costs some number of cycles. When a script runs, it spends cycles little by little, just like spending money from a budget.

At the blockchain level, there is a strict rule called max_block_cycles. This is the maximum number of cycles that all scripts in one block are allowed to use together. If the total goes over this limit, the whole block is rejected. This guarantees that blocks stay fast and that no block can overload the network. On the current mainnet (MIRANA), this limit is 3.5 billion cycles per block. There is no fixed limit for a single transaction or a single script. A script can use a lot of cycles, as long as the entire block stays under the maximum.

### How cycles are measured

When a script is first loaded into the VM, it already costs cycles. Every byte of code loaded costs a small amount. This encourages developers to write small, efficient programs and prevents attacks that use very large programs to waste resources.

After loading, every instruction the script runs consumes cycles. Most normal instructions cost 1 cycle, but some instructions cost more. Jumps and branches are more expensive because they affect control flow. Memory access costs more than simple math. Multiplication is expensive, and division is very expensive. This reflects how real computer hardware works.

System calls (syscalls) are also expensive. Every syscall has a base cost of about 500 cycles, plus extra cost depending on how much data is transferred. This is because syscalls require the VM to do complex bookkeeping before copying data.

### Optimizations and extensions

In CKB-VM version 1, a "B extension" was added for bit manipulation instructions. These are efficient and only cost 1 cycle each.

It also introduces something called Macro-Operation Fusion (MOP Fusion). This is a technique where several instructions are combined into one internal operation. When this happens, the combined instruction only costs as much as the most expensive part. This makes some cryptographic and mathematical programs much faster.

Version 2 of the VM adds more of these fused instructions, allowing even better performance for certain calculations.

Next, the document explains how cycles work for "spawn" and process-related syscalls in VM version 2. These are used when one script starts another script or communicates with it. Because this is complex, extra cycles are charged for creating processes, switching states, sending data, and waiting for results.

For example, spawning a new script costs a base amount, plus extra cost for data transfer and internal management. Reading and writing through pipes also costs cycles depending on how much data is sent.

---

## VM Selection

Over time, CKB introduces new VM versions to improve security, performance, and add new features. However, older scripts must continue working, so developers are given the option to choose which VM version their scripts use. This ensures backward compatibility and prevents upgrades from breaking existing applications.

In CKB, each VM version has its bundled instruction set, syscalls and cost model.

When a hard fork happens, a specific activation epoch is set. Blocks before this epoch use the old VM version, while blocks after use the new rules. After activation, the VM version used for a script is determined by the hash_type field in the script. Different hash_type values map to different VM versions, and scripts with different values are processed separately.

If a script uses type hash, CKB automatically runs it on the newest available VM version. This allows developers to benefit from the latest improvements. If a script uses data hash, the VM version is fixed and does not change after upgrades. This gives developers more stability and predictability.

Because of this design, developers can choose between stability and new features. Data hash gives strong determinism, while type hash provides automatic upgrades. Choosing between them depends on whether the developer prefers consistent behavior or better performance and new capabilities.

The system is designed this way to balance flexibility and determinism. Always using the newest VM was rejected because it would make execution depend on the current chain state. Using deployment time was also rejected because code could be redeployed to force version changes. The current method avoids these problems.

For backward compatibility, scripts that use data hash continue running on the same VM version even after forks. Scripts that use type hash may run on newer versions, so developers must ensure their code remains compatible and update it when needed.

If serious bugs are found in a specific VM version, the CKB maintenance team may add temporary protections at higher layers, such as in the mempool, to block risky transactions. These protections are not part of consensus rules. Users can still include such transactions manually if they want.

Overall, this VM selection system allows CKB to upgrade safely while preserving user control. It protects the network from known issues, supports long-term compatibility, and lets developers decide how their scripts evolve over time.

---

## VM History

This section explains the history and evolution of CKB-VM on Nervos CKB, showing how different VM versions were introduced to improve security, performance, fix bugs, and support new RISC-V features. Even though the network is upgraded through hard forks, CKB is designed so that old scripts can still run. Developers can choose whether their scripts are locked to a specific VM version or automatically follow upgrades by using data hash or type hash. This flexibility allows both stable and upgradable designs, but it also means developers must understand each VM version and its risks.

**VM 0** has existed since the genesis of the CKB network. It is based on the RV64IMC instruction set and includes basic syscalls for loading transactions, scripts, cells, headers, and witnesses, as well as debugging. Its cost model charges cycles based on real hardware performance. Each byte loaded during initialization and syscalls costs extra cycles, and most instructions cost one cycle, while branches, memory access, multiplication, division, and syscalls cost more. VM 0 has no known reported issues.

**VM 1** was introduced in 2021 with the Mirana hard fork. It added the RISC-V B extension for bit manipulation and introduced several macro-operation fusions (MOPs) to improve performance. It also added new syscalls and updated the cost model so that B instructions and MOPs have specific cycle costs. However, VM 1 has a known issue: the exec syscall is deprecated and may cause unexpected behavior, so it is not recommended.

**VM 2** was introduced in 2024 with the Meepo hard fork. It added more advanced MOPs for arithmetic operations and introduced several new spawn-related syscalls that allow scripts to create and manage processes, communicate through pipes, and access process information. The cost model was updated to include detailed cycle charges for these new operations, especially for spawning and data transfer. VM 2 currently has no known reported issues.

Overall, this document shows how CKB-VM has evolved from a basic execution environment into a more powerful and flexible virtual machine. Each new version adds performance improvements and new features while preserving compatibility with older scripts. Developers must choose their VM version carefully, understand the cost model, and be aware of known issues to avoid unexpected behavior and security risks.

---

## Type ID for Upgradeable Script

In CKB, scripts run on-chain and cannot normally be stopped or changed once deployed. However, because bugs or improvements may be needed later, developers need a safe way to upgrade scripts without breaking existing applications.

Type ID is a commonly used pattern that uses the hash_type field in the Script structure to make a script upgradable. The main idea is to ensure that only one live cell in the entire network can have a specific Type Script. This makes the script unique and gives it a permanent identity. A Type Script is considered unique when its code hash, hash type, and arguments together exist in only one live cell.

To achieve this, the Type ID script checks that there is at most one output cell and at most one input cell using the same Type Script. If an existing cell already has that Type Script, the transaction is allowed. If a new one is being created, the script verifies that its arguments match the first input's OutPoint. This ensures that no one can duplicate or forge the same Type Script. If someone tries to copy it, the system either rejects the transaction or detects double spending.

This mechanism prevents attackers from creating fake cells with the same identity. If they change the arguments, the script becomes different. If they try to reuse the same arguments, the blockchain blocks it. As a result, each Type ID corresponds to a single unique cell, which acts like a permanent identifier for the script.

Originally, CKB resolved scripts by matching the code hash with the data hash of dependency cells. This caused a problem for upgrades because when script data changed, its hash also changed, so old references stopped working. To fix this, CKB introduced the hash_type field. If hash_type is set to data, CKB matches scripts using data hashes. If it is set to type, CKB matches scripts using the Type Script hash instead. Since a Type Script can remain unchanged while the code data is updated, this allows upgrades without breaking references.

With this system, when a transaction runs, CKB looks at the script's code hash and hash type, then searches dependency cells using the appropriate method. If a matching cell is found, its data is used as the executable code. This makes script resolution predictable and supports controlled upgrades.

However, there is a possible attack if Type Scripts are not unique. An attacker could create another cell with the same Type Script but different code and use it to trick the system. To prevent this, developers must ensure that their Type Scripts are unique and unforgeable. This is exactly what the Type ID pattern guarantees.

To avoid recursive dependencies, the CKB team implemented the Type ID script as a special genesis script built into the system. It uses a predefined Type Script hash based on the text "TYPE_ID". This makes the mechanism reliable and available from the start of the network.

Finally, the document provides Rust examples showing how developers can integrate Type ID into their own scripts. These examples demonstrate how to check for existing Type ID cells, compute a unique ID from the first input and output index, and verify that new cells follow the Type ID rules.

Overall, Type ID provides a secure way to give scripts a permanent identity. By combining unique Type Scripts with the hash_type mechanism, CKB allows developers to upgrade deployed code safely, prevent forgery, and maintain deterministic behavior across network upgrades.

---

## Spawn: Direct Cross-Script Calls

Spawn was introduced in Nervos CKB during the Meepo hard fork in 2024 to improve how scripts communicate with each other. Before Spawn, CKB mainly used a system called exec to run another script. When a script used exec, its own execution stopped and was replaced by the new script, meaning it could not continue afterward. This made development inefficient because the original script lost its context and could not receive results from the script it called.

Spawn was created to solve this problem. With Spawn, a script can start another script as a "child process," wait for it to run, receive its result, and then continue its own execution. This is similar to how programs work in operating systems like Linux. Each running script is treated as a separate process with its own memory and execution space, even though everything still runs inside one blockchain transaction.

To support this system, CKB-VM introduced process IDs (PIDs) and file descriptors (FDs). Every script process gets a unique PID, starting from 0 for the main script. There are limits to prevent resource abuse: only up to 16 child processes and 4 active virtual machines can run at once. File descriptors are used to manage communication channels called pipes, and up to 64 can exist at the same time. These are used to send data between parent and child scripts.

Because scripts cannot directly access each other's memory, they communicate using pipes. A pipe has a read end and a write end. One script writes data into the pipe, and the other reads it. Before spawning a child script, the parent creates pipes and passes the file descriptors to the child. The child can then use them to send results back. Reading and writing also help the system decide when to switch between processes.

Spawn works through special system calls such as spawn, wait, pipe, read, write, and close. The spawn function creates a child process and starts running another script without destroying the parent's state. The wait function pauses the parent until the child finishes. The pipe function creates a communication channel. The read and write functions transfer data, and close releases resources. There is also a spawn_cell helper that finds scripts automatically using their hash information.

Process scheduling in CKB-VM is cooperative, not parallel. This means only one process runs at a time, and switching happens when a syscall is made. Even though multiple processes exist logically, they are executed step by step in a single thread. This design avoids complexity while still allowing structured cooperation between scripts.

The text also explains common errors that can happen, such as using invalid file descriptors, exceeding resource limits, or trying to read from a closed pipe. One important error, "OtherEndClosed," makes it possible to detect when all data has been sent, which allows developers to read complete messages from pipes.

An example is given where a child script acts like a small library: it receives arguments, combines them, and sends the result back to the parent through a pipe. The parent spawns the child, passes arguments, reads the result, and waits for the child to finish. This shows how Spawn can be used to build reusable on-chain libraries.

Compared to traditional blockchain systems like the EVM, where every contract call creates and destroys execution environments repeatedly, Spawn is more efficient. With pipes, a parent script can communicate many times with a child script while keeping its execution environment alive. This reduces cost and improves performance.

Typical use cases of Spawn include building public libraries that many scripts can reuse, managing very large scripts by splitting them into smaller parts, and improving security by isolating sensitive logic in separate processes. Overall, Spawn makes CKB scripts more modular, efficient, and powerful. It allows developers to write complex, multi-part applications where scripts can cooperate like programs in an operating system, while still running safely on the blockchain.


###Inter-Process Communication (IPC) in Scripts: A Deep Dive into ckb-script-ipc Library
Inter-Process Communication (IPC) means communication between processes running on the same machine. It is now possible in Nervos CKB through the new Spawn syscall introduced in the Meepo hard fork. Spawn allows one script to start another script as a child process and communicate with it. This is a major improvement because it enables modular, reusable, and structured communication between scripts inside a single transaction.

IPC is different from RPC (Remote Procedure Call), which is used in distributed systems over networks. In CKB’s case, all scripts run within the same transaction and environment, so the communication model matches IPC rather than RPC. Because it is local communication, the system does not need complex features like encryption, retry logic, or network transport protocols. It focuses only on lightweight process-to-process communication.

Using Spawn to implement IPC involves several steps. A developer must define service interfaces, create communication channels using pipes, serialize parameters, convert them into a binary wire format, send them to another process, receive and decode responses, dispatch the correct method, and handle return values and errors. Since building all this manually is complex, the ckb-script-ipc library was created to simplify the process.

The ckb-script-ipc library currently supports Rust, C, and JavaScript/TypeScript, although Rust has the most automation support. Because Spawn does not support concurrent execution, communication follows a client-server model. The server provides methods, and the client sends requests and waits for responses. Everything happens sequentially within one transaction.

A key part of the system is the wire format, which defines how data is structured when sent between processes. Instead of sending raw streams, the library uses a packet-based protocol. Each packet can be either a Request or a Response. The Request contains a version, method ID, length, and payload. The Response contains a version, error code, length, and payload. All numeric fields use VLQ (Variable Length Quantity) encoding, which makes small numbers compact while still supporting very large values. The payload is typically serialized using JSON through Rust’s serde framework, allowing structured data types to be safely transmitted between processes.

Beyond on-chain communication, the library also supports interaction between on-chain scripts and native machine code. Developers can run a CKB virtual machine locally, load a script binary, spawn it, and communicate with it using the same IPC model. This creates a bridge between off-chain applications and on-chain logic. However, the current implementation cannot access full transaction context data in native mode, and integration requires manual setup.

Future improvements aim to integrate this functionality directly into CKB nodes through an HTTP service and allow context-aware execution so off-chain applications can interact more deeply with transaction data.

Overall, Spawn and ckb-script-ipc significantly enhance CKB’s development model. They make it possible to build modular, reusable, and structured on-chain systems where scripts can communicate like processes in an operating system. This opens the door to more advanced applications and stronger integration between on-chain and off-chain environments.

### Debug Scripts

**What it is:** A doc/tooling section for **debugging CKB scripts** when you're writing or fixing them (e.g. Rust/C lock or type scripts). Not part of the “Introduction to Script” concepts; use it when you develop scripts.

**CKB-Debugger:** Standalone CLI to run scripts off-chain with a transaction file. Lets you see return code, cycles, and (with GDB) step through execution.

- **Install:** `cargo install --git https://github.com/nervosnetwork/ckb-standalone-debugger ckb-debugger` (≥ v0.113.0). On macOS: `brew install protobuf` first.
- **Input:** A JSON transaction (e.g. from `ckb-cli mock-tx dump --tx-hash <hash> --output-file mock_tx.json`, or from a failed tx + ckb-transaction-dumper).
- **Run:** `ckb-debugger --tx-file <file> --cell-index <n> --cell-type input|output --script-group-type lock|type` to execute the lock or type script of a specific cell. Use `--bin <path>` to swap in a different script binary for testing.

**Other options:** GDB mode (`--mode gdb --gdb-listen 127.0.0.1:9999`), VSCode (NativeDebug/CodeLLDB), profiling (`--pprof out.pprof` → inferno-flamegraph), and **Native Simulator** (ckb-std feature for debugging on the host; use with ckb-script-templates / ckb-testtool).

**When to use:** When writing or debugging your own scripts; not required for the Intro to Script reading. Prefer CKB-Debugger for existing/maintenance projects (same VM as chain); Native Simulator for new projects and heavy debugging (faster, but not identical to on-chain).

### Recommended Workflow for Script Upgrades

the recommended workflow for upgrading scripts on Nervos CKB is using the Type ID mechanism. Type ID allows a cell to have a unique Type Script across all live cells while still being upgradable. However, whether the code is actually changeable does not depend on Type ID alone. The Lock Script attached to the cell plays a crucial role in controlling whether upgrades are allowed.

The key idea is that a Type ID cell can change its Lock Script during upgrades. This provides flexibility. When a script is first deployed, developers usually do not know whether it is fully stable or free of bugs. Because of this uncertainty, it is recommended to deploy the code inside a Type ID cell with an upgradable lock, such as a multi-signature lock. This allows bug fixes and feature additions later.

As development continues, the script can go through multiple iterative upgrades using the Type ID design. Each upgrade modifies the code stored in the same uniquely identified cell. Over time, the system may reach a point where the code is considered stable and trusted, especially if it controls significant assets. At that stage, the developer can “freeze” the code by changing the Lock Script to an unlockable state. This prevents any further modification of the cell. Even though the Type ID remains, the cell becomes permanently unchangeable because the lock no longer allows updates.

The workflow separates responsibilities between two roles. A script developer decides whether to deploy code in an upgradable Type ID cell. A dApp developer independently decides whether to reference that code using the Type ID approach. This separation gives flexibility. One developer can provide upgradeable infrastructure, while another can choose whether to rely on the latest version or not.

An example is given using UDT (User-Defined Token) code deployed with Type ID. If two tokens use the same UDT code but reference it differently through their hash_type settings, their behavior during upgrades will differ. A token using hash_type “type” will automatically adopt the upgraded version of the code. A token using a data-based hash_type will continue using the old version, even after an upgrade. This shows that upgrades can be selectively adopted depending on how the code is referenced.

Even if an older version of code is no longer stored in a live cell, it is not permanently lost. Since the blockchain keeps historical data, developers can locate the old version from chain history and redeploy it if necessary. There is also discussion about potentially allowing transactions to temporarily include code in witness data, which could provide additional flexibility in the future.

Overall, the recommended workflow combines Type ID and Lock Scripts to create a structured upgrade path. Scripts can start flexible and upgradable, go through iterative improvements, and eventually be frozen for security and stability. This approach balances adaptability during development with long-term safety once the system matures.

### Common Error Codes

CKB provides a set of basic system-level error codes that scripts may encounter during execution. Some errors relate to data access. For example, CKB_INDEX_OUT_OF_BOUND occurs when all available indices of a certain type have already been fetched, and CKB_ITEM_MISSING happens when a requested entity does not exist, such as trying to load a Type Script from a cell that does not contain one. CKB_LENGTH_NOT_ENOUGH indicates invalid data length, such as incorrect script arguments or malformed signatures. CKB_INVALID_DATA is triggered when there is an error in Molecule serialization format.

Other error codes are related to process management and inter-process communication introduced by the Spawn syscall. CKB_WAIT_FAILURE happens when an invalid file descriptor is used during a wait syscall. CKB_INVALID_FD means the file descriptor does not belong to the current process. CKB_OTHER_END_CLOSED indicates that the opposite side of a communication pipe has been closed. There are also resource limit errors such as CKB_MAX_VMS_SPAWNED, which signals that the maximum number of spawned processes has been reached, and CKB_MAX_FDS_CREATED, which indicates that the system has reached the maximum number of created pipes.

In situations where read, write, or wait operations block each other in a circular manner, a deadlock can occur. When this happens, the CKB virtual machine detects the internal error and immediately terminates execution to prevent the transaction from hanging indefinitely.

The document also lists Molecule-specific error codes defined in the standard C library. These errors relate to issues in structured data parsing and validation, such as incorrect total size, invalid headers, wrong offsets, unknown items, index out-of-bounds access, incorrect field counts, malformed data, or numeric overflow. These codes help developers diagnose serialization and deserialization problems in structured blockchain data.

Finally, it mentions that additional standardized error codes are provided through the ckb-script-error-codes library, which allows common scripts to use consistent and recognizable error definitions.

Overall, this section explains how CKB categorizes and reports execution, serialization, resource, and communication errors, helping developers debug scripts and understand why a transaction fails during validation.

### Script testing guide 
This guide provides a comprehensive overview of testing smart contracts, known as Scripts, on the Nervos CKB network, which operates on a unique UTXO model. Unlike many other blockchains, CKB transactions are composed of Inputs (consumed resources), Outputs (newly created resources), and CellDeps (referenced code or dependent cells). The basic unit of storage is the Cell, which contains a Lock Script to define ownership, an optional Type Script for validation logic, and the Data itself. A script is defined by its code hash, a hash type that determines how to interpret that code, and specific arguments passed to it.
The verification process in CKB relies on a specific grouping logic where cells are organized based on the hash of their scripts. It is critical to note that while Lock and Type scripts in the Inputs are executed, only Type scripts are executed in the Outputs; Lock scripts in the outputs are ignored during verification. The CKB-VM executes these groups in order, tracking the "Cycles" consumed to ensure the transaction stays within computational limits.
When designing test cases, developers must account for various combinations of inputs and outputs. Lock Script testing focuses primarily on input validation, while Type Script testing covers a wider range of scenarios, including burning cells (input only), transferring (input to output), splitting or combining cells, and minting (output only). For example, a Simple UDT (sUDT) contract must be tested to ensure that the total token amount in the inputs is at least equal to the total in the outputs, preventing unauthorized creation of tokens. These tests should also verify owner mode privileges and ensure that data encoding, such as the 16-byte length requirement for UDT amounts, is strictly followed.
Final testing considerations involve exploring failure cases, such as scripts returning non-zero values or transactions exceeding the 1000M Cycle limit. Because the CKB-VM has memory constraints, developers should test the upper size boundaries for fields like script arguments, witnesses, and cell data. Ultimately, a robust test suite requires a deep understanding of the UTXO model to ensure contract stability under all conditions

### 
Fuzzing is an automated testing technique where random, unexpected, or malformed inputs are continuously fed into a program to see if it crashes, panics, overflows memory, orsafe ways. Unlike unit tests, which are written by developers to check specific scenarios, fuzzing explores unpredictable edge cases that humans often overlook. For CKB scripts, this is especially important because scripts parse transaction data such as witnesses and inputs, and malicious or malformed data could trigger hidden bugs.

Modern fuzzers do not just generate completely random data. They track code coverage, meaning they monitor which parts of the code are executed by each input. They then prioritize inputs that explore new code paths. They also use evolutionary mutation strategies and start from a set of initial “seed” inputs called a corpus. These corpuses are often generated from existing unit tests or mock transactions. Over time, fuzzers mutate the behaves in unse corpuses to discover new behaviors and potential crashes. Fuzzing is typically long-running and can run continuously for days or months.

The recommended workflow for fuzzing CKB scripts is to first generate initial corpuses from unit tests or mock transaction generators. During active development, fuzzing can be integrated into CI for short runs (for example 15–60 minutes). For mature scripts, dedicated machines can run fuzzing 24/7. If fuzzers continue to find issues, more resources should be allocated; if not, fuzzing intensity can be reduced.

An important concept discussed is that fuzzers rely heavily on compiler sanitizers (such as AddressSanitizer and UndefinedBehaviorSanitizer). These sanitizers automatically detect memory corruption, null pointer dereferences, buffer overflows, and undefined behavior. In fuzzing setups, the return value of the script is often ignored because most random inputs naturally fail validation. Instead, crashes detected by sanitizers are treated as indicators of real issues. In special cases where two implementations of the same algorithm exist, fuzzing can compare their outputs to ensure consistency.

Several practical recommendations are given: do not overthink the choice of fuzzer, start with one and iterate; always begin with useful corpuses; add additional fuzzers if bugs are discovered; and treat fuzzing as a long-term security investment.