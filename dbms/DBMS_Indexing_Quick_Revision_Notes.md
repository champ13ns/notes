# DBMS Indexing — Quick Revision Notes

> **IBPS SO IT Revision Sheet**  
> Topic: **Indexing, File Organization, B/B+ Trees, Hash Indexing**

---

## 1. Big Picture

Indexes can be classified in different ways.

### Based on the relationship with physical/logical file ordering
F
```text
                    INDEX
                      |
      ---------------------------------
      |               |               |
   Primary        Clustering       Secondary
    Index           Index            Index
      |               |               |
File ordered     File ordered     File NOT ordered
on KEY field     on NON-KEY       on this index field
                 field
      |               |               |
Usually sparse   Usually sparse    Usually dense
```

### Based on index density

```text
Index Density
   |
----------------
|              |
Dense         Sparse
```

### Based on index organization/data structure

```text
Index Organization
   |
--------------------------------
|              |               |
Flat/Ordered   B/B+ Tree       Hash
               Ordered         Unordered by key
```

> **Important:**  
> `Primary / Clustering / Secondary` and `Dense / Sparse` are **not competing classifications**.  
> They describe different properties of an index.

---

# 2. Ordered File vs Unordered File

## Ordered File

Records are logically stored according to some ordering field.

Example:

```text
101
102
103
104
109
110
```

If the ordering field is `RollNo`, the records/pages are maintained in RollNo order.

---

## Unordered File

Records are not maintained according to the search-key order.

Example:

```text
110
103
101
111
...
```

This is often called a **heap/unordered file organization**.

> Physical ordering in DBMS theory means the **logical order of records/pages managed by the DBMS**, not necessarily consecutive hard-disk sectors.

---

# 3. Primary Index

A **Primary Index** is created when the data file itself is ordered on a **key field**.

Example: `Student` table ordered by `RollNo`.

```text
Block B1
101 Aman
102 Ravi
103 Neha

Block B2
104 Tina
105 Raj
106 Sam

Block B3
107 John
108 Ana
109 Tom
```

Here:

```text
RollNo = key field + ordering field
```

A sparse primary index can be:

```text
101 -> B1
104 -> B2
107 -> B3
```

The first key of each block acts as an index entry.

---

## Why can a Primary Index usually be sparse?

Because the actual data file is already ordered.

Suppose we search for:

```text
RollNo = 105
```

Index:

```text
101 -> B1
104 -> B2
107 -> B3
```

Since:

```text
104 <= 105 < 107
```

we know `105` must be in `B2`.

Then we search only inside `B2`.

---

# 4. Dense Index

A **Dense Index** contains an entry for every search-key value.

If the indexed field is unique, this effectively means one entry per record.

Example:

```text
101 -> R1
102 -> R2
103 -> R3
104 -> R4
105 -> R5
106 -> R6
107 -> R7
```

For a non-unique field, one key may point to multiple matching records.

Example:

```text
CSE -> [R1, R4, R9]
ECE -> [R2, R8]
```

### Advantages

- Very direct lookup.
- Does not require the underlying data file to be physically ordered on that field.

### Disadvantages

- Larger index.
- More storage.
- More maintenance during INSERT/UPDATE/DELETE.

---

# 5. Sparse Index

A **Sparse Index** contains entries only for some search-key values, commonly one per block/range.

Example:

```text
101 -> B1
104 -> B2
107 -> B3
```

Actual records:

```text
101
102
103
104
105
106
107
108
109
```

Only:

```text
101, 104, 107
```

appear in the index.

### Advantage

- Much smaller index.

### Disadvantage

- After locating the relevant block/range, DBMS may still need to search inside that block.

### Important Requirement

A sparse index generally requires the underlying data to be **ordered on the indexed/search field**.

---

# 6. Dense vs Sparse

| Dense Index                      | Sparse Index                            |
| -------------------------------- | --------------------------------------- |
| Entry for every search-key value | Entries for only some search-key values |
| Larger index                     | Smaller index                           |
| More direct lookup               | May require an additional block search  |
| Can work on unordered data       | Usually requires ordered data           |
| More maintenance                 | Less index storage                      |

### Shortcut

```text
Dense  = many index entries
Sparse = gaps between index entries
```

---

# 7. Clustering Index

A **Clustering Index** is created when the file is ordered/grouped on a **non-key field**.

