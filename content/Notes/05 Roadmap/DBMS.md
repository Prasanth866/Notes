---
title: "Building a DBMS in C++ — Complete Roadmap"
tags: [roadmap, cpp, systems, database, projects]
date: 2026-08-07
---

# Building a DBMS in C++ from Scratch

> **Goal:** Design and implement a resume-quality, modular Database Management System in C++20 that teaches systems programming deeply — not just a database clone, but an engineered learning project.

---

## Part 1 — Choose the Database Type

### Comparison Table

| Type | Learning Value | Difficulty | Resume Impact | Core Concepts Covered |
|---|---|---|---|---|
| **SQL Relational** | 5/5 | Hard | 5/5 | Storage, indexes, joins, query planning, transactions |
| **Key-Value Store** | 4/5 | Medium | 4/5 | LSM-trees/B-trees, WAL, compaction, bloom filters |
| **Document DB** | 3/5 | Medium | 3/5 | Schema-less storage, JSON parsing, secondary indexes |
| **Graph DB** | 3/5 | Hard | 3/5 | Graph traversal, adjacency storage, path algorithms |
| **Time-Series** | 2/5 | Medium | 2/5 | Compression, downsampling, retention policies |
| **In-Memory** | 2/5 | Easy | 2/5 | Hash maps, skip lists — misses disk/IO concepts |
| **Hybrid (KV + SQL)** | 5/5 | Very Hard | 5/5 | Everything above combined |

### Deep Dives

#### SQL Relational Database *Recommended*
- **Learning value:** Highest. Forces you to implement every layer: storage → parsing → planning → execution → concurrency.
- **Difficulty:** Hard, but milestones are well-understood (SQLite is a public reference).
- **Resume impact:** Extremely high. Interviewers immediately understand the scope.
- **Concepts:** Heap files, B+ trees, buffer pools, joins, SQL parsing, query optimization, MVCC.

#### Key-Value Store
- **Learning value:** Very high for storage-engine work specifically.
- **Difficulty:** Medium. Well-defined scope.
- **Resume impact:** High, especially if you implement LSM-trees (RocksDB-style).
- **Concepts:** Memtable, SSTable, compaction, WAL, bloom filters.
- **Weakness:** Skips SQL, query planning, and joins.

#### Document Database
- **Learning value:** Good for schema-less storage and JSON internals.
- **Difficulty:** Medium.
- **Resume impact:** Moderate.
- **Weakness:** Skips formal query optimization and relational algebra.

#### Graph Database
- **Learning value:** Good for graph algorithms and adjacency storage.
- **Difficulty:** Hard, but a different kind of hard (algorithms vs. systems).
- **Resume impact:** Niche — impressive in graph-related roles.
- **Weakness:** Misses most storage-engine and transaction fundamentals.

#### In-Memory Database
- **Learning value:** Low for systems programming — avoids the hardest parts (disk, durability).
- **Difficulty:** Easy.
- **Resume impact:** Low unless combined with persistence.

### Recommendation: SQL Relational Database

A SQL database forces you to touch *every* layer of systems programming:
- Disk I/O and binary serialization
- Buffer pool and memory management
- B+ tree indexing
- SQL parsing and query planning
- Joins and aggregations
- Transactions, WAL, and crash recovery
- Concurrency control (MVCC or 2PL)

It is the category where your learning is most transferable to real-world systems (PostgreSQL, MySQL, SQLite internals) and most legible on a resume.

---

## Part 2 — Differentiate from Existing Systems

Rather than just recreating SQLite, add **one or two focused innovations** to make your project genuinely interesting.

### Innovation 1: Adaptive Index Selection

**What:** Implement a lightweight adaptive indexing system where the database *automatically promotes* frequently-accessed columns to an index without requiring the user to manually run `CREATE INDEX`.

**How it works:**
1. Track per-column scan frequency with a lightweight counter table stored in the catalog.
2. When a column's scan count exceeds a configurable threshold, auto-generate a B+ tree index for it in the background.
3. Expose a system table (`_adaptive_indexes`) so users can inspect what was auto-indexed and why.

**Why it's realistic:** No real query optimizer required — just frequency counters + a B+ tree you're already building.

