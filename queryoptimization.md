**Note: contents presented here have been taken from an online [Database Zone tutorial](https://dzone.com/articles/how-to-optimize-mysql-queries-for-speed-and-perfor) authored by Francis Ndungu**

# QUERY OPTIMIZATION
Following tips can be useful in optimizing queries:

## Index all columns used in where, order by and group by clauses:

- An index allows MYSQL server to fetch results faster from a database. 
- It is also very useful when sorting records.
- Indexes may take up more storage space and decrease performance on inserts, deletes, and updates, however they can considerably reduce select query execution time.

Following examples will use tables from the sample **world** mysql database.
make sure you have switched to world database before running the examples.

```sql
select CountryCode , District, Population from city where name like 'Ka%';
```

The query above will force MySQL server to conduct a full table scan (start to finish) to retrieve the record we are searching.

MySQL has a special 'EXPLAIN' statement that you can use alongside select, delete, insert, replace and update statements to analyze your queries.

```sql
explain select CountryCode , District, Population from city where name like 'Ka%';
```


| id |select\_type | table | partitions | type |possible\_keys | key  | key\_len| ref  | rows | filtered | Extra       |
|----|-------------|-------|------------|------|---------------|------|---------|------|------|----------|-------------|
|  1 | SIMPLE      | city  | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 4187 |    11.11 | Using where |

The optimizer has displayed very important information that can help us to fine-tune our database table.
-	First, it is clear that MySQL will conduct a full table scan because key column is 'NULL'.
- Second, MySQL server has clearly indicated that it's going to conduct a full scan on the rows in our database.

To optimize the above query, we can just add an index to the 'Name' field using the below syntax:

```sql
 create index City_Name ON city(Name);
```

If we run the explain statement one more time, we will get the following:

| id | select\_type| table | partitions | type  | possible\_keys| key       | key\_len| ref  | rows | filtered | Extra                 |
|----|-------------|-------|------------|-------|---------------|-----------|---------|------|------|----------|-----------------------|
|  1 | SIMPLE      | city  | NULL       | range | City\_Name    |City\_Name | 35      | NULL |   89 |   100.00 | Using index condition |



## Optimize Like Statements With Union Clause

Sometimes, you may want to run queries using the comparison operator 'or' on different fields or columns in a particular table. When the 'or' keyword is used too much in where clause, it might make the MySQL optimizer to incorrectly choose a full table scan to retrieve a record.

A union clause can make the query run faster especially if you have an index that can optimize one side of the query and a different index to optimize the other side.

Example, consider a case where you are running the below query with the 'first\_name' and 'last\_name' indexed:

```sql
select * from students where first_name like  'Ade%'  or last_name like 'Ade%' ;
```

The query above can run far much slower compared to the below query which uses a union operator merge the results of 2 separate fast queries that takes advantage of the indexes.

```sql
select  from students where first_name like  'Ade%'  union all select  from students where last_name like  'Ade%' ;
```

## Avoid Like Expressions With Leading Wildcards

MySQL is not able to utilize indexes when there is a leading wildcard in a query. If we take our example above on the students table, a search like this will cause MySQL to perform full table scan even if you have indexed the 'first\_name' field on the students table.

## Take Advantage of MySQL Full-Text Searches

If you are faced with a situation where you need to search data using wildcards and you don't want your database to underperform, you should consider using MySQL full-text search (FTS) because it is far much faster than queries using wildcard characters.

Furthermore, FTS can also bring better and relevant results when you are searching a huge database.

To add a full-text search index to the students sample table, we can use the below MySQL command:

```sql
Alter table students ADD FULLTEXT (first_name, last_name);
Select * from students where match(first_name, last_name) AGAINST ('Ade');
```

In the above example, we have specified the columns that we want to be matched (first\_name and last\_name) against our search keyword ('Ade').



## Optimize Your Database Schema

Even if you optimize your MySQL queries and fail to come up with a good database structure, your database performance can still halt when your data increases.

 **Normalize Tables**

First, normalize all database tables even if it will involve some trade-offs. For instance, if you are creating two tables to hold customers data and orders, you should reference the customer on the orders table using the customer id as opposed to repeating the customer's name on the orders table. The latter will cause your database to bloat.

Always use the same data type for storing similar values even if they are on different tables, for instance, the schema uses 'INT' data type to store 'customer\_id' both in the customers and orders table.

**Use Optimal Data Types**

MySQL supports different data types including integer, float, double, date, date\_time, Varchar, and text, among others. When designing your tables, you should know that "shorter is always better."

For instances, if you are designing a system user's table which will hold less than 100 users, you should use 'TINYINT' data type for the 'user\_ id' field because it will accommodate all your values from -128 to 128.

**Avoid Null Values**

Null is the absence of any value in a column. You should avoid this kind of values whenever possible because they can harm your database results. For instance, if you want to get the sum of all orders in a database but a particular order record has a null amount, the expected result might misbehave unless you use MySQL 'ifnull' statement to return alternative value if a record is null.

In some cases, you might need to define a default value for a field if records don't have to include a mandatory value for that particular column/field.

**Avoid Too Many Columns** 

Wide tables can be extremely expensive and require more CPU time to process. If possible, don't go above a hundred unless your business logic specifically calls for this.

Instead of creating one wide table, consider splitting it apart in to logical structures. For instance, if you are creating a customer table but you realize a customer can have multiple addresses, it is better to create a separate table for holding customers addresses that refer back to the customers table using the 'customer\_id' field.

**Optimize Joins**

Always include fewer tables in your join statements. An SQL statement with poorly designed pattern that involves a lot of joins may not work well. A rule of thumb is to have utmost a dozen joins for each query.

## MySQL Query Caching
If your website or application performs a lot of select queries (e.g. WordPress), you should take advantage of MySQL query caching feature. This will speed up performance when read operations are conducted.

The technology works by caching the select query alongside the resulting data set. This makes the query run faster since they are fetched from memory if they are executed more than once. However, if your application updates the table frequently, this will invalidate any cached query and result set.

You can check if your MySQL server has query cache enabled by running the command below:

```sql 
show variables like 'have_query_cache';
```
**Setting the MySQL Server Query Cache**

You can set the MySQL query cache values by editing the configuration file ('/etc/mysql/my.cnf' or '/etc/mysql/mysql.conf.d/mysqld.cnf'). This will depend on your MySQL installation. Don't set a very large query cache size value because this will degrade the MySQL server due to cached overhead and locking. Values in the range of tens of megabytes are recommended.

To check the current value, use the command below:

```sql
show variables like 'query_cache_%' ;
```
Then to adjust the values, include the following on the MySQL configuration file:

query\_cache\_type=1
query\_cache\_size = 10M
query\_cache\_limit=256k

You can adjust the above values according to your server needs.

The directive 'query\_cache\_type=1' turns MySQL caching on if it was turned off by default.

The default 'query\_cache\_size' is 1MB and like we said above a value a range of around 10 MB is recommended. Also, the value must be over 40 KB otherwise MySQL server will throw a warning, "Query cache failed to set size".

The default 'query\_cache\_limit' is also 1MB. This value controls the amount of individual query result that can be can be cached.
