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
