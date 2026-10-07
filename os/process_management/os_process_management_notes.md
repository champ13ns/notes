# Operating Systems — Process Management Notes

> **Goal:** Strong conceptual understanding + RRB SO IT / banking IT exam revision  
> **Scope:** Process Management topics covered so far. Memory Management/Paging is kept separate.

---

## 1. Program vs Process

### Program
A **program** is a passive file stored on secondary storage (SSD/HDD).

Examples:

```text
app.exe
a.out
chrome.exe
```

A program contains instructions, code, constants, etc., but it is **not executing**.

### Process
A **process** is a program that is currently being executed.

```text
Program on Disk
      ↓
     Run
      ↓
Process is Created
```

A process has:

- its own virtual address space
- CPU execution context
- registers
- program counter
- stack
- heap
- process state
- OS bookkeeping information

### Exam Trap

```text
Program = passive
Process = active
```

---

## 2. Executable → Process

Typical flow:

```text
Source Code
   ↓
Compiler
   ↓
Executable / Binary
   ↓
Stored on Disk
   ↓
User runs it
   ↓
OS creates Process
```

The executable file itself is not a process.

When executed, the OS creates the required process-management structures and virtual address space.

---

## 3. Process vs Thread

A program does **not simply get converted into many threads**.

Better model:

```text
Program
   ↓
Process
   ↓
One or More Threads
```

A process normally starts with at least one execution thread and may create additional threads.

### Process owns resources

Examples:

- virtual address space
- heap
- code/data
- open files
- OS resources

### Thread is an execution path

Threads of the same process generally share:

- code
- heap
- global data
- process resources

Each thread has its own:

- program counter
- CPU registers
- stack

Conceptually:

```text
Process
│
├── Shared Code
├── Shared Heap
├── Shared Data
│
├── Thread 1
│   ├── Stack
│   ├── Registers
│   └── Program Counter
│
└── Thread 2
    ├── Stack
    ├── Registers
    └── Program Counter
```

---

## 4. Process Memory Layout

A process sees a virtual address space roughly like:

```text
High Address
+----------------------+
| Stack                |
| grows downward       |
+----------------------+
|                      |
| Free / Mapped Area   |
|                      |
+----------------------+
| Heap                 |
| grows upward         |
+----------------------+
| BSS                  |
+----------------------+
| Data                 |
+----------------------+
| Code / Text          |
+----------------------+
Low Address
```

### Code/Text
Compiled machine instructions.

### Data
Initialized global/static variables.

### BSS
Uninitialized global/static variables.

### Heap
Dynamic memory, e.g.:

```c
malloc()
new
```

### Stack
Function calls, local variables, return addresses, etc.

---

## 5. Program Counter (PC)

The **Program Counter is NOT the whole process data structure**.

It is a CPU register that stores the address of the next instruction to execute.

Example:

```text
PC = address of next machine instruction
```

When a process is interrupted, the OS saves its program counter so execution can later resume from the correct instruction.

---

## 6. PCB — Process Control Block

The OS keeps information about every process in a structure called the:

**Process Control Block (PCB)**

Conceptually:

```text
PCB
├── Process ID (PID)
├── Process State
├── Program Counter
├── CPU Registers
├── Scheduling Information
├── Memory-Management Information
├── Open File / I/O Information
└── Accounting / Other Metadata
```

Important:

> Stack and heap are not literally stored inside the PCB.

The PCB stores **management information about the process**.

---

## 7. Process States

Basic 3-state model:

```text
READY → RUNNING → WAITING
          ↓
         EXIT
```

More complete common model:

```text
NEW
 ↓
READY
 ↓
RUNNING
 ├──→ WAITING/BLOCKED
 │       ↓
 │     READY
 │
 ├──→ READY
 │
 └──→ TERMINATED
```

---

## 8. Meaning of Each State

### NEW
Process is being created.

### READY
Process is ready to execute and is waiting for CPU time.

Important:

> **READY does NOT mean the process is on disk.**

Ready is a **CPU scheduling state**, not a memory-location description.

### RUNNING
A CPU/core is currently executing the process/thread.

### WAITING / BLOCKED
The process cannot continue until some event completes.

Typical reason:

```text
I/O request
```

Example:

```text
Process requests disk read
        ↓
Process cannot continue immediately
        ↓
Waiting / Blocked
```

### TERMINATED / EXIT
Process has completed or has been terminated.

---

## 9. Important State Transitions

### Ready → Running
CPU scheduler chooses the process.

```text
Ready Queue
    ↓
Scheduler
    ↓
Dispatcher
    ↓
Running
```

### Running → Waiting
Process requests I/O or waits for an event.

```text
Running
   ↓
I/O request
   ↓
Waiting
```

### Waiting → Ready
I/O/event completes.

```text
Waiting
   ↓
I/O complete
   ↓
Ready
```

### Running → Ready
Can happen due to:

- time quantum expiry
- higher-priority process
- preemption

### Running → Terminated
Process finishes execution.

---

## 10. CPU Scheduler vs Dispatcher

These two are related but different.

