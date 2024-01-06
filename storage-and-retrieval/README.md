A data store provides two major functionalities:
- Data storage
- Data retrieval

Even for an application developer, who is never going to implement his own **storage engine**, it makes sense to have a **rough idea** of the storage engine to **tune** it according to the **workload**.

Of the many families of storage engines, two worth mentioning are:
- Log-structured storage engine
- Page-oriented storage engine

## Log structure storage engine

A log structured storage engine keeps writing to an append only data file.

A key-value store can be easily implemented using a log-structured storage engine. Let's assume the data is:
{'name': 'bob', 'place': {'city': 'Hyderabad', 'country': 'India'}}

It would look like the following in file.
name,bob
place,{'city': 'Hyderabad', 'country': 'India'}

It would look like the following on disk:
n a m e , b o b \n p l a c e , { c i t y : H

Each character would take say 1 byte on disk. Every byte is separated from the next by a space.

Writing a key-value pair can be very performant as it's append-only. Worth mentioning, it would be sequential write and not random write.
However reading from this file can be O(n) if there is no index.

Two things worth mentioning about indexes:
- It is metadata, and not actual data. Index can be deleted without deleting the actual data.
- It's *derived* from real data.

The downside of indexes are they create **overheads** during write. The **benefit** is it improves read performance.

An in-memory hash-map can be used as an index. The key of hash-map would be the same key as that used during writing. The value would be the byte offset in the file.
The hash-map would look like:
{name: 0, place: 9}
This hash-map can make even reads operation as O(1). Only one file **seek** would have to be performed.

This storage-engine is used in **Bitcask** storage engine of **Riak** database.

This storage engine can be very efficient if all keys can fit in the memory.

### Deal with growing file size

If a single apend-only file is used it can grow, become unmanageable and can reach the hard-disk limit.

- Use segments instead of a single large file.
- Compaction and merging of segments

### Deleting records

Add a deletion record to the data file. Also referred as **tombstone**.

During compaction and merging, this tombstone is used to ignore and delete this key and associated values.

### Crash recovery

In case the system crashes or is restarted, the index, i.e in-memory hash-map is lost.
It can be rebuilt from the data but it would take O(n).

Thus a **snapshot** of in-memory hash is stored on disk, probably periodically. This can help build the index quickly in case of restarts.

### Partial writes

Crashes can happen during middle of write causing partial writes. These writes should be rolled back. The system maintains a **checksum** allowing corrupted parts of the log to be detected and ignored.

### Concurrent writes

Keep a single writer thread. Since the files are apend-only and sequential so this approach works.
They can be read concurrently by multiple threads though.

## Page-oriented storage engine
