A data store provides two major functionalities:
- Data storage
- Data retrieval

Even for an application developer, who is never going to implement his own **storage engine**, it makes sense to have a **rough idea** of the storage engine to **tune** it according to the **workload**.

Of the many families of storage engines, two worth mentioning are:
- Log-structured storage engine
- Page-oriented storage engine

## Log structured storage engine

A log is an append-only file.
Appending to a file is the simplest possible write operation and it's hard to beat it's performance.

A log structured storage engine keeps writing to an append only data file.

### Index

We can keep appending records to a file. This is the storage part.
However, during retrieval the entire file would have to be scanned unless there is an index. This would have a time complexity of O(n).

Indexes act like a **signpost**.

Two things worth mentioning about indexes:
- It is metadata, and not actual data. Index can be deleted without deleting the actual data.
- It's *derived* from real data.

Indexes can significantly improve read. However indexes add an **overhead** during writes.

### Hash indexes

A key-value database can be implemented using a log-structured storage engine. Let's assume the data is:

{'name': 'bob', 'place': {'city': 'Hyderabad', 'country': 'India'}}

It would look like the following in file.
name,bob
place,{'city': 'Hyderabad', 'country': 'India'}

It would look like the following on disk:

    n a m e , b o b \n p l a c e , { c i t y : H

Each character would take say 1 byte on disk.

Writing a key-value pair can be very performant as it's append-only. Worth mentioning, it would be sequential write and not random write.
However reading from this file can be O(n) if there is no index.

An in-memory hash-map can be used as an index. The key of hash-map would be the same key as that used during writing. The value would be the byte offset in the file.
The in-memory hash-map index would look like:
{name: 0, place: 9}
This hash-map can make even reads operation as O(1). Only one file **seek** would have to be performed. And that too would be avoided if the data is in the filesystem cache.

This storage-engine is used in **Bitcask** storage engine of **Riak** database.

#### Deal with growing file size

If a single apend-only file is used it can grow, become unmanageable and can reach the disk limit.

- Use segments instead of a single large file.
- Compaction and merging of segments

Compaction: Throwing away duplicate keys from a segment and keeping only the most recent update for each key.
Merging: Combining several segments into a single segment.

#### Deleting records

Add a deletion record to the data file. Also referred as **tombstone**.

During compaction and merging, this tombstone is used to ignore and delete this key and associated values.

#### Crash recovery

In case the system crashes or is restarted, the index, i.e in-memory hash-map is lost.
It can be rebuilt from the data but it would take O(n).

Thus a **snapshot** of in-memory hash is stored on disk, probably periodically. This can help build the index quickly in case of restarts.

#### Partial writes

Crashes can happen during middle of write causing partial writes. These writes should be rolled back. The system maintains a **checksum** allowing corrupted parts of the log to be detected and ignored.

#### Concurrent writes

Keep a single writer thread. Since the files are apend-only and sequential so this approach works.
They can be read concurrently by multiple threads though.

#### Limitations
- The keyspace **must** fit in memory.
- Range queries are not efficient. Each key has to be looked up individually and value has to be seeked individually.

### SSTable

SSTable stands for Sorted String Table. It's a little deviation from the earlier format where writes to the database is immediately written to the append-log.
With SSTable database writes are not written to the append-log immediately. The goal is to write key in sorted order to the append-log.

A balanced tree is used as index in this case.
When database writes happen, the key and value are first added to an in-memory balanced tree. This balanced-tree is always remains sorted. This is referred as **memtable**.

Periodically a segment is created out of this sorted balanced tree when it's size exceeds a threshold.

SSTable provides the following advantages:
- Keyspace can be larger than available memory.
- Compaction and merging is simpler because segments stay sorted.
- Sparse index of SSTable on disk. This is possible because of sorted nature of data.
- Data compression can be achieved as read requests search for data in a block.

### Crash recovery

Separate log for memtable.

LSM Trees are based on SSTables.

## Page structured storage engine

### B-Trees

B-Trees are another indexing alternative.

The key-value are stored in a sorted fashion allowing for:
- Efficient key lookup
- Range queries

B-Trees store data in fixed-size blocks/pages rather than variable size segments.
They read or write one page at a time.

Every page contains keys or reference to other pages.
Leaf pages contain the value for key inline or reference to the page with value.

Branching factor - Number of references to child pages in one page of B-Tree.

B-Tree performs in-place write in case of updates.

A B-Tree with 4 KB pages and a branching factor of 500 can store (4KB * 500) data i.e 2 MB data.
With a four-level tree, the addressable storage is (4 KB * 500 * 500 * 500 * 500) i.e 250 TB.

#### Crash recovery

Every write operation is also written to a Write Ahead Log (WAL) also referred as Redo Log.
This is an append-only-file where every modification is written before being applied to the pages.

Redo Log is used for crash recovery.

#### Concurrency control

Concurrency control is more complex in page-oriented storages compared to Append-only-log storages like LSMTree.

One process might be reading from the page while other is simultaneouly trying to write to it.

**Latches**, lightweight locks, are used for concurrency control.

#### B-Tree optimizations

a. The pages might not be sequential. Example: 100 keys might be on one page while the next 100 on other page. These pages may be at a distance from each other.
Thus range queries for these 200 keys would need two seeks and head rotation.
One optimization could be keeping leaf pages in sequential order

b. Using more pointers. Example: Leaf pages keeping reference to sibling pages.

### Storing values with index

Two approaches:
1. Clustered Index: Either store value i.e row with index.
2. Heap file: Store value i.e row elsewhere. This is called heap file. The index has a reference to heap file.

Heap file approach prevents data duplication.

### Clustered Index

Index row value to be kept along with index on the same page rather than keeping a reference to a heap file.

In MySQL InnoDB, primary key is a clustered index. And secondary keys refer to the primary key.

### Covering Index

Covering index store value for some columns along with index.

Assume table `person` with 4 columns:
- id: primary key
- name
- age
- city
- gender

Let's assume MySQL InnoDB storage engine.

Thus there is a clustered index on id. Let's assume there is a secondary index on city.

  select name, age from person where city = 'Hyderabad'

In case there is a covering index on city with columns name and age covered, this query can be answered from the index.

However doing:

  select name, age, gender from person where city = 'Hyderabad'

It would need to scan the relevant rows to fetch the name.

### Graph databases

Used when there are a lot of different kind of entities and many many connections between these entities.

Examples:
- Web graph: Web pages related to each other
- Social graph: People, Locations, Check-ins, Photos etc.
- Road and rail networks

In relational database, the number of joins to get relevant data is usually known in advance. However, with graph database the number of joins or traversal needed to retrieve the relevant data is not known in advance.

Neo4j is a graph database. Cypher is the query language used to query Neo4j.

**Property Graph Model** is one of the data models used in graph databases.

## OLTP vs OLAP

Characteristics of an OLTP system:
- Interactive
- Real-time
- Low latency read and writes
- Highly available

OLAP system characteristics:
- Expensive queries
- Ad-hoc queries

OLAP queries might be highly expensive and could impact performance of other concurrently executing queries. Thus database administrators want to guard the OLTP systems and not allow analysts to use this system.

OLAP systems usually work with a separate database called Data Warehouses.

## Data warehouse

Optimized for analytic queries. An ETL process extracts the data from transactional systems, cleans and transforms the data and load into a data warehouse.

Data Warehouses also mostly have SQL like interface. However their internal storage layer implementation highly differ from OLTP systems because of the kind of access patterns and workloads they have to deal with.

Data warehouses usually have two types of tables:
- Facts table
- Dimension table

Facts table store the events. Dimension tables store additional information and more columns around the events. Facts table's columns have FK to dimension tables.

Usually Data warehouse tables are **wide**, i.e could have 100s of columns.

### Data warehouse models

There are usually two models:
- Star model
- Snowflake model

#### Star model

Fact table is at the center. And it is related to multiple dimension tables.

#### Snowflake model

Even dimensions table have relation or FK to more tables.

Snowflake model is usually more normalized than star model.

## Columnar storage

Data warehouses usually have column-oriented storage.
In OLTP systems, storage is laid out in a row-oriented fashion.

However in OLAP systems, we rarely need to access all columns of a particular row. Instead often we need to **select multiple rows** and apply **aggregation** on them.
Laying out data in a column-oriented fashion gives better throughput. Data warehouses usually have column-oriented storage layout.

### Compression

In Column-oriented storage the different data points have the same type and there are not a lot of distinct values. This gives opportunity to compress them.
very superior compression can be achieved in column-oriented storage.

Very often, **bitmap encoding** is used to perform compression.

### Reduced bandwidth

Less bandwidth to transfer compressed data from disk to memory.

### Efficient use of CPU cycle

At times, instead of using memory, query optimizers take compressed data and load into CPU's L1 cache and perform tight loops on this compressed data. **Tight loop** means no function calls.

### Vectorized processing

Operations are performed on compressed column data. This is called vectorized processing.
