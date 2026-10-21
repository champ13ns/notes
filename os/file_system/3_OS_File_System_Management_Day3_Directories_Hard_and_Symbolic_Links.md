# Operating Systems — File System Management
## Day 3: Directory Structures, Hard Links and Symbolic Links

**Target:** IBPS RRB SO IT Officer / IBPS SO IT / conceptual GATE preparation  
**Language:** English  
**Prerequisites:** File, inode, file descriptor, file offset, block mapping  
**Revision principle:** Understand the diagrams before memorizing definitions. Save calculations for a later practice session.

---

## 1. What is a directory?

A directory organizes names of filesystem objects. In traditional Unix-style file systems, a directory is a special type of file containing **directory entries** that associate names with inode numbers.

Example:

```text
/home/sachin/                 Directory entries
    a.txt                    a.txt    -> inode 169
    b.txt                    b.txt    -> inode 170
    projects/                projects -> inode 180
```

- A **directory entry** associates a name with an object (typically an inode number in Unix-style systems).
- An **inode** stores metadata such as type, permissions, ownership, timestamps, size and information for locating file content (traditional pointers or modern extents, depending on filesystem).
- A **data block** stores contents, which for a directory are entries rather than the text of a user document.
- Directories have their own inodes, permissions and storage allocation.
- A file name is not normally stored inside the traditional Unix inode.

### Path resolution: `/home/sachin/a.txt`

```text
Root (/) directory
  find "home"   -> inode of /home
  find "sachin" -> inode of /home/sachin
  find "a.txt"  -> inode 169
  use inode 169 -> file mapping -> requested bytes
```

The OS must resolve the path before it can use an inode to locate the file's data. Permissions and mounts can influence pathname traversal.

**Exam trap:** The pathname is not a disk address. A directory entry is not the file's entire data-block map.

---

## 2. Absolute and relative paths

- **Absolute path:** begins with `/`, resolved from filesystem namespace root, e.g. `/home/sachin/projects/app.js`.
- **Relative path:** resolved relative to the current working directory, e.g. `projects/app.js` when the current directory is `/home/sachin`.
- `.`: current directory; `..`: parent directory (subject to root/mount behavior).
- `~`: commonly expanded by a Unix shell to a home directory; it is *not* an operator interpreted by the kernel's pathname resolution.

```bash
pwd
cd /home/sachin/projects
cd ..
pwd   # /home/sachin
```

### Directory permissions (Unix-style)

- `r` on a directory generally allows reading/listing its names (subject to other access controls).
- `x` allows searching/traversing it when a component name is known.
- `w` together with the required permissions typically allows altering entries (create/delete/rename), subject to sticky bits, ACLs, and other constraints.

**Trap:** Directory read permission and traversal/execute permission are different.

---

## 3. Directory structure models

### 3.1 Single-level directory

```text
Root/
  a.txt
  notes.txt
  photo.jpg
```

One directory for all entries; straightforward but filenames must be unique within it, making grouping and multiuser naming difficult.

### 3.2 Two-level directory

```text
Master Directory
  User_A/
    notes.txt
  User_B/
    notes.txt
```

Each user has a separate directory. Identical leaf filenames are fine across directories; arbitrary nested subdirectories are not part of the strict two-level model.

### 3.3 Tree-structured directory

```text
/
  home/
    sachin/
      projects/
        app.js
    rahul/
  etc/
```

Hierarchical organization and nesting; a file has a path through its parent directories. Sharing through links makes the resulting namespace more than a strict tree.

### 3.4 Acyclic-graph directory

A shared object can have multiple incoming references while the graph has no directed cycles.

```text
/home/sachin/a.txt ------+
                         +----> inode 169
/home/rahul/shared.txt --+
```

Hard links to regular files provide such sharing. Ordinary Unix systems normally disallow user-created hard links to directories to preserve manageable hierarchy.

### 3.5 General-graph directory

Following references may introduce cycles (symbolic links are a practical example).

```text
A --> B --> C
^           |
+-----------+
```

Recursive traversals must account for loops; systems also impose symlink-resolution limits.

