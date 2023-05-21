# Databases

## What is transaction?

transaction - it's atomic operation, which can be completed fully, or not
We can combine different operations into one transaction, when we need to be sure, that they completed/not completed together

For example, when we transfer money from one account to another, we need to make it one logical transaction:

```
BEGIN TRANSACTION;

UPDATE bank_accounts
SET amount = amount - 100
WHERE user_id = 123

UPDATE bank_accounts
SET amount = amount + 100
WHERE user_id = 321

END TRANSACTION;
```

Transaction have these main commands:
begin
end / commit
rollback

## Which properties transaction have?

**Atomicity** - means that operation (transaction) can be fully done or rollback'ed, yes or no, 1 or 0

**Consistency** - it's logical property. In previous example with bank transfer consistency means, that total amount of money will be the same after transaction. It's like law of conservation of energy

**Isolation** - it means, that each transaction independent of other transactions (see Isolation levels).
It'ssimilar to **Serializibility**, that means, that transactions runs like that runs serializably (one after one), even in real life they run concurrently

**Durability** - means, that if we get proof of end of transaction, that means, that there is can't happen something, to rollback that transaction, and it stored in database forever

## CAP, BASE

- `Consistency` 
- `Availiability`
- `Partitionability`

Three of them almost impossible simaltaniously, and usually all picks Availiability and Partitionability
So there BASE approach:

- `Basically availilale` - In the end query will return result (but it can takes time)
- `Soft State` - All works without you
- `Eventually Consistent` - Eventual consistency

## Consystency types:

- **Strict Consistency** - Full Consistency. Hard to achieve
- **Sequental consistency** - Guarantee, that latest processes will read/write newest value
- **Casual Consistency** - Data splits by dependend and independed. And Consistency exists only for Depended
- **Processor Consistency** - pass 
- **Weak Consistency** - pass
- **Eventual Consistency** - Means, that eventually (after some time) all data will be in consistent state 
- **Release Consistency** - pass
- **Entry Consistency** - pass

## Isolation Levels

- **Read Commited**: have two main parts 
	- no dirty reads: other transactions don't read any uncommited writes
	- no dirty writes: other transaction, that wants to write need to wait until fist transaction commits

- **Snapshot Isolation Level**: whtn each transaction reads their own version of data, that commited when that transaction started

- **Serializability**: based on one of three approaches:
	- real isolation: when you have not so much transactions, and all of them fast you can just evaluate queries one by one
	- 2 phased commit: quite slow
	- Serializable Snapshot Isolation

## What is Cursor. Types of Cursors

Area in memory, when result of query stored.
Can be client and server cursors.

## SQL vs No-SQL

by SQL it mostly means relational rowstored databases

No-SQL examples:

- **Column Store**: sparse matrix
	- CLickHouse
	- HBase
	- Google BigTable
	- Cassandra

- **Key:Value**: hash-table
	- S3
	- Voldemort

- **Document**: Tree
	- MongoDB

- **Graph**: graph
	- Allegro
	- InfinitieGraph

## How JOIN works

- Nested Loop:
For each row of the left table, iterate over whole right table
it's O(NxM)

- Merge Join:
At first we need to sort both tables, and after that
for first row of left table find all rows in right table and stops (remember place, where stop)
For second row of left table loop started from that place, where stopped.
So it's O(N+M)

- Hash Join:
At first we need to create hashmap from smallest table.
Then for each row from biggest table we calculate hash() and go to the hasmap.
so it's O(N)

## Indexes

### What is Index

**Index** - it's different data structure, that can perform select, but slow insert and update

In a nutshell - index it's like Table of contents for book. When you need particular chapter you don't need to fullscan all book.
You can just look at TOC and see in which page range needed chapter are stored

Indexes can be clustered and nonclustered (In MSSQL, for example):

- **Clustered Index** - It's index, that part of the particular table. It can be only one clustered Index for each table, because Clustered index actually  (it's B-Tree in MSSQL)
- **Non-clustered Index** - It's separate from table "TOC". It can be many nonclustered indexes for each table

PostgreSQL doesn't have that concepts: all data stored in heaps, so all indexes are non-clustered.

### Types of Indexes

Indexes can be implemented with different data structures

- **B-Tree** - Most used type at that time. 
- **Hash Index** - Mostly used for key:value storage
- **SS-Table** - tbh i don't get that fully. need to be filled later
- **LSM-Tree** -tbh i don't get that fully. need to be filled later

### PostgreSQL Types of Indexes:

B-Tree (default), Hash, Generalized Inverted Index (GIN), Generalized Search Tree (GiST), SP-GiST (Space-Partitioned GiST), Block Range Index (BRIN)

### When should indexes be avoided?

- You should not use indexes on small tables
- You should not use indexes on tables that face large and frequent batch UPDATE and INSERT operations
- You should not use indexes on columns that have many NULL values
- You should not use indexes on columns that are frequently edited

