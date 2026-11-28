# OS I/O Management — Part 3: Caching (Complete Revision Notes)

**Exam:** IBPS RRB Scale-II Specialist Officer (IT)  
**Level:** Concept-first + exam traps + Linux/Node.js applications  
**Coverage:** Linux page cache; page/frame/block distinctions; cache hits/misses; sharing; `mmap`; eviction; read-ahead; write-through/write-back; dirty pages; `fsync` / `fdatasync`; ext4 journaling, commit/checkpoint/replay; practical examples and MCQs.  
**Approach:** Numerical-heavy problems deferred.

---

## 1. Big picture: where caching fits in I/O

**Caching** keeps a copy of data in relatively fast storage so that future access may avoid a slower operation. For file I/O, Linux uses physical RAM for a **kernel-managed page cache**.

An ordinary buffered file read often follows:

1. Application makes a file read request (e.g., Node.js `fs.readFileSync`).
2. Kernel checks whether the requested file ranges are in the page cache.
3. **Hit:** data can be served from RAM without fetching those bytes from storage.
4. **Miss:** filesystem identifies data locations; storage reads bring data into RAM, usually with DMA-capable hardware.
5. Kernel supplies bytes to the application; cache may retain them for later reads.

**Key distinctions:**
- **Page cache** = retains file data, normally in RAM.
- **Read-ahead** = proactively fetches file data likely to be requested next.
- **DMA** = hardware-based data movement; not a caching policy.
- **Interrupt** = notification mechanism; not the content or copy of a file.
- **Buffering** = holds data in transit / while producer and consumer operate at different rates; caching primarily targets repeated access. One physical memory area can serve more than one purpose.

## 2. Filesystem blocks versus pages versus frames

The three concepts must not be mixed:

| Term | Meaning | Location / layer |
|---|---|---|
| Virtual memory page | Unit of a process's virtual address space | Virtual memory |
| Physical memory frame | Unit of physical RAM allocation/mapping | RAM |
| File-cache page / folio | Cached file data held using RAM frames | Linux kernel page cache |
| Filesystem block | Logical allocation/organization unit of a filesystem | Filesystem / secondary-storage mapping |
| Storage logical block / sector | Addressable unit exposed by block device | SSD/HDD interface |
| NAND flash page | Internal NAND read/program unit | Inside SSD hardware |

**Exam trap:** Saying “file pages are stored on SSD” is imprecise for page-cache discussion. A filesystem's file data is mapped to **filesystem/storage blocks** on disk; the kernel may cache file offsets in **RAM pages/folios**. SSD flash also has internal physical pages, but those are unrelated to the virtual-memory/page-cache terms.

**Example, with 4 KiB filesystem blocks and 4 KiB memory pages:**
- A 12 KiB file can occupy three logical filesystem blocks; the physical storage locations need not be adjacent.
- A cached copy might occupy three 4 KiB page-cache pages (actual folio organization may differ).
- A modern ext4 file commonly uses **extents** to represent block mappings rather than the traditional direct/single/double/triple-indirect pointers taught in textbook inode diagrams.
- The units need **not** match 1:1 in general: filesystem block size and memory page size may differ.

Remember: threads are not “pieces of a process's virtual memory.” Threads in a process normally **share the address space** while having their own stacks and execution state. A process's address space is divided into virtual pages for memory management.

## 3. Page-cache hits and misses

**Cache hit:** requested file data already resides in page cache.
- No physical disk read is needed for those cached bytes.
- Still requires CPU work, system-call/runtime overhead, and potentially a kernel-to-user copy.

**Cache miss:** requested file data is not resident or not current.
- Kernel may need actual storage I/O.
- Data can populate the page cache, possibly along with additional prefetched data.

**Important:** A second read is *not guaranteed* to be faster: data might be evicted, the first read might already have been cached, and allocations/copying/CPU load matter.

**Also:** The entire file need not be cached. A 10 GiB file can be read on an 8 GiB RAM machine, because reading and caching can happen incrementally. The kernel can reclaim old clean cached pages as necessary.

## 4. Shared kernel page cache versus private application buffers