Example: Student table physically/logically grouped by `Department`.

```text
Block B1:
CSE   Aman
CSE   Ravi
CSE   Neha

Block B2:
ECE   Tina
ECE   Sam

Block B3:
ME    John
ME    Ravi
```

`Department` is not unique.

Many rows can have:

```text
Department = CSE
```

A clustering index may look conceptually like:

```text
CSE -> first CSE block/record
ECE -> first ECE block/record
ME  -> first ME block/record
```

### Key Point

```text
Ordering field is NON-KEY
```

Therefore duplicates are expected.

---

# 8. Primary Index vs Clustering Index

## Primary Index

File ordered on a **key field**.

```text
101
102
103
104
```

Each key identifies a unique row.

## Clustering Index

File ordered/grouped on a **non-key field**.

```text
CSE
CSE
CSE
ECE
ECE
ME
```

Duplicates are allowed.

### Memory Rule

```text
Ordering field unique/key?
        |
   -------------
   |           |
  Yes          No
   |           |
Primary      Clustering
 Index         Index
```

---

# 9. Secondary Index

A **Secondary Index** is created on a field that does **not determine the physical/logical ordering of the file**.

Suppose the Student table is ordered by `RollNo`, but we frequently search using `Name`.

We can create:

```text
Name Index

Aman    -> R101
Neha    -> R102
Raj     -> R103
Sachin  -> R104
Tina    -> R105
```

This is a **Secondary Index**.

### Why is it usually dense?

Because the underlying file is not ordered on the secondary field.

We cannot infer the location of missing values by ranges, so the index generally needs entries/pointers for all indexed values.

> It is **not** dense merely because duplicate names can exist.  
> It is dense mainly because the data file is **not ordered on that field**.

For duplicates, the index can use a list/set of row pointers.

Example:

```text
Sachin -> [R104, R210, R998]
```

---

# 10. One Physical Ordering, Multiple Secondary Indexes

A table can have only one main physical/logical row ordering at a time.

It cannot simultaneously be stored as:

```text
sorted by RollNo
AND
sorted by Name
AND
sorted by Salary
```

However, we can create multiple secondary indexes:

```text
Index on Name
Index on Email
Index on Phone
Index on Department
Index on Salary
```

---

# 11. Primary Key vs Primary Index

Do **not** confuse these.

## Primary Key

A logical constraint:

```text
UNIQUE
+
NOT NULL
```

## Primary Index

A physical/access structure used to retrieve records efficiently.

They are related in textbook examples, but they are **not the same concept**.

---

# 12. Why Not Index Every Column?

Indexes improve reads but are not free.

Suppose we create indexes on:

```text
email
phone
name
city
age
department
salary
```

Now an INSERT must conceptually do:

```text
1. Insert the actual row
2. Update email index
3. Update phone index
4. Update name index
5. Update city index
6. Update age index
...
```

Similar maintenance happens for UPDATE and DELETE.

Therefore:

```text
More indexes
     ↓
SELECT often faster ✅

but

INSERT slower ❌
UPDATE slower ❌
DELETE slower ❌
More storage ❌
```

---

# 13. Why Not Store the Index as One Giant Sorted Array?

Suppose the index is:

```text
1 2 3 4 5 6 7 8 9
```

Now insert:

```text
4.5
```

In a simple contiguous sorted array, many later entries may need to move.

This becomes expensive for millions of entries.

So databases commonly use structures such as:

```text
B-Tree
B+ Tree
```

to maintain sorted indexes efficiently.

---

# 14. B-Tree

A **B-Tree** is a balanced multiway search tree.

### Storage

```text
Internal nodes:
keys + child pointers + possibly record/data pointers

Leaf nodes:
keys + record/data pointers
```

Therefore a search may finish at an internal node.

Example:

```text
              [30* | 60*]
              /         \
        [10* 20*]     [70* 80*]

* = may point to actual record
```

---

# 15. B+ Tree

A **B+ Tree** is also a balanced multiway search tree.

### Storage

```text
Internal nodes:
keys + child pointers

Leaf nodes:
keys + actual record pointers
```

Example:

```text
               [30 | 60]
              /    |    \
             /     |     \
      [10 20] -> [30 40 50] -> [60 70 80]
```

### Important Properties