**Why it's impressive:** Adaptive indexing is a real active research area (see: Microsoft's AutoAdmin, Postgres's `pg_stat_user_tables`). You can write about it intelligently in interviews.

**Engineering value:** Forces you to design the index manager as a first-class citizen, not an afterthought.

---

### Innovation 2: Pluggable Storage Formats (Row vs. Column)

**What:** Design the storage engine with a clean `StorageFormat` interface, and implement *both* row-oriented (heap file) and columnar (PAX-style) storage. Let the user choose per-table.

**How it works:**
1. Define an abstract `Page` interface with virtual methods: `insert()`, `scan()`, `fetch_by_slot()`.
2. Implement `RowPage` (NSM — N-ary Storage Model) as the default.
3. Implement `ColumnPage` (DSM — Decomposed Storage Model / PAX) as an opt-in format using `CREATE TABLE ... STORAGE = columnar`.
4. The execution engine does not change — it works through the interface.

**Why it's realistic:** The page interface is small. Row-store is ~300 lines; column-store adds ~200 more.

**Why it's impressive:** This is the architectural distinction between OLTP (PostgreSQL) and OLAP (DuckDB) systems. Knowing why and how they differ — and having *implemented both* — is extremely rare.

**Engineering value:** Teaches the abstraction skills that define great systems engineers.

---

## Part 3 — Learning Roadmap

The roadmap is structured in **8 stages**, each with a concrete mini-project. Complete them in order — each one is a prerequisite for the next.

---

### Stage 0 — Modern C++ Foundations
**Estimated Duration:** 3–4 weeks
**Difficulty:** 2/5

#### Concepts to Learn
- RAII and deterministic resource management
- Smart pointers: `unique_ptr`, `shared_ptr`, `weak_ptr`
- Move semantics and perfect forwarding (`std::move`, `std::forward`)
- Templates and type traits
- `std::optional`, `std::variant`, `std::expected`
- Lambdas and `std::function`
- `constexpr` and compile-time evaluation
- Ranges and views (C++20)
- Threading primitives: `std::thread`, `std::mutex`, `std::atomic`

#### Mini-Project: Memory Arena Allocator
Build a fixed-size slab allocator and a general-purpose arena allocator.
- Allocate in large blocks; sub-allocate from them.
- Track live objects, support reset/reuse.
- Benchmark against `malloc/free`.

**Why it matters:** The buffer pool is essentially a managed memory arena. This teaches you how to manage raw memory safely.

**Skills gained:** Raw pointers, placement new, RAII wrappers, benchmarking.

---

### Stage 1 — File I/O and Binary Serialization
**Estimated Duration:** 2–3 weeks
**Difficulty:** 2/5

#### Concepts to Learn
- POSIX file I/O: `open`, `read`, `write`, `pread`, `pwrite`, `fsync`
- `mmap` vs. direct I/O
- Page-aligned I/O (`O_DIRECT`, `posix_memalign`)
- Endianness and byte ordering
- Binary vs. text formats
- Fixed-width vs. variable-length record encoding
- Checksums (CRC32) for data integrity

#### Mini-Project: Binary Record File
Build a simple fixed-schema record file (like a CSV but binary).
- Define a schema: `{id: u64, name: char[64], score: f64}`.
- Write and read records by page (4096-byte pages).
- Add a file header with magic bytes, version, and page count.
- Verify integrity with CRC32 checksums per page.

**Why it matters:** Everything in a database is a page on disk. This is the foundation.

**Skills gained:** Binary encoding, page layout, file headers, data integrity.

---

### Stage 2 — Storage Engine and Buffer Pool
**Estimated Duration:** 3–4 weeks
**Difficulty:** 3/5

#### Concepts to Learn
- Page layout (header, slots, records)
- Slotted page design for variable-length records
- Buffer pool architecture: frame table, page table, dirty pages
- Replacement policies: LRU, Clock, LRU-K
- Page pinning and reference counting
- Disk Manager abstraction
- Free space management (free list or bitmap)

