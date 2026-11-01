# OS File System Management — Advanced Inode Addressing (Detailed Notes)

## Scope and assumptions
These are textbook inode-addressing models, not claims about every modern file system. Unless specified otherwise, assume fixed-size data and index blocks, a pointer occupies a fixed number of bytes, file blocks are numbered from zero, no sparse blocks, and all required metadata blocks must be allocated. Use KiB = 1024 bytes and MiB = 1024 KiB.

## Key distinctions
- **File byte offset:** position of a byte from the beginning of a file (zero-based).
- **Logical file block:** floor(byte_offset / data_block_size).
- **Offset within data block:** byte_offset % data_block_size.
- **Storage block address:** address found through the inode's block mapping, not a RAM frame number.
- **Direct pointer:** inode slot containing a data-block address.
- **Indirect pointer:** inode slot pointing to an *index block* filled with addresses; those addresses may lead to data blocks or further index blocks.
- An indirect index block **does not contain the user file data** it can address.

## Pointer capacity
Let B = block size in bytes, P = pointer size in bytes, D = number of direct pointers, N = B/P assuming exact divisibility and no index block header.

- Direct region: D data blocks.
- Single-indirect region: N data blocks.
- Double-indirect region: N^2 data blocks.
- Triple-indirect region: N^3 data blocks.
- If an inode contains one of each indirect pointer, the maximum data capacity is (D + N + N^2 + N^3) * B bytes.
- Do not count index blocks as user data in the maximum logical file-size calculation.

## Worked example 1: byte offset lookup
B = 1024 bytes, D = 3, P = 4 bytes. N = 256.

File byte offset X = 4300:
1. Logical block L = floor(4300/1024) = 4.
2. Within-block byte offset = 4300 % 1024 = 204.
3. Blocks 0, 1, 2 use direct pointers. The first single-indirect block is logical block 3.
4. Single-indirect entry = L - D = 4 - 3 = 1.
5. Read the block address from index entry 1; then read byte 204 of that data block.

Alternate method: relative to the indirect region, 4300 - 3*1024 = 1228; entry = floor(1228/1024) = 1 and data-byte displacement = 1228 % 1024 = 204.

**Trap:** 4300 % (256*1024) = 4300 is *not* the byte offset inside a data block. The indirect region has capacity 256 KiB but is composed of 256 distinct 1-KiB data blocks.

## Worked example 2: maximum logical data size
B = 4096 bytes (4 KiB), P = 4 bytes; D = 10 and one single, one double, one triple pointer.

N = 4096/4 = 1024.

- Direct: 10 * 4096 = 40 KiB.
- Single: 1024 * 4096 = 4 MiB.
- Double: 1024^2 * 4096 = 4 GiB.
- Triple: 1024^3 * 4096 = 4 TiB.

Maximum theoretical logical file data size = 40 KiB + 4 MiB + 4 GiB + 4 TiB. Real implementations may have other limits.

## Worked example 3: index block overhead for a 2,000-block file
Assume B = 4096 bytes, P = 4 bytes, D = 10 direct pointers, and one single and one double pointer.

N = B/P = 1024 addresses per index block.

1. File requires **2000 data blocks**.
2. Direct pointers cover 10 data blocks. Remaining: 2000 - 10 = 1990.
3. Single-indirect pointer covers 1024 data blocks and needs **1 single-indirect index block**. Remaining: 1990 - 1024 = 966.
4. Double-indirect pointer must cover 966 data blocks. A double-indirect tree needs **1 top-level/root index block** and **ceil(966/1024) = 1 second-level index block**.
5. Total extra index blocks = 1 + 1 + 1 = **3**.
6. Total allocated blocks (data + these index blocks) = 2000 + 3 = **2003** blocks, excluding inode and all other file-system metadata.
7. Allocated storage in these blocks = 2003 * 4096 = 8,204,288 bytes. File data = 2000 * 4096 = 8,192,000 bytes.

**Subtlety:** A partially populated index block still occupies a full storage block under this model. At 966 entries, the second-level index block contains 966 pointers and 58 unused pointer slots.

## General algorithm: number of index blocks
Assume one each of single/double/triple pointers and the file uses F data blocks, with N pointers per index block and D direct pointers.

1. Direct data blocks: d = min(F, D); remaining R = max(F-D, 0).
2. Single region data: s = min(R, N). If s>0, allocate 1 single index block. Set R = R-s.
3. Double region data: t = min(R, N^2). If t>0, allocate 1 double root index block + ceil(t/N) second-level index blocks. Set R = R-t.
4. Triple region data: u = min(R, N^3). If u>0, allocate 1 triple root + ceil(u/N^2) second-level blocks + ceil(u/N) third-level blocks. Set R = R-u.
5. If R remains positive, file exceeds this inode model's capacity.

The ceilings assume a compact fill of index entries from the start of each region. Sparse files or other allocation patterns need different accounting.

## Worked example 4: single-to-double boundary
B = 1024 bytes, P = 4 bytes, D = 3. N=256.

- A file with F=259 data blocks uses 3 direct + 256 indirect; **one** index block suffices.
- A file with F=260 blocks requires 3 direct + 256 single + 1 double-indirect data block; index blocks = 1 single + 1 double root + 1 double leaf = **3**.
- Crossing this boundary increases index-block overhead by two blocks for just one extra data block.

## Worked example 5: number of disk reads
Assume the inode is already in memory, no relevant index or data blocks are in cache, and we retrieve one data block:

- Direct: 1 data-block read = **1 I/O**.
- Single: 1 index + 1 data = **2 I/Os**.
- Double: 2 index + 1 data = **3 I/Os**.
- Triple: 3 index + 1 data = **4 I/Os**.

If the inode itself must first be read from disk, add one I/O under that assumption. If index blocks are cached, fewer reads may occur. Distinguish *index blocks allocated* from *index blocks read in one lookup*.

## Recurring exam traps
1. 1 KiB = 1024 bytes, **not** 1024 kilobytes; bit vs byte is b vs B.
2. The file's logical block 0 is the first block; if there are ten direct pointers, single-indirect addressing begins at logical block 10.
3. `ceil(file_size/block_size)` gives number of data blocks for a non-sparse, fully allocated file.
4. `floor(offset/block_size)` gives logical block; `offset % block_size` gives byte position *inside* it.
5. Pointer size determines index fan-out, not the amount of data contained in the index block.
6. In double indirection, count the root index block **and** the second-level index blocks.
7. For a file with F blocks, file data capacity and allocated storage including index blocks are different.
8. If an index block is already cached, do not charge it as an additional physical disk read.

## Practice (no answers here)
**Q1.** A file system has 2-KiB blocks and 8-byte pointers. How many data blocks are addressable by a double-indirect pointer?

**Q2.** An inode contains eight direct pointers, one single-indirect pointer, one double-indirect pointer. Blocks are 1 KiB; pointers are 4 bytes. How many index blocks are required for a file with 400 data blocks (ignore other metadata)?

**Q3.** Using B=1 KiB, P=4 bytes, D=3, find the addressing region, relevant index entry or entries, and byte displacement for file byte offset 300,000, assuming the file is large enough.

**Q4.** A fully allocated file has 10,000 data blocks, B=4 KiB, P=4 bytes, D=12, and one single and one double pointer. How many additional index blocks are allocated?

**Q5.** A target data block is reached by triple indirection, the inode is already in memory, and the second-level and third-level index blocks are cached but the top-level index and data block are not. How many storage block reads are needed?