- Actual record pointers are stored at **leaf nodes**.
- Internal nodes are mainly used for navigation.
- Leaf nodes are kept in sorted order.
- Leaf nodes are generally linked.
- All leaves are at the same level.
- High fan-out keeps tree height small.
- Excellent for range queries.

---

# 16. B-Tree vs B+ Tree

| Feature                         | B-Tree                                  | B+ Tree               |
| ------------------------------- | --------------------------------------- | --------------------- |
| Record pointers                 | Internal + leaf nodes                   | Leaf nodes only       |
| Internal nodes                  | Keys + child + possible record pointers | Keys + child pointers |
| Search may end at internal node | Yes                                     | Usually no            |
| Leaf nodes linked               | Not necessarily                         | Yes, generally        |
| Range queries                   | Good                                    | Excellent             |
| Fan-out                         | Lower                                   | Usually higher        |
| Tree height                     | Can be slightly higher                  | Usually lower         |
| Sequential access               | Less convenient                         | Very efficient        |

### Memory Trick

```text
B-Tree:
data pointers can appear throughout the tree.

B+ Tree:
top = directions
bottom = actual index entries/data pointers
```

---

# 17. Why B+ Tree Is Excellent for Range Queries

Suppose leaves are:

```text
[1 2 3] -> [4 5 6] -> [7 8 9] -> [10 11 12]
```

Query:

```sql
WHERE roll_no BETWEEN 4 AND 10;
```

DBMS:

```text
1. Traverses tree to find 4
2. Reaches the leaf containing 4
3. Walks through linked leaves
4. Reads 4,5,6,7,8,9,10
```

No need to traverse from the root for every value.

---

# 18. B+ Tree Insert/Delete Idea

B+ Tree keeps values ordered but avoids maintaining one giant contiguous array.

### Insert

```text
Find correct leaf
     ↓
Insert key
     ↓
If leaf is full
     ↓
Split node
     ↓
Update parent
```

### Delete

```text
Delete key
   ↓
If node becomes under-filled
   ↓
Borrow from sibling
or
Merge nodes
```

### Important

B+ Tree does **not remove** insert/delete cost.

It makes maintenance **local and efficient** using pages, splits, redistribution, and merges.

---

# 19. Fan-Out

**Fan-out** means the number of children an internal tree node can have.

Example:

```text
          [20 | 40 | 60]
          /    |    |    \
```

This node has four child pointers.

Higher fan-out means:

```text
More children per node
        ↓
Fewer tree levels
        ↓
Fewer page/disk accesses
```

B+ Trees generally have high fan-out because internal nodes do not store actual record pointers.

---

# 20. Hash Indexing

Hash indexes are mainly designed for **exact equality searches**.

Example:

```sql
WHERE roll_no = 105;
```

Conceptually:

```text
105
 ↓
Hash Function
 ↓
Bucket
 ↓
Record pointer
 ↓
Actual row
```

Example hash function:

```text
h(key) = key mod 10
```

Then:

```text
21 -> bucket 1
35 -> bucket 5
48 -> bucket 8
52 -> bucket 2
```

---

# 21. Hash Collision

A **collision** occurs when different keys map to the same bucket.

Example:

```text
h(key) = key mod 10

15 -> bucket 5
25 -> bucket 5
35 -> bucket 5
```

All three collide.

The bucket may store multiple entries:

```text
Bucket 5:
15 -> R1
25 -> R8
35 -> R20
```

Collisions are normal and must be handled.

---

# 22. Overflow Bucket/Page

If a bucket becomes full, an overflow bucket/page may be used.

Example:

```text
Bucket 5
[15,25,35,45]
       |
       ↓
Overflow
[55,65,...]
```

Too many collisions/overflow pages reduce performance.

---

# 23. Static vs Dynamic Hashing

## Static Hashing

- Fixed number of buckets.
- Simple.
- Can suffer from many overflow pages when data grows significantly.

## Dynamic Hashing

Bucket structure can grow/change as data grows.

Examples:

```text
Extendible Hashing
Linear Hashing
```

For first-pass revision, remember only the core idea.

---

# 24. Hash Index vs B+ Tree

