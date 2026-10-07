	
# DBMS Interview Notes — Topic-wise (High → Low Priority)

---

### CTE : Common Table Expression is a temporary result which you can use further to perform operations

## Tier 1 

### 1. SQL Queries & Joins

**Definition:** SQL is the language used to define, query, and manipulate relational data. A join combines rows from two or more tables based on a related column.

**Real-world example:** An e-commerce app has `Orders` and `Customers` tables. To show "customer name + their order total," you join on `customer_id`.

**Query example:**

```sql
SELECT c.name, o.order_id, o.amount
FROM Customers c
INNER JOIN Orders o ON c.customer_id = o.customer_id;
```

- INNER JOIN → only matching rows.
- LEFT JOIN → all left rows + matched right rows (NULL if no match).
- RIGHT JOIN → mirror of LEFT.
- FULL OUTER JOIN → all rows from both sides.
- SELF JOIN → table joined with itself (e.g., employee–manager hierarchy). **Follow-up details (asked constantly):**
- **`WHERE` vs `HAVING`** — `WHERE` filters rows _before_ grouping; `HAVING` filters groups _after_ `GROUP BY`/aggregation.
    
    ```sql
    SELECT dept, COUNT(*) FROM Employees
    WHERE status = 'active'      -- filters rows first
    GROUP BY dept
    HAVING COUNT(*) > 5;         -- filters groups after
    ```
    
- **Subquery vs Join** — a subquery is a query nested inside another (runs first, returns a value/set); a join combines tables directly in one pass. Joins are usually faster for large datasets; subqueries are clearer for existence checks or single-value comparisons.
    
    ```sql
    -- Subquery: customers who placed an order
    SELECT name FROM Customers
    WHERE customer_id IN (SELECT customer_id FROM Orders);
    
    -- Same result via join
    SELECT DISTINCT c.name FROM Customers c
    JOIN Orders o ON c.customer_id = o.customer_id;
    ```
    
- **`UNION` vs `UNION ALL`** — both stack results of two queries with the same column structure. `UNION` removes duplicate rows (extra sort/dedup cost); `UNION ALL` keeps all rows, including duplicates, and is faster.
    
    ```sql
    SELECT city FROM Customers
    UNION ALL
    SELECT city FROM Suppliers;   -- keeps duplicates, no dedup overhead
    ```
    

---

### 2. Normalization (1NF → BCNF)

**Definition:** Process of organizing tables to reduce data redundancy and avoid update/insert/delete anomalies, by splitting data into related tables.

**Real-world example:** A single `Orders` table storing customer name + address in every row wastes space and risks inconsistency if the customer moves — split into `Customers` and `Orders` linked by `customer_id`.

**Levels (know all four cold):**

