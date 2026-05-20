# 🍔 The Grand Restaurant Guide: Docker & Kubernetes (ECI Sandbox Edition)

Eita holo tomar puro Polyglot Sandbox Architecture-er ekta compact "cheat-sheet". Jokhon-i dharona gulo gholafe hobe, ei Restaurant Analogy-ta ekbar pore nibe!

---

## 🍳 Phase 1: Docker (The Kitchen & Food Prep)
Docker-er kaaj holo code-ke pack kora jate sheta jekono platform-e ek-i bhabe kaaj korte pare.

| Term | Restaurant Analogy | Technical Definition & Workings |
| :--- | :--- | :--- |
| **Dockerfile** | **Rannar Recipe (Niomaboli)** | Ekta text file jekhane application chalate ki ki dependency lagbe (OS, libraries, environment vars) tar step-by-step instruction thake. |
| **Image** | **Frozen Food Packet** | Recipe dekhe jokhon shob ingredient pack kore frozen kore rakha hoy. Eita statically ready thake (inactive/compiled), kintu ekhono execute hocche na. |
| **Container** | **Served Hot Dish (Janto Khabar)** | Frozen packet-ta ke jokhon oven-e diye customer-er samne served kora hoy. Image jokhon live execute hoy, tokhon sheta Container. Ekta image theke 100 ta container run kora jay. |

---

## 🏢 Phase 2: Kubernetes / K8s (The Restaurant Management)
Container-er boro "Cluster" ba complex system managed korar dayitto Kubernetes-er.

| Term | Restaurant Analogy | Technical Definition & Workings |
| :--- | :--- | :--- |
| **Pod** | **Serving Tray / Food Box** | K8s shorashori Container chine na. Se container-ke ekta Tray ba Pod-er bhetor rakhe. Protita Pod-er ekta unique Internal IP thake. Sadharonoto 1 Pod = 1 Container. |
| **Node** | **Kitchen Counter / Server Table** | Actual physical ba virtual computer (CPU, RAM, Hard Disk). Ekta boro Node-er upore onek gulo Pod (Tray) thakte pare. Node full hole cluster-e notun Node add korte hoy. |
| **Namespace** | **Private Dining Rooms / Zones** | Cluster-ke virtually vhag korar rasta. Jemon: `eci-system` namespace-e thake core kitchen tools (Gateway, Orchestrator), ar `eci-sandboxes` room-e thake student-der isolated pods. |
| **YAML File** | **Manager-er Kache Odrer Slip** | Declarative instruction. Tumi manager-ke slip-e likhe dao: *"Amar 3ta C++ Pod lagbe, 512MB RAM-er bhetor."* K8s sheta dekhe automatically setup kore dey. |
| **Control Loop** | **The Alert Restaurant Manager** | K8s-er bhetor ekta loop onoboroto ghure check kore: Order Slip-e ki lekha ase (Desired State) vs Asol table-e koyta plate ase (Actual State). Plate bhenghe gele se automatic notun plate ready kore dey. |

---

## 🛡️ Phase 3: Advanced Sandboxing (The Security Systems)
Amader Polyglot Platform-e amra je specialized security implement koresi, shegulo restaurant context-e e bhabe kaaj kore:

### 1. Resource Limits (`rlimit` / `cgroups`) = Portion Control
* **Analogy:** Kono bipodjjok khadok jeno eka kitchen-er shob mangsho (RAM) ba gas (CPU) sesh na kore felte pare, tai tar thalar size strictly limit (`MAX_MEMORY_MB = 512`) kore deya. Limit cross korlei table theke thala shoraye (Process Kill) deya hoy.
* **Result:** Memory full hoye main server crash kore na (`std::bad_alloc` or `SIGSEGV`).

### 2. Fork Bomb Protection (`RLIMIT_NPROC`) = Anti-Cloning Rule
* **Analogy:** Kono chora-customer kitchen-e dhuke jodi infinitely nijer clone (Child processes) toiri korte thake jate counter-e ar keu kaj na korte pare (`while(fork())`), manager take max 50 ta clone-e block kore dey.
* **Result:** Pod-er process ID (PID) exhaust hoye node crash hoy na.

### 3. Seccomp Network Block (`socket`) = Gate Guard Blocking Calls
* **Analogy:** Kitchen-er bhetor theke untrusted chef jeno baire phone kore kono chora-mal order korte na pare (`socket` creation), tai dorjay ekta strict guard thake.
* **`SCMP_ACT_KILL` Mode:** Call korar chesta korlei guard guli kore dibe (`Signal 31 - SIGSYS`).
* **`SCMP_ACT_ERRNO(EACCES)` Mode:** Guard gently phone kete diye bolbe "Permission Denied" jate chef gracefully error handle korte pare.

### 4. Privilege Drop (`setuid` / `setgid`) = Secret Inspector Drop
* **Analogy:** Prothome limits ar security context set korar jonno setup team-ke **Root (Owner)** er chabi niye dhukte hoy. Kazu shuru howar thik ager muhurte tara chabi feliye diye normal unprivileged **`sandboxuser` (ID: 10002)** hoye jay. 
* **Result:** Code execute howar shomoy attacker container-er root access pay na.

### 5. Memory Disk (`tmpfs` in K8s) = High-Speed Chopping Board
* **Analogy:** Chef jokhon shobji kate, se jodi bar bar pichone giye main storage box-e (Hard Disk I/O) file rakhe, ranna slow hobe. Tai counter-er uporei ekta temporary dynamic speed-board (RAM Disk `/tmp`) mount kora hoy.
* **Result:** Execution blazing fast hoy, disk lifecycle bache.

### 6. Compiler Optimization (Dead Code Elimination) = Lazy Chef Smart Trick
* **Analogy:** Tumi slip-e bolla "600MB RAM allocate koro, abar delete koro" (`volatile char* ptr = new char[...]`). Smart chef binary build korar shomoy dekhlo ei khabar keu khabena, se ranna-i korlo na (Instruction erased by `g++ -O2`). 
* **Fix:** Chef-ke badhho korte protyekta thalay ekta kore bites (`ptr[i] = 'X'`) likhe check korte hoy!

---

## 🛠️ Quick Recall Core Workflow
1. **`make all-local`** = Raw code theke parallel recipe use kore notun fresh frozen food packet (Images) banano ar local kitchen counter-e (K8s) order set kora.
2. **`make tst`** = Fake student-কে diye master gauntlet attack run korano ar check kora manager thikmoto guli korche naki block korche!

**Keep this file safe and look back whenever you design microservices!**
