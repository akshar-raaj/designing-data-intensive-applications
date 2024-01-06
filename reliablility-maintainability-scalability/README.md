## Reliable, Scalable and Maintainable Applications

Three major **goals** of any system:
- Reliability
- Maintainability
- Scalability

### Reliability

The system should be **robust** and **resilient**.

The system should perform its intended **function** at desired level of **performance** even in the face of **adversity**.

### Scalability

As the system grows, there should be reasonable ways of dealing with that growth.

The growth could be in the following form:
- Traffic growth
- Data volume growth
- Complexity growth

### Maintainability

It should be easy to add **enhancements** and **evolve** the system over time.

---

### Reliability

System should be fault-tolerant/resilient. It should be able to **anticipate** the things that can go wrong and be prepared for it.

There are usually two kinds of errors that can happen
- Hardware faults
- Software Errors

#### Hardware Faults

- Hard Disk crash
- Power grid blackout
- Faulty RAM

We can plan for hardware faults by increasing **redundancy** of the system. Examples:
- Raid configuration for Hard Disk
- Dual Power Supply for CPU

Software Errors should be anticipated well and the system should be extensively tested to avoid these.

Cost of an unreliable system
- Productivity Loss
- Revenue loss

### Scalability

Scalability is the ability of the system to cope up with increased load.

Load can be described by **load parameters**.

Examples of load parameters
- Number of requests per second to a web server
- Number of concurrent users
- Ratio of read to write in a database

This section discusses an interesting use case of Twitter. Check it [here](../use-cases/twitter/).

We should be measuring the current **performance** of the system under a baseline load. And then see how the performance is impacted or degrades when the load increases.

Thus we should have a way to measure the **performance**. Some performance metrics are:
- Throughput
- Response time, **incorrectly** referred as Latency.

This chapter briefly talks about two concepts related to Response time
- Network delay
- Queueing delay

Very often we measure **average response time**. However it's not a very useful metric. A better metric would be in terms of **percentiles**.

- **Head of line blocking** can significantly affect the response time at high percentiles.

#### Coping with Load

How to maintain a desired **performance** when **load parameters** increase.

Systems can scale in two directions:
- Scale up: Vertical scaling
- Scale out: Horizontal scaling

Systems can scale in two ways:
- Auto Scaling. Can also be referred as Elastic scaling
- Manual Scaling. Gives more confidence and would have less operational surprises.

Common wisdom:
- Scale out with stateless services, like application servers.
- Scale up stateful services, like database.

### Maintainability
- Operability
- Simplicity
- Evolvability

#### Operability

Make it easy to operate the system. Things like the following should be easy
- Understanding system health
- Investigating issues

Thus the following can be considered as part of maintainable systems
- Monitoring
- Logging

#### Simplicity

Simple is better than complex. We should write proper abstractions.

Abstraction: Hide complex implementation details behind a simple and clean facade.

#### Evolvability

System is amenable to change and enhancements.

Following practices can aide in keeping an evolvable system
- Agile methodologies
- TDD

### Miscellaneous

- AWS guarantees elasticity and makes reliablity trade-off. AWS Instances have been known to stop abruptly.
- p95, p99, and high percentile response times are usually high. This can happen when a slow request keeps the CPU cores occupied and this request has a long queue time.
