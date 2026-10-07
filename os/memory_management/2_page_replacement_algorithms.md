# Operating Systems — Page Replacement Algorithms Notes

> **Goal:** Strong conceptual understanding + RRB SO IT / banking IT exam revision  
> **Covered:** FIFO, Belady's Anomaly, Optimal, LRU, Second Chance, Clock  
> **Context:** Used when a page fault occurs and RAM has no free frame.

---

# 1. Why Page Replacement Is Needed

Suppose CPU accesses a virtual page that is not currently in RAM.

```text
CPU accesses Page X
        ↓
TLB / Page Table lookup
        ↓
Present = 0
        ↓
Page Fault
```

Now OS needs to load Page X into RAM.

Two cases:

```text
Free frame available?
       / \
     Yes  No
      |    |
 Load X   Need Page Replacement
```

If RAM is full, OS must choose a **victim page** to remove.

That decision is made by a:

> **Page Replacement Algorithm**

---

# 2. Important Terminology

## Page Fault

Requested page is not currently resident in RAM.

## Page Hit

Requested page is already present in one of the frames.

## Victim Page

The page chosen for eviction.

## Frame

Fixed-size block of physical RAM.

Important:

> We remove/evict a **page** and reuse its **frame**.

Do not say "remove the oldest frame."

---

# 3. Dirty Bit During Replacement

Before evicting a victim page, OS may check its dirty bit.

```text
Dirty = 0
→ Page not modified
→ Can usually discard directly
```

```text
Dirty = 1
→ Page modified in RAM
→ May need to write it back before eviction
```

---

# 4. FIFO — First In First Out

## Rule

> Remove the page that entered RAM earliest.

FIFO tracks **arrival order**, not recent usage.

Example:

```text
Frames = 3
Reference String:
1 2 3 1 4
```

Flow:

```text
1 → [1,-,-] PF
2 → [1,2,-] PF
3 → [1,2,3] PF
1 → HIT
```

Important:

```text
FIFO hit does NOT refresh order.
```

Oldest order is still:

```text
1 → 2 → 3
```

Now:

```text
4 → Page Fault
```

FIFO removes:

```text
1
```

Result:

```text
[4,2,3]
```

---

# 5. Belady's Anomaly

Normally:

```text
More frames
→ fewer page faults
```

But in FIFO sometimes:

```text
More frames
→ MORE page faults
```

This is called:

> **Belady's Anomaly**

Classic example:

```text
Reference String:
1 2 3 4 1 2 5 1 2 3 4 5
```

Using FIFO:

```text
3 Frames → 9 Page Faults
4 Frames → 10 Page Faults
```

Important exam fact:

```text
FIFO → Belady's Anomaly possible
LRU  → No
Optimal → No
```

---

# 6. Optimal Page Replacement

## Rule

> Remove the page whose next use is farthest in the future.

If a page is never used again, evict it.

Example:

```text
Frames = [1,2,3]

Future:
1 2 5 1 2 3 4 5
```

Next use:

```text
1 → soon
2 → soon
3 → much later
```

So Optimal removes:

```text
3
```

---

# 7. Why Optimal Is "Optimal"

For a given reference string and number of frames, Optimal gives the:

> **minimum possible number of page faults**

It is a theoretical benchmark.

Why not practical?

```text
Real OS does not know future references exactly.
```

---

# 8. LRU — Least Recently Used

## Rule

> Remove the page that has not been used for the longest time in the past.

Optimal looks at:

```text
Future
```

LRU looks at:

```text
Past
```

On every hit or load, the page becomes most recently used.

Example:

```text
Frames = [1,2,3]
Reference = 1
```

Recency becomes:

```text
Least Recent → 2 → 3 → 1 ← Most Recent
```

If page `4` arrives:

```text
remove 2
```

---

# 9. FIFO vs LRU

```text
FIFO
→ Hit does NOT change replacement order
```

```text
LRU
→ Hit DOES update recency
```

Example:

```text
Reference History:
1 2 3 1
```

Current frames:

```text
[1,2,3]
```

FIFO oldest:

```text
1
```

LRU least recently used:

```text
2
```

---

# 10. LRU vs LFU

Do not confuse them.

```text
LRU
→ Last kab use hui?
```

```text
LFU
→ Kitni baar use hui?
```

A page can have been used many times in the past and still become LRU if it has not been accessed recently.

---

# 11. Easy LRU Solving Trick

Maintain:

```text
Least Recent ----------------→ Most Recent
```

If page fault occurs and frames are full:

```text
remove leftmost page
```

If a page is hit:

```text
move it to most-recent end
```

---

# 12. Second Chance Algorithm

Second Chance improves FIFO.

FIFO says:

```text
Oldest page → evict
```

