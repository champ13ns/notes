# Operating Systems — File System Management
## Day 8, Part 2: POSIX ACLs, ACL Masks, SetUID, SetGID and Sticky Bit

**Target:** IBPS RRB Scale-II SO IT Officer (medium to upper-medium), with enough conceptual depth for real Linux troubleshooting.  
**Prerequisites:** Unix permissions (`rwx`), owners/groups, `chmod`, file vs directory permissions and `umask`.  
**Notation:** Examples assume Linux/POSIX ACL support, ordinary users without overriding privileges, and no extra restrictions (such as SELinux/AppArmor policies) unless specifically stated.

---

## 1. Quick revision: base Unix permissions

A file commonly has one owner, one owning group, and nine basic permission bits divided across owner, group and others.

```text
-rw-r----- 1 sachin developers 100 report.txt
  rw- r-- ---
  user group other
```

* Owner: read and write.
* Members of the owning group: read.
* Others: no permission.

The owning group is not a list of specific users stored in the inode; users belong to groups according to the system account database, and the file metadata records the owning group ID. The Linux permission check uses a process's credentials (effective filesystem UID/GID and supplementary groups, with relevant capabilities and other security controls).

**Octal shortcuts:** `r=4`, `w=2`, `x=1`, so `7=rwx`, `6=rw-`, `5=r-x`, `4=r--`. `chmod 750 file` gives owner `rwx`, group `r-x`, others none.

### File permissions are not directory permissions

| Bit | On regular file | On directory |
|---|---|---|
| `r` | Read data | List names |
| `w` | Modify file data | Add/remove/rename entries (normally also needs `x`) |
| `x` | Execute file (subject to valid format/interpreter etc.) | Traverse/search by name |

Deleting a file generally depends on write + search permission on its **parent directory**, not file write permission. A directory's sticky bit adds a separate deletion restriction.

---

## 2. Why do we need Access Control Lists (ACLs)?

The traditional owner/group/other model is not specific enough for some policies. Suppose `report.txt` is owned by `sachin`, belongs to group `developers`, and group members have read-only access. You want one user, `rahul`, to have read-write access without allowing all developers to write.

A **POSIX Access Control List (ACL)** stores more fine-grained permissions, including named users and groups. It supplements the normal permission bits.

Example ACL displayed by `getfacl report.txt`:

```text
# file: report.txt
# owner: sachin
# group: developers
user::rw-
user:rahul:rw-
group::r--
mask::rw-
other::---
```

Interpretation:

- `user::rw-`: owner Sachin gets read and write.
- `user:rahul:rw-`: named user Rahul is assigned read and write.
- `group::r--`: owning group is assigned read.
- `mask::rw-`: the maximum effective permissions for named users and group-class entries.
- `other::---`: everyone else gets no access.

**Important:** The ACL is an access-control policy attached to the file, not a separate application index, a copy of the file's data, or a disk block allocation table.

### Common commands

```bash
getfacl report.txt
setfacl -m u:rahul:rw- report.txt
setfacl -m g:auditors:r-- report.txt
setfacl -x u:rahul report.txt
```

The named accounts/groups must actually exist. You can inspect your current identity and groups with `id` and `groups`.

---

## 3. ACL mask: the most testable part

**Effective ACL permission = assigned permission ∩ mask**, for the entries to which the mask applies.

The mask limits:

1. Named users (`user:rahul`),
2. The owning group (`group::`),
3. Named groups (`group:auditors`).

It does **not** limit:

1. The file owner (`user::`),
2. Others (`other::`).

Example:

```text
user::rw-
user:rahul:rwx        #effective:r--
group::r--
mask::r--
other::---
```

For Rahul, `rwx AND r-- = r--`; therefore Rahul can only read the file (assuming the pathname itself is traversable).

For the owner, `user::rw-` remains read-write; **do not** apply the ACL mask to the owner.

### Bitwise examples

| Assigned ACL | Mask | Effective |
|---|---|---|
| `rwx` (7) | `r--` (4) | `r--` (4) |
| `rwx` (7) | `r-x` (5) | `r-x` (5) |
| `rw-` (6) | `r-x` (5) | `r--` (4) |
| `-wx` (3) | `rw-` (6) | `-w-` (2) |

### Subtle exam trap: `ls -l` with extended ACLs

