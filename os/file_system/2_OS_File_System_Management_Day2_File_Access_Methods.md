# Operating Systems — File System Management
## Day 2: File Access Methods (Sequential, Direct, Indexed)

**Target:** IBPS RRB SO IT Officer + GATE-level conceptual confidence  
**Language:** English  
**Prerequisites:** File, inode, file descriptor, open file description, file offset, file blocks

---

## 0. Connection with Day 1

- **File**: named sequence of bytes (for a regular file) and associated metadata.
- **Inode** (in Unix-style inode file systems): stores a file's metadata and information required to locate its content. A directory maps a name to an inode number.
- **File descriptor (FD)**: process-local integer indexing an entry in that process's descriptor table.
- **Open file description**: kernel-side object tracking the file offset and file status flags for an open instance. Different FDs may share this object after `dup()` or `fork()`.
- **File offset**: position in the *logical contents of the file*, measured in bytes. It is **not** a RAM address or a physical disk address.
- **File system block**: logical allocation/mapping unit on storage. **Not** the same as a physical RAM frame.

### Warm-up: `open()` vs `dup()`

```c
int a = open("data.txt", O_RDONLY); // distinct open file description A
int b = open("data.txt", O_RDONLY); // distinct open file description B
int c = dup(a);                      // same open file description as a
```

- `a` and `c` **share** an offset.
- `b` has an **independent** offset, although all three refer to the same underlying file.
- Two live FDs in one process cannot have the same integer; different processes may each have FD 6.

---

## 1. What is a File Access Method?

An **access method** describes how a program finds and uses positions or records **within a file**.

Do not confuse:

| Concept | Question answered |
|---|---|
| File access method | How do we navigate to the needed bytes or records? |
| File allocation method | How are the file's data blocks assigned on storage? |
| File descriptor | Which open resource does this process mean? |
| Inode/block mapping | Where is the file's data stored, and what is its metadata? |

A fragmented file on an SSD can still be read sequentially or randomly. Access order does **not** imply physical placement.

## 2. Sequential Access

**Meaning:** Process contents in order, typically progressing from the beginning toward the end.

```text
File logical order:
  [Record 0] -> [Record 1] -> [Record 2] -> [Record 3] -> [Record 4]
                ^ reading proceeds in sequence
```

**Operations in the classical model:** `read next`, `write next`, `rewind` (or reset).

**Example:** Scan a server log line-by-line to count errors.

```js
const fs = require('node:fs');
const readline = require('node:readline');
const rl = readline.createInterface({
  input: fs.createReadStream('server.log'),
});
rl.on('line', line => {
  if (line.includes('ERROR')) console.log(line);
});
```

**Good for:** Logs, streamed exports, media streaming, full-file scans, tape-like processing.

**Tradeoff:** A purely sequential scan to the 900th record may have to traverse the 899 preceding records. But caching, search algorithms, and backing structures can change real performance.

**Exam trap:** Reading sequentially does **not** require contiguous physical disk blocks.

## 3. Direct (Random / Relative) Access

**Meaning:** Navigate directly to a specified file position or numbered fixed-size record, without reading every preceding item.

```text
Logical file bytes:
  [0 ... 999][1000 ... 1999][2000 ... 2999][3000 ...]
                   direct seek ------------------------^
```

### Unix example: `lseek()`

```c
lseek(fd, 5000, SEEK_SET);  // change this open file description's offset
ssize_t n = read(fd, buffer, 100); // attempt to read 100 bytes
```

- After `lseek`, current offset is 5000 if it succeeds.
- After `read`, current offset normally advances by the **actual** number of bytes read (which may be less than 100).
- `SEEK_SET`: position relative to file start.
- `SEEK_CUR`: position relative to current offset.
- `SEEK_END`: position relative to end of file.
- Not all descriptors support seeking (e.g. pipes generally do not).

### `pread()` — position-based read without changing shared current offset

```c
ssize_t n = pread(fd, buffer, 100, 5000);
```

For a regular file, this attempts to read 100 bytes starting at byte offset 5000, while leaving the open file description's current offset unchanged. This is useful when threads share an open file description but need independent positional reads.

### Fixed-record numerical

File contains equal-length records of 128 bytes, numbered **0, 1, 2, ...**.

Find starting byte offset of record number 500:

```
Offset = record_number * record_size
       = 500 * 128
       = 64,000 bytes
```

If a question uses **1-based record numbers**, then `start_offset = (record_number - 1) * record_size`.

