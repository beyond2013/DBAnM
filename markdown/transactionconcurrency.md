# Transaction Processing and Concurrency Control


## Acknowledgment:

Parts of this content have been generated or refined using ChatGPT (by OpenAI) to support clarity, structure, and accessibility. All material has been reviewed and curated by the author.


## Introduction

In multi-user database environments, many transactions occur simultaneously. It is crucial to ensure that these concurrent transactions do not interfere with each other in a way that compromises the correctness of the database. This lecture covers the principles of transaction processing and the mechanisms used to ensure data consistency and isolation in the presence of concurrent access.

---

## ACID Properties

A reliable transaction management system adheres to the ACID properties:

1. **Atomicity**
   A transaction is an indivisible unit. It either completes entirely or not at all.

2. **Consistency**
   Transactions take the database from one consistent state to another, maintaining all defined rules, constraints, and relationships.

3. **Isolation**
   Concurrent transactions are isolated from each other. Intermediate states of one transaction are not visible to others.

4. **Durability**
   Once a transaction is committed, its effects are permanently recorded, even in the event of a system crash.

---

## Serializability

Serializability ensures that the outcome of executing transactions concurrently is the same as if they were executed serially (one after another).

### Types of Serializability:

* **Conflict Serializability**:
  Based on the idea that conflicting operations (read/write) can affect the outcome. A schedule is conflict-serializable if it can be transformed into a serial schedule by swapping non-conflicting operations.

* **View Serializability**:
  A more general form of serializability, focusing on whether the final view (result) of data is the same as in some serial execution.

---

## Lock-Based Protocols

Concurrency control is often enforced using locks to control access to data items.

### Types of Locks:

* **Shared Lock (S-Lock)**: Allows reading but not writing. Multiple transactions can hold a shared lock simultaneously.
* **Exclusive Lock (X-Lock)**: Allows both reading and writing. Only one transaction can hold it at a time.

### Two-Phase Locking (2PL):

A transaction follows two phases:

1. **Growing Phase**: Acquires all the locks it needs.
2. **Shrinking Phase**: Releases locks and cannot acquire new ones afterward.

This protocol guarantees conflict serializability but can lead to **deadlocks**.

---

## Timestamp-Based Protocols

Each transaction is assigned a unique timestamp when it begins. Transactions are scheduled in the order of their timestamps.

### Basic Rules:

* If a transaction tries to access data in a way that would violate timestamp ordering, it is **aborted** and restarted.

### Advantages:

* No need for locks.
* Avoids deadlocks.

### Disadvantages:

* High rate of aborts in high-contention environments.
* Overhead of managing timestamps.

---

## Deadlocks

A **deadlock** occurs when two or more transactions wait indefinitely for each other to release locks.

### Deadlock Handling Strategies:

* **Prevention**: Ensure that the system never enters a deadlock state. For example:

  * Assign priorities or timestamps to avoid circular wait.

* **Detection**: Allow deadlocks to occur but detect them using a **Wait-For Graph (WFG)** and resolve by aborting one transaction.

* **Recovery**: After detection, one or more transactions are rolled back to break the cycle.


Absolutely. Here's an extended **section at the end of the lecture** that addresses **which aspects are handled by the DBMS and which fall under the DBA's responsibilities**:

---

## Division of Responsibilities: DBMS vs. DBA

Modern relational database management systems (RDBMSs) such as MySQL, PostgreSQL, Oracle, and SQL Server come equipped with built-in mechanisms for transaction processing and concurrency control. However, the **Database Administrator (DBA)** plays a crucial role in configuring, monitoring, and tuning these mechanisms to ensure optimal performance and reliability.

### What the DBMS handles automatically:

* **ACID Enforcement**:
  Most DBMSs automatically enforce ACID properties during transaction processing, assuming developers correctly define transactions using `BEGIN`, `COMMIT`, and `ROLLBACK`.

* **Lock Management**:
  The DBMS automatically applies appropriate locks (shared or exclusive) during read and write operations, based on the isolation level set.

* **Timestamp Management (for systems using timestamp ordering)**:
  If the system uses a timestamp-based protocol, the DBMS generates and maintains timestamps internally.

* **Deadlock Detection and Resolution**:
  Many commercial DBMSs include built-in deadlock detection mechanisms and automatically abort one of the transactions involved in a deadlock.

* **Default Isolation Levels**:
  The DBMS applies a default isolation level (e.g., Read Committed, Repeatable Read) to all transactions, unless explicitly overridden.

---

### What the DBA is responsible for:

* **Choosing and Configuring Isolation Levels**:
  DBAs decide the appropriate isolation level based on the application’s need for consistency versus performance. For example:

  * Financial applications may require **Serializable**.
  * Reporting systems may tolerate **Read Committed**.

* **Monitoring Locks and Contention**:
  DBAs must monitor long-running queries and lock contention using tools like:

  * `SHOW ENGINE INNODB STATUS;` in MySQL
  * `pg_stat_activity` and `pg_locks` in PostgreSQL
  * Oracle Enterprise Manager for Oracle DB

* **Resolving Persistent Deadlocks or Blocking Issues**:
  While the DBMS may resolve individual deadlocks, recurring problems need DBA intervention to optimize queries, tune indexes, or revise application logic.

* **Configuring Transaction Logs and Recovery Settings**:
  DBAs ensure that redo logs, undo logs, and write-ahead logging mechanisms are properly configured to support durability and fast recovery after crashes.

* **Backups and Recovery Strategy**:
  The DBA is responsible for ensuring data durability by scheduling regular backups, testing recovery procedures, and managing failover and high availability strategies.

* **Performance Tuning**:
  The DBA may tune transaction throughput by adjusting parameters like:

  * Lock timeout thresholds
  * Cache sizes
  * Log file sizes
  * Concurrency settings

* **Training Developers and Enforcing Best Practices**:
  DBAs help application developers write efficient and safe transaction logic, often by reviewing code or providing guidelines.

---

## Summary Table

| Task                        | Handled by DBMS  | Handled by DBA                   |
| --------------------------- | -----------------| ---------------------------------|
| ACID enforcement            | ✅               | 🔸 Monitoring only              |
| Locking and unlocking       | ✅               | 🔸 Monitoring/Resolution        |
| Deadlock detection          | ✅               | 🔸 Resolving recurring issues   |
| Timestamp management        | ✅               | ❌                              |
| Isolation level enforcement | ✅               | ✅ Configuration                |
| Transaction log management  | ✅               | ✅ Configuration and monitoring |
| Recovery on crash           | ✅               | ✅ Planning and testing         |
| Tuning for performance      | ❌               | ✅                              |
| Developer guidance          | ❌               | ✅                              |

🔸 = Shared responsibility

---

Let me know if you'd like this added to the Word document or turned into a separate handout.


## Conclusion

Understanding transaction processing and concurrency control is critical for managing multi-user databases. The ACID properties form the foundation of reliable transactions, and concurrency is handled through serializability, locks, timestamps, and deadlock strategies. Mastery of these topics equips database administrators to maintain data integrity, performance, and user satisfaction in concurrent environments.


