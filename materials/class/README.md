# Notes

1. SQL
2. Files / Buffers
3. Indexing
4. Hashing / Sorting
5. Relational Algebra
6. Joins
7. Concurrency
8. Transactions

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

```
Heap File
-------------------------------
| 2,5 | 1,6 | 4,7 | 3,9 | 8,9 |
-------------------------------
```

```
Sorted File
-------------------------------
| 1,3 | 3,1 | 5,4 | 7,5 | 9,8 |
-------------------------------
```

```
----------------------------------------------
|                   Heap   Sorted    Index   |
----------------------------------------------
| insert          | O(1)   O(B)      O(logB) |
| delete          | O(B)   O(B)      O(logB) |
| scan all        | O(B)   O(B)      O(B)    |
| range search    | O(B)   O(logB)   O(logB) |
| equality search | O(B)   O(logB)   O(logB) |
----------------------------------------------
```

# Indexing

Index:
- Data structure that enables fast lookup by search key
- There are many types of them: Hash, B-Tree, R-Tree, GiST...

B+ Tree:
- Consists of: root, internal nodes, leaves
- Data records are stored only in leaf nodes
- Root node may be a leaf or a node with 2+ children
- N - max number of data entries that can fit in node
- Each internal node must contains N/2...N search keys
- Each internal node has (N+1)/2...N+1 children, where N >= 2
- Each leaf node must contains N/2...N pairs of <key,record_ptr>
- Each leaf node contains pointers to sibling (pointer = page number)

Index Files:
- clustered: record stored in index file <key, record>
- unclustered: record not stored in index file <key, record_id>

```
              +-Page2-------+
              | F | 002 | 1 | - root
              | M | 199 | 3 |   (gender, page_id)
              +-------------+
             /               \
+-Page1------------+   +-Page3------------+
| F | 002,Ann,140  |   | M | 199,Joe,380  | - leaves
| F | 012,Jay,335  |   | M | 250,Jim,155  |   (gender, row)
| F | 090,Mia,1200 |   | M | 278,Edd,9400 |
| F | 234,Emma,650 |   | M | 279,John,350 |
+------------------+   +------------------+
```

```
                  +-Page2-------+
                  | F | 002 | 1 | - root
                  | F | 359 | 3 |   (gender, page_id)
                  | M | 008 | 4 |
             _____| M | 199 | 5 |_____
            /     +-------------+     \
           /     /               \     \
+-Page1---+  +-Page3---+   +-Page4---+  +-Page5---+
| F | 002 |  | F | 359 |   | M | 008 |  | M | 199 | - leaves
| F | 012 |  | F | 750 |   | M | 028 |  | M | 250 |   (gender, row_id)
| F | 090 |  | F | 820 |   | M | 135 |  | M | 278 |
| F | 234 |  | F | 860 |   | M | 198 |  | M | 279 |
+---------+  +---------+   +---------+  +---------+
```

Search(k)
```
- leaf-0 when k < 10
- leaf-2 when k >= 20
- leaf-1 when 10 <= k < 20
                            inner
                            +----+----+----+----+
                            | 10 | 20 |    |    |
                            +----+----+----+----+
                           /     |     \
                      ____/      |      \____
                     /           |           \
+----+----+----+----+  +----+----+----+----+  +----+----+----+----+
|  1 |  2 |  3 |    |->| 11 | 12 | 13 |    |->| 21 | 22 | 23 |    |
+----+----+----+----+  +----+----+----+----+  +----+----+----+----+
leaf-0                 leaf-1                 leaf-2
```

Insert(k, r)
```
1. inserting the pair (k, r) does not cause overflow
2. inserting the pair (k, r) does cause node n to overflow
on overflow n - split into left/right nodes, mid key pushed up

Insert(4) - no overflow
                            +----+----+----+----+
                            | 10 | 20 |    |    |
                            +----+----+----+----+
                           /     |     \
                      ____/      |      \____
                     /           |           \
+----+----+----+----+  +----+----+----+----+  +----+----+----+----+
|  1 |  2 |  3 |  4 |->| 11 | 12 | 13 |    |->| 21 | 22 | 23 |    |
+----+----+----+----+  +----+----+----+----+  +----+----+----+----+

Insert(5) - overflow occurs
[1,2,3,4] => [1,2,x,x]->[3,4,5,x]
                            +----+----+----+----+
                            |  3 | 10 | 20 |    | - mid key was 3
                            +----+----+----+----+
                           /     |    |     \
                      ____/      |     \_____\______________________
                     /           |           \                      \
+----+----+----+----+  +----+----+----+----+  +----+----+----+----+  +----+----+----+----+
|  1 |  2 |    |    |->|  3 |  4 |  5 |    |->| 11 | 12 | 13 |    |->| 21 | 22 | 23 |    |
+----+----+----+----+  +----+----+----+----+  +----+----+----+----+  +----+----+----+----+
```

