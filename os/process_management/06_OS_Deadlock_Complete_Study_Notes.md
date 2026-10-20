# Operating Systems — Deadlock: Complete Study Notes

**Level:** IBPS SO IT / IBPS RRB SO IT, with GATE foundation  
**Format:** Concept → example → algorithm → exam traps  
**Revision rule:** `Need` belongs to Banker's Safety Algorithm; `Request` belongs to Deadlock Detection.

---

## 1. Deadlock: definition and intuition

**Deadlock** is a state in which a **set of processes** is blocked indefinitely because every process in that set is waiting for a resource or event that can be provided only by another process in the set.

> **Important correction:** The *entire operating system* need not be stuck. Processes outside the deadlocked set can continue executing.

**Classic example (one instance of each resource):**
- P1 holds R1, requests R2.
- P2 holds R2, requests R1.
- Neither process releases its held resource until it obtains the requested one.
- Both remain blocked: **deadlock**.

**Ordinary waiting vs deadlock vs starvation**

| Phenomenon | Meaning | Example |
|---|---|---|
| Waiting/blocking | Process waits for a resource or event that may arrive normally | Waiting for disk I/O |
| Deadlock | A set of processes mutually blocks further progress | P1 waits for P2; P2 waits for P1 |
| Starvation | A process is indefinitely denied resources/scheduling while others may progress | Low-priority process repeatedly bypassed |

A deadlocked process can be waiting forever, but *indefinite waiting alone* does not prove deadlock.

---

## 2. Coffman's four necessary conditions

| Condition | Meaning | Example |
|---|---|---|
| **Mutual exclusion** | At least one resource is non-shareable | Exclusive printer or mutex |
| **Hold and wait** | Process holds resources while waiting for new ones | P1 holds R1, requests R2 |
| **No preemption** | Resources cannot be forcibly reclaimed safely | Device operation cannot be interrupted |
| **Circular wait** | Circular chain of processes waiting on each other | P1 → P2 → P3 → P1 |

**Theorem for classical resource deadlock:** All four conditions are necessary simultaneously. Breaking *any one* prevents deadlock of this kind. Their mere presence is **not sufficient** to prove that a deadlock currently exists.

### Deadlock Prevention

| Break this condition | How? | Drawback |
|---|---|---|
| Mutual exclusion | Make a resource shareable, where possible | Cannot share all resources |
| Hold and wait | Request all needed resources together, or release held ones before requesting more | Poor utilization, potential starvation |
| No preemption | Reclaim safely preemptible resources and retry | Impossible or unsafe for some resources |
| Circular wait | Fix a total ordering R1 < R2 < R3; acquire in increasing order | Restricts flexibility |

**Exam trap:** Resource ordering prevents **circular wait**, not hold-and-wait.

---

## 3. Resource Allocation Graph (RAG)

A **directed bipartite graph** representing processes and resource types:

- **Circle:** Process (`P1`, `P2`).
- **Rectangle:** Resource type (`R1`, `R2`).
- **Dots inside rectangle:** Number of instances of that resource type.
- **Request edge:** `P1 → R1` (P1 is requesting R1).
- **Assignment edge:** `R1 → P1` (one R1 instance is allocated to P1).

**Arrow direction matters; geometric shape does not.** Diagonal, straight, and curved edges all have the same meaning.

### How to interpret cycles

| Graph state | Conclusion |
|---|---|
| **No cycle** | No deadlock |
| **Cycle; every resource type has one instance** | Deadlock exists (**necessary and sufficient**) |
| **Cycle; some types have multiple instances** | Deadlock **may or may not** exist |

Careful: Resource *types* have instances; it is inaccurate to say “each process has one instance.”

### Example A: cycle exists, but no deadlock

- R1 has **two** instances, R2 has **one**.
- P1 holds one R1, requests R2.
- P2 holds R2, requests R1.
- P3 holds the other R1, requests **nothing**.

