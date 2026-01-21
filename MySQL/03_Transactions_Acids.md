| No  | Topics | Reference Link | 
|-----|--------|----------------|
|     | [ACID properties in MySQL with internals](#acid-properties-in-mysql-with-internals)       |                |


# How indexes makes database reads faster? [Arpit's Explanation](https://www.youtube.com/watch?v=3G293is403I)

# ACID properties in MySQL with internals
## Atomicity:
The entire transaction takes place at once or doesn't happen at all.
* Abort: If a transaction aborts, changes made to the database are not visible.
* Commit: If a transaction commits, changes made are visible.

### Undo Log

Before modifying a row, InnoDB writes the old version to the undo log

* If a transaction ROLLBACKs or crashes before commit:
    - InnoDB uses undo logs to revert changes

### Internals

Undo logs are stored in undo tablespaces

Each row contains:
  * roll_pointer → points to undo record
  * trx_id → transaction ID
## Consistency:
The integrity constraints must be maintained so that the database is consistent before and after the transaction.

### What Consistency means
    * Constraints (PK, FK, UNIQUE)
    * Triggers
    * Data types
    * Application invariants
> note: MySQL ensures consistency before and after a transaction — not during.
## Isolation:
Multiple transactions occurs independenlty without interference. Changes occuring in a particular transactions will not be visible to any other transactions until that particular change in that transaction is written to memory or has been commited.

## Durability:
Ensures that once the transaction has completed execution, then updates and modifications to database are stored in and written to disk and they persist even if a system failure occurs.

These updates now become permanent and stored in non-volatile memory.


----
# What is MVCC?
MVCC (Multi-Version Concurrency Control) allows multiple transactions to read and write the same data concurrently without blocking each other by maintaining multiple versions of a row.

👉 Readers don’t block writers, and writers don’t block readers.


```
MVCC allows concurrent transactions by maintaining multiple row versions. 
In MySQL InnoDB, it is implemented using hidden transaction IDs, 
undo logs for old versions, and read views to 
provide consistent snapshots without blocking reads.
```

## Why MVCC Is Needed

**Without MVCC:**

- Reads take locks
- Writes block reads
- High contention → poor performance

**With MVCC:**

- Reads are lock-free
- Writes create new row versions
- High concurrency with consistency

-----