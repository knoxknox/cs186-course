# Notes

1. SQL
2. Files / Buffers
3. Indexing
4. Hashing / Sorting
5. Relational Algebra
6. Query Optimization
7. DB Design
8. Transactions
9. Concurrency
10. Recovery
11. Distribution

# SQL

Resources:
- http://sqlfiddle.com/
- https://www.w3schools.com/sql/

DB:
```
Set of named Relations (Table+Schema) =>
Attributes (Field/Column) => Tuples (Row/Record)
```

SQL:
- DDL - Data Definition Language (Define and modify schema)
- DML - Data Manipulation Language (Queries can be written intuitively)

Query:
```
SELECT
  s.dept,
  COUNT(*),
  AVG(s.gpa)
FROM Students s
WHERE s.gender = 'F'
GROUP BY s.dept
HAVING COUNT(*) >= 2
ORDER BY s.dept DESC LIMIT 100;
```

SQL Evaluation:
```
FROM              - compute cross product of tables
 WHERE            - check conditions, discard tuples that fail
  SELECT          - project away columns (just keep those used in SELECT, GROUP, HAVING)
   GROUP BY       - form groups & aggregate (per group)
    [HAVING]      - filter groups (only aggregate queries)
     [DISTINCT]   - remove duplicate of rows before output
```

SQL Aggregation:
```
SELECT color, AVG(age) FROM boats GROUP by color HAVING COUNT(*) > 2
--------------------------------------------------------------------
tuples       group by       avg(), count()       having count(*) > 2
--------------------------------------------------------------------
   a 1            a 1              a 4.0 2                     b 4.3
   b 3            a 7              b 4.3 3                     c 3.7
   a 7            b 3              c 3.7 3
   c 2            b 1
   c 5            b 9
   b 1            c 2
   c 4            c 5
   b 9            c 4
--------------------------------------------------------------------
```

Set Semantics:
```
S = {a, a, b, b, b, c, e}
R = {a, a, a, a, b, b, c, d}

EXCEPT: {d}
INTERSECT: {a, b, c}
UNION: {a, b, c, d, e}

EXCEPT ALL: {a, a, d}
INTERSECT ALL: {a, a, b, b, c}
UNION ALL: {a, a, a, a, a, a, b, b, b, b, b, c, c, d, e}
```
