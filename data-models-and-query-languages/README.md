## Data models

Data models are likely the most important part of an application. It has a **profound effect** on:
- How we write software
- How we think about the problem

Few data models are:
- Relational model
- Document model
- Graph model

### Relational model

Data is organized into relations. Each relation is an unordered collection of tuples.

Earlier tools forced application developers to think a lot about the internal representation of the data. The goal of the relational model was to hide the implementation details behind a cleaner interface.
We talked about Abstraction in Chapter 1 :)

### NoSQL Database

The driving forces behind the evolution of NoSQL database
- Scalability challenges of relational databases
- Restrictiveness of relational databases
- Impedance mismatch of relational databases
- Need for better data locality

#### Impedance mismatch

**Impedance mismatch** means the disconnect between the application objects and the database rows and the need of a translation layer to bridge this gap.

JSON model can reduce the impedance mismatch. MongoDB and CouchDB handle JSON data very well.

Many companies would adopt a hybrid approach of using a relational database alongside a non-relational datastore, referred as **polyglot persistence**.

---

### Document-oriented database

Document-oriented databases are more suited for self-contained documents. Example: Modelling a Linkedin resume.

A resume would have the following:
- name
- contact_number
- array of experiences
- array of positions held

Having a separate table for users, experiences and positions would be an overkill.
