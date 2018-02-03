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

# Files / Buffers

DBMS is organized in layers.<br/>
Each layer abstracts the layer below.

```
----------------------------
| SQL Client               |
----------------------------
| Query Parsing            | - translate sql into query plan
| Relational Operators     | - execute dataflow by operating on records & files
| Files & Index Management | - organizing tables & records as groups of pages in file
| Buffer Management        | - RAM (page1, page2, ..., pageX) => Disk Space Management
| Disk Space Management    | - translating page requests into physical bytes on device(hdd)
----------------------------
| Database                 |
----------------------------
```

Files:
```
         Serialized Schema___________________________________
        /                                                    \
       +----------+----------+----------+----------+----------+
Page 0 | 00000001 | 00000001 | 01111111 | 00000001 | 00000100 | - schema
       +----------+----------+----------+----------+----------+
       +----------+----------+----------+----------+----------+
Page 1 | 1001xxxx | 01111010 | xxxxxxxx | xxxxxxxx | 01100001 |
       +----------+----------+----------+----------+----------+
Page 2 | 1101xxxx | 01110010 | 01100100 | xxxxxxxx | 01101111 | - records
       +----------+----------+----------+----------+----------+
Page 3 | 0011xxxx | xxxxxxxx | xxxxxxxx | 01111010 | 00100001 |
       +----------+----------+----------+----------+----------+
        \________/ \________/ \________/ \________/ \________/
          valid?    record 0   record 1   record 2   record 3
```

Tables stored as logical files consisting of pages with records.<br/>
Filled with binary data, and can be represented as: files(pages(records)).<br/>
Example above: 5-byte pages, 1-byte records, 1-st byte checks if record is valid.

Disks:
- READ - transfer 'page' of data from disk to RAM
- WRITE - transfer 'page' of data from RAM to disk

Pages are managed:
- on disk by the disk space manager: pages read/written to physical disk/files
- in memory by the buffer manager: higher levels of DBMS only operate in memory

Disk Space Management:
- DBMS interfaces to storage at Block Level
- Block - unit of transfer for disk read/write (4KB, nowdays 64KB+)
- Page (ref to block on disk) - fixed size contiguous chunk of memory
- DSM read/write a page on/from disk, allocate/de-allocate logical pages

Kinds of Database Files:
- Index Files (B+ Trees, Hash Tables, etc..)
- Sorted Files (pages & records are in sorted order)
- Unordered Heap Files (records placed arbitrarily across pages)
- Clustered Heap File and Hash Files (pages & records are grouped)
