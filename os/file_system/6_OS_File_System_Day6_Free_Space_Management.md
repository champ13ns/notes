# Operating Systems — File System Management, Day 6
## Free Space Management (IBPS RRB SO IT Focus)

**Purpose:** Detailed English revision notes, limited to concepts and numerical types most useful for IBPS RRB SO IT. **Not** a GATE advanced-problem set.

## 1. What is free space management?

A file system allocates storage blocks to hold file data and its metadata. As files are created, enlarged, truncated or deleted, blocks move between allocated and free states. **Free space management** is the file system's way of recording which allocation units are available and finding blocks when new storage is needed.

Key distinction:
- **File allocation method:** How the file system associates an individual file with its data blocks (e.g., contiguous, linked or indexed allocation in textbook models).
- **Free space management method:** How it identifies and tracks storage blocks that are currently free across the volume (e.g., bitmap or free list).

The two methods solve different problems but work together.

### Example
A volume contains 16 numbered allocation blocks (0–15). Suppose blocks 1, 4, 6, 7, 12 and 15 are currently allocated. The remaining 10 are free: 0, 2, 3, 5, 8, 9, 10, 11, 13 and 14. The system must be able to find those free blocks without searching every file's contents.

### Why it matters
- Allocate blocks when creating or growing files.
- Reclaim blocks when a file is actually released.
- Find suitable consecutive blocks for contiguous allocation where required.
- Track available volume capacity and avoid allocating the same block to two files.

**Reminder from hard links and file descriptors:** Removing a filename does not necessarily mean its data blocks are freed immediately. If other hard links or open references keep the file alive, the allocation remains in use under the file system's rules.

## 2. Bitmap / Bit Vector

A **bitmap** has one bit corresponding to each allocation block. A convention must be stated; in these notes:
- **1 = free**
- **0 = allocated**

Some textbooks or systems use the opposite convention. Always check what the exam question specifies.

Example (blocks 0–7):

| Block | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|---|
| State | Free | Used | Free | Free | Used | Free | Used | Used |
| Bit | 1 | 0 | 1 | 1 | 0 | 1 | 0 | 0 |

Bitmap string: `10110100` under the chosen convention.

If block 3 is allocated, bit 3 changes from 1 to 0. If block 4 is released, bit 4 changes from 0 to 1.

### Bitmap capacity formula
For **N allocation blocks**:

- Bitmap length = **N bits**.
- Bitmap size in bytes = **ceil(N / 8)**, assuming the bitmap is packed without extra padding.
- If volume size = **V bytes** and allocation block size = **B bytes**, assuming an exact division: **N = V / B**.
- So bitmap bytes = **ceil((V / B) / 8)**.

The bitmap may itself require allocated storage blocks; an exam may ask for **bitmap bytes** versus **blocks needed to store the bitmap**. Do not confuse these.

### Worked example A: 64 blocks
- Number of blocks = 64.
- Bitmap = 64 bits = 8 bytes.

### Worked example B: 1 GiB volume, 4 KiB blocks
- Volume size = 2^30 bytes.
- Block size = 2^12 bytes.
- Number of data/allocation blocks = 2^(30−12) = 2^18 = 262,144 blocks.
- Bitmap = 262,144 bits.
- Bitmap size = 262,144 / 8 = 32,768 bytes = **32 KiB**.
- If the bitmap itself is stored in 4 KiB blocks, this occupies **8 blocks** (excluding any additional bookkeeping).

**Exam trap:** 1 GiB = 1024 MiB; 1 GB (decimal) = 1000 MB. Use the units specified in the question.

### Advantages of bitmap
- Conceptually simple and compact (one bit per allocation unit).
- Easy to mark a block free or used by updating its bit.
- Can efficiently check runs of adjacent blocks when implemented using word-level bit operations and appropriate search strategies.

### Limitations
- The entire bitmap may be large for volumes with huge numbers of small blocks.
- The system has to find suitable zero/one bits (according to convention); an inefficient scan could be costly.
- File-system crash safety still requires consistent updates of bitmap and metadata.

## 3. Linked List / Free List

The file system can maintain a chain of free blocks. A reference identifies the first free block; each free block holds a pointer to the next free block.

Example free list: `3 -> 8 -> 11 -> 14 -> NULL`.

- Allocate: remove a free block from the list and use it.
- Free: return a released block to the list.

### Advantages
- Does not require a bit entry for every block in one central bitmap.
- Can be simple for allocating an individual free block.

### Disadvantages
- Following the chain to inspect many free blocks can be inefficient.
- Finding several **consecutive** free blocks is awkward, since list order may not be storage address order.
- Pointer management must remain consistent; a damaged link can affect traversal.

**Important:** This list tracks *free* storage blocks. Do not confuse it with **linked file allocation**, in which pointers form the chain of *allocated data blocks belonging to a particular file*.

## 4. Grouping

Grouping improves on a basic free list. In a classical grouping scheme, one free block contains addresses of several other free blocks; one entry may lead to another address-containing block. This means several free-block addresses can be obtained at once instead of following a single pointer from block to block.

**Key idea:** Store multiple free-block addresses together.

