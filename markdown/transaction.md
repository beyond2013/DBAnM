Credits: contents presented here are taken from the book **Database Systems An application oriented approach** 2nd Edition by **Michael Kifer, Arthur Bernstein, Philip M. Lewis**

# Transactions
- A transaction is a special kind of program.
- It executes within an application in which a database models the state of some real-world enterprise.
## Example
- If the enterprise is a bank, then the value of balance attribute in the row corresponding to your account is the net amount of money held for you in the account by the bank.

- The job of transaction is to maintain this model as the state of the enterprise changes. 
- Thus, whenever the state of the real world changes, a transaction is executed that updates the database to reflect that change. 

- Because of the requirement that a transaction processing application must maintain an accurate model of the state of the enterprise, the execution of transactions is constrained by certain properties that do not apply to ordinary programs.

- These special properties are frequently referred to using the acronym ACID: Atomic, Consistent, Isolated, Durable. 

## Consistency
Think of a database as playing both an active and a passive role in relation to the real world enterprise that it models.
- In its passive role, it maintains the correspondence between the database state and the enterprise data. For example, the Student Registration System must accurately maintain the identity and number of students who have registered for each course. 
- In its active role, it enforces certian rules of the enterprise. For example the number of students registered for a course must not exceed the maximum enrollment for that course. 
- A transaction that attempts to register a student for a course that is already full must not complete successfully. Consistency is the term that is used to describe these issues and it has two aspects. 
1. Internal consistency: It is often convenient to store the same information in different forms. For example, we might store the number of students registered for a course as well as a list whose entries name each student registered for the course. A database state in which the length of the list is not equal to the number of registrants is not allowed.
2. Enterprise rules: Enterprise rules restrict the possible state of the enterprise. When such a rule exists, the possible state of the database are similarly restricted. The rule relating the number of registrants and the maximum enrollment in a course is one example. 

The restrictions are referred to as integrity constraints or consistency constraints.

## Atomicity
The system must ensure that either the transaction runs to completion  or if it does not complete, it has no effect at all (as if it had never been started).

- If a transaction successfully completes and the system agrees to preserve its effects, we say that it has **committed**. If the transaction does not successfully complete we say that it has **aborted**, and the system must ensure that whatever changes the transaction made to the database are undone or **rolled back**.

### why transactions abort
A transaction might be aborted for several reasons. 
- One possibility is that the system crashes during its execution(before it commits). Other possibilities include the following:

1. Allowing the transaction to complete would cause a violation of an integrity constraint.
2. Allowing the transaction to complete would violate the isolation requirement, meaning that there is a possibility of a "bad interaction" with another executing transaction.
3. The transaction is involved in a deadlock, meaning two or more transactions are each waiting for the others to complete, and hence none would complete if the system did not abort one of them.

## Durability:
The system must ensure that once the transaction commits, its effects remain in the database even if the computer or medium on which the database is stored subsequently fails. 

Durability can be achieved by storing data redundantly on different backup devices. 

## Isolation:
The concurrent execution of a set of transactions has the same effect as that of some serial execution of that set.

## MySQL commands for writing transactions

MySQL provides following three commands for programming transactions:
1. start transaction
2. commit, and
3. rollback



