## Distributed Data

Multiple machines are involved in storage and retrieval of data.

### Reasons

Reasons to distribute a database across multiple machines:

- Availability and reliability

Keep a replica or copy of data on multiple machines for increased availability. When one machine fails, another can take-over.

- Scalability

Single machine might not have storage or compute capacity to handle the load.

- Low latency

Keeping copy of data geographically closer to customer for low latency.

We need to be aware about the **constraints** and **trade-offs** of distributed data or systems.

Two common approaches to distribute data across multiple nodes:
1. Replication
2. Partitioning

## Replication

Keeping a copy of data on multiple machines or nodes.

Replication algorithm
- Single-leader
- Multi-leader
- Leaderless

Two approaches:
- Synchronous replication
- Asynchronous replication

### Leaders and Followers

- Writes happen on leader
- Leader sends the replication log to followers

Writes are only accepted on the leader.

Leader based replication is not only limited to database. It's used by other systems like:
- RabbitMQ
- Kafka

### Synchronous replication

Advantage:
- Follower is guaranteed to have a copy consistent with the leader

Trade-off:
- Leader must block until the follower acknowledges. Leads to higher latency.

### Asynchronous replication

Advantage:
- Leader can proceed with writes without waiting for data to be copied on the replicas.

Trade-off:
- Durability guarantee is compromised. In case there is a leader outage before data has been copied over to any follower, this data would have to be discarded.

### Setting up new followers

- Copy leader snapshot on follower
- Snapshot must be associated with **binlog coordinates**.
- Follower requests all changes after the binlog coordinates from leader and applies them
- Follower now **caught up** with leader.

### Replication Log implementation

Two approaches:
- Statement based replication
- WAL shipping
- Row based replication

#### Statement-based replication

Challenges:
- Problems in dealing with non-deterministic functions like now() or rand().

Advantages:
- Compact

#### WAL shipping

The write-ahead-log is shipped to the replicas.

Trade-off:
- Replication closely coupled with storage engine.
- Thus leader and followers must use the same versions, storage engines etc.

#### Row-based replication

Also referred as logical replication. The replication process is decoupled from the storage engine.

Replication Logs provide row level granularity telling the rows which changed along with new values. Same for updates, deletes etc.
