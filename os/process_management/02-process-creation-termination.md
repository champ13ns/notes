# OS PROCESS MANAGEMENT — 02: PROCESS CREATION & TERMINATION

## 1. Parent and Child Processes

A running process can create another process.

The creating process is called the:

> **Parent Process**

The newly created process is called the:

> **Child Process**

Conceptually:

```text
Parent P1
   │
   └── creates
        ↓
      Child P2
```

---

# 2. `fork()` — Basic Idea

In Unix/Linux-style systems, `fork()` creates a new child process.

Before:

```text
P1
```

After:

```text
        P1
       /  \
 Parent   Child
          P2
```

Important:

> `fork()` creates a new **process**, not a new executable/program file.

Both processes continue from the logical execution point immediately after the `fork()` call.

---

## 3. Who Runs the Child?

`fork()` does not directly "run" the child by magic.

Conceptually:

```text
Parent calls fork()
        ↓
System call enters kernel
        ↓
Kernel creates child process
        ↓
Child becomes runnable/READY
        ↓
OS Scheduler
        ↓
CPU executes parent or child
```

The scheduling order is not guaranteed.

---

## 4. Parent vs Child After `fork()`

Suppose:

```text
Before fork:
P1 is running
```

After:

```text
P1 → parent
P2 → child
```

Both continue from the next logical instruction after `fork()`.

So:

```text
fork();
next_instruction;
```

can be executed by both parent and child.

---

## 5. `fork()` Return Value

A successful `fork()` returns different values in parent and child.

```text
Parent → child PID (> 0)
Child  → 0
Failure → negative value
```

This allows the program to distinguish parent and child execution paths.

Conceptually:

```text
if fork result == 0
    child path
else
    parent path
```

---

# 6. Parent/Child Scheduling Order Is Not Guaranteed

After `fork()`:

```text
Parent may run first
```

or:

```text
Child may run first
```

or their executions may be interleaved.

Important:

> Source-code lines being adjacent does NOT guarantee the parent executes the next line first.

Scheduling happens at machine-instruction/execution level.

---

# 7. Separate Virtual Address Spaces

Parent and child are separate processes.

Therefore logically:

```text
Parent Virtual Address Space
≠
Child Virtual Address Space
```

Initially they may contain identical-looking data.

Example:

```text
Before fork:
x = 10
```

After fork:

```text
Parent sees x = 10
Child sees x = 10
```

If parent later changes:

```text
x = 50
```

the child does not simply become `50` because each process has its own logical address space.

---

# 8. Copy-on-Write (COW)

Copying every memory page immediately at `fork()` would be expensive.

Modern systems commonly use:

> **Copy-on-Write**

Initially:

```text
Parent virtual page ─┐
                     ├──→ Same physical frame
Child virtual page ──┘
```

The page can be shared while neither process modifies it.

If parent attempts to write:

```text
Parent writes
    ↓
COW protection triggers
    ↓
OS creates/copies private page
    ↓
Parent mapping updated
```

After:

```text
Parent VA → Physical Frame A (modified)
Child VA  → Physical Frame B (old value)
```

Important:

> Parent and child remain logically isolated even if physical pages are initially shared.

---

# 9. Multiple `fork()` Calls

Example:

```text
fork();
fork();
```

First fork:

```text
1 process → 2 processes
```

Second fork is reached by both existing processes:

```text
2 processes → 4 processes
```

For `n` unconditional successful forks where every process reaches every fork:

> **Total processes = 2^n**

Example:

```text
3 forks → 2^3 = 8 total processes
```

New child processes created:

```text
8 total - 1 original parent = 7 children
```

---

## 10. Important Fork Counting Rule

Do NOT blindly count the number of `fork()` calls.

Always ask:

> **How many processes reach this particular `fork()`?**

Example:

```text
if (fork() == 0) {
    fork();
}
```

First fork creates:

```text
Parent P1
Child  P2
```

Only the child enters the condition and executes the second fork.

Final count:

```text
P1
P2
P3
```

Total = 3 processes.

---

# 11. `exec()` — Basic Idea

`exec()` does **not** create a new process.

Instead:

> **`exec()` replaces the current process's program image with another program.**

Before:

```text
Process P1
PID = 100
Running Program A
```

After successful `exec()`:

```text
Process P1
PID = 100
Running Program B
```

The process identity/PID generally remains the same.

---

# 12. What Does `exec()` Replace?

Conceptually, old user-space program memory is replaced:

```text
Before exec:

Process P1
├── Old Code
├── Old Data
├── Old Heap
└── Old Stack
```

After:

```text
Process P1
├── New Program Code
├── New Data
├── New Heap
└── New Stack
```

The Program Counter is set to the new executable's entry path rather than the instruction after the successful `exec()`.

---

# 13. Successful `exec()` Does Not Return

Conceptually:

```text
Old Program
   ↓
exec(new_program)
   ↓
Old program image replaced
   ↓
New program starts
```

Therefore code after a **successful** `exec()` in the old program is not executed.

If `exec()` fails, the old program remains and execution can continue with an error.

---

# 14. `fork()` vs `exec()`

| `fork()` | `exec()` |
|---|---|
| Creates a new process | Does not create a new process |
| Parent + child exist | Same process continues |
| Child gets new PID | PID generally unchanged |
| Child initially resembles parent | Program image is replaced |
| Both continue after fork | Successful exec does not return to old program |

Best memory trick:

