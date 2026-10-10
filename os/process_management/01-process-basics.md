# OS PROCESS MANAGEMENT — 01: PROCESS BASICS

## 1. Program vs Process

### Program
A **program** is a passive set of instructions stored on secondary storage (SSD/HDD).

Examples:

```text
calculator.exe
chrome.exe
a.out
```

A program is not executing by itself.

### Process
A **process** is a program in execution.

A better mental model:

> **Process = Program + Execution State + Allocated Resources**

A running process can have:

- Code
- Data
- Heap
- Stack
- CPU register state
- Program Counter
- Open files
- Sockets
- PID
- Scheduling information
- Memory-management information

### One Program Can Create Multiple Processes

```text
              calculator program
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
        Process    Process    Process
         P1         P2         P3
```

The program file may be the same, but each process has its own execution state.

---

## 2. Program → Process

Conceptually:

```text
Source Code
    ↓
Compiler / Linker
    ↓
Executable File
    ↓
User runs executable
    ↓
OS creates process
    ↓
Process becomes runnable
    ↓
Scheduler gives CPU time
```

The executable is stored on disk.

When it is executed, the OS creates a process and establishes the resources needed for execution.

---

## 3. Typical Process Virtual Address Space

A process is normally given a **virtual address space**.

Conceptually:

```text
High Address

+----------------------+
|        Stack         |
|          ↓           |
|                      |
|                      |
|          ↑           |
|         Heap         |
+----------------------+
|         BSS          |
+----------------------+
|     Data Segment     |
+----------------------+
|     Text / Code      |
+----------------------+

Low Address
```

### Text / Code Segment

Contains compiled machine instructions.

Usually read-only.

Example source:

```c
x = x + 10;
```

The corresponding machine instructions live in the code/text area.

### Data Segment

Contains initialized global/static variables.

```c
int x = 100;
```

### BSS

Contains uninitialized or zero-initialized global/static variables.

```c
int x;
```

### Heap

Used for dynamic memory allocation.

Conceptually:

```text
malloc/new
    ↓
Heap
```

The heap often grows upward.

### Stack

Used for function execution state, such as:

- Local variables
- Function parameters
- Return addresses
- Saved registers

The stack often grows downward.

---

## 4. Virtual Address → Physical Address

A process normally works with **virtual addresses**, not raw physical RAM addresses.

Conceptually:

```text
CPU generates virtual address
        ↓
       MMU
        ↓
Page Table / TLB translation
        ↓
Physical Address
        ↓
RAM
```

The **MMU (Memory Management Unit)** performs address translation using memory-management structures configured by the OS.

This connects Process Management with Paging/Virtual Memory.

---

## 5. Process Is More Than Memory

A process is not just:

```text
Code + Heap + Stack
```

It also has execution and resource state.

```text
Process
│
├── Address Space
│   ├── Code
│   ├── Data
│   ├── Heap
│   └── Stack(s)
│
├── CPU State
│   ├── Program Counter
│   ├── Registers
│   └── Stack Pointer
│
├── OS Resources
│   ├── Open files
│   ├── Sockets
│   └── Permissions
│
└── Metadata
    ├── PID
    ├── State
    ├── Priority
    └── Scheduling information
```

---

## 6. PCB — Process Control Block

The operating system must remember important information about every process.

This information is maintained in a kernel data structure called the:

> **PCB — Process Control Block**

Conceptually:

```text
Process
   ↓
OS representation
   ↓
PCB
```

Typical PCB information:

```text
PCB
│
├── Process ID (PID)
├── Process State
├── Program Counter
├── CPU Registers
├── Scheduling Information
│   ├── Priority
│   └── Queue information
├── Memory-Management Information
│   ├── Page-table/address-space info
│   └── Memory mappings
├── I/O Information
│   ├── Open files
│   └── Devices
└── Accounting Information
    ├── CPU time
    └── Owner/user
```

Exact contents depend on the operating system.

---

## 7. Program Counter

The **Program Counter (PC)** tells the CPU where execution should continue.