The Linux page cache is kernel-managed and can benefit many threads and processes accessing the **same underlying file data**.

**Ordinary `read()` / Node.js `fs.readFileSync()`**:
- Process A reads `a.txt`: Linux may load its contents into page cache and copy bytes into A's user buffer.
- Process B reads the same underlying file: Linux may reuse the cached pages but normally copies bytes into B's own user buffer.
- **Shared page-cache source does not imply a shared Node.js Buffer**.
- Exiting Process A does not automatically invalidate the shared kernel cache.

**File mapping (`mmap`)**:
- Virtual addresses belonging to different processes can map to the same file-backed physical cache frames.
- Shared mappings and private copy-on-write mappings behave differently when modified.
- Do not assume ordinary `readFileSync()` operates like `mmap()`.

**Node.js caveat:** `fs.readFile` typically delegates normal filesystem work to libuv's worker pool on Linux, while `fs.readFileSync` blocks the calling JavaScript thread. Both can benefit from the same Linux page cache. Neither guarantees that the SSD is accessed.

## 5. Cache replacement and eviction

**Eviction** = remove cached data from RAM to make capacity available. Evicting a **clean cached file page** does not delete the original file on SSD.

Theory algorithms:

| Algorithm | Which entry does it remove? | Main weakness |
|---|---|---|
| FIFO | First entered | Oldest entry might be very popular |
| LRU | Least recently accessed | Requires tracking/approximating recency |
| LFU | Least frequently accessed | Once-hot, now-cold entries may linger |
| Optimal | Next use is farthest in the future | Future accesses generally unknown |

**LRU example** (3 slots): `A,B,C` initially ordered least-to-most recent. Access `A` → order `B,C,A`. Insert `D` → evict `B` → `C,A,D`.

**Actual Linux is not pure textbook LRU.** The kernel has advanced memory-reclaim approaches, historically active/inactive LRU-style lists and, on suitable systems, multi-generational LRU (MGLRU). It considers access history, reclaim costs, working sets, memory pressure and other factors.

- **Hot data** = accessed recently/frequently; generally worth protecting.
- **Cold data** = older/less recently used; often better reclaim candidate.
- **Clean file page** = can normally be discarded and fetched again.
- **Dirty file page** = contains modifications pending writeback, so must not be simply discarded without preserving those changes.

## 6. Read-ahead (prefetching)

**Read-ahead** proactively loads file data into the page cache before an application explicitly requests it.

**Illustration:** Application asks for file offset 0–4095; Linux may also fetch later logical offsets 4096–8191, 8192–12287, etc. These refer to **logical file regions**; the filesystem blocks they map to need not be contiguous on SSD.

- Effective for **sequential reads**: log scan, CSV/large file streaming.
- Less effective for **unpredictable random reads**: unnecessary prefetch can waste RAM, bandwidth and cache space.
- Read-ahead is **adaptive**, not a promise to fetch the same fixed number of bytes each time.
- On a miss, the kernel may perform synchronous demand work and arrange asynchronous prefetch for upcoming offsets.

**Don't confuse these**:
- Node.js `createReadStream(..., { highWaterMark: 64 * 1024 })`: stream-side buffering threshold (not a hard cap on all memory use).
- Linux block-device `/sys/block/<device>/queue/read_ahead_kb`: device read-ahead setting, not a guarantee of exact bytes prefetched on each application read.
- `posix_fadvise(..., POSIX_FADV_SEQUENTIAL)` / `POSIX_FADV_RANDOM`: hints about a file's expected access pattern.

**Read-ahead + DMA:** Kernel chooses *which future file regions to read*; storage hardware may transfer their bytes into RAM using DMA. Distinct layers.

## 7. What is write-back caching?

Ordinary Linux buffered writes can be **acknowledged before the underlying SSD has durably stored the new data**.

Example:

```
Initial:   Page cache = HELLO, SSD = HELLO
write():   Page cache = WORLD (dirty), SSD may still = HELLO
writeback: SSD updated to WORLD; page may become clean
```

