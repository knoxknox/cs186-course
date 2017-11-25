# MySQL Indexing

- BTree: common type
- RTree: MyISAM only (GIS)
- Hash Indexes: NDB, Memory
- FullText Indexes: MyISAM, InnoDB 5.6+

## BTree
- Find all rows with KEY=5 (lookup)
- Find all rows with KEY>5 (open range)
- Find all rows with 5<KEY<10 (closed range)

## String Indexes
There is no difference
- Sort order is defined for strings ("AAA" < "AAB")

Prefix LIKE is a special type of Range
- LIKE "ABC%" means "ABC[LOW]" < KEY < "ABC[HIGHEST]"
- LIKE "%ABC" cannot be optimized by use of the index

## Multiple Column Indexes
Sort order matters
- (1,2,3) < (1,3,1)
- KEY(col1,col2,col3)

It is still one BTree Index
- Not a separate BTree index for each level

PK is implicitly appended to all indexes
- KEY(column) is really KEY(column,ID) internally

# How MySQL Uses Indexes

- Sorting
- Data Lookups
- Avoiding Reading
- Special Optimizations

## Data Lookups
The classical use of index on (LAST_NAME)
- SELECT * FROM EMPLOYEES WHERE LAST_NAME="Smith"

Can use multiple column indexes (DEPT, LAST_NAME)
- SELECT * FROM EMPLOYEES WHERE LAST_NAME="Smith" AND DEPT="Accounting"

## Multicolumn Index
Index (A,B,C) - order of columns matters

Will use index
- A>5
- A=5 AND B>6
- A=5 AND B=6 AND C=7
- A=5 AND B IN(2,3) AND C>5

Will not use index
- B>5 (leading column is not referenced)
- B=6 AND C=7 (leading column is not referenced)

Will use part of index
- A>5 AND B=2 (range on 1st column, only use this key part)
- A=5 AND B>6 AND C=2 (range on second column, use two parts)

MySQL will stop using key parts in multipart index as soon as it met range (>,<,BETWEEN)<br/>
It however is able to continue using key parts further to the right if IN(.....) range is used

## Using Indexes with Joins
MySQL performs joins as "Nested Loops"
- SELECT * FROM POSTS,COMMENTS WHERE AUTHOR="John" AND COMMENTS.POST_ID=POSTS.ID
- Scan table POSTS, then for every such post go to COMMENTS table to fetch all comments

Index is only needed on table which is being looked up
- Very important to have all JOINs indexed
- The index on POSTS.ID is not needed for this query performance

## Using Indexes with Sorting
SELECT * FROM PLAYERS ORDER BY SCORE DESC LIMIT 10
- Will use index on SCORE column
- Without index MySQL will do "filesort" which is very expensive

SELECT * FROM PLAYERS WHERE COUNTRY="US" ORDER BY SCORE DESC LIMIT 10
- Best served by index on (COUNTRY,SCORE)
- Often combined with using index for lookup

KEY(A,B) will use index for sorting
- ORDER BY A (sorting by leading column)
- A=5 ORDER BY B (EQ filtering by 1st and sorting by 2nd)
- ORDER BY A DESC, B DESC (sorting by 2 columns in same order)
- A>5 ORDER BY A (range on the column, sorting on the same column)

KEY(A,B) will not use index for sorting
- A IN(1,2) ORDER BY B (in-range on first column)
- ORDER BY B (sorting by second column in the index)
- ORDER BY A ASC, B DESC (sorting in the different order)
- A>5 ORDER BY B (range on first column, sorting by second)

You can't sort in different order by 2 columns<br/>
You can only have eq comparison (=) for columns which are not part of ORDER BY