### What about variable-size records?

If records have different lengths, the offset of record 500 is not simply `500 * constant`. You may need a record directory, index, or scanning to identify its start.

**Exam traps:**
- File byte offset != RAM physical address.
- Random/direct access is possible even when file storage blocks are non-contiguous.
- Direct access is not necessarily constant-time at the device level; storage/device costs and lookup structures matter.
- File allocation (contiguous, linked, indexed) and file access method (sequential, direct, indexed) are different classifications.

## 4. Indexed Access

**Meaning:** An additional *index structure* helps locate the desired record using a key (e.g., customer ID) or logical record number.

```text
Wanted: customer ID = 9001
                  |
                  v
       Index (key -> location)
       1020 -> position 0
       5020 -> position 4
       9001 -> position 9  -----> [matching record in data file]
```

**Important:** This section describes *indexed access to file records*. It is not identical to the *indexed allocation* method where an index block stores pointers to storage blocks. Similar terminology, different layers/problems.

### Example: student records

```text
Data file:
  [ID 1020][ID 9001][ID 3050][ID 5020]

Index (conceptual):
  1020 -> data offset 0
  3050 -> data offset 256
  5020 -> data offset 384
  9001 -> data offset 128
```

Assume four fixed-size 128-byte records. An index entry maps each ID to its starting byte offset. To locate ID 9001, search index, then read the data at offset 128.

### Pros
- Efficient key-based lookups if the index is properly organized.
- Can avoid full sequential scans.
- Supports meaningful searches (e.g., by customer ID) rather than only numeric byte offsets.

### Costs
- Index consumes extra storage/RAM.
- Inserts/deletes/updates may need index maintenance.
- Lookup cost depends on index design (e.g., B-tree/B+ tree, hash, sorted index), size, and caching.

### Dense vs sparse index (introductory)

- **Dense index:** roughly one index entry per record or search-key value, depending on index design.
- **Sparse index:** entries point to selected positions/groups; then the target is found within the group. Typically requires suitable ordering of the data file for efficient search.

Do not assume all indexes are dense, all are sparse, or all take precisely two disk accesses.

### Indexed Sequential Access (ISAM) — introductory note

An indexed-sequential file organization keeps records in a searchable order and uses an index to locate a starting region. It can support both indexed lookups and ordered sequential processing. Classical ISAM differs from modern dynamically balanced indexes such as B+ trees.

## 5. Three ways to locate a record

Suppose 1,000 fixed-length records exist; you want ID=9001.

| Method | General approach | Typical strength | Main caveat |
|---|---|---|---|
| Sequential | Scan until ID matches | Simple; good for complete scans | Potentially many records examined |
| Direct | Go to a known record number or byte offset | Fast positional access | Must know/compute position |
| Indexed | Search key in index, then follow location | Efficient key-based search | Index storage and maintenance overhead |

**Key insight:** Direct access answers "go to byte 64,000"; indexed access answers "find customer 9001" by supplying or deriving a location.

## 6. File offset vs index vs storage block mapping

```text
Application wants customer ID 9001
             |
             v
      Search an index (if present)
             |
             v
    Data file byte offset = 128
             |
             v
 OS maps relevant file block(s)
             |
             v
  Cache / storage volume blocks
             |
             v
      Bytes returned to app
```

The *index* may exist in application software or DBMS rather than in the general-purpose OS file system. The OS file system does not automatically provide a customer-ID index for arbitrary files.

## 7. Connection to Memory Management

**Paging:** virtual memory page -> physical RAM frame.

**File system mapping:** logical file byte/block -> file-system storage block(s), often with metadata such as extents or pointers.

Both use mappings; they are not the same address space or the same hardware. The OS may also cache file contents in RAM as page-cache pages.

## 8. Quick worked problems

### Problem A: Byte offset

A file contains 400 records, each 64 bytes, numbered 0–399. Find the starting position of record 125.

```
125 * 64 = 8,000 bytes
```

### Problem B: Current offset

A read-only FD for a regular file is at offset 20. A `read(fd, buf, 12)` successfully returns 9 bytes. New offset?

```
20 + 9 = 29 bytes
```

**Trap:** use actual bytes returned, not requested bytes.

### Problem C: `lseek` + read

An open file description is at offset 70. `lseek(fd, -10, SEEK_CUR)` succeeds, then a `read` returns 4 bytes.

```
After seek: 60
After read: 64
```

### Problem D: `dup` versus independent `open`