When an access ACL has a mask, the *group permission field* shown by `ls -l` represents the **ACL mask**, not necessarily the owning group's `group::` entry. A `+` may appear at the end of the mode string to show extended ACLs.

For example, `-rw-rw----+` does not prove that the owning group itself has write permission. Use `getfacl` to inspect the real entries and their effective permissions.

### Another practical trap: `setfacl` recalculates masks

On many ordinary invocations, `setfacl -m` automatically recalculates the ACL mask as a union of relevant permissions unless the mask is explicitly supplied or automatic recalculation is disabled. To make a reproducible restricted-mask example, set both explicitly:

```bash
setfacl -m u:rahul:rwx,m::r-- report.txt
getfacl report.txt
```

Or use `-n` when preserving an existing mask intentionally. Always **inspect** with `getfacl` rather than assuming the mask stayed unchanged.

---

## 4. How does the OS select the applicable ACL permission?

Simplified POSIX access ACL decision order:

1. If process identity matches the file owner: check `user::`; mask does not apply.
2. Else, if a matching named-user entry exists: check that entry **and** mask. The process does not get extra access merely because it also belongs to a group with broader permissions.
3. Else, examine matching owning-group and named-group entries; calculate permitted group-class rights under the ACL rules and apply the mask.
4. If none matched, use `other::`.

Additional real-world restrictions may come from pathname traversal permissions, capabilities, mount settings or mandatory access controls.

**Example:** Rahul has a named-user entry `user:rahul:r--`, but also belongs to `developers` and `group::rwx` with mask `rwx`. If Rahul matches the named-user entry, Rahul does not receive write via the group entry. The named-user check takes precedence.

---

## 5. Access ACL vs Default ACL

**Access ACL:** Controls permissions to the current file or directory.

**Default ACL:** An inheritance template stored on a directory; it helps determine ACLs on new children created inside it. Regular files do not carry default ACLs.

Example for an existing group `developers`:

```bash
setfacl -m d:g:developers:rwx shared/
getfacl shared/
```

New files and directories can inherit ACL entries from the directory's default ACL, subject to the creation mode and ACL rules. Newly created subdirectories can themselves inherit a default ACL.

**Important exam caveat:** If a directory has a default ACL, permissions on children are not always predicted by the simplistic `666/777 minus umask` shortcut; their ACL inheritance rules must be considered.

---

## 6. Special permission bits — overview

In addition to ordinary owner/group/other permission bits, Unix modes include three special bits:

| Special bit | Octal leading digit | Symbolic setting | Primary use |
|---|---:|---|---|
| SetUID | `4` | `chmod u+s FILE` | Effective UID of executed program |
| SetGID | `2` | `chmod g+s FILE_OR_DIR` | Effective GID on executable; group inheritance on directory |
| Sticky bit | `1` | `chmod +t DIR` | Restricts deletion/rename in shared directory |

When using four octal digits, the first digit represents special bits and the next three represent ordinary permissions:

- `4755` = SetUID + `755`
- `2755` = SetGID + `755`
- `2775` = SetGID + `775`
- `1777` = Sticky bit + `777`
- `6755` = SetUID + SetGID + `755`

Special bits can be combined using addition in that leading octal digit.

---

## 7. SetUID: effective user identity on execution

Suppose an ordinary user needs to run a carefully designed system utility that must modify a protected system file. The utility may be installed with SetUID so that, while executing, it operates with the executable file owner's **effective UID**.

Conceptual example:

```text
-rwsr-xr-x 1 root root ... /path/to/privileged-program
```

The `s` in the owner's execute position shows SetUID where owner execute is also present.

`chmod 4755 executable` requests this mode. `chmod u+s executable` can set the bit symbolically.

**It does NOT mean:**

- Everyone becomes root permanently.
- The invoking user's real UID changes to the file owner's UID.
- Every file with SetUID automatically runs with privilege.

The effect applies to executing supported binaries when OS rules permit it. Restrictions such as `nosuid` mounts, security policies, and Linux's handling of scripts or unsafe conditions may prevent the intended privilege change.

**Security:** SetUID programs must be carefully designed; granting elevated effective identity can create vulnerabilities. Do not set SetUID on arbitrary programs as an experiment.

---

## 8. SetGID: two meanings

### A. SetGID on executable files

When a supported executable with SetGID runs, its process effective GID becomes the executable file's owning group, subject to system restrictions.

Example: A utility owned by group `auditors` could execute with effective group `auditors` if permissions and security rules allow it.

