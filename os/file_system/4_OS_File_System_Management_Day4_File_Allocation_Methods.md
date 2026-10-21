# Operating Systems — File System Management
## Day 4: File Allocation Methods (Contiguous, Linked, Indexed)

**Level:** IBPS RRB SO IT Officer + GATE conceptual depth  
**Language:** English  
**Prerequisites:** inode, directory entry, file offset, file-system block, direct vs indexed file access, hard/symbolic links.

---

## 1. Why file allocation is needed

A file system manages how logical blocks belonging to a file are placed in the logical block-address space of a storage volume. The blocks for a file do not have to be consecutive.

**Important:** File-system storage blocks are **not RAM frames**. The OS can cache disk/SSD data in RAM, but the storage mapping and RAM virtual-memory page table are different mechanisms.

Consider a 14 KiB file and 4 KiB allocation blocks:

- Logical file block 0: offsets 0–4095
- Logical file block 1: offsets 4096–8191
- Logical file block 2: offsets 8192–12287
- Logical file block 3: offsets 12288–14335 (2 KiB of file data)
- Allocated blocks: ceil(14/4) = **4**
- Unused allocated capacity in final block: 16 – 14 = **2 KiB**

The three textbook techniques differ in **how logical file blocks map to allocated storage blocks**.

---

## 2. Contiguous allocation

The file occupies a consecutive run of storage blocks.

Example: logical file blocks 0, 1, 2, 3 are stored in storage blocks **40, 41, 42, 43**.

```text
File offset:   0..4KiB    4..8KiB    8..12KiB   12..14KiB
Logical block:    0          1          2          3
                  |          |          |          |
Storage block:   40         41         42         43

File allocation metadata: start_block = 40, block_count = 4
```

**Mapping formula (zero-based logical block index i):**

`storage_block = start_block + i`

For byte offset `x`, `i = floor(x / block_size)`, and position inside the block is `x mod block_size`.

**Advantages**
- Simple metadata: initial storage block and number of blocks.
- Fast direct/random access, since the nth block's location is computed immediately.
- Good locality for sequential reads (especially relevant on HDDs).

**Disadvantages**
- **External fragmentation:** enough free blocks may exist, but not in one sufficiently long contiguous run.
- Growing a file may require moving/reallocating the file if blocks immediately following it are occupied.
- May require size prediction or extents/other design improvements.

**External fragmentation example:** free runs have lengths 3, 2, and 4 blocks. Total free = 9 blocks, but a file requiring 5 contiguous blocks cannot fit within any individual run.

**Exam trap:** Storage block addresses are logical addresses of the volume, not necessarily physically adjacent flash cells inside an SSD.

---

## 3. Linked allocation

The file's blocks can be anywhere; each allocated storage block holds a pointer to the next block (in the classical textbook model).

```text
Directory/inode metadata: first block = 7

Storage block 7  ->  storage block 12  ->  storage block 15
                                                   |
                                                   v
                              EOF <- storage block 19

File block index:     0              1              2              3
```

To retrieve file block 3, a classical linked allocation system walks 7 -> 12 -> 15 -> 19. There is no simple arithmetic formula like `start + i`.

**Advantages**
- No **external fragmentation** requirement: nonconsecutive free blocks can be used.
- Easy incremental growth: allocate a free block and link it at the end.
- Good fit for sequential traversal.

**Disadvantages**
- Poor direct/random block access if the chain must be traversed from the beginning: O(n) in block count for the nth block.
- Pointers consume metadata space; if pointer bytes are stored inside each allocation block, they reduce payload space.
- Pointer corruption can break the remainder of a chain.
- Nonlocal accesses can increase seeking overhead on HDDs.

**Important nuance:** There can still be unused capacity in the final block (internal fragmentation); "no external fragmentation" does **not** mean "no fragmentation of any kind."

### File Allocation Table (FAT): linked allocation variant

FAT stores next-block/cluster links in a separate table, rather than embedded in each file-data block:

```text
FAT entry for block 7  = 12
FAT entry for block 12 = 15
FAT entry for block 15 = 19
FAT entry for block 19 = EOF
```

If relevant FAT entries are cached in RAM, chain traversal can avoid reading intermediate **data** blocks, but finding the nth link still requires following the chain unless an additional index/cache is used. FAT does not automatically turn the chain into a simple O(1) indexed mapping.

---

## 4. Indexed allocation

An **index block** (or more general index structure) stores the storage-block addresses for a file's logical blocks.

```text
File metadata -> Index block 300

Index block 300:
  entry[0] = 7
  entry[1] = 12
  entry[2] = 15
  entry[3] = 19

Logical file block 2 -> index entry[2] -> storage block 15
```

**Advantages**
- Supports direct access to arbitrary file blocks through the index.
- No need for data blocks to be contiguous; avoids the external fragmentation requirement of simple contiguous allocation.
- Sequential reads also work.

