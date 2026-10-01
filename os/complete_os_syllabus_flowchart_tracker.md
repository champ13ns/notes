# Operating Systems — Complete Syllabus Flowchart & Study Tracker

> **Purpose:** Complete OS roadmap for strong conceptual understanding + RRB SO IT / banking IT exam preparation.  
> **Note:** This is a comprehensive Operating Systems roadmap, not an official topic-by-topic IBPS syllabus.
>
> **Legend**
> - ✅ Covered
> - 🟡 Partially covered / revision needed
> - ⬜ Pending
> - ⭐ High priority
> - 🔹 Secondary / conceptual depth

---

# MASTER FLOWCHART

```text
OPERATING SYSTEMS
│
├── 1. OS FUNDAMENTALS ⭐
│   ├── What is an OS?
│   ├── Goals / Functions of OS
│   ├── Kernel vs User Mode
│   ├── System Calls
│   ├── Interrupts / Traps / Exceptions
│   ├── Types of Operating Systems
│   └── OS Structures
│
├── 2. PROCESS MANAGEMENT ⭐
│   ├── Program vs Process
│   ├── Process Memory Layout
│   ├── PCB
│   ├── Process States
│   ├── State Transitions
│   ├── Ready / Waiting Queues
│   ├── Scheduler
│   ├── Dispatcher
│   ├── Context Switching
│   ├── Process Creation / Termination
│   ├── fork() / exec()
│   ├── CPU-bound vs I/O-bound
│   └── IPC Basics
│
├── 3. THREADS ⭐
│   ├── Process vs Thread
│   ├── User / Kernel Threads
│   ├── Multithreading Models
│   ├── Thread Context
│   └── Benefits of Multithreading
│
├── 4. CPU SCHEDULING ⭐
│   ├── Scheduling Criteria
│   ├── Preemptive vs Non-Preemptive
│   ├── FCFS
│   ├── SJF
│   ├── SRTF
│   ├── Priority Scheduling
│   ├── Round Robin
│   ├── Multilevel Queue
│   ├── Multilevel Feedback Queue
│   ├── Starvation
│   ├── Aging
│   └── Numericals
│
├── 5. PROCESS SYNCHRONIZATION ⭐
│   ├── Race Condition
│   ├── Critical Section
│   ├── Mutual Exclusion
│   ├── Atomic Operations
│   ├── Mutex
│   ├── Semaphore
│   ├── Spinlock
│   ├── Monitor
│   ├── Condition Variables
│   └── Classical Problems
│       ├── Producer Consumer
│       ├── Readers Writers
│       └── Dining Philosophers
│
├── 6. DEADLOCK ⭐
│   ├── Resource Allocation
│   ├── 4 Coffman Conditions
│   ├── Resource Allocation Graph
│   ├── Prevention
│   ├── Avoidance
│   ├── Safe / Unsafe State
│   ├── Banker's Algorithm
│   ├── Detection
│   └── Recovery
│
├── 7. MEMORY MANAGEMENT ⭐
│   ├── Logical vs Physical Address
│   ├── MMU
│   ├── Address Binding
│   ├── Dynamic Loading / Linking
│   ├── Swapping
│   ├── Contiguous Allocation
│   ├── Fixed / Variable Partitions
│   ├── First / Best / Worst Fit
│   ├── Fragmentation
│   │   ├── Internal
│   │   └── External
│   ├── Paging
│   ├── Segmentation
│   └── Paging vs Segmentation
│
├── 8. PAGING ⭐
│   ├── Page / Frame
│   ├── Page Size / Frame Size
│   ├── Page Number / Offset
│   ├── Logical → Physical Translation
│   ├── Page Table
│   ├── PTE
│   │   ├── Frame Number
│   │   ├── Present / Valid Bit
│   │   ├── Dirty Bit
│   │   ├── Reference / Accessed Bit
│   │   └── Protection Bits
│   ├── TLB
│   ├── TLB Hit / Miss
│   ├── Effective Access Time
│   ├── Multi-Level Paging
│   ├── Inverted Page Table
│   └── Page Table Size Numericals
│
├── 9. VIRTUAL MEMORY ⭐
│   ├── Virtual Memory Concept
│   ├── Demand Paging
│   ├── Page Fault
│   ├── Page Fault Handling
│   ├── Locality of Reference
│   ├── Page Replacement
│   │   ├── FIFO
│   │   ├── Belady's Anomaly
│   │   ├── Optimal
│   │   ├── LRU
│   │   ├── Second Chance
│   │   └── Clock
│   ├── Frame Allocation
│   ├── Local vs Global Replacement
│   ├── Thrashing
│   ├── Working Set
│   └── Page Fault Frequency
│
├── 10. SEGMENTATION ⭐
│   ├── Segment Number
│   ├── Offset
│   ├── Segment Table
│   ├── Base / Limit
│   ├── Address Translation
│   ├── Protection / Sharing
│   ├── Fragmentation
│   └── Segmentation with Paging
│
├── 11. FILE SYSTEMS ⭐
│   ├── File Concept
│   ├── File Attributes
│   ├── File Operations
│   ├── Access Methods
│   │   ├── Sequential
│   │   ├── Direct
│   │   └── Indexed
│   ├── Directory Structures
│   ├── File Allocation
│   │   ├── Contiguous
│   │   ├── Linked
│   │   └── Indexed
│   ├── Free Space Management
│   ├── Inodes
│   ├── File Permissions
│   ├── Mounting
│   └── Journaling Basics
│
├── 12. DISK / SECONDARY STORAGE ⭐
│   ├── Disk Structure
│   ├── Seek Time
│   ├── Rotational Latency
│   ├── Transfer Time
│   ├── Disk Scheduling
│   │   ├── FCFS
│   │   ├── SSTF
│   │   ├── SCAN
│   │   ├── C-SCAN
│   │   ├── LOOK
│   │   └── C-LOOK
│   ├── SSD Basics
│   ├── Swap Space
│   └── RAID Basics
│
├── 13. I/O MANAGEMENT 🔹
│   ├── I/O Hardware
│   ├── Device Controllers
│   ├── Device Drivers
│   ├── Polling
│   ├── Interrupt-Driven I/O
│   ├── DMA
│   ├── Buffering
│   ├── Caching
│   └── Spooling
│
├── 14. IPC ⭐
│   ├── Shared Memory
│   ├── Message Passing
│   ├── Pipes
│   ├── Named Pipes
│   ├── Message Queues
│   ├── Signals
│   └── Sockets
│
├── 15. PROTECTION & SECURITY ⭐
│   ├── Protection Domain
│   ├── Access Matrix
│   ├── ACL
│   ├── Capability Lists
│   ├── User / Kernel Mode
│   ├── Privileged Instructions
│   └── Least Privilege
│
├── 16. SYSTEM CALLS / KERNEL 🔹
│   ├── Process Control Calls
│   ├── File Management Calls
│   ├── Device Management Calls
│   ├── Information Calls
│   ├── Communication Calls
│   ├── Trap to Kernel
│   └── Return to User Mode
│
├── 17. OS ARCHITECTURE 🔹
│   ├── Monolithic Kernel
│   ├── Microkernel
│   ├── Layered OS
│   ├── Modular Kernel
│   ├── Hybrid Kernel
│   └── Virtual Machines Basics
│
└── 18. OPTIONAL / ADVANCED 🔹
    ├── Copy-on-Write
    ├── Memory-Mapped Files
    ├── Huge Pages
    ├── NUMA
    ├── Real-Time Scheduling
    ├── Multiprocessor Scheduling
    ├── Containers / Namespaces
    └── Virtualization
```