**Exam focus:** Know the differences between single-level, two-level, tree, DAG, general graph; distinguish a hard link to a file from symbolic traversal cycles.

---

## 4. What is a hard link?

A hard link creates **another directory entry pointing to the same inode** on the same filesystem. It does not create another set of file data blocks.

```bash
printf 'HELLO\n' > a.txt
ln a.txt b.txt
ls -li a.txt b.txt
```

Conceptual state:

```text
a.txt ------+
            +--> inode 169 --> file data blocks
b.txt ------+
```

After `ln`, both names refer to the same underlying file. Changing file contents through one name changes what is read through the other.

```bash
printf 'UPDATED\n' > a.txt
cat b.txt                    # UPDATED
```

### Link count

Traditional Unix inodes have a link count that counts directory entries (hard links) referring to the inode.

```text
Initial: a.txt -> inode 169; link count = 1
Add b.txt -> inode 169; link count = 2
Remove a.txt; link count = 1
Remove b.txt; link count = 0
```

When the last hard link is removed, storage can normally be reclaimed **once no open references remain**. Therefore unlinking a name need not instantly make an opened file inaccessible to a process that already has a descriptor.

```c
int fd = open("a.txt", O_RDONLY);
unlink("a.txt");
/* fd may still be used to read the already-open file. */
close(fd);
```

### Hard-link restrictions

- Hard links generally cannot cross filesystem boundaries, because the inode number has meaning within its filesystem.
- Normal user operations do not create hard links to directories on Unix-like systems.
- A hard link is not a new independent copy of the file.

**Trap:** The *link count* is not the number of currently open descriptors. An inode can have link count 1 but be opened by many processes.

---

## 5. What is a symbolic link (soft link)?

A symbolic link is a **separate filesystem object** whose contents provide a pathname to resolve. It has its own inode on inode-based filesystems.

```bash
printf 'HELLO\n' > a.txt
ln -s a.txt shortcut.txt
ls -li a.txt shortcut.txt
ls -l shortcut.txt
readlink shortcut.txt      # a.txt
```

Conceptually:

```text
shortcut.txt --> symlink inode 200 --> stores target path "a.txt"
                                           |
                                           v  path resolution
                                      a.txt -> inode 169 -> data
```

- `a.txt` and `shortcut.txt` have **different inodes**.
- Using `shortcut.txt` normally follows its stored target pathname and accesses the target's content.
- The symbolic link itself does not increment the target inode's hard link count.
- Symbolic links can point to directories, can cross file systems, and can target nonexistent paths.
- If the target path becomes invalid, the symlink becomes **dangling** (broken).

**Relative symlink rule:** If a symlink stores a relative target such as `a.txt`, resolve it relative to the **directory containing the symlink**, not relative to the process's current directory.

Example:

```bash
mkdir -p demo
printf 'HELLO\n' > demo/a.txt
ln -s a.txt demo/shortcut.txt
cat demo/shortcut.txt     # HELLO
```

### Delete/rename comparison

```bash
printf 'HELLO\n' > a.txt
ln a.txt hard.txt
ln -s a.txt soft.txt
rm a.txt
cat hard.txt   # HELLO: inode still has a hard link
cat soft.txt   # fails: stored path a.txt is missing
```

For a regular file opened through a symbolic link, the open operation typically follows the symlink and returns a descriptor to the target file; special flags can alter following behavior.

---

## 6. Hard link versus symbolic link

| Property | Hard link | Symbolic link |
|---|---|---|
| What is created? | New directory entry for the target inode | New link object holding a pathname |
| Same inode as target? | Yes | No |
| Shares target contents? | Yes (same file) | Resolves target pathname to access contents |
| Increments target hard-link count? | Yes | No |
| Can cross filesystem boundaries? | Generally no | Yes |
| Can point to directory? | Normally disallowed for users | Yes |
| Target original name deleted? | Surviving hard link still works | Link may become dangling |
| Can point to absent target? | No | Yes |

**Important nuance:** A symlink whose target pathname is `a.txt` might later resolve to a *different file* if `a.txt` is removed and replaced. A hard link continues to refer to the original inode as long as that inode exists.

