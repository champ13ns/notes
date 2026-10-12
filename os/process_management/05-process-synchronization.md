# Operating Systems — Process Synchronization
## Revision Notes for IBPS RRB SO IT Officer / IBPS SO IT Officer

> These notes cover the synchronization concepts studied so far, with examples, exam traps and a self-test. The detailed Deadlocks chapter (RAG, prevention, avoidance, Banker’s algorithm, detection and recovery) will be covered separately.

## 1. Why Synchronization Is Needed

Operating systems run multiple processes and threads concurrently. They may share mutable data or resources: bank balances, counters, queues, database connections, files or memory structures. Without coordination, the result can depend on the timing/interleaving of operations.

### Race condition
A **race condition** occurs when a result depends on the timing of concurrent operations and an interleaving can produce an incorrect or unintended result.

Example: `balance = 100`; T1 adds 50 and T2 subtracts 20. Expected result: `130`.

A simplified update can involve three steps:

```text
read value → calculate new value → write value
```

Possible interleaving:

```text
T1 reads 100
T2 reads 100
T1 calculates 150
T2 calculates 80
T1 writes 150
T2 writes 80
```

Final balance becomes **80**: T1’s update was lost. This is a **lost update**.

Key points:
- A race can occur on a **single CPU core** due to interleaving; true parallel execution is not required.
- The risky case usually involves shared mutable state and at least one write.
- Multiple reads of stable data generally do not need mutual exclusion merely to prevent a write race.
- A race condition is not automatically a deadlock.

## 2. Critical Section and Mutual Exclusion

A **critical section (CS)** is code that accesses/modifies shared state and needs coordination.

```text
Entry section
     ↓
Critical section  ← access/update shared state
     ↓
Exit section
     ↓
Remainder section
```

Example:

```text
lock(M)
balance = balance + 50
unlock(M)
```

**Mutual exclusion** means at most one thread at a time may enter a particular protected exclusive region.

### Three classic critical-section requirements

| Requirement | Meaning |
|---|---|
| **Mutual exclusion** | Two processes/threads cannot be in the same protected critical section at the same time. |
| **Progress** | If nobody is in the critical section and some want to enter, selection of the next entrant should not be postponed indefinitely by processes that are not trying to enter. |
| **Bounded waiting** | After a process requests entry, there is a bound on how many times others can enter before it is granted access. This prevents indefinite postponement. |

**Exam trap:** Mutual exclusion alone does not guarantee fairness. A safe design can still starve a thread.

## 3. Mutexes

A **mutex** (mutual-exclusion lock) protects shared state so only one thread at a time holds the lock.

```text
T1: acquire M ── critical section ── release M
T2:             waits for M ─────── acquire M ── critical section
```

With a **blocking mutex**, a thread that tries to acquire a busy mutex may transition from `RUNNING` to `BLOCKED/WAITING`.

### Correct use
1. Use the **same mutex** for every conflicting access to the same shared state. If T1 protects `x` with `M1` but T2 uses `M2`, the locks do not coordinate.
2. Keep the critical section as small as practical while protecting the complete operation that must remain consistent.
3. Lock before reading if that read must be synchronized with updates.
4. Release locks on appropriate exit paths.
5. Avoid acquiring multiple locks in inconsistent orders; this can cause deadlock.

### Check-then-act race
This is unsafe if multiple threads execute it concurrently:

```text
if (balance >= 100) {
    balance = balance - 100
}
```

The **check and update must be protected together**. Two threads might both observe `balance = 150`, both pass the check, and both attempt a withdrawal. Locking only the subtraction is insufficient.

### Stale local copies
A lock does not magically update a value already copied into a local variable.

```text
T1: localA = sharedX       // localA becomes 50
T2: lock(M); sharedX=100; unlock(M)
T1: lock(M); use localA; unlock(M)
```

`localA` can still be **50**. If fresh data is required, read `sharedX` inside the synchronized region.

**Exam trap:** Acquiring a lock later does not retroactively refresh earlier reads or redo calculations.

## 4. Spinlocks (Brief Exam Note)

A **spinlock** makes a waiting thread repeatedly check whether the lock is available instead of sleeping immediately.

- **Potential advantage:** avoids blocking/wakeup overhead when the wait is extremely short.
- **Potential disadvantage:** wastes CPU while spinning, especially if the lock is held for a long time.
- Often suitable only for very short critical sections, depending on OS and context.

A spinlock and a blocking mutex can both enforce exclusive access, but their waiting behavior differs.

## 5. Semaphores

A **semaphore** is a synchronization primitive with a permit count. It can control access to a number of resources or coordinate events.

