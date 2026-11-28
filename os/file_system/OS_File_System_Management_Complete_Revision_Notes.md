# Operating Systems — File System Management
## Complete Revision Notes | IBPS RRB Scale II SO (IT Officer)

**Scope:** Consolidated notes for the file-system topics we studied: fundamentals, file access, file descriptors, allocation, inode addressing, directories and links, free-space management, crash consistency and journaling, Linux file protection, ACLs, and practical commands.

**Level:** Concepts + medium/upper-medium questions, with the essential numericals. No GATE-only mathematical detours.  
**Conventions:** Unless a question says otherwise, use **1 KiB = 1024 bytes**, **1 MiB = 1024 KiB**, zero-based file-block/byte offsets, and conventional Unix/Linux semantics. Implementation details can vary across file systems.

---

## Contents

1. [File-system fundamentals](#1-file-system-fundamentals)
2. [File metadata, inodes, and path resolution](#2-file-metadata-inodes-and-path-resolution)
3. [File descriptors and open-file offsets](#3-file-descriptors-and-open-file-offsets)
4. [File access methods](#4-file-access-methods)
5. [File allocation methods](#5-file-allocation-methods)
6. [Inode direct/indirect addressing and numericals](#6-inode-directindirect-addressing-and-numericals)
7. [Directory structures, hard links and symbolic links](#7-directory-structures-hard-links-and-symbolic-links)
8. [Free-space management](#8-free-space-management)
9. [Consistency, journaling and crash recovery](#9-consistency-journaling-and-crash-recovery)
10. [Page cache, buffering and write durability](#10-page-cache-buffering-and-write-durability)
11. [File protection: Unix permissions and chmod](#11-file-protection-unix-permissions-and-chmod)
12. [Directory permissions and deletion](#12-directory-permissions-and-deletion)
13. [umask](#13-umask)
14. [POSIX Access Control Lists (ACLs)](#14-posix-access-control-lists-acls)
15. [SetUID, SetGID and Sticky Bit](#15-setuid-setgid-and-sticky-bit)
16. [Practical Linux lab](#16-practical-linux-lab)
17. [Exam traps, quick formulas and final checklist](#17-exam-traps-quick-formulas-and-final-checklist)

---

## 1. File-system fundamentals

### 1.1 What problem does a file system solve?

An HDD or SSD provides persistent storage. A **file system** is the software/data-structure layer used by an operating system to give meaningful organization to that storage: files, directories, names, permissions, metadata, allocation maps, and free space.

Without a file system, storage would not automatically provide user-friendly names such as `/home/sachin/report.txt`, a directory hierarchy, inode metadata, or a mechanism to track which physical storage areas are occupied.

Core responsibilities:

- Map **file names** to file objects.
- Map a file's **logical byte/block positions** to allocated storage blocks/extents.
- Store metadata: owner, group, permissions, size, timestamps, link count, block references.
- Allocate and release storage space.
- Control access, organize directories, and recover from interruptions.

A **logical file** is a named sequence of bytes. The bytes may be distributed among different storage blocks. Logical continuity does not imply physical continuity.

### 1.2 Blocks, bytes, and offsets

A storage allocation block is a unit in which a file system manages space. Typical textbook problems assume a block is 1 KiB, 2 KiB, 4 KiB, etc. Real systems can distinguish device logical sectors, file-system blocks, and memory pages: they are not automatically the same concept.

For a block size `B` bytes and a **zero-based** file byte offset `x`:

```text
Logical file block number = floor(x / B)
Offset within that block  = x mod B
```

**Worked example:** `B = 1024 bytes`, file byte offset `x = 4300`:

```text
4300 = 4 × 1024 + 204
Logical block = 4
Within-block byte offset = 204
```

The file system must still translate *logical file block 4* to whatever storage block or extent actually holds it.

### 1.3 Metadata versus data

- **Data:** Actual contents, e.g. bytes corresponding to `sachin\n`.
- **Metadata:** Facts about a file and its storage, such as its length, ownership, timestamp, allocated blocks, and inode's block mapping.
- **Directory metadata/data:** Information that links names to inodes. In Unix-style directories, directory entries are stored in directory data blocks, but those blocks are often treated as **journaled metadata** because they affect the file-system structure.

**Exam trap:** Inode is **not** a file's contents; its block references help locate the contents.

---

## 2. File metadata, inodes, and path resolution

### 2.1 What is an inode?

In a traditional Unix-style file system, an **inode** is a persistent structure representing a file-system object. Typical fields include:

- File type and permission bits.
- Owner UID and group GID.
- File size.
- Access, modification and metadata-change timestamps.
- Hard-link count.
- Pointers/extents or other information used to locate file data.

The ordinary file **name is not stored as the filename field inside its inode**. A parent directory normally stores a mapping:

```text
Directory entry:  report.txt → inode 169
Inode 169:        owner, permissions, size, block mapping, ...
```

The same inode can be referenced by multiple directory entries (hard links).

### 2.2 Pathname resolution

To open `/home/sachin/report.txt`, the kernel resolves each path component using directory information, checks traversal/search permission at the relevant points, locates the final inode, evaluates permissions, and establishes an open-file description for the process.

Do not confuse **pathname** (a way to find an inode) with **file descriptor** (a process-local handle to an open file).

### 2.3 Data blocks versus index blocks

- **Data block:** Contains file bytes.
- **Index/indirect block:** Contains addresses of other blocks.
- **Inode block pointer:** A stored address/reference; its numeric size (e.g., 4 bytes) is **not** the size of the data block it references.

A 4-byte pointer can reference a complete 1 KiB data block. In an indirect block, many 4-byte pointers fit in one 1 KiB index block.

---

## 3. File descriptors and open-file offsets

### 3.1 What is a file descriptor?

A file descriptor (FD) is a small nonnegative integer held in a **process-local FD table**. It refers to a kernel **open-file description**, which tracks an open file's status flags and current file offset. That kernel structure in turn refers to a file-system object/inode.

Common conventional descriptors:

```text
0 = standard input
1 = standard output
2 = standard error
```

FD `3` in process A has no necessary relationship to FD `3` in process B.

### 3.2 Independent `open()` calls have independent offsets

```c
int fd1 = open("report.txt", O_RDONLY);
int fd2 = open("report.txt", O_RDONLY);
read(fd1, buffer, 100);
```

Assuming both started at position zero and nothing else changed them:

```text
fd1 current offset = 100
fd2 current offset = 0
```

**Why?** Two independent opens create distinct open-file descriptions with their own offsets.

### 3.3 `dup()` and inherited open descriptions can share offsets

```c
int fd1 = open("report.txt", O_RDONLY);
int fd2 = dup(fd1);
read(fd1, buffer, 100);
```

Here `fd1` and `fd2` refer to the **same open-file description**. Both observe the common offset, now 100 (assuming a successful 100-byte read).

After `fork()`, the parent's and child's inherited FDs also normally share the underlying open-file description and its offset, unless separately reopened.

### 3.4 `lseek()` and `close()`

- `lseek(fd, offset, SEEK_SET)` changes the open-file description's offset (on seekable objects).
- `read()` and `write()` normally advance the current offset by the bytes transferred.
- `close()` releases the process's descriptor. It **does not automatically guarantee crash durability** of recently buffered writes.

**Exam traps:** Two FDs ≠ necessarily two independent offsets; two different opens ≠ the same offset; `dup()` shares an open-file description.

---

## 4. File access methods

**Access methods** describe *how software accesses records/bytes*. Do not confuse them with **allocation methods**, which describe *how disk blocks are assigned to a file*.

### 4.1 Sequential access

Data is processed in order, from the current position onward.

Examples: Reading a log from beginning to end, streaming a text file, scanning a dataset.

Pros: Simple, good for full scans.  
Cons: To reach an arbitrary record, sequential scanning may be costly if no other index/seek mechanism is used.

### 4.2 Direct (random) access

Read/write at a selected offset, without first scanning every preceding byte.

Examples: `lseek()` followed by `read()`; fixed-size records with addressable positions.

For fixed-size records of `R` bytes and zero-based record number `n`:

```text
Byte offset of record n = n × R
```

Example: record 25 (zero-based), 128 bytes per record → byte offset `25 × 128 = 3200`.

### 4.3 Indexed access

An application-level index maps a search key to a location of data or record(s).

Examples: Database B-tree/B+tree index, indexed sequential access in classic operating-systems textbooks.

- Useful for key-based lookup.
- Requires index storage and maintenance.
- **Not the same as indexed disk-block allocation** using an indirect block or index block.

| Term | Meaning |
|---|---|
| Indexed **access** | Search key → record/data location |
| Indexed **allocation** | File logical block → storage block address |
| Inode indirect **index block** | Array of pointers forming part of block-address translation |

---

## 5. File allocation methods

### 5.1 Contiguous allocation

A file occupies consecutive blocks. Traditional representation: starting block and length.

If a file starts at physical block 100 and occupies five blocks, its blocks are `100, 101, 102, 103, 104`.

Advantages:

- Simple addressing: physical block `start + logical_block_index`.
- Fast sequential and random access, good locality.

Disadvantages:

- **External fragmentation**: sufficient total free space may exist without a sufficiently large contiguous run.
- Expansion can require relocation or allocating additional extents, depending on implementation.

Modern extent-based schemes mitigate some limitations without demanding that the entire file occupy one unbroken run.

### 5.2 Linked allocation

Each allocated data block contains (or is associated with) a pointer to the next allocated block.

Example: `51 → 83 → 14 → 90`.

Advantages:

- No requirement for physically consecutive blocks.
- Easy to grow a file block-by-block.

Disadvantages:

- Random access is inefficient when you must follow a pointer chain.
- Pointer overhead and vulnerability to damaged links.
- Data locality may be poor.

**FAT variation:** File Allocation Table stores the chain's next-block references in a central table rather than inside each data block. The access behavior depends on whether relevant FAT entries are cached.

### 5.3 Indexed allocation

An index structure contains pointers to the file's allocated blocks. The data blocks do not have to be contiguous.

Advantages:

- Direct access to logical file blocks via indexed pointers.
- File growth does not require one contiguous run.

Disadvantages:

- Additional space for index structures.
- Multiple pointer traversals for large files using multilevel indexing.

### 5.4 Comparison

| Property | Contiguous | Linked | Indexed |
|---|---|---|---|
| Need adjacent blocks | Traditionally yes | No | No |
| Direct access | Excellent | Poor without auxiliary index | Good |
| Sequential access | Excellent | Good logically, locality varies | Good logically, locality varies |
| Main overhead | Fragmentation/relocation | Link pointers | Index blocks |
| Growth | Can be difficult | Straightforward | Requires index expansion |

**Distinguish fragmentation:** External fragmentation concerns separated free extents; internal fragmentation concerns unused bytes within allocated units (e.g., last partially filled block).

---

## 6. Inode direct/indirect addressing and numericals

This was one of our most important numerical sections.

### 6.1 Types of pointers

**Direct pointer:** Points from inode directly to **one data block**.

**Single-indirect pointer:** Points to an index block containing pointers to data blocks.

**Double-indirect pointer:** Points to a first-level index block; its entries point to second-level index blocks; their entries point to data blocks.

**Triple-indirect pointer:** Adds one more index level.

A **pointer's size** (say 4 B) is the space used to store an address, not the amount of file data referenced.

### 6.2 The central formula

If:

```text
File-system block size = B bytes
Pointer/address size   = P bytes
Pointers per index block = N = B / P
```

Then one pointer provides capacity for:

```text
One direct pointer           = 1 data block
One single-indirect pointer  = N data blocks
One double-indirect pointer  = N² data blocks
One triple-indirect pointer  = N³ data blocks
```

For `D` direct pointers and one pointer of each indirect type:

```text
Maximum logical data blocks = D + N + N² + N³
Maximum byte capacity       = (D + N + N² + N³) × B
```

This is a **classic simplified inode model**, not a claim that every modern file system uses exactly this layout.

### 6.3 Worked standard example

Given:

```text
Block size         = 1 KiB = 1024 bytes
Pointer size       = 4 bytes
Number of direct pointers = 3
One single-indirect and one double-indirect pointer
```

Calculate pointers per index block:

```text
N = 1024 / 4 = 256 pointers
```

Address ranges for **zero-based logical file block numbers**:

| Region | Block count | Logical file block numbers |
|---|---:|---|
| 3 direct entries | 3 | 0–2 |
| Single-indirect | 256 | 3–258 |
| Double-indirect | 256² = 65,536 | 259–65,794 |

**Question:** Where does the double-indirect region begin?  
**Answer:** `3 + 256 = logical file block 259`.

**Important:** The first double-indirect logical block is **259**, not 256 or 260. These ranges assume zero-based numbering.

### 6.4 Locate a block inside double-indirect addressing

Suppose logical file block = `516`.

1. The double-indirect region starts at logical block 259.
2. Relative position within it = `516 - 259 = 257`.
3. Each second-level index block addresses 256 data blocks.
4. First-level index entry = `floor(257 / 256) = 1`.
5. Second-level entry = `257 mod 256 = 1`.

Thus **indices `(1, 1)`**, with both index arrays zero-based.

General double-indirect formula:

```text
relative = logical_file_block - (D + N)
first-level index  = floor(relative / N)
second-level index = relative mod N
```

### 6.5 Why does double indirect give N², not 2N?

Each of the **N pointers** in the first-level index block leads to an entire second-level index block. Each second-level block has **N pointers to data**. Therefore `N × N = N²`.

With `N = 256`, double indirect handles `256 × 256 = 65,536 data blocks`.

Triple indirect similarly handles `256³ = 16,777,216 data blocks` per triple-indirect pointer.

### 6.6 Index block overhead example

Suppose the double-indirect region uses **three complete second-level index blocks** with `N = 256`:

```text
Data blocks addressed  = 3 × 256 = 768
Data capacity          = 768 KiB (if each data block is 1 KiB)
Index blocks required  = 1 first-level + 3 second-level = 4
```

The 4 index blocks are **overhead**; they do not themselves store those 768 KiB of file data.

### 6.7 Limits of the simplified model

Actual file systems may use extents, sparse files, delayed allocation, different addressing structures, and metadata placement. For exam questions, **follow the specified model**. Do not automatically assume every file starts with 12 direct pointers or that modern ext4 necessarily maps ordinary files through classic direct/single/double pointers.

### 6.8 Essential numerical traps

- 1 KiB = 1024 **bytes**, not 1024 bits.
- 1 byte = 8 bits.
- 1 GiB = 1024 MiB = 1024³ bytes.
- Use `floor()` for the logical block and `%` for displacement.
- Know whether record/block numbering starts from 0 or 1.
- Include all direct-block coverage before evaluating indirect ranges.
- Don't confuse the number of **index blocks** with the number of **data blocks**.
- A 4-byte pointer can address a full block; it does not store 4 bytes of file data.

---

## 7. Directory structures, hard links and symbolic links

### 7.1 Directory structures

**Single-level directory:** One shared namespace; simple but name conflicts and poor organization.

**Two-level directory:** Separate directory for each user; reduces name conflicts but limited organization/sharing.

**Tree-structured directory:** Hierarchical folders and paths; common model, supports subdirectories and absolute/relative paths.

**Acyclic graph directory:** Allows sharing through links without cycles, under the particular textbook abstraction.

**General graph directory:** Allows cycles, requiring extra care in traversal, deletion, garbage collection, or cycle detection. Real implementations differ in what links they permit.

### 7.2 Absolute and relative paths

- Absolute: `/home/sachin/report.txt`.
- Relative: `projects/report.txt` interpreted from current working directory.
- `.` refers to current directory; `..` to parent-directory traversal semantics.

### 7.3 Hard links

```bash
printf 'Hello\n' > original.txt
ln original.txt hard.txt
ls -li original.txt hard.txt
```

Two directory entries now point to the **same inode** (normally within the same file system):

```text
original.txt → inode 169
hard.txt     → inode 169
```

Consequences:

- Same underlying file content and metadata.
- Modifying data through one link changes the data seen through the other.
- Removing one filename (`rm original.txt`) does not remove the other.
- The inode's hard-link count tracks directory references.
- Storage is normally reclaimable when the **last hard link is removed and no open references remain** (aside from implementation-specific situations).
- Ordinary users generally cannot hard-link directories; hard links cannot cross distinct file systems.

A hard link is not a duplicate copy of the contents.

### 7.4 Symbolic (soft) links

```bash
ln -s original.txt soft.txt
ls -li original.txt soft.txt
```

A symbolic link is a **separate file-system object/inode** whose contents encode a target pathname. It can point across file systems and to directories.

If the target is deleted, the symlink usually remains but becomes **dangling**. Creating/removing the symlink does not generally increment/decrement the target's hard-link count.

### 7.5 Hard link vs symlink

| Feature | Hard link | Symbolic link |
|---|---|---|
| Inode | Same as target | Different inode |
| Stores path to target | No | Yes |
| Survives deletion of other name | Yes | May dangle |
| Cross-file-system target | No | Yes |
| Typical directory linking | Restricted | Allowed |
| Example command | `ln source hard` | `ln -s source soft` |

### 7.6 Why a filename can be deleted without write access to file contents

Deleting `a.txt` ordinarily removes **a directory entry** from its parent directory. It is fundamentally a directory mutation. The file's own `w` permission isn't the controlling permission for unlinking its name; parent directory write/search permissions and special restrictions matter. See §12.

---

## 8. Free-space management

The file system needs to know which allocation blocks are free.

### 8.1 Bitmap (bit vector)

Typically **one bit per allocation block** records whether it is free/used (bit convention can be either 0=free or 1=free; use the problem's convention).

```text
Bitmap size in bits  = total number of allocation blocks
Bitmap size in bytes = ceil(number of blocks / 8)
```

**Example 1:** 16,384 blocks:

```text
16,384 bits / 8 = 2,048 bytes = 2 KiB
```

**Example 2:** 65,536 blocks:

```text
65,536 bits / 8 = 8,192 bytes = 8 KiB
```

Advantages: Compact and easy to scan for free runs.  
Tradeoffs: May require scanning portions of bitmap; tracking/modifying metadata must be crash-safe.

**Exam trap:** 65,536 **bits** is **8 KiB**, not 64 KiB.

### 8.2 Linked list of free blocks

A free block or separate metadata reference indicates the next free block. The file system manages a chain of available blocks.

Advantage: Simple concept; can take a free block from the chain.  
Disadvantage: Finding a large consecutive free run may be inefficient.

Do not confuse **free-list linking** with **linked file allocation**. One tracks free blocks; the other tracks blocks belonging to a file.

### 8.3 Grouping

A free block stores **addresses of several independent free blocks**. Another entry/structure may lead to the next group of free addresses. It reduces the need to traverse one pointer per free block.

The listed blocks **do not have to be physically consecutive**.

### 8.4 Counting

Store the **starting free block** and a **count of consecutive free blocks**.

Example:

```text
(start = 800, count = 5)
Free blocks = 800, 801, 802, 803, 804
```

Counting compresses information well when long contiguous free runs exist.

### 8.5 Grouping vs counting — frequent confusion

| Grouping | Counting |
|---|---|
| Stores a collection of free block **addresses** | Stores **start block + length** of a consecutive run |
| Blocks need not be adjacent | Describes contiguous blocks |
| Example: 7, 20, 51, 89 | Example: start 20, count 4 → 20–23 |

---

## 9. Consistency, journaling and crash recovery

### 9.1 Why consistency is difficult

Creating or extending a file can modify multiple metadata structures:

- Inode allocation bitmap.
- Data-block allocation bitmap.
- Inode fields (size, mappings, timestamps, link count).
- Parent directory data (filename → inode number).
- Actual file-data blocks.

These updates are not necessarily written to persistent storage simultaneously. A crash between writes may leave contradictory metadata.

Examples:

- Bitmap says a block is **allocated**, but no inode refers to it → potential leaked space.
- Inode points to a block marked **free** → dangerous inconsistent allocation state.
- A directory entry points to an inode with inconsistent link accounting.

### 9.2 What is journaling?

Journaling uses a persistent **journal area** to record file-system updates so a valid committed transaction can be recovered after interruption.

In traditional redo-style **metadata journaling**, the journal contains **updated metadata block images**, plus information identifying their destination locations and the transaction boundaries. It is not necessarily a human-readable sequence like `allocate inode 169`.

The journal is stored on persistent storage. While records can be buffered in RAM temporarily, reliable recovery requires them to reach the journal device as specified by the journaling protocol.

Linux ext4 uses **JBD2** for journaling. In standard configurations it may use a reserved journal inode (commonly inode 8), or an external journal device.

### 9.3 Our central example: `echo "sachin" > a.txt`

Assume `a.txt` did not exist and the file system needs to:

1. Reserve inode 169.
2. Reserve data block 500.
3. Update allocation bitmaps.
4. Set inode 169's metadata and block mapping.
5. Insert directory entry `a.txt → inode 169`.
6. Write file bytes `sachin\n` using its data-writing policy.

These are **conceptual operations**. The real operation can span multiple journal transactions, and delayed allocation may change the precise timing.

### 9.4 Three distinct milestones

**A. Journal records written:** Required metadata block images have been persisted in the journal. Merely finding these records is **not** the same as a committed transaction.

**B. Journal commit:** After the required journal records are safely recorded, the journal's **valid commit record** establishes that the transaction is complete and eligible for replay. A crash before that valid commit means normal recovery **does not replay it as a committed transaction**.

**C. Checkpoint:** Committed metadata changes reach their **normal file-system locations** (inode table, bitmaps, directory blocks). During normal operation the system may write cached modified metadata blocks there; it does not need to reread the journal in order to re-execute the application command.

When a transaction's updates are safely checkpointed, the corresponding journal space can eventually be reused.

**Important:** A file-system journal transaction is **not** an application-wide atomic transaction of an entire shell command, and `write()` completion does not automatically equal durable journal commit.

### 9.5 Commit versus checkpoint

| Journal commit | Checkpoint |
|---|---|
| Transaction's recoverable record is durable | Transaction's metadata has reached normal storage locations |
| The final inode/bitmap/directory locations may still be old | Final locations now reflect committed updates |
| Recovery can redo valid committed journal updates | Corresponding journal records are no longer needed for that recovery |
| Not the same as app write durability | Not a substitute for app-level `fsync()` semantics |

### 9.6 Crash scenarios

**Crash before the journal commit record:** Journal may contain some or all metadata records, but without a **valid commit**, redo recovery does **not** treat this transaction as committed. Proper write-ordering/journaling rules prevent unsafe uncommitted modifications from being exposed as a complete committed transaction.

**Crash after journal commit, before checkpoint:** Journal survives on SSD/HDD. The next mount/recovery can read and replay the committed metadata updates to their destinations.

**Crash halfway through checkpoint:** Some normal locations contain new metadata, others old. Journal still contains committed transaction; recovery can safely reapply its updates.

**Crash during replay:** A block-image redo journal is designed so committed metadata images may be replayed again; repeating an assignment of the same final block contents is idempotent. Committed journal information must remain protected until safe to reclaim.

**Crash after successful checkpoint:** Relevant normal metadata locations already contain the committed updates; that transaction ordinarily needs no additional replay.

### 9.7 How recovery works when RAM is lost

The critical point from our doubts:

- The **old RAM is gone** after power loss.
- The **persistent journal remains** on storage.
- The rebooted system can locate the journal through persistent file-system structures, read it into **new RAM**, recognize valid committed transactions, and restore modified metadata blocks to their recorded destination addresses.
- The system does **not** need to rerun the original `echo`, `touch`, or application command.

Illustration of what an entry conceptually provides:

```text
Transaction T100 (committed)
Journal record 1: destination FS block 10 + updated bitmap contents
Journal record 2: destination FS block 20 + updated inode metadata block
Journal record 3: destination FS block 40 + updated directory block
```

The destination addresses above are invented for the illustration; real journal records use the file system's on-disk format.

### 9.8 Replay versus checkpoint

- **Checkpoint:** Normal operation; flush already committed updated metadata to final locations, often using cached buffers.
- **Replay:** Recovery after interruption; read committed journal records from persistent storage and redo the necessary writes.

Both can update the same final metadata blocks. They happen for different reasons.

### 9.9 Three important ext4 data modes

| Mode | Journaled contents | Data ordering |
|---|---|---|
| `data=journal` | File data **and** metadata | File data goes through journal |
| `data=ordered` | Metadata (traditional default mode) | Associated file data is written to normal locations before relevant metadata journal commit |
| `data=writeback` | Metadata | No corresponding data-before-metadata ordering guarantee |

**Crucial qualification:** `data=ordered` is not a promise that every successful application `write()` is durable. It defines an ordering constraint around a committing transaction. Actual file-data and directory-name durability require additional care.

### 9.10 `fsck()` versus journal recovery

**Journal recovery** aims to replay valid committed transactions quickly and repair transaction-interruption consistency issues.

**`fsck` (file-system check)** scans structures to find and sometimes repair broader inconsistencies, potentially including corrupted metadata, link-count mismatches, and orphaned references. It can be more expensive; it cannot magically reconstruct any missing original data. Some file systems and configurations use different check/repair utilities.

**Exam distinction:** Journaling reduces the work needed for crash consistency; it is **not** a backup, and does not eliminate every possible corruption scenario.

---

## 10. Page cache, buffering and write durability

### 10.1 Where does `write(fd, "HELLO", 5)` write?

For a normal **buffered** Linux file write:

1. The program supplies five bytes from its **user-space buffer** (stored in process-accessible virtual memory, backed by RAM as needed).
2. Kernel copies or otherwise transfers the content into kernel-managed **page cache**, where the file's cached pages/folios become **dirty**.
3. `write()` may return **before** the data reaches persistent storage.
4. Later writeback sends dirty file data to the underlying device, subject to I/O, file-system and device ordering.

This simplified description does **not** cover direct I/O, memory-mapped writing, or every special file.

**A process's virtual pages are not scheduled CPU time.** CPU scheduling chooses runnable tasks/threads; paging maps virtual memory to physical memory and storage-backed pages.

### 10.2 Different things called a cache or buffer

| Thing | Typically located | Purpose |
|---|---|---|
| Application buffer | User-space virtual memory backed by RAM | Bytes application passes to `write()` |
| Linux page cache | Kernel-managed RAM | Cached file contents; dirty pages await writeback |
| CPU L1/L2/L3 cache | Processor hardware | Caches recently accessed memory for CPU execution |
| SSD/controller write cache | Device/controller hardware | May buffer device writes; durability depends on device features and flush behavior |
| Journal | Persistent file-system storage area | Records recoverable file-system updates |

A successful `write()` is not proof that bytes are power-loss safe.

### 10.3 `fsync()` and `fdatasync()`

```c
int fd = open("report.txt", O_WRONLY);
write(fd, "HELLO", 5);
fsync(fd);
close(fd);
```

- `write()`: Requests the write; normal buffered writes can remain in RAM.
- `fsync(fd)`: Requests synchronization of the file's changed data **and relevant metadata** to storage; durability depends on successful return and underlying storage honoring flush/order guarantees.
- `fdatasync(fd)`: Synchronizes file data and metadata needed to retrieve the data, but may omit unrelated metadata updates (e.g. some timestamps).
- `close(fd)`: Releases FD, not a substitute for a successful `fsync()`.

For **newly created files or renamed names**, synchronizing the regular file alone may not make the **containing directory entry** crash-durable. Synchronizing the parent directory may also be needed on Linux.

### 10.4 Process exit does not immediately delete dirty page-cache contents

A page cache is kernel-managed and not just a private process buffer. If the writing process exits successfully after `write()` without `fsync()`, dirty cached file data is **not automatically erased simply because the process ended**. The kernel can continue writeback. But a subsequent power failure may lose any not-yet-durable updates.

### 10.5 Durability versus consistency

- **Consistency:** File-system metadata structures are mutually valid enough to mount/use correctly after a crash.
- **Durability:** Specific application data/changes survive power failure.
- **Journaling:** Primarily provides recoverable file-system transactions; it is **not** automatically equivalent to a database transaction spanning arbitrary application operations.

---

## 11. File protection: Unix permissions and chmod

### 11.1 Authentication versus authorization

- **Authentication:** Who is the user/process?
- **Authorization:** What action is that identity allowed to perform on this object?

Unix file mode bits provide one of several authorization mechanisms.

### 11.2 Owner, group, others

A file can have:

```text
-rwxr-x--- 1 sachin developers ... app
```

Interpret as:

```text
-    rwx    r-x    ---
     owner  group  others
```

First character: `-` regular file, `d` directory, `l` symlink in typical `ls -l` display.

| Class | File mode field | Meaning |
|---|---|---|
| Owner (`u`) | First `rwx` trio | File's owning user |
| Group (`g`) | Second trio | File's owning group, according to matching rules |
| Others (`o`) | Third trio | Remaining users |

`r` = read, `w` = write, `x` = execute **for a regular executable**.

**Don't add all three classes together.** The system selects/evaluates applicable permissions based on the user's credentials and any ACLs. The file owner is not automatically allowed to use 'other' permissions as fallback if owner permissions deny something.

### 11.3 Octal permission values

| Bit | Value |
|---|---:|
| Read (`r`) | 4 |
| Write (`w`) | 2 |
| Execute (`x`) | 1 |

Examples:

```text
7 = rwx = 4+2+1
6 = rw- = 4+2
5 = r-x = 4+1
4 = r-- = 4
0 = ---
```

`chmod 750 app` → `rwxr-x---`.  
`chmod 740 app` → `rwxr-----` (**group has read, NOT write**).  
`chmod 644 report.txt` → `rw-r--r--`.  
`chmod 600 secrets.txt` → `rw-------`.

Use symbolic forms too:

```bash
chmod u+x app              # add execute for owner
chmod g-w report.txt       # remove group-class write bit
chmod o= report.txt        # clear all others permissions
chmod 750 app              # set traditional permission mode
```

Note: On files with an extended POSIX ACL, changing group-class mode bits can modify the **ACL mask**, thereby changing named users' effective permissions. See §14.

### 11.4 Changing ownership and group

```bash
ls -l report.txt
sudo chown sachin report.txt
sudo chgrp developers report.txt
sudo chown sachin:developers report.txt
```

Normal users cannot arbitrarily transfer ownership; applicable privileges and file-system rules matter.

---

## 12. Directory permissions and deletion

### 12.1 Directory `r`, `w`, `x` meanings

| Permission | File | Directory |
|---|---|---|
| `r` | Read bytes | Read/list names of directory entries |
| `w` | Modify bytes | Modify entries (create, remove, rename), normally with `x` |
| `x` | Execute as program | **Search/traverse** the directory |

A directory's `x` does not mean execute a program called the directory.

### 12.2 Known-name access without directory read

A user can sometimes access `dir/known.txt` **without permission to list the directory**, provided they have search (`x`) permission on the path components and access rights on the file itself.

Conversely, having an ACL read entry on a file is insufficient if an ancestor directory denies the user search/traverse permission for the pathname.

### 12.3 Why `chmod 444 file.txt` doesn't prevent deletion

Deleting a name normally modifies the **parent directory**:

```text
Directory: file.txt → inode 169
```

`rm file.txt` removes the directory entry. The file mode can be `444` while deletion still succeeds if the user has parent-directory **write + execute** permission and no sticky bit/ACL/other restrictions block it.

The inode/data can remain alive through another hard link or open reference.

### 12.4 Permissions required for practical operations

| Operation | Typical required permissions |
|---|---|
| Read file by pathname | Search on ancestor directories + read on file |
| Write existing file | Search on ancestor directories + write on file |
| List names in directory | Read on directory (and usually search for richer metadata/path operations) |
| Create entry in directory | Write + search (`wx`) on directory |
| Remove/rename entry | Relevant parent directory `wx`, plus sticky-bit and other conditions |

---

## 13. umask

`umask` is a per-process attribute inherited by children. It **removes** specified permission bits from the mode requested when files/directories are created.

**Precise formula:**

```text
result = requested_mode & ~umask
```

This is **bitwise** masking, not ordinary octal subtraction.

Textbook/default examples usually assume:

```text
Regular file requested mode = 0666 (rw-rw-rw-)
Directory requested mode    = 0777 (rwxrwxrwx)
```

A program can request a more restrictive mode; a default ACL can also affect effective inherited permissions.

### 13.1 Example: umask 022

```text
File:      666 & ~022 = 644 = rw-r--r--
Directory: 777 & ~022 = 755 = rwxr-xr-x
```

### 13.2 Example: umask 027

```text
File:      666 & ~027 = 640 = rw-r-----
Directory: 777 & ~027 = 750 = rwxr-x---
```

### 13.3 Example: umask 003

```text
File:      666 & ~003 = 664 = rw-rw-r--
Directory: 777 & ~003 = 774 = rwxrwxr--
```

**Exam trap:** `666 - 003` as ordinary arithmetic would produce `663`, which is **wrong** here. Use bit masking.

Useful commands:

```bash
umask        # current mask (format may differ)
umask -S     # symbolic display
umask 027    # set in current shell session
```

`chmod` changes permissions of an **existing** object; `umask` influences the creation mode of a **new** object.

---

## 14. POSIX Access Control Lists (ACLs)

### 14.1 Why ACLs?

Traditional owner/group/others mode bits cannot express every per-user requirement. Suppose owner Sachin wants Rahul (same owning group as Nitesh) to read only, while **only Nitesh** can read/write. POSIX ACLs allow individually named user and group entries.

Sample:

```text
# owner: sachin
# group: developers
user::rw-
user:nitesh:rw-
group::r--
mask::rw-
other::---
```

### 14.2 Meanings of entries

| Entry | Meaning |
|---|---|
| `user::rw-` | Owner's access |
| `user:nitesh:rw-` | Named-user entry |
| `group::r--` | Owning group entry |
| `group:auditors:r-x` | Named-group entry |
| `mask::rw-` | Maximum effective rights for named users and group-class entries |
| `other::---` | Remaining identities |

An ACL entry does **not** change ownership or grant a user someone else's identity.

### 14.3 ACL mask and effective permissions

For a **named user** or **group-class entry**:

```text
Effective permissions = entry permissions AND ACL mask
```

Example:

```text
user:nitesh:rwx
mask::r-x
Effective:  r-x
```

The mask **does not restrict** the owning user's `user::` entry or the `other::` entry.

Example:

```text
user::rwx
user:nitesh:rwx
group::rw-
mask::r--
other::---
```

- Owner effective = `rwx`.
- Nitesh effective = `r--`.
- Owning group effective = `r--`.
- Others effective = `---`.

### 14.4 Which ACL entry applies first?

Simplified POSIX ACL evaluation:

1. If caller is **file owner**, use `user::` (not mask-limited).
2. Else if caller matches a **named user entry**, use that entry with mask.
3. Else evaluate applicable **owning/named group** entries with mask (there may be multiple matching groups).
4. Else use `other::`.

**High-value trap:** A matching named-user entry takes precedence over broader group permissions.

Example:

```text
user:nitesh:r--
group::rw-
mask::rw-
```

Even if Nitesh belongs to that group, his matching named-user entry gives **read only**; he cannot fall back to the group's write permission.

### 14.5 ACL mask and `chmod` interaction

Example initial ACL:

```text
user::rw-
user:nitesh:rw-
group::r--
mask::rw-
other::---
```

Then:

```bash
chmod g-w report.txt
```

For a normal extended POSIX ACL on Linux, the group-class mode bits correspond to the ACL **mask**. `chmod g-w` removes write from the mask, resulting in:

```text
user:nitesh:rw-  #effective:r--
mask::r--
```

**Nitesh's stored entry still says `rw-`, but effective rights become `r--`.**

### 14.6 Access ACL versus default ACL

- **Access ACL:** Used for permissions on an existing file/directory.
- **Default ACL:** Stored on a directory as an inheritance template for newly created children.
- Setting a default ACL does **not automatically modify preexisting files**.
- Inherited ACL and actual effective permissions are influenced by the child's requested mode and ACL rules.

Examples:

```bash
setfacl -m u:nitesh:rw- report.txt   # access ACL on existing file
getfacl report.txt                    # inspect ACL
setfacl -m m::r-- report.txt          # explicit restrictive mask
setfacl -d -m u:nitesh:rwX shared/   # default ACL for new children
```

Some file systems/environments require ACL support and the `acl` utilities to be installed. `setfacl` may automatically recalculate the ACL mask unless explicitly controlled.

### 14.7 What does `+` in `ls -l` mean?

Example:

```text
-rw-rw----+ 1 sachin developers report.txt
```

The `+` commonly signals extended ACL information. Use `getfacl` to see the precise entries and effective permissions. When an extended ACL mask is present, the displayed group-class mode bits describe **the mask**, not necessarily the owning group's exact permissions.

### 14.8 Directory traversal still applies

Even if `report.txt` explicitly grants Nitesh read access, he cannot open `/home/sachin/private/report.txt` by that pathname if he lacks search (`x`) permission on `/home/sachin/private` and has no alternative authorized path.

ACL on **the file** does not magically grant access through all **parent directories**.

---

## 15. SetUID, SetGID and Sticky Bit

These are **special mode bits** in addition to owner/group/others rwx permissions.

| Special bit | Octal leading value | Common use |
|---|---:|---|
| SetUID | 4 | Executable uses file owner's effective UID |
| SetGID | 2 | Executable uses file group as effective GID; directories provide group inheritance |
| Sticky bit | 1 | Restricts deletion/renaming in writable shared directories |

### 15.1 SetUID on executable

Suppose:

```text
Owner = sachin
Group = developers
Mode  = -rwsr-x--- (4750)
```

Nitesh is a member of `developers` and executes the program, assuming SetUID is honored.

```text
Real UID (RUID):      nitesh
Effective UID (EUID): sachin
```

**SetUID does not itself grant permission to execute**. Nitesh needs `x` via his applicable mode/ACL permissions first.

Compare:

```bash
chmod 740 app       # owner rwx, group r--, others ---
chmod u+s app       # add SetUID, does NOT grant group execute
# equivalent common single command for mode + SetUID:
chmod 4740 app
```

In mode `4740`, group = `4` = **read only**, so group member Nitesh still cannot execute it.

To allow group members to execute, assuming the group is the desired audience:

```bash
chmod 4750 app
```

To allow only a particular named user via ACL, you could combine a named-user execute ACL with SetUID, but applying SetUID to user-controlled programs can create serious security risks. Practice this concept theoretically; don't turn arbitrary programs into privileged executables.

**Security caveats:** SetUID is mainly applicable to suitable binary executables; Linux generally ignores SetUID on interpreted scripts. Restrictions such as `nosuid`, `no_new_privs` and other security controls may suppress privilege changes. SetUID changes effective identity of the **program**, not the entire login session or account.

### 15.2 SetGID on executable

With a suitable executable:

```text
Owner = sachin
Group = finance
Mode  = -rwxr-s--- (2750)
```

If Nitesh can execute it and SetGID is honored:

```text
Real UID:      nitesh
Effective UID: nitesh (unless also SetUID)
Effective GID: finance
```

Group membership can allow execution, but membership **does not itself cause the effective-GID switch**: the **SetGID bit** does.

### 15.3 SetGID on directory

Suppose:

```text
Directory = /tmp/team-practice
Owner     = sachin
Group     = developers
Mode      = drwxrws--- (2770)
```

Nitesh creates `test.txt` in it:

```text
New file owner = nitesh (usual creating-process filesystem UID)
New file group = developers (inherited from parent directory)
```

**Do not confuse these:** Directory SetGID affects the **new file's group**; executable SetGID affects the **running process's effective group identity**.

If the parent directory does not impose this inheritance, normal Linux behavior generally uses the creating process's filesystem group (normally effective GID), subject to filesystem/mount conventions.

SetGID on a directory does **not automatically give the group write permission** on every new file; creation mode, umask, default ACL and other checks still apply.

### 15.4 Sticky bit on directory

Consider `/tmp`-style shared directory:

```bash
chmod 1777 /tmp/example-shared
```

Mode typically displays `drwxrwxrwt`.

Although many users have directory write/search access, sticky bit normally restricts unlinking or renaming an entry to its file owner, directory owner, or sufficiently privileged process (subject to other restrictions).

**The file's own write bit does not control filename deletion**; sticky bit is an additional protection imposed at the directory level.

### 15.5 Special mode calculations

```text
4755 = SetUID + ordinary 755
2750 = SetGID + ordinary 750
1777 = Sticky + ordinary 777
6755 = SetUID + SetGID (4 + 2) + ordinary 755
```

Display convention:

- Lowercase `s` / `t` = relevant special bit present **and corresponding execute/search bit also set**.
- Uppercase `S` / `T` = special bit present **but corresponding execute/search bit absent**.

**Example:** Directory ends with `T` → sticky bit set; others' search (`x`) bit is absent.

### 15.6 SetUID vs ACL — essential distinction

- **chmod / ACL** answers: *Can this user execute this file?*
- **SetUID** answers: *If this is an allowed SetUID executable and the bit takes effect, which effective UID does its process run with?*
- **SetGID executable** similarly affects effective GID.
- **SetGID directory** determines group inheritance for newly created entries.

---

## 16. Practical Linux lab

Use a disposable Ubuntu/WSL VM or personal practice system. **Do not modify production accounts, root-owned executables, or real `/tmp` permissions for practice.** You may need `sudo` and the `acl` package (`sudo apt install acl`) for `setfacl`/`getfacl`.

### 16.1 Create users and a group

```bash
sudo adduser sachin
sudo adduser nitesh
sudo adduser rahul
sudo groupadd developers
sudo usermod -aG developers sachin
sudo usermod -aG developers nitesh

groups nitesh
id nitesh
getent group developers
```

`-aG` appends supplementary group membership; using only `-G` may replace other supplementary groups. Fresh login/session may be needed before changed membership is reflected in an existing session.

```bash
su - nitesh      # enter Nitesh's password
whoami
id
exit             # return to previous shell
```

### 16.2 SetGID shared directory demonstration

```bash
sudo mkdir -p /tmp/team-practice
sudo chown sachin:developers /tmp/team-practice
sudo chmod 2770 /tmp/team-practice
ls -ld /tmp/team-practice

sudo -u nitesh touch /tmp/team-practice/nitesh.txt
ls -l /tmp/team-practice/nitesh.txt
```

Expected ownership on a typical Linux setup:

```text
nitesh developers ... nitesh.txt
```

The `developers` group comes from the SetGID parent directory.

### 16.3 ACL mask demonstration

```bash
LAB=$(mktemp -d /tmp/acl-lab.XXXXXX)
chmod 755 "$LAB"
printf 'Study notes\n' > "$LAB/report.txt"
chmod 640 "$LAB/report.txt"

setfacl -m u:nitesh:rw- "$LAB/report.txt"
getfacl "$LAB/report.txt"
sudo -u nitesh cat "$LAB/report.txt"

setfacl -m m::r-- "$LAB/report.txt"
getfacl "$LAB/report.txt"
sudo -u nitesh sh -c 'echo new >> "$1"' sh "$LAB/report.txt"  # expected Permission denied
```

If you change ACL mask with `chmod g-w` after granting Nitesh `rw-`, `getfacl` can show `#effective:r--`.

### 16.4 Hard and symbolic links

```bash
mkdir -p /tmp/links-lab
cd /tmp/links-lab
printf 'hello\n' > original.txt
ln original.txt hard.txt
ln -s original.txt soft.txt
ls -li
rm original.txt
cat hard.txt   # still works
cat soft.txt   # dangling symlink; usually fails
```

### 16.5 Sticky-bit example in your own directory

```bash
sudo mkdir -p /tmp/sticky-lab
sudo chmod 1777 /tmp/sticky-lab
ls -ld /tmp/sticky-lab
sudo -u sachin touch /tmp/sticky-lab/sachin-file
sudo -u nitesh rm /tmp/sticky-lab/sachin-file   # normally denied
```

**Caution:** Always verify permissions and ownership first, and choose a separate lab directory. Never use `chmod -R 777 /` or similar destructive commands.

---

## 17. Exam traps, quick formulas and final checklist

### 17.1 One-page formula sheet

| Task | Formula |
|---|---|
| Logical file block for byte offset `x` | `floor(x / B)` |
| Within-block byte offset | `x mod B` |
| Pointers per index block | `N = B / P` |
| Data blocks under one single indirect | `N` |
| Data blocks under one double indirect | `N²` |
| Data blocks under one triple indirect | `N³` |
| Max data blocks, classic inode | `D + N + N² + N³` if one pointer of each level |
| Start of single indirect, 0-based | `D` |
| Start of double indirect, 0-based | `D + N` |
| Double-indirect first-level entry | `floor((L - (D + N)) / N)` |
| Double-indirect second-level entry | `(L - (D + N)) mod N` |
| Free bitmap in bytes | `ceil(total_blocks / 8)` |
| umask resulting mode | `requested_mode & ~umask` |
| ACL effective named-user rights | `ACL_entry & ACL_mask` |

### 17.2 Highest-value exam traps

1. **Inode versus directory:** A filename maps to an inode via a directory entry; inode holds file attributes and block information.
2. **FD versus inode:** FD is process-local; an inode is a file-system object; an open-file description tracks offset.
3. **Two `open()` calls versus `dup()`:** Independent opens have separate offsets; `dup()` shares an offset.
4. **Indexed access versus indexed allocation:** Search index for records is different from block-pointer indexing.
5. **Pointer size versus block size:** 4-byte pointer ≠ 4-byte data block.
6. **Direct vs indirect:** One direct pointer addresses one block; a single-indirect pointer addresses up to `B/P` data blocks via an index block.
7. **Zero-based indexing:** With 3 direct and N=256, double indirect starts at logical block **259**.
8. **Hard vs symlink:** Hard links share inode; symlinks have their own inode/path; symlinks can dangle.
9. **Grouping vs counting:** Arbitrary addresses versus contiguous `(start, length)` representation.
10. **Bitmap size:** One **bit** per allocation block, then divide by 8 for bytes.
11. **Journal record versus commit:** Data recorded without a valid commit record is not a committed redo transaction.
12. **Commit vs checkpoint:** Journal-durable recoverable transaction versus metadata persisted at normal destinations.
13. **Replay:** On reboot read committed journal records from SSD into **new RAM**; do not rerun original user command.
14. **Page cache vs SSD:** Dirty file pages are in RAM until writeback. `write()` returning is not a durability guarantee.
15. **`data=ordered`:** Metadata journaled, related file data ordered before relevant metadata commit; not a promise that every un-synced write survives.
16. **`fsync()` vs `close()`:** A successful `close()` is not equivalent to synchronization; newly created pathnames may also need directory sync.
17. **File vs directory x:** On directory, `x` means search/traverse, not run a program.
18. **Deleting read-only files:** Deleting a name normally requires parent-directory `wx`, not file's own `w`; sticky bit can restrict.
19. **umask:** Use bit masking. `666` with umask `003` gives **664**, not 663.
20. **ACL mask:** Applies to named users and group-class entries, **not** owner or others.
21. **Named user versus group:** Matching named-user ACL entry takes precedence, even if group rights are broader.
22. **`chmod g-w` with extended ACL:** Can lower the mask and named users' effective write privileges.
23. **SetUID versus execution authorization:** SetUID changes effective UID when honored; it does not grant `x` permission.
24. **SetGID executable versus directory:** Effective process GID versus inheritance of newly created file's group.
25. **Sticky bit:** `1777` restricts deletion/rename of others' entries in a shared writable directory.
26. **Modes:** `4750` = SetUID + owner rwx, group r-x, others none; `6755` = SetUID and SetGID + 755.

### 17.3 Ten short worked checks (revision, not your final mock)

**Check 1:** 1 KiB block; byte offset 4300 → logical file block **4**, within-block offset **204**.

**Check 2:** 1 KiB index block and 4-byte address → **256** addresses.

**Check 3:** 3 direct + 1 single (N=256): double indirect starts at logical block **259**.

**Check 4:** Double-indirect logical block 516 in the above setup → first/second indices **(1,1)**.

**Check 5:** 16,384 file-system allocation blocks → bitmap size **2 KiB**.

**Check 6:** `chmod 740` → owner `rwx`, group `r--`, others `---`.

**Check 7:** File requested `666`; `umask 027` → **640**; directory requested `777` → **750**.

**Check 8:** ACL `user:nitesh:rwx` with `mask::r--` → effective **`r--`**.

**Check 9:** `chmod 2770` on a group-owned directory → SetGID enabled; new children normally inherit the **directory's group**.

**Check 10:** Crash after journal commit but before finishing checkpoint → replay valid committed journal updates during recovery.

### 17.4 Final exam-readiness checklist

Before attempting the 15-question File System assessment, you should be able to explain *without looking*:

- [ ] What an inode stores versus what a directory entry stores.
- [ ] Why two independent opens have different offsets but duplicated FDs can share one.
- [ ] Sequential/direct/indexed **access** versus contiguous/linked/indexed **allocation**.
- [ ] Block-offset formula and one single/double-indirect numerical.
- [ ] Hard-link deletion and dangling symlink behavior.
- [ ] Bitmap, grouping, counting and their differences.
- [ ] Journal write, valid commit, checkpoint and replay.
- [ ] Why restarting loses RAM but journal recovery still works.
- [ ] Why successful `write()` is not equivalent to crash durability.
- [ ] File permissions versus directory search/write permissions.
- [ ] How `umask` and ACL masks actually remove effective bits.
- [ ] SetUID, executable SetGID, directory SetGID, and sticky bit.

**Recommended revision:** Read sections 2–3, 5–9 and 12–15 carefully, solve the worked inode numericals once on paper, then attempt the **15-question mixed assessment already shared in the chat** without referring back to this file.

---

*End of File System Management revision notes. The next planned OS chapter after assessment is CPU Scheduling: FCFS, SJF, SRTF, Priority and Round Robin.*
