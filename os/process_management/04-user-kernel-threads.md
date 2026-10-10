# OS PROCESS MANAGEMENT — 04: USER-LEVEL & KERNEL-LEVEL THREADS

## 1. Core Question

The main question in this topic is:

> **Who manages and schedules the thread — a user-space runtime or the OS kernel?**

This gives us two useful concepts:

- User-Level Threads (ULT)
- Kernel-Level Threads (KLT)

Do NOT confuse them with:

- User Mode
- Kernel Mode

These are different concepts.

---

# 2. User-Level Thread (ULT)

A **User-Level Thread** is managed by a user-space runtime/thread library.

Conceptually:

```text
Process
│
├── U1
├── U2
├── U3
│
└── User-Space Thread Runtime
```

The kernel may not know each individual ULT.

User-space runtime can manage:

- Thread creation
- Thread switching
- Thread scheduling
- Runnable/waiting bookkeeping

without necessarily entering the kernel for every thread operation.

---

# 3. Why User-Level Thread Management Can Be Fast

If switching occurs entirely in user space:

```text
U1
 ↓
save user-thread context
 ↓
U2
```

the runtime may avoid a kernel transition.

Therefore ULT management can have:

- Low overhead
- Fast creation
- Fast switching
- Flexible application/runtime scheduling

---

# 4. Kernel-Level Thread (KLT)

A **Kernel-Level Thread** is directly known to the OS kernel.

Conceptually:

```text
Process P1

Kernel-visible execution entities:
K1
K2
K3
```

The OS scheduler can independently schedule them onto CPU cores.

```text
K1 → Core 1
K2 → Core 2
K3 → Core 3
```

---

# 5. Who Actually Gets CPU Time?

At OS scheduling level:

> **The OS scheduler gives CPU time to kernel-visible schedulable execution entities, commonly kernel threads.**

Conceptually:

```text
User-Level Work
      ↓
User Runtime
      ↓
Kernel Threads
      ↓
OS Scheduler
      ↓
CPU Cores
```

---

# 6. ULT and KLT Are Not "User Mode" and "Kernel Mode"

This is one of the most important distinctions.

### ULT / KLT

These describe:

> **thread representation/management/scheduling layers**

### User Mode / Kernel Mode

These describe:

> **CPU privilege levels**

So:

```text
User-Level Thread ≠ User Mode
Kernel-Level Thread ≠ Kernel Mode
```

---

# 7. System Call Does NOT Convert a ULT into a KLT

Suppose a thread reaches:

```text
read(file)
```

Conceptually:

```text
Thread executes application code
        ↓
USER MODE
        ↓
system call
        ↓
CPU enters KERNEL MODE
        ↓
Kernel handles request
        ↓
return
        ↓
USER MODE
```

The thread did not "turn into" a kernel thread.

The **CPU privilege mode changed** while kernel code executed on its behalf.

---

# 8. Shared Code and System Calls

Suppose shared process code contains:

```text
Instruction 500 → read(file)
```

Threads:

```text
T1 PC = 150
T2 PC = 498
T3 PC = 700
```

T2 is likely to reach the read instruction next.

If T2 reaches it:

```text
T2
 ↓
read(file)
 ↓
system call
```

What happens next depends on how user threads are mapped to kernel threads.

---

# 9. Thread Mapping Models

Main mapping models:

1. Many-to-One
2. One-to-One
3. Many-to-Many

These describe how:

```text
User-Level Threads
        ↓ map to
Kernel-Level Threads
```

---

# 10. Many-to-One Model

Many user threads map onto one kernel thread.

```text
U1 ─┐
U2 ─┼──→ K1
U3 ─┤
U4 ─┘
```

The user runtime chooses which ULT uses K1 at a given moment.

Example:

```text
K1 executing U1
       ↓
runtime switches
       ↓
K1 executing U3
```

From kernel perspective:

```text
K1 is the schedulable entity
```

---

## 11. Advantages of Many-to-One

- Cheap user-level thread management
- Low kernel-thread count
- User-space scheduling can be fast
- Thread creation can be lightweight

---

## 12. Disadvantages of Many-to-One

### Blocking Problem

Suppose U2 executes a blocking system call:

```text
U2
 ↓
K1 enters blocking operation
 ↓
K1 → BLOCKED
```

Since all ULTs depend on K1:

```text
U1
U3
U4
```

cannot execute through K1 while it is blocked.

### Limited Parallelism

Even with:

```text
8 CPU cores
```

if the process has only:

```text
1 kernel thread
```

then only one kernel execution stream from that process can run at a time.

---

# 13. One-to-One Model

Each user thread maps to a separate kernel thread.

```text
U1 → K1
U2 → K2
U3 → K3
```

The kernel sees:

```text
K1
K2
K3
```

and can schedule them independently.

---

## 14. Blocking in One-to-One

Suppose U2 performs blocking I/O:

```text
U2/K2
  ↓
K2 → BLOCKED
```

Other threads:

```text
K1 → can still run
K3 → can still run
```

Therefore one blocked thread does not necessarily block the whole process.

---

## 15. Advantages of One-to-One

- Independent blocking
- Easy kernel scheduling
- True multicore parallelism
- Kernel directly knows each schedulable thread

---

## 16. Disadvantages of One-to-One

Every user thread requires kernel-thread representation.

Large thread counts can therefore cause:

- More kernel bookkeeping
- More stack/resource usage
- Scheduling overhead
- Context-switch overhead

Example:

```text
100,000 application threads
        ↓
100,000 kernel threads
```

can be expensive.

---

# 17. Many-to-Many Model

Many user-level threads are multiplexed over multiple kernel threads.

Example:

```text
User Threads:
U1 U2 U3 U4 U5 U6

        ↓ runtime mapping

Kernel Threads:
K1 K2 K3
```

At one moment:

```text
K1 → U2
K2 → U4
K3 → U6
```

Later runtime may schedule:

```text
K1 → U1
K2 → U5
K3 → U3
```

---

# 18. Advantages of Many-to-Many

- Large number of lightweight user tasks
- More parallelism than many-to-one
- Fewer kernel threads than strict one-to-one
- Runtime can perform flexible scheduling
- One blocked KLT need not stop all user work if other KLTs remain available

---

# 19. Disadvantage of Many-to-Many

The runtime/OS interaction is more complex.

It must manage:

- User-thread scheduling
- Kernel-thread availability
- Blocking behavior
- Mapping of user tasks to kernel execution contexts

---

# 20. Many-to-Many Blocking Example

Suppose:

```text
ULTs: U1 U2 U3 U4 U5
KLTs: K1 K2

Currently:
K1 → U2
K2 → U4
```

U2 performs blocking I/O.

Possible result:

```text
K1 → BLOCKED
K2 → still runnable
```

Therefore:

```text
U4 can continue on K2
```

and the runtime may later schedule other runnable ULTs on available kernel-thread capacity.

---

# 21. Mapping Model Comparison

| Model | Mapping | Parallelism | Blocking Behavior | Overhead |
|---|---|---|---|---|
| Many-to-One | Many ULT → 1 KLT | Limited | One blocking KLT can stop all mapped ULTs | Low |
| One-to-One | 1 ULT → 1 KLT | Good | Other KLTs can continue | Higher |
| Many-to-Many | Many ULT → Multiple KLT | Good/flexible | Other KLTs can continue | More runtime complexity |

---

# 22. User-Level Thread States

A user runtime can maintain states such as:

```text
U1 → Runnable
U2 → Running on K1
U3 → Waiting
U4 → Runnable
```

These are runtime-level states.

The OS kernel may not know U1/U2/U3/U4 individually.

---

# 23. Kernel-Level Thread States

Kernel-visible threads can move through familiar scheduling states:

```text
READY
  ↓
RUNNING
  ↓
WAITING / BLOCKED
  ↓
READY
  ↓
RUNNING
  ↓
EXIT
```

This is the most precise scheduling-level interpretation in a modern multithreaded OS.

---

# 24. Relationship to the Classic Process Lifecycle

Textbooks often use:

```text
Process:
READY → RUNNING → WAITING
```

In a single-threaded process, this is a good abstraction.

In a multithreaded implementation:

```text
Process owns resources
       ↓
Kernel-visible thread(s)
       ↓
READY / RUNNING / WAITING
       ↓
CPU
```

So the actual schedulable entities are often threads.

---

# 25. CPU Core Limitation

Suppose:

```text
10 ULTs
10 KLTs
4 CPU cores
```

Maximum truly simultaneous execution in the simplified core model:

