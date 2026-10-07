# Operating Systems — Thrashing, Working Set Model & Page Fault Frequency (PFF)

> **Scope:** Only Thrashing, Working Set Model, and Page Fault Frequency (PFF)  
> **Goal:** Conceptual understanding + quick RRB SO IT / banking IT exam revision

---

# 1. Thrashing

## What is Thrashing?

Thrashing is a condition in virtual memory where the system/process spends **too much time servicing page faults and moving pages between RAM and secondary storage**, instead of executing useful instructions.

```text
Process executes
      ↓
Page Fault
      ↓
Required page loaded
      ↓
Another required page missing
      ↓
Page Fault
      ↓
Again Page Fault
      ↓
...
```

If this happens continuously:

```text
Page Fault Rate ↑↑
Disk I/O ↑↑
Useful CPU Work ↓
Performance ↓↓
```

This condition is called **Thrashing**.

---

# 2. Why Thrashing Happens

Suppose a process actively uses:

```text
P1 P2 P3 P4 P5
```

but has only:

```text
3 physical frames
```

Reference pattern:

```text
P1 → P2 → P3 → P4 → P1 → P5 → P2 ...
```

Because only 3 pages can remain resident, actively needed pages keep evicting one another.

So:

> **Too few frames for the current locality can produce a very high page-fault rate.**

---

# 3. Symptoms of Thrashing

Typical symptoms:

```text
Very High Page Fault Rate
High Disk / Page I/O
Low CPU Utilization
Poor Throughput
Slow System Performance
```

Important distinction:

```text
Occasional Page Fault
→ normal

Continuous / excessive Page Faults
→ possible Thrashing
```

---

# 4. Thrashing and Degree of Multiprogramming

Suppose many processes are waiting for page I/O:

```text
Processes waiting
      ↓
CPU gets less useful work
      ↓
CPU utilization decreases
```

OS may think:

```text
CPU is underutilized
→ admit more processes
```

But then:

```text
More Processes
      ↓
Fewer Frames per Process
      ↓
More Page Faults
      ↓
More Disk I/O
      ↓
More Waiting
      ↓
Even Lower CPU Utilization
```

This creates a vicious cycle.

---

# 5. How Thrashing Can Be Controlled

Two important approaches:

```text
1. Working Set Model
2. Page Fault Frequency (PFF)
```

Both try to ensure that processes get enough frames.

---

# 6. Working Set Model

## Core Idea

At any moment, a process usually uses only a subset of all its virtual pages.

These currently active pages form the:

> **Working Set**

Example:

```text
Current active pages:
P2, P3, P7, P8
```

Then:

```text
Working Set = {P2, P3, P7, P8}
Working Set Size = 4
```

The process should ideally have enough frames to keep these active pages resident.

---

# 7. Working Set Window — Δ

The recent reference window is represented by:

```text
Δ
```

Example:

```text
Reference String:
1 2 3 2 4 2 1 5
```

If:

```text
Δ = last 5 references
```

then:

```text
2 4 2 1 5
```

Distinct pages:

```text
{1, 2, 4, 5}
```

Therefore:

```text
Working Set = {1,2,4,5}
Working Set Size = 4
```

Repeated references count only once.

---

# 8. Working Set Size (WSS)

For process `Pi`:

```text
WSSi
=
number of distinct pages referenced
during the recent window Δ
```

Example:

```text
References:
2 4 2 1 5
```

Then:

```text
WSS = 4
```

---

# 9. Why Working Set Helps

Suppose:

```text
Working Set = {P2,P3,P7,P8}
WSS = 4
```

If process gets around 4 useful frames, active pages can remain resident.

If it gets only 2 frames:

```text
P2 loaded
P3 loaded
P7 needed → replacement
P8 needed → replacement
P2 needed again → fault
...
```

So conceptually:

```text
Frames >= Working Set Size
→ healthier execution

Frames << Working Set Size
→ high page-fault risk
→ possible thrashing
```

---

# 10. Working Set Changes Over Time

Working Set is **not static**.

Programs execute in phases.

Example:

```text
Phase 1:
{1,2,3,4}

Phase 2:
{8,9,10}

Phase 3:
{15,16}
```

So:

```text
Time →

{1,2,3,4}
      ↓
   {8,9,10}
          ↓
       {15,16}
```

This happens because of locality of reference.

---

# 11. Working Set and Locality

## Temporal Locality

Recently used pages are likely to be reused soon.

## Spatial Locality

Nearby memory locations are likely to be accessed soon.

Working Set attempts to capture the current locality.

---

# 12. Choosing Δ

## Δ Too Small

May miss pages that are genuinely part of current locality.

```text
Actual locality:
{1,2,3,4}

Very small Δ:
{1,2}
```

This can underestimate memory need.

## Δ Too Large

May include old pages that are no longer active.

```text
Old locality:
{1,2,3}

Current locality:
{8,9}

Large Δ:
{1,2,3,8,9}
```

So:

```text
Δ too small → underestimate
Δ too large → overestimate
```

---

# 13. Total Working Set Demand

For multiple processes:

```text
P1 WSS = 4
P2 WSS = 3
P3 WSS = 5
```

Total demand:

```text
D = Σ WSSi
```

Therefore:

```text
D = 4 + 3 + 5
  = 12 frames
```