All resource instances are initially allocated, and the graph contains the cycle `P1 → R2 → P2 → R1 → P1`.

Yet P3 can finish and release R1; P2 then gets R1, finishes, releases R2; P1 then finishes.

**Possible completion sequence:** `P3 → P2 → P1`. So **no deadlock**.

### Example B: genuine multi-instance deadlock

Change only one detail: **P3 also requests R2**.

- P1 holds R1a, waits for R2.
- P2 holds R2, waits for any instance of R1.
- P3 holds R1b, waits for R2.

Available: R1 = 0; R2 = 0. Each process needs an unavailable resource to finish; nobody can release one first (under the stated assumptions). **All three are deadlocked.**

### Wait-for Graph (WFG)

A graph of **processes only**, with `P1 → P2` meaning P1 waits for a resource held by P2.

For **single-instance resource types**, a WFG cycle indicates deadlock. For multiple-instance resources, simple process-cycle checking is not sufficient.

---

## 4. Four ways to handle deadlock

| Approach | When? | Goal | Main example |
|---|---|---|---|
| **Prevention** | Beforehand | Make one necessary condition impossible | Resource ordering |
| **Avoidance** | On each relevant resource request | Never allocate into an unsafe state | Banker's Algorithm |
| **Detection** | After allocation; periodically/on demand | Discover existing deadlocks | Wait-for Graph / detection matrices |
| **Recovery** | After detection | Break the deadlock | Abort, rollback, resource preemption |

Some systems pragmatically tolerate rare deadlocks and rely on restarts/timeouts; this does **not** guarantee freedom from deadlock.

---

## 5. Safe vs unsafe state

**Safe state:** At least one **safe sequence** exists: an order in which all processes can finish even if they request up to their declared remaining maxima.

**Unsafe state:** No sequence guarantees completion under the maximum-claim assumptions.