Suppose:

```text
Instruction 100
Instruction 101
Instruction 102
Instruction 103
```

If execution is interrupted around instruction 102, the OS must preserve the appropriate next execution address.

That is why the Program Counter is part of the saved execution context.

---

## 8. CPU Registers

During execution, registers may contain temporary but essential values:

```text
R1 = 10
R2 = 20
R3 = 30
```

If another process/thread starts using the CPU, these registers will change.

Therefore during a context switch:

```text
Current CPU register state
          ↓ save
PCB / thread context
```

Later:

```text
Saved state
    ↓ restore
CPU registers
```

---

## 9. Basic Process States

Classic five-state model:

```text
NEW
 ↓
READY
 ↓
RUNNING
 ↓
TERMINATED
```

A running process can also wait for an event:

```text
               ┌───────────────┐
               │               ↓
NEW → READY → RUNNING → WAITING/BLOCKED
       ↑         │              │
       │         │              │ event/I/O completes
       └─────────┘              │
       preemption               ↓
                              READY
```

---

## 10. Meaning of Each State

### NEW
The process is being created.

### READY
The process can execute, but is waiting for CPU time.

> **READY = waiting for CPU**

### RUNNING
The process/thread is currently executing on a CPU core.

### WAITING / BLOCKED
The process cannot currently make progress because it is waiting for an event/resource.

Examples:

- Disk I/O
- Network response
- Keyboard input
- Synchronization event

> **WAITING ≠ waiting for CPU**

### TERMINATED
The process has completed or has been killed/terminated.

---

## 11. Important State Transitions

### NEW → READY
Process creation completes and the process becomes runnable.

### READY → RUNNING
Scheduler selects it.

```text
READY
  ↓ scheduler dispatch
RUNNING
```

### RUNNING → READY
Usually caused by **preemption**.

Example:

- Time quantum expires
- Higher-priority runnable work appears

### RUNNING → WAITING
The process requests I/O or waits for some event.

### WAITING → READY
The event/I/O completes.

Important:

```text
WAITING → READY
```

not usually:

```text
WAITING → RUNNING
```

because it must again be selected by the scheduler.

### RUNNING → TERMINATED
Execution completes or process is terminated.

---

## 12. READY vs WAITING — Important Exam Trap

### READY

```text
"I can execute.
I only need CPU."
```

### WAITING

```text
"Even if you give me CPU,
I cannot proceed yet."
```

Example:

```text
P1 → READY
waiting for CPU

P2 → WAITING
waiting for disk read
```

---

## 13. Context Switching

A **context switch** occurs when the CPU stops executing one schedulable execution context and begins another.

Conceptually:

```text
P1 / T1 running
      ↓
Save its execution context
      ↓
Load P2 / T2 context
      ↓
P2 / T2 running
```

Execution context may include:

- Program Counter
- Registers
- Stack pointer
- Processor status
- Other architecture/OS-specific state

### Example

```text
P1:
PC = 1200
R1 = 50
R2 = 20
```

OS saves it, then restores:

```text
P2:
PC = 850
R1 = 90
R2 = 12
```

P1 does **not** restart from the beginning when scheduled again.

---

## 14. Context Switch Is Overhead

During switching, CPU time is used for management rather than direct application work.

```text
Too many context switches
        ↓
Overhead increases
```

However, switching is essential for responsiveness and multitasking.

---

## 15. Process vs Thread-Level Scheduling

Textbooks often say:

```text
Process → READY → RUNNING → WAITING
```

This is fine for conceptual/basic models.

In modern multithreaded systems, the OS scheduler often schedules **threads or kernel-visible schedulable entities**.

So a more precise modern picture is:

```text
Process owns resources
       ↓
Threads execute work
       ↓
Kernel-visible threads/entities
       ↓
OS Scheduler
       ↓
CPU cores
```

---

# USER MODE AND KERNEL MODE

## 16. User Mode

User mode is a **restricted CPU privilege mode**.

Normal application code usually executes here.

