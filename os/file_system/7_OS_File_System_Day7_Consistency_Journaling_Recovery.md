# Operating Systems — File System Management, Day 7
## File-System Consistency, Journaling, Crash Recovery, and Durability

**Target:** IBPS RRB SO IT Officer (with moderate-to-advanced conceptual depth)  
**Language:** English  
**Prerequisites:** Inodes, directories, hard links, free-space bitmap, blocks, file descriptors, page cache.

---

## 1. The Fundamental Problem: One Operation Changes Multiple Structures

A seemingly simple operation such as creating a file may require changing several related pieces of persistent metadata:

1. An inode allocation bitmap, to mark an inode as allocated.
2. A data-block bitmap, to mark data blocks as allocated.
3. The inode itself, containing size, timestamps and allocation information.
4. A directory, to associate a filename with its inode.
5. Actual data blocks, containing the file's bytes.

These are **logical effects**, not a mandatory write order. Implementations may use delayed allocation, batching, journaling and caching.

**Crash-consistency problem:** A power failure or kernel crash may interrupt the operation after some changes have reached persistent storage but before the others have.

### Case 1: Space leak
- Bitmap claims block 500 is allocated.
- No inode references block 500.
- The space may appear unavailable despite no reachable file using it.

### Case 2: Dangerous free/used disagreement
- An inode references block 500 as file data.
- Bitmap claims block 500 is free.
- Allocator might reuse block 500 for another file and corrupt data.

### Case 3: Directory/inode mismatch
- Two hard-link directory entries refer to one inode.
- Its stored link count disagrees with the actual number of hard links.

### Case 4: Partly updated file size / allocation metadata
- An inode's recorded size and valid block mapping are not mutually consistent.
- Truncation, extension and creation require coordinated changes.

**Consistency** means on-disk file-system structures satisfy required invariants. **Durability** means changes for which persistence has been promised survive a crash. A file system can recover to a consistent state without preserving every latest user write.

---

## 2. Why a Successful `write()` Is Not a Power-Loss Guarantee

A normal buffered `write(fd, bytes, length)` copies or stages data into the kernel's page cache and returns, possibly before the data is written to persistent storage. Modified cached pages are called **dirty pages**. Later, the kernel writes them back to the storage device.

- **Page cache:** RAM used to cache file contents; improves repeated reads and batches writes.
- **Writeback:** Transfers modified cached pages to storage.
- **Flush/barrier:** Storage-ordering or cache-flush mechanisms help honor durability requirements.
- **Successful `write()`:** Ordinarily means the requested bytes were accepted by the kernel, not necessarily durable after sudden power loss.
- **`close()`:** Releases the open descriptor; do not equate it with a durability barrier.
- **`fsync(fd)`:** Requests that file data and relevant metadata be written to durable storage, subject to proper kernel/device behavior.
- **`fdatasync(fd)`:** Synchronizes data and only metadata required to retrieve it, such as a size extension.

**Directory entry subtlety:** `fsync()` on a newly created file does not necessarily ensure its *directory entry* is persistent. A separate sync of the containing directory may be required on Linux/POSIX-like systems.

### Example
1. Application writes 100 bytes.
2. `write()` returns 100.
3. The bytes are dirty in RAM.
4. Sudden power loss occurs before durable writeback.

The 100 bytes may be lost. Metadata journaling alone does not automatically make every application write durable.

---

## 3. Consistency Checking (`fsck`)

Traditional file-system checking tools inspect metadata to detect and repair corruption or inconsistencies. On Linux, ext2/ext3/ext4 use `e2fsck`; `fsck` may dispatch to a filesystem-specific checker.

Typical checks:
- Valid inode references and block allocation.
- Allocated blocks compared with references from inodes.
- Illegal or duplicate references to data blocks.
- Directory entry validity and reachability.
- Hard-link counts versus directory references.
- Valid metadata structure and free-space accounting.

A checker may repair metadata but cannot necessarily reconstruct user data that never reached persistent storage. A full check can take time on large file systems and may require the volume to be offline or in a safe state. Do not run destructive `fsck` repairs on a mounted read-write root file system casually.

### Exam distinctions
- `fsck` = detection/repair through examination of on-disk structures.
- Journaling = log-based recovery of bounded recorded transactions.
- Neither is a substitute for backups.