- **Write-back** = allow cache to accept changes and write them to backing store later.
- Enables batching/coalescing of repeated modifications and lower write latency.
- Exposes a window in which a crash may lose recent data.
- The dirty page does not necessarily get evicted once successfully written: it may remain in RAM as a **clean cached page**.
- On a storage error, Linux can report writeback errors; do not assume it will retry/retain every byte forever.

**Write-through** (conceptual contrasting policy) = do not acknowledge a write until required backing-store updates complete. It may reduce the volatile pending-write window but often costs latency.

These are *cache update policies*, **not** synonymous with synchronous versus asynchronous JavaScript methods.

| Topic | Write-back | Write-through |
|---|---|---|
| When acknowledge? | May acknowledge before backing storage finishes | After backing write completes per policy |
| Performance | Often faster, can batch writes | May require more immediate I/O |
| Risk | Data pending in volatile cache may be lost | Less volatile pending data at acknowledgment |

**Exam trap:** ext4's mount mode `data=writeback` is a **specific journaling/data-ordering mode**, not the generic name of Linux buffered write-back caching.

## 8. Dirty pages, writeback, and clean pages

**Dirty:** Cached file content or related state has been modified and needs synchronization with the backing representation.

**Writeback:** Kernel initiates I/O to write the modified content to storage. Writeback may be asynchronous to the application's original write call.

**Clean:** No remaining unsynchronized modification for that cached data. It may continue to be cached in RAM.

**Evicted:** Cached copy removed from RAM when reclaimable. Not the same as being clean.

**Why it matters:** Clean cache entries are cheap to reclaim; dirty entries may need to be written before they can be reclaimed safely.

Linux diagnostics:

```bash
free -h
grep -E '^(Cached|Active\(file\)|Inactive\(file\)|Dirty|Writeback|MemAvailable):' /proc/meminfo
cat /proc/sys/vm/dirty_background_ratio
cat /proc/sys/vm/dirty_ratio
cat /proc/sys/vm/dirty_expire_centisecs
```

These are system-wide stats/settings. They are **not** exact real-time guarantees that a particular file is already on SSD, and dirty expiry settings are not a per-write durability timer.

## 9. `write()`, `close()`, `fsync()`, and `fdatasync()`

### `write()` / `fs.writeFileSync()`

A successful buffered write ordinarily means the OS accepted the bytes into its I/O path; it **does not necessarily mean durable storage**. Node.js's synchronous API blocks the *calling thread* until the underlying operation completes, **not necessarily until media persistence**.

### `fsync(fd)` / Node.js `fs.fsyncSync(fd)`

Requests synchronization of modified **file contents and relevant metadata**, and waits until the storage stack reports completion, subject to hardware/filesystem guarantees. May involve file-data writeback **and** relevant journal commits.

**Important:** `fsync()` is an *interface/operation expressing a durability requirement*, not itself a journaling protocol.

### `fdatasync(fd)`

Similar to `fsync()` but may omit metadata unnecessary for correct subsequent data retrieval. File size/extent information needed to retrieve the new bytes **may still need synchronization**.

### `close(fd)`

Not generally a replacement for `fsync()` when application durability is required.

### Newly created files / rename

Making a file's contents durable does **not automatically ensure the parent directory entry** is durable. Robust create/rename workflows on suitable local POSIX filesystems often synchronize the containing directory as well.

### Error handling

`fsync` / `fdatasync` can fail (`EIO`, `ENOSPC`, etc.). Do not tell the user an operation is durable after a reported error. Even when a write fails, some bytes might already have reached storage: treat the final state carefully.

## 10. Node.js practical code

### A. Cached reads

```javascript
const fs = require('node:fs');

for (let i = 1; i <= 3; i++) {
  const start = process.hrtime.bigint();
  const data = fs.readFileSync('sample.bin');
  const ms = Number(process.hrtime.bigint() - start) / 1e6;
  console.log(`Read ${i}: ${data.length} bytes, ${ms.toFixed(2)} ms`);
}
```

**Interpretation:** Later reads *may* benefit from page cache; differences in timing are not proof on their own. The first read could have been warm already.

### B. Streaming reads (incremental memory use)