| Feature           | Hash Index | B+ Tree                   |
| ----------------- | ---------- | ------------------------- |
| Equality `=`      | Excellent  | Excellent                 |
| Range queries     | Poor       | Excellent                 |
| `<`, `>`          | Poor       | Excellent                 |
| `BETWEEN`         | Poor       | Excellent                 |
| Ordered traversal | No         | Yes                       |
| Sequential access | Poor       | Excellent                 |
| Main organization | Buckets    | Balanced ordered tree     |
| Collision concept | Yes        | No hash-collision concept |

### Shortcut

```text
Hash:
"Give me exactly this key."

B+ Tree:
"Exact key? Fine.
Range? Fine.
Sorted traversal? Fine."
```

---

# 25. Index Organization — Final Picture

## 1. Ordered Flat/Sequential Structure

```text
1,2,3,4,5...
```

- Simple.
- Ordered.
- Expensive insertion/deletion if maintained as one large contiguous sorted structure.

---

## 2. Ordered Tree Structure

```text
B-Tree / B+ Tree
```

- Maintains keys in sorted order.
- Efficient search.
- Efficient insert/delete compared with one giant sorted array.
- B+ Tree is especially good for range queries.

---

## 3. Hash Structure

```text
key
 ↓
hash function
 ↓
bucket
```

- Does not preserve search-key ordering.
- Excellent for exact equality queries.
- Poor for range queries.

---

# 26. Important Exam Traps

### Trap 1

**Dense index does not require the data file to be ordered.**

### Trap 2

**Sparse index generally requires the data file to be ordered on the indexed field.**

### Trap 3

**Primary Key and Primary Index are different concepts.**

### Trap 4

A **Secondary Index** is usually dense because the data file is not ordered on the secondary field, not simply because duplicates exist.

### Trap 5

**B+ Tree is an ordered structure**, not an unordered structure.

### Trap 6

Hashing does **not preserve key order**, therefore it is poor for range queries.

### Trap 7

B+ Tree actual record pointers are stored at **leaf nodes**; internal nodes mainly guide the search.

### Trap 8

More indexes improve many reads but increase storage and write-maintenance cost.

---

# 27. One-Minute Revision

```text
Primary Index
→ file ordered on key field
→ usually sparse

Clustering Index
→ file ordered/grouped on non-key field

Secondary Index
→ file not ordered on that field
→ usually dense
```

```text
Dense
→ entry for every search-key value

Sparse
→ entries for only some values/blocks
→ generally requires ordered data
```

```text
B-Tree
→ data pointers can exist in internal + leaf nodes

B+ Tree
→ internal = routing
→ leaf = actual index entries/pointers
→ linked leaves
→ great for ranges
```

```text
Hash Index
→ key → hash → bucket
→ great for equality
→ poor for ranges
→ collisions possible
```

---

# 28. Final Mental Model

```text
                        DATABASE INDEXING
                               |
        ------------------------------------------------
        |                      |                       |
Relationship with file     Density               Organization
ordering
        |                      |                       |
Primary                  Dense                   Flat ordered
Clustering               Sparse                  B/B+ Tree
Secondary                                        Hash
```

The same index can have properties from multiple categories.

Example:

```text
Primary + Sparse + B+ Tree-based organization
```

or:

```text
Secondary + Dense + B+ Tree-based organization
```

or:

```text
Secondary + Dense + Hash-based organization
```

These classifications answer different questions:

```text
Primary/Clustering/Secondary
→ What is the relationship between the index and file ordering?

Dense/Sparse
→ How many search-key values have index entries?

B+ Tree/Hash
→ What data structure/organization is used to search and maintain the index?
```

File Organization -> It tell how actual data present inside database  is stored into blocks/pages.

                     DATABASE STORAGE
                           |
              ----------------------------
              |                          |
         Actual Data                  Indexes
              |                          |
     --------------------      -------------------------
     |      |      |    |      |        |       |      |
    Heap Sequential Hash Clustered    B+Tree   Hash  Multilevel


RAID -> Redundant Array of independent disks.

a. Striping -> Split across multiple disks.
b. Mirrroring -> Spliting same data across multiple disks.
c. Parity ->  Storing recovery information by using XOR operator.

RAID Levels -> 

RAID 0 -> uses striping only, no fault tolerance, no mirroring execllent peroformance
RAID 1 -> uses mirroring , min disks required : 2, 
RAID 2 -> Bit level striping + hammming code (it is used for error correction)
RAID 3 -> Byte level striping + dedicated parity risk
RAID 4 -> 
RAID 5
RAID 6
RAID 10