- **1NF:** atomic column values, no repeating groups.
- **2NF:** 1NF + no partial dependency (non-key attribute depends on the _whole_ composite key).
- **3NF:** 2NF + no transitive dependency (non-key attribute doesn't depend on another non-key attribute).
- **BCNF:** every determinant is a candidate key (stricter 3NF).

**BCNF in detail (classic interview trap — 3NF passes but BCNF fails):**

A table can satisfy 3NF and still have redundancy if a **non-candidate-key attribute determines another attribute** that is itself part of a candidate key. BCNF closes this gap.

**Formal rule:** For every functional dependency `X → Y` in the table, `X` must be a **super key** (a candidate key or a superset of one). If some `X` that determines other columns is _not_ a candidate key, the table violates BCNF.

**Classic example:** `StudentAdvisor(student_id, subject, advisor)`

- Rule: each student can study multiple subjects, but each subject is taught by only one advisor, and each advisor teaches only one subject (1:1 between subject and advisor).
- Functional dependencies: `(student_id, subject) → advisor` and `advisor → subject`.
- Candidate key: `(student_id, subject)`.
- Check 3NF: `advisor` is a prime-adjacent value here and the dependency `advisor → subject` involves `subject`, which _is_ part of the candidate key — because the only "violation" is a non-key attribute (`advisor`) determining part of the key (`subject`), 3NF's exception clause ("Y is a prime attribute") lets this slide. **So this table is in 3NF.**
- Check BCNF: `advisor → subject` — but `advisor` alone is **not** a candidate key (it doesn't determine `student_id`). This violates BCNF's stricter rule (every determinant must be a full candidate key).

**Decomposition to fix it:** Split into two tables:

```sql
-- Advisor uniquely determines subject
CREATE TABLE AdvisorSubject (
  advisor VARCHAR(50) PRIMARY KEY,
  subject VARCHAR(50)
);

-- Student-advisor mapping
CREATE TABLE StudentAdvisor (
  student_id INT,
  advisor VARCHAR(50),
  PRIMARY KEY (student_id, advisor),
  FOREIGN KEY (advisor) REFERENCES AdvisorSubject(advisor)
);
```

Now every determinant (`advisor` in the first table, `(student_id, advisor)` in the second) is a candidate key — BCNF satisfied.

**Trade-off to mention if asked:** BCNF decomposition is not always **dependency-preserving** — sometimes you lose the ability to check a functional dependency without a join, so in practice 3NF is often chosen over BCNF when dependency preservation matters more than eliminating every last bit of redundancy.

**Follow-up details:**

- **2NF violation example:** `Orders(order_id, product_id, product_name, quantity)` where `(order_id, product_id)` is the composite key — `product_name` depends only on `product_id` (part of the key), not the whole key → violates 2NF. Fix: move `product_name` to a separate `Products` table.
- **3NF violation example:** `Employees(emp_id, dept_id, dept_name)` — `dept_name` depends on `dept_id`, which itself depends on `emp_id` (a transitive dependency, non-key → non-key). Fix: split into `Employees(emp_id, dept_id)` and `Departments(dept_id, dept_name)`.
- **When denormalization is fine:** read-heavy systems (analytics dashboards, reporting/OLAP) where join costs hurt performance more than redundancy hurts consistency — you trade some redundancy for fewer joins and faster reads.

---

### 3. ACID Properties & Transactions

**Definition:** A transaction is a sequence of operations executed as one logical unit. ACID guarantees it's processed safely.

- **Atomicity:** all-or-nothing (bank transfer: debit + credit both happen, or neither).
- **Consistency:** DB moves from one valid state to another (constraints never violated).
- **Isolation:** concurrent transactions don't see each other's intermediate state.
- **Durability:** once committed, survives a crash (written to disk/log).

**Real-world example:** Transferring ₹500 between two bank accounts — if the debit succeeds but the credit fails, atomicity rolls back both.

**Query example:**

```sql
BEGIN TRANSACTION;
UPDATE Accounts SET balance = balance - 500 WHERE acc_id = 1;
UPDATE Accounts SET balance = balance + 500 WHERE acc_id = 2;
COMMIT;
```

**Follow-up details:**

- **Crash mid-transaction:** the DBMS uses a transaction log (write-ahead log) recording every change before it's applied. On restart, it replays the log — committed transactions are redone (roll-forward), and incomplete/uncommitted ones are undone (rollback) — so the DB returns to a consistent state.
- **`COMMIT` vs `ROLLBACK`:** `COMMIT` permanently saves all changes made in the transaction (durability kicks in). `ROLLBACK` undoes all changes made since the transaction began, as if it never happened.

    ```sql
    BEGIN TRANSACTION;
    UPDATE Accounts SET balance = balance - 500 WHERE acc_id = 1;
    -- something goes wrong / balance check fails
    ROLLBACK;   -- undo the debit, nothing is saved
    ```
    
- **Savepoints:** allow partial rollback within a larger transaction (`SAVEPOINT sp1; ... ROLLBACK TO sp1;`) without undoing everything.

---

### 4. Keys (Primary, Foreign, Candidate, Composite, Unique)

**Definition:** Keys enforce uniqueness and relationships between tables.

- **Primary key:** uniquely identifies each row; can't be NULL.
- **Candidate key:** any column set that _could_ be a primary key.
- **Foreign key:** column referencing another table's primary key — enforces referential integrity.
- **Composite key:** primary key made of 2+ columns.
- **Unique key:** enforces uniqueness but allows one NULL.

**Real-world example:** In `Enrollment(student_id, course_id)`, neither column alone is unique, but the pair is → composite primary key; `student_id` is a foreign key referencing `Students`.

**Follow-up details — deleting a referenced row:**

- **`ON DELETE CASCADE`** — deleting the parent row auto-deletes all matching child rows. Example: deleting a `Course` also deletes its `Enrollment` rows.
- **`ON DELETE RESTRICT` (or `NO ACTION`)** — blocks the delete if child rows still reference it; you must delete children first.
- **`ON DELETE SET NULL`** — child rows' foreign key is set to `NULL` instead of deleting them (column must be nullable). Example: deleting an `Employee` sets `manager_id = NULL` for their reports instead of deleting the reports.
    ```sql
    ALTER TABLE Enrollment
    ADD CONSTRAINT fk_course FOREIGN KEY (course_id)
    REFERENCES Courses(course_id) ON DELETE CASCADE;
    ```
---

### 5. Indexing

**Definition:** A data structure (typically B-Tree or Hash) that speeds up row lookups so the DB doesn't scan every row.

**Real-world example:** Searching for a user by `email` in a 10-million-row `Users` table — without an index, it's a full table scan; with one, it's near-instant lookup.

**Query example:**

```sql
CREATE INDEX idx_email ON Users(email);
```

- **Clustered index:** physically sorts table data by the index key (one per table).
- **Non-clustered index:** separate structure pointing to row locations (many per table).
- **Trade-off:** faster reads, but slower writes (index must be updated) + extra storage.

**Follow-up details:**

- **When NOT to index:** low-cardinality columns (e.g., a `gender` column with only 2-3 distinct values — the index barely narrows the scan), small tables (full scan is already fast), columns updated very frequently (write overhead outweighs read benefit), or columns rarely used in `WHERE`/`JOIN`/`ORDER BY`.
- **B-Tree vs Hash index:** B-Tree keeps keys sorted, so it supports range queries (`BETWEEN`, `<`, `>`, `ORDER BY`) and equality — it's the default in most DBs. Hash index only supports exact-match (`=`) lookups, is faster for pure equality checks, but can't do range scans.

---

## Tier 2 

### 6. Concurrency Control & Deadlocks

**Definition:** Mechanisms ensuring multiple simultaneous transactions don't corrupt data.

**Real-world example:** Two users booking the last movie seat at the same time — locking prevents both from succeeding.

- **Locking:** shared (read) lock vs exclusive (write) lock.
- **Deadlock:** Transaction A holds lock on X, waits for Y; Transaction B holds lock on Y, waits for X → both stuck. DBMS detects via a wait-for graph and kills one (victim selection). **Follow-up details:**
- **Preventing deadlocks:**
    - _Lock ordering:_ always acquire locks on resources in the same global order across all transactions (e.g., always lock lower `account_id` first) so circular waits can't form.
    - _Timeouts:_ a transaction waiting too long for a lock is automatically aborted and retried.
    - _Deadlock detection:_ DBMS periodically checks the wait-for graph for cycles and kills one transaction (the "victim") to break the cycle.
    -
- **Optimistic vs Pessimistic locking:**
    - **_Pessimistic**:_ lock the row upfront before reading/updating, assuming conflicts are likely the 
	    means while updating the data at say row 5, that row will be blocked until the transaction finishes.
	    (e.g., `SELECT ... FOR UPDATE`). Safer under heavy contention, but reduces concurrency.
	    
	- **_Optimistic**:_  **No locking at all, at any point.** Everyone reads and works on the row freely and simultaneously.
		- Transaction A reads the row (say, `version = 3`).
		- Transaction A does its logic, then tries to write back.
		- At write time, it checks: "Is `version` still 3?" If yes → write succeeds, bump to `version = 4`. If someone else already changed it (version is now 4) → **reject A's update**, and A must retry

---
### 7. Transaction Isolation Levels

###  What is isolation?

```
Isolation ensures that concurrently running transactions don't see each other's _intermediate, uncommitted_ state — each transaction behaves as if it were running alone, even though it's actually running alongside others.
```

**Definition:** Controls how much one transaction's changes are visible to others before commit — trades consistency for performance.

**The bellow are the isolation levels**

| Level            | Dirty Read | Non-Repeatable Read | Phantom Read |
| ---------------- | ---------- | ------------------- | ------------ |
| Read Uncommitted | Possible   | Possible            | Possible     |
| Read Committed   | Prevented  | Possible            | Possible     |
| Repeatable Read  | Prevented  | Prevented           | Possible     |
| Serializable     | Prevented  | Prevented           | Prevented    |
|                  |            |                     |              |

**Real-world example:** Read Committed (default in PostgreSQL/Oracle) is common for typical apps; Serializable is used where correctness (e.g., financial ledgers) outweighs throughput.
#### What happens if isolation levels are weak??
that will lead to the following causes

- **Dirty read:** Transaction A reads a row that Transaction B has modified but not yet committed. If B rolls back, A has read data that never really existed. _Example: A reads a balance updated mid-transfer by B; B then rolls back the transfer — A used a value that was never real._

- **Non-repeatable read:** Transaction A reads the same row twice, but Transaction B updates and commits that row in between → A gets two different values in the same transaction. _Example: A reads a product's price twice during checkout; B updates the price in between the two reads._

- **Phantom read:** Transaction A runs the same query twice, but Transaction B inserts/deletes rows matching the query's condition in between → A sees a different _set_ of rows (not just a changed value). _Example: A counts "orders with status = pending" twice; B inserts a new pending order in between — the count changes._

---

### 8. ER Model & Relationships

**Definition:** Entity-Relationship model represents real-world entities and how they relate, before converting to tables.

- **1:1** — one passport per person.
- **1:N** — one teacher, many students.
- **M:N** — students enroll in many courses, courses have many students (needs a junction table).

**Real-world example:** `Students` and `Courses` are M:N, resolved via `Enrollment(student_id, course_id)`.

**Follow-up details:**

- **ER diagram → relational tables (conversion rules):**
    - Each strong entity → its own table, attributes become columns, identifier becomes primary key.
    - 1:1 or 1:N relationship → add a foreign key on the "many" (or either) side.
    
    - M:N relationship → create a **junction/bridge table** holding foreign keys from both entities (e.g., ``Enrollment(student_id, course_id)``).
    
    - Multi-valued attributes → become their own separate table.
- **Weak entity vs strong entity:** a strong entity has its own primary key and exists independently (e.g., `Students`). A weak entity has no standalone primary key and depends on a strong entity for identification (e.g., `Dependents` of an employee — identified only via `emp_id + dependent_name`), linked via an identifying relationship.

---
### 9. Query Optimization

**Definition:** Techniques to make queries run faster — the DB's query planner chooses an execution path; you can guide it.

**Real-world example:** A slow dashboard query filtering `WHERE status = 'pending'` on a million-row table — adding an index on `status`, or checking the query plan, can cut runtime drastically.

- *Use Where clause before grouping to filter the record earlier*
- *use `LIM`  to get only range of records*
- *Don't use `SELECT * ...` for everything, use only the columns needed*
- Use Indexing to optimize the retrieval performance of the query

**Query example:**

```sql
EXPLAIN SELECT * FROM Orders WHERE status = 'pending';
```

**Key ideas:** avoid `SELECT *`, index frequently filtered/joined columns, avoid functions on indexed columns in `WHERE`, prefer `EXISTS` over `IN` for large subqueries.

---

## Tier 3 — Common, especially backend/product-company rounds

### 10. Views, Triggers, Stored Procedures

**Definition:**

- **View:** a saved virtual table defined by a query (doesn't store data itself).
- **Trigger:** code that auto-runs on an INSERT/UPDATE/DELETE event.
- **Stored procedure:** precompiled reusable SQL logic block.

**Real-world example:** A trigger that auto-updates `last_modified` timestamp whenever a row changes; a view that exposes only non-sensitive columns to a reporting team.

```sql
CREATE VIEW ActiveUsers AS
SELECT id, name FROM Users WHERE is_active = TRUE;
```

---

### 11. Denormalization & OLTP vs OLAP

**Definition:** Denormalization intentionally adds redundancy to speed up reads (reverses normalization).

- **OLTP** (Online Transaction Processing): the operations which will have many small read/write transactions, highly normalized. Example: banking app.
	*High transaction, small operation*
	
- **OLAP** (Online Analytical Processing): The process which does complex read-heavy aggregations, often denormalized. Example: sales analytics dashboard. grouping and joining large data from different tables reduces the performance.
	*High transaction, High operation*
	

**Likely follow-ups:** when denormalization is justified (reporting systems, read-heavy caches).

---

### 12. SQL vs NoSQL & CAP Theorem

SQL Data Bases are relational databases which store the data in row and column format
No-sql are Non-relational databases which is used to store semi and unstructured data

#### Main use of NO-SQL Databases
- *Flexibility* : it can be highly scalable horizontally
- *Large* : it can store large amount of data, specially designed for handling the data of increasing network usage users
#### Types
- **Document Store :** which stores the document type of data such as JSON
- **Key-Value  store** : store the values in key and values format, high speed retrieve
- **Graph - Database**: store the data interm of `nodes`(entities) and `edges`(Attributes) 
  Basically used for traversing the data, used in fraud detentions, connection recommendation and all



**CAP theorem:** a distributed system can only guarantee 2 of 3 — **C**onsistency, **A**vailability, **P**artition tolerance.

**Real-world example:** A social media "likes" counter can tolerate slight inconsistency (AP system like Cassandra) for high availability; a bank ledger needs consistency (CP system).

**Likely follow-ups:** document vs key-value vs column-family vs graph databases, when you'd pick SQL over NoSQL.

---

### 13. Database Architecture & Data Dictionary

**Definition:** The data dictionary stores metadata — table names, columns, data types, constraints, indexes — that the DBMS uses internally for parsing and optimization.

**Real-world example:** Running `DESCRIBE Orders;` in MySQL pulls from the data dictionary to show column names/types.

---

## Tier 4 — Advanced (good to mention, shows depth)

### 14. Sharding & Replication

**Definition:**

- **Sharding:** splitting a large database horizontally across multiple servers by a key (e.g., `user_id % N`).
- **Replication:** copying data across servers for availability/read-scaling (master-replica setup).

**Real-world example:** A large-scale app shards `Users` by region so no single server holds all data; read replicas serve read traffic while the primary handles writes.

---

### 15. File Organization & Storage Structures

**Definition:** How rows are physically stored on disk — heap files, sorted files, hashed files — affects retrieval speed.

**Real-world example:** A clustered index effectively sorts the physical file by key, so range queries (`WHERE age BETWEEN 20 AND 30`) are fast.

---

## Quick revision checklist

- [ ]  Explain ACID with a real transaction example
- [ ]  Write 1NF → BCNF for a sample table
- [ ]  Draw ER diagram → convert to tables (1:1, 1:N, M:N)
- [ ]  Compare clustered vs non-clustered index
- [ ]  List isolation levels + anomalies each prevents
- [ ]  Explain a deadlock scenario and how to avoid it
- [ ]  Compare OLTP vs OLAP, SQL vs NoSQL, CAP theorem