#### Mini-Project: Buffer Pool Manager
Build a `BufferPoolManager` that:
- Manages a fixed pool of in-memory frames.
- Implements `fetch_page(page_id)` and `unpin_page(page_id, is_dirty)`.
- Uses Clock replacement to evict unpinned pages.
- Flushes dirty pages to disk via a `DiskManager`.
- Exposes `new_page()` and `delete_page()`.

**Why it matters:** The buffer pool is the most critical subsystem. Every other component (indexes, query execution, WAL) goes through it.

**Skills gained:** Memory management, cache design, abstraction layers.

---

### Stage 3 — Data Structures for Databases
**Estimated Duration:** 4–5 weeks
**Difficulty:** 3/5

#### Concepts to Learn
- B+ Tree: structure, search, insert, delete, splits and merges
- Extendible hash tables
- Skip lists
- Bloom filters
- Bitmap indexes
- Difference between internal and leaf nodes
- Duplicate key handling
- Iterator pattern on tree structures

#### Mini-Project A: B+ Tree Index
Implement an on-disk B+ Tree where:
- Nodes map to pages in the buffer pool (every node is a page).
- Leaves store `(key, RID)` pairs where `RID = (page_id, slot_id)`.
- Internal nodes store separator keys and child page pointers.
- Support `insert(key, rid)`, `delete(key)`, `search(key)`, `range_scan(lo, hi)`.
- Handle splits correctly — propagate separator keys upward.

**Why it matters:** B+ trees are the universal index structure. PostgreSQL, MySQL, SQLite, and almost every other database uses them.

#### Mini-Project B: Extendible Hash Index
Implement an extendible hash table for equality lookups:
- Directory of bucket pointers.
- Handle bucket overflow by doubling the directory and splitting buckets.
- Compare performance vs. B+ tree for point lookups.

**Skills gained:** On-disk data structures, page-based tree traversal, iterator design.

---

### Stage 4 — Catalog, Heap Files, and Schema Management
**Estimated Duration:** 2–3 weeks
**Difficulty:** 3/5

#### Concepts to Learn
- Heap file organization
- Slotted page (revisited, now with real table data)
- System catalog: storing metadata about tables, columns, indexes
- Record ID (RID) / tuple ID
- NULL handling and fixed vs. variable-length columns
- Free space management across pages

#### Mini-Project: Table Heap + System Catalog
Build a `TableHeap` backed by the buffer pool:
- `insert_tuple(tuple) -> RID`
- `fetch_tuple(rid) -> Tuple`
- `update_tuple(rid, new_tuple)`
- `delete_tuple(rid)` (mark as deleted — tombstone)
- Iterator: `TableIterator` to scan all live tuples sequentially.

Build a `Catalog` class:
- Stores `TableMetadata` (name, column names, types, index list).
- Persists catalog data in a reserved set of pages (page IDs 0–7).
- Provides `create_table()`, `drop_table()`, `get_table()`.

**Why it matters:** This is the first component that makes the storage engine feel like a real database.

**Skills gained:** Schema design, metadata management, tombstone deletion.

---

### Stage 5 — SQL Parsing and Query Representation
**Estimated Duration:** 3–4 weeks
**Difficulty:** 3/5

#### Concepts to Learn
- Lexing and tokenization
- Recursive descent parsing
- Abstract Syntax Tree (AST) design
- SQL grammar (SELECT, INSERT, UPDATE, DELETE, CREATE TABLE)
- Visitor pattern for AST traversal
- Expression trees (comparisons, arithmetic, AND/OR/NOT)

#### Mini-Project: SQL Parser
Write a handwritten recursive descent SQL parser (no parser generators).

Supported subset:
```sql
CREATE TABLE t (id INT, name VARCHAR(64), score FLOAT);
INSERT INTO t VALUES (1, 'Alice', 95.5);
SELECT id, name FROM t WHERE score > 90.0;
SELECT * FROM t WHERE id = 1 AND name = 'Bob';
UPDATE t SET score = 100 WHERE id = 1;
DELETE FROM t WHERE id = 2;
```

Output: An AST. Each statement is a node hierarchy like `SelectStatement { table, projections, filter_expr }`.

**Why it matters:** Writing a parser from scratch teaches you grammar, recursive descent, and AST design — skills that transfer to compilers, interpreters, and configuration languages.

