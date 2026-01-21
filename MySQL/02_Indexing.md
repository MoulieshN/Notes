| S.No | Topics | Reference Link |
|------|--------|----------------|
|      | [How do B+ tree indexes work in MySQL?](#how-do-b-tree-indexes-work-in-mysql-youtube) | [Youtube: MustWatch](https://www.youtube.com/watch?v=6ZquiVH8AGU)
|      | [When will MySQL NOT use an index?](#when-will-mysql-not-use-an-index) | |
|      | [What happens if you use a UUID as a primary key?](#what-happens-if-you-use-a-uuid-as-a-primary-key) | [Planetscale-blog MUST READ](https://planetscale.com/blog/the-problem-with-using-a-uuid-primary-key-in-mysql)


# How do B+ tree indexes work in MySQL? [youtube](https://www.youtube.com/watch?v=6ZquiVH8AGU)

# When will MySQL NOT use an index?
MySQL does NOT use an index when the optimizer decides that using the index is more expensive than a table scan or when the query prevents index usage.


```
NOTE: 

MySQL will not use an index if the query has low selectivity, 
functions on indexed columns, leading wildcards, data type mismatches, 
OR conditions without full indexing, 
wrong composite index order, or when a table scan is cheaper.
```

## 1️⃣ When the Index Is Not Selective

If the index filters too many rows, MySQL prefers a full table scan.

```sql
SELECT * FROM users WHERE gender = 'M';
```
- If 90% of rows are M, index is useless.
- Low cardinality columns → index ignored.

📌 Rule of thumb:
If most rows match, index won’t be used.

## 2️⃣ Using Functions on Indexed Columns

Applying functions breaks index usage.
```sql
SELECT * FROM users WHERE YEAR(created_at) = 2024;
```

❌ Index on created_at will NOT be used

✅ Correct:
```sql
SELECT * FROM users 
WHERE created_at >= '2024-01-01' 
AND created_at < '2025-01-01';
```

## 3️⃣ Leading Wildcard in LIKE
Indexes can’t be used when the pattern starts with %.
```sql
SELECT * FROM users WHERE name LIKE '%chandra';
```
❌ Index ignored

✅ Index used only for:
```sql
LIKE 'chandra%'
```

## 4️⃣ Data Type Mismatch (Implicit Conversion)

If column type ≠ value type, index is skipped.
```sql
-- phone is VARCHAR
SELECT * FROM users WHERE phone = 9876543210;
```

❌ MySQL converts column → index unusable

✅ Correct:
```sql
WHERE phone = '9876543210';
```

## 5️⃣ Using OR Without Indexes on All Columns

If OR conditions don’t have indexes on all columns, MySQL avoids indexes.
```sql
SELECT * FROM users 
WHERE email = 'a@x.com' OR age = 30;
```

- Index on email ✔
- Index on age ❌: ➡ Full table scan

## 6️⃣ Small Tables

For very small tables, index usage is slower than scanning.

📌 MySQL heuristic:

Table scan is cheaper than index lookup + row fetch

## 7️⃣ NOT, !=, <> Conditions
Negative conditions often prevent index usage.
```sql
SELECT * FROM users WHERE status != 'active';
```

❌ Index usually ignored

## 8️⃣ ORDER BY / GROUP BY Mismatch

Index not used if sorting order differs.
```sql
SELECT * FROM users ORDER BY created_at DESC;
```

- Index on created_at ASC
- If filesort cheaper → index ignored

## 9️⃣ Composite Index – Wrong Column Order

MySQL only uses leftmost prefix.

```sql
INDEX (country, state, city)
```

❌ Query:
```sql
WHERE state = 'KA';
```

✅ Index NOT used

✅ Index used only if country is included.


----

#  What happens if you use a UUID as a primary key?
[MUST READ ARTICLE](https://planetscale.com/blog/the-problem-with-using-a-uuid-primary-key-in-mysql)
## Advantages (Pros)
- Global Uniqueness: Ensures IDs are unique across different tables, databases, and servers, ideal for distributed systems.
- Decentralized Generation: Allows generating IDs on client-side applications without hitting the database, avoiding bottlenecks.
- Security/Privacy: Prevents exposing the number of records (e.g., User #1, #2, #3) and makes URLs unguessable.
- No Single Point of Failure: Avoids issues with integer sequences in distributed environments. 
## Disadvantages (Cons)
- Performance Overhead: Random UUIDs cause B-tree index fragmentation and page splits, leading to more disk I/O and slower inserts/updates.
- Storage Intensive: Takes significantly more space (e.g., 16 bytes vs. 4-8 bytes for integers), increasing table and index size, impacting cache efficiency.
- Slower Comparisons: String/binary comparisons for UUIDs are slower than integer comparisons, impacting query performance. 

-----