### B. SetGID on directories (high-value exam concept)

A SetGID directory typically causes newly created files/subdirectories to inherit the **directory's group**, instead of the creator's ordinary default group.

Suppose:

```text
team-project/  owner=sachin, group=developers, mode=2775
```

Rahul creates `notes.txt` inside `team-project/`. If Rahul has sufficient permissions and the system behaves as described, the **new file's owning group** becomes `developers`.

This is helpful for team collaboration. Newly created subdirectories typically inherit the parent directory's SetGID bit.

Commands for a shared directory whose ownership/group is already appropriately configured:

```bash
chmod g+s team-project/
ls -ld team-project/
```

Typical display: `drwxrwsr-x` (depending on its ordinary mode bits).

**Important:** SetGID directory inheritance does not by itself guarantee that every group member has write access to every new file. The child's own mode/ACL still matters.

---

## 9. Sticky bit: deletion protection for shared directories

Imagine `/tmp` is writable by many users. Without additional restriction, ordinary directory write+execute access can be enough to remove another user's directory entry.

The **sticky bit** modifies deletion/renaming rules on a directory: ordinarily, a user may remove/rename an entry only if they own the entry, own the directory, or have sufficient privilege, even when the directory is world-writable.

Typical `/tmp` permissions:

```text
drwxrwxrwt
```

The trailing `t` indicates sticky bit with others' execute permission also present.

```bash
chmod 1777 shared-temp/
# Equivalent setting of the sticky bit on a suitably permissioned directory:
chmod +t shared-temp/
```

**Crucial distinction:** Sticky bit is about deletion/renaming directory entries. It doesn't prevent another user from editing a file if that other user independently has permission to modify its contents.

---

## 10. Reading special permission letters: lowercase vs uppercase

The displayed mode uses the execute character position to show the special bit:

| Letter | Means |
|---|---|
| `s` in owner's execute position | SetUID + owner execute |
| `S` in owner's execute position | SetUID set, owner execute not set |
| `s` in group's execute position | SetGID + group execute |
| `S` in group's execute position | SetGID set, group execute not set |
| `t` in others' execute position | Sticky + others execute |
| `T` in others' execute position | Sticky set, others execute not set |

A capital letter does **not** mean a stronger privilege. It indicates the ordinary execute/search bit for that class is absent.

---

## 11. Comparison and exam traps

| Mechanism | Problem solved | Does it change file ownership? | Important trap |
|---|---|---|---|
| Traditional `rwx` | Owner/group/other access | No | Directory `x` means traversal |
| POSIX ACL | Named-user/group access | No | Mask limits effective named-user/group-class rights |
| `umask` | Filters requested creation mode | No | It removes bits; it doesn't add them |
| SetUID executable | Runs with file owner's effective UID | No permanent login change | Not applicable indiscriminately to scripts |
| SetGID executable | Runs with file group's effective GID | No permanent login change | Distinct from directory use |
| SetGID directory | Inherit directory group for new children | New child's group set via inheritance | Doesn't guarantee group write |
| Sticky directory | Prevent other users deleting/renaming entries | No | Doesn't protect file contents from separately authorized writes |

---

## 12. Safe Linux mini-lab (no root privileges required)

These commands modify only files/directories you create in your own home directory. The SetUID/SetGID *executable* privilege behavior is not tested; that would require a carefully designed environment.

```bash
mkdir -p ~/os-permissions-lab
cd ~/os-permissions-lab
printf 'hello\n' > report.txt
chmod 640 report.txt
ls -l report.txt
id
getfacl report.txt

mkdir team-project
chmod 2775 team-project
ls -ld team-project

mkdir shared-temp
chmod 1777 shared-temp
ls -ld shared-temp
```

`setfacl` and `getfacl` may require the ACL utilities package to be installed. A named-user ACL example only works if the named user exists:

```bash
# Example if Linux account 'rahul' exists and ACL support is available:
setfacl -m u:rahul:rwx,m::r-- report.txt
getfacl report.txt
```

Use `getfacl` to confirm effective permissions and the mask. Avoid experimenting with `chmod 777` on sensitive directories and never set SetUID on untrusted executables.

---

## 13. Worked exam examples

### Example 1 — ACL mask numerical

ACL `user:rahul:rwx`, mask `r-x`. Rahul's effective rights: `rwx ∩ r-x = r-x` (read + execute, no write).

