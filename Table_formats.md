# Table formats

## Row vs Columnar file/data formats 

phisycal models to store data can be these:<br>
example table:<br>

|rows|col_a|col_b|col_c|
|----|-----|-----|-----|
|row0| a0  | b0  | c0  |
|row1| a1  | b1  | c1  |
|row2| a2  | b2  | c2  |
|row3| a3  | b3  | c3  |

- Row format (each file contains some rows). they can united in longer rows, but, in a nutshell it's a continuous amount of rows.<br>
It's cool to use at row-store formats/dbs, **for OLTP/Write data-access pattern**, where you need to insert you just append row, where you need update/delete, you just need to find start of row, and read all continiously , because equential reading better for disk.<br>
it can be like in an example as is:

|     |     |     |     |     |     |
|-----|-----|-----|-----|-----|-----|
| a0  | b0  | c0  |	a1  | b1  | c1  |
| a2  | b2  | c2  | a3  | b3  | c3  |


- Pure Columnar

When your store one column in sequence.<br>
So, you need to find start of needed columns and read sequentially all data.<br>

|     |     |     |     |     |     |
|-----|-----|-----|-----|-----|-----|
| a0  | a1  | a2  |	a3  | b0  | b1  |
| b2  | b3  | c0  | c1  | c2  | c3  |

Or you can use different file for each column. Also, it's neccessary to undeerstand offset (~index) of each elemnt in file.
But it can be hard to organise that files physycally. So real-life implenetation of Parquet looks not like that, despite it's columnar table foramt

Columnar format **better suited for OLAP load pattern**, when you need to read from file/db

- Columnar (Hybrid)

That means, that you have both columns (horizontal) and columns(vertical) partitioning.

|     |     |     |     |     |     |
|-----|-----|-----|-----|-----|-----|
| a0  | a1  | b0  |	b1  | c0  | c1  |
| a2  | a3  | b2  | b3  | c2  | c3  |

So it's pure-columnar, but splitted by chunks (here is chunk length equals 2)

## Row formats

### Apahe Avro ([doc1](https://avro.apache.org/docs/), [doc2](https://avro.apache.org/docs/1.11.1/getting-started-python/)) 

it's an row format.
It also can be understood as json, but serialised in binary (which is oversimplifies whole thing, but still)

