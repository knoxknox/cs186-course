MySQL Indexing

BTREE - common type
RTREE - MyISAM only (GIS)
HASH Indexes - NDB, MEMORY
FULLTEXT Indexes - MyISAM, InnoDB 5.6+

What Operations can BTREE Index do ?
- Find all rows with KEY=5 (lookup)
- Find all rows with KEY>5 (open range)
- Find all rows with 5<KEY<10 (closed range)

String Indexes
There is no difference
- Sort order is defined for strings ("AAAA" < "AAAB")
Prefix LIKE is a special type of Range
- LIKE "ABC%" means "ABC[LOW]"<KEY<"ABC[HIGHEST]"
- LIKE "%ABC" can't be optimized by use of the index

Multiple Column Indexes
Sort Order matters
- (1,2,3) < (1,3,1)
- KEY(col1,col2,col3)
It is still one BTREE Index
- Not a separate BTREE index for each level
PK is implicitly appended to all indexes
- KEY(column) is really KEY(column,ID) internally

How MySQL Uses Indexes
- Sorting
- Data Lookups
- Avoiding Reading
- Special Optimizations

Data Lookups
The classical use of index on (LAST_NAME)
- SELECT * FROM EMPLOYEES WHERE LAST_NAME="Smith"
Can use multiple column indexes (DEPT,LAST_NAME)
- SELECT * FROM EMPLOYEES WHERE LAST_NAME="Smith" AND DEPT="Accounting"

Multiple Columns Index
Index (A,B,C) - order of columns matters
Will use Index
- A>5
- A=5 AND B>6
- A=5 AND B=6 AND C=7
- A=5 AND B IN (2,3) AND C>5
Will NOT use Index
- B>5 (leading column is not referenced)
- B=6 AND C=7 (leading column is not referenced)
Will use Part of the index
- A>5 AND B=2 (range on 1st column, only use this key part)
- A=5 AND B>6 AND C=2 (range on second column, use two parts)

MySQL will stop using key parts in multi part index as soon as it met range (<,>,BETWEEN)
It however is able to continue using key parts further to the right if IN(...) range is used

Using Index for Sorting
SELECT * FROM PLAYERS ORDER BY SCORE DESC LIMIT 10
- Will use index on SCORE column
- Without index MySQL will do "filesort" which is very expensive
SELECT * FROM PLAYERS WHERE COUNTRY="US" ORDER BY SCORE DESC LIMIT 10
- Best served by Index on (COUNTRY,SCORE)
- Often combined with using Index for lookup
KEY(A,B) - multi column index
Will use Index for Sorting
- ORDER BY A (sorting by leading column)
- A=5 ORDER BY B (EQ filtering by 1st and sorting by 2nd)
- ORDER BY A DESC, B DESC (sorting by 2 columns in same order)
- A>5 ORDER BY A (range on the column, sorting on the same column)
Will NOT use Index for Sorting
- A IN(1,2) ORDER BY B (in-range on first column)
- ORDER BY B (sorting by second column in the index)
- ORDER BY A ASC, B DESC (sorting in the different order)
- A>5 ORDER BY B (range on first column, sorting by second)

You can't sort in different order by 2 columns
You can only have equality comparison (=) for columns which are not part of ORDER BY

Indexes and Joins
MySQL Performs Joins as "Nested Loops"
- SELECT * FROM POSTS,COMMENTS WHERE AUTHOR="John" AND COMMENTS.POST_ID=POSTS.ID
- Scan table POSTS, then for every such post go to COMMENTS table to fetch all comments
Index is only needed on table which is being looked up
- Very important to have all JOINs Indexed
- The index on POSTS.ID is not needed for this query performance
