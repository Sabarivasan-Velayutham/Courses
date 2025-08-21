

1. INTRODUCTION
---------------
- Hive is a data warehouse tool built on top of Hadoop for querying and analyzing large datasets. Built on Write-Once and Read-Many concept. It is designed for batch processing and works well with big data.
- Query language: HiveQL (similar to SQL).
- Schema-on-read: schema is applied while reading the data, unlike RDBMS where schema is enforced at write.
- Schema-on-Read Concept in Hive: Schema-on-Read means that the structure (schema) of the data is applied only when the data is read or queried, not when it is stored. This is different from traditional databases (like RDBMS), which use Schema-on-Write, where the schema is enforced when the data is written to the database. In Hive, data is stored in its raw format (e.g., CSV, JSON, or Parquet) in Hadoop's HDFS. When you query the data using HiveQL, the schema is applied dynamically at query time.

Advantages of Schema-on-Read:
- Flexibility: You can store raw data in any format and define multiple schemas for different use cases.
- No Preprocessing: No need to enforce a schema when loading data, saving time.

Key Differences from RDBMS:
- Hive is not for OLTP (transactional systems), it is for OLAP (analytics).
- Schema-on-read vs Schema-on-write.
- Latency: Hive queries are slower since they run as MapReduce/Tez/Spark jobs.

2. HIVE DATA TYPES
-------------------
Primitive Types:
- Numeric: TINYINT, SMALLINT, INT, BIGINT, FLOAT, DOUBLE, DECIMAL
- String: STRING, VARCHAR, CHAR
- Date/Time: DATE, TIMESTAMP
- Boolean: BOOLEAN

Complex Types:
- ARRAY<datatype>
- MAP<keytype, valuetype>
- STRUCT<col1:type1, col2:type2>
  Example: col_A STRUCT<a:STRING, b:INT, c:BOOLEAN>
- UNIONTYPE<type1, type2,...>
  Example: col_B UNIONTYPE<STRING, INT, BOOLEAN>
- UNION - a data type that can have one of many data types. It is useful when you want to store something that could be one of several types. -> EG: col_B UNIONTYPE<STRING, INT, BOOLEAN>

Hive Query Commands:
- show tables
- describe <table>
- describe extended <table>
- to clear the hive terminal screen: !clear
- to connect to hive cli using jdbc connection: beeline 

To view the list of tables in HDFS CLI: hadoop fs -ls /user/hive/warehouse

HDFS CLI COMMANDS:
To execute any HDFS commands, it should start with either:
- hadoop fs
- hdfs dfs

The above commands help to give list of all possible HDFS commands

hadoop fs -copyFromLocal <localFilePath> <hdfsPath>
Use copyFromLocal to copy a file from your local file system to a HDFS location
Can also use "put" instead of using copyFromLocal

hadoop fs -copyToLocal <hdfsPath> <localFilePath>
Use copyToLocal to copy a file from your HDFS to local
Can also use "get" instead of using copyToLocal

hadoop fs -help

To run hive commands from your local command prompt itself:
- to run a hive query, commands starts with hive -e '<hive_query>'
- to run a sql script which has a list of queries to be executed, hive -f hivescript.sql

3. HIVE TABLES
--------------
Types:
- Managed Table: Hive manages data & metadata, dropping table deletes data too.
- External Table: Hive manages only metadata, dropping table does not delete data.

Managed tables: Hive data is managed by hive and is stored in its warehouse directory.
External tables: Hive data exists outside the warehouse directory. "EXTERNAL" keyword is used to create external tables.

EXTERNAL TABLES ARE USED WHEN YOU WANT THE UNDERLYING DATA TO BE SHARED BETWEEN MULTIPLE PROGRAMS
EX: THE SAME FILES CAN BE USED BY HIVE, PIG, HBASE, ETC

