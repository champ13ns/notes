## Process Creation & Termination

### `fork()`

`fork()` current process se **new child process** create karta hai.

```text
Parent
   |
 fork()
   |
 ┌─┴─┐
Parent Child
```

Important points:

- Parent and child are separate processes.
- Child gets a new PID.
- Both continue execution from immediately after `fork()`.
- Parent and child scheduling order is not guaranteed.
- Modern OS generally uses **Copy-on-Write (COW)** instead of immediately copying all physical memory.

Return value:

```text
Parent → child PID (>0)
Child  → 0
Failure → negative
```

For `n` unconditional forks:

```text
Total processes = 2^n
```

only if every process reaches every `fork()`.

---

### `exec()`

`exec()` does **not** create a new process.

It replaces the current process's program image with another program.

```text
Process P1 running Program A
          |
        exec()
          |
Process P1 running Program B
```

PID normally remains the same.

Successful `exec()` does not return to the old program.

Remember:

```text
fork() → new process
exec() → new program in same process
```

---

### `fork()` + `exec()`

Common Unix pattern:

```text
Shell
  |
 fork()
  |
 ┌┴────────┐
Parent    Child
Shell      |
         exec()
           |
       New program
```

This allows shell to stay alive while child runs another command.

---

### `wait()`

Parent uses `wait()` to collect the termination status of a child.

If child is still running:

```text
Parent → WAITING/BLOCKED
```

When child terminates:

```text
Parent → READY
```

and later scheduler gives it CPU.

---

### Zombie Process

Child has terminated, but parent has not yet collected its exit status.

```text
Child exits
    ↓
Zombie
    ↓
Parent wait()
    ↓
Fully removed
```

Zombie:

- does not execute CPU instructions
- has mostly released its normal resources
- still has a small process-table entry

**Zombie = dead child not yet reaped.**

---

### Orphan Process

Parent terminates while child is still alive.

```text
Parent → terminated
Child  → still running
          ↓
        Orphan
```

The child is normally re-parented to an appropriate system process/subreaper.

**Orphan = live child whose original parent died.**

---

### Zombie vs Orphan

```text
Zombie:
Child dies first
Parent still alive

Orphan:
Parent dies first
Child still alive
```

---

### Fork Question Rule

Never blindly count the number of `fork()` statements.

Instead ask:

> How many processes reach this particular `fork()`?

Example:

```c
fork();

if (fork() == 0) {
    printf("A");
}
```

After first fork → 2 processes.

Both execute second fork → 2 new children.

Only those two children get return value `0`.

Therefore:

```text
A → 2 times
```