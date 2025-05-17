# Lab Tutorial: Data Models - JSON in MySQL vs. UDTs in PostgreSQL

## Course: Database Administration and Management
## Lecture: Introduction to Data Models
## Objective:
This lab aims to illustrate the advantages of using PostgreSQL's user-defined types (UDTs) and advanced features over traditional approaches like storing JSON in MySQL. Students will compare how both databases handle structured data, specifically contact information, through various operations (create, insert, update, select).

## Introduction

In this lab, we'll explore how different database systems handle complex data. We'll use a common example: storing address information. We'll compare two approaches:
- MySQL: Storing address information as a JSON string.
- PostgreSQL: Storing address information using a user-defined type (UDT).
By the end of this lab, you should understand the benefits of using UDTs in PostgreSQL for managing structured data, including improved data integrity, query efficiency, and maintainability.


## MySQL Setup and JSON Example

### Create a MySQL Database and Table
Connect to your MySQL server using the command-line client 
Create a new database:

```sql
CREATE DATABASE address_data_mysql;
USE address_data_mysql;


Create a table to store address information as JSON:
CREATE TABLE contacts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255),
    address JSON
);
```

###  Insert Data into the MySQL Table
Insert some sample data into the contacts table:

```sql
INSERT INTO contacts (name, address) VALUES
    ('John Doe', '{"house_no": "10", "street_no": "2", "road": "Main Road", "city": "Anytown", "postal_code": "12345"}'),
    ('Jane Smith', '{"house_no": "25", "street_no": "5", "road": "Oak Avenue", "city": "Somecity", "postal_code": "67890"}'),
    ('Peter Jones', '{"house_no": "1", "street_no": "1", "road": "First Street", "city": "Villagetown", "postal_code": "54321"}');
```

### Query Data from the MySQL Table 
Select all data:

```sql
SELECT * FROM contacts;


Select specific data within the JSON:
SELECT name, address->>"$.city" AS city FROM contacts;
```

Note the use of ->> to extract the city as text. Without this, it might be returned as a JSON value.

### Update data within the JSON:

```sql
UPDATE contacts
SET address = JSON_SET(address, '$.city', 'NewCity')
WHERE name = 'John Doe';
```

### Challenges with JSON in MySQL

- Data Integrity: MySQL doesn't enforce a strict schema on the JSON data. It's possible to have inconsistencies in the structure of the JSON documents. For example, some addresses might have "street_no", while others have "street".
- Query Complexity: Querying data within the JSON document can become complex, especially for nested structures. The JSON path syntax can be cumbersome.
- Performance: While MySQL has JSON functions, querying and updating JSON data can be less efficient than working with native data types.
- Type Safety: JSON fields can hold any type of data, potentially leading to unexpected results or errors if you expect a specific type.

4. PostgreSQL Setup and UDT Example
### Create a PostgreSQL Database
Connect to your PostgreSQL server using the psql command-line client or pgAdmin.
Create a new database:
```sql
CREATE DATABASE address_data_pg;
\c address_data_pg
```

### Create a User-Defined Type (UDT) in PostgreSQL

Create a UDT to represent an address:

```sql
CREATE TYPE address AS (
    house_no VARCHAR(10),
    street_no VARCHAR(10),
    road VARCHAR(255),
    city VARCHAR(255),
    postal_code VARCHAR(20)
);
```

### Create a Table in PostgreSQL Using the UDT
Create a table that uses the address UDT:

```sql
CREATE TABLE contacts_pg (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    address address
);
```

### Insert Data into the PostgreSQL Table
Insert data into the contacts_pg table, using the UDT constructor:

```sql
INSERT INTO contacts_pg (name, address) VALUES
    ('John Doe', ROW('10', '2', 'Main Road', 'Anytown', '12345')::address),
    ('Jane Smith', ROW('25', '5', 'Oak Avenue', 'Somecity', '67890')::address),
    ('Peter Jones', ROW('1', '1', 'First Street', 'Villagetown', '54321')::address);
```

### Query Data from the PostgreSQL Table

Select all data:

```sql
SELECT * FROM contacts_pg;
```

Select specific fields from the UDT:

```sql
SELECT name, (address).city FROM contacts_pg;
```

Note the more natural way to access fields within the UDT using (address).city.
Update data within the UDT:

```sql
UPDATE contacts_pg
SET address = ROW(
    (address).house_no,
    (address).street_no,
    (address).road,
    'NewCity',  -- Updated city
    (address).postal_code
)::address
WHERE name = 'John Doe';

Or

UPDATE contacts_pg
SET address.city = 'NewCity'
WHERE name = 'John Doe';
```

### Advantages of UDTs in PostgreSQL
- Data Integrity: The UDT enforces a strict schema for the address data. PostgreSQL ensures that all address values have the correct fields and data types.
- Query Efficiency: Querying data within a UDT is more efficient than parsing JSON. PostgreSQL can optimize queries based on the UDT's structure.
- Readability and Maintainability: The code is more readable and easier to maintain. Using (address).city is more intuitive than using JSON path expressions.
- Type Safety: UDTs provide type safety. You define the data type of each field within the UDT, preventing unexpected data type issues.
- Data Modeling: UDTs allow you to model your data more closely to the real-world domain.

## Comparison and Discussion
| Feature          | MySQL (JSON)                               | PostgreSQL (UDT)                               |
| ---------------- | ------------------------------------------ | ----------------------------------------------- |
| Data Integrity   | Low                                        | High                                              |
| Query Complexity | High                                        | Low                                               |
| Performance      | Lower for complex queries                    | Higher for complex queries                    |
| Readability      | Lower                                        | Higher                                            |
| Type Safety      | Low                                        | High                                              |
| Data Modeling    | Less intuitive                             | More intuitive                                 |
| Flexibility      | High (schema-less within the JSON field) | Lower (strict schema defined by UDT)            |

## Discussion Points:
When is using JSON in MySQL appropriate? (e.g., for highly flexible, semi-structured data where you don't need to query deeply)
When are UDTs in PostgreSQL a better choice? (e.g., for structured data that needs to be queried efficiently and with high data integrity)
How do UDTs improve the overall design and maintainability of a database?
Explore other advanced PostgreSQL features like domains, and discuss how they compare to UDTs.
## Conclusion
This lab demonstrated the differences between using JSON in MySQL and UDTs in PostgreSQL for storing structured data. While JSON offers flexibility, PostgreSQL's UDTs provide superior data integrity, query efficiency, and type safety. For applications that require robust data management and efficient querying of structured data, PostgreSQL's UDTs are a powerful tool.
