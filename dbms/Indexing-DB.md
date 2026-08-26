                BIG PICTUREs
        
                    INDEX
                      |
      ---------------------------------
      |               |               |
   Primary        Clustering       Secondary
    Index           Index            Index
      |               |               |
File ordered     File ordered     File NOT ordered
on KEY field     on NON-KEY       on this field
                 field
      |               |               |
Usually sparse   Usually sparse/   Usually dense
                 per group



Ordered File -> Supporse records physically appears like this 
            101
            102
            103
            104
            109
            110


Unordered File -> Suppose records physically appears like this
                110
                103
                101
                111
                ...


Physical Index -> When index is built on primary key/order key when the data file iteself is sorted accoerding to that primary key.
            Suppose : we have a student table.

            Student is physically stored as:
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

Roll No is primary key + physical ordering file.

So we can create a priamry index like 
101 -> B1
104 -> B2
107 -> B3.

This is usually called as sparse index (roughly one entry per data block).

Why can primary index be sparse     -> Because the actual data file is stored in sorted manner.

Dense Index -> A dense index has an index entry for every seach-key-value / record.
Example : 101 → R1
        102 → R2
        103 → R3
        104 → R4
        105 → R5
        106 → R6
        107 → R7

        

Sparse Index -> Index entries exist only for some records/search-key values

Example:
101 → B1
104 → B2
107 → B3

Actual records:
101
102
103
104
105
106
107
108
109

Only 101,104,107 are stored in originnal index file.
Advantage -> Smaller Index.
Disadvantage -> after finding the relevant block/range, we may still need to search inside that block

13. Dense vs Sparse
Dense	                            Sparse
Entry for every record/key	        Entries for only some
Larger index	                    Smaller index
Faster/direct lookup	            May need small additional search
Can work for secondary indexes	    Usually requires ordered data
More maintenance	                Less index storage

Clustering index -> File ordered on a non key field.
                    Suppose Student table is physically ordered by
                    Department and not roll no.

                    Example:

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

        Notice Department is not a unique key.
        Many students can have department = CSE


Secondary Index ->  Suppose our student table is physically ordered by Roll No (which is primary key only), 
                    but we often search thorugh name, so we can create another index on that field, and ofc
                    it will mostly be a dense index (since many studnets can have same name), that index is called
                    secondary index.
                    Name Index
                    Example.
                    Aman    → R101
                    Neha    → R102
                    Raj     → R103
                    Sachin  → R104
                    Tina    → R105




### Notice:

Primary/Secondary/Clustering and Dense/Sparse are not two competing classifications.
They describe different properties.
For example:
Primary Index
+
Sparse
is a common combination.

And:
Secondary Index
+
Dense
is another common combination.


### Do not confuse Primary Key with Primary Index

This is an exam trap.
A:
PRIMARY KEY
is a logical database constraint:

Unique
+
Not Null

A:
PRIMARY INDEX
is a physical/access structure used to retrieve records efficiently.

They are related in textbook examples, but they are not the same concept.

21. What is the downside of indexing?
If indexes make reads faster, why not index every column?
Because indexes are not free.
Suppose you have:
Index on:
email
phone
name
city
age
department
salary

Now insert a record:

INSERT INTO Employee (...)

DBMS has to:

1. Insert actual row
2. Update email index
3. Update phone index
4. Update name index
5. Update city index
...

Same with UPDATE/DELETE.

Therefore:

More indexes
     ↓
SELECT often faster ✅

but

INSERT slower ❌
UPDATE slower ❌
DELETE slower ❌
More storage ❌

Your notes specifically flag this as an examiner's trap: indexes speed reads but add storage and write-maintenance cost



### Summary

Primary Index

Data file is ordered on a key field; commonly sparse.

Clustering Index

Data file is ordered/grouped on a non-key field.

Secondary Index

Index exists on a field that does not determine physical file ordering; usually dense.

Dense Index

Index entry for every record/search-key value.

Sparse Index

Index entries for only some records, typically one per block/range in an ordered file.F

B-Tree
Internal nodes:
keys + possibly record pointers

Leaf nodes:
keys + record pointers
B+ Tree
Internal nodes:
only keys + child pointers

Leaf nodes:
keys + actual record pointers





INDEX ORGANIZATION

1. Ordered flat/sequential structure
   1,2,3,4,5...
   → simple
   → expensive insert/delete

2. Ordered tree structure
   B / B+ Tree
   → maintains sorted keys efficiently
   → range queries excellent

3. Unordered hash structure
   key → hash function → bucket
   → equality queries excellent
   → range queries poor

Multilevel Indexing -> creating an index over another index until the topmost index is small enough to search efficiently.
Conceptually : 
                 TOP INDEX
                |
          ----------------
          |              |
      Index Block     Index Block
          |              |
       --------        --------
       |      |        |      |
      Data   Data     Data   Data



Level 3
   ↓
Level 2
   ↓
Level 1
   ↓
Data

Numerical intuition

Suppose:
Number of data blocks = 1,000,000
Suppose each index block can contain:
100 index entries

Level 1 -> Only one entry per data block -> total blocks needed -> 1000000 / 100 = 10,000
Level 2 -> One entry per level 1 block ->  10000/100 = 100
Level 3 -> One entry per level 2 block -> 100/100 = 1 block.

Level 3 = 1 block
Level 2 = 100 blocks
Level 1 = 10,000 blocks
Data    = 1,000,000 blocks

Multilevel indexing creates higher-level indexes over lower-level indexes to reduce search cost.

File Organization -> meaning how the actual table records are arranged/stored in database pages/blocks. This is different from indexing: indexing creates an additional access structure, while file organization is about the actual data itself.