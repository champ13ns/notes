# Operating Systems — Segmentation, Paging vs Segmentation & Segmentation with Paging

> **Scope:** These notes continue **after Thrashing, Working Set Model, and PFF**.  
> **Topics:** Segmentation, Segment Table, Fragmentation, Paging vs Segmentation, Segmentation with Paging, and final Memory Management wrap-up.

---

# 1. Segmentation — Core Idea

In paging, a process is divided into **fixed-size pages**.

In segmentation, a process is divided according to its **logical components**.

Example:

```text
Process
├── Code Segment
├── Data Segment
├── Heap Segment
└── Stack Segment
```

These segments are **variable-sized**.

Example:

```text
Code  = 20 KB
Data  = 8 KB
Heap  = 15 KB
Stack = 4 KB
```

So:

```text
Paging
→ Fixed-size division

Segmentation
→ Variable-size logical division
```

---

# 2. Why Segmentation?

A programmer naturally thinks of a program as logical sections:

```text
Code
Data
Heap
Stack
Functions
Modules
Shared Libraries
```

Segmentation preserves this logical organization.

Paging does not care about these boundaries. A page is simply a fixed-size block of virtual memory.

---

# 3. Logical Address in Segmentation

In segmentation, a logical address is represented as:

```text
[ Segment Number | Offset ]
```

Example:

```text
Segment = 2
Offset  = 500
```

Meaning:

> Access byte/location 500 inside Segment 2.

---

# 4. Segment Table

Paging uses a:

```text
Page Table
```

Segmentation uses a:

```text
Segment Table
```

Each segment-table entry mainly contains:

```text
Base
Limit
```

## Base

Physical starting address of the segment.

## Limit

Length/size of that segment.

Example:

```text
Segment   Base    Limit
0         1000     400
1         5000    1200
2         9000     700
```

---

# 5. Address Translation in Segmentation

Suppose CPU generates:

```text
(Segment = 1, Offset = 500)
```

Segment 1:

```text
Base  = 5000
Limit = 1200
```

First check:

```text
Offset < Limit ?
```

```text
500 < 1200
```

Valid.

Then:

```text
Physical Address
= Base + Offset

= 5000 + 500
= 5500
```

---

# 6. Invalid Segment Access

Suppose:

```text
Segment = 1
Offset  = 1500
```

But:

```text
Limit = 1200
```

Check:

```text
1500 < 1200
```

False.

So:

```text
Invalid logical address
```

The system raises a protection/address fault.

Important rule:

```text
If Offset < Limit
→ Valid

Physical Address = Base + Offset
```

Otherwise:

```text
Invalid Access
```

---

# 7. Segmentation Numerical Example

Given:

```text
Segment   Base   Limit
0         2000    600
1         5000    900
2         8000    400
```

## A. (0, 450)

```text
450 < 600
→ Valid
```

Physical Address:

```text
2000 + 450 = 2450
```

## B. (1, 850)

```text
850 < 900
→ Valid
```

Physical Address:

```text
5000 + 850 = 5850
```

## C. (2, 500)

```text
500 < 400
→ False
```

Therefore:

```text
Invalid Address
```

No physical address is calculated.

---

# 8. Physical Placement of Segments

The whole process does not have to be one single contiguous block.

Example:

```text
RAM

+----------------+
| Segment B      |
+----------------+
| Free           |
+----------------+
| Segment A      |
+----------------+
| Segment D      |
+----------------+
| Free           |
+----------------+
| Segment C      |
+----------------+
```

However, in **pure segmentation**, each individual segment is traditionally stored in a contiguous region.

So:

```text
Whole Process Contiguous?
→ No

Each Individual Segment Contiguous?
→ Yes
```

---

# 9. External Fragmentation in Segmentation

Because segments are variable-sized and require contiguous physical space, external fragmentation can occur.

Example:

```text
Free Holes:

4 KB
5 KB
4 KB
```

Total free memory:

```text
13 KB
```

Suppose a new segment requires:

```text
8 KB
```

There is enough total free memory:

```text
13 KB
```

But there is no single contiguous 8 KB hole.

Therefore:

> **External Fragmentation**

Important:

If total required memory itself is larger than total available memory, that is **not fragmentation**. That is simply insufficient memory.

Example:

```text
Available = 13 KB

Code = 6 KB
Heap = 8 KB

Total required = 14 KB
```

This is not fragmentation.

It is:

```text
Insufficient memory
```

---

# 10. Internal Fragmentation in Segmentation

Classical exam answer:

```text
Pure Segmentation
→ External fragmentation possible
→ Little/no fixed-block internal fragmentation
```

Why?

Because segments are variable-sized, so allocation can approximately match the requested segment size.

Unlike paging:

```text
Need 18 KB
Page size = 4 KB

5 pages allocated = 20 KB
2 KB unused
```

That unused space is paging's internal fragmentation.

---

# 11. Paging vs Segmentation

## Paging

```text
Process
→ fixed-size pages
```

Physical memory:

```text
RAM
→ fixed-size frames
```

Address:

