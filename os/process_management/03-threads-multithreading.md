# OS PROCESS MANAGEMENT — 03: THREADS & MULTITHREADING

## 1. What is a Thread?

A **thread** is the basic unit of execution inside a process.

A process may contain one or more threads.

Useful mental model:

> **Process = Resource container**  
> **Thread = Execution path inside that process**

---

# 2. Single-Threaded Process

```text
Process P1
│
├── Code
├── Data
├── Heap
│
└── Thread T1
     ├── Program Counter
     ├── CPU Registers
     └── Stack
```

Only one execution flow exists.

---

# 3. Multithreaded Process

```text
Process P1
│
├── Code        ← Shared
├── Data        ← Shared
├── Heap        ← Shared
├── Open Files  ← Shared
│
├── Thread T1
│   ├── PC
│   ├── Registers
│   └── Stack
│
├── Thread T2
│   ├── PC
│   ├── Registers
│   └── Stack
│
└── Thread T3
    ├── PC
    ├── Registers
    └── Stack
```

---

# 4. What Threads of the Same Process Share

Threads usually share:

- Code segment
- Data/global variables
- Heap
- Virtual address space
- Open files
- Sockets
- Process resources

Example:

```text
Shared Heap:
balance = 100

T1 → can access it
T2 → can access it
T3 → can access it
```

If T1 changes:

```text
balance = 200
```

T2 may observe the new value because the heap is shared.

---

# 5. What Is Private to Each Thread

Each thread normally has its own:

- Program Counter
- CPU registers
- Stack
- Stack pointer
- Thread ID
- Scheduling state
- Thread-local storage

Example:

```text
T1 Stack:
functionA()
x = 10

T2 Stack:
functionB()
x = 50
```

The stacks are separate.

---

# 6. Why Does Every Thread Need Its Own Stack?

Each thread has an independent function call sequence.

Example:

```text
T1 → functionA()
T2 → functionB()
```

Each call chain needs its own:

- Local variables
- Parameters
- Return addresses
- Saved execution state

Therefore:

> **Each thread requires its own stack.**

---

# 7. Why Does Every Thread Need Its Own Program Counter?

Threads can be at different locations in shared code.

```text
Shared Code:

Instruction 100
Instruction 200
Instruction 300
Instruction 500
```

At one moment:

```text
T1 PC → 100
T2 PC → 300
T3 PC → 500
```

Same code is shared, but execution position is independent.

---

# 8. Can Multiple Threads Execute the Same Function?

Yes.

Suppose:

```text
Shared function calculate()
```

Threads:

```text
T1 → calculate()
T2 → calculate()
T3 → calculate()
```

No separate copy of the function's code is required.

Each thread has its own:

- PC
- Registers
- Stack

Therefore they can independently execute the same shared machine instructions.

---

# 9. Same Instruction and Different Threads

Suppose shared code contains:

```text
Instruction 500 → read(file)
```

Current thread PCs:

```text
T1 PC = 150
T2 PC = 498
T3 PC = 700
```

T2 may reach instruction 500 next.

If all threads execute the same function/control path, different threads may execute instruction 500 at different times.

Threads do not randomly jump to an instruction merely because the code is shared.

> Their **control flow + Program Counter** determines which instruction each thread executes.

---

# 10. Process vs Thread

## Process

A process:

- Has a virtual address space
- Owns/contains resources
- Provides isolation
- Contains one or more threads

## Thread

A thread:

- Executes instructions
- Exists inside a process
- Shares process resources
- Has its own execution context

---

# 11. Process vs Thread Comparison

| Process | Thread |
|---|---|
| Heavyweight | Lightweight |
| Separate address space from other processes | Shares process address space |
| Better isolation | Lower isolation |
| Communication usually more expensive | Sharing/communication easier |
| Context switch generally heavier | Same-process thread switch generally lighter |
| Owns process resources | Shares process resources |
| Contains thread(s) | Execution unit |

---

# 12. Why Threads Are Lighter Than Processes

Creating another process may involve more independent state:

```text
New address space
Memory mappings
Page-table context
Process metadata
Resource bookkeeping
```

Creating another thread usually reuses the existing process environment and mainly needs:

```text
Thread stack
Register context
Program Counter
Thread metadata
Scheduling state
```

Therefore:

> Thread creation and switching are generally cheaper than creating/switching separate processes.

---

# 13. Thread Context Switch

Suppose:

```text
T1 → RUNNING
T2 → READY
```

Switch:

