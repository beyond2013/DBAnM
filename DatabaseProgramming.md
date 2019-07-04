Note: Following contents have been taken from [MySQL Documentation](https://dev.mysql.com/doc/connector-net/en/connector-net-tutorials-stored-procedures.html).

# What is a stored procedure

A stored routine is a set of SQL statements that can be stored in the server.

A stored procedure is a method to encapsulate repetitive tasks. They allow for variable declarations, flow control and other useful programming techniques.

# Working with Stored Procedures

 Putting database-intensive operations into stored procedures lets you define an API for your database application. You can reuse this API across multiple applications and multiple programming languages. This technique avoids duplicating database code, saving time and effort when you make updates due to schema changes, tune the performance of queries, or add new database operations for logging, security, and so on. 

In the MySQL Client program, connect to the world database and enter the following stored procedure:

```sql
DELIMITER //
CREATE PROCEDURE country_hos
(IN con CHAR(20))
BEGIN
  SELECT Name, HeadOfState FROM country
  WHERE Continent = con;
END //
```

Test that the stored procedure works as expected by typing the following into the mysql command interpreter:

```sql
CALL country_hos('Europe');
```
## Delimiter
The delimiter is the character or string of characters that you'll use to tell the mySQL client that you've finished typing in an SQL statement. For ages, the delimiter has always been a semicolon. That, however, causes problems, because, in a stored procedure, one can have many statements, and each must end with a semicolon. In the example code above delimiter is //

## Modify a stored Procedure 
MySQL provides an ALTER PROCEDURE statement to modify a routine, but only allows for the ability to change certain characteristics. If you need to alter the body or the parameters, you must drop and recreate the procedure.

## Delete a Stored Procedure

```sql 
DROP PROCEDURE IF EXISTS p2;
```

## Parameters
Let's examine how you can define parameters within a stored procedure.

- CREATE PROCEDURE proc1 () : Parameter list is empty  
- CREATE PROCEDURE proc1 (IN varname DATA-TYPE) : One input parameter. The word IN is optional because parameters are IN (input) by default.  
- CREATE PROCEDURE proc1 (OUT varname DATA-TYPE) : One output parameter.  
- CREATE PROCEDURE proc1 (INOUT varname DATA-TYPE) : One parameter which is both input and output.  

Of course, you can define multiple parameters defined with different types.

### IN example

```sql
 DELIMITER //
CREATE PROCEDURE `proc_IN` (IN var1 INT)
BEGIN
    SELECT var1 + 2 AS result;
END//
```

### OUT example
```sql
DELIMITER //
CREATE PROCEDURE `proc_OUT` (OUT var1 VARCHAR(100))
BEGIN
    SET var1 = 'This is a test';
END //

DELIMITER ;

CALL proc_OUT(@var1);
```

``

### OUT example

```sql
DELIMITER //
CREATE PROCEDURE `proc_OUT` (OUT var1 VARCHAR(100))
BEGIN
    SET var1 = 'This is a test';
END //
```


### INOUT example

```sql
delimiter $
create procedure gradePoint(IN marks_obt INT, OUT GP FLOAT(2,1)) 
	begin 
	IF marks_obt <= 60 and marks_obt > 50 then
	set GP = 1.0;
	elseif marks_obt <=70  and marks_obt > 60 then
	set GP = 2.0;
	end if;
	end$
Delimiter ;
```

## Variables


## List all procedures defined within a database

```sql
show procedure status where db = 'database_name';
```

The following step will teach you how to define variables, and store values inside a procedure. You must declare them explicitly at the start of the BEGIN/END block, along with their data types. Once you've declared a variable, you can use it anywhere that you could use a session variable, or literal, or column name.

Declare a variable using the following syntax:

DECLARE varname DATA-TYPE DEFAULT defaultvalue;
Let's declare a few variables:

```sql
DECLARE a, b INT DEFAULT 5;
 
DECLARE str VARCHAR(50);
 
DECLARE today TIMESTAMP DEFAULT CURRENT_DATE;
 
DECLARE v1, v2, v3 TINYINT;
```

### Working with variables

Once the variables have been declared, you can assign them values using the SET or SELECT command:

```sql
DELIMITER //
 
CREATE PROCEDURE `var_proc` (IN paramstr VARCHAR(20))
BEGIN
    DECLARE a, b INT DEFAULT 5;
    DECLARE str VARCHAR(50);
    DECLARE today TIMESTAMP DEFAULT CURRENT_DATE;
    DECLARE v1, v2, v3 TINYINT;    
 
    INSERT INTO table1 VALUES (a);
    SET str = 'I am a string';
    SELECT CONCAT(str,paramstr), today FROM table2 WHERE b >=5; 
END //
```