> **fork() = new process**  
> **exec() = new program in current process**

---

# 15. Why `fork()` + `exec()` Are Commonly Used Together

Suppose a shell wants to run another program.

If the shell directly replaces itself with that program, the shell disappears.

Instead:

```text
Shell
  ↓
fork()
  ↓
┌─────────────┐
│             │
Parent       Child
Shell          ↓
             exec(new program)
               ↓
         Child now runs new program
```

Parent shell remains available.

---

# 16. `exec()` and Resource Cleanup

Successful `exec()` replaces the old user-space address space:

- Old code disappears
- Old heap disappears
- Old stacks disappear
- Old globals disappear

However, not every kernel-managed process resource is automatically discarded in the same way.

Important example:

> Open file descriptors can survive across `exec()` unless configured as close-on-exec.

That is useful for shell redirection and process pipelines.

---

## 17. User-Space Buffers vs Kernel Resources

Suppose an application has:

```text
User-space DB object
       ↓
contains descriptor/socket reference
       ↓
Kernel-managed socket
```

`exec()` destroys the old user-space object because the old heap is replaced.

But an inherited kernel file descriptor/socket can remain open unless close-on-exec behavior applies.

Important:

> `exec()` is not the same as graceful application cleanup.

---

# 18. `wait()` — Parent Waiting for Child

A parent may need to know when its child finishes.

The parent can perform a wait operation.

If child is still active:

```text
Parent calls wait()
      ↓
Parent → WAITING/BLOCKED
```

When child terminates:

```text
Child terminates
      ↓
Parent becomes READY
```

The scheduler later gives parent CPU time.

---

# 19. Why `wait()` Matters

When a child terminates, the OS may preserve limited termination information so the parent can collect it.

Examples:

- Exit status
- Termination reason
- Some accounting information

The parent then **reaps** the child by collecting this termination state.

---

# 20. Zombie Process

A zombie is:

> **A child process that has terminated, but whose parent has not yet collected/reaped its exit status.**

Flow:

```text
Child running
    ↓
Child exits
    ↓
Zombie
    ↓
Parent wait()/reaps child
    ↓
Final process-table entry removed
```

Important:

A zombie:

- Is not actively executing
- Does not receive normal CPU time
- Has released most ordinary execution resources
- Still has a small process-table/termination record

Memory trick:

> **Zombie = dead child not yet reaped**

---

# 21. Orphan Process

An orphan occurs when:

> **The parent terminates while the child is still alive.**

Flow:

```text
Parent P1 → terminates

Child P2 → still running
             ↓
           Orphan
```

Unix-like systems re-parent such a child to an appropriate system process/subreaper.

Traditional textbook explanation:

```text
Orphan → adopted by init/system process
```

Memory trick:

> **Orphan = live child whose original parent died**

---

# 22. Zombie vs Orphan

| Zombie | Orphan |
|---|---|
| Child terminated first | Parent terminated first |
| Parent still alive | Child still alive |
| Child no longer executing | Child may keep executing |
| Exit status not yet reaped | Child gets re-parented |

Quick trick:

```text
Zombie = dead child
Orphan = parentless live child
```

---

# 23. Process Termination

A process can terminate due to:

- Normal completion
- Explicit exit
- Fatal error
- Kill/termination signal
- Unhandled exception
- OS/resource/policy action

After termination, the OS normally reclaims execution resources, but parent-child termination bookkeeping may remain until reaped.

---

# 24. Too Many Zombies

If a long-running parent continuously creates children but never reaps them:

```text
Parent
├── Zombie 1
├── Zombie 2
├── Zombie 3
├── Zombie 4
└── ...
```

process-table entries can accumulate.

Thus well-designed parent processes should reap terminated children.

---

# 25. Connection with Process States

Normal execution:

```text
NEW
 ↓
READY
 ↓
RUNNING
 ↓
WAITING / READY ...
 ↓
TERMINATED
```

Zombie is best thought of as a **special post-termination bookkeeping condition**, not one of the classic five basic scheduling states.

---

# IMPORTANT EXAM POINTS

1. `fork()` creates a child process.
2. `fork()` does not create a new executable file.
3. Parent and child continue after the `fork()` point.
4. Scheduling order after `fork()` is not guaranteed.
5. Parent and child have separate virtual address spaces.
6. Copy-on-Write avoids immediate full physical memory duplication.
7. Parent gets child PID from successful fork; child gets `0`.
8. For `n` unconditional forks reached by every process, total processes = `2^n`.
9. Always count how many processes reach each conditional fork.
10. `exec()` replaces the current program image.
11. `exec()` does not create a new process.
12. PID usually remains the same across `exec()`.
13. Successful `exec()` does not return to old program code.
14. `wait()` allows a parent to wait for/reap child termination.
15. Zombie = terminated child not yet reaped.
16. Orphan = child still alive after original parent terminates.
17. Zombie does not normally consume CPU as an executing process.
18. Zombie process-table entries can accumulate if parent never reaps children.

---

# QUICK REVISION MAP

```text
Parent Process
      ↓
    fork()
      ↓
┌──────────────┐
│              │
Parent        Child
│              │
│            exec()
│              ↓
│         New Program
│              ↓
│            Exit
│              ↓
│        Zombie (if not reaped)
│              ↓
└──── wait() / reap
```

Copy-on-Write:

```text
After fork:

Parent VA ─┐
           ├── same physical frame
Child VA ──┘

On write:
       ↓
private physical copy created
```
