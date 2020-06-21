# Database Tuning

**Tuning is the process of modifying an application and adjusting the parameters of the underlying DBMS to improve performance**. Performance is measured in terms of the response time seen by a user (the time it takes to perform a task—for example, to execute an SQL statement) and throughput (the amount of work completed in a unit of time). It is important to realize that tuning does not affect the semantics of the system: the tuned and the original systems return the same information to the user and are left in the same final state when subjected to the same sequence of requests.

The first step in tuning a system is to determine where the bottlenecks are. If the system spends only 2% of its time executing a particular (hardware or software) module, then no matter how inefficient it is, revising or replacing it can improve performance by at most 2%.

**An application and DBMS, taken together, form an exceedingly complicated system, and many different aspects of it are subject to tuning**.

The SQL code and schema are at the highest level. Tuning at this level is concerned with such issues as how queries should be expressed and what indices should be created. 

This section discusses methods that you can use to encourage the DBMS to use the technique that performs the best for the particular application you are implementing.

The DBMS occupies the next level. Examples of performance issues at this level are the physical placement of data on secondary storage and how the DBMS manages its buffers. Decisions in this area are largely under the control of the database administrator, and hence the application programmer can influence them indirectly. 

The lowest tuning level is the hardware level. In order to perform well the system must be supported by CPUs and secondary storage devices, and adequate communication facilities. The specification of these resources is generally beyond the control of the application programmer, and we do not discuss these issues.

## Disk Caches

There is huge difference between the speed of the CPU and the time to transfer a page between the CPU and mass store. In recognition of this, the cost of a query plan is measured as the estimated number of page transfers it incurs, and the job of the query optimizer is to find the plan that minimizes this number. While that plan is generally a good one, its cost is often still significant, and other measures are necessary to make query processing efficient. One of the most significant of these is the cache. 

**A cache is a main memory buffer in the DBMS in which recently accessed database pages are stored**. When a transaction accesses a database item, the DBMS brings the database page(s) on disk that contain that item into the cache and then copies the value of the item from the cache into the application’s buffer. The page is generally retained in the cache under the assumption that there is a high probability that the application will either update the item or read another item in the same page at a later time. Or another application might concurrently reference an item in the page. In either case, a disk access will have been avoided since the page will be directly accessible in the cache. For example, an index page has a high probability of being accessed frequently.

Although it is natural to think of the database item as a page of a table or an index, it can also be the execution plan for an SQL statement or a stored procedure. In fact, some DBMSs maintain a separate procedure cache for this purpose. Although I/O cost is the major limitation on the performance of an application, the CPU cost of building an appropriate execution plan is also substantial. Hence, once an execution plan has been determined, it is saved since it might be possible to reuse it. Prior to preparing a new execution plan, existing plans are scanned to see if any are usable.

If a database item is to be updated, the database page containing the item must first be brought into the cache (if it is not already there), and it is the cache copy of the page that is modified (not the original copy in the database).

Eventually the cache becomes full, and any new page fetched from the database must overwrite a page, p, in the cache. If p has not been updated since arriving in the cache, its contents are identical to the corresponding page in the database, and hence it can simply be overwritten by the new page. However, if p has been updated since arriving in the cache, it must be written back to the database before the space it occupies in the cache can be freed. In order to distinguish between these two cases, the DBMS marks pages that have been updated as dirty and those that have not as clean. 

Decisions concerning which pages should be kept in the cache and which can be overwritten when a new page is to be fetched are made by a **page replacement algorithm** whose goal is to maximize the number of database accesses that can be satisfied by pages in the cache. A **least recently used (LRU) algorithm**, for example, selects the least recently used page in the cache as the one to be replaced. It concludes that since no application has accessed the page recently it is no longer useful. Hence it tends to keep actively used pages in the cache.

A more sophisticated algorithm takes into account the circumstances under which a page was brought into the cache. For example, if the page was brought in as part of a table scan (which is typical, for example, when sorts are performed), once the rows in the page have been accessed it is not likely that the application will reference the page again. In this case a **most recently used algorithm (MRU)** is preferable. Hence a page replacement policy might use a combination of an LRU and an MRU algorithm depending on what information is contained in the page (index or data) and in what context the page is referenced.

If a transaction’s access request can be satisfied from the cache, a **hit** is said to have taken place; if it cannot be satisfied, then a **miss** has occurred. To obtain a high throughput, many designers consider it mandatory to obtain a hit rate of over 90% (90% of the accesses can be satisfied from the cache). To achieve such a hit rate, the cache size must often be a significant percentage of the size of the database. Cache sizes in the megabyte range are normal. In some large applications, the cache size is measured in tens of gigabytes.

