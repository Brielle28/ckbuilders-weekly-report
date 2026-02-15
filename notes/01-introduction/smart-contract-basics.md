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