**Skills gained:** Lexing, parsing, AST, visitor pattern.

---

### Stage 6 — Query Planning and Execution
**Estimated Duration:** 4–5 weeks
**Difficulty:** 4/5

#### Concepts to Learn
- Relational algebra: select, project, join, aggregate
- Volcano / iterator model (pull-based)
- Physical plan operators: SeqScan, IndexScan, Filter, Projection, NestedLoopJoin, HashJoin, Aggregation, Sort, Limit
- Predicate pushdown
- Rule-based query optimization
- Cost estimation (simple heuristics)
- Expression evaluation

#### Mini-Project A: Volcano Executor
Implement the Volcano iterator model:
- Each operator has `Open()`, `Next() -> Tuple | nullptr`, `Close()`.
- `SeqScan` — scans a `TableHeap` sequentially.
- `Filter` — evaluates a predicate expression on each tuple.
- `Projection` — selects and renames columns.
- `NestedLoopJoin` — basic nested loop for two tables.
- `HashJoin` — build hash table on inner, probe with outer.
- `Aggregation` — COUNT, SUM, AVG, MIN, MAX with GROUP BY.

#### Mini-Project B: Query Planner
Write a `Planner` that takes an AST and produces a physical plan:
- SELECT with no join → SeqScan + Filter + Projection.
- SELECT with index-able filter → IndexScan + Filter + Projection.
- SELECT with join → choose between NLJ and hash join.
- Add one optimization rule: predicate pushdown below joins.

**Why it matters:** This is where the database becomes queryable. Interviews almost always ask about query execution internals.

**Skills gained:** Iterator pattern, operator composition, basic optimization.

---

### Stage 7 — Transactions, Concurrency, and Recovery
**Estimated Duration:** 5–6 weeks
**Difficulty:** 5/5

#### Concepts to Learn

**Transactions:**
- ACID properties
- Transaction states: active, committed, aborted
- Isolation levels: Read Uncommitted, Read Committed, Repeatable Read, Serializable

**Concurrency:**
- Two-Phase Locking (2PL) and Strict 2PL
- Lock modes: shared (S), exclusive (X), intention locks (IS, IX, SIX)
- Deadlock detection (wait-for graph + cycle detection)
- Deadlock prevention (wait-die / wound-wait)
- MVCC: version chains, snapshot isolation, garbage collection

**Recovery:**
- Write-Ahead Logging (WAL): log before write rule
- Log records: BEGIN, COMMIT, ABORT, UPDATE(before, after), CLR
- ARIES recovery algorithm: Analysis → Redo → Undo
- Checkpoints (fuzzy checkpoints)

#### Mini-Project A: Lock Manager
Build a `LockManager`:
- Grant S and X locks per RID (tuple-level locking).
- Implement a wait queue per record.
- Build a wait-for graph; detect cycles; abort victim transaction.
- Support lock upgrade: S → X.

#### Mini-Project B: WAL and Recovery Manager
Build a WAL system:
- `LogManager` that writes structured log records to a log file.
- `log_update(txn_id, page_id, slot_id, before_image, after_image)`.
- Implement ARIES: on restart, read the log, redo all committed operations, undo all uncommitted operations.
- Test by killing the process mid-transaction and verifying recovery.

#### Mini-Project C: Transaction Manager
Tie it together:
- BEGIN → creates a Transaction object with a unique txn_id.
- Executor acquires locks through LockManager before reading/writing.
- COMMIT → writes COMMIT log record, releases all locks.
- ABORT → applies undo log records in reverse, writes ABORT record.

**Why it matters:** This is the hardest and most impressive part of any DBMS. Interviewers at database companies will ask specifically about this.

**Skills gained:** Concurrency, deadlock, logging, crash recovery, ARIES.

---

### Stage 8 — Networking and Client-Server (Optional)
**Estimated Duration:** 2–3 weeks
**Difficulty:** 3/5

#### Concepts to Learn
- POSIX sockets: socket, bind, listen, accept, send, recv
- Non-blocking I/O and epoll/kqueue
- Wire protocol design
- Thread-per-connection vs. event loop
- Connection pooling basics