- **wait / P / down:** request/acquire a permit; if none is available, wait.
- **signal / V / up:** release a permit, increasing availability and potentially waking a waiter.

### Binary semaphore
A binary semaphore has two states, commonly `0` and `1`. It can be used for signaling or exclusion, but is **not automatically identical to a mutex**. A mutex usually has ownership semantics (the thread that locks it is expected to unlock it); a semaphore generally does not enforce the same rule.

### Counting semaphore
A counting semaphore represents multiple available units of a resource.

Example: a pool has 3 database connections.

```text
Initial S = 3
T1 acquires → S = 2
T2 acquires → S = 1
T3 acquires → S = 0
T4 requests → waits until a permit is returned
```

### Correct resource accounting
If all 3 connections are in use and an unrelated thread signals the semaphore without returning a connection, the semaphore may advertise a resource that does not actually exist.

**Exam trap:** No mutex-style ownership rule does **not** mean a thread may signal a resource semaphore arbitrarily. Permit counts must match real resource availability.

## 6. Mutex vs Semaphore

| Feature | Mutex | Semaphore |
|---|---|---|
| Main idea | Exclusive ownership of protected region/resource | Permit count for exclusion, signaling or resource capacity |
| Typical use | Protect shared data or an invariant | Manage N resources; coordinate producer/consumer |
| Count | Usually locked/unlocked | Binary or counting |
| Ownership | Usually has ownership semantics | Generally no mutex-style ownership |
| Concurrency | One holder at a time | Counting semaphore can allow up to N permit holders |
| Common mistake | Different mutexes used for same shared data | Signaling without a matching event/resource release |

**Quick rule:** Use a mutex to protect shared state exclusively; use a counting semaphore to manage a limited number of permits/resources. The correct choice depends on the design.


## 7. Bounded Buffer and Producer–Consumer

A **buffer** is temporary storage between a producer and a consumer. A **bounded buffer** has a fixed capacity.

- Producer inserts items.
- Consumer removes items.
- Producer must not insert when full.
- Consumer must not remove when empty.
- Shared queue contents, indexes and metadata need safe updates.

For capacity `N`, the classic semaphore setup is:

```text
empty = N    // free slots
full  = 0    // available items
mutex = 1    // protects buffer structure
```

Conceptual producer sequence:

```text
wait(empty)       // reserve a free slot first
wait(mutex)       // enter buffer critical section
insert item
signal(mutex)
signal(full)      // announce an available item
```

Conceptual consumer sequence:

```text
wait(full)        // reserve an available item first
wait(mutex)
remove item
signal(mutex)
signal(empty)     // announce a free slot
```

### Why the order matters

Incorrect producer sequence:

```text
wait(mutex)
wait(empty)
```

If the buffer is full (`empty = 0`), the producer can block while holding `mutex`. The consumer needs that same mutex to remove an item and create an empty slot.

```text
Producer holds mutex → waits for empty slot
Consumer needs mutex → cannot remove an item
```

A deadlock can result.

**Key lesson:** Wait for resource availability before holding the lock needed by another participant to make that resource available.

### Buffer count example

Capacity = 4; initially `empty = 4`, `full = 0`.

- Producer inserts 3 items → `empty = 1`, `full = 3`.
- Consumer removes 1 item → `empty = 2`, `full = 2`.

For a correctly maintained bounded buffer:

```text
empty + full = capacity
```

Use this as a consistency check in numerical questions.

## 8. Backpressure — Basic Idea

**Backpressure** is how a system slows, blocks, queues, rejects, drops or otherwise regulates incoming work when downstream processing cannot keep up.

Example:
- Producer creates 500 messages/second.
- Consumer processes 100 messages/second.
- Buffer capacity = 400 messages.

The buffer temporarily absorbs the rate difference, but cannot solve a sustained mismatch. If arrivals keep exceeding processing, it eventually fills.

Possible policies:
- Block or slow the producer
- Throttle incoming requests
- Queue work upstream
- Reject or drop some work
- Spill to durable storage, if supported

**Exam trap:** A larger buffer delays saturation; it does not fix a permanent throughput mismatch.

## 9. Readers–Writers Problem

A shared object is accessed by readers and writers.

- Multiple readers may read simultaneously.
- A writer needs exclusive access.
- Readers must not overlap with a writer.
- Two writers must not write simultaneously.

| Combination | Allowed? |
|---|---|
| Reader + Reader | **Yes** |
| Reader + Writer | **No** |
| Writer + Writer | **No** |

A **reader–writer lock** offers:
- **Shared/read lock:** multiple readers can hold it together.
- **Exclusive/write lock:** one writer holds it alone.