---

## 4. Journaling: A Transaction Log for Crash Recovery

A journaling file system maintains a **journal** (log) containing records for transactions affecting important file-system structures.

For a transaction to update a bitmap, inode and directory:

1. Collect the relevant journal records.
2. Ensure the required records are persistent before declaring transaction committed.
3. Persist a commit marker / record according to the journaling protocol.
4. Subsequently apply committed changes to normal storage locations (**checkpoint**).
5. Safely reclaim old journal space once checkpointing makes replay unnecessary.

The precise ordering and record format vary by implementation. The Linux ext4 JBD2 journal, for example, ensures that a committed metadata transaction can be replayed when its final-home writes were interrupted.

**Write-Ahead Logging (WAL) principle:** Critical log information must be durable *before* the corresponding protected updates are treated as durably committed. The core idea also appears in databases, but database transaction semantics and file-system transactions are not automatically identical.

### Crash timeline (simplified redo journal)

| Crash point | Recovery interpretation |
|---|---|
| Before durable journal commit | Transaction is incomplete; do not replay its uncommitted updates |
| After durable commit, before checkpoint | Replay committed journal records to restore protected metadata |
| During checkpoint | Replay committed journal records, if needed, to finish protected metadata updates |
| After checkpoint | Relevant final-location metadata already installed; replay for this transaction unnecessary |

**Idempotence:** A recovery protocol must handle reapplying journaled changes safely when checkpointing was only partial. This property is important because a second crash can occur during recovery.

**Journaling is not a backup.** It does not guarantee recovery from physical media failure, malicious deletion, or data that was never persisted. It protects the file system against specific crash-consistency failures.

---

## 5. Metadata Journaling vs Data Journaling

A file has two kinds of information:

- **Metadata:** inode attributes, block allocation information, directory entries, bitmaps, etc.
- **File data:** The actual contents of a file (for example, `Hello` or image bytes).

Many journaling file systems prioritize journaling **metadata** to reduce overhead. This preserves file-system structural consistency but does not inherently guarantee the newest file contents survive a crash.

**Example:** If a new file's directory and block-allocation metadata were journaled and recovered, its content could still be old, missing or only partially updated, depending on the journaling mode and durability guarantees.

### ext4 data modes (classical distinctions)

| Mode | What is journaled? | Relevant ordering | Trade-off |
|---|---|---|---|
| `data=journal` | File data and metadata | Data and metadata go through journal | Stronger crash behavior; higher write overhead |
| `data=ordered` | Metadata | Associated file data written before related metadata journal commit | Common/default ext4 mode; helps avoid stale exposure for certain newly allocated blocks, but not an application durability promise |
| `data=writeback` | Metadata | Data may be written after related metadata journal commit | Less ordering overhead, weaker crash-time file-content expectations |

**Nuance:** Even `data=journal` does not mean that any successful `write()` has completed a durable journal commit. An application may still need an explicit synchronization operation. Specific ext4 mode availability and behavior can change by kernel version.

References:
- Linux kernel ext4 journaling: https://www.kernel.org/doc/html/latest/filesystems/ext4/journal.html
- Linux kernel ext4 general information: https://cdn.kernel.org/doc/html/latest/admin-guide/ext4.html

---

## 6. Recovery by Journal Replay, and Why the Commit Record Matters

Consider a protected metadata transaction `T1`:

- Journal has a valid, durable set of records for `T1`.
- A valid, durable commit record for `T1` exists.
- System crashes before all final-location metadata blocks have been installed.

Recovery may replay `T1` from its journal records. By contrast, if `T1` never achieved a valid committed state, recovery does not replay it as a completed transaction in the simplified redo-journaling model.

This creates transaction-level atomicity for **the journaled structures**: a partially installed group of related metadata updates should not leave the file system in an arbitrary half-updated state after successful recovery.

Do **not** generalize this to 'one application-level file write is always atomic', or 'every recent file write survives a crash'. The scope of the journaled transaction matters.

---

## 7. `fsync`, `fdatasync`, and Durable File Replacement

### `fsync`

On Linux, `fsync(fd)` requests that modified file data and its metadata reach the storage device in a manner that can be recovered after a crash. This includes appropriate flushing of volatile caches as needed; durable behavior ultimately relies on a correctly operating storage stack.

