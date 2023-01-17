---
title: Database
has_children: true
nav_order: 2
permalink: docs/database
resource: true
desc: "Database interview questions and answers."
categories: [Database]
---

# Database
{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## SQL performance 


### SQL performance in productions

#### Indexing

Indexing can greatly improve the performance of SQL queries by allowing the database to quickly locate and retrieve specific rows of data. For example, creating an index on a frequently searched column can improve query performance.


```sql
CREATE INDEX idx_name ON table_name (column_name);

```

#### Normalization

Normalizing the database can reduce data redundancy and improve performance by reducing the amount of data that needs to be read and processed. For example, breaking a large table into smaller, more specific tables can improve performance.


```sql
CREATE TABLE orders (
                      order_id INT PRIMARY KEY,
                      customer_id INT,
                      order_date DATE
);

CREATE TABLE order_items (
                           order_id INT,
                           product_id INT,
                           quantity INT,
                           FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

```

#### Caching

Caching can improve performance by storing frequently accessed data in memory, allowing for faster retrieval. For example, using a caching solution like Redis or Memcached can improve performance by reducing the number of database queries required.

```sql

# Example of caching a query result in Redis
import redis
r = redis.Redis(host='localhost', port=6379, db=0)
r.set("query_result", query_result)


```

#### Optimizing Queries

Optimizing SQL queries can improve performance by reducing the amount of data that needs to be read and processed. For example, using the EXPLAIN keyword to analyze a query and identify which indexes are being used, or rewriting a query to use JOINs instead of subqueries can improve performance.


```sql
EXPLAIN SELECT * FROM orders
JOIN order_items ON orders.order_id = order_items.order_id;

```

#### Partitioning

Partitioning a table can improve performance by distributing large amounts of data across multiple storage devices, reducing the amount of I/O required to access the data. For example, partitioning a table based on a specific column can improve performance by reducing the number of rows that need to be searched.

```sql
ALTER TABLE orders
PARTITION BY RANGE (order_date)
(PARTITION p_early VALUES LESS THAN ('2020-01-01'),
  PARTITION p_late VALUES LESS THAN ('2021-01-01'));

```


#### Proper use of transactions

Misuse of transactions can cause performance issues such as deadlocks, lock waits and blocking. Proper use of transactions such as using the correct isolation level, using as smaller transactions as possible and keeping transactions as short as possible can avoid these issues.



```sql
BEGIN;
UPDATE orders SET status = 'shipped' WHERE order_id = 1;
COMMIT;
```


#### Use of Stored Procedures

Stored procedures are precompiled, which makes them faster than executing the same SQL statement repeatedly. Stored procedures are also more secure because they can be executed with the necessary permissions and can be invoked with parameters, which reduces the risk of SQL injection.


```sql
CREATE PROCEDURE update_order_status (IN order_id INT, IN new_status VARCHAR(20))
BEGIN
UPDATE orders SET status = new_status WHERE order_id = order_id;
END;
```


#### Monitoring and Performance tuning

Regular monitoring of the database performance and identifying and fixing any bottlenecks can improve performance. Tools like SQL Profiler, Performance Monitor, and Query Store can help identify the issues and fine-tune the performance of the database.

These are general suggestions and the best approach may vary depending on the specific needs of your database. It's important to test and measure the performance before and after any changes to ensure that the performance has been improved.


### Increase SQL performance

- Every index increases the time takes to perform INSERTS, UPDATES, and DELETES, so the number of indexes should not be too much. Try to use maximum 4-5 indexes on one table, not more. If you have read-only table, then the number of indexes may be increased.

- Keep your indexes as narrow as possible. This reduces the size of the index and reduces the number of reads required to read the index.

- Try to create indexes on columns that have integer values rather than character values.

- If you create a composite (multi-column) index, the orders of the columns in the key are very important. Try to order the columns in the key as to enhance selectivity, with the most selective columns to the leftmost of the key.

- If you want to join several tables, try to create surrogate integer keys for this purpose and create indexes on their columns. Create surrogate integer primary key (identity for example) if your table will not have many insert operations.

- Clustered indexes are more preferable than nonclustered, if you need to select by a range of values or you need to sort results set with GROUP BY or ORDER BY. If your application will be performing the same query over and over on the same table, consider creating a covering index on the table.

- You can use the SQL Server Profiler Create Trace Wizard with "Identify Scans of Large Tables" trace to determine which tables in your database may need indexes. This trace will show which tables are being scanned by queries instead of using an index.


### Reasons of poor performance of query.

Following are the reasons for the poor performance of a query:

-  No indexes.
-  Excess recompilations of stored procedures.
-  Procedures and triggers without SET NOCOUNT ON.
-  Poorly written query with unnecessarily complicated joins.
-  Highly normalized database design.
-  Excess usage of cursors and temporary tables.
-  Queries with predicates that use comparison operators between different columns of the same table.
-  Queries with predicates that use operators, and any one of the following are true:
-  There are no statistics on the columns involved on either side of the operators.
-  The distribution of values in the statistics is not uniform, but the query seeks a highly selective value set. This situation can be especially true if the operator is anything other than the equality (=) operator.
-  The predicate uses the not equal to (!=) comparison operator or the NOT logical operator.
-  Queries that use any of the SQL Server built-in functions or a scalar-valued, user-defined function whose argument is not a constant value.
-  Queries that involve joining columns through arithmetic or string concatenation operators.
-  Queries that compare variables whose values are not known when the query is compiled and optimized.



---

## TRUNCATE vs DELETE, vs  DROP

|TRUNCATE       | DELETE	                                |DROP   |
|------------------|--------------------------|-------------------------------------------|
|  The TRUNCATE TABLE the command deletes the data inside a table, but not the table itself. TRUNCATE deletes all the rows of a table at once. It only logs once in the transaction log.| The DELETE query deletes all records from a table of a database without deleting the table schemas like columns, indexes, etc.  | The DROP TABLE statement is used to drop an existing table in a database. DROP TABLEquery removes the table definition and all data, indexes, triggers, constraints, and permissions for that table.      |
|  TRUNCATE TABLE table_name;| DELETE FROM table_name WHERE condition;|  DROP TABLE table_name;     |



### TRUNCATE
- TRUNCATE is a Data Definition Language(DDL) command.
- TRUNCATE is executed using a table lock and the whole table is locked to remove all records.
- TRUNCATE removes all rows from a table at once.
- WHERE clause cannot be used with TRUNCATE.
- It is faster performance-wise than DELETE as it only logs once in the transaction log.
- TRUNCATE cannot be used with indexed views.
- To use Truncate on a table, ALTER permission is required on the table.

### DELETE
- DELETE is a Data Manipulation Language(DML) command.
- DELETE is executed using a row lock mechanism, each row in the table is locked for deletion.
- WHERE clause can be used with DELETE query to filter out records and then delete them.
- DELETE maintains an entry in the transaction log for each row deleted. Hence it is slower than TRUNCATE.
- DELETE permission is required on the table to delete the records.
- The DELETE can be used with indexed views.


### DROP
- The DROP command removes a table from the database.
- All the tables’ rows, indexes, and privileges will also be removed.
- No Data Manipulation Language(DML) triggers will be fired.
- DROP is a Data Definition Language(DDL).
- DELETE operations can be rolled back (undone), while DROP and TRUNCATE operations cannot be rolled back.

---

## Joins

The SQL Joins clause is used to combine records from two or more tables in a database. A JOIN is a means for combining fields from two tables by using values common to each.

- [Pictorial Reference](https://commons.wikimedia.org/wiki/File:SQL_Joins.svg)
- **Inner join**: Rows common in both T1 and T2
- **Left outer join**: All rows in T1
- **Right outer join**: All rows in T2
- **Full join**: All rows in T1 and T2
- **Cross join**: All rows in T1 * all rows in T2

### Inner join

The INNER JOIN creates a new result table by combining column values of two tables (table1 and table2) based upon the join-predicate. The query compares each row of table1 with each row of table2 to find all pairs of rows which satisfy the join-predicate. When the join-predicate is satisfied, column values for each matched pair of rows of A and B are combined into a result row.

Syntax -
```sql
SELECT table1.column1, table2.column2...
FROM table1
INNER JOIN table2
ON table1.common_field = table2.common_field;
```

```sql
SQL> SELECT  ID, NAME, AMOUNT, DATE
FROM CUSTOMERS
INNER JOIN ORDERS
ON CUSTOMERS.ID = ORDERS.CUSTOMER_ID;
```



### Left outer join

The SQL LEFT JOIN returns all rows from the left table, even if there are no matches in the right table. This means that if the ON clause matches 0 (zero) records in the right table; the join will still return a row in the result, but with NULL in each column from the right table.

This means that a left join returns all the values from the left table, plus matched values from the right table or NULL in case of no matching join predicate.

Syntax-
```sql
SELECT table1.column1, table2.column2...
FROM table1
LEFT JOIN table2
ON table1.common_field = table2.common_field;
```

```sql
SQL> SELECT  ID, NAME, AMOUNT, DATE
FROM CUSTOMERS
LEFT JOIN ORDERS
ON CUSTOMERS.ID = ORDERS.CUSTOMER_ID;
```


### Right outer join

The SQL RIGHT JOIN returns all rows from the right table, even if there are no matches in the left table. This means that if the ON clause matches 0 (zero) records in the left table; the join will still return a row in the result, but with NULL in each column from the left table.

This means that a right join returns all the values from the right table, plus matched values from the left table or NULL in case of no matching join predicate.

Syntax-
```sql
SELECT table1.column1, table2.column2...
FROM table1
RIGHT JOIN table2
ON table1.common_field = table2.common_field;
```

```sql
SQL> SELECT  ID, NAME, AMOUNT, DATE
FROM CUSTOMERS
RIGHT JOIN ORDERS
ON CUSTOMERS.ID = ORDERS.CUSTOMER_ID;
```

### Full join

The SQL FULL JOIN combines the results of both left and right outer joins.

The joined table will contain all records from both the tables and fill in NULLs for missing matches on either side.

Syntax −
```sql
SELECT table1.column1, table2.column2...
FROM table1
FULL JOIN table2
ON table1.common_field = table2.common_field;
```

```sql
SQL> SELECT  ID, NAME, AMOUNT, DATE
FROM CUSTOMERS
FULL JOIN ORDERS
ON CUSTOMERS.ID = ORDERS.CUSTOMER_ID;
```


If your Database does not support FULL JOIN (MySQL does not support FULL JOIN), then you can use UNION ALL clause to combine these two JOINS as shown below.
```sql
SQL> SELECT  ID, NAME, AMOUNT, DATE
FROM CUSTOMERS
LEFT JOIN ORDERS
ON CUSTOMERS.ID = ORDERS.CUSTOMER_ID
UNION ALL
SELECT  ID, NAME, AMOUNT, DATE
FROM CUSTOMERS
RIGHT JOIN ORDERS
ON CUSTOMERS.ID = ORDERS.CUSTOMER_ID

```

### Cross join

The CARTESIAN JOIN or CROSS JOIN returns the Cartesian product of the sets of records from two or more joined tables. Thus, it equates to an inner join where the join-condition always evaluates to either True or where the join-condition is absent from the statement.

Syntax  −

```sql
SELECT table1.column1, table2.column2...
FROM  table1, table2 [, table3 ]
```

```sql
SQL> SELECT  ID, NAME, AMOUNT, DATE
FROM CUSTOMERS, ORDERS;
```

### Self Join

The SQL SELF JOIN is used to join a table to itself as if the table were two tables; temporarily renaming at least one table in the SQL statement.

Syntax−

```sql
SELECT a.column_name, b.column_name...
FROM table1 a, table1 b
WHERE a.common_field = b.common_field;
```

```sql
SQL> SELECT  a.ID, b.NAME, a.SALARY
FROM CUSTOMERS a, CUSTOMERS b
WHERE a.SALARY < b.SALARY;
```

---

## SQL vs NoSQL

### SQL

SQL stands for Structured Query Language. SQL is a standard language for storing, manipulating, and retrieving data in relational database systems.

---

### NoSQL

NoSQL or `non-SQL` is a non-relational database that does not require a fixed schema and is easy to scale.

NoSQL does not necessarily mean that a database does not support SQL. Instead, it means that the database is not an RDBMS.

While traditional RDBMS rely on SQL syntax to store and query data, on the other hand, NoSQL database systems use other technologies and programming languages to store structured, unstructured or semi-structured data.


### SQL vs NoSQL


|Comparing       | SQL	                                |NoSQL   |
|------------------|--------------------------|-------------------------------------------|
|Query Language 	    |  Structured query language (SQL)                 | No declarative query language                |
| Database type      |         Table          |     Key-value, document, wide-column, and graph            |
|  Schema     |    Predefined               |    Dynamic             |
| Data model      |   Relational                |  Non-relational               |
|  Popular database examples     | MySQL, PostgreSQL, Oracle, and MS-SQL                  |  MongoDB, Apache HBase, Amazon DynamoDB, Redis, Couchbase, Cassandra, and Elasticsearch               |
| Ability to scale      |  Vertical                 |   Horizontal              |
| ACID vs BASE      |    ACID               |     BASE            |
|  Advantages     |   Cross-platform support, secure and free                | Easy to use, high performance, and flexible tool                |
|  Disadvantages     |  Complex to maintain and inefficient if processing big data, complex relational database systems are difficult to export into other systems, not good for handling various data types                 | Data is less structured, NoSQL databases are not as reliable (no ACID support), NoSQL databases are newer and may offer less features than their SQL counterparts                |
| Use cases      |  ACID support, complex queries, no changes or growth                 | Real-time data, volumes of data with no structure, agile business, cloud computing                |



### Database Types


Database types depend on the way the data is stored.
`SQL` has a table-based database. Table database stores data into tables with fixed rows and columns.

`NoSQL` has 4 types of databases:
1. Key-value database – Stores every data element as an attribute name or key together with its value.
2. Document database – Stores data in JSON, BSON, or XML documents.
3. Wide-column database – Stores and groups data into columns instead of rows.
4. Graph database – Optimized to capture and search the connections between data elements.


### Schema


A database schema is a structure that defines how a database is constructed. It defines how the data is organized and how the relations among data are associated. There are two types of schemas:

- Predefined
- Dynamic

SQL needs a predefined schema for unstructured data. You need to predefine data structure in the form of tables before you start to use SQL to manipulate data.

However, a NoSQL database does not require a predefined schema. NoSQL uses a dynamic schema for unstructured data. A dynamic schema allows storing data before applying schema. Schema completely depends on how you want to store data.



### Data Model


The data model shows the logical structure of the database. It organizes elements of data and standardizes how they relate to each other. There are two types of data models:

- Relational
- Nonrelational

We can observe differences between these data models by looking at the multiple entities. Consider an order from a restaurant as an example and two entities: Order and Delivery Address.

SQL uses a `relational data model`. SQL relational model uses many-to-many relationship. In many-to-many relationship, a single Order row can relate to several Delivery Address rows. Similarly, each Delivery address row can relate to several Order rows.

NoSQL uses a `nonrelational data model` that does not use relationships. NoSQL databases denormalize data by duplicating Delivery address in each Order row that contains that delivery address. Therefore, data is stored multiple times. This enables easy storage and data retrieval and increases the speed of the query.



### Ability to Scale

Database scalability is the ability to hold increasing amounts of data without sacrificing performance. There are two types of scalability:

- Vertical
- Horizontal

SQL databases are vertically scalable. In vertical scaling, data resides on a single node, and the only way to scale up is by adding more hardware resources, such as CPU and RAM, to one existing machine. This makes vertical scaling more costly. An additional downside of vertical scaling is that it runs on one machine so if the server goes down, your application will go down too.

NoSQL databases are horizontally scalable. In horizontal scaling, each node contains only part of the data which allows you to add more machines to the existing group of distributed systems. This makes horizontal scaling cheaper and quicker.


### ACID vs BASE


SQL databases use the ACID consistency model. ACID stands for:

- Atomic – All operations in a transaction succeed or every operation is rolled back. Partial success is not allowed.
- Consistent – Each transaction moves the database from one valid state to another. Transaction can’t leave the database in an inconsistent state.
- Isolated – Transactions can’t interfere with each other.
- Durable – The results of applying a transaction are permanent, even in the presence of failures.

The main feature of the ACID model is consistency. When you complete a transaction, its data is consistent and stable.

NoSQL databases use the BASE consistency model. BASE stands for:

- Basically Available – All users can perform a query. The database spreads data across several systems so in case that a failure happens to a segment of data, the database will not experience a complete outage.
- Soft State – Database state can change over time.
- Eventual Consistency – If the system is functioning and we wait long enough, the database will eventually become consistent.

The advantage of the BASE consistency model is that transactions are committed faster. Databases that use the BASE model prefer availability over consistency of replicated data.


### Use Cases


Not every database fits every business needs. Let’s take a closer look at use cases for both types of databases.


### Reasons to use an SQL database:


- When you need ACID support – With ACID support you get data consistency and 100% data integrity.
- When you are working with complex queries and reports – SQL is a better fit for complex query environments when compared to NoSQL.
- When you don’t anticipate a lot of changes or growth – If your business is not growing exponentially, there is no
  reason to use a system designed to support an increase in data volume.


### Reasons to use a NoSQL database:


- When you need real-time data – NoSQL does not require schemas, so it makes the information process quicker.
- When you store volumes of data with no structure – NoSQL supports all data types.
- When you run an agile business – NoSQL does not require the preparation process, so it reduces downtime.
- When you want to make most out of cloud computing and storage – For a cloud solution to be scalable, the data must be
  easy to share across multiple servers.

---

## UNION vs UNION ALL

UNION removes duplicate records (where all columns in the results are the same), UNION ALL does not.

There is a performance hit when using UNION instead of UNION ALL, since the database server must do additional work to
remove the duplicate rows, but usually you do not want the duplicates (especially when developing reports).

To identify duplicates, records must be comparable types as well as compatible types. This will depend on the SQL
system. For example the system may truncate all long text fields to make short text fields for comparison (MS Jet), or
may refuse to compare binary fields (ORACLE)

UNION Example:

```sql
SELECT 'foo' AS bar
UNION
SELECT 'foo' AS bar
```

Result:
```log
+-----+
| bar |
+-----+
| foo |
| foo |
+-----+
2 rows in set (0.00 sec)
```

UNION ALL example:

```sql
SELECT 'foo' AS bar
UNION ALL
SELECT 'foo' AS bar
```

Result:
```log
+-----+
| bar |
+-----+
| foo |
| foo |
+-----+
2 rows in set (0.00 sec)
```



---

## For more information

- https://medium.com/javarevisited/know-the-differences-between-truncate-delete-and-drop-4ee70bb736fb
- [Cheat sheet](http://files.zeroturnaround.com/pdf/zt_sql_cheat_sheet.pdf)
- [Clustered vs Non-clustered indexes](https://docs.microsoft.com/en-us/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described)
- [DB Index Wiki](https://en.wikipedia.org/wiki/Database_index)
- [SQL vs NoSQL: What's the Difference?](https://phoenixnap.com/kb/sql-vs-nosql)
- [SQL vs NoSQL: when to use?](https://www.imaginarycloud.com/blog/sql-vs-nosql/)