If only 10 frames are available:

```text
D > Available Frames
```

Thrashing risk increases.

---

# 14. Working Set Condition

Let:

```text
D = Σ WSSi
m = total available physical frames
```

If:

```text
D <= m
```

there are enough frames, in principle, for the current working sets.

If:

```text
D > m
```

there are not enough frames.

The OS may reduce the degree of multiprogramming.

---

# 15. Working Set vs LRU

## LRU

Question:

> Which page should I evict?

LRU is a:

```text
Page Replacement Algorithm
```

## Working Set

Question:

> Which pages does this process actively need right now?

Working Set is a:

```text
Locality / memory-allocation / thrashing-control model
```

So:

```text
LRU → chooses victim page
Working Set → estimates active page requirement
```

---

# 16. Page Fault Frequency (PFF)

PFF stands for:

> **Page Fault Frequency**

Instead of explicitly finding a working set, PFF monitors:

```text
How frequently is this process generating page faults?
```

It uses page-fault rate as feedback for frame allocation.

---

# 17. PFF Thresholds

Conceptually:

```text
Upper Page-Fault Threshold
Lower Page-Fault Threshold
```

Three cases:

```text
PFF > Upper Limit
PFF < Lower Limit
Lower < PFF < Upper
```

---

# 18. PFF Above Upper Threshold

If:

```text
PFF > Upper Limit
```

the process is generating too many page faults.

Likely meaning:

```text
Too few frames
```

So OS should try to:

```text
Allocate more frames
```

---

# 19. PFF Below Lower Threshold

If:

```text
PFF < Lower Limit
```

page faults are very low.

The process may have more frames than needed.

So OS can:

```text
Reclaim some frames
```

and use them elsewhere.

---

# 20. PFF Within Acceptable Range

If:

```text
Lower Limit < PFF < Upper Limit
```

then:

```text
Current allocation is acceptable
```

No major change is required.

---

# 21. PFF Control Rule

Easy rule:

```text
PFF too high
→ Give MORE frames
```

```text
PFF too low
→ Take SOME frames away
```

```text
PFF within range
→ Keep allocation roughly unchanged
```

---

# 22. What if PFF Is High but No Free Frames Exist?

If a process needs more frames but none are free, the OS may need to:

```text
Reduce degree of multiprogramming
```

For example, temporarily suspend another process.

This frees frames for the remaining processes.

---

# 23. Working Set vs PFF

| Working Set Model | PFF |
|---|---|
| Tracks recently used pages | Tracks page-fault rate |
| Estimates current locality | Observes memory pressure indirectly |
| Uses window Δ | Uses upper/lower thresholds |
| Finds how many pages are active | Adjusts frames from fault behavior |
| More direct locality model | Feedback/control mechanism |

Easy memory trick:

```text
Working Set:
"What pages am I actively using?"
```

```text
PFF:
"Am I page-faulting too often?"
```

---

# 24. Complete Connection

```text
Too Few Frames
      ↓
Active Pages Cannot Stay in RAM
      ↓
More Replacements
      ↓
More Page Faults
      ↓
More Disk I/O
      ↓
Less Useful CPU Work
      ↓
THRASHING
```

Control:

```text
Thrashing
   ↓
 ┌──────────────────────┐
 │                      │
Working Set            PFF
 │                      │
Find active           Monitor
locality              fault rate
 │                      │
Give enough           Dynamically
frames                adjust frames
```

---

# 25. Important Exam Facts

### Thrashing

```text
Excessive paging/page faults
→ poor CPU utilization and performance
```

### Working Set

```text
Set of distinct pages referenced
within recent window Δ
```

### Working Set Size

```text
Number of distinct pages in working set
```

### Total Demand

```text
D = Σ WSSi
```

If:

```text
D > available frames
```

thrashing risk rises.

### PFF

```text
High PFF → allocate more frames
Low PFF  → reclaim some frames
```

---

# 26. Common Exam Traps

## Trap 1

Thrashing means one page fault.

Wrong.

```text
Thrashing = excessive repeated page faults
```

## Trap 2

Working Set counts every page reference.

Wrong.

It counts:

```text
distinct pages
```

## Trap 3

Working Set is constant.

Wrong.

It changes with the process's locality.

## Trap 4

High PFF means remove frames.

Wrong.

```text
High PFF → likely needs more frames
```

## Trap 5

Low PFF means allocate more frames.

Wrong.

```text
Low PFF → some frames may be reclaimable
```

---

# 27. Fast Revision Tree

```text
THRASHING
│
├── Cause
│   ├── Too few frames
│   ├── Too many active processes
│   └── High page-fault rate
│
├── Effects
│   ├── Disk I/O ↑
│   ├── CPU utilization ↓
│   └── Performance ↓
│
├── Working Set Model
│   ├── Recent window Δ
│   ├── Distinct active pages
│   ├── WSS
│   ├── D = Σ WSS
│   └── D > frames → thrashing risk
│
└── PFF
    ├── Monitor page-fault rate
    ├── Above upper limit → more frames
    ├── Below lower limit → reclaim frames
    └── Within range → keep allocation
```

---

# 28. One-Line Revision

```text
Thrashing
→ too many page faults

Working Set
→ pages actively needed right now

PFF
→ adjust frames based on page-fault rate
```