### `fdatasync`

Synchronizes changed file data and metadata needed to correctly access it (for example, an increased file size), without necessarily flushing changes like only the access timestamp.

### `rename` is not a substitute for `fsync`

A common update pattern for replacing a configuration file on the same file system is:

1. Create `config.tmp` in the same directory.
2. Write complete new content to `config.tmp`.
3. `fsync(config.tmp)` and close it; handle errors.
4. `rename(config.tmp, config.json)` (atomic namespace replacement in the intended local same-file-system scenario).
5. `fsync` the containing directory so the name change has a stronger durability guarantee after a crash.

**Distinguish two properties:**
- Atomic visibility: readers see either the old or new named file, not a halfway name replacement under the right semantics.
- Crash durability: the new contents and directory entry survive power loss.

Source: https://man7.org/linux/man-pages/man2/fsync.2.html

---

## 8. Journaling vs Database Transactions

Because DBMS systems also use logs, it is useful to compare:

| Concept | File-system journaling | Database write-ahead logging |
|---|---|---|
| Typical consistency target | File-system metadata relationships | Database pages, records, transaction semantics |
| Commit scope | Journal transaction grouping OS metadata and possibly file data | DB transaction with application-visible transaction semantics |
| Crash recovery | Replay committed filesystem changes | Redo/undo or other techniques according to DB recovery design |
| Guarantees every app `write()` is durable? | No | Not relevant: depends on DB commit durability policy |
| Substitutes for backups? | No | No |

A journaling file system underneath a database does **not** replace the DBMS's own transaction log. The file system cannot infer DB-level constraints such as balances, unique transactions or foreign keys.

---

## 9. Copy-on-Write: An Alternative Crash-Consistency Approach

Some modern file systems (notably Btrfs and ZFS) use copy-on-write (CoW) strategies rather than relying on classic in-place updates plus a traditional metadata journal for their principal on-disk consistency model.

Conceptual CoW update:

1. Write changed blocks to new locations, keeping old blocks untouched.
2. Build updated metadata referencing the new blocks.
3. Switch a top-level reference or root to make the new structure visible, using the file system's required ordering/commit protocol.

The old consistent structure remains available until the new one is safely committed. This can support snapshotting and integrity mechanisms, although it brings its own performance and complexity trade-offs.

**Exam caution:** CoW and journaling solve related crash-consistency problems with different mechanisms; they are not synonyms.

---

## 10. Quick Definitions for Revision

- **Crash consistency:** Recoverability of file-system structures after an unexpected interruption.
- **Metadata:** File information (inode, size, permissions, block maps, directory entries, etc.).
- **Dirty page:** Modified cached file data not yet persisted.
- **Journaling:** Recording transaction changes in a log so interrupted updates can be recovered.
- **Commit record:** Indicator that the journal transaction has durably completed according to the journal protocol.
- **Replay/redo:** Reapply committed logged changes during recovery.
- **Checkpointing:** Transfer committed journaled changes to their regular storage locations.
- **`fsck`:** Consistency-checking and repair tool.
- **`fsync`:** Request to make modified file data and relevant metadata durable.
- **`fdatasync`:** Like `fsync`, but only syncs data and metadata needed to access the data.
- **Data journaling:** File data plus metadata are journaled.
- **Metadata journaling:** Metadata is journaled; actual file contents need separate handling.
- **Write ordering:** Constraints ensuring certain writes persist before dependent writes become committed.

---

## 11. Exam Traps

1. A successful `write()` does not always mean a durable disk write.
2. `close()` is not generally equivalent to `fsync()`.
3. Metadata journaling protects metadata consistency, not necessarily recent file data.
4. A valid committed transaction can be replayed even if home-location writes were incomplete.
5. Incomplete/uncommitted journal records should not be replayed as committed transactions in the simplified redo model.
6. `fsck` repairs detected inconsistencies; journaling uses recorded transactions for recovery.
7. Journal replay need not undo a user's intentionally deleted file.
8. `rename()` can give atomic namespace replacement, but durable persistence also requires correct synchronization.
9. Keeping a journal does not guarantee protection from hardware failure, ransomware or accidental deletion.
10. **Atomicity, consistency, durability, and availability are distinct ideas.**