---

## 7. Connection to file descriptors and open file descriptions

These are three different layers:

```text
Directory entry "a.txt" -> inode 169 -> storage mapping/data
                          ^
                          |
Process FD 3 -> open file description (offset = 120)
Process FD 4 -> separate open file description (offset = 0)
```

- **Directory entry:** how pathname resolves to a file object.
- **Inode:** metadata and block mapping for that file object.
- **File descriptor:** process-local integer referring to an open instance.
- **Open file description:** kernel-side open state and current offset; can be shared by `dup()` or `fork()`.

Removing a pathname is different from closing a descriptor. The former updates directory entries and link count; the latter releases the calling process's open reference.

---

## 8. Exam traps to memorize *after* understanding

1. File name is generally stored in a directory entry, not a traditional Unix inode.
2. An inode number is filesystem-local; the same number can exist in different filesystems.
3. A hard link is not a duplicate copy and has no separate target data mapping.
4. A soft link has its own inode and stores a pathname (or equivalent target representation).
5. Removing the original filename does not invalidate another hard link.
6. Removing a symlink does not delete its target file.
7. Removing the last hard link may leave data accessible through an already-open descriptor until its open references are gone.
8. A symlink can become dangling; a hard link to a regular file cannot point to a nonexistent inode.
9. `ln` creates a hard link; `ln -s` creates a symbolic link.
10. `.` and `..` are directory traversal components; `~` is generally shell expansion.
11. A directory itself has metadata and typically an inode.
12. Directory `x` permission is about search/traversal, not executing a directory as a program.

---

## 9. Concept checks (attempt later)

**Q1.** In an inode-based filesystem, two directory names refer to the same inode. Which of the following is necessarily true?  
A. They have independent contents  
B. They are hard links to the same underlying file  
C. One must be a symbolic link  
D. Both names must be in the same directory

**Q2.** You create `ln -s report.txt shortcut.txt`, then remove `report.txt` without replacing it. What happens when you try to read `shortcut.txt`?  
A. The link still contains the original file contents  
B. The link automatically creates a new file  
C. Target lookup fails because the link is dangling  
D. The target inode stays alive solely because the symlink exists

**Q3.** Which statement describes a directory on a traditional Unix-style filesystem?  
A. It has no inode  
B. It must contain consecutive blocks  
C. Its entries relate names to inode numbers  
D. It stores all the bytes of each child file

**Q4.** A process opens `a.txt`, retains its file descriptor, and another process unlinks the only filename for the same inode. Assuming a Unix-like filesystem and no other failures, what normally remains possible?  
A. Reading through the existing FD  
B. Resolving `a.txt` by its former name  
C. Accessing the file via a newly invented hard link after inode reclamation  
D. Assuming the name is still listed in its directory

**Q5.** In `/work/links/link` (a symlink containing relative target `../files/data.txt`), relative target lookup starts from:  
A. the calling process's current directory  
B. the root directory regardless of the stored string  
C. the directory `/work/links` containing the symlink  
D. the file system's device root

### Answer key — open only after practicing

1. **B:** Both entries identify the same inode, so they are hard links to the same underlying object.
2. **C:** The link stores the path and no longer finds a file at that pathname.
3. **C:** Directory data identifies child names and inode numbers in traditional Unix-style systems.
4. **A:** The open description still holds the file accessible even after its last name is unlinked.
5. **C:** A relative symlink target is interpreted relative to the containing directory.

---

## 10. Revision checklist

- [ ] I can explain how pathname traversal locates an inode.
- [ ] I can distinguish pathname, inode number, FD, file offset and storage block number.
- [ ] I know five standard directory organization models.
- [ ] I can draw both hard-link and symbolic-link diagrams.
- [ ] I understand what happens on modifying, deleting and renaming a target.
- [ ] I understand link count versus open-file reference count.
- [ ] I know how relative symlink targets are resolved.

**Next session:** File allocation methods — contiguous, linked, and indexed allocation; FAT; fragmentation; and allocation numericals. Do not confuse file *access* methods with file *allocation* methods.