### Scheduler
Chooses **which ready process/thread should run next**.

### Dispatcher
Actually gives CPU control to the selected process/thread.

Dispatcher work may include:

- context switch
- restoring registers
- restoring program counter
- switching to user mode
- starting/resuming execution

Important:

```text
Dispatcher ≠ Disk → RAM loader
```

Dispatcher is primarily related to **CPU execution/context switching**.

---

## 11. Ready Queue

Processes/threads that are ready to execute wait in the ready queue.

Conceptually:

```text
Ready Queue

P3 → P7 → P2 → P9
           ↓
       Scheduler
           ↓
          CPU
```

The exact implementation can vary.

---

## 12. Context Switch

Suppose CPU is running Process A and must switch to Process B.

OS must save A's execution context:

```text
Process A
├── Program Counter
├── Registers
├── Stack Pointer
└── Other CPU State
```

Then restore B's saved context.

```text
Save A
  ↓
Load B
  ↓
CPU continues B
```

This is a **context switch**.

### Context Switch is Overhead
During a context switch, useful application work is temporarily paused while the OS changes execution context.

---

## 13. CPU-Bound vs I/O-Bound Processes

### CPU-Bound
Spends most of its time doing computation.

Examples:

- heavy calculations
- video encoding
- scientific computation

Typical behavior:

```text
Long CPU bursts
Few I/O waits
```

### I/O-Bound
Frequently waits for I/O.

Examples:

- file reads
- database/network operations
- user input

Typical behavior:

```text
Short CPU bursts
Frequent I/O waits
```

This distinction matters in CPU scheduling.

---

## 14. Process State and Memory Location Are Different Concepts

Do not mix these:

### CPU Scheduling Question

```text
Ready
Running
Waiting
```

answers:

> What is the process doing with respect to the CPU?

### Memory Management Question

```text
Page in RAM?
Page not in RAM?
Frame number?
```

answers:

> Where is the process's memory currently resident?

A process can be:

```text
RUNNING
```

while only some of its virtual pages are physically resident in RAM.

---

## 15. fork()

On Unix-like systems:

```c
fork();
```

creates a new process.

After `fork()`:

```text
Parent Process
     |
   fork()
     |
     +--------+
     |        |
  Parent    Child
```

The child gets its own process identity.

Conceptually, parent and child have separate virtual address spaces.

Modern systems may initially use **Copy-on-Write (COW)** rather than physically copying every memory page immediately.

---

## 16. exec()

`exec()` does not normally create a new process.

Instead, it replaces the current process's program image with another program.

Conceptually:

```text
Existing Process
     ↓
   exec()
     ↓
Same process identity
but new program/code image
```

Classic pattern:

```text
fork()
  ↓
Child Process
  ↓
exec()
  ↓
Child starts another program
```

---

## 17. fork() vs exec()

| Topic | fork() | exec() |
|---|---|---|
| Creates new process? | Yes | No |
| New PID? | Child gets new PID | Normally same process |
| Address space | Child gets its own logical process address space | Current program image is replaced |
| Common use | Create child | Run another program |

---

## 18. Race Condition

A race condition happens when multiple threads/processes access shared data concurrently and the final result depends on execution order.

Example:

```text
counter = 10
```

Thread A and Thread B both execute:

```text
counter++
```

Possible internal steps:

```text
Read
Modify
Write
```

If they interleave badly, final result may become:

```text
11
```

instead of:

```text
12
```

This is a race condition.

---

## 19. Critical Section

The part of code that accesses shared data/resources is called a:

**Critical Section**

Goal:

> Prevent unsafe simultaneous execution of critical sections.

---

## 20. Mutex

Mutex = **Mutual Exclusion Lock**

Typical behavior:

```text
lock()
   ↓
Critical Section
   ↓
unlock()
```

Only one thread should hold the mutex at a time.

Important property:

> Mutex has ownership.

Usually, the thread that locks it should unlock it.

---

## 21. Semaphore

A semaphore is a synchronization primitive based on a counter.

Example:

```text
Semaphore S = 5
```

means up to 5 permitted units/users may access the controlled resource concurrently.

Conceptually:

```text
wait(S)
   ↓
Use Resource
   ↓
signal(S)
```

### Binary Semaphore

Values conceptually around:

```text
0 / 1
```

Can behave similarly to a lock in some use cases.

### Counting Semaphore

Example:

```text
S = 5
```

allows up to five simultaneous users/resources.

---

## 22. Mutex vs Semaphore

| Feature | Mutex | Semaphore |
|---|---|---|
| Main purpose | Mutual exclusion | Signaling/resource counting |
| Counter | Usually lock/unlock state | Integer count |
| Ownership | Yes | Usually no ownership concept |
| Multiple permits | No | Yes for counting semaphore |

Exam trap:

> Semaphore is not simply "a mutex with a different name."

---

## 23. Deadlock

Deadlock occurs when processes/threads wait indefinitely for resources held by one another.

Example:

```text
Process P1 holds Resource A
P1 waits for Resource B

Process P2 holds Resource B
P2 waits for Resource A
```

Neither can proceed.