---

## 12. Practice Questions — IBPS SO IT (Moderate/Upper-Medium)

**Q1.** A file-system free-space bitmap marks block 80 as free, while an allocated inode references block 80 as file data. Which risk is most direct?

A. The file has too many open descriptors.  
B. Another file may receive block 80, leading to corruption.  
C. The inode must contain a symbolic link.  
D. The block has become a cache entry.

**Q2.** In a simplified redo-only metadata journal, a transaction's journal records are persisted but the transaction commit record has not become durable. A crash occurs. Which is the expected recovery action?

A. Replay it as committed.  
B. Treat it as incomplete rather than replaying it as committed.  
C. Permanently erase all user files.  
D. Rebuild the entire storage device.

**Q3.** A metadata transaction was durably committed in the journal. Power fails while its home-location blocks are only partially updated. What is the usual purpose of journal replay?

A. Retrieve missing application data from the network.  
B. Apply the committed metadata changes to restore consistency.  
C. Delete the inode and start the disk format again.  
D. Convert linked allocation into contiguous allocation.

**Q4.** On a normal buffered Linux file write, which statement is the most accurate?

A. A successful `write()` alone always guarantees power-loss durability.  
B. `close()` always performs a durability flush.  
C. An application generally needs an operation such as `fsync()` when it needs an explicit file durability guarantee.  
D. Journaling makes the page cache unnecessary.

**Q5.** Which ext4 mode journals metadata and orders related data-block writes before metadata journal commit, without writing data blocks through the journal?

A. `data=ordered`  
B. `data=writeback`  
C. `data=journal`  
D. `data=none`

**Q6.** Why is `fsync()` on a newly written file sometimes insufficient to guarantee that the new filename itself will survive a crash?

A. An inode can never be synchronized.  
B. The containing directory entry may require a separate directory `fsync()`.  
C. A hard link must be created first.  
D. The file must be in the root directory.

**Q7.** Which is the best distinction between `fsck` and journal replay?

A. `fsck` works on RAM only, replay works on CPU registers.  
B. `fsck` checks file-system consistency; journal replay reapplies recorded committed changes.  
C. Journal replay always recovers deleted user files; `fsck` cannot.  
D. Both guarantee restoration of any unpersisted file contents.

**Q8.** A file system is structurally consistent after recovery, but a recent user write was not preserved. Which conclusion is valid?

A. Consistency and durability are different properties.  
B. The file system must still be inconsistent.  
C. It is impossible for a journaled file system.  
D. The inode pointer size must be too small.

### Answers (open only after practicing)

<details>
<summary>Answer key and reasoning</summary>

1. **B:** The allocator could reuse a referenced block, corrupting either file.
2. **B:** Without a committed transaction, redo replay must not treat it as completed.
3. **B:** Committed metadata changes can be replayed after interrupted checkpointing.
4. **C:** A buffered write can return before data is durable; request appropriate syncing.
5. **A:** `data=ordered` journals metadata and orders associated data writes before metadata commit.
6. **B:** A new directory name/entry is a separate directory metadata update.
7. **B:** `fsck` scans/repairs; replay uses the known transaction log.
8. **A:** Structural consistency does not mean every recent user write survived.

</details>

---

## 13. Recommended Study Order After Day 7

1. Answer the eight practice questions above.
2. Revise the 10 exam traps and distinguish metadata vs data.
3. Proceed to **File Permissions and Protection** (r/w/x on files and directories, chmod, umask, ACL, owner/group, mount basics).
4. Then take a mixed File System Management revision test with short numericals, inode/link scenarios and recovery questions.

## Primary Technical References

- Linux kernel: ext4 Journaling (JBD2): https://www.kernel.org/doc/html/latest/filesystems/ext4/journal.html
- Linux kernel: ext4 mount modes and ordering: https://cdn.kernel.org/doc/html/latest/admin-guide/ext4.html
- Linux man-pages: `fsync(2)` / `fdatasync(2)`: https://man7.org/linux/man-pages/man2/fsync.2.html

*Notes use classical textbook models when explaining journal phases and inode mapping. Actual crash guarantees depend on the file system implementation, mount options, kernel, and storage hardware.*
