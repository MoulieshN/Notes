# SQL JOINs – Complete Guide

## What is a JOIN
A JOIN is used to combine rows from two or more tables based on a related column.

---

## Left vs Right Table Rule
- The table written **before JOIN** is the **LEFT table**
- The table written **after JOIN** is the **RIGHT table**

Example:
```sql
SELECT *
FROM A
LEFT JOIN B ON A.id = B.id;
```

## INNER JOIN

Returns only the rows that have matching values in both tables.

```sql
SELECT *
FROM A
INNER JOIN B ON A.id = B.id;
```

## LEFT JOIN (LEFT OUTER JOIN)

Returns all rows from the LEFT table and matching rows from the RIGHT table.
If no match exists, RIGHT table columns are NULL.

```sql
SELECT *
FROM A
LEFT JOIN B ON A.id = B.id;
```

## RIGHT JOIN (RIGHT OUTER JOIN)

Returns all rows from the RIGHT table and matching rows from the LEFT table.

```sql
SELECT *
FROM A
RIGHT JOIN B ON A.id = B.id;
```

## Equivalent LEFT JOIN:

```sql
SELECT *
FROM B
LEFT JOIN A ON A.id = B.id;
```

## FULL JOIN (FULL OUTER JOIN)

Returns all rows from both tables.
Non-matching rows contain NULL values.

```sql
SELECT *
FROM A
FULL OUTER JOIN B ON A.id = B.id;
```

## MySQL FULL JOIN Workaround
```sql
SELECT *
FROM A
LEFT JOIN B ON A.id = B.id
UNION
SELECT *
FROM A
RIGHT JOIN B ON A.id = B.id;
```

## CROSS JOIN

Returns the Cartesian product of both tables.
```sql
SELECT *
FROM A
CROSS JOIN B;
```

## SELF JOIN

A table joined with itself using aliases.

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```