**Disadvantages**
- Index storage occupies extra space; may be inefficient for tiny files.
- A single index block has finite pointer capacity.
- For very large files, additional levels of indexing or a different mapping method are needed.
- If index metadata is not cached, extra storage reads may be required.

### Single-index-block numerical example

Assume index-block size 4 KiB and storage block addresses/pointers are 4 bytes each, with the entire index block available for pointers.

`pointers_per_index_block = 4096 / 4 = 1024`

A single index block can map at most **1024 data blocks**. At 4 KiB per data block, the maximum **data** capacity is:

`1024 × 4096 = 4,194,304 bytes = 4 MiB`

This excludes index-block overhead and assumes the specified simple, single-index-block model. Unix-style inode schemes with direct, single-indirect, double-indirect, and triple-indirect pointers use additional structures that we will study separately.

---

## 5. Important distinction: Indexed FILE ACCESS vs Indexed FILE ALLOCATION

**Indexed file access (record search):** `customer_id=9001 -> index (e.g., B+ tree) -> record position`.

**Indexed file allocation (storage block mapping):** `logical file block 2 -> index pointer -> storage block 15`.

The first solves a **search-by-key** problem; the second solves a **file-block location** problem. Both might be used together when a DBMS retrieves a record from a database file.

---

## 6. Comparison table

| Feature | Contiguous | Linked (classical) | Indexed (single index level) |
|---|---|---|---|
| Placement | Consecutive blocks | Anywhere; chained | Anywhere; index mapping |
| Mapping data | Start + length | First block + next pointers | Index block/pointer structure |
| Access nth file block | O(1) mapping | O(n) chain traversal | O(1) index entry after index available |
| Sequential access | Good | Good in order | Good |
| External fragmentation issue | Yes | No requirement for continuous free run | No requirement for continuous free run |
| Growth | Difficult if adjacent space used | Easy to append | Easy within index capacity |
| Extra metadata | Low | Per-block pointers / FAT | Index block(s) |
| Final-block internal fragmentation | Possible | Possible | Possible |

**Important:** Big-O entries describe metadata navigation in these simplified designs; actual I/O time depends on caching, device, layout, file size, and implementation.

---

## 7. How to choose a method in a question

1. "Start block 50, length 8" and address `start+i` -> contiguous allocation.
2. "Every block stores the location of the next block" -> linked allocation.
3. "A table of block pointers for the file" -> indexed allocation.
4. "FAT entries point to the next cluster" -> linked allocation with a separate allocation table.
5. "Search a record by customer ID using B+ tree" -> indexed **file access**, not necessarily indexed allocation.

Modern file systems can use **extents**, trees, and combinations of techniques, rather than one simplistic textbook model. An extent describes a contiguous run and allows multiple runs per file; this combines some locality advantages of contiguous allocation with greater flexibility for growth.

---

## 8. Practice — answer without looking up solutions

**Q1.** A file occupies six contiguous storage blocks starting at block 240. Where is logical file block 4 stored, with zero-based numbering?

A. 243  B. 244  C. 245  D. 246

**Q2.** Free storage is split into runs of 2, 3, and 4 blocks. A new file needs 5 blocks. Which of the three classical methods is prevented specifically by the lack of a 5-block continuous free run?

A. Simple contiguous allocation  B. Linked allocation  C. Indexed allocation  D. All three

**Q3.** In a classical linked-allocation file, block pointers are stored in each data block. Which statement is correct?

A. Direct retrieval of block n always uses start+n.
B. A block's pointer refers to the preceding block.
C. Reaching block n generally requires following the chain from the beginning.
D. Data blocks must be adjacent.

**Q4.** An index block has 2,048 bytes, and each block pointer occupies 8 bytes. How many direct data-block addresses fit in one fully dedicated index block?

A. 128  B. 256  C. 512  D. 2,048

**Q5.** A file uses an inode with indexed block mappings. An application separately uses a B+ tree on customer IDs to find record byte offsets. Which is accurate?

A. The B+ tree replaces the inode.
B. Both mechanisms do the same job.
C. The B+ tree locates the logical record; file-system metadata maps its file offset to storage.
D. The inode stores the customer ID values.

**Q6.** A file occupies 10 KiB of data and block size is 4 KiB. Under full-block allocation with no sparseness, what is the allocated capacity and unused last-block capacity?

A. 8 KiB and 0  B. 12 KiB and 2 KiB  C. 12 KiB and 4 KiB  D. 16 KiB and 6 KiB

---

## 9. Planned next subtopic

Once allocation fundamentals are solid: **FAT vs Unix inode addressing**, including direct/indirect/double-indirect/triple-indirect pointers, pointer counts, maximum file-size formulas, and GATE numericals.

**Self-check:** Can you explain why `file_block_2 -> storage_block_15` is possible under linked/indexed allocation, and why storing blocks non-consecutively is different from RAM paging?