---

# DETAILED STUDY TRACKER

## 1. OS Fundamentals ⭐

- 🟡 What is an Operating System?
- 🟡 Role of OS between hardware and applications
- ⬜ OS goals: convenience, efficiency, resource management
- ⬜ Kernel vs user space
- ⬜ User mode vs kernel mode
- 🟡 Interrupt vs trap vs exception
- ⬜ Booting basics
- ⬜ System calls
- ⬜ Types of OS
- ⬜ Monolithic / Layered / Microkernel / Modular / Hybrid

---

## 2. Process Management ⭐

### Program / Process
- ✅ Program vs process
- ✅ Executable / binary
- ✅ Process creation idea
- ✅ Process virtual address space
- ✅ Code / Data / BSS / Heap / Stack

### Process Control
- ✅ PCB
- ✅ PID
- ✅ Program Counter
- ✅ CPU registers
- ✅ Process state
- ✅ Scheduling info
- ✅ Memory-management info

### Process States
- ✅ New
- ✅ Ready
- ✅ Running
- ✅ Waiting / Blocked
- ✅ Terminated
- ✅ State transitions
- ✅ Ready ≠ secondary memory
- ✅ Running ≠ entire process in RAM

### Scheduling Infrastructure
- ✅ Ready queue
- ✅ Scheduler concept
- ✅ Dispatcher
- ✅ Context switching
- ✅ CPU-bound vs I/O-bound
- 🟡 Context-switch overhead