[nice video about avro internals](https://www.youtube.com/watch?v=0HsMaiLbXFk)

### Thrift

pass

### ProtoBuf

pass

## Columnar formats

### ORC (Optimized Row Columnar) ([docs](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC))

![orc](./images/tf_orc_1_1.webp)

ORC it's pure Columnar storaging format

[Article about ORC](https://medium.com/data-and-beyond/exploring-the-orc-file-format-advantages-use-cases-and-best-practices-for-data-storage-and-79c607ee9289)

ORC **optimized for reading**, mostly, because it has predicate pushdown, which allows Spark to skip over irrelevant data when querying large datasets.

### Apache Parquet ([docs](https://parquet.apache.org/docs/file-format/))

Parquet uses Columnar (hybrid) storaging format

**So, high-level:**<br>

- Parquet file consits of one or more **Row group** (can be configured by user)
- **Row group** contains exactly one column chunk per column
- **Column chunk** contains one or more pages 
- **Page** it's individible part of parquet

**And example:**<br>

![Parquet file representation](./images/tf_parquet_1_1.gif)

- 4-byte magic number "PAR1"
- Row-group 1 (128mb by default)
	- Column A chunk 0
		- Page 0
			- Page metadata<br>
				_min_, _max_, _count_
			- Repetition levels
			- Definition levels<br>
				_for nested schemas_
			- Encoded values<br>
				_actual data in encoding state_
		- ...
		- Page N
	- Column B chunk 0
	- ...
	- Column N chunk 0
- Row-group 2 
	- Column A chunk 1
		- Page 0
		- ...
		- Page N
	- Column B chunk 1
	- ...
	- Column N chunk 1
- Row-group M
- Footer **(File metadata)**
	- file
	- row-group 
	- column metadata start locations

![Parquet Architecture Again](./images/tf_parquet_1_2.gif)

**Parquet has 6 encoding schemes, but there basic:** <br>

- Plain
	- Fixed with: means, that values stored back-to-back. cool for integers, for example:<br>
		file example: 1, 5, 6, 3, ...
	- Non-fixed with: used for different length data (strings). in that case data have length prefix
		file example: 2, yo, 3, no, 8, hobahoba, ...

- RLE_Dictionary
	- Run-length encoding + bit-packing + dictionary compression
	- Assumes duplicate and repeated values

		example data:
		a, b, c, a, a, a, f, f, b, a

		dictionary-compression stage:
		_it's just creation of hashmap_
		a: 0, b:1, and so on

		so data will be
		0, 1, 2, 0, 0, 0, 6, 6, 1, 0

		RLE+Bit packaging stage:
		0, 1, 2, (3, 0), (2, 6), 1, 0

_Note1:_ beacuse of all that metadata things stored in file, and rowgroups, you shoukd avoid use small files
_Note2:_ Also avoid to use hufe files, because that means huge footer, but algorythms for foorer managment not optimized for speed.
_Note3:_ So, size of 1gb is ok

#### Conclusion about Parquet:

- Redused I/O
	- because of size (compression to RLE_Dictionary)
	- because of avoioding read irrelevant data (based on metadata min max and dictionary filtering)
- Redused overhead
	- if you avoid using small files

Parquet is **optimized for write-heavy workloads**, as it includes features such as encoding and dictionary compression that can improve write performance.

### Apache Arrow ([docs](https://arrow.apache.org/overview/))

its in-memory columnar format, which makes easier and faster tranfser between compnents.

Without a standard columnar data format, every database and language has to implement its own internal data format. This generates a lot of waste. Moving data from one system to another involves costly serialization and deserialization. In addition, common algorithms must often be rewritten for each data format.

Arrow's in-memory columnar data format is an out-of-the-box solution to these problems. Systems that use or support Arrow can transfer data between them at little-to-no cost. Moreover, they don't need to implement custom connectors for every other system. On top of these savings, a standardized memory format facilitates reuse of libraries of algorithms, even across languages.

![before](./images/tf_arrow_1_1.png)
![after](./images/tf_arrow_1_2.png)

[Video about arrow from creator](https://www.youtube.com/watch?v=R4BIXbfKBtk&t=548s)

## Data lakehouse formats

It's crucial to understand, that formats, listed below it's not programs, but it's not only data formats.
It's combination of file format itself + set of APIs to different languagaes, to operate with that formats.
So file in standalone mode - it's not enough

In other hand, there is no compute, or some daemon running processes, to manipulate with that formats - every optimization done by that API/Convention, while you actually "call" to compute that files

### Apache Hoodie ([docs](https://hudi.apache.org/docs/concepts/))

it's not file format (because Hudi stores data in Parquets), but it's data lake building framework.

#### COW vs MOR storage models

There is two basic principles of how hudi store data:

**Copy on Write**: Data is stored in a columnar format (Parquet), and each update creates a new version of files during a write. COW is the default storage type. So all the things happend on **writer side**

![COW](./images/tf_hudi_cow_1_1.webp)

**Merge on read**: Data is stored using a combination of columnar (Parquet) and row-based (Avro) formats. Updates are logged to row-based delta files and are compacted as needed to create new versions of the columnar files.

![MOR](./images/tf_hudi_mor_1_1.webp)

**COW vs MR**:
So, in a nutshell, COW have bigger latency on writing, but faster reading. MOR - vice-versa.

[Table types in Hudi](https://medium.com/@simpsons/different-table-types-in-apache-hudi-datalake-apachehudi-cow-mor-f508c474cb8c)
[Copy-on-write](https://medium.com/@Eswaramoorthy.P/demystifying-copy-on-write-in-apache-hudi-understanding-read-and-write-operations-3aa274017884)
[COW vs MOR](https://www.onehouse.ai/blog/comparing-apache-hudis-mor-and-cow-tables-use-cases-from-uber-and-shopee)

#### Query types ([docs](https://hudi.apache.org/docs/next/table_types/))

**Snapshot query**: It's possibility to query present state of date. (it's like to have SCD1)

**Time Travel**: It is what it says - you can query state of the table in concrete moment of time. So it's like shapshot query, but you can "choose snapshot", that you need (imageine, that you have SCD2)

**Incremental query**: Query, that can see new data written/changed in the table from particular commit/point in time

**Read optimized Queries**: it's when you read only base parquet files, without implementing all new deltas, that probably have place. It's more efficient read, but probably you will lose some latest data, that haven't compacted to new base parquet file still

There are some limitations, so:
COW tables can take: Snapshot, Time Travel, Incremental and Incremental CDC queries
MOR tables can take: Snapshot, Time Travel, Incremental and Read optimized queries

And there are examples for each type of table:

![snapshot vs incremental for COW](./images/tf_hudi_cow_query_1_1.png)

![snapshot vs incremental for MOR](./images/tf_hudi_mor_query_1_1.png)

[Article about different query types](https://medium.com/@simpsons/different-query-types-with-apache-hudi-e14c2064cfd6)

#### Indexes

**Global Index**: it's unique key all over the table, i.e. it guarantees, that exactly one row with that key exists in the whole table. It offer that stron guarantees, but uodate/delete it's quite costy operation

**Non-Global Index**: it's uniquet key only in the partition. So there is no such strong guarantees of uniqness, but it has better performance

##### Index types

- Bloom Index
- Simple Index
- HBase Index
- Bucket Index
- Bucket Index with consistent hashing

[Make upserts faster](https://www.onehouse.ai/blog/top-3-things-you-can-do-to-get-fast-upsert-performance-in-apache-hudi)
[Bucket index](https://medium.com/@simpsons/speed-up-your-write-latencies-using-bucket-index-in-apache-hudi-2f7c297493dc)
[Simple index](https://www.linkedin.com/pulse/apache-hudi-accelerating-upsert-simple-index-choosing-soumil-shah%3FtrackingId=pmENaqg9Rt6NG6aCzIQ2Eg%253D%253D/?trackingId=pmENaqg9Rt6NG6aCzIQ2Eg%3D%3D)

#### Operation Types ([docs](https://hudi.apache.org/docs/next/write_operations/#operation-types))

- Upsert: if row doesn't existed in table, it will be inserted. If it's already existed - it will be updated
- Delete: There are **soft** (it's only Null'ed all data in the row, but the row and key (!) will be remain) and **hard** (deletes whole row with an index) deletes in Hudi
- Insert: Just insert without scanning indexes, which can corrupt data, but it's pretty fast
- Insert overwrite ([article about](https://soumilshah1995.blogspot.com/2023/03/rfc-18-insert-overwrite-in-apache-hudi.html)): It's similar to upsert, but faster, because get rid of some index things. If you want to reinit some of your partition (for example you partitioned by date), you can just insert overwrite whole partition, and it will be fastest way to do that
- Delete partition: deletes entire partition
- Bulk insert ([article about](https://medium.com/@simpsons/bulk-insert-sort-modes-with-apache-hudi-c781e77841bc)): it's efficient when you deal with huge amoun of data

[AWS blog about hudi + glue](https://aws.amazon.com/blogs/big-data/part-1-get-started-with-apache-hudi-using-aws-glue-by-implementing-key-design-concepts/)

[DeepDive video about Hudi](https://www.youtube.com/watch?v=nGcT6RPjez4)
[Hudi QuickStart demo](https://hudi.apache.org/docs/docker_demo)
[Hudi+PySpark example](https://medium.com/@sagarlakshmipathy/a-beginners-guide-to-apache-hudi-with-pyspark-part-1-of-2-8a4e78f6ad2e)
[Getting started with Hudi](https://datacouch.medium.com/getting-started-with-apache-hudi-711b89c107aa)
[Another one](https://medium.com/walmartglobaltech/a-beginners-guide-to-using-apache-hudi-for-data-lake-management-6af50ade43ad)

### Apache Iceberg ([docs](https://iceberg.apache.org/spec/#overview))

First of all engine needs a data catalog.
It can be hive, for example, where latest path to Metadata file is stored.

All engine need that type of files in "Metadata" folder:
- Metadata file (core json file): Stores metadata about table in certain point in time. 
	- conatins: table_uuid, location, schema, partitions, current_snapshot, snapshots (list of manifests)
- Manifests list (list of manifest and short statistic about them in avro format)
	- contains: manifest path, snaphsot_id which manifest belongs to, partitions spec and info
- Manifest (conatains list of stats about data files, in avro format)
	contains: file_path, file_format(parquet by default), partition, count(), null_value_stats, lower_bounds, upper_bounds for each column
- Data file (pqrauet)

**Reading**: read metastore -> read metadata -> manifests list - > manifests -> data files
**Writing**: write to data file -> change manifest -> change manifest list ... -> change metadata file -> change metastore

![metadata overview](./images/tf_iceberg_1_1.png)

**ACID**:
So, while each writing changes whole metadata, and we change our metastore each time (we point to newer version) in last order, and it can be happen only if all previous process goes well, that give us **ACID** write transactions, because, if transaction fails on lower levels - it didn't change the metadata, and our metastore still will be point on previous (last success) metadata file (which contains last good snapshot)

**Schema Evolution**:
Because all process looks like COW tables in hudi (each transaction makes new snapshot), and while we keep store previous versions of metadata, and because we store the schema in metadata file - we can just select that time-travel query, and it will go by old path with previous metadata and previous schema, and previous files underneath


[Iceberg Qucik Overview from Dremio. Recommend](https://www.youtube.com/watch?v=stJLaIZRcJs)
[Iceberg Deep Overview playlist from Dremio](https://www.youtube.com/playlist?list=PL-gIUf9e9CCskP6wP-NKRU9VhofMHYjcd)

### Delta

pass


[Hudi vs Delta vs Iceberg comparison](https://www.onehouse.ai/blog/apache-hudi-vs-delta-lake-vs-apache-iceberg-lakehouse-feature-comparison)