### Why you can't just create as many indexes as possible, if then so great?

- Because each index should be rebuild after any changes in table, which slow writes in database

## Rowstore and ColumnStore

### Rowstore

Data stored by rows, so DBMS iterate over rows and handle all the data

**Pro**:
	- It's cool for OLTP, because it's easy to append data at the end: you write in only one file (in general)
**Contra**:
	- In querits you need to handle all row with you, that fill memory

### Columnstore

Data stored by colummns, so RDBMS iterate over rows and handle all the data

**Pro**:
	- It's cool for OLAP, because generally you select only some columns, so RDMBS need to take care only of that particular columns, and don't store additional data
	- It's simple to compress columns, for example you can compress column AAAAAAABBCCAAA to A7B2C2A3  (and then apply BitMap, for example)
**Contra**:
	- It takes more time to insert data, becaues you need to insert data in all columns files

## SQL Questions

- Difference between Truncate and Delete: Delete delete row by row and log that. Truncate just drop whole table and recreates it
- What languages SQL has?: DDL (definition: create, alter), DML (manipulation: select), DCL (control: grant)
- Can you join with Null?: No, Null - not compatible, so Join Null on Null will be Null, so result will not appeared
- Difference between Union and Union ALL: Union drop duplicates in result query
- What types of window functions do you know?
	
	- lag(): previous row
	- lead(): next row
	- rownumber: just add rows number counter
	- rank: if there two rows with the same number, next number will be skipped
	- dense rank: no numbers skipped
	- min, max, etc.

- What is order of opertation:
	1. From
	2. Join
	3. Where
	4. Group by
	5. Having
	6. Select + Window functions
	7. Order by
	8. Limit

- What is normaliztion, which advantages it brings: Normalization in SQL is the process of organizing data to avoid duplication and redundancy. Some of the advantages are:
	- Better Database organization
	- More Tables with smaller rows
	- Efficient data access
	- Greater Flexibility for Queries
	- Quickly find the information
	- Easier to implement Security
	- Allows easy modification
	- Reduction of redundant and duplicate data
	- More Compact Database
	- Ensure Consistent data after modification


## Execution plan, Statistics (PostgrSQL)

### Execution plan

It's logical plan, which RDBMS will use to execute query. Can be run by two commands.

It's a tree where each leaf it's operation.
And each operation have their own cost, that can be changed by administrator (but default pretty ok for most cases)
PostgreSQL Planner select right plan by himself, and you can't "help" with that

- **Explain** - It's shows estimated (!) plan to given query
- **Explain Analyze** - It actually runs (!) query and then shows protocol of actually ised plan

Operations in execution plan:
- **Scan** - Selection of data
- **Bitmap** - Bit Maps construction
- **Sort, Aggregate, Append, Limit** - Actual operations with data
- **Intersect, Except** - Sets operations
- **Nested loop, hash join, merge join** - Joins
- **Init Plan, SubPlan** - Nested queries

Information that execution plan gives:
- **cost** - Operation cost
- **rows** - amount of rows
- **width** - width/weight (in bytes!)
- **output** - returned columns
- **actual** - actual data (shown only in analyze)
- **buffers** - weight of data
- **rows removed** - removed rows

Scan operations:
- **Sequential Scan** - sequential (full) scan
- **Index Scan** - scan of index
- **Index only Scan** - scan of index only
- **TID Scan** - scan for phisical row identifier
- **CTE Scan** - scan of CTE tables
- **Values Scan** - scan of constants
- **Function Scan** - scan of result of function

### Statistics

Stored in two tables, basically:

- **pg_class**
	- reltuples: amount of rows
	- relpages: amount of pages

- **pg_statistics**(table)/**pg_stats**(view)
Store data about fields/columns of table

	- null_frac: share of nulls
	- n_distinct: share of unique values
	- most_common_values: array of most frequent values (100 limit by default, but can be changed)
	- most_common_freqs: array of freauency of that most frequent values

### Weigth of some basid types

|type|bytes|
|----|-----|
|bool|1|
|int2|2|
|int4|4|
|int8|8|
|varchar(n)|n+4|
|char(n)|n+4|
|timestamp|8|

## How data stored (PostgreSQL)

Table consist of segments

Segment consist of pages

In each page stored actual data.
Write in each page performs in two directions:
from start of file added pointers to data
from end added actual data

Index data structure looks the same, but pointers in pages look not onto data, but on links to data

## Scalability, Distribution, Partition

Two main scalability options:
- Master-Slave:
	slaves hold copies of master, and clients send queries to less loaded slave
- Sharding:
	When you split data to different machines (like partitioning, but on machines)

- Distribution :
	Process, that distributes data by some key (random, preferrably), on smaller parts, so all nodes can perform query
- Partitions:
	Partition by monthes


## Links and literature

### Not sources of that page, but good sources in general

- Designing Data-Intensive Applications (Martin Kleppmann)