## Tuning the Cache

Now that you have an understanding of how the cache works, the question is, “What can the application programmer or the database administrator do to optimize the way the DBMS uses the cache to improve the performance of her application?”

+ DBMSs generally offer several mechanisms that can be invoked for this purpose. Some DBMSs allow pieces to be carved out of the (default) cache to be managed as separate caches. The programmer can then bind a particular item (e.g., a table or an index) to a specific cache and in so doing cause all pages of that item to be buffered in that cache. For example, if tables T1, and T2 are bound to different caches, a page of T; can never overwrite a page of T2. This approach might be useful if T2 was not used very often but fast response time was required of the application that referenced it.

+ Some DBMSs allow a particular cache to be subdivided into several distinct pools of buffers of different sizes. For example, while by default all buffers in a cache might have 2K bytes, it might be possible to reallocate the cache storage area so that several buffer pools are created with sizes 2K, 4K, 8K, etc. If a table is bound to such a cache, the query optimizer then has the option of choosing the I/O size that best suits a query plan that accesses that table. For example, the page size on secondary storage might be 2K bytes, and the DBMS might allocate disk space to a table in contiguous blocks of eight pages. It then follows that the time to retrieve an eight-page block is not much larger than the time to retrieve a single page since the seek time is the same in both cases. If the query plan call for a table scan, and the table is bound to a cache that has a 16K pool of buffers, the query optimizer can save time by retrieving eight pages with a single I/O operation.

Some query optimizers take this idea one step further by prefetching pages. Ordinarily, during the scan of a table or index, the next page is requested when the page fetched by the previous I/O operation has been scanned. The scan must then wait until the I/O operation for the next page completes. It is possible to improve on this in situations, such as scans, in which the optimizer can anticipate future requests. In such cases the optimizer can initiate the I/O operation for a page that has not yet been requested. Then, if the time to process a page is long enough, the next page will already be in the cache when it is requested.

By using both prefetching and a large I/O size, the time to do a table scan can be greatly reduced. This possible reduction has an impact on the query optimizer’s choice between an access path that involves an index and an access path that uses a table scan for a particular query. It also raises the question of whether the application programmer should create an index for that query.

+ Some DBMSs provide commands that allow the page replacement policy for a cache to be specified. This is particularly useful when multiple caches are used.  A policy appropriate to the items bound to the cache can then be chosen.

In addition to configuring a data cache to best suit the application, the programmer can design the application to make the best use possible of a procedure cache. For example, performance can be improved by not using explicit constants in SQL statements. The execution plan for the statement

```sql
SELECT P.Name
FROM PROFESSOR P
WHERE P.DeptId = 'EE' 
```
is essentially the same as the execution plan for the statement with the constant EE replaced by CS. But since the two statements are different, the DBMS might miss this fact when it scans the procedure cache looking for an execution plan, and as a result the DBMS might create a separate execution plan for each statement. It is possible to eliminate this overhead by instead using the statement

```sql
SELECT P.Name
FROM PROFESSOR P
WHERE P.DeptId = :deptid
```

where deptid is a host variable, and successively assigning EE and CS to that variable. Since the same statement is now executed twice, the execution plan created when the statement is first executed will be reused when it is executed for the second time.

## Tuning the Schema

The schema you design for your database is at the heart of the application. If the schema is well designed, it is possible to write SQL statements that perform efficiently. Your strategy in tuning at the application level is to first design a normalized database and then estimate the sizes of the tables, the distribution of column values, and the nature and frequency of the queries and updates that will be addressed to the database. Adjustments to the normalized schema to facilitate the most frequent operations follow from these estimates. Adding indices is the most important of these adjustments, and we discuss it first. Another technique is denormalization, which involves adding redundancy so that items of information that are generally associated with one another through frequently executed queries can be found in one place. Finally, we discuss partitioning, which is a rather specialized technique for dealing with very large tables.

1. Indices
Different query plans for a particular query might have wildly different costs and in many cases the differences are a function of the indices used in the plan. For better or worse, the choice of plan is made by the optimizer based on the indices available to it at the time the query is prepared. It is the role of the application programmer to "encourage" a good choice by making sure that appropriate indices have been created. In this section our goal is to expose the reasoning a programmer might use in deciding what indices to create.

Indices might seem like the ultimate database tuning device. However, free computational lunches are rare. Each index carries an associated storage overhead. More importantly, extra indices might significantly increase the processing time of statements that modify the database since every index must be updated whenever the table it references is changed. Thus, you should think twice before creating an index on a table where rows are frequently inserted or deleted. Similarly, you should think twice before creating an index with a search key involving a frequently updated column. Will the performance gain realized in processing queries be sufficient to compensate for the added cost of processing statements that modify the table? To illustrate some of the considerations involved in the tuning process, consider the following examples 