#### Mini-Project: TCP Server
Build a simple TCP server:
- Accept connections on a configured port.
- Each connection reads a SQL query string, passes it to the engine, and sends back results as newline-delimited JSON.
- Write a minimal CLI client that connects and sends queries interactively.

**Why it matters:** Makes the project demable as a real service. Teaches networking fundamentals.

**Skills gained:** Sockets, wire protocols, multi-client servers.

---

## Part 4 — Final Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Client / Application                         │
└────────────────────────────────┬────────────────────────────────────┘
                                 │ SQL Query String
                      ┌──────────▼──────────┐
                      │   Network Layer      │  (optional)
                      │  TCP Server + Wire   │
                      │      Protocol        │
                      └──────────┬──────────┘
                                 │ SQL String
                      ┌──────────▼──────────┐
                      │      Parser          │
                      │  Lexer + Recursive   │
                      │  Descent → AST       │
                      └──────────┬──────────┘
                                 │ AST
                      ┌──────────▼──────────┐
                      │   Query Planner      │
                      │  AST → Physical Plan │
                      │  (Rule-based Opt.)   │
                      └──────────┬──────────┘
                                 │ Physical Plan Tree
                      ┌──────────▼──────────┐
                      │  Execution Engine    │
                      │  Volcano Iterators   │
                      │  SeqScan/IndexScan   │
                      │  Filter/Project/Join │
                      └──┬───────────────┬──┘
                         │               │
           ┌─────────────▼──┐    ┌───────▼──────────┐
           │  Index Manager  │    │ Transaction Mgr   │
           │  B+ Tree Index  │    │ BEGIN/COMMIT/ABORT│
           │  Hash Index     │    └───────┬──────────┘
           └─────────────┬──┘            │
                         │        ┌──────▼──────────┐
                         │        │  Lock Manager    │
                         │        │  S/X Locks       │
                         │        │  Deadlock Detect │
                         │        └──────┬──────────┘
                         │               │
                      ┌──▼───────────────▼──┐
                      │    Catalog           │
                      │  Table/Column/Index  │
                      │  Metadata            │
                      └──────────┬──────────┘
                                 │
                      ┌──────────▼──────────┐
                      │   Buffer Pool Mgr    │
                      │  Frame Table         │
                      │  Clock Replacement   │
                      │  Dirty Page Flush    │
                      └──────────┬──────────┘
                                 │
                      ┌──────────▼──────────┐
                      │   Recovery Manager   │
                      │  WAL / Log Manager   │
                      │  ARIES: Redo + Undo  │
                      └──────────┬──────────┘
                                 │
                      ┌──────────▼──────────┐
                      │    Disk Manager      │
                      │  Page Read/Write     │
                      │  fsync, O_DIRECT     │
                      └─────────────────────┘