```javascript
const fs = require('node:fs');
const input = fs.createReadStream('large.csv', { highWaterMark: 64 * 1024 });
let bytes = 0;
input.on('data', chunk => { bytes += chunk.length; });
input.on('end', () => console.log({ bytes }));
input.on('error', console.error);
```

**Interpretation:** Node.js buffering and Linux read-ahead are separate layers.

### C. Explicitly synchronize a write

```javascript
const fs = require('node:fs');
const fd = fs.openSync('balance.txt', 'w');
try {
  fs.writeFileSync(fd, '9000');
  fs.fsyncSync(fd);
  console.log('File synchronization completed');
} finally {
  fs.closeSync(fd);
}
```

**Important:** This writes/synchronizes an individual file; if **creating** the name or making an important **rename**, also consider directory synchronization for crash safety. This snippet is educational; storing monetary balances in an ad-hoc flat file is not a sound production transaction design.

### D. About DMA versus JS sync/async

- `fs.readFileSync`: JS calling thread waits; disk may still use DMA.
- `await fs.promises.readFile`: ordinary libuv filesystem work can use worker threads; disk may still use DMA.
- Data already in page cache: no disk DMA necessary for those cached bytes.
- **DMA does not imply JavaScript `async`**; these concepts belong to different layers.

## 11. Journaling: separate mechanism, may work with `fsync`

**Write-back caching**: *When/how do modified cached bytes reach backing storage?*  
**`fsync()`**: *Please synchronize this file's data and required metadata.*  
**Journaling**: *How can recorded filesystem transactions be recovered to a consistent state after a crash?*

### ext4's common `data=ordered` mode

- Journal primarily records **metadata** (inode/extent/block allocation, etc.).
- Associated file data is ordered to be written to its normal storage location before the relevant metadata transaction commits.
- **This does not mean the journal contains a backup of every new file byte.**
- If permanent device failure prevents file data from being stored, metadata journaling alone cannot reconstruct arbitrary missing data.

### Example: append causes block allocation

1. Application appends bytes to existing `a.txt`; new data may initially be dirty in page cache.
2. Filesystem must choose more storage blocks and update mappings (ext4 may use **delayed allocation**, so actual placement can happen later).
3. `fsync()` is requested.
4. For a successful sync in ordered mode, necessary file data is written and required metadata/journal transaction is made durable in the appropriate order.
5. `fsync()` may return **before journal checkpointing** is finished.
6. Crash occurs; the committed journal records can be replayed to restore needed metadata mappings.

**Journal stages**:

| Stage | Meaning |
|---|---|
| Transaction prepared | Related metadata changes collected |
| Journal commit | Committed transaction is durably recorded and recoverable |
| Checkpoint | Committed updates propagated to normal filesystem metadata locations, freeing journal space eventually |
| Journal replay | On recovery, apply needed committed journal records; not necessarily “restart checkpoint where it stopped” |

**Not a universal database rollback:** For common redo-style journaling, uncommitted changes are not replayed as committed transactions; committed changes may be replayed. Journaling does not promise an atomic debit/credit business operation.

### ext4 journaling modes

| Mode | General behavior |
|---|---|
| `data=ordered` | Normally journals metadata; coordinates data-before-metadata-commit ordering |
| `data=writeback` | Journals metadata without the same data-ordering requirement |
| `data=journal` | Journals both file data and metadata; substantially different and often slower |

**Note:** Features/mount options can change across kernels; check actual filesystem configuration for real systems.

### Crash/failure scenarios

| Scenario | Correct interpretation |
|---|---|
| Modified data only in RAM, then power loss | Latest bytes may be lost |
| Metadata journal committed, crash before checkpoint | Replay can recover committed metadata changes |
| `fsync()` returns success | File's modified data and relevant metadata are intended to be durable, assuming filesystem/device fulfill guarantees |
| `fsync()` returns `EIO` | Durability uncertain; journaling cannot guarantee restoration of missing file contents |
| File created and its FD `fsync`ed, directory not synchronized | Filename/directory-entry durability may still be at risk |

## 12. Cache vs buffer vs spooling — anticipation

| Term | One-line purpose | Example |
|---|---|---|
| Buffering | Absorb timing/rate mismatch | TCP receive buffer, stream buffer |
| Caching | Avoid repeated costly access | Linux page cache |
| Spooling | Queue jobs for a device/service to process independently | Printer job queue |