1. Consider the query

```sql
SELECT P.DeptId  
FROM PROFESSOR P   
WHERE P.Name = :name  
```  

Since the primary key of PROFESSOR is Id, we can expect that the DBMS has created a clustered index on that attribute. That index is no help for this query because we need a quick way to find all professors with a particular name. One possibility is to explicitly create an unclustered index on Name. Assuming that only a few professors have the same name, this index should speed things up. 

But suppose this is not the case; many professors have the same name, Then  a better solution is to make the index on Name clustered and the index on Id unclustered. As a result, rows with the same name will be grouped together and can be retrieved in a single (or a few) 1/O operations. The index could be a B+ tree or a hash (since the condition on Name involves equality).

The lesson here is that since a table can have only one clustered index, it is pointless to waste it on an attribute that cannot take advantage of clustering. DBMSs generally create a clustered index on the primary key, but you should not be intimidated by this. An unclustered index on the primary-key attribute is sufficient to guarantee the key’s uniqueness, and since at most one row can have a particular key value, clustering cannot be justified as a means of grouping rows with the same value of the attribute. So, if we are unlikely to want to order rows based on the primary key (as is the case with PROFESSOR), there is no reason to use a clustered index for this purpose.

Keep in mind that replacing one clustered index with another is a time consuming operation since it implies a complete reorganization of the storage structure. You certainly do not want to create a new clustered index each time you execute a query. You should analyze your application in advance, considering the kinds of queries you expect and their frequency, create the clustered index that will do the most good, and stick with it until performance considerations indicate that the system needs a tune-up.

2. Consider the query

```sql
SELECT T.Name, T.CrsCode
FROM TRANSCRIPT T 
WHERE 1T.Grade = :grade 
```

One is tempted to cluster the rows around Grade since we want to retrieve all rows with the same grade, but suppose that our first priority is to speed the response to a different query, a request for a class roster, and for that purpose we use a Clustered index on the primary key (CrsCode, Semester, StudlId). We could create an unclustered index on Grade, but using such an index might not be a good idea. In most cases the number of rows with a particular grade is a large fraction of the total number of rows (since the domain of Grade is small). In those cases we can expect that a large fraction of the table’s pages will be fetched, one by one, in random order, through the unclustered index. Unfortunately, the optimizer does not know what grade will be supplied at run time, and even if it did, it would not know which ones produced small result sets (we will correct this inadequacy shortly). Hence a table scan might be a better solution. 

A number of lessons can be drawn from this example. First, an unclustered index is appropriate if only a few rows of a table are to be retrieved, anda full table scan is appropriate if a large fraction of the rows are to be retrieved. Determining a reasonable break-even point is not easy. One vendor states that a table scan is appropriate if more than 20% of the rows of the table are to be accessed. A more cautious approach would be to simulate the workload if more than a few rows are to be accessed to determine if building an index is a good idea. Second, do not create an index on a column with a small domain if attribute values tend to be evenly distributed over the domain. The query optimizer is unlikely to choose such an index since it will recognize that the selectivity of the access path through this index is large for any value of search key. Finally, do not create indices indiscriminately: they are costly to maintain, and, with the techniques described in Section Disk Cache, table scans can be quite fast.

3. Suppose that the most frequent access path to TRANSCRIPT selects rows based on a condition involving both StudId and CrsCode. A less frequently used path selects rows based on a condition on Semester. If we build one index on (StudId,CrsCode) (actually an index on the primary key (StudId,CrsCode, Semester) would work fine) and another on Semester, which should be clustered? At first glance, it might seem that the index on (StudId,CrsCode) should be clustered because it is the main access path. However, even though (StudId, CrsCode) is not a candidate key, the number of TRANSCRIPT rows that agree on both of these attributes will be one in almost all cases—only when a student retakes a course can this number be larger than one. Therefore, clustering around (StudId, CrsCode) will not yield significant benefits. Also, it is not likely that range queries will be asked against this pair of attributes, so the overhead of a B+ tree index does not seem justified—a hash index is probably the best solution here. On the other hand, a clustered B+ tree index on Semester can greatly improve the efficiency of selections and joins on that attribute and makes an excellent choice for a secondary access path.

The lesson here is that clustering is useful to group together rows that might be output in a result set. These rows might be grouped because they all agree on the value of an attribute(s) or because they fall within a range of values of that attribute(s). In either case, when a choice has to be made as to what attribute to cluster on, you should make the choice based on the size of the result sets you expect in your application.