```c
int fd1 = open("f", O_RDONLY);
int fd2 = open("f", O_RDONLY);
int fd3 = dup(fd1);
```

`fd1` and `fd3` share their file offset; `fd2` has an independent offset.

## 9. Frequent exam misconceptions

1. **Wrong:** Sequential access requires adjacent disk blocks.  
   **Correct:** Sequential describes access order, not physical placement.
2. **Wrong:** File descriptor is the disk block address.  
   **Correct:** FD is a per-process reference to an open kernel resource.
3. **Wrong:** Two independent `open()` calls always share offsets if they reference the same inode.  
   **Correct:** They ordinarily create separate open file descriptions with independent offsets.
4. **Wrong:** `dup()` makes another inode.  
   **Correct:** It makes another FD referring to the same open file description.
5. **Wrong:** `lseek` reads skipped bytes.  
   **Correct:** It adjusts the offset; it does not read the skipped bytes.
6. **Wrong:** Indexed access and indexed allocation are the same.  
   **Correct:** One is a record-location strategy, the other is a block-allocation strategy.
7. **Wrong:** Fixed-size record number 1 begins at one full record size from the beginning.  
   **Correct:** Depends on numbering: in 1-based numbering, record 1 starts at offset 0.
8. **Wrong:** Every index lookup costs exactly one index read plus one data read.  
   **Correct:** Cost depends on levels, layout, caching and device details.

## 10. Practice: attempt BEFORE looking up explanations

**Q1.** A regular file has 250-byte fixed-size records numbered from 0. What is the starting byte offset of record 36?  
A. 8,750  B. 9,000  C. 9,250  D. 9,500

**Q2.** Which statement about sequential access is correct?  
A. It requires contiguous file allocation.  
B. It describes an access order independent of physical contiguity.  
C. It implies data is loaded entirely into RAM.  
D. It is impossible on SSDs.

**Q3.** A regular file's current offset is 100. `lseek(fd, -40, SEEK_CUR)` succeeds. The next `read()` returns 15 bytes. What is the resulting offset?  
A. 45  B. 60  C. 75  D. 115

**Q4.** You know the customer ID, but do not know its position in a large unsorted record file. Which strategy is most directly designed for key-based lookup?  
A. Seek to an arbitrary byte position  
B. Sequential rewind  
C. An index mapping customer IDs to file positions  
D. Changing the file's inode number

**Q5.** `fd3 = dup(fd1);` Which is correct?  
A. Both FDs refer to different inodes.  
B. Each FD has an independent open file description.  
C. Both FDs refer to the same open file description and share an offset.  
D. Both FDs must have the same integer value.

**Q6.** Which operation changes the current offset without reading skipped bytes?  
A. `lseek`  B. `read`  C. `close`  D. `unlink`

**Q7.** Which statement most accurately distinguishes direct and indexed access?  
A. Direct uses key search; indexed can only scan.  
B. Direct uses a known position; indexed can map a search key to a position.  
C. Both require magnetic tape.  
D. Both necessarily store file blocks contiguously.

**Q8.** A successful positional `pread(fd, buffer, 64, 4096)` on a regular file does what to the open file description's *current* offset?  
A. Sets it to 4096  
B. Sets it to 4160  
C. Leaves it unchanged  
D. Resets it to 0

**Q9.** A file has variable-length records. What is the major problem with jumping directly to record number 100 using `100 * record_size`?  
A. There is no uniform record size to multiply by.  
B. Files are always stored in one block.  
C. The inode is missing.  
D. Direct reads are forbidden on SSDs.

**Q10.** Two processes independently call `open()` on the same file. Each receives FD 6 in its own process. Which statement is correct?  
A. An FD number is globally unique.  
B. Different processes may use the same FD integer with independent open file descriptions.  
C. They necessarily share one offset.  
D. One open must fail.

*Answer the questions in chat for grading and discussion of traps. Answers intentionally omitted from this self-test section.*

---

## 11. Next lessons

**Day 3:** Directory structure, absolute/relative paths, hard links, symbolic links.  
**Day 4:** Contiguous, linked, indexed allocation; fragmentation and disk I/O tradeoffs.  
**Day 5:** Free-space management.  
**Day 6:** Multi-level inode pointers, maximum file size and GATE numericals.  
**Day 7:** Permissions, mounting, consistency, journaling and crash recovery.

**One-sentence recall:** Sequential = read in order; Direct = jump to a known position; Indexed = use a key lookup structure to find the position.
