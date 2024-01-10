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