---

## 24. Four Necessary Conditions for Deadlock

All four must hold simultaneously for deadlock to be possible.

### 1. Mutual Exclusion
At least one resource is non-shareable.

### 2. Hold and Wait
Process holds at least one resource while waiting for another.

### 3. No Preemption
Resource cannot simply be forcibly taken away.

### 4. Circular Wait
Circular dependency exists.

Example:

```text
P1 waits for P2
P2 waits for P3
P3 waits for P1
```

Mnemonic:

```text
M H N C
Mutual Exclusion
Hold and Wait
No Preemption
Circular Wait
```

---

## 25. Deadlock Prevention vs Avoidance

### Prevention
Design system so at least one necessary deadlock condition cannot occur.

### Avoidance
Allow requests dynamically, but grant them only if system remains in a **safe state**.

Banker's Algorithm is a classic deadlock-avoidance algorithm.

---

## 26. Banker's Algorithm — Core Idea

The OS checks:

> If I grant this resource request, can all processes still finish in some safe sequence?

Important terms:

```text
Available
Max
Allocation
Need
```

Formula:

```text
Need = Max - Allocation
```

If a safe sequence exists:

```text
System is in Safe State
```

Safe state does not mean no waiting occurs.

It means:

> There exists at least one order in which all processes can finish.

---

## 27. Safe State vs Deadlock

```text
Safe State
→ guaranteed existence of some completion sequence
```

```text
Unsafe State
→ may lead to deadlock
```

Important:

> Unsafe state is not automatically the same as deadlock.

---

## 28. Process Scheduling — Current Status

We have covered the **conceptual flow**:

```text
Ready
Running
Waiting
Dispatcher
Context Switch
```

But detailed scheduling algorithms were intentionally not studied deeply yet.

Pending topics include:

- FCFS
- SJF / SRTF
- Priority Scheduling
- Round Robin
- Waiting Time / Turnaround Time numericals

---

# 29. Big Picture — Complete Process Lifecycle

```text
Program / Executable on Disk
            ↓
          Run
            ↓
     OS Creates Process
            ↓
  Virtual Address Space Created
            ↓
           NEW
            ↓
          READY
            ↓
        Scheduler
            ↓
        Dispatcher
            ↓
         RUNNING
       /    |     \
      /     |      \
 I/O Wait  Preempt  Finish
    ↓        ↓        ↓
 WAITING    READY   TERMINATED
    ↓
 I/O Complete
    ↓
  READY
```

At the same time, memory management operates independently:

```text
Virtual Memory of Process
          ↓
Pages
          ↓
MMU / Page Table
          ↓
Physical Frames in RAM
```

CPU scheduling answers:

> Who gets CPU?

Memory management answers:

> Where is the required memory?

---

# 30. High-Priority Exam Facts

1. **Program is passive; process is active.**
2. **PCB stores process-management information.**
3. **Program Counter stores address of next instruction.**
4. **Ready state means waiting for CPU, not waiting on disk.**
5. **Running → Waiting usually occurs due to I/O/event wait.**
6. **Waiting → Ready occurs after the event/I/O completes.**
7. **Dispatcher gives CPU control to the process selected by scheduler.**
8. **Context switching has overhead.**
9. **Threads of same process share code/data/heap but have separate stacks/registers/PCs.**
10. **fork() creates a child process; exec() replaces the current program image.**
11. **Mutex has ownership; semaphore is a counter-based synchronization primitive.**
12. **Deadlock needs all four Coffman conditions.**
13. **Banker's Algorithm is deadlock avoidance, not prevention.**
14. **Need = Max - Allocation.**
15. **Unsafe state does not necessarily mean deadlock already exists.**

---

# 31. Quick Revision Tree

```text
PROCESS MANAGEMENT
│
├── Program vs Process
│   ├── Executable
│   └── Process Creation
│
├── Process Structure
│   ├── Code
│   ├── Data/BSS
│   ├── Heap
│   └── Stack
│
├── PCB
│   ├── PID
│   ├── State
│   ├── Program Counter
│   ├── Registers
│   └── Scheduling/Memory Info
│
├── Process States
│   ├── New
│   ├── Ready
│   ├── Running
│   ├── Waiting
│   └── Terminated
│
├── CPU Scheduling Basics
│   ├── Ready Queue
│   ├── Scheduler
│   ├── Dispatcher
│   └── Context Switch
│
├── Process / Thread
│   ├── Shared Resources
│   └── Per-thread Stack/Registers/PC
│
├── Process Creation
│   ├── fork()
│   └── exec()
│
├── Synchronization
│   ├── Race Condition
│   ├── Critical Section
│   ├── Mutex
│   └── Semaphore
│
└── Deadlock
    ├── 4 Necessary Conditions
    ├── Prevention
    ├── Avoidance
    └── Banker's Algorithm
```

---

## Next Topic

After this revision, continue with:

```text
Memory Management
   ↓
Paging
   ↓
Demand Paging
   ↓
Page Replacement Algorithms
   ↓
FIFO
Optimal
LRU
Second Chance / Clock
```