4. Assume the PROFESSOR table has the additional attribute Salary, and suppose we want to optimize the performance of the range query:

```sql
SELECT P.Name
FROM PROFESSOR P
WHERE P.Salary BETWEEN :lower AND :upper
```

The analysis here is similar to that of example 1 with the exception that we now want a clustered index on Salary and it must be a B\+ tree.

5. If two different queries would benefit from two different clustered indices on the same table, we have a problem since only one clustered index is possible. One solution is to make it possible for the optimizer to use an index-only strategy. 
For example, suppose that TEACHING already has a clustered B\+ tree index on Semester, but another important query would benefit from a clustered index on ProfId in order to quickly access the course codes associated with a given professor. We can sidestep the problem by creating an unclustered B+ tree index with search key (ProfId, CrsCode). Then all the information required by the query is contained in the index (and the index is often referred to as a covering index), and TEACHING does not have to be accessed at all! We simply search down the index using ProfId to the leaf level. Since the values of CrsCode at that level are clustered around ProfId, we can scan forward from that point at the leaf level of the index to get the required result set using only the index entries. This approach produces the same effect as that of a clustered index with search key (ProfId, CrsCode) on TEACHING (in fact, it is more efficient because the index is smaller and hence scanning a section of the leaf level requires fewer I/O operations than scanning a section of TEACHING).

Index-only query processing comes in two varieties. In this example we searched the index using ProfId to quickly locate the associated course codes. Suppose, however, another query required that we find the ProfIds of all professors who had taught a particular course. Unfortunately, although all the information we need is in the index, it cannot be searched because CrsCode is not the first attribute of the search key. But all is not lost. Another way to produce the desired result set is to scan the entire leaf level of the index. This is not as efficient as a search, but it might be better than having to scan the entire data file (the index is smaller!) or create and use an unclustered index on CrsCode.

6. The ability to nest queries is one of the most powerful features of SQL. Unfortunately, however, nested queries are very difficult to optimize. Consider the query

```sql
SELECT P.Name, C.CrsName
FROM PROFESSOR P, COURSE C
WHERE P.Department = 'CS' AND C.DeptId = 'MAT' AND
C.CrsCode IN 
       (SELECT T.CrsCode 
			  FROM TEACHING T 
				WHERE T.Semester = 'S2003' AND T.ProfId = P.Id)
```

that returns a set of rows in which the value of the first attribute is the name of a CS professor who has taught a course in the Math Department in the spring of 2003 and the value of the second is the name of one such course.

Typically, a query optimizer splits this query into two separate parts. The inner query is considered as an independently optimized unit. The outer query is also optimized independently (with the result set of the inner SELECT statement viewed as a database relation). In this case, the subquery is correlated, so it is crucial that it be executed efficiently since it will be executed many times. 

For example, a clustered index on TEACHING with search key (ProfId, Semester) would permit quick retrieval of all courses taught by a particular professor in a semester (and hopefully this is a small set). If possible (as in this example), the search key should involve all the attributes of the WHERE clause to avoid retrieving rows unnecessarily.

However, there is another point to note here. Because the two queries are optimized separately, certain alternatives might not be considered by the optimizer. For instance, the use of a clustered index on TEACHING with search key CrsCode would not be considered since the correlated nested subquery produces a set of course codes for each value of P.Id that is supplied and CrsCode is not even mentioned in the WHERE clause of that subquery. On the other hand, it is easy to see that the above query is equivalent to

```sql
SELECT C.CrsName, P.Name
FROM PROFESSOR P, TEACHING T, COURSE C
WHERE _T.Semester='S2003' AND P. Department='CS'
AND C.DeptId = 'MAT' AND P.Id = T.ProfId AND T.CrsCode=C.CrsCode 
```
and the use of that index would be considered in optimizing this query. One strategy that can take advantage of this index corresponds to the following expression:

$ \sigma_{Department} = _{'CS'} (PROFESSOR) \bowtie _{Id = ProfId} $ \\
$ \sigma_{Semester} = _{'S2003'}(TEACHING \bowtie _{CrsCode=CrsCode} \sigma_{DeptId}=_{'MAT'}(COURSE)) $

After computing $\sigma_{DeptId}=_{'MAT'}(COURSE)$, the index for CrsCode in TEACHING can be used to compute the join \bowtie _{CrsCode=CrsCode} in the index-nested loops algorithm.

It should be remarked that some query optimizers do, in fact, try to eliminate nested subqueries and take other steps to reduce the cost of processing them. However, it is still a good idea to avoid query nesting whenever possible. * This strategy may or may not be the best for this query depending on the sizes of the relations, selectivity of the attributes, and other parameters.
