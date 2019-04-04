Note: Contents of this text have been taken from Chapter 9 **Object-Relational DBMSs** of the book **Database Systems: A practical Approach to Design, Implementation, and Management** by **Thomas Connolly** and **Carolyn Begg**

# Object Relational DBMSs

- Object orientation is an approach to software construction that has shown considerable promise for solving some of the classic problems of software development.  
- The underlying concept behind object technology is that all software should be constructed out of standard, reusable components wherever possible.   
- Traditionally,software engineering and database management have existed as separate disciplines.   
- Database technology has concentrated on the static aspects of information storage, while software engineering has modeled the dynamic aspects of software.  

- With the arrival of the third generation of database management systems, namely Object-Relational Database Management Systems (ORDBMSs) and Object-Oriented Database Management Systems (OODBMSs), the two disciplines have been combined to allow the concurrent modeling of both data and the processes acting upon the data.

## Advance Database Applications
- Existing RDBMSs have proven inadequate for applications whose needs are quite different from those of traditional business database applications. These applications include:  

+ computer-aided design (CAD);
+ computer-aided manufacturing (CAM);
+ computer-aided software engineering (CASE);
+ network management systems;
+ office information systems (OIS) and multimedia systems;
+ digital publishing;
+ geographic information systems (GIS);
+ interactive and dynamic Web sites.

Other advanced database applications include:  
+ Scientific and medical applications, which may store complex data representing systems such as molecular models for synthetic chemical compounds and genetic material.  
+ Expert systems, which may store knowledge and rule bases for AI applications.  

## Weaknesses of RDBMSs

1. Poor representation of “real-world” entities
2. Semantic overloading
3. Poor support for integrity and enterprise constraints
4. Homogeneous data structure
5. Limited operations
6. Difficulty handling recursive queries
7. Impedance mismatch
8. Other problems with RDBMSs associated with concurrency, schema changes, and poor
navigational access

1. Poor representation of “real-world” entities: The process of normalization generally leads to the creation of relations that do not correspond to entities in the “real-world.” The fragmentation of a “real-world” entity into many relations, with a physical representation that reflects this structure, is inefficient leading to many joins during query processing.

2. The relational model has only one construct for representing data and relationships between data: the relation. For example, to represent a many-to-many (*:*) relationship between two entities A and B, we create three relations, one to represent each of the entities A and B and one to represent the relationship. There is no mechanism to distinguish between entities and relationships, or to distinguish between different kinds of relationship that exist between entities. For example, a 1:* relationship might be Has, Owns, Manages, and so on. If such distinctions could be made, then it might be possible to build the semantics into the operations. It is said that the relational model is semantically overloaded.

3. Integrity refers to the validity and consistency of stored data. Integrity is usually expressed in terms of constraints, which are consistency rules that the database is not permitted to violate. of constraint.  
Unfortunately, many commercial systems do not fully support these constraints and it is necessary to build them into the applications. This, of course, is dangerous and can lead to duplication of effort and, worse still, inconsistencies.
Furthermore, there is no support for general constraints in the relational model, which again means that they have to be built into the DBMS or the application.

4. Homogeneous data structure
The relational model assumes both horizontal and vertical homogeneity. Horizontal homogeneity means that each tuple of a relation must be composed of the same attributes. Vertical homogeneity means that the values in a particular column of a
relation must all come from the same domain. Additionally, the intersection of a row and column must be an atomic value. This fixed structure is too restrictive for many “real-world” objects that have a complex structure, and it leads to unnatural
joins, which are inefficient as mentioned previously.










