**Credits: Contents presented here have been taken from [MySQL online reference manual](https://dev.mysql.com/doc/refman/8.0/en/internal-locking.html)**

# Locking in MySQL

## Internal Locking Methods
This section discusses internal locking; that is, locking performed within the MySQL server itself to manage contention for table contents by multiple sessions. This type of locking is internal because it is performed entirely by the server and involves no other programs.

- Row-Level Locking
- Table-Level Locking
- Choosing the Type of Locking

Deadlocks affect performance rather than representing a serious error, because InnoDB automatically detects deadlock conditions by default and rolls back one of the affected transactions.