### Fairness policies
- **Reader preference:** readers may keep entering while a writer waits; a writer can starve.
- **Writer preference:** when a writer is waiting, new readers may be held back; readers may experience long waits.
- **Fair policy:** attempts to prevent indefinite postponement of either group.

**Exam trap:** “Many readers OR one writer” is the rule. “One writer plus readers” is not allowed.

This is conceptually similar to database shared and exclusive locks.

## 10. Lock Granularity

**Lock granularity** means how much state a lock protects.

### Coarse-grained locking
One lock protects a large structure or operation.
- Simpler to reason about.
- May reduce concurrency because unrelated operations wait for each other.

### Fine-grained locking
Separate locks protect independent pieces of state.
- Can improve concurrency for independent operations.
- Increases implementation complexity.
- Lock ordering becomes important.
- More locks can create more opportunities for deadlock.

If `x`, `y` and `z` are independent, separate locks can let a thread updating `x` proceed while another updates `y`.

But if an operation must transfer value atomically between `x` and `y`:

```text
x = x - 10
y = y + 10
```

the **whole logical transfer** must be protected as one consistency boundary. Protecting each assignment separately can expose an inconsistent intermediate state.

**Exam trap:** Fine-grained locking is not always better. Preserve invariants and avoid inconsistent lock ordering.

## 11. Dining Philosophers Problem

Five philosophers sit around a table with one fork between each neighboring pair. Each alternates between thinking and eating. To eat, a philosopher needs both adjacent forks.

```text
             P0
         F4      F0
       P4          P1
       F3          F1
          P3--F2--P2
```

This is a schematic: each fork is shared by two neighboring philosophers.

### Deadlock scenario
1. Every philosopher picks up the left fork.
2. Every philosopher waits for the right fork.
3. Each right fork is held by a neighbor.
4. Everyone waits indefinitely.

```text
P0 holds F0, waits for F4
P1 holds F1, waits for F0
P2 holds F2, waits for F1
P3 holds F3, waits for F2
P4 holds F4, waits for F3
```

Each philosopher holds one resource and waits for another held by someone else. The dependencies form a cycle.

### Common solutions

**1. Global lock ordering**

Number the forks and require every philosopher to acquire the lower-numbered fork first, then the higher-numbered fork. Consistent ordering breaks the circular-wait pattern.

**2. Limit contenders**

Use a semaphore to allow at most **four of the five** philosophers to attempt to acquire forks at once. This prevents the classic all-five-hold-one-fork scenario, assuming the acquire/release protocol is implemented correctly.

**3. Asymmetric acquisition**

Odd-numbered philosophers acquire left then right; even-numbered philosophers acquire right then left. This breaks the symmetric all-left-first pattern, though the complete implementation still needs correctness and fairness analysis.

**4. Waiter/arbitrator**

A coordinator grants permission to acquire forks only when the requested combination is safe.

**5. One global mutex**

A single mutex around the entire acquisition/eating sequence can prevent deadlock, but it unnecessarily serializes philosophers and reduces concurrency.

### What the problem teaches
- Individual locks may be correct, yet acquiring multiple locks can still deadlock.
- Lock acquisition order matters.
- Deadlock freedom and starvation freedom are different properties.
- Non-neighboring philosophers can eat simultaneously if they do not share forks.

## 12. Deadlock vs Starvation vs Livelock

| Term | What happens? | Example |
|---|---|---|
| **Deadlock** | Participants are stuck waiting; none can make the required progress. | Each philosopher holds one fork and waits for another held by a neighbor. |
| **Starvation** | The system continues to make progress, but one participant may wait indefinitely. | A writer keeps waiting because new readers repeatedly get priority. |
| **Livelock** | Participants remain active and respond to each other, but useful progress does not happen. | Threads repeatedly pick up/release/retry resources so nobody eats. |

**Memory aid**
- Deadlock = stuck.
- Starvation = left waiting while others progress.
- Livelock = active, but not progressing.

These terms are related but not interchangeable. Detailed deadlock theory is a separate chapter.

## 13. Common Exam Traps