```text
Save T1 PC/registers
        ↓
Restore T2 PC/registers
        ↓
T2 RUNNING
```

If T1 and T2 are in the same process:

```text
Same address space remains active
```

which can make the switch cheaper than switching between different processes.

---

# 14. Concurrency vs Parallelism

## Concurrency

Multiple tasks make progress during overlapping time periods.

On one CPU core:

```text
T1 runs
 ↓
switch
 ↓
T2 runs
 ↓
switch
 ↓
T1 runs
```

They are concurrent, but not literally executing at the exact same instant.

## Parallelism

Different CPU cores execute tasks simultaneously:

```text
Core 1 → T1
Core 2 → T2
Core 3 → T3
```

Remember:

> **Concurrency = overlapping progress**  
> **Parallelism = simultaneous execution**

---

# 15. Why Multithreading Is Useful

Possible benefits:

- Better responsiveness
- Resource sharing
- Better I/O overlap
- Improved CPU utilization
- Parallelism on multicore CPUs
- Lower overhead than multiple independent processes for some workloads

Example:

```text
Web Server

T1 → Request A
T2 → Request B
T3 → Request C
```

---

# 16. Multithreading Is Not Automatically Faster

Performance depends on:

- Number of CPU cores
- Workload type
- Lock contention
- Scheduling overhead
- Shared-memory contention
- I/O behavior
- Cache effects

More threads can sometimes make performance worse.

---

# 17. Isolation Difference

Separate processes:

```text
Process P1 address space
Process P2 address space
```

are normally isolated.

Threads:

```text
Process P1
├── T1
├── T2
└── T3
```

share the same process memory.

Therefore a severe memory bug in one thread can affect the whole process.

---

# 18. Shared Memory Creates Synchronization Problems

Suppose:

```text
balance = 100
```

T1 wants:

```text
balance += 50
```

T2 wants:

```text
balance -= 20
```

Possible unsafe interleaving:

```text
T1 reads 100
T2 reads 100

T1 calculates 150
T2 calculates 80

T1 writes 150
T2 writes 80
```

Final:

```text
80
```

Expected logical result:

```text
130
```

This is a:

> **Race Condition**

It directly leads to:

- Critical Section
- Mutual Exclusion
- Locks
- Mutex
- Semaphores
- Synchronization

---

# 19. Threads and CPU Scheduling

In modern systems, CPU scheduling is often performed on **kernel-visible threads/schedulable entities**.

So when textbooks say:

```text
Process P1 is RUNNING
```

a more precise implementation-level interpretation may be:

```text
A thread belonging to process P1 is RUNNING
```

---

# 20. CPU Core Limitation

Suppose:

```text
10 runnable threads
4 CPU cores
```

Maximum truly simultaneous execution is roughly:

```text
4 execution streams
```

assuming one hardware execution context per considered core in the simplified model.

Remaining threads wait/runnable in scheduling queues.

---

# 21. Thread vs Process Failure Isolation

Threads:

```text
T1 severe memory bug
      ↓
Process may crash
      ↓
T2/T3 also disappear
```

Separate processes:

```text
P1 crashes
P2 may continue
```

This is one reason large applications sometimes use multiple processes for stronger fault isolation.

---

# IMPORTANT EXAM POINTS

1. Thread = basic execution unit inside a process.
2. A process may contain one or more threads.
3. Threads of one process share code, data, heap, address space, files.
4. Each thread has its own stack, PC and CPU registers.
5. Threads can execute different locations of the same shared code.
6. Multiple threads can execute the same function simultaneously.
7. Thread creation/switching is generally lighter than process creation/switching.
8. Process isolation is stronger than thread isolation.
9. Concurrency does not necessarily mean true parallelism.
10. Parallelism requires multiple available hardware execution resources/cores.
11. More threads do not automatically mean better performance.
12. Shared-memory threads introduce race-condition/synchronization problems.
13. Modern OS scheduling is often more accurately thought of at thread level.

---

# QUICK REVISION MAP

```text
                 PROCESS
                    │
       ┌────────────┴────────────┐
       │                         │
Shared Resources             Threads
├── Code                     ├── T1
├── Data                     ├── T2
├── Heap                     └── T3
├── Files
└── Address Space

Each Thread:
├── Program Counter
├── CPU Registers
└── Stack
```

```text
Concurrency:
T1 → T2 → T1 → T3

Parallelism:
Core 1 → T1
Core 2 → T2
```

Next topic:

```text
Shared Data
   ↓
Race Condition
   ↓
Critical Section
   ↓
Synchronization
```