Spooling is the *next topic*, not a synonym for the page cache.

## 13. Linux commands and safe practical checks

```bash
# Read approximate cached/buffered memory and available memory
free -h

# Examine file-backed activity and dirty/writeback counters
grep -E '^(Cached|Active\(file\)|Inactive\(file\)|Dirty|Writeback|MemAvailable):' /proc/meminfo

# Explore page-reclaim counters (system-wide)
grep -E '^(pgscan|pgsteal|workingset_refault_file)' /proc/vmstat

# Find a storage device name (example: nvme0n1, sda)
lsblk

# For a device that exists on your system
cat /sys/block/nvme0n1/queue/read_ahead_kb

# Time sequential reads (results affected by existing page cache)
time dd if=testfile.bin of=/dev/null bs=4M status=none
```

**Safety:** Do not casually drop filesystem caches on a production system; this disrupts performance and is not required to learn the concept. `read_ahead_kb` may differ based on the system; paths must match existing devices.

## 14. Exam traps — memorize distinctions, not just definitions

1. Page cache is stored in **RAM**, not a second SSD partition.
2. SSD filesystem **blocks** are not the same thing as virtual-memory **pages**.
3. Page cache is **shared by the kernel**, not exclusively owned by the process that first read a file.
4. Two ordinary `read()` calls often get **separate user buffers** even when served from the same cached pages.
5. `mmap()` can map file-backed cached physical frames into multiple address spaces.
6. Cache eviction of a **clean** page doesn't delete the file on disk.
7. Linux reclaim is **not guaranteed to be textbook pure LRU**.
8. Sequential reads usually benefit more from read-ahead than random reads.
9. Read-ahead may waste RAM/bandwidth if predictions are wrong.
10. Node.js `highWaterMark` does **not** configure kernel read-ahead.
11. `writeFileSync()` does **not** automatically guarantee persistent storage.
12. Write-back caching isn't the same thing as `data=writeback` ext4 journaling mode.
13. Dirty cached pages and journaled metadata are related but **not one identical “dirty bit.”**
14. Journal **commit** ≠ checkpoint; commit can be recoverable while checkpoint is pending.
15. Journal commit ≠ all application data at final SSD locations in every filesystem/mode.
16. `fsync()` can use journaling but **is not the journal**.
17. Successful `fsync()` on a file may not persist the parent **directory entry**.
18. A hardware I/O error can defeat durability; journaling doesn't manufacture lost file bytes.
19. DMA does not automatically make a user-space API asynchronous.
20. App-level caching (Redis, in-process Map) is distinct from the kernel filesystem page cache.

## 15. Multiple-choice practice (answers below)

**Q1.** Process A and Process B read the same file using normal `read()`. What can be shared?  
A. Their JS Buffers are always identical objects  
B. Linux's underlying cached file pages  
C. Their entire virtual address space  
D. Their user-space stack frames

**Q2.** What does evicting a clean page-cache page normally do?  
A. Delete the file  
B. Remove cached copy from RAM  
C. Remove inode from SSD  
D. Commit a journal transaction

**Q3.** Which best describes read-ahead?  
A. Encrypt all file reads  
B. Preload likely future file data  
C. Delay all writes  
D. Convert SSD filesystem blocks into virtual pages

**Q4.** A Node.js stream uses `highWaterMark: 65536`. What does this set?  
A. Fixed SSD read-ahead size  
B. Number of physical frames in RAM  
C. Stream buffering threshold  
D. Number of journal transactions

**Q5.** Which file pages are cheapest to reclaim in general?  
A. Clean, no pending modifications  
B. Dirty, unsynchronized  
C. Locked/pinned  
D. All equally

**Q6.** A successful `writeFileSync()` with no `fsync()` guarantees:  
A. SSD is durably updated  
B. Journal checkpoint is complete  
C. Write finished at the API level, but media durability is not guaranteed  
D. File data is necessarily stored in the journal