```

### Module Responsibilities

#### Disk Manager
- Manages the raw database file.
- Reads and writes fixed-size pages by `page_id`.
- Allocates new pages, tracks page count in file header.
- Ensures durability via `fsync`.

#### Buffer Pool Manager
- Keeps a fixed number of frames (pages) in memory.
- Maps `page_id → frame_id` via a hash map.
- Tracks pinned pages (cannot be evicted).
- Uses Clock eviction to select victim frames.
- Writes dirty pages back to disk before eviction.

#### Recovery Manager / Log Manager
- Writes WAL log records to a separate log file before any page modification.
- Records UPDATE, BEGIN, COMMIT, ABORT, and CLR (compensation log records).
- On startup, runs ARIES: Analysis (find dirty pages + active txns) → Redo (replay log) → Undo (rollback uncommitted txns).

#### Catalog
- Stores metadata: table names, column names and types, index associations.
- Persisted in reserved pages (e.g., page IDs 0–3).
- Provides APIs to `create_table`, `drop_table`, `get_table_schema`, `add_index`.

#### Index Manager
- Creates and manages B+ tree and hash indexes.
- Associates indexes with tables via the Catalog.
- Provides `insert(key, rid)`, `delete(key)`, `search(key)`, `range_scan(lo, hi)`.
- Adaptive Index extension: monitors scan frequency and auto-creates indexes.

#### Parser
- Tokenizes the SQL string (lexer).
- Parses tokens into an AST using recursive descent.
- Supports: SELECT, INSERT, UPDATE, DELETE, CREATE TABLE, DROP TABLE.
- Returns typed AST nodes with source locations for error reporting.

#### Query Planner
- Converts the AST into a physical query plan (tree of operators).
- Consults the Catalog to resolve table and column names.
- Checks for available indexes to select IndexScan over SeqScan.
- Applies simple rule-based optimizations: predicate pushdown, projection pruning.

#### Execution Engine
- Implements the Volcano pull model: each operator has Open/Next/Close.
- Core operators: SeqScan, IndexScan, Filter, Projection, NestedLoopJoin, HashJoin, Sort, Limit, Aggregation.
- Acquires locks via the Lock Manager before reading/writing tuples.
- Interacts with the Transaction Manager to handle BEGIN/COMMIT/ABORT.

#### Transaction Manager
- Assigns unique transaction IDs.
- Tracks active transactions and their states.
- Coordinates with Lock Manager (acquire/release) and Log Manager (write log records).
- On COMMIT: flush log, release locks.
- On ABORT: apply undo log in reverse, release locks.

#### Lock Manager
- Grants S/X locks per RID at tuple granularity.
- Maintains a wait queue per resource.
- Runs deadlock detection via wait-for graph cycle detection on a background thread.
- Aborts the youngest transaction in a deadlock cycle (victim selection).

#### Network Layer (optional)
- TCP server accepting SQL queries over a socket.
- Sends results as structured responses (newline-delimited JSON or custom binary).
- Thread-per-connection model for simplicity.

---

## Part 5 — Milestones

Each milestone produces a **working, demonstrable system**.

---

### Milestone 1: Persistent Storage Engine
Complete Stage 0 + Stage 1 + Stage 2

**What you can demo:**
- Allocate pages on disk.
- Write/read pages through the buffer pool.
- Show Clock eviction working with a pool smaller than the dataset.
- Measure I/O throughput with a simple benchmark.

---

### Milestone 2: Table Storage with Indexes
Complete Stage 3 + Stage 4

**What you can demo:**
- Insert 100,000 rows into a TableHeap.
- Build a B+ tree index on the primary key.
- Point lookup: fetch by id = X using the index.
- Range scan: fetch rows where id BETWEEN 1000 AND 2000.
- Benchmark: compare full table scan vs. index scan latency.

---

### Milestone 3: Queryable SQL Database
Complete Stage 5 + Stage 6

**What you can demo:**
```sql
CREATE TABLE employees (id INT, name VARCHAR(64), salary FLOAT);
INSERT INTO employees VALUES (1, 'Alice', 120000);
INSERT INTO employees VALUES (2, 'Bob', 95000);
SELECT name, salary FROM employees WHERE salary > 100000;
SELECT COUNT(*), AVG(salary) FROM employees;
```

A real SQL interface working end-to-end.

---

### Milestone 4: ACID Transactions
Complete Stage 7

**What you can demo:**
1. Run two concurrent transactions that conflict → show one blocked and the other proceeding.
2. Insert 10 rows inside a transaction, crash the process, restart → show rows are gone (rollback recovery).
3. Insert 10 rows, commit, crash the process, restart → show rows are present (redo recovery).
4. Create a deadlock artificially → show the system detecting and aborting a transaction.

This milestone is the most impressive for resume and interviews.

---

### Milestone 5: Networked Database Server (Optional)
Complete Stage 8

**What you can demo:**
```bash
# Terminal 1
./dbmsd --port 5555 --data ./mydb