### Process Creation
- ✅ `fork()`
- ✅ `exec()`
- ✅ Parent / child basics
- 🟡 Copy-on-Write intuition
- ⬜ Zombie process
- ⬜ Orphan process

---

## 3. Threads ⭐

- ✅ Process vs thread
- ✅ Shared code / data / heap
- ✅ Separate thread stack
- ✅ Separate registers
- ✅ Separate program counter
- ⬜ User-level threads
- ⬜ Kernel-level threads
- ⬜ Many-to-One
- ⬜ One-to-One
- ⬜ Many-to-Many
- ⬜ Benefits / limitations of multithreading

---

## 4. CPU Scheduling ⭐

### Concepts
- 🟡 Scheduling basics
- ✅ Ready → Running
- ✅ Running → Waiting
- ✅ Waiting → Ready
- ✅ Running → Ready / preemption
- ⬜ Preemptive vs non-preemptive

### Criteria
- ⬜ CPU utilization
- ⬜ Throughput
- ⬜ Turnaround time
- ⬜ Waiting time
- ⬜ Response time

### Algorithms
- ⬜ FCFS
- ⬜ SJF
- ⬜ SRTF
- ⬜ Priority Scheduling
- ⬜ Round Robin
- ⬜ Multilevel Queue
- ⬜ Multilevel Feedback Queue
- ⬜ Starvation
- ⬜ Aging
- ⬜ Gantt-chart numericals

---

## 5. Process Synchronization ⭐

- ✅ Race condition
- ✅ Critical section
- ✅ Mutual exclusion
- 🟡 Atomicity
- ✅ Mutex
- ✅ Mutex ownership
- ✅ Semaphore
- ✅ Binary semaphore
- ✅ Counting semaphore
- ✅ Mutex vs semaphore
- ⬜ Spinlock
- ⬜ Monitor
- ⬜ Condition variable
- ⬜ Compare-and-Swap
- ⬜ Producer–Consumer
- ⬜ Readers–Writers
- ⬜ Dining Philosophers

---

## 6. Deadlock ⭐

- ✅ Deadlock meaning
- ✅ Mutual Exclusion
- ✅ Hold and Wait
- ✅ No Preemption
- ✅ Circular Wait
- ✅ Prevention concept
- ✅ Avoidance concept
- ✅ Safe state
- ✅ Unsafe state
- ✅ Unsafe ≠ necessarily deadlocked
- ✅ Banker's Algorithm
- ✅ `Need = Max - Allocation`
- 🟡 Safe sequence numericals
- ⬜ Resource Allocation Graph
- ⬜ Detection
- ⬜ Recovery

---

## 7. Memory Management ⭐

### Basics
- ✅ RAM vs secondary storage
- ✅ Logical / virtual address
- ✅ Physical address
- ✅ Byte-addressable memory
- ✅ Bit vs byte vs word
- ✅ MMU
- 🟡 `malloc()` and virtual-memory intuition
- ⬜ Address binding
- ⬜ Dynamic loading
- ⬜ Dynamic linking

### Contiguous Allocation
- ✅ Contiguous allocation concept
- 🟡 Fixed partitions
- 🟡 Variable partitions
- ⬜ First Fit
- ⬜ Best Fit
- ⬜ Worst Fit
- ⬜ Next Fit
- ✅ External fragmentation
- ✅ Internal fragmentation
- ⬜ Compaction

---

## 8. Paging ⭐

### Basics
- ✅ Page
- ✅ Frame
- ✅ Page size = frame size
- ✅ Non-contiguous allocation
- ✅ External fragmentation removed
- ✅ Internal fragmentation possible

### Address Math
- ✅ `1K = 1024 = 2^10`
- ✅ 4 KB = `2^12` bytes
- ✅ Offset bits
- ✅ Page-number bits
- ✅ Number of pages
- ✅ Number of frames
- ✅ Frame-number bits
- ✅ Virtual address range
- ✅ Physical address range
- ✅ Word-addressable vs byte-addressable
- ✅ `Page Number = floor(VA / Page Size)`
- ✅ `Offset = VA % Page Size`
- ✅ `PA = Frame × Frame Size + Offset`

### Translation
- ✅ `[Page Number | Offset]`
- ✅ Page Number → Frame Number
- ✅ Offset remains unchanged
- ✅ `[Frame Number | Offset]`
- ✅ CPU → MMU → RAM flow

### Page Table / PTE
- ✅ Page table
- ✅ PTE
- ✅ Frame number
- ✅ Present / valid bit
- ✅ Dirty / modified bit
- ✅ Reference / accessed bit
- ✅ Protection bits