1. **A race can occur on one core.** Interleaving is enough.
2. **`x = x + 1` is not necessarily atomic.** It can involve read, calculate and write.
3. **Protect check and action together.** Locking only the final update may be insufficient.
4. **Use the same synchronization discipline for all conflicting accesses.** Two unrelated mutexes do not protect a shared variable from each other.
5. **Local copies can be stale.** A later lock does not refresh a value already copied into a local variable.
6. **Mutex and semaphore are not synonyms.** Ownership and permit-count semantics differ.
7. **Semaphore signaling must match resource/event accounting.** No ownership rule does not mean arbitrary signaling is safe.
8. **`empty` counts free slots; `full` counts available items.**
9. **Wait for `empty`/`full` before locking the buffer mutex.** Otherwise a thread may wait while holding the lock the other side needs.
10. **Reader–reader is allowed; reader–writer and writer–writer are not.**
11. **Reader preference can starve writers; writer preference can delay readers.**
12. **Fine-grained locks can improve concurrency but add complexity and deadlock risk.**
13. **If an invariant spans multiple variables, protect the full logical operation.**
14. **Deadlock, starvation and livelock are different.**
15. **A global lock may be safe but unnecessarily reduce concurrency.**

## 14. Quick Revision Sheet

- **Race condition:** result depends on concurrent timing/interleaving.
- **Critical section:** code accessing shared state that needs protection.
- **Mutual exclusion:** at most one participant in the protected exclusive region.
- **Progress:** entry should not be postponed indefinitely by non-contenders.
- **Bounded waiting:** a requester cannot be bypassed indefinitely.
- **Mutex:** exclusive lock, typically with ownership.
- **Spinlock:** repeatedly checks availability; can waste CPU.
- **Semaphore:** wait/acquire a permit; signal/release a permit.
- **Binary semaphore:** two-state permit; not automatically a mutex.
- **Counting semaphore:** tracks N available resource permits.
- **Producer–consumer:** `empty = free slots`, `full = items`, `mutex = buffer protection`.
- **Backpressure:** regulate input when downstream cannot keep up.
- **Readers–writers:** many readers OR one writer.
- **Fine-grained locking:** more concurrency potential, more complexity.
- **Dining philosophers:** multiple-resource acquisition can create circular waiting.
- **Deadlock:** stuck waiting; **starvation:** one waits indefinitely while others progress; **livelock:** active but no useful progress.

## 15. Self-Test — Try Before Checking the Key

### Q1
Two threads both read `counter = 7`, increment their local values, and write back `8`. What is the main issue?

A. Starvation  
B. Lost update due to a race condition  
C. Semaphore overflow  
D. Page fault

### Q2
A program checks `stock > 0` and then decrements stock. Which is safest if multiple threads may buy the last item?

A. Lock only the decrement  
B. Lock only the check  
C. Protect check and decrement as one logical operation  
D. Use two unrelated mutexes

### Q3
A resource pool contains 4 identical connections. What initial count best represents the available resources in a counting semaphore?

A. 0  
B. 1  
C. 4  
D. Unlimited

### Q4
A buffer has capacity 6, with `empty = 2` and `full = 4`. A consumer removes two items. What are the new values?

A. `empty = 0, full = 6`  
B. `empty = 4, full = 2`  
C. `empty = 2, full = 2`  
D. `empty = 6, full = 0`

### Q5
Which reader–writer combination is normally permitted?

A. One writer and two readers simultaneously  
B. Two writers simultaneously  
C. Multiple readers simultaneously  
D. A writer and a reader simultaneously

### Q6
Why can fine-grained locking be useful?

A. It guarantees deadlock can never happen  
B. It may let operations on independent state proceed concurrently  
C. It removes the need to protect shared data  
D. It always uses less CPU

### Q7
All five philosophers hold one fork and wait for the other. Which concept best explains the circular dependency?

A. Paging  
B. Deadlock  
C. Thrashing  
D. Backpressure

### Q8
A writer waits indefinitely while readers keep getting admitted under a reader-preference policy. This is:

A. Livelock  
B. Starvation  
C. Mutual exclusion  
D. Context switching

### Q9
Threads repeatedly react to each other by releasing and retrying locks, but none makes useful progress. This is:

A. Livelock  
B. Deadlock  
C. Bounded waiting  
D. A page fault

### Q10
A producer waits on `empty` while holding the buffer mutex; the consumer needs that mutex to remove an item. What can happen?

A. The buffer automatically expands  
B. Deadlock  
C. The semaphore becomes a mutex  
D. The consumer bypasses the lock safely

### Answer key
1. **B** — both writes can overwrite one another's increment.
2. **C** — check and decrement must be one protected operation.
3. **C** — four available connections means four permits.
4. **B** — two slots become free and two items are consumed.
5. **C** — multiple readers can coexist.
6. **B** — independent resources can be protected separately to allow concurrency.
7. **B** — each participant waits for a resource held by another in a cycle.
8. **B** — others progress while the writer may wait indefinitely.
9. **A** — activity continues without useful progress.
10. **B** — the producer blocks while holding the mutex needed by the consumer.
