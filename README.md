# 📘 Fundamentals of Database Engineering

---

## ⚙️ ACID Properties

**Transaction:**  
A group of SQL queries treated as a single unit of work.

**Atomicity:**  
All operations in a transaction must complete successfully. If one fails, the entire transaction is rolled back.

**Consistency:**  
Ensures that the database remains in a valid state before and after the transaction (e.g., referential integrity using foreign keys).

**Isolation:**  
Prevents concurrent transactions from interfering with each other.

### ➤ Read Phenomena (Problems due to low isolation):
- **Dirty Read:** Reading uncommitted changes from another transaction.
- **Non-repeatable Read:** Same query returns different results if another transaction modifies the data in between.
- **Phantom Read:** New rows appear in results when re-running the same query (e.g., new row inserted in a range).
- **Lost Update:** Two transactions overwrite each other's updates (e.g., `count++` logic).

### ➤ Isolation Levels:
- **Read Uncommitted:** Can read uncommitted data → dirty reads possible.
- **Read Committed:** Only committed data is visible → no dirty reads. (Default in most DBs.)
- **Repeatable Read:** Same row returns consistent values throughout the transaction → prevents non-repeatable reads.
- **Snapshot (MVCC):** Transaction sees a snapshot of the DB as it was at the start.
- **Serializable:** Strictest isolation → no concurrent transactions; behaves like transactions are run one after the other.

![WhatsApp Image 2025-06-22 at 10 01 17 AM](https://github.com/user-attachments/assets/c662f386-2be0-46ce-ba05-21676aa6725b)

**Durability:**  
Once a transaction is committed, its data is persisted—even after a crash or power failure.

---

## 🔄 Eventual Consistency

- Often seen in master-slave (leader-follower) setups.
- Writes go to the master; updates are pushed asynchronously to slaves.
- Reads from slaves might be slightly outdated. If you always want the latest data, make sure to read from the master.

---

## 📊 Database Indexing

**What is an Index?**  
A data structure that improves read performance by allowing faster lookups.

**Important Concepts:**
- Indexes point to rows in the heap (actual data). If data is too large or not selective, the DB may skip the index to avoid overhead.
- Indexes **don’t store all columns**, so fetching non-indexed columns requires additional table reads.

### ➤ Scan Types:
- **Index Scan:** Uses index to locate matching rows.
- **Bitmap Scan:** Used when index scan is too inefficient but full table scan is overkill.

### 🔑 Key vs Non-Key Index:
- **Non-Key Index (Covering Index):** Includes additional columns used for filtering to avoid heap reads.

### ➕ Composite Index:
- One index on multiple columns.
- Useful when filtering/sorting on multiple columns (especially with `AND` conditions).
- **Column order matters**: Index is effective from left to right.

> Use composite index when filtering/sorting with multiple columns.

⚠️ **Avoid UUID (v4) as Index:**  
Random distribution leads to poor B+Tree performance and high write overhead due to frequent rebalancing.

---

## 📂 Secondary Index

- Created on non-clustered (non-ordered) columns.
- Even though the column isn’t ordered in the table, the index itself is ordered (B+Tree).
- Comes with overhead on writes (due to B+Tree rebalancing).

---

## 📁 Database Partitioning

**What is Partitioning?**  
Splitting a large table horizontally into smaller chunks (partitions) based on a column value (e.g., date range).

**Benefits:**
- Queries can target specific partitions → improves performance.
- Indexes are created **per partition**, making scans faster if pruning is enabled.

**Notes:**
- Partition key should be **NOT NULL**.
- Indexes must be defined per partition.
- **Enable partition pruning** to let the DB skip unnecessary partitions.
- Updating rows that move across partitions is costly.

---

## 🔀 Database Sharding

**What is Sharding?**  
Dividing one large database into multiple smaller databases (shards), each with a portion of the data.

**Sharding Key:**  
Used to determine which shard holds the data (via consistent hashing).

**Partitioning vs Sharding:**
- **Partitioning:** Splits one table across multiple internal partitions in the same DB.
- **Sharding:** Splits entire DB into smaller independent databases.

**Pros:**
1. Scalability
2. Smaller indexes
3. Security and access control

**Cons:**
1. No transaction support across shards
2. Difficult to update schema
3. Hard to do joins across shards

> Use sharding **only when absolutely needed** due to its complexity.

---

## 🔒 Locks & Concurrency

- **Exclusive Lock:** No one else can read or write the locked row.
- **Shared Lock:** Others can read but not write.

### 🔁 Double Booking Problem (e.g., seat booking system)
- Use `SELECT ... FOR UPDATE` to avoid race conditions and make sure the first transaction that locks the ticket gets it.

---

## 📄 Pagination

- **Offset-based pagination is slow** for large datasets because DB must scan the (`OFFSET` + `LIMIT`) number of rows and then discard `OFFSET` rows before returning the `LIMIT` results.
- Consider **keyset pagination** (using a `WHERE id > last_seen_id`) for better performance.

---

## 🌐 Database Replication

**Master-Slave Replication:**
- **Master:** Handles all writes and syncs data to slaves.
- **Slaves:** Serve read-only traffic (can lag behind master).

**Types:**
- **Asynchronous:** Slaves eventually catch up.
- **Semi-Async:** Master waits for at least one slave to confirm.
- **Synchronous:** Master waits for all slaves (rare, slow).

**Pros:**
1. Horizontal scaling
2. Region based queries - DB per region

**Cons:**
1. Eventual consistency
2. Slow synchronous writes

---

## 🆚 SQL vs NoSQL

| Feature | **SQL (Relational DB)** | **NoSQL (Non-relational DB)** |
|--------|--------------------------|-------------------------------|
| **Data Model** | Structured (tables, rows, columns) | Flexible (documents, key-value, graph, etc.) |
| **Schema** | Fixed schema (predefined structure) | Dynamic schema (schema-less) |
| **Query Language** | SQL (Structured Query Language) | Varies by DB (e.g., MongoDB uses JSON-like queries) |
| **ACID Compliance** | Strong | Often relaxed; prefers eventual consistency |
| **Scalability** | Vertical (scale up: stronger server) | Horizontal (scale out: more servers) |
| **Joins** | Powerful support for joins | Limited or no joins (denormalization is preferred) |
| **Use Case** | Complex queries, transactional systems (e.g., banking) | High scalability, fast reads/writes, flexible data (e.g., social media) |

---

## 🧠 Concurrency Control

- **Optimistic Locking:** No locks; uses a version column to detect conflicts.
- **Pessimistic Locking:** Locks the row to prevent conflicts (`FOR UPDATE`).
