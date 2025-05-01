# Data Models

## What is the relevance of various types of data models to this course?

Understanding various types of data models is foundational to Database Administration and Management.

1. Foundation for Administrative Decisions: As a database administrator (DBA) or manager, one of your first responsibilities is selecting, designing, and maintaining the right type of database system. To make informed decisions, you must understand:

- Which data model suits the application domain (relational vs. document vs. object-oriented)
- How it affects data storage, retrieval, scalability, and performance
- Which models support distributed databases, schema flexibility, or real-time processing

2. Supports Physical Design and Tuning : Data models directly influence:

- Indexing strategies
- Storage structure
- Normalization and denormalization trade-offs
- Query optimization techniques

If you do not grasp the conceptual structure behind a model, you will struggle with low-level tasks like tuning or query optimization.

3. Shapes Security and Integrity Mechanisms: Different models offer varying levels of:

- Access control granularity
- Constraint enforcement
- Transaction handling

For example, relational databases enforce referential integrity natively, while NoSQL systems like MongoDB require application-level enforcement.

4. Drives the Choice of DBMS Tools and Techniques: Many DBMS systems are optimized for a specific data model. For example:


Data Model       |	Common DBMS Examples
-----------------|-------------------------
Relational       |	MySQL, PostgreSQL, Oracle
Document         |	MongoDB, Couchbase
Key-Value        |	Redis, DynamoDB
Object-Oriented  |	ObjectDB, db4o (legacy)
Graph	         |  Neo4j, ArangoDB

The DBA must know what is under the hood of each system to install, configure, and manage it effectively.

5. Enables Future-Readiness: Your course includes emerging trends. Newer  like multi-model databases, graph databases, and NewSQL systems are being increasingly adopted. Without a grounding in traditional and advanced models, you will find it hard to transition.

**Summary: Why It Matters**
Understanding data models is not just academic theory — it is how a database administrator aligns the architecture of the data with the goals of the organization.

## Real world buisness / organization and data models 

You are already familiar, and interact with various businesses and organizations. Each organization has different information need and use data models most suitable to fulfill these needs. 

- NADRA: Requires a relational or object-relational model for complex citizen profiles with strict integrity, centralized records, and identity management.
- FBR: Needs relational models for tax data, but also data warehouses for analytics and fraud detection.
- Railways: Traditional relational model, possibly moving toward graph models to track routes, connections, and logistics.
- Jazz/Upaisa, Ufone: Use relational + NoSQL hybrids; transactional systems are relational, but wallets, logs, and analytics often use document or column-based models.
- Commercial Banks & SBP: Strong consistency and ACID compliance mean relational databases are dominant. For real-time fraud detection or risk models, graph databases and streaming platforms are emerging.
- Meta (Facebook): Uses graph databases for user connections and relationships; document stores and key-value stores for fast access to posts, messages, media.
- Google: Uses Bigtable (column-oriented), Spanner (NewSQL, globally distributed relational), and Firebase (NoSQL) depending on service.

## Definition of Data Model

A data model is a collection of conceptual tools for describing data, data relationships, data semantics, and consistency constraints.

## Types of Data Models:

### Relational Model

- Based on tables (relations)
- Most widely used
- Examples: MySQL, PostgreSQL, Oracle, SQL Server

#### Limitations of Relational Model:

- Poor at handling complex data types (e.g., multimedia, geospatial)
- Inadequate support for encapsulation and inheritance

These limitations led to advanced data models like Object-Relational and Object-Oriented models.


### Object-Oriented Data Model (OODM)

#### Concept
Extends the object-oriented programming (OOP) paradigm to databases. Suitable for complex applications such as CAD/CAM, multimedia systems, and AI.

#### Key Features

- Objects: Encapsulate both state (data) and behavior (methods)
- Classes: Blueprints for objects
- Inheritance: Classes can inherit properties from other classes
- Encapsulation: Data and methods bundled together
- Identity: Objects have a unique identity (OID)

```cpp
class Employee {
  string name;
  int id;
  virtual void calculateSalary();
};
``` 
#### Advantages:

- Intuitive for object-oriented programmers
- Handles complex data more naturally

#### Example Databases: db4o, ObjectDB, Versant

### Object-Relational Data Model (ORDBMS)

#### Concept:
Bridges the gap between relational and object-oriented models by extending the relational model to support objects.

#### Key Features:

- Support for complex data types
- User-defined types (UDTs)
- Inheritance and encapsulation (to some extent)
- Table-based structure with extended capabilities

Example (PostgreSQL):

```sql
CREATE TYPE address AS (
  street VARCHAR,
  city VARCHAR,
  zipcode VARCHAR
);

CREATE TABLE person (
  id SERIAL PRIMARY KEY,
  name VARCHAR,
  home_address address
);
```
#### Advantages:

- Combines robustness of relational model with flexibility of OOP
- Easier migration for organizations already using RDBMS
#### Popular Systems: PostgreSQL, Oracle, IBM Db2

### Comparison of Data Models

Feature         | Relational Model	    | Object-Oriented Model | Object-Relational Model
----------------|-----------------------|-----------------------|-------------------------
Structure	    | Tables                | Objects/Classes       | Tables + Objects 
Inheritance     | not supported         | Supported             | Partial support
Encapsulation   | not supported	        | Supported             | Partial support
Query Language	| SQL	                | OQL/Object methods	| Extended SQL
Use Case	    | General-purpose	    | Complex data systems	| Enterprise systems