Applications normally cannot directly:

- Modify page tables
- Access arbitrary kernel memory
- Disable interrupts
- Execute privileged hardware instructions
- Directly control arbitrary devices

---

## 17. Kernel Mode

Kernel mode is a **privileged CPU mode**.

The OS kernel can:

- Manage memory
- Schedule processes/threads
- Handle interrupts
- Access devices
- Execute privileged instructions

---

## 18. System Call

When application code needs a privileged OS service, it performs a **system call**.

Conceptually:

```text
Application code
     ↓
USER MODE
     ↓
system call
     ↓
CPU enters KERNEL MODE
     ↓
Kernel performs requested operation
     ↓
return
     ↓
USER MODE
```

Important:

> A process does not contain one permanent "user half" and one permanent "kernel half".

Instead, a thread executes application code in user mode and the CPU may temporarily enter kernel mode to execute kernel code on its behalf.

---

## 19. Function Call ≠ System Call

A normal library/application function call does not automatically require kernel mode.

Example:

```text
calculatePhysics()
```

may execute entirely in user mode.

A function such as `printf()` may perform user-space work first and eventually invoke an OS write system call when output must reach a device/file.

---

# COMPILATION, HEADERS AND LINKING — USEFUL PROCESS CONTEXT

## 20. Header Files

A C/C++ header such as:

```c
#include <stdio.h>
```

primarily provides declarations/types/interfaces to the compiler.

Conceptually:

```c
int printf(const char *format, ...);
```

The complete library implementation is not normally copied from the header into the executable.

---

## 21. Static Linking

With static linking, required compiled library code becomes part of the final executable.

```text
Your object code
      +
Static library code
      ↓
    Linker
      ↓
Executable
```

General effect:

> Statically linked executables are usually larger.

---

## 22. Dynamic Linking

With dynamic/shared linking, library code exists separately, e.g.:

```text
Linux   → .so
Windows → .dll
```

The executable references the shared library.

At runtime, library code can be mapped into the process virtual address space.

```text
Process Virtual Address Space
│
├── Application Code
├── Heap
├── Shared Library Mapping
└── Stack
```

Important:

> A dynamically linked library does NOT normally become a separate process.

Its code executes as part of the calling process's execution.

---

## 23. Shared Library Memory

Multiple processes can map the same read-only shared-library code pages:

```text
P1 virtual page ─┐
P2 virtual page ─┼──→ same physical read-only library frame(s)
P3 virtual page ─┘
```

This connects dynamic linking with virtual memory and paging.

---

# IMPORTANT EXAM POINTS

1. Program is passive; process is active.
2. One program can have multiple processes.
3. Process = program + execution state + resources.
4. Every process normally has a virtual address space.
5. MMU translates virtual addresses to physical addresses using OS-managed mappings.
6. PCB stores process-related execution and management information.
7. Program Counter identifies where execution continues.
8. READY = waiting for CPU.
9. WAITING/BLOCKED = waiting for an event/resource.
10. RUNNING → READY generally indicates preemption.
11. RUNNING → WAITING occurs on blocking/event wait.
12. WAITING → READY occurs when the event completes.
13. Context switching saves one execution context and restores another.
14. Context switching has overhead.
15. User mode/kernel mode are CPU privilege modes.
16. System calls cause controlled entry into kernel mode.
17. Static linking embeds required library machine code in the executable.
18. Dynamic linking keeps shared-library code separately mapped/referenced.

---

# QUICK REVISION MAP

```text
Program on Disk
      ↓ execute
Process
      ↓
Virtual Address Space
├── Code
├── Data/BSS
├── Heap
└── Stack
      ↓
OS maintains
      ↓
PCB
├── PID
├── State
├── PC
├── Registers
├── Scheduling info
└── Memory/I/O info

States:
NEW → READY → RUNNING → TERMINATED
               │
               ↓
             WAITING
               │
               ↓
             READY

Context Switch:
Save current state
        ↓
Restore next state
        ↓
Continue execution
```