Delete(k)
```
Delete(2)
                            +----+----+----+----+
                            | 10 | 20 |    |    |
                            +----+----+----+----+
                           /     |     \
                      ____/      |      \____
                     /           |           \
+----+----+----+----+  +----+----+----+----+  +----+----+----+----+
|  1 |  2 |    |    |->| 11 | 12 | 13 |    |->| 21 | 22 | 23 |    |
+----+----+----+----+  +----+----+----+----+  +----+----+----+----+

Delete(1)
                            +----+----+----+----+
                            | 10 | 20 |    |    |
                            +----+----+----+----+
                           /     |     \
                      ____/      |      \____
                     /           |           \
+----+----+----+----+  +----+----+----+----+  +----+----+----+----+
|  1 |    |    |    |->| 11 | 12 | 13 |    |->| 21 | 22 | 23 |    |
+----+----+----+----+  +----+----+----+----+  +----+----+----+----+

Should not rebalance tree
                            +----+----+----+----+
                            | 10 | 20 |    |    |
                            +----+----+----+----+
                           /     |     \
                      ____/      |      \____
                     /           |           \
+----+----+----+----+  +----+----+----+----+  +----+----+----+----+
|    |    |    |    |->| 11 | 12 | 13 |    |->| 21 | 22 | 23 |    |
+----+----+----+----+  +----+----+----+----+  +----+----+----+----+
```

# Hashing / Sorting

External merge sort:
- Assume we have: 10 MB RAM, 1000 MB data
- Read in 10 MB at a time, sort it, write to disk (producing 100 files of 10 MB)
- Merge 10 files of 10 MB together to make 100 MB, then merge 10 files of 100 MB

```
[3,4] [6,2] [9,4] [8,7] [5,6] [3,1] [7,4] [6,1] - input file
8 pages with 2 records each; we have space only available for 2 pages

 _1_______   _2_______   _3_______   _4_______
|         | |         | |         | |         |
[3,4] [6,2] [9,4] [8,7] [5,6] [3,1] [7,4] [6,1]   - select next 2 pages (buffer)
                                                  - apply sort algorithm (quicksort)
   [2,3]       [4,7]       [1,3]       [1,4]      - write sorted data result back to disk
   [4,6]       [8,9]       [5,6]       [6,7]
        \     /                 \     /           - merge data (see merge algo description)
         [2,3]                   [1,1]
         [4,4]                   [3,4]
         [6,7]                   [5,6]
         [8,9]                   [6,7]
              \                 /
               -----------------                  - now data is sorted on disk (all 8 pages)
                     [1,1]
                     [2,3]
                     [3,4]
                     [4,4]
                     [5,6]
                     [6,6]
                     [7,7]
                     [8,9]

Merge files:
1) Open 2 files
2) Read 1 value from F1 & F2
3) Compare F1.value with F2.value
4) Write less value to new file F3
5) Increment position in file where value is less
6) Repeat step 3 until both iterators complete work

Example of merge:
F1 = [2,3,4,6]
F2 = [4,7,8,9]

I1=0, I2=0; V1=2, V2=4; (2 < 4); F3 = [2]
I1=1, I2=0; V1=3, V2=4; (3 < 4); F3 = [2,3]
I1=2, I2=0; V1=4, V2=4; (4 < 4); F3 = [2,3,4]
I1=3, I2=0; V1=6, V2=4; (6 > 4); F3 = [2,3,4,4]
I1=3, I2=1; V1=6, V2=7; (6 < 7); F3 = [2,3,4,4,6]
I1=4, I2=1; V1=?, V2=7; (? < 7); F3 = [2,3,4,4,6,7]
I1=4, I2=2; V1=?, V2=8; (? < 8); F3 = [2,3,4,4,6,7,8]
I1=4, I2=3; V1=?, V2=9; (? < 9); F3 = [2,3,4,4,6,7,8,9]
```