**Do not confuse** grouping with an indirect inode index. Grouping stores *free-space information*, not the mapping of an individual file's data blocks.

## 5. Counting

Rather than list every free block individually, store information about **runs of consecutive free blocks**.

For example, if free blocks are 20, 21, 22, 23, 24, 50 and 51, counting can represent them as:
- (start=20, count=5)
- (start=50, count=2)

The numbers represent *free extents/runs* in the textbook scheme.

### Advantages
- Efficient when large contiguous runs are free.
- Compact representation: store the start and length rather than every address.

### Limitations
- Less compact when free blocks are highly scattered in many small runs.
- Updating runs may involve splitting or merging adjacent ranges.

## 6. Comparing the methods

| Method | Representation | Strength | Main limitation |
|---|---|---|---|
| Bitmap / bit vector | One bit for each block | Compact and easy to check block state | May require searching a large bitmap |
| Free linked list | Each free block leads to the next | Simple single-block allocation | Traversal and contiguous-run search can be slow |
| Grouping | One location lists several free block addresses | Obtains multiple free addresses at once | More pointer/list management |
| Counting | Start block + length of a free run | Efficient for consecutive free blocks | Poorer compression when heavily scattered |

Do not assume only one universal implementation. Real file systems may combine techniques, maintain per-group metadata, or use trees of free extents.

## 7. Relation to fragmentation

### External fragmentation
Enough total free blocks exist, but no sufficiently large **contiguous run** exists for a request that requires one continuous range.

Example free runs: 3 blocks, 2 blocks and 4 blocks (9 blocks free total); a request for 5 contiguous blocks cannot be satisfied without rearrangement or a different strategy.

### Internal fragmentation
A file's last allocated block may not be completely filled. In a fixed-block example:
- File size = 10 KiB.
- Allocation block size = 4 KiB.
- Blocks required = ceil(10/4) = 3.
- Allocated capacity = 12 KiB.
- Unused part of last block = 2 KiB.

**Do not confuse:** A bitmap does not itself solve fragmentation; it tracks which units are available.

## 8. Crash consistency and block reclamation (conceptual)

When blocks are allocated or released, metadata such as file block maps and free-space information must be kept consistent. Unexpected crashes during updates can otherwise produce inconsistencies. Journaling, copy-on-write strategies and consistency checkers are related mechanisms, covered in a later session.

## 9. IBPS RRB SO IT exam traps

1. A file allocation technique and a free-space tracking technique answer different questions.
2. The meaning of bitmap bit 0 or 1 depends on the convention stated.
3. Each bitmap **bit** describes one allocation block; it does not contain that block's file data.
4. Bitmap size = N **bits**, not N bytes.
5. Dividing by 8 converts bits to bytes. Dividing by 1024 converts KiB/MiB/GiB levels (binary).
6. A free list is different from linked allocation of a file's own data.
7. Grouping stores multiple free addresses together; counting stores start-and-run-length pairs.
8. A request for a contiguous free run may fail even if total free blocks are sufficient.
9. Deleting a directory entry does not necessarily free its file blocks immediately if references still keep the file alive.
10. For bitmap **storage blocks**, first calculate bitmap bytes and then ceil-divide by block size.

## 10. Practice questions (answers withheld for self-test)

**Q1.** A file system has 8,192 allocation blocks. How many bytes are needed for a packed one-bit-per-block bitmap?  
A. 512   B. 1,024   C. 4,096   D. 8,192

**Q2.** A bitmap uses `1 = free` and `0 = allocated`. Which string represents blocks 0–3 as free, allocated, allocated, free?  
A. 0110   B. 1001   C. 1010   D. 1100

**Q3.** Which method records a run as `(starting free block, number of consecutive free blocks)`?  
A. Bitmap   B. Linked allocation   C. Counting   D. Indexed allocation

**Q4.** A disk has free runs of 4, 2 and 3 blocks. A classical contiguous allocation request needs 5 blocks. What follows?  
A. It succeeds because 9 blocks are free.  
B. It fails without rearrangement because no single free run has 5 blocks.  
C. It fails only if a bitmap is used.  
D. It succeeds only with linked free-space management.

**Q5.** A volume has 1 GiB of allocation space and 4 KiB blocks. How much packed bitmap space is required?  
A. 8 KiB   B. 32 KiB   C. 256 KiB   D. 1 MiB

**Q6.** Which statement is correct?  
A. Linked free-space management describes one file's allocated data-block chain.  
B. Grouping stores several free-block addresses together.  
C. Bitmap-based free-space tracking always prevents fragmentation.  
D. Counting always uses one pointer for every free block.

## 11. Revision checklist

- [ ] Explain free space management vs file allocation.
- [ ] Calculate bitmap bits, bytes and storage blocks.
- [ ] Explain a free-block list and distinguish it from linked file allocation.
- [ ] Explain grouping and counting with a short example.
- [ ] Identify external vs internal fragmentation.
- [ ] Solve Q1–Q6 without looking at notes.

**Next in sequence:** File System Consistency, Journaling and Recovery; then File Protection/Permissions; final mixed File System MCQs.