```text
4 threads
```

because only four cores are available.

The remaining runnable threads wait for scheduling.

---

# 26. User Thread and Kernel Thread — Better Mental Model

Do NOT imagine:

```text
Process contains only ULTs
KLTs are unrelated external processes
```

Better:

```text
Process
│
├── Shared resources/address space
│
└── Execution work
      ↓
User/runtime thread representation
      ↓ mapping
Kernel schedulable thread representation
      ↓
OS Scheduler
      ↓
CPU
```

In a one-to-one model, the user-side and kernel-side representations closely correspond to the same logical execution path.

---

# 27. Node.js Connection

Node.js is often called "single-threaded" because normal JavaScript execution mainly occurs on one main JS thread.

However the complete Node process may use:

- libuv thread pool
- OS threads
- worker threads
- Kernel asynchronous I/O

So:

> **Single-threaded JavaScript ≠ entire process has exactly one OS thread**

---

# 28. Go Connection

Go goroutines are lightweight runtime-managed execution units.

Conceptually:

```text
Many Goroutines
      ↓
Go Runtime Scheduler
      ↓
Smaller set of OS/Kernel Threads
      ↓
CPU Cores
```

This resembles a many-to-many style idea.

Therefore:

> **Goroutine ≠ OS/kernel thread**

---

# IMPORTANT EXAM TRAPS

## Trap 1

"User-level threads always mean user mode."

❌ Wrong.

ULT/KLT and user/kernel mode are different concepts.

---

## Trap 2

"A user thread becomes a kernel thread when it makes a system call."

❌ Wrong.

The CPU enters kernel mode while the kernel performs work on behalf of the executing thread.

---

## Trap 3

"User-level threads can never run in parallel."

❌ Too broad.

Correct:

> In a many-to-one model with only one KLT, they cannot obtain true multicore parallel execution through that one KLT.

With multiple underlying kernel threads, user-level tasks can participate in parallel execution.

---

## Trap 4

"If one thread blocks, the whole process always blocks."

❌ Wrong.

It depends on the mapping model.

- Many-to-one: can happen
- One-to-one: other KLTs can continue
- Many-to-many: other KLTs may continue

---

## Trap 5

"Kernel thread means the CPU is always in kernel mode."

❌ Wrong.

A kernel-visible thread can represent execution of application code in user mode and then enter kernel mode for system calls/interrupt handling.

---

# IMPORTANT EXAM POINTS

1. ULT is managed primarily by user-space runtime/library.
2. KLT is known to and scheduled by the kernel.
3. OS scheduler gives CPU time to kernel-visible schedulable entities.
4. ULT/KLT are not privilege modes.
5. User mode/kernel mode are CPU privilege levels.
6. System call causes controlled entry into kernel mode.
7. System call does not convert ULT into KLT.
8. Many-to-one = many ULTs on one KLT.
9. Many-to-one is lightweight but has blocking/parallelism limitations.
10. One-to-one = each ULT has a KLT.
11. One-to-one allows independent blocking and multicore parallelism.
12. One-to-one can become expensive with huge thread counts.
13. Many-to-many maps many ULTs to multiple KLTs.
14. Many-to-many provides flexible runtime scheduling and parallelism.
15. Kernel-thread states fit READY/RUNNING/WAITING/EXIT scheduling model.
16. Actual parallel execution is limited by available hardware cores/execution contexts.

---

# QUICK REVISION MAP

```text
USER-SPACE WORK

U1 U2 U3 U4 U5
      │
      ↓
User Thread Runtime
      │
      ↓ mapping
K1 K2 K3
      │
      ↓
OS Scheduler
      │
      ↓
CPU Cores
```

### Many-to-One

```text
U1 ─┐
U2 ─┼──→ K1
U3 ─┘
```

### One-to-One

```text
U1 → K1
U2 → K2
U3 → K3
```

### Many-to-Many

```text
U1 U2 U3 U4
 \ | /  \ |
  K1     K2
```

### Privilege Transition

```text
Thread executing app code
        ↓
USER MODE
        ↓
system call
        ↓
KERNEL MODE
        ↓
return
        ↓
USER MODE
```

Final mental model:

> **ULT/KLT tells us how execution threads are managed and mapped.**  
> **User/Kernel Mode tells us what privilege level the CPU is currently using.**