Second Chance says:

```text
Oldest page
   ↓
Was it recently referenced?
```

It uses the:

> **Reference / Accessed Bit**

Replacement rule:

```text
R = 0
→ evict
```

```text
R = 1
→ give second chance
→ set R = 0
→ inspect next page
```

---

# 13. Second Chance Example

FIFO order:

```text
Oldest → Page 1 → Page 2 → Page 3
```

Reference bits:

```text
Page 1 → R=1
Page 2 → R=0
Page 3 → R=1
```

New page `4` arrives.

Check Page 1:

```text
R=1
→ give second chance
→ set R=0
```

Check Page 2:

```text
R=0
→ evict Page 2
```

Load Page 4 into that frame.

---

# 14. Clock Algorithm

Clock is a common efficient implementation of Second Chance.

Frames are treated circularly and a pointer acts like a clock hand.

Replacement:

```text
If R = 0
→ evict current page

If R = 1
→ set R = 0
→ move pointer forward
```

Continue until an `R=0` page is found.

Example:

```text
Page 1 → R=1
Page 2 → R=1
Page 3 → R=0
```

Pointer starts at Page 1:

```text
Page 1: R=1 → set 0 → move
Page 2: R=1 → set 0 → move
Page 3: R=0 → evict
```

---

# 15. Reference Bit vs Dirty Bit

Do not confuse them.

```text
Reference Bit
→ Was page accessed?
```

```text
Dirty Bit
→ Was page modified?
```

Typical cases:

```text
Read:
Reference = 1
Dirty = 0
```

```text
Write:
Reference = 1
Dirty = 1
```

---

# 16. Algorithm Comparison

| Algorithm | Victim Selection | Future Needed? | Hit Changes Order? | Belady's Anomaly? |
|---|---|---:|---:|---:|
| FIFO | Oldest loaded page | No | No | Possible |
| Optimal | Farthest next use | Yes | N/A | No |
| LRU | Least recently used | No | Yes | No |
| Second Chance | Oldest with `R=0` | No | Via reference bit | FIFO improvement |
| Clock | Circular Second Chance | No | Via reference bit | FIFO improvement |

---

# 17. Quick Mental Model

```text
FIFO
→ Who came first?
```

```text
Optimal
→ Who will be needed last?
```

```text
LRU
→ Who was used longest ago?
```

```text
Second Chance
→ Oldest page, but was it recently used?
```

```text
Clock
→ Second Chance in circular form
```

---

# 18. Full Runtime Flow

```text
CPU accesses Page X
        ↓
Page not resident
        ↓
PAGE FAULT
        ↓
OS handles fault
        ↓
Free frame?
    /       \
  Yes        No
   |          |
Load X     Run Page Replacement
              ↓
         Select Victim
              ↓
          Dirty = 1?
           /      \
         Yes       No
          |         |
      Write Back    |
           \        /
             ↓
          Free Frame
             ↓
         Load Page X
             ↓
       Update Page Table
             ↓
       Resume Process
```

---

# 19. Common Exam Traps

## Trap 1
Initial filling of empty frames also causes page faults.

## Trap 2
FIFO hit does not refresh page age/order.

## Trap 3
LRU does not count frequency.

```text
LRU → recent use
LFU → usage count
```

## Trap 4
Optimal is not exactly implementable in a real online OS because it requires future knowledge.

## Trap 5
Belady's anomaly is classically associated with FIFO.

## Trap 6
Second Chance primarily uses the reference/accessed bit, not the dirty bit.

---

# 20. RRB SO IT Priority

## Must Know

```text
FIFO
Optimal
LRU
Page Fault
Belady's Anomaly
```

## Know Conceptually

```text
Second Chance
Clock
Reference Bit
Dirty Bit
```

Be comfortable solving:

```text
Reference String + Number of Frames
→ Page Faults
→ Page Hits
→ Victim Pages
```

---

# 21. Fast Revision Tree

```text
PAGE REPLACEMENT
│
├── FIFO
│   ├── Oldest loaded page
│   ├── Hit does not refresh
│   └── Belady's Anomaly
│
├── Optimal
│   ├── Farthest future use
│   ├── Minimum faults
│   └── Theoretical benchmark
│
├── LRU
│   ├── Least recent past use
│   ├── Hit refreshes recency
│   └── LRU ≠ LFU
│
└── Second Chance / Clock
    ├── FIFO improvement
    ├── Reference bit
    ├── R=1 → second chance
    ├── R=0 → evict
    └── Clock = circular implementation
```

---

# 22. One-Line Final Revision

```text
FIFO      → oldest arrival
Optimal   → farthest future use
LRU       → least recent past use
2ndChance → FIFO + reference bit
Clock     → circular Second Chance
```