# Terminal 2
./dbms-cli --host localhost --port 5555
> SELECT * FROM employees;
```

Multiple clients connecting and querying concurrently.

---

### Milestone 6: Adaptive Indexing (Innovation 1)

**What you can demo:**
- Run a workload that repeatedly scans on `salary`.
- Show `SELECT * FROM _adaptive_indexes` to see that `salary` was auto-indexed.
- Benchmark query time before and after auto-indexing.

---

### Milestone 7: Columnar Storage (Innovation 2)

**What you can demo:**
```sql
CREATE TABLE analytics (ts INT, value FLOAT, region VARCHAR(32)) STORAGE = columnar;
```
- Insert 1 million rows.
- Run `SELECT AVG(value) FROM analytics` and benchmark against row-oriented storage.
- Show that columnar is significantly faster for aggregations (fewer I/Os).

---

## Suggested Project Structure

```
dbms/
├── CMakeLists.txt
├── src/
│   ├── storage/
│   │   ├── disk_manager.h / .cpp
│   │   ├── page.h / .cpp
│   │   ├── buffer_pool_manager.h / .cpp
│   │   ├── table_heap.h / .cpp
│   │   └── slotted_page.h / .cpp
│   ├── index/
│   │   ├── b_plus_tree.h / .cpp
│   │   ├── hash_index.h / .cpp
│   │   └── index_manager.h / .cpp
│   ├── catalog/
│   │   └── catalog.h / .cpp
│   ├── parser/
│   │   ├── lexer.h / .cpp
│   │   ├── parser.h / .cpp
│   │   └── ast.h
│   ├── planner/
│   │   └── query_planner.h / .cpp
│   ├── executor/
│   │   ├── executor.h / .cpp
│   │   ├── seq_scan.h / .cpp
│   │   ├── index_scan.h / .cpp
│   │   ├── filter.h / .cpp
│   │   ├── projection.h / .cpp
│   │   ├── nested_loop_join.h / .cpp
│   │   ├── hash_join.h / .cpp
│   │   ├── aggregation.h / .cpp
│   │   └── sort.h / .cpp
│   ├── concurrency/
│   │   ├── lock_manager.h / .cpp
│   │   └── transaction_manager.h / .cpp
│   ├── recovery/
│   │   ├── log_manager.h / .cpp
│   │   └── recovery_manager.h / .cpp
│   ├── network/
│   │   ├── server.h / .cpp
│   │   └── wire_protocol.h / .cpp
│   └── common/
│       ├── types.h
│       ├── config.h
│       └── result.h
├── tests/
│   ├── storage/
│   ├── index/
│   ├── executor/
│   └── recovery/
├── benchmarks/
│   └── bench_main.cpp
└── tools/
    └── dbms_cli.cpp
```

---

## Key Design Decisions and Trade-offs

| Decision | Choice | Trade-off |
|---|---|---|
| Page size | 4096 bytes (match OS page) | Larger pages = fewer seeks but more read amplification |
| Buffer pool replacement | Clock (approx LRU) | LRU-K is more accurate but complex |
| Index structure | B+ tree | LSM-tree is better for write-heavy; B+ is better for balanced |
| Concurrency | 2PL (strict) | MVCC has better read performance but much harder to implement |
| Logging | Physical ARIES | Logical logging is more compact but harder to implement correctly |
| Joins | NLJ + Hash Join | Sort-merge join is missing but rarely necessary for learning |
| Lock granularity | Tuple-level | Table locks are simpler; page locks are a good middle ground |
| Query optimization | Rule-based | Cost-based requires statistics, histograms, and a much larger scope |

---

## Resources

### Books
- **"Database Internals"** — Alex Petrov (best modern resource, practical)
- **"Database System Concepts"** — Silberschatz et al. (classic theory)
- **"Designing Data-Intensive Applications"** — Martin Kleppmann (broad context)

### Reference Implementations
- **SQLite** — read `btree.c` and `vdbe.c` for B-tree and execution engine
- **CMU 15-445/645 Bustub** — educational database; excellent reference architecture
- **DuckDB** — for columnar storage and modern OLAP design
- **LevelDB / RocksDB** — for LSM-tree and WAL design

### Courses
- **CMU 15-445: Database Systems** (Andy Pavlo) — free on YouTube + assignments
- **CMU 15-721: Advanced Database Systems** — grad-level, covers MVCC, vectorized execution

### Papers
- **ARIES** — Mohan et al. (1992) — the definitive WAL and recovery algorithm
- **The Volcano Model** — Graefe (1990) — iterator execution model
- **Adaptive Indexing** — Idreos et al. (2007) — "Database Cracking"

---

*Built with C++20 | No external database libraries | Single developer, production thinking*