### TLB
- ✅ TLB concept
- ✅ TLB stores translation, not page data
- ✅ TLB hit
- ✅ TLB miss
- ✅ TLB miss ≠ page fault
- ✅ Effective Access Time basics

### Multi-Level Paging
- ✅ Why single-level table is large
- ✅ 32-bit VA + 4 KB page
- ✅ `20-bit VPN + 12-bit offset`
- ✅ `10-bit L1 + 10-bit L2 + 12-bit offset`
- ✅ L1 contains entries, not pages
- ✅ Same virtual address space
- ✅ Lower-level tables only when needed
- ⬜ Inverted page table
- ⬜ Deeper multi-level paging intuition

---

## 9. Virtual Memory ⭐

### Demand Paging
- ✅ Entire process need not be in RAM
- ✅ Pages can load on demand
- ✅ Backing storage concept
- ✅ Page-in / page-out
- ✅ Paging vs traditional swapping distinction

### Page Fault
- ✅ Page fault concept
- ✅ Page fault ≠ automatically an error
- ✅ Trap to OS
- ✅ Valid vs invalid access
- ✅ Page fault vs segmentation fault
- ✅ Process may block for disk I/O
- ✅ Waiting → Ready
- ✅ Faulting instruction retried
- ✅ Free-frame vs no-free-frame case

### Locality
- ✅ Temporal locality
- ✅ Spatial locality

---

## 10. Page Replacement ⭐

- ✅ FIFO
- ✅ FIFO hit does not refresh order
- ✅ Belady's Anomaly
- ✅ Optimal
- ✅ Optimal = theoretical minimum
- ✅ LRU
- ✅ LRU uses last access, not access count
- ✅ LRU vs LFU distinction
- ⬜ Second Chance
- ⬜ Clock
- ⬜ Local replacement
- ⬜ Global replacement
- ⬜ Frame allocation policies

---

## 11. Thrashing & Working Set ⭐

- ⬜ Thrashing
- ⬜ Why excessive page faults occur
- ⬜ Degree of multiprogramming
- ⬜ Working Set model
- ⬜ Working Set window
- ⬜ Page Fault Frequency
- ⬜ Thrashing control

---

## 12. Segmentation ⭐

- ⬜ Why segmentation?
- ⬜ Logical memory segments
- ⬜ Segment number
- ⬜ Offset
- ⬜ Segment table
- ⬜ Base
- ⬜ Limit
- ⬜ Address translation
- ⬜ Protection
- ⬜ Sharing
- ⬜ External fragmentation
- ⬜ Paging vs segmentation
- ⬜ Segmentation with paging

---

## 13. File Systems ⭐

### File Basics
- ⬜ File concept
- ⬜ File attributes
- ⬜ Create / open / read / write / seek / close / delete

### Access Methods
- ⬜ Sequential
- ⬜ Direct / random
- ⬜ Indexed

### Directory Structures
- ⬜ Single-level
- ⬜ Two-level
- ⬜ Tree
- ⬜ Acyclic graph
- ⬜ General graph

### File Allocation
- ⬜ Contiguous
- ⬜ Linked
- ⬜ Indexed

### Free Space
- ⬜ Bitmap
- ⬜ Linked free list
- ⬜ Grouping / counting

### Unix/Linux Concepts
- ⬜ inode
- ⬜ hard link
- ⬜ symbolic link
- ⬜ permissions
- ⬜ mount
- ⬜ journaling basics

---

## 14. Disk / Secondary Storage ⭐

- ⬜ Track
- ⬜ Sector
- ⬜ Cylinder
- ⬜ Seek time
- ⬜ Rotational latency
- ⬜ Transfer time
- ⬜ FCFS
- ⬜ SSTF
- ⬜ SCAN
- ⬜ C-SCAN
- ⬜ LOOK
- ⬜ C-LOOK
- ⬜ Disk scheduling numericals
- 🟡 RAID basics
- ⬜ Swap-space management
- ⬜ SSD vs HDD basics

---

## 15. I/O Management 🔹

- ⬜ I/O devices
- ⬜ Device controller
- ⬜ Device driver
- ⬜ Polling
- ⬜ Interrupt-driven I/O
- ⬜ DMA
- ⬜ Buffering
- ⬜ Caching
- ⬜ Spooling

---

## 16. IPC ⭐

- ⬜ Shared memory
- ⬜ Message passing
- ⬜ Anonymous pipes
- ⬜ Named pipes / FIFO
- ⬜ Message queues
- ⬜ Signals
- ⬜ Sockets
- ⬜ Shared memory vs message passing
- ⬜ IPC synchronization

---

## 17. Protection & Security ⭐