Example:
```
Buffer: 4 elements


Take 2 pages: [3,4] [6,2]
Sort in memory: [2,3] [4,6]
Write back to disk: file1 = [2,3] [4,6]

Take 2 pages: [9,4] [8,7]
Sort in memory: [4,7] [8,9]
Write back to disk: file2 = [4,7] [8,9]

Take 2 pages: [5,6] [3,1]
Sort in memory: [1,3] [5,6]
Write back to disk: file3 = [1,3] [5,6]

Take 2 pages: [7,4] [6,1]
Sort in memory: [1,4] [6,7]
Write back to disk: file4 = [1,4] [6,7]


Take 2 files (F1,F2): [2,3,4,6] [4,7,8,9]
Apply merge, write to file: [2,3,4,4,6,7,8,9]

Take 2 files (F3,F4): [1,3,5,6] [1,4,6,7]
Apply merge, write to file: [1,1,3,4,5,6,6,7]


Take 2 files (F5,F6): [2,3,4,4,6,7,8,9] [1,1,3,4,5,6,6,7]
Apply merge, write to file: [1,1,2,3,3,4,4,4,5,6,6,6,7,7,8,9]
```

# Relational Algebra

Lecture materials can be found here: https://bit.ly/2H85yqO

# Joins

Algorithms:
- hash join
  - index type => hash table
  - special case of index nested-loop
- sort-merge join
  - can use not only equi operator
  - can be used with large data-sets
- nested-loop join
  - can always be applied
  - complexity = [R x S] => O(n * n)
- index nested-loop
  - index type => any index (like b-tree, etc..)
  - index should be build on one of compared attrs

hash
```
JP(r, s) = (r.x == s.x)

join(R, S, JP(r, s))
  index = hash_table(R.x)

  foreach s in S
    hash_key = hash(s.x)
    result = index.get(hash_key)
    output([r, s]) if result.present?
```

sort-merge
```
JP(r, s) = (r.x == s.x)

join(R, S, JP(r, s))
  sort(R on R.x)
  sort(S on S.x)
  pointer pr = r[0]
  pointer ps = s[0]
  do
    if pr.x == ps.x
      output([pr, ps])
    pr.x <= ps.x ? pr++ : ps++
  while pr != R.end && ps != S.end
```

nested-loop
```
JP(r, s) = (r.x == s.x)
GT(r, s) = (r.x >= s.x)
LT(r, s) = (r.x <= s.x)

join(R, S, JP(r, s))
  foreach r in R
    foreach s in S
      output([r, s]) if JP(r, s)
```

index nested-loop
```
JP(r, s) = (r.x == s.x)

join(R, S, JP(r, s))
  index = catalog.get(indexes, R.x)

  foreach s in S
    result = index.get(s.x)
    output([r, s]) if result.present?
```

Join Types:
```
------------------------------users-     ------------------------------orders-
| id | name                        |     | id | user_id |        payment_sum |
------------------------------------     -------------------------------------
|  1 | George Washington           |     |  1 |       1 |             234.56 |
|  2 | John Adams                  |     |  2 |       3 |              78.50 |
|  3 | Thomas Jefferson            |     |  3 |       2 |             124.00 |
|  4 | James Madison               |     |  4 |       3 |              65.50 |
|  5 | James Monroe                |     |  5 |       7 |              25.50 |
|    |                             |     |  6 |       9 |              14.40 |
------------------------------------     -------------------------------------



FULL JOIN                                INNER JOIN
select * from users full join orders     select * from users inner join orders
------------------------------------     -------------------------------------
| George Washington    | $234.56 |1|     | George Washington     | $234.56 |1|
| Thomas Jefferson     |  $78.50 |3|     | John Adams            | $124.00 |2|
| John Adams           | $124.00 |2|     | Thomas Jefferson      |  $78.50 |3|
| Thomas Jefferson     |  $65.50 |3|     | Thomas Jefferson      |  $65.50 |3|
| NULL                 |  $25.50 |0|     -------------------------------------
| NULL                 |  $14.40 |0|
| James Madison        |    NULL |4|
| James Monroe         |    NULL |5|
------------------------------------

LEFT JOIN                                RIGHT JOIN
select * from users left join orders     select * from users right join orders
------------------------------------     -------------------------------------
| George Washington    | $234.56 |1|     | George Washington     | $234.56 |1|
| John Adams           | $124.00 |2|     | Thomas Jefferson      |  $78.50 |3|
| Thomas Jefferson     |  $78.50 |3|     | John Adams            | $124.00 |2|
| Thomas Jefferson     |  $65.50 |3|     | Thomas Jefferson      |  $65.50 |3|
| James Madison        |    NULL |4|     | NULL                  |  $25.50 |0|
| James Monroe         |    NULL |5|     | NULL                  |  $14.40 |0|
------------------------------------     -------------------------------------
```
