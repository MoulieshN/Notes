# MySQL Architecture & Internals (HIGH PRIORITY)

### Q1. Explain MySQL architecture. What is the role of the storage engine?
### Q2. What happens internally when a SQL query is executed?
### Q3. Difference between MySQL server layer and InnoDB engine?
### Q4. What is a clustered index? Why does InnoDB require a primary key?
### Q5. How does a secondary index lookup work internally?
### Q6. What are redo logs and undo logs? When are they written?
### Q7. What is the InnoDB buffer pool? Why is it critical for performance?
### Q8. How does MySQL handle crash recovery?

👉 Expect follow-ups on page size, B+ trees, and logs.

---------

# Indexing (VERY IMPORTANT)

### Q9. How do B+ tree indexes work in MySQL?
### Q10. Difference between primary and secondary indexes in InnoDB?
### Q11. What happens if you use a UUID as a primary key?
### Q12. When will MySQL NOT use an index?
### Q13. What is a covering index?
### Q14. Composite index — how does column order matter?
### Q15. What is index selectivity?
------

# Transactions & ACID

### Q16. Explain ACID properties in MySQL with internals.
### Q17. What is MVCC? How does MySQL implement it?
### Q18. What isolation level does MySQL use by default? Why?
### Q19. Difference between READ COMMITTED and REPEATABLE READ?
### Q20. What are phantom reads? How does InnoDB prevent them?

------

# Locking & Concurrency (FREQUENTLY ASKED)

### Q21. Difference between row lock and table lock?
### Q22. What are gap locks and next-key locks?
### Q23. When does MySQL use gap locks?
### Q24. What happens when two transactions update the same row?
### Q25. How does SELECT ... FOR UPDATE work internally?
### Q26. Can SELECT block UPDATE? When?

------

# Performance & Optimization (CORE SDE-II)

### Q27. How do you debug a slow MySQL Query?
### Q28. What is EXPLAIN? What are the key fields you look at?
### Q29. Difference between EXPLAIN and EXPLAIN ANALYZE?
### Q30. What is the N+1 Query problem? How do you fix it?
### Q31. How does MySQL handle pagination internally?
### Q32. Why is LIMIT OFFSET slow for large offsets?

------

# Schema Design & Data Modeling

### Q33. How do you choose a primary key?
### Q34. When would you denormalize a schema?
### Q35. How do foreign keys affect performance?
### Q36. How do you design tables for high write throughput?
### Q37. What are soft deletes? Pros and cons?

-----

# Replication, Binlog & HA

### Q38. What is MySQL binlog?
### Q39. Difference between row-based and statement-based replication?
### Q40. What happens if a replica lags behind the master?
### Q41. How does MySQL ensure replication consistency?
### Q42. Can writes happen on replicas?

# Failure Scenarios & Edge Cases (SENIOR SIGNAL)

### Q43. What happens if MySQL crashes during a transaction?
### Q44. How do you handle duplicate writes (idempotency)?
### Q45. How do you design MySQL schema for multi-tenant systems?
### Q46. How do you avoid deadlocks?
### Q47. What happens if buffer pool is too small?
------

# SQL Practical Questions (EXPECTED)

### Q48. Find the 2nd highest salary.
### Q49. Delete duplicate rows safely.
### Q50. Find records present in table A but not in table B.
### Q51. Running total / window functions.
### Q52. Top N per group.