- ⬜ Protection vs security
- ⬜ Protection domain
- ⬜ Access matrix
- ⬜ ACL
- ⬜ Capability list
- ⬜ User mode
- ⬜ Kernel mode
- ⬜ Privileged instructions
- ⬜ Authentication basics
- ⬜ Least privilege

---

## 18. System Calls & Kernel 🔹

- ⬜ Process-control calls
- ⬜ File-management calls
- ⬜ Device-management calls
- ⬜ Information calls
- ⬜ Communication calls
- ⬜ Protection calls
- ⬜ Trap into kernel
- ⬜ Return to user mode

---

## 19. OS Architecture 🔹

- ⬜ Monolithic kernel
- ⬜ Microkernel
- ⬜ Layered OS
- ⬜ Modular kernel
- ⬜ Hybrid kernel
- ⬜ Architecture trade-offs

---

## 20. Optional / Advanced 🔹

- 🟡 Copy-on-Write
- ⬜ Memory-mapped files
- ⬜ Huge pages
- ⬜ NUMA
- ⬜ Multiprocessor scheduling
- ⬜ Real-time scheduling
- ⬜ Priority inversion
- ⬜ Virtual machines
- ⬜ Hypervisors
- ⬜ Containers / namespaces
- ⬜ Cgroups

---

# RECOMMENDED STUDY ORDER FROM CURRENT POSITION

```text
You are here:
Page Replacement
    ├── FIFO ✅
    ├── Optimal ✅
    └── LRU ✅

NEXT:
Second Chance / Clock
        ↓
Thrashing
        ↓
Working Set + Page Fault Frequency
        ↓
Segmentation
        ↓
Paging vs Segmentation
        ↓
CPU Scheduling Algorithms
        ↓
Synchronization Classical Problems
        ↓
Deadlock Detection / Recovery
        ↓
File Systems
        ↓
Disk Scheduling
        ↓
I/O Management
        ↓
IPC
        ↓
Protection / Security
        ↓
System Calls + OS Architecture
```

---

# RRB SO IT — HIGH-PRIORITY REVISION MAP

```text
OS
│
├── Process Management ⭐⭐⭐
│   ├── PCB
│   ├── States
│   ├── Context Switch
│   ├── Process vs Thread
│   └── fork / exec
│
├── CPU Scheduling ⭐⭐⭐
│   ├── FCFS
│   ├── SJF / SRTF
│   ├── Priority
│   ├── Round Robin
│   └── Numericals
│
├── Synchronization ⭐⭐⭐
│   ├── Race Condition
│   ├── Critical Section
│   ├── Mutex
│   └── Semaphore
│
├── Deadlock ⭐⭐⭐
│   ├── 4 Conditions
│   ├── Prevention
│   ├── Avoidance
│   └── Banker's Algorithm
│
├── Memory Management ⭐⭐⭐
│   ├── Fragmentation
│   ├── Paging
│   ├── Segmentation
│   ├── TLB
│   ├── Page Table
│   ├── Page Fault
│   └── Address Numericals
│
├── Virtual Memory ⭐⭐⭐
│   ├── Demand Paging
│   ├── FIFO
│   ├── Optimal
│   ├── LRU
│   ├── Belady's Anomaly
│   └── Thrashing
│
├── File Systems ⭐⭐
│   ├── Allocation Methods
│   ├── Directories
│   └── Access Methods
│
├── Disk Scheduling ⭐⭐
│   ├── FCFS
│   ├── SSTF
│   ├── SCAN / C-SCAN
│   └── LOOK / C-LOOK
│
└── I/O / Kernel / Protection ⭐
    ├── DMA
    ├── Interrupts
    ├── System Calls
    └── User vs Kernel Mode
```

---

# CURRENT PROGRESS SNAPSHOT

```text
Process Fundamentals             ✅
Process States / PCB             ✅
Threads Basics                   ✅
Scheduling Fundamentals          ✅
Scheduling Algorithms            ⬜
Synchronization Basics           ✅
Deadlock Basics / Banker         ✅
Memory Basics                    ✅
Fragmentation                    ✅
Paging                           ✅
Page Table / PTE / TLB           ✅
Multi-Level Paging               ✅
Demand Paging / Page Fault       ✅
FIFO                             ✅
Optimal                          ✅
LRU                              ✅
Second Chance / Clock            ⬜  ← NEXT
Thrashing                        ⬜
Segmentation                     ⬜
File Systems                     ⬜
Disk Scheduling                  ⬜
I/O Management                   ⬜
IPC                              ⬜
Protection / Security            ⬜
System Calls / Architecture      ⬜
```
