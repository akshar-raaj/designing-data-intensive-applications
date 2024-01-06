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
  - In terms of data volume
  - In terms of write througput
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

#### Limitations

Document database suffer from following limitations.
1. Difficult to model many-to-one relationship or many-to-many relationship. One-to-many is easy.
2. Weak join support.

Consider two entities.
- Country
- Person

Many person belong to same country. We need to operate on person entity independently of country. Thus person cannot be embedded in country. Hence country reference will have to be kept in person.
With document database the responsibility of join shifts from database to application code.

If we want to avoid join, the country info would have to be duplicated for each person. This would lead to data denormalization with it's own challenges.
- Data duplication
- Write overheads during update
- Possibility of losing data integrity during updates

In such cases, document-oriented databases become **less appealing**.

### More comparison

Relational database enforce a fixed schema at write time. Document database do not enforce a schema, the schema is only inferred at read time by client.
This is very similar to how static type languages differ from dynamic type languages.

### Relational Database vs Document-oriented database

| Relational Database | Document-oriented database |
|---------------------|------------------------------|
| Difficult to scale-out | More amenable to scale-out |
| Restrictive schema | Permissive schema |
| Impedance mismatch | Data representation more close to application code | 
| No data locality. More disk seeks might be needed to pull related data | Better data locality |
| Easy to model many-to-one | Difficult to model many-to-one |
| Joins can be natively performed | Weak join support. Join responsibility shifted to application code |

---

## Query Language

A query language can be categorized into:
- Declarative language
- Imperative language

Declarative language specifies the **what**. It doesn't specify **how**. Example: SQL
SQL allows users to say I want the following columns from a particular relation fulfilling a particular condition. It is upto the Query optimizer to use appropriate index to provide better performance.
Hence in SQL users only specify **what**. The implementation details are figured out by the query optimizer.

Imperative language expects the users to know how the data is internally structured and which sequence of instructions to execute.

---

## Miscellaneous

- Alter table to add a column is highly expensive in MySQL. The whole table needs to be moved on disk.
- Updating value in document-database is a normal operation. However if the document size needs to change, the document needs to be moved on disk which can be expensive.
