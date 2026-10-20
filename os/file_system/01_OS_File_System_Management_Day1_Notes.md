# Operating Systems — File System Management
## Day 1: Foundations | IBPS RRB SO IT + GATE

> Style: Hinglish explanations; exam-ready definitions; examples, diagrams, traps, and practice. These notes cover **Day 1**, not the entire file-system syllabus.

## 1. Why do we need a file system?
A storage device exposes addressable storage, but users/applications need persistent named objects (`notes.txt`), directories, permissions and a way to find file contents. A **file system** is the software-defined organization and associated data structures/mechanisms used to name, store, locate, retrieve and manage files on a volume.

**Mental model:** Application → OS system calls / VFS → specific file-system implementation → block I/O layer → storage device.

```
Application: open("docs/notes.txt")
          |
       System call
          |
 OS: VFS / file-system implementation
    path lookup → permissions → metadata → data block mapping
          |
     Block device / SSD
```

**Trap:** Storage hardware and a file system are not the same thing. A volume may be unformatted or contain a file system such as ext4, NTFS, FAT32, APFS, etc. Their designs differ.

## 2. What is a file?
A **file** is a named collection of related information on nonvolatile storage. Applications interpret bytes according to their format; the OS usually handles byte streams and metadata (exact semantics depend on OS and file system).

### File contents vs metadata
- **Data:** actual contents of `notes.txt`, image, program, DB data.
- **Metadata:** file type, logical size, owner, permissions, timestamps, storage location mapping, identifiers.
- **Name-to-file mapping:** Unix-like directories map names to inode identifiers; the name is usually not kept inside the inode.

| Attribute | Purpose |
|---|---|
| Name | User-facing directory entry name |
| Identifier | File-system internal identity (e.g. inode number) |
| Type | Object kind (regular file, directory, etc.) |
| Size | Logical length in bytes |
| Location | Mapping to data blocks/extents |
| Permissions | Authorized operations |
| Owner | User/group identifiers |
| Timestamps | Access/modification/metadata-change and sometimes creation time |

**Exam traps:** (1) Metadata ≠ contents. (2) `ctime` on Unix is *inode status-change time*, not necessarily creation time. (3) Creation/birth time support differs among file systems.

## 3. Typical file operations
- `create`: create a new directory entry and underlying file object.
- `open`: resolve path, check access, obtain a handle/descriptor.
- `read`: transfer data from file at a chosen offset.
- `write`: modify file data, possibly extending length.
- `append`: write at end of file (specific atomicity depends on OS and file type).
- `seek`: change current offset (not a physical head movement guarantee).
- `close`: release a process reference/handle.
- `truncate`: adjust file size.
- `unlink`: remove one directory entry; underlying file may persist while hard links or open references remain (Unix-like semantics).

### Example (Node.js)
```js
const fs = require('node:fs/promises');
async function run() {
  const fd = await fs.open('notes.txt', 'w+'); // create/truncate if needed
  await fd.writeFile('Hello OS!');
  await fd.close();
}
run();
```

**`w+` trap:** It truncates an existing file. It is not suitable when you want to preserve its old contents.

### File descriptor & open-file state
```
Process FD table -- fd 3 --> open file description (offset, status flags)
                                     |
                              in-memory file object/inode
                                     |
                               file data blocks
```
- An FD is a process-level reference, **not** the inode number.
- `dup()` produces a second descriptor referencing the same open file description, **sharing** its current file offset.
- Two independent `open()` calls ordinarily have separate open file descriptions and offsets even if referring to the same inode.

## 4. Blocks, size and internal fragmentation
A file system typically allocates storage in units of blocks/clusters. File data may occupy multiple, nonadjacent logical blocks.

If file size is **S bytes** and block size is **B bytes**, assuming full block allocation for file data:

`data blocks = ceil(S / B)`

`allocated data capacity = ceil(S / B) × B`

`unused last-block capacity = allocated capacity − S`

**Worked example:** File = 10 KiB; block = 4 KiB. Required blocks = ceil(10/4) = **3**; allocated capacity = **12 KiB**; unused final-block capacity = **2 KiB**.

