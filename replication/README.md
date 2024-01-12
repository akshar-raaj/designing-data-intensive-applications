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