```text
[ Page Number | Offset ]
```

Mapping:

```text
Page Number
→ Page Table
→ Frame Number
```

## Segmentation

```text
Process
→ logical variable-size segments
```

Address:

```text
[ Segment Number | Offset ]
```

Mapping:

```text
Segment Number
→ Segment Table
→ Base + Limit
```

Then:

```text
Physical Address = Base + Offset
```

after checking the limit.

---

# 12. Paging vs Segmentation — Comparison Table

| Feature | Paging | Segmentation |
|---|---|---|
| Division | Fixed-size pages | Variable-size logical segments |
| Main Goal | Efficient physical memory management | Logical program organization |
| Logical Address | Page + Offset | Segment + Offset |
| Table | Page Table | Segment Table |
| Table Entry | Frame number + control bits | Base + Limit |
| Physical Contiguity | Pages can be anywhere | Each segment traditionally contiguous |
| External Fragmentation | No | Yes |
| Internal Fragmentation | Possible | Usually little/no fixed-block IF |
| Programmer View | Mostly hidden | Logical and meaningful |
| Protection/Sharing | Possible at page level | Natural at segment level |

---

# 13. Why Paging Is Usually Better for Physical Allocation

Pure paging avoids the large contiguous-allocation problem.

Suppose:

```text
Need = 10 KB
Page size = 4 KB
```

The process needs:

```text
3 pages
```

Those pages can be placed in unrelated frames:

```text
Page 0 → Frame 10
Page 1 → Frame 37
Page 2 → Frame 4
```

No large contiguous 10 KB hole is required.

That is a major advantage over pure segmentation.

---

# 14. Why Segmentation Is Still Useful

Segmentation solves a different problem.

It preserves logical regions such as:

```text
Code
Data
Heap
Stack
Shared Library
```

This makes protection and sharing conceptually natural.

Example:

```text
Code Segment
→ Read + Execute

Data Segment
→ Read + Write

Stack Segment
→ Read + Write
```

So:

```text
Paging
→ stronger for physical memory allocation

Segmentation
→ stronger for logical organization
```

---

# 15. Segmentation with Paging

To combine advantages of both techniques:

> First divide the process into logical segments, then divide each segment into fixed-size pages.

Example:

```text
Process

Code Segment
├── Page 0
├── Page 1
├── Page 2
└── Page 3

Heap Segment
├── Page 0
├── Page 1
└── Page 2

Stack Segment
├── Page 0
└── Page 1
```

Now each page can be placed in any available physical frame.

---

# 16. Why Segmented Paging Helps

In pure segmentation:

```text
Code Segment = 20 KB
```

The complete segment may require one contiguous 20 KB physical region.

In segmentation with paging:

```text
Code Segment
├── P0
├── P1
├── P2
├── P3
└── P4
```

These pages can be placed like:

```text
P0 → Frame 9
P1 → Frame 23
P2 → Frame 4
P3 → Frame 40
P4 → Frame 15
```

So the segment no longer needs to occupy one contiguous physical area.

---

# 17. Logical Address in Segmented Paging

The logical address conceptually becomes:

```text
[ Segment Number | Page Number | Offset ]
```

Meaning:

```text
Segment Number
→ Which logical segment?

Page Number
→ Which page inside that segment?

Offset
→ Which byte/location inside that page?
```

---

# 18. Translation Flow in Segmented Paging

```text
CPU Logical Address
[ Segment | Page | Offset ]
          ↓
Segment Table
          ↓
Find that Segment's Page Table
          ↓
Use Page Number
          ↓
Find Frame Number
          ↓
Physical Address
[ Frame | Offset ]
```

So the segment table helps select the correct page table.

Then the page table maps the page to a frame.

---

# 19. Segmented Paging Numerical

Suppose:

```text
Page Size = 1 KB = 1024 bytes
```

Segment Table:

```text
Segment    Page Table
S0         PT0
S1         PT1
S2         PT2
```

Page table for `S1`:

```text
Page      Frame
0          4
1          9
2         12
3          7
```

CPU generates:

```text
Segment = 1
Page    = 2
Offset  = 300
```

Step 1:

```text
Segment 1
→ Select PT1
```

Step 2:

```text
PT1[2]
→ Frame 12
```

Step 3:

```text
Physical Address
= Frame × Page Size + Offset
```

```text
= 12 × 1024 + 300
= 12288 + 300
= 12588
```

Final:

```text
Physical Address = 12588
```

---

# 20. Does Segmented Paging Eliminate Page Faults?

No.

Suppose Code Segment has:

```text
P0 P1 P2 ... P10
```

and the process has only 5 physical frames available.

All 11 code pages do not need to be resident simultaneously.

If current execution repeatedly uses:

```text
P0 P1 P2
```

then 5 frames are enough.

But if the active locality is larger than the allocated frames, then:

```text
Page Faults
Page Replacement
High PFF
Possible Thrashing
```

can still occur.

So segmented paging solves:

```text
Logical organization
+
contiguous-allocation problem
```

It does **not** remove:

```text
Page Faults
Page Replacement
Thrashing
```

---

# 21. Segmented Paging and Working Set Connection

Suppose the process contains 50 total pages across all segments.

That does not mean all 50 must be in RAM.

Only the current working set may need to remain resident.

Example:

```text
Total Process Pages = 50

Current Working Set:
{Code P2, Code P3, Heap P1, Stack P0}
```

Working Set Size:

```text
4
```

If enough frames are available for the active locality, execution can proceed with relatively few faults.

---

# 22. Three-Way Comparison

| Feature | Pure Paging | Pure Segmentation | Segmentation + Paging |
|---|---|---|---|
| Logical Division | Fixed pages | Logical segments | Segments then pages |
| Unit Size | Fixed | Variable | Pages fixed |
| Whole Segment Physically Contiguous? | Not applicable | Yes traditionally | No |
| External Fragmentation | No | Yes | Largely eliminated by paging |
| Internal Fragmentation | Possible | Little/no fixed-block IF | Possible at page level |
| Logical Organization | Weak | Strong | Strong |
| Page Faults Possible | Yes | Not in same paging sense | Yes |
| Page Replacement Possible | Yes | Not page-based unless paged | Yes |
| Thrashing Possible | Yes | Not in same paging sense | Yes |

---

# 23. Final Mental Picture

## Pure Paging

```text
Process
↓
P0 P1 P2 P3 P4 ...
↓
Page Table
↓
Physical Frames
```

Address:

```text
[ Page | Offset ]
```

## Pure Segmentation

```text
Process
↓
Code | Data | Heap | Stack
↓
Segment Table
↓
Contiguous region for each segment
```

Address:

```text
[ Segment | Offset ]
```

## Segmentation + Paging

```text
Process
↓
Segments
↓
Each Segment divided into Pages
↓
Segment-specific Page Tables
↓
Pages mapped to arbitrary Frames
```

Address:

```text
[ Segment | Page | Offset ]
```

---

# 24. Memory Management — Final Topic Flow

```text
Memory Management
│
├── Contiguous Allocation
│
├── Fragmentation
│   ├── Internal Fragmentation
│   └── External Fragmentation
│
├── Paging
│   ├── Pages
│   ├── Frames
│   ├── Page Number
│   ├── Offset
│   ├── Page Table
│   ├── PTE
│   ├── Present / Valid Bit
│   ├── Dirty Bit
│   ├── Reference Bit
│   ├── Protection Bits
│   └── TLB
│
├── Multi-Level Paging
│
├── Virtual Memory
│   ├── Demand Paging
│   └── Page Fault
│
├── Page Replacement
│   ├── FIFO
│   ├── Belady's Anomaly
│   ├── Optimal
│   ├── LRU
│   └── Second Chance / Clock
│
├── Thrashing
│   ├── Working Set Model
│   └── PFF
│
├── Segmentation
│   ├── Segment Number
│   ├── Offset
│   ├── Base
│   ├── Limit
│   └── External Fragmentation
│
└── Segmentation + Paging
    ├── Segment Number
    ├── Page Number
    ├── Offset
    └── Segment Table → Page Table → Frame
```

---

# 25. Final Exam Traps

## Trap 1

```text
Paging divides RAM into pages.
```

Wrong.

```text
Process/virtual memory → Pages
Physical RAM → Frames
```

## Trap 2

```text
Segmentation uses fixed-size segments.
```

Wrong.

```text
Segments are variable-sized logical units.
```

## Trap 3

```text
Base means segment size.
```

Wrong.

```text
Base = physical starting address
Limit = segment size / valid offset range
```

## Trap 4

```text
Physical Address = Base + Offset
```

Not automatically.

First:

```text
Offset < Limit
```

must be true.

## Trap 5

External fragmentation does not mean total memory is insufficient.

It means:

```text
Enough total free memory exists
but it is split into non-contiguous holes.
```

## Trap 6

Pure segmentation can suffer from:

```text
External Fragmentation
```

## Trap 7

Segmented paging does not eliminate page faults.

## Trap 8

All pages of a code segment do not need to remain in RAM simultaneously.

Demand paging still applies.

---

# 26. One-Line Revision

```text
Paging
→ Fixed-size pages mapped to frames
```

```text
Segmentation
→ Variable-size logical segments using Base + Limit
```

```text
Segmented Paging
→ Logical segments, each further divided into fixed-size pages
```

```text
Pure Segmentation
→ External fragmentation possible
```

```text
Paging / Segmented Paging
→ Internal fragmentation possible
```

---

# 27. Final Memory Management Summary

```text
Contiguous Allocation
→ simple but fragmentation problems

Paging
→ fixed-size allocation, no external fragmentation

Virtual Memory
→ whole process need not be resident

Demand Paging
→ load pages when required

Page Replacement
→ choose victim when no free frame

Thrashing
→ excessive page faults

Working Set / PFF
→ control memory pressure

Segmentation
→ preserve logical program structure

Segmentation + Paging
→ logical organization + non-contiguous physical placement
```

> **Memory Management section wrapped.**