Create Syntax:
CREATE TABLE employees (
  id INT,
  name STRING,
  salary DOUBLE
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

External Table:
CREATE EXTERNAL TABLE employees_ext (
  id INT,
  name STRING,
  salary DOUBLE
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hive/employees';

Partitioned Table:
CREATE TABLE sales (
  id INT,
  amount DOUBLE
)
PARTITIONED BY (year INT, month INT);

Bucketed Table:
CREATE TABLE students (
  id INT,
  name STRING
)
CLUSTERED BY (id) INTO 4 BUCKETS;

When creating a table, Hive throws error if it exists. Use:
CREATE TABLE IF NOT EXISTS sales_data (...);

Duplicate schema from another table:
CREATE TABLE sales_data_dup LIKE sales_data;

Temporary Tables:
- Temporary tables exist only within session and deleted after it ends.
- If permanent and temporary tables share the same name, the temporary one takes precedence.

CREATE TEMPORARY TABLE temp_employees (
  id INT,
  name STRING,
  salary DOUBLE
);

IF THERE ARE TWO TABLES WITH THE SAME NAME - ONE PERMANENT, ONE TEMPORARY,
THE USER CAN'T ACCESS THE PERMANENT TABLE WITHOUT DROPPING THE TEMPORARY TABLES
ALL THE REFERENCES MAPPED TO THAT TABLE NAME WILL RESOLVE TO THE TEMPORARY TABLE

1. Loading Data:
Hive's primary way to load data is from local files or HDFS.

From Local File System:
LOAD DATA LOCAL INPATH '/home/user/employees.csv' INTO TABLE employees;

From HDFS:
LOAD DATA INPATH '/user/hdfs/path/new_employees.csv' INTO TABLE employees;

Insert Overwrite / Into:

-- Overwrite existing data in the table (or partition)
INSERT OVERWRITE TABLE employees
SELECT 101,'David', 45, 60000;

-- Append data to the table (or partition)
INSERT INTO TABLE employees
SELECT 101,'David', 45, 60000;

-- Insert results of a query
INSERT INTO TABLE employees
SELECT * FROM another_table WHERE join_date > '2023-01-01';

Inserting data to a table: use insert statements or import data into the table from other files
LOAD DATA LOCAL INPATH '/Users/navdeepsingh/Desktop/campus_housing.txt' 
OVERWRITE INTO TABLE Campus_Housing;

Instead of overriding, if you need appending data to it, remove overwrite keyword
Use -> INTO TABLE Campus_Housing;

FORMAT CAN BE CHANGED TO MATCH YOUR FILE USING THE ALTER STATEMENT
ALTER TABLE Campus_Housing
SET SERDEPROPERTIES ('field.delim' = ',');

IMPORT DATA INTO A TABLE FROM OTHER TABLES:
INSERT OVERWRITE TABLE stores_product_data
SELECT StoreLocation, Product FROM Sales_Data;

Where Sales_Data table contains 4 cols and stores_product_data table contains only 2 cols.
We are replacing the data of stores_product_data(2cols) by the data from the Sales_Data(4cols) table.

Multitable inserts are also possible in hive:
FROM Sales_Data
INSERT OVERWRITE TABLE stores_product_data SELECT StoreLocation, Product
INSERT OVERWRITE TABLE product_data SELECT DISTINCT Product;

Where product_data is a single column table.
Sales_Data will be the source table.

ALTER TABLES:

ALTER TABLE revenue_per_day RENAME TO revenue_table;

ALTER TABLE revenue_table ADD COLUMNS (new_date DATE);

To delete a column from a table: TO DELETE A COLUMN DON'T PUT IT IN THE REPLACE COLUMNS LIST
ALTER TABLE revenue_table
REPLACE COLUMNS(OrderDate DATE, Revenue DECIMAL(10,2));

- DROP command to delete tables
- TRUNCATE command to delete only the table data and not the table

DELETE FROM WHERE conditions is not possible in hive. So you need to delete the whole data. If you want to delete partial data, what we can do is run a query to save the result in a file and delete the original table. Now create the same table again with the new data stored in the file which we had.

Hive metastore service stores the metadata for hive tables and partitions
Metastore is present at = hive.metastore.warehouse.dir
Tables and databases are stored as directories in HDFS.
Tables will be stored at /user/hive/warehouse directory.

4. HIVE FUNCTIONS
-----------------
Standard Functions: operate on row values
Example: CONCAT(fname, " ", lname), LENGTH(string), REVERSE(string), REGEXP_REPLACE()

Aggregate Functions: operate on multiple rows
Example: SUM(), COUNT(), AVG(), MIN(), MAX()

Table-Generating Functions (UDTFs): produce multiple rows
Example: EXPLODE(array)

Example:
SELECT EXPLODE(ARRAY(1, 2, 3));
Output:
    1
    2
    3

Example:
SELECT EXPLODE(subordinates)
FROM employeeDetails
WHERE empName = "Vitthal";

Output:
    Anuradha
    Arun
    Swetha

- EXPLODE() breaks up the array of subordinates into multiple rows.

Including employer Name with EXPLODE():
This will throw an error:
    SELECT empName, EXPLODE(subordinates)
    FROM employeeDetails
    WHERE empName = "Vitthal";

Reason:
    empName has 1 row, but EXPLODE(subordinates) has 3 rows.

Solution:
    Use LATERAL VIEW.

Example:
SELECT empName, subordinate
FROM employeeDetails
LATERAL VIEW EXPLODE(subordinates) exp AS subordinate
WHERE empName = "Vitthal";

Output:
    Vitthal   Anuradha
    Vitthal   Arun
    Vitthal   Swetha

- LATERAL VIEW joins the exploded table back to the original table.
- By default, it is an INNER JOIN.
  If an employee has NULL in the subordinates column, they will be excluded.

CASE..WHEN:
Example:
SELECT empname,
       CASE
           WHEN tenure < 2 THEN 0
           WHEN tenure >= 2 AND tenure <= 3 THEN 2
           WHEN tenure > 3 THEN 3
       END AS extra_vacation_days
FROM employeeTenures;

SIZE():
- Returns the number of elements in arrays and maps.
- Does not work with structs and unions.

Examples:
SELECT SIZE(ARRAY(1, 2, 3));
SELECT SIZE(MAP("name", "Swetha", "age", 30));

CAST():
- Converts from one data type to another.
- Hive enforces type safety — cannot insert a value of one type into a column of a different type.

Example:
SELECT CAST("25" AS BIGINT);
- If a value cannot be converted, NULL is returned.

Example:
SELECT CAST("abc" AS BIGINT);
Output:
    NULL

DESCRIBE FUNCTION:
- To get details of a function:
    DESCRIBE FUNCTION concat;
- To get description along with example:
    DESCRIBE FUNCTION EXTENDED concat;

5. HIVE WINDOWING/ANALYTIC FUNCTIONS
------------------------------------
- Allow operations across a set of rows related to the current row.
- Syntax: function() OVER (PARTITION BY ... ORDER BY ... ROWS/RANGE ...)

Examples:
ROW_NUMBER(): unique row number within partition
RANK(): ranking with gaps
DENSE_RANK(): ranking without gaps
NTILE(n): divides rows into n buckets

Example:
SELECT name, revenue,
ROW_NUMBER() OVER (ORDER BY revenue DESC) AS row_num,
RANK() OVER (ORDER BY revenue DESC) AS rnk,
DENSE_RANK() OVER (ORDER BY revenue DESC) AS dense_rnk
FROM company;

Difference between RANK() and DENSE_RANK():
- RANK() leaves gaps when values tie.
- DENSE_RANK() does not leave gaps.

6. HIVE JOINS
-------------
Types:

Types:
- INNER JOIN → matching rows only.
- LEFT OUTER JOIN → all left + matches.
- RIGHT OUTER JOIN → all right + matches.
- FULL OUTER JOIN → all rows from both sides.
- LEFT SEMI JOIN → checks existence (like IN/EXISTS). Returns only left table columns.
- LEFT ANTI JOIN → returns rows from left that do NOT have matches in right table.
- CROSS JOIN → Cartesian product.

INNER JOIN: returns matching rows only.
SELECT a.id, b.value
FROM table1 a
JOIN table2 b
ON a.id = b.id;

LEFT OUTER JOIN: all rows from left + matching rows from right.
SELECT a.id, b.value
FROM table1 a
LEFT JOIN table2 b
ON a.id = b.id;

RIGHT OUTER JOIN: all rows from right + matching rows from left.
SELECT a.id, b.value
FROM table1 a
RIGHT JOIN table2 b
ON a.id = b.id;

FULL OUTER JOIN: all rows from both, match where possible.
SELECT a.id, b.value
FROM table1 a
FULL OUTER JOIN table2 b
ON a.id = b.id;

LEFT SEMI JOIN: efficient form of IN/EXISTS subquery.
SELECT names.symbol
FROM names
LEFT SEMI JOIN revenue
ON names.symbol = revenue.symbol;

Notes:
- In left semi join, you cannot use right table columns in SELECT.
- More efficient than IN/EXISTS because it stops after finding the first match.
- Returns rows from left table that have matching rows in right table.

LEFT ANTI JOIN: opposite of LEFT SEMI JOIN
Returns rows from left table that do NOT have matching rows in right table.
SELECT names.symbol
FROM names
LEFT ANTI JOIN revenue
ON names.symbol = revenue.symbol;

Example: Find customers who have never made a purchase
SELECT c.customer_id, c.customer_name
FROM customers c
LEFT ANTI JOIN orders o
ON c.customer_id = o.customer_id;

CROSS JOIN: Cartesian product of two tables
Returns every combination of rows from both tables.
SELECT a.id, b.value
FROM table1 a
CROSS JOIN table2 b;

Example: If table1 has 3 rows and table2 has 4 rows, result will have 12 rows.
Use CROSS JOIN when you need all possible combinations.

7. JOIN OPTIMIZATIONS
---------------------
- Place the largest table last in join query, so smaller tables are loaded into memory and the last (biggest) table uses reducer.
- WHERE clause is applied after join, not before.

Map Side Joins:
- Joins that avoid reducer (faster).
- If all but one table are small, small ones are loaded into memory (distributed cache), large one split across mappers.
- Works differently depending on join type:
- Inner Join: small table either side.
- Left Join: large table left, small right.
- Right Join: large table right, small left.
- Full Outer Join: not supported.

Force Map Side Join:
SELECT /*+ MAPJOIN(small_table) */
a.id, b.value
FROM large_table a
JOIN small_table b
ON a.id = b.id;

8. WHEN TO USE WHICH JOIN
--------------------------
- Use INNER JOIN: when you want only matching rows.
- Use LEFT OUTER JOIN: when you want all from left + matches.
- Use RIGHT OUTER JOIN: when you want all from right + matches.
- Use FULL OUTER JOIN: when you need complete data from both.
- Use LEFT SEMI JOIN: when checking existence (IN/EXISTS replacement).
- Use LEFT ANTI JOIN: when you want rows that don't match (NOT IN/NOT EXISTS replacement).
- Use CROSS JOIN: when you need all possible combinations of rows.
- Use MAP JOIN: when one table is small enough to fit in memory.

9. HIVE OPTIMIZATION TECHNIQUES
-------------------------------

1. File Formats:
ORC (Optimized Row Columnar): Best for Hive. Stores data in columns instead of rows.
Example: CREATE TABLE sales_orc (id INT, amount DOUBLE) STORED AS ORC;
When to use: For analytical queries, reporting, and when you need fast query performance.
Why it's better: Compresses data well, skips reading unnecessary columns, supports ACID transactions.

Parquet: Another columnar format, good for cross-platform use.
Example: CREATE TABLE sales_parquet (id INT, amount DOUBLE) STORED AS PARQUET;
When to use: When sharing data between Hive, Spark, and other tools.

TextFile: Default format, stores data as plain text.
Example: CREATE TABLE sales_text (id INT, amount DOUBLE) STORED AS TEXTFILE;
When to use: For initial data loading or when data needs to be human-readable.

2. Compression:
Reduces storage space and network transfer time.
Example: SET hive.exec.compress.output=true;
         SET mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;

When to use: Always use compression for production tables.
Simple explanation: Like zipping files - takes less space and transfers faster.

3. Partitioning:
Divides table into smaller parts based on column values.
Example: CREATE TABLE sales (id INT, amount DOUBLE) PARTITIONED BY (year INT, month INT);

When to use: When you frequently filter by date, region, or category.
Simple explanation: Like organizing files in folders by date - you only search in relevant folders.

4. Bucketing:
Divides data into fixed number of buckets based on hash of column values.
Example: CREATE TABLE users (id INT, name STRING) CLUSTERED BY (id) INTO 10 BUCKETS;

When to use: For efficient joins and sampling.
Simple explanation: Like sorting books into numbered shelves - makes finding specific books easier.

5. Execution Engines:
Tez: Faster than MapReduce, good for interactive queries.
Example: SET hive.execution.engine=tez;

Spark: Even faster, good for complex analytics.
Example: SET hive.execution.engine=spark;

When to use: Always use Tez or Spark instead of default MapReduce.
Simple explanation: Like upgrading from a bicycle to a car - gets you there much faster.

6. Vectorization:
Processes multiple rows at once instead of one by one.
Example: SET hive.vectorized.execution.enabled=true;

When to use: With ORC format for analytical queries.
Simple explanation: Like processing a batch of letters together instead of one by one.

7. Cost-Based Optimizer (CBO):
Hive analyzes table statistics to choose the best query plan.
Example: ANALYZE TABLE sales COMPUTE STATISTICS;
         ANALYZE TABLE sales COMPUTE STATISTICS FOR COLUMNS id, amount;

When to use: After loading data and before running complex queries.
Simple explanation: Like GPS finding the fastest route - analyzes traffic (data) to choose best path.

8. Join Optimizations:
Map Join: Loads small table in memory for faster joins.
Example: SELECT /*+ MAPJOIN(small_table) */ * FROM large_table l JOIN small_table s ON l.id = s.id;

Bucket Map Join: When both tables are bucketed on join keys.
Example: SET hive.optimize.bucketmapjoin=true;

When to use: When one table is much smaller than the other.
Simple explanation: Keep the small table in your pocket (memory) instead of going back and forth to check it.

9. Parallel Execution:
Runs multiple operations simultaneously.
Example: SET hive.exec.parallel=true;
         SET hive.exec.parallel.thread.number=8;

When to use: For complex queries with multiple independent operations.
Simple explanation: Like having multiple workers doing different parts of a job at the same time.

10. Predicate Pushdown:
Applies filters early in the query process.
Example: Automatically happens with WHERE clauses on partitioned columns.

When to use: Always use WHERE clauses to filter data early.
Simple explanation: Like filtering water at the source instead of after it travels through all pipes.

11. Strict Mode:
Prevents accidentally scanning entire large tables.
Example: SET hive.mapred.mode=strict;

When to use: In production to prevent costly mistakes.
Simple explanation: Like safety locks that prevent you from accidentally doing expensive operations.

10. HIVE ARCHITECTURE (OVERVIEW)
-------------------------------
Components:
* CLI / JDBC / ODBC / WebUI
* Driver (manages session, compiles query)
* Compiler (parses, generates execution plan)
* Metastore (stores schema, metadata)
* Execution Engine (executes plan as MR/Tez/Spark jobs)
* Hadoop HDFS (stores actual data)

Flow:
User submits query → Driver → Compiler → Execution Engine → Hadoop cluster → Results returned.