### Example 2 — ACL mask vs owner

ACL `user::rw-`, `user:rahul:rwx`, mask `r--`. Owner still has `rw-`; Rahul only has `r--`.

### Example 3 — SetGID directory

Directory owned by group `developers` with SetGID. A new child usually inherits `developers` as its owning group, even if the creator's primary group is `staff`.

### Example 4 — Sticky directory

A directory has `1777`. Rahul has directory write+execute access, but ordinarily cannot delete Sachin's entry unless he also owns the directory or has suitable privilege.

### Example 5 — Special octal bits

`chmod 4755` = SetUID + owner `rwx` + group `r-x` + others `r-x`.

### Example 6 — Uppercase S

`-rwSr-xr-x` shows a SetUID bit but the owner's execute permission is absent. The capital `S` is a display clue; it is not an extra permission.

---

## 14. Practice MCQs (IBPS medium to upper-medium)

**Q1.** An ACL contains `user::rw-`, `user:rahul:rwx`, `group::r--`, `mask::r-x`, `other::---`. What are Rahul's effective permissions?

A. `rwx`  B. `r-x`  C. `rw-`  D. `r--`

**Q2.** An ACL contains `user::rw-`, `user:rahul:rwx`, `mask::r--`. What permissions does the **file owner** have?

A. `r--`  B. `rw-`  C. `rwx`  D. `---`

**Q3.** What is the primary effect of SetGID on a directory in a typical Linux file system?

A. New children inherit the directory's owning group.  
B. All files become executable.  
C. Files are encrypted.  
D. Other users cannot list names.

**Q4.** A shared directory is owned by root and has permissions `drwxrwxrwt`. Rahul and Sachin can both write into the directory, and Rahul does not own the directory or Sachin's file. Which statement is generally correct?

A. Rahul can freely unlink Sachin's file.  
B. Rahul cannot normally unlink Sachin's file because of the sticky bit.  
C. The sticky bit prevents Rahul reading every file.  
D. The sticky bit disables group access.

**Q5.** Which octal mode represents SetUID together with ordinary permissions `rwxr-xr-x`?

A. `1755`  B. `2755`  C. `4755`  D. `6755`

**Q6.** Which statement about an extended POSIX ACL is true?

A. The mask restricts the owner entry.  
B. The mask restricts the others entry.  
C. The mask restricts named-user and group-class entries.  
D. The mask always grants execute access.

**Q7.** A directory's special bit is displayed as uppercase `T` in the others' execute position. What does that indicate?

A. Sticky bit set, others' execute bit absent.  
B. Sticky bit absent, others' execute bit set.  
C. SetGID and execute both set.  
D. SetUID and execute both set.

**Q8.** Which situation is not solved by ordinary directory SetGID alone?

A. Giving new children the directory's owning group.  
B. Making new subdirectories inherit the SetGID bit.  
C. Ensuring every group member can write every new file regardless of that file's own permission mode.  
D. Making team-folder group ownership more consistent.

### Answer key with concise reasons

1. **B** — `rwx ∩ r-x = r-x`.
2. **B** — ACL mask does not restrict `user::`.
3. **A** — SetGID directory affects group inheritance.
4. **B** — Sticky bit restricts deletion/renaming of others' entries.
5. **C** — Leading octal `4` = SetUID.
6. **C** — Named users, owning group and named groups are subject to ACL mask.
7. **A** — Capital `T` means sticky is set while others' execute is absent.
8. **C** — The child file's actual mode/ACL still determines group write permission.

---

## 15. References

- Linux ACL documentation: https://man7.org/linux/man-pages/man5/acl.5.html
- Linux `getfacl` manual: https://www.man7.org/linux/man-pages/man1/getfacl.1.html
- Linux `setfacl` manual: https://man7.org/linux/man-pages/man1/setfacl.1.html
- GNU Coreutils `chmod` manual: https://man7.org/linux/man-pages/man1/chmod.1.html
- GNU Coreutils on directory SetGID: https://www.gnu.org/software/coreutils/manual/html_node/Directory-Setuid-and-Setgid.html
- Debian Handbook, managing permissions: https://www.debian.org/doc/manuals/debian-handbook/sect.rights-management.en.html

**Next study checkpoint:** Attempt Q1–Q8 and explain especially Q2, Q4 and Q8. After checking these, consolidate complete File System Management revision notes before moving to the next OS chapter.
