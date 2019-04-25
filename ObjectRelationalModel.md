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


**Poor representation of “real-world” entities:** The process of normalization generally leads to the creation of relations that do not correspond to entities in the “real-world.” The fragmentation of a “real-world” entity into many relations, with a physical representation that reflects this structure, is inefficient leading to many joins during query processing.

**Semantic overloading:** The relational model has only one construct for representing data and relationships between data, the relation. For example, to represent a many-to-many (*:*) relationship between two entities A and B, we create three relations, one to represent each of the entities A and B and one to represent the relationship. There is no mechanism to distinguish between entities and relationships, or to distinguish between different kinds of relationship that exist between entities. For example, a 1:* relationship might be Has, Owns, Manages, and so on. If such distinctions could be made, then it might be possible to build the semantics into the operations. It is said that the relational model is semantically overloaded.

 **Poor support for integrity and enterprise constraints:**
Integrity refers to the validity and consistency of stored data. Integrity is usually expressed in terms of constraints, which are consistency rules that the database is not permitted to violate. of constraint.  
Unfortunately, many commercial systems do not fully support these constraints and it is necessary to build them into the applications. This, of course, is dangerous and can lead to duplication of effort and, worse still, inconsistencies.
Furthermore, there is no support for general constraints in the relational model, which again means that they have to be built into the DBMS or the application.

**Homogeneous data structure:**
The relational model assumes both horizontal and vertical homogeneity. Horizontal homogeneity means that each tuple of a relation must be composed of the same attributes. Vertical homogeneity means that the values in a particular column of a
relation must all come from the same domain. Additionally, the intersection of a row and column must be an atomic value. This fixed structure is too restrictive for many “real-world” objects that have a complex structure, and it leads to unnatural
joins, which are inefficient as mentioned previously.  

Many RDBMSs now allow the storage of binary large objects (BLOBs). A BLOB is a data value that contains binary information representing an image, a digitized video or audio sequence, a procedure, or any large unstructured object. The DBMS
does not have any knowledge concerning the content of the BLOB or its internal structure. This prevents the DBMS from performing queries and operations on inherently rich and structured data types. Typically, the database does not manage
this information directly, but simply contains a reference to a file. The use of BLOBs is not an elegant solution and storing this information in external files denies it many of the protections naturally afforded by the DBMS.

**Limited operations:** The relational model has only a fixed set of operations, such as set and tuple-oriented
operations, operations that are provided in the SQL specification. However, SQL does not allow new operations to be specified. Again, this is too restrictive to model the behavior of many real-world objects. For example, a GIS application typically uses points, lines, line groups, and polygons, and needs operations for distance, intersection, and containment.

**Impedance Mismatch:** Until the 1999 release of the standard, known as SQL:1999 or SQL3, SQL contained only these definitional and manipulative commands; it did not contain flow of control commands, such as IF . . . THEN . . . ELSE, GO TO, or DO . . .WHILE. These commands had to be implemented using a programming or job-control language, or interactively by the decisions of the user. Owing to this lack of computational completeness, SQL can be used in two ways. The first way is to use SQL interactively by entering the statements at a terminal. The second way is to embed SQL statements in a procedural language.  However, this approach produces an impedance mismatch, because we are mixing different programming paradigms:

- SQL is a declarative language that handles rows of data, whereas a high-level language such as C is a procedural language that can handle only one row of data at a time.
- SQL and 3GLs use different models to represent data. For example, SQL provides the built-in data types Date and Interval, which are not available in traditional programming languages.

Thus it is necessary for the application program to convert between the two representations, which is inefficient both in programming effort and in the use of runtime resources. It has been estimated that as much as 30% of programming effort and code space is expended on this type of conversion. Furthermore, as we are using two different type systems, it is not possible to type check the application as a whole automatically. It is argued that the solution to these problems is not to replace relational languages by row-level object-oriented languages, but to introduce set-level facilities into programming languages.

The latest releases of the SQL standard, SQL:2003, SQL:2006, SQL:2008, and SQL:2011 address some of these deficiencies with the introduction of many new features, such as the ability to define new data types and operations as part of the data definition language, and the addition of new constructs to make the language computationally complete. 