- **Safe ≠ currently every process can run.** Often some must wait for others to finish.
- **Unsafe ≠ already deadlocked.** Processes might request less than the declared max and complete.
- **Deadlocked ⇒ unsafe** (in the Banker's resource model).
- **Multiple safe sequences can exist.** Finding one is sufficient.

### Key relationship

`Safe states` and `Unsafe states` do not overlap. `Deadlocked states ⊂ Unsafe states`.

---

## 6. Banker's Algorithm: data structures

Banker's Algorithm is a **deadlock-avoidance** technique. It needs an advance upper bound on each process's resource demand.

For `n` processes and `m` resource types:

| Item | Size | Meaning |
|---|---|---|
| `Available` | m-vector | Free instances of each resource type |
| `Max` | n × m | Maximum possible claim of each process |
| `Allocation` | n × m | Currently allocated instances |
| `Need` | n × m | Remaining maximum need |
| `Work` | m-vector | Temporary available vector during simulation |
| `Finish` | n booleans | Whether process can be completed in simulation |

**Formulas:**

```text
Need[i][j] = Max[i][j] - Allocation[i][j]
Total[j]   = Available[j] + Σ_i Allocation[i][j]
```

**Never confuse** `sum of Max` with actual `Total`: different processes can reuse instances released by earlier processes.

### Safety Algorithm

1. Set `Work = Available`; `Finish[i] = false` for each process.
2. Find unfinished process `Pi` with **`Need[i] <= Work` component-wise**.
3. Pretend Pi gets its remaining need, finishes, and returns resources:
   - `Work = Work + Allocation[i]`
   - `Finish[i] = true`
4. Repeat until all processes finish (**safe**) or no eligible unfinished process remains (**unsafe**).

**Why add `Allocation`, not `Need`?** The process temporarily receives its Need, then returns *all* resources when it finishes. The net gain to currently free resources is exactly its original Allocation.

### Complete worked example (3 resource types)

Total resources: `A=10, B=5, C=7`.

| Process | Allocation (A,B,C) | Max (A,B,C) | Need (A,B,C) |
|---|---|---|---|
| P0 | (0,1,0) | (7,5,3) | (7,4,3) |
| P1 | (2,0,0) | (3,2,2) | (1,2,2) |
| P2 | (3,0,2) | (9,0,2) | (6,0,0) |
| P3 | (2,1,1) | (2,2,2) | (0,1,1) |
| P4 | (0,0,2) | (4,3,3) | (4,3,1) |

Total allocated = `(7,2,5)`; so `Available = (3,3,2)`.

| Chosen process | Work before | Allocation returned | Work after |
|---|---|---|---|
| P1 | (3,3,2) | (2,0,0) | (5,3,2) |
| P3 | (5,3,2) | (2,1,1) | (7,4,3) |
| P4 | (7,4,3) | (0,0,2) | (7,4,5) |
| P0 | (7,4,5) | (0,1,0) | (7,5,5) |
| P2 | (7,5,5) | (3,0,2) | (10,5,7) |

A safe sequence is **`P1 → P3 → P4 → P0 → P2`**. The system is **safe**.

**Component-wise comparison trap:** `Need=(4,0,0)` cannot be satisfied by `Work=(3,3,3)` although the sum of Work is larger than the sum of Need.

---

## 7. Banker's Resource-Request Algorithm

Suppose process `Pi` makes a new request `Request_i`.

1. Check **`Request_i <= Need_i`**. Otherwise, request exceeds declared maximum; invalid.
2. Check **`Request_i <= Available`**. Otherwise, process must wait.
3. **Tentatively** update:

```text
Available'      = Available      - Request_i
Allocation_i'   = Allocation_i   + Request_i
Need_i'         = Need_i         - Request_i
```

4. Run **Safety Algorithm** on tentative state:
   - **Safe:** Commit the allocation.
   - **Unsafe:** Roll back tentative changes; Pi waits.

**Exam trap:** `Request <= Available` alone does **not** justify granting a request.

### Worked example: request that can be granted

Use the five-process example above; initial `Available=(3,3,2)`.

P1 requests `(1,0,2)`, with original Need `(1,2,2)`.

- `Request <= Need`: yes.
- `Request <= Available`: yes.
- Tentative `Available' = (2,3,0)`.
- Updated P1 Allocation `(3,0,2)`; updated P1 Need `(0,2,0)`.

P1 can now finish; **Work = (2,3,0) + (3,0,2) = (5,3,2)**. Then P3, P4, P0, P2 can finish.

Thus a safe sequence exists: **`P1 → P3 → P4 → P0 → P2`**. Grant request.

### Counterexample: available, yet unsafe

One resource type, Total = 10:

| Process | Allocation | Max | Need |
|---|---:|---:|---:|
| P1 | 3 | 9 | 6 |
| P2 | 2 | 4 | 2 |
| P3 | 2 | 7 | 5 |

Available = 3. P1 requests 2: valid and currently available.

Tentative Available = 1; new Needs: P1=4, P2=2, P3=5. None can satisfy its full Need with 1 free instance. **Unsafe; do not grant**, even though currently requested resources exist.

---

## 8. Deadlock Detection Algorithm

**Purpose:** Determine whether there is a deadlock **now**, when future maxima are not necessarily known.

For multiple-instance resources, detection typically uses:

- `Available`: currently free instances.
- `Allocation`: instances held by processes.
- **`Request`: current outstanding requests**, NOT Max or Need.

### Standard matrix-based detection steps

1. `Work = Available`.
2. Initialize `Finish[i] = true` for processes whose Allocation row is all zero; otherwise `false`.
3. Find `i` with `Finish[i] == false` and **`Request[i] <= Work`**.
4. Simulate completion: `Work += Allocation[i]`; set `Finish[i] = true`.
5. Repeat until no such process exists.
6. Processes remaining with `Finish[i] == false` are **deadlocked** (under the model's assumptions).

**Difference from Banker's:**

| Banker's Safety | Deadlock Detection |
|---|---|
| Future maximum remaining `Need` | Current outstanding `Request` |
| Looks for a safe completion guarantee | Checks which processes are currently stuck |
| Avoidance | Detection |

### Worked detection example

`Available=(0,0)`.

| Process | Allocation (A,B) | Request (A,B) |
|---|---|---|
| P1 | (1,0) | (0,1) |
| P2 | (0,1) | (1,0) |
| P3 | (1,0) | (0,0) |

- Initially Work `(0,0)`.
- P3 can finish (`Request=(0,0)`), releases `(1,0)` → Work `(1,0)`.
- P2 can finish, releases `(0,1)` → Work `(1,1)`.
- P1 can finish, releases `(1,0)` → Work `(2,1)`.

**No deadlock**, despite no free resources initially.

**Change:** If P3 instead requests `(0,1)`, no process can finish from `Work=(0,0)`. Under the assumptions, **all three are deadlocked**.

---

## 9. Recovery after detection

### A. Process termination

- **Abort all deadlocked processes:** Fast but potentially loses substantial work.
- **Abort one at a time:** Select victim, terminate, release its resources, and rerun detection until deadlock breaks.

Victim selection may depend on priority, cost to restart, progress already made, resources held, and resources still needed.

### B. Resource preemption

Safely reclaim resources from a victim and reassign them to break the circular dependency.

Three challenges:

1. **Victim selection:** Which process is cheapest to disrupt?
2. **Rollback:** Return a victim to a saved consistent checkpoint if needed.
3. **Starvation:** Avoid repeatedly selecting the same victim.

**Prevention vs recovery:** Both can involve preemption, but prevention stops a deadlock from forming; recovery breaks an already detected deadlock.

**Real-world example:** Two database transactions take row locks in opposite orders; a DBMS may detect deadlock, abort/rollback one transaction, release its locks, and allow the other to continue. Retry aborted transactions appropriately.

---

## 10. Common exam traps

1. Deadlock does **not** require the whole OS to freeze.
2. All four Coffman conditions are **necessary**, but not necessarily sufficient.
3. A RAG cycle proves deadlock **only when every resource type has one instance**.
4. No free resources does **not** automatically imply deadlock.
5. With multiple instances, a cycle can exist without deadlock.
6. In Banker's, **Need = Max − Allocation**.
7. Actual **Total = Available + total allocated**, not sum of Max.
8. Compare vectors **component-wise**, never by summing their components.
9. When a process finishes during a safety/detection simulation, **`Work += Allocation`**.
10. A system can have **multiple valid safe sequences**.
11. **Unsafe does not mean deadlocked**.
12. Resource Request Algorithm allocates **tentatively**, runs safety check, then commits or rolls back.
13. Detection uses **Request**; Banker's uses **Need**.
14. Resource ordering breaks **circular wait**, not mutual exclusion.
15. During recovery, repeated selection of the same victim can cause **starvation**.

---

## 11. Last-minute formula sheet

```text
Need[i]            = Max[i] - Allocation[i]
Total              = Available + column-wise sum(Allocation)
Initial Work       = Available

Banker's safe step:
    if Need[i] <= Work:
        Work += Allocation[i]
        Finish[i] = true

Detection step:
    if Request[i] <= Work:
        Work += Allocation[i]
        Finish[i] = true

Resource request:
    Request_i <= Need_i
    Request_i <= Available
    Available'    = Available - Request_i
    Allocation_i' = Allocation_i + Request_i
    Need_i'       = Need_i - Request_i
    Grant only if new state is safe
```

### What to practice next

- RAG: single vs multiple resource instances.
- Count valid safe sequences (be careful: there may be **more than one**).
- Banker's: compute Need; find one safe sequence.
- Resource-Request: tentative updates and safety proof.
- Detection: identify **exactly which processes** remain unfinished.
- Recovery: choosing termination/preemption/rollback and recognizing starvation.

**Study tip:** Solve these with neat tables showing `Work before`, `chosen process`, and `Work after`. It reduces numerical mistakes.
