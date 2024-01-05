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
