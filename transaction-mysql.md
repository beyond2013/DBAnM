Credit:
1. contents presented here have been taken from [mysql online documentation](https://dev.mysql.com/doc/refman/8.0/en/commit.html)
2. banking fund transfer example has been taken from [w3resource](https://www.w3resource.com/mysql/mysql-transaction.php)

# Writing transactions in mysql

## syntax

```sql
START TRANSACTION
    [transaction_characteristic [, transaction_characteristic] ...]

transaction_characteristic: {
    WITH CONSISTENT SNAPSHOT
  | READ WRITE
  | READ ONLY
}

BEGIN [WORK]
COMMIT [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
ROLLBACK [WORK] [AND [NO] CHAIN] [[NO] RELEASE]
SET autocommit = {0 | 1}
```

- START TRANSACTION or BEGIN start a new transaction.
- COMMIT commits the current transaction, making its changes permanent.
- ROLLBACK rolls back the current transaction, canceling its changes.
- SET autocommit disables or enables the default autocommit mode for the current session.


By default, MySQL runs with autocommit mode enabled. This means that, when not otherwise inside a transaction, each statement is atomic, as if it were surrounded by START TRANSACTION and COMMIT. You cannot use ROLLBACK to undo the effect; however, if an error occurs during statement execution, the statement is rolled back.

To disable autocommit mode implicitly for a single series of statements, use the START TRANSACTION statement:

## Example
To understand the concept of a transaction, consider a banking database. Suppose a bank customer transfers money from his account (1111) to another account, then more than one SQL statements need to be executed.

1. To Debit 1111 a/c.
2. To Credit 2222 a/c.
3. Record in transaction journal


- Create the journal table

```sql
create table journal
(
date Date,
description varchar(50),
Sender varchar(10),
Receiver varchar(10),
Amount decimal(15,2)
) Engine=InnoDB;
```

- Create account table

```sql
create table account
(
AccountNo varchar(10), 
balance decimal(15,2) 
) Engine=InnoDB;
```
- Insert some data in account table

```sql
insert into account values('1111', 10500.90);
insert into account values('2222', 12000.00);
```

- Write a transaction that transfers 2000 from account 1111 to 2222

```sql
start transaction;
update account  set balance = balance - 2000 where AccountNo='1111';
update account  set balance = balance + 2000 where AccountNo='2222';
insert into journal values( '2020-10-05', 'transfer 2000 from 1111 to 2222', '1111', '2222', 2000);
commit
```