```
File: 10 KiB
[Block A: 4 KiB used] [Block B: 4 KiB used] [Block C: 2 KiB used + 2 KiB unused]
```

**Units:** 1 KiB = 1024 bytes. Some examination questions write KB but intend 1024 bytes; follow stated assumptions. Real file systems may support inline data, sparse files, compression, tail packing, etc. Block-count formula is a standard simplified exam model, not a universal on-disk usage guarantee.

## 5. Inode concept
In a classical Unix-like inode-based file system, an inode contains metadata (mode, ownership, timestamps, length, data block address information) but **not the filename**. A directory associates filenames with inode numbers.

```
Directory mapping: "notes.txt" -> inode 105
                                    |
                  inode 105: metadata + block mapping
                                    |
                           data blocks / extents
```

Two names can refer to the same inode through **hard links**. Removing one name need not delete file contents when another hard link (or active open reference) remains. Details and exceptions: later lessons.

## 6. File system vs database system
```
Application
    ↓
DBMS (records, SQL queries, indexes, transactions)
    ↓
OS File System (files, dirs, file metadata, block mapping)
    ↓
Storage device
```
A DBMS page and a file-system block need not be equal in size. Some databases can also use raw devices or other storage abstractions; diagram shows a common arrangement.

## 7. Key exam traps (Day 1)
1. File system ≠ storage hardware.
2. File name ≠ inode number; inode usually does not directly store the name.
3. File descriptor ≠ inode number; FD is process-specific.
4. `dup()` shares an open file description and offset; independent `open()` ordinarily does not.
5. File data blocks need not be physically contiguous.
6. Internal fragmentation is unused space **inside** an allocated block; external fragmentation is free space split into noncontiguous regions (relevance depends on allocation strategy).
7. `close` does not necessarily mean data has reached durable physical storage; buffering/caching and synchronization matter.
8. `unlink` is not always immediate permanent deletion of underlying bytes.
9. Disk-block size, logical file size, and DBMS page size are separate quantities.
10. Unix `ctime` ordinarily means metadata/status change, not file creation.

## 8. Day 1 practice — solve before checking with tutor
**Q1.** On a classical Unix inode-based system, which statement is correct?
A. Inode always stores the full pathname
B. Directory entries map names to inode numbers
C. Inode number is identical to a process file descriptor
D. Every filename has unique physical blocks even with hard links

**Q2.** A regular file is 18 KiB and file-system block size is 4 KiB. Under simple full-block allocation, how many data blocks are allocated and how much capacity remains unused inside the last block?
A. 4 blocks, 2 KiB
B. 5 blocks, 2 KiB
C. 5 blocks, 0 KiB
D. 6 blocks, 6 KiB

**Q3.** A process opens the same file independently twice. Which statement is generally true on POSIX systems?
A. Both descriptors must share one current offset
B. Both descriptors are necessarily equal integers
C. They typically reference distinct open file descriptions with independent offsets
D. Each new open creates a different inode

**Q4.** Which of these is file metadata rather than file content?
A. The string in a text document
B. The RGB pixel array in an image
C. The file's access permission bits
D. The bytes representing an executable's machine instructions

**Q5.** A process has fd 3 and calls `dup(3)` to create fd 4. It then reads 100 bytes through fd 3. What is usually true of their current offset (assuming a seekable regular file and no concurrent changes)?
A. fd 4 still points to initial offset
B. Both descriptors share the updated current offset
C. A new inode is allocated to fd 4
D. `dup` copies file contents

**Q6 — Reasoning.** Differentiate *logical file size*, *allocated data capacity*, and *metadata overhead* with a 13 KiB file stored using 4 KiB blocks in a simplified file system. Calculate only the data-block capacity; explain why total actual disk usage may differ.

**Q7 — Concept.** Why doesn't `close()` alone always guarantee that writes are safely persistent after sudden power failure? Mention buffering and synchronization in your answer.

### Next lesson
**Access methods:** Sequential, Direct/Random, Indexed; file pointer semantics and examples; then directory structures, allocation techniques and inode numericals.

> Notes are a foundation, not a full treatment of all file-system management topics. Extend these after practice and concept-checking.
