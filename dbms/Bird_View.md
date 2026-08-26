DATABASE MANAGEMENT SYSTEM
│
├── 1. DATABASE FUNDAMENTALS & ARCHITECTURE ✅
│   │
│   ├── Data
│   │   └── Raw facts
│   │
│   ├── Information
│   │   └── Processed / meaningful data
│   │
│   ├── Database
│   │   └── Organized collection of related data
│   │
│   ├── DBMS
│   │   └── Software used to manage databases
│   │
│   ├── File System vs DBMS
│   │   ├── Redundancy
│   │   ├── Consistency
│   │   ├── Security
│   │   ├── Concurrency
│   │   └── Backup / Recovery
│   │
│   ├── Advantages / Disadvantages of DBMS
│   │
│   ├── Database Users
│   │   ├── DBA
│   │   ├── Database Designer
│   │   ├── Application Programmer
│   │   └── End User
│   │
│   ├── Three-Schema Architecture
│   │   ├── External / View Level
│   │   ├── Conceptual / Logical Level
│   │   └── Internal / Physical Level
│   │
│   ├── Data Independence
│   │   ├── Physical Data Independence
│   │   └── Logical Data Independence
│   │
│   ├── Schema vs Instance
│   │
│   └── DBMS Languages
│       ├── DDL
│       │   ├── CREATE
│       │   ├── ALTER
│       │   ├── DROP
│       │   └── TRUNCATE
│       │
│       ├── DML
│       │   ├── SELECT
│       │   ├── INSERT
│       │   ├── UPDATE
│       │   └── DELETE
│       │
│       ├── DCL
│       │   ├── GRANT
│       │   └── REVOKE
│       │
│       └── TCL
│           ├── COMMIT
│           ├── ROLLBACK
│           └── SAVEPOINT
│
│
├── 2. DATA MODELS & ER MODEL ✅
│   │
│   ├── Data Models
│   │   ├── Hierarchical Model → Tree
│   │   ├── Network Model → Graph
│   │   ├── Relational Model → Tables
│   │   └── Object-Oriented Model
│   │
│   ├── ER Model
│   │   ├── Entity
│   │   ├── Entity Set
│   │   ├── Attribute
│   │   └── Relationship
│   │
│   ├── ER Symbols
│   │   ├── Rectangle → Entity
│   │   ├── Double Rectangle → Weak Entity
│   │   ├── Ellipse → Attribute
│   │   ├── Double Ellipse → Multivalued
│   │   ├── Dashed Ellipse → Derived
│   │   ├── Underlined Attribute → Key
│   │   └── Diamond → Relationship
│   │
│   ├── Types of Attributes
│   │   ├── Simple
│   │   ├── Composite
│   │   ├── Single-valued
│   │   ├── Multivalued
│   │   ├── Derived
│   │   └── Key Attribute
│   │
│   ├── Strong Entity
│   ├── Weak Entity
│   │
│   ├── Mapping Cardinality
│   │   ├── 1 : 1
│   │   ├── 1 : N
│   │   ├── N : 1
│   │   └── M : N
│   │
│   ├── Generalization
│   │   └── Bottom-up
│   │
│   ├── Specialization
│   │   └── Top-down
│   │
│   └── Aggregation
│
│
├── 3. RELATIONAL MODEL & KEYS ✅
│   │
│   ├── Relation → Table
│   ├── Tuple → Row
│   ├── Attribute → Column
│   ├── Domain → Allowed values
│   ├── Degree → Number of columns
│   ├── Cardinality → Number of rows
│   │
│   ├── Keys
│   │   ├── Super Key
│   │   ├── Candidate Key
│   │   ├── Primary Key
│   │   ├── Alternate Key
│   │   ├── Composite Key
│   │   └── Foreign Key
│   │
│   ├── Key relationship
│   │
│   │   Super Key
│   │      ↓
│   │   Candidate Key
│   │      ↓
│   │   Primary Key
│   │
│   ├── Integrity Constraints
│   │   ├── Domain Constraint
│   │   ├── Key Constraint
│   │   ├── Entity Integrity
│   │   │   └── Primary Key ≠ NULL
│   │   │
│   │   └── Referential Integrity
│   │       └── FK → existing PK or NULL
│   │
│   └── NULL
│       ├── NULL ≠ 0
│       ├── NULL ≠ empty string
│       ├── IS NULL
│       └── IS NOT NULL
│
│
├── 4. RELATIONAL ALGEBRA ✅
│   │
│   ├── Fundamental Operations
│   │   │
│   │   ├── Selection σ
│   │   │   └── Select ROWS
│   │   │
│   │   ├── Projection π
│   │   │   └── Select COLUMNS
│   │   │
│   │   ├── Union ∪
│   │   ├── Set Difference −
│   │   ├── Cartesian Product ×
│   │   └── Rename ρ
│   │
│   └── Derived Operations
│       ├── Intersection
│       ├── Join
│       │   ├── Natural Join
│       │   ├── Theta Join
│       │   ├── Equi Join
│       │   └── Outer Join
│       │
│       └── Division
│
│
├── 5. SQL ✅
│   │
│   ├── SQL Basics
│   │   ├── Declarative language
│   │   └── DDL / DML / DCL / TCL
│   │
│   ├── SELECT
│   ├── FROM
│   ├── WHERE
│   ├── DISTINCT
│   ├── ORDER BY
│   ├── LIKE
│   ├── NULL handling
│   │
│   ├── Aggregate Functions
│   │   ├── COUNT()
│   │   ├── SUM()
│   │   ├── AVG()
│   │   ├── MIN()
│   │   └── MAX()
│   │
│   ├── GROUP BY
│   ├── HAVING
│   │
│   ├── WHERE vs HAVING
│   │
│   ├── Logical Query Execution Order
│   │
│   │   FROM
│   │     ↓
│   │   WHERE
│   │     ↓
│   │   GROUP BY
│   │     ↓
│   │   HAVING
│   │     ↓
│   │   SELECT
│   │     ↓
│   │   ORDER BY
│   │
│   ├── Joins
│   │   ├── INNER JOIN
│   │   ├── LEFT JOIN
│   │   ├── RIGHT JOIN
│   │   └── FULL OUTER JOIN
│   │
│   └── Subqueries
│       ├── IN
│       ├── EXISTS
│       ├── NOT EXISTS
│       └── SELECT 1 / SELECT 2 idea
│
│
├── 6. NORMALIZATION ✅
│   │
│   ├── Why Normalization?
│   │   ├── Reduce redundancy
│   │   └── Remove anomalies
│   │
│   ├── Anomalies
│   │   ├── Insertion Anomaly
│   │   ├── Update Anomaly
│   │   └── Deletion Anomaly
│   │
│   ├── Functional Dependency
│   │   └── X → Y
│   │
│   ├── Dependency Types
│   │   ├── Full Functional Dependency
│   │   ├── Partial Dependency
│   │   └── Transitive Dependency
│   │
│   ├── Prime Attribute
│   ├── Non-Prime Attribute
│   │
│   └── Normal Forms
│       │
│       ├── 1NF
│       │   └── Atomic values
│       │
│       ├── 2NF
│       │   ├── Must be 1NF
│       │   └── No Partial Dependency
│       │
│       ├── 3NF
│       │   ├── Must be 2NF
│       │   └── No Transitive Dependency
│       │
│       └── BCNF
│           └── Every determinant must be
│               a Candidate Key
│
│
├── 7. TRANSACTIONS & ACID ✅
│   │
│   ├── Transaction
│   │   └── Logical unit of work
│   │
│   ├── Operations
│   │   ├── Read(X)
│   │   ├── Write(X)
│   │   ├── Commit
│   │   └── Rollback
│   │
│   ├── ACID
│   │   │
│   │   ├── A → Atomicity
│   │   │   └── All or nothing
│   │   │
│   │   ├── C → Consistency
│   │   │   └── Valid state → valid state
│   │   │
│   │   ├── I → Isolation
│   │   │   └── Concurrent transactions behave independently
│   │   │
│   │   └── D → Durability
│   │       └── Committed data survives failure
│   │
│   ├── Transaction States
│   │   ├── Active
│   │   ├── Partially Committed
│   │   ├── Committed
│   │   ├── Failed
│   │   └── Aborted
│   │
│   └── Concurrency Problems
│       ├── Dirty Read
│       ├── Lost Update
│       ├── Unrepeatable Read
│       └── Phantom Read
│
│
├── 8. CONCURRENCY CONTROL & SERIALIZABILITY ✅
│   │
│   ├── Schedule
│   │   │
│   │   ├── Serial Schedule
│   │   │
│   │   └── Non-Serial / Concurrent Schedule
│   │
│   ├── Serializable Schedule
│   │   │
│   │   ├── Conflict Serializable
│   │   │
│   │   └── View Serializable
│   │
│   ├── Conflicting Operations
│   │   │
│   │   ├── R-W → conflict
│   │   ├── W-R → conflict
│   │   ├── W-W → conflict
│   │   └── R-R → NO conflict
│   │
│   │   Conditions:
│   │   ├── Different transactions
│   │   ├── Same data item
│   │   └── At least one WRITE
│   │
│   ├── Conflict Serializability Test
│   │   │
│   │   └── Precedence / Serialization Graph
│   │       │
│   │       ├── No Cycle
│   │       │   └── Conflict Serializable ✅
│   │       │
│   │       └── Cycle
│   │           └── Not Conflict Serializable ❌
│   │
│   ├── View Serializability
│   │   ├── Initial Read
│   │   ├── Read-From relation
│   │   └── Final Write
│   │
│   ├── Recoverability
│   │   │
│   │   ├── Recoverable Schedule
│   │   │
│   │   ├── Cascadeless Schedule
│   │   │
│   │   └── Strict Schedule
│   │   │
│   │   └── Strength:
│   │
│   │       Strict
│   │          ↓
│   │       Cascadeless
│   │          ↓
│   │       Recoverable
│   │
│   ├── Locks
│   │   │
│   │   ├── Shared Lock (S)
│   │   │   └── READ
│   │   │
│   │   └── Exclusive Lock (X)
│   │       └── WRITE
│   │
│   ├── Lock Compatibility
│   │
│   │        Request
│   │        S     X
│   │   S   YES   NO
│   │   X   NO    NO
│   │
│   ├── Two-Phase Locking — 2PL
│   │   │
│   │   ├── Growing Phase
│   │   │   └── Acquire locks
│   │   │
│   │   └── Shrinking Phase
│   │       └── Release locks
│   │
│   │   Key:
│   │   └── 2PL guarantees Conflict Serializability
│   │
│   ├── Strict 2PL
│   │   └── Exclusive locks held until commit
│   │
│   └── Deadlock
│       ├── T1 waits for T2
│       ├── T2 waits for T1
│       └── Wait-for Graph cycle
│
│
├── 9. INDEXING, FILE STORAGE & RAID ✅
│   │
│   ├── File Organization
│   │   ├── Ordered / Sequential
│   │   ├── Unordered / Heap
│   │   └── Hashed
│   │
│   ├── Index
│   │
│   ├── By relation with file ordering
│   │   │
│   │   ├── Primary Index
│   │   │   ├── File ordered on KEY
│   │   │   └── Usually Sparse
│   │   │
│   │   ├── Clustering Index
│   │   │   ├── File ordered on NON-KEY
│   │   │   └── Usually Sparse
│   │   │
│   │   └── Secondary Index
│   │       ├── File NOT ordered on indexed field
│   │       └── Usually Dense
│   │
│   ├── By Density
│   │   ├── Dense Index
│   │   └── Sparse Index
│   │
│   ├── Primary Key vs Primary Index
│   │
│   ├── Why not index every column?
│   │   ├── Faster SELECT
│   │   └── Slower INSERT / UPDATE / DELETE
│   │
│   ├── B-Tree
│   │
│   ├── B+ Tree
│   │   ├── Internal nodes → navigation
│   │   ├── Data pointers → leaf nodes
│   │   ├── Leaves linked
│   │   └── Excellent for Range Queries
│   │
│   ├── B-Tree vs B+ Tree
│   │
│   ├── Hash Index
│   │   ├── Key → Hash Function → Bucket
│   │   ├── Excellent for =
│   │   └── Poor for range
│   │
│   ├── Hash Collision
│   ├── Overflow Bucket
│   ├── Static Hashing
│   ├── Dynamic Hashing
│   │   ├── Extendible Hashing
│   │   └── Linear Hashing
│   │
│   ├── B+ Tree vs Hash
│   │
│   │   Equality:
│   │      Hash ✅
│   │      B+ Tree ✅
│   │
│   │   Range:
│   │      Hash ❌
│   │      B+ Tree ✅
│   │
│   └── RAID
│       │
│       ├── RAID 0
│       │   └── Striping + ZERO redundancy
│       │
│       ├── RAID 1
│       │   └── Mirroring
│       │
│       ├── RAID 5
│       │   └── Distributed parity
│       │
│       ├── RAID 6
│       │   └── Double parity
│       │
│       └── RAID 10
│           └── Mirroring + Striping
│
│
└── 10. DATA WAREHOUSING & DATA MINING 🟡
    │
    ├── Data Warehouse ✅
    │   │
    │   ├── Central analytical repository
    │   ├── Historical data
    │   ├── Multiple data sources
    │   │
    │   └── Bill Inmon Characteristics
    │       ├── Subject-Oriented
    │       ├── Integrated
    │       ├── Time-Variant
    │       └── Non-Volatile
    │
    ├── ETL ✅
    │   ├── Extract
    │   ├── Transform
    │   └── Load
    │
    ├── OLTP vs OLAP ✅
    │   │
    │   ├── OLTP
    │   │   └── Transactions
    │   │
    │   └── OLAP
    │       └── Analysis
    │
    ├── Data Mart ✅
    │
    ├── Data Mining ✅
    │   │
    │   ├── Classification
    │   │   └── Predefined classes
    │   │
    │   ├── Clustering
    │   │   └── Discover groups
    │   │
    │   ├── Association
    │   │   └── Market Basket Analysis
    │   │
    │   └── Regression
    │       └── Predict numeric value
    │
    ├── Dimensional Modelling ✅
    │   │
    │   ├── Fact Table
    │   │   └── Numerical / measurable data
    │   │
    │   ├── Dimension Table
    │   │   └── Descriptive/context data
    │   │
    │   ├── Measure
    │   └── Grain
    │
    ├── Warehouse Schemas ✅
    │   │
    │   ├── Star Schema
    │   │   ├── ONE Fact Table
    │   │   ├── Denormalized Dimensions
    │   │   └── Fewer joins / faster
    │   │
    │   ├── Snowflake Schema
    │   │   ├── ONE Fact Table
    │   │   ├── Normalized Dimensions
    │   │   └── More joins
    │   │
    │   └── Fact Constellation / Galaxy
    │       ├── MULTIPLE Fact Tables
    │       └── Shared Dimensions
    │
    ├── Slowly Changing Dimensions ✅
    │   │
    │   ├── Type 0 → No change
    │   ├── Type 1 → Overwrite
    │   ├── Type 2 → New Row / Full History
    │   └── Type 3 → New Column / Limited History
    │
    ├── OLAP Cube ✅
    │   └── Multidimensional analytical view
    │
    ├── OLAP Operations ✅
    │   │
    │   ├── Roll-Up
    │   │   └── Detail → Summary
    │   │
    │   ├── Drill-Down
    │   │   └── Summary → Detail
    │   │
    │   ├── Slice
    │   │   └── Fix ONE dimension
    │   │
    │   ├── Dice
    │   │   └── Filter MULTIPLE dimensions
    │   │
    │   └── Pivot
    │       └── Rotate/reorient view
    │
    │
    ├── MOLAP / ROLAP / HOLAP ⬜
    ├── Three-Tier Warehouse Architecture ⬜
    ├── Metadata ⬜
    ├── KDD Process ⬜
    ├── Support / Confidence / Lift ⬜
    │
    ├── NoSQL ⬜
    │   ├── Key-Value
    │   ├── Document
    │   ├── Column-Family
    │   └── Graph
    │
    ├── CAP Theorem ⬜
    │   ├── Consistency
    │   ├── Availability
    │   └── Partition Tolerance
    │
    ├── Distributed Databases ⬜
    │   ├── Fragmentation
    │   ├── Replication
    │   └── Transparency
    │
    └── Big Data V's ⬜