**Q7.** Why can a committed ext4 journal transaction be useful after a crash *before checkpoint*?  
A. It contains recoverable committed metadata updates  
B. It contains a permanent backup of all file contents in default mode  
C. It deletes the dirty page  
D. It increases DMA bandwidth

**Q8.** Which statement is most accurate for `fdatasync()`?  
A. It never syncs metadata  
B. It may sync metadata necessary to retrieve file data correctly  
C. It always commits all filesystem journals globally  
D. It guarantees the parent directory entry is durable

**Q9.** Which is most correct about Programmed I/O and DMA in relation to caching?  
A. Both are cache replacement algorithms  
B. DMA can move data into RAM; caching decides whether to retain it  
C. Caching is possible only when CPU copies each byte  
D. A cache hit always needs another DMA transfer

**Q10.** Linux already has file data cached, and two processes call `read()` on it. What is normally true?  
A. Both use the same physical JS Buffer  
B. Neither can copy data  
C. Each can receive its own user buffer from the same kernel cache  
D. One must wait for the other process to exit

### Answers and explanations

1. **B** — The kernel shares cached file data; ordinary user buffers are typically separate.
2. **B** — Eviction drops cached RAM copy, not the SSD source.
3. **B** — Prefetch likely upcoming logical file offsets.
4. **C** — Node.js stream buffering ≠ kernel read-ahead.
5. **A** — Clean cache can normally be reclaimed without writeback.
6. **C** — Synchronous function call is not the same as durable disk synchronization.
7. **A** — Committed journal records may be replayed to restore consistent metadata.
8. **B** — Required metadata (file length/extent info) may still be synchronized.
9. **B** — Different layers: transfer versus retention.
10. **C** — Shared kernel cache, separate user-visible copies.

## 16. Short revision questions (answer in your own words)

1. What is the difference between a filesystem block, a page-cache page, and a physical memory frame?
2. Why can a different process get a cache hit from a file read by another process?
3. Why might repeated reads be fast even if the original application exited?
4. Can a 10 GiB file be read sequentially with only 8 GiB RAM? Explain.
5. Why might read-ahead hurt performance for scattered file offsets?
6. Why is a clean page easier to evict than a dirty page?
7. How can a journal commit be durable while the metadata checkpoint is not complete?
8. Why can `fsync()` fail to recover `WORLD` if an SSD permanently fails during writeback?
9. What is the role of the directory `fsync()` after creating a file?
10. Can hardware DMA be active while `fs.readFileSync()` blocks the Node.js calling thread?

## 17. Final 60-second recap

- **Read:** SSD filesystem data blocks → RAM page cache → user buffer (unless file mapping follows another path).
- **Share:** One kernel page cache can serve multiple processes; normal reads still usually copy to process-local buffers.
- **Evict:** Clean cached pages can be discarded; dirty pages need changes preserved/written back.
- **Prefetch:** Read-ahead guesses future sequential file offsets to avoid later waiting.
- **Write:** Buffered write can finish in RAM before persistent storage catches up.
- **Synchronize:** `fsync()` requests durability for file data/required metadata; handle errors and directory metadata requirements.
- **Journal:** Commit records make relevant transactions recoverable; checkpoint writes committed metadata to home locations; replay recovers after crash.
- **Remember:** **Write-back = timing of data persistence; `fsync` = persistence request; journaling = recovery mechanism.**

---

## References (official, useful for checking technical details)

- [Linux kernel: ext4 journal (JBD2), data modes and checkpointing](https://www.kernel.org/doc/html/latest/filesystems/ext4/journal.html)
- [Linux kernel: ext4 admin guide](https://cdn.kernel.org/doc/html/latest/admin-guide/ext4.html)
- [Linux `fsync(2)` / `fdatasync(2)` manual](https://man7.org/linux/man-pages/man2/fsync.2.html)
- [Linux kernel: Memory-management APIs / Readahead](https://origin.kernel.org/doc/html/latest/core-api/mm-api.html)
- [Linux kernel: Multi-Gen LRU](https://docs.kernel.org/mm/multigen_lru.html)

*These notes aim for exam-ready conceptual accuracy. Actual behavior can vary by filesystem, kernel version, mount options, device firmware, and storage hardware.*
