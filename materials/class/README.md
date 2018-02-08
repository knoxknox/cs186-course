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

Heap File:
```
    Header Page
   -------------          Notes:
   |xxxxxxxxxxx|          x - ref to data page
+--|xxxxxxxxxxx|          header page like a catalog for data pages
|  -------------          header page include #free bytes on ref pages
+->|xxxxxxxxxxx|
+--|xxxxxxxxxxx| -- ref
|  -------------    \
+->|xxxxxxxxxxx|     -----
   |xxxxxxxxxxx|     | x | - data page
   -------------     -----
```

Heap file allows to retrieve rows:
- by scanning all records sequentially
- by specifying the record id (page id + offset)
- to fetch records by value - use indexes (dep='CS'; gpa>3 AND age=42)

Page Layout:
- fixed length records
- variable length records

```
fixed length
-------------------------------------
| 11010000                          | - header
| R > R > E > R > E > E > E > E     | - records
-------------------------------------

id - record number in page
insert - find first empty slot in bitmap
delete - clear bit from page header bitmap
can contain: free space & number of records
```

```
variable length
-------------------------------------
| R > R > E > R > ...               | - records
| 18, 20, null, 24, FP          [3] | - slot-table
-------------------------------------

id - location in slot-table (from right)
insert - add in a free space, create pointer in slot-table, update FP
delete - set slot to null (empty slots will be reorganized via fragmentation)
slot-table (18, 20, null, 24), ref to free space (FP), number of full slots ([3])
```

Record Layout:
- fixed length format
- variable length format

```
fixed length
-----------------------------------------
|   4 |       8 |   1 |   4 |         7 |
-----------------------------------------
|   3 |   3.142 |   T |   3 |   HELLO_W | - 24 bytes
-----------------------------------------
    int    double  bool   int     char(7)

field types same for all records in file
on disk byte representation same as in memory
finding i-th field? fast - done via arithmetic
```

```
variable length
-------------------------------------------
|   1 |    4 |       4 |     3 |        6 |
-------------------------------------------
|   M |   32 |   94000 |   Bob |   Market | - 18 bytes + header
-------------------------------------------
   char    int       int   vchar      vchar

contains record header
store information about variable lengths
move all variable length fields to end (enable fast access)
```

Cost of Operations:
- R: number of records per block (=2)
- B: number of data blocks in file (=5)

Heap File
```
-------------------------------
| 2,5 | 1,6 | 4,7 | 3,9 | 8,9 |
-------------------------------
```

Sorted File
```
-------------------------------
| 1,3 | 3,1 | 5,4 | 7,5 | 9,8 |
-------------------------------
```

```
-------------------------------------------
|                     Heap     Sorted     |
-------------------------------------------
| insert          |   2        logB+B     |
| delete          |   B/2      logB+B     |
| scan all        |   B        B          |
| range search    |   B        logB+pages |
| equality search |   B/2      logB       |
-------------------------------------------
```
