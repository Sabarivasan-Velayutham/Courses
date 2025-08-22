## 1. Introduction

*   Hive is a data warehouse tool built on top of Hadoop for querying and analyzing large datasets.
*   Built on the "Write-Once and Read-Many" concept.
*   Designed for batch processing and works well with big data.

*   **Query language:** HiveQL (similar to SQL).

*   **Schema-on-read:** Schema is applied while reading the data, unlike RDBMS where schema is enforced at write.
*   **Schema-on-Read Concept in Hive:** This means that the structure (schema) of the data is applied only when the data is read or queried, not when it is stored. This is different from traditional databases (like RDBMS), which use Schema-on-Write, where the schema is enforced when the data is written to the database. In Hive, data is stored in its raw format (e.g., CSV, JSON, or Parquet) in Hadoop's HDFS. When you query the data using HiveQL, the schema is applied dynamically at query time.
    *   **Advantages of Schema-on-Read:**
        *   **Flexibility:** You can store raw data in any format and define multiple schemas for different use cases.
        *   **No Preprocessing:** No need to enforce a schema when loading data, saving time.

*   **Key Differences from RDBMS:**
    *   Hive is not for OLTP (Online Transaction Processing - transactional systems); it is for OLAP (Online Analytical Processing - analytics).
    *   Schema-on-read vs. Schema-on-write.
    *   **Latency:** Hive queries are slower since they run as MapReduce/Tez/Spark jobs.

## 2. Hive Data Types

### Primitive Types:
*   **Numeric:** `TINYINT`, `SMALLINT`, `INT`, `BIGINT`, `FLOAT`, `DOUBLE`, `DECIMAL`
*   **String:** `STRING`, `VARCHAR`, `CHAR`
*   **Date/Time:** `DATE`, `TIMESTAMP`
*   **Boolean:** `BOOLEAN`

### Complex Types:
*   `ARRAY<datatype>`
*   `MAP<keytype, valuetype>`
*   `STRUCT<col1:type1, col2:type2>`
    *   **Example:** `col_A STRUCT<a:STRING, b:INT, c:BOOLEAN>`
*   `UNIONTYPE<type1, type2,...>`
    *   **Example:** `col_B UNIONTYPE<STRING, INT, BOOLEAN>`
    *   `UNION` - a data type that can hold one of several specified types. It is useful when you want to store something that could be one of various types.

### Hive Query Commands:
*   `show tables;`
*   `describe <table>;`
*   `describe extended <table>;`
*   To clear the Hive terminal screen: `!clear`
*   To connect to Hive CLI using JDBC connection: `beeline`

### HDFS CLI Commands:
*   To view the list of tables in HDFS CLI: `hadoop fs -ls /user/hive/warehouse`
*   To execute any HDFS commands, they should start with either:
    *   `hadoop fs`
    *   `hdfs dfs`
    *   (These commands help to list all possible HDFS commands when used with `-help`)
*   `hadoop fs -copyFromLocal <localFilePath> <hdfsPath>`: Copies a file from your local file system to an HDFS location. Can also use `put` instead of `copyFromLocal`.
*   `hadoop fs -copyToLocal <hdfsPath> <localFilePath>`: Copies a file from HDFS to your local file system. Can also use `get` instead of `copyToLocal`.
*   `hadoop fs -help`

### Running Hive commands from the local command prompt:
*   To run a Hive query: `hive -e '<hive_query>'`
*   To run an SQL script with multiple queries: `hive -f hivescript.sql`

## 3. Hive Tables

### Types:
*   **Managed Table:** Hive manages both data and metadata. Dropping the table deletes the data too. Hive data is stored in its warehouse directory.
*   **External Table:** Hive manages only metadata. Dropping the table does not delete the data. Hive data exists outside the warehouse directory. The `EXTERNAL` keyword is used to create external tables.
    *   **Use Case:** External tables are used when you want the underlying data to be shared between multiple programs (e.g., the same files can be used by Hive, Pig, HBase, etc.).

### Create Syntax:

```sql
CREATE TABLE employees (
  id INT,
  name STRING,
  salary DOUBLE
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```

### External Table:

```sql
CREATE EXTERNAL TABLE employees_ext (
  id INT,
  name STRING,
  salary DOUBLE
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hive/employees';
```

### Partitioned Table:

```sql
CREATE TABLE sales (
  id INT,
  amount DOUBLE
)
PARTITIONED BY (year INT, month INT);
```

### Bucketed Table:

```sql
CREATE TABLE students (
  id INT,
  name STRING
)
CLUSTERED BY (id) INTO 4 BUCKETS;
```

### Table Creation Utilities:
*   When creating a table, Hive throws an error if it exists. Use `CREATE TABLE IF NOT EXISTS sales_data (...)` to avoid this.
*   To duplicate the schema from another table: `CREATE TABLE sales_data_dup LIKE sales_data;`

### Temporary Tables:
*   Temporary tables exist only within the current session and are deleted after it ends.
*   If permanent and temporary tables share the same name, the temporary one takes precedence.
    *   **Note:** If there are two tables with the same name (one permanent, one temporary), the user can't access the permanent table without dropping the temporary table. All references mapped to that table name will resolve to the temporary table.

```sql
CREATE TEMPORARY TABLE temp_employees (
  id INT,
  name STRING,
  salary DOUBLE
);
```

### Loading Data:
Hive's primary way to load data is from local files or HDFS.

*   **From Local File System:**
    ```sql
    LOAD DATA LOCAL INPATH '/home/user/employees.csv' INTO TABLE employees;
    ```
*   **From HDFS:**
    ```sql
    LOAD DATA INPATH '/user/hdfs/path/new_employees.csv' INTO TABLE employees;
    ```

### Insert Overwrite / Into:
*   **Overwrite existing data in the table (or partition):**
    ```sql
    INSERT OVERWRITE TABLE employees
    SELECT 101,'David', 45, 60000;
    ```
*   **Append data to the table (or partition):**
    ```sql
    INSERT INTO TABLE employees
    SELECT 101,'David', 45, 60000;
    ```
*   **Insert results of a query:**
    ```sql
    INSERT INTO TABLE employees
    SELECT * FROM another_table WHERE join_date > '2023-01-01';
    ```

*   **Example of loading data with overwrite/append:**
    ```sql
    LOAD DATA LOCAL INPATH '/Users/navdeepsingh/Desktop/campus_housing.txt'
    OVERWRITE INTO TABLE Campus_Housing;
    ```
    *   To append data instead of overwriting, remove the `OVERWRITE` keyword:
        ```sql
        LOAD DATA LOCAL INPATH '/Users/navdeepsingh/Desktop/campus_housing.txt'
        INTO TABLE Campus_Housing;
        ```

*   **Changing table format properties:**
    ```sql
    ALTER TABLE Campus_Housing
    SET SERDEPROPERTIES ('field.delim' = ',');
    ```

*   **Import data into a table from other tables:**
    ```sql
    INSERT OVERWRITE TABLE stores_product_data
    SELECT StoreLocation, Product FROM Sales_Data;
    ```
    (Here, `stores_product_data` has 2 columns, and `Sales_Data` has 4. Data from `Sales_Data`'s `StoreLocation` and `Product` columns replaces data in `stores_product_data`.)

*   **Multitable Inserts (Hive):**
    ```sql
    FROM Sales_Data
    INSERT OVERWRITE TABLE stores_product_data SELECT StoreLocation, Product
    INSERT OVERWRITE TABLE product_data SELECT DISTINCT Product;
    ```
    (Where `product_data` is a single-column table. `Sales_Data` is the source table.)

### Altering Tables:
*   **Rename a table:**
    ```sql
    ALTER TABLE revenue_per_day RENAME TO revenue_table;
    ```
*   **Add columns:**
    ```sql
    ALTER TABLE revenue_table ADD COLUMNS (new_date DATE);
    ```
*   **Delete a column from a table:** (To delete a column, don't include it in the `REPLACE COLUMNS` list.)
    ```sql
    ALTER TABLE revenue_table
    REPLACE COLUMNS(OrderDate DATE, Revenue DECIMAL(10,2));
    ```

### Deleting Data and Tables:
*   `DROP` command: Deletes tables.
*   `TRUNCATE` command: Deletes only the table data, not the table structure itself.
*   **Note on `DELETE FROM WHERE`:** `DELETE FROM WHERE` conditions are not directly possible in Hive for partial row deletion (in older versions/text formats). To delete partial data, you can:
    1.  Run a query to save the desired data (that you want to keep) into a temporary file.
    2.  Delete the original table.
    3.  Re-create the same table.
    4.  Load the data from the temporary file into the newly created table.

### Hive Metastore:
*   Hive Metastore service stores the metadata for Hive tables and partitions.
*   Metastore location: `hive.metastore.warehouse.dir`
*   Tables and databases are stored as directories in HDFS.
*   Tables will be stored at `/user/hive/warehouse` directory.

## 4. Hive Functions

### Standard Functions:
Operate on row values.
*   **Example:** `CONCAT(fname, " ", lname)`, `LENGTH(string)`, `REVERSE(string)`, `REGEXP_REPLACE()`

### Aggregate Functions:
Operate on multiple rows.
*   **Example:** `SUM()`, `COUNT()`, `AVG()`, `MIN()`, `MAX()`

### Table-Generating Functions (UDTFs):
Produce multiple rows from a single input row.
*   **Example:** `EXPLODE(array)`

*   **`EXPLODE()` Example:**
    ```sql
    SELECT EXPLODE(ARRAY(1, 2, 3));
    ```
    **Output:**
            1
        2
        3
    *   **`EXPLODE()` with a column:**
    ```sql
    SELECT EXPLODE(subordinates)
    FROM employeeDetails
    WHERE empName = "Vitthal";
    ```
    **Output:**
            Anuradha
        Arun
        Swetha
        `EXPLODE()` breaks up the array of subordinates into multiple rows.

*   **Including other columns with `EXPLODE()` (requires `LATERAL VIEW`):**
    *   **Incorrect approach (will throw error):**
        ```sql
        SELECT empName, EXPLODE(subordinates)
        FROM employeeDetails
        WHERE empName = "Vitthal";
        ```
        **Reason:** `empName` has 1 row, but `EXPLODE(subordinates)` produces multiple rows, leading to a mismatch.
    *   **Solution: Use `LATERAL VIEW`**
        ```sql
        SELECT empName, subordinate
        FROM employeeDetails
        LATERAL VIEW EXPLODE(subordinates) exp AS subordinate
        WHERE empName = "Vitthal";
        ```
        **Output:**
                    Vitthal   Anuradha
            Vitthal   Arun
            Vitthal   Swetha
                *   `LATERAL VIEW` joins the exploded table back to the original table.
        *   By default, it performs an `INNER JOIN`. If an employee has `NULL` in the subordinates column, they will be excluded from the result.

### `CASE..WHEN` Statement:
```sql
SELECT empname,
       CASE
           WHEN tenure < 2 THEN 0
           WHEN tenure >= 2 AND tenure <= 3 THEN 2
           WHEN tenure > 3 THEN 3
       END AS extra_vacation_days
FROM employeeTenures;
```

### `SIZE()` Function:
*   Returns the number of elements in arrays and maps.
*   Does not work with structs and unions.
*   **Examples:**
    ```sql
    SELECT SIZE(ARRAY(1, 2, 3));
    SELECT SIZE(MAP("name", "Swetha", "age", 30));
    ```

### `CAST()` Function:
*   Converts from one data type to another.
*   Hive enforces type safety — you cannot insert a value of one type into a column of a different type without casting.
*   **Example:**
    ```sql
    SELECT CAST("25" AS BIGINT);
    ```
*   If a value cannot be converted, `NULL` is returned.
    *   **Example:**
        ```sql
        SELECT CAST("abc" AS BIGINT);
        ```
        **Output:**
                    NULL

### `DESCRIBE FUNCTION`:
*   To get details of a function:
    ```sql
    DESCRIBE FUNCTION concat;
    ```
*   To get description along with an example:
    ```sql
    DESCRIBE FUNCTION EXTENDED concat;
    ```

## 5. Hive Windowing/Analytic Functions

*   Allow operations across a set of rows related to the current row.
*   **Syntax:** `function() OVER (PARTITION BY ... ORDER BY ... ROWS/RANGE ...)`

### Examples:
*   `ROW_NUMBER()`: Assigns a unique, sequential number to each row within its partition, starting from 1.
*   `RANK()`: Assigns a rank to each row within its partition, with gaps for ties.
*   `DENSE_RANK()`: Assigns a rank to each row within its partition, without gaps for ties.
*   `NTILE(n)`: Divides the rows in a partition into `n` groups (buckets) and assigns a bucket number to each row.

```sql
SELECT name, revenue,
ROW_NUMBER() OVER (ORDER BY revenue DESC) AS row_num,
RANK() OVER (ORDER BY revenue DESC) AS rnk,
DENSE_RANK() OVER (ORDER BY revenue DESC) AS dense_rnk
FROM company;
```

### Difference between `RANK()` and `DENSE_RANK()`:
*   `RANK()` leaves gaps when values tie (e.g., 1, 1, 3).
*   `DENSE_RANK()` does not leave gaps (e.g., 1, 1, 2).

## 6. Hive Joins

### Types:
*   **INNER JOIN:** Returns only matching rows from both tables.
*   **LEFT OUTER JOIN:** Returns all rows from the left table, and the matching rows from the right table. If no match, `NULL`s for right table columns.
*   **RIGHT OUTER JOIN:** Returns all rows from the right table, and the matching rows from the left table. If no match, `NULL`s for left table columns.
*   **FULL OUTER JOIN:** Returns all rows when there is a match in either the left or right table. Unmatched rows will have `NULL`s for the missing side.
*   **LEFT SEMI JOIN:** Returns rows from the left table that have matching rows in the right table. It's efficient for checking existence (similar to `IN`/`EXISTS` subqueries).
*   **LEFT ANTI JOIN:** Returns rows from the left table that do *not* have matching rows in the right table.
*   **CROSS JOIN:** Returns the Cartesian product of two tables (every combination of rows).

### Examples:

*   **INNER JOIN:**
    ```sql
    SELECT a.id, b.value
    FROM table1 a
    JOIN table2 b
    ON a.id = b.id;
    ```
    Returns matching rows only.

*   **LEFT OUTER JOIN:**
    ```sql
    SELECT a.id, b.value
    FROM table1 a
    LEFT JOIN table2 b
    ON a.id = b.id;
    ```
    Returns all rows from left + matching rows from right.

*   **RIGHT OUTER JOIN:**
    ```sql
    SELECT a.id, b.value
    FROM table1 a
    RIGHT JOIN table2 b
    ON a.id = b.id;
    ```
    Returns all rows from right + matching rows from left.

*   **FULL OUTER JOIN:**
    ```sql
    SELECT a.id, b.value
    FROM table1 a
    FULL OUTER JOIN table2 b
    ON a.id = b.id;
    ```
    Returns all rows from both, matching where possible.

*   **LEFT SEMI JOIN:**
    ```sql
    SELECT names.symbol
    FROM names
    LEFT SEMI JOIN revenue
    ON names.symbol = revenue.symbol;
    ```
    **Notes:**
    *   In a `LEFT SEMI JOIN`, you cannot use columns from the right table in the `SELECT` clause.
    *   More efficient than `IN`/`EXISTS` because it stops after finding the first match in the right table.
    *   Returns rows from the left table that have matching rows in the right table.

*   **LEFT ANTI JOIN:** (opposite of `LEFT SEMI JOIN`)
    ```sql
    SELECT names.symbol
    FROM names
    LEFT ANTI JOIN revenue
    ON names.symbol = revenue.symbol;
    ```
    Returns rows from the left table that do *not* have matching rows in the right table.

    **Example: Find customers who have never made a purchase**
    ```sql
    SELECT c.customer_id, c.customer_name
    FROM customers c
    LEFT ANTI JOIN orders o
    ON c.customer_id = o.customer_id;
    ```

*   **CROSS JOIN:** (Cartesian product of two tables)
    ```sql
    SELECT a.id, b.value
    FROM table1 a
    CROSS JOIN table2 b;
    ```
    Returns every combination of rows from both tables.
    **Example:** If `table1` has 3 rows and `table2` has 4 rows, the result will have 12 rows.
    Use `CROSS JOIN` when you need all possible combinations.

## 7. Join Optimizations

*   Place the largest table last in a join query. This allows smaller tables to be loaded into memory, and the last (biggest) table can utilize the reducer phase more efficiently.
*   `WHERE` clause is applied after the join, not before, for performance considerations in Hive's execution model.

### Map Side Joins:
*   Joins that avoid the reducer phase (making them faster).
*   If all but one table are small enough to fit into memory, the small tables are loaded into the distributed cache, and the large table is split across mappers.
*   **Behavior differs by join type:**
    *   **Inner Join:** Small table can be on either side.
    *   **Left Join:** Large table on the left, small table on the right.
    *   **Right Join:** Large table on the right, small table on the left.
    *   **Full Outer Join:** Not supported for Map Side Join.

*   **Force Map Side Join:**
    ```sql
    SELECT /*+ MAPJOIN(small_table) */
    a.id, b.value
    FROM large_table a
    JOIN small_table b
    ON a.id = b.id;
    ```

## 8. When to Use Which Join

*   **INNER JOIN:** When you want only matching rows from both tables.
*   **LEFT OUTER JOIN:** When you want all rows from the left table and matching rows from the right.
*   **RIGHT OUTER JOIN:** When you want all rows from the right table and matching rows from the left.
*   **FULL OUTER JOIN:** When you need complete data from both tables, including non-matching rows.
*   **LEFT SEMI JOIN:** When checking for the existence of rows (as an efficient replacement for `IN`/`EXISTS`).
*   **LEFT ANTI JOIN:** When you want rows from the left table that do not have matches in the right table (as a replacement for `NOT IN`/`NOT EXISTS`).
*   **CROSS JOIN:** When you need all possible combinations of rows from two tables.
*   **MAP JOIN:** When one table is small enough to fit in memory for faster join execution.

## 9. Hive Optimization Techniques

### 1. File Formats:
*   **ORC (Optimized Row Columnar):** Best for Hive. Stores data in columns instead of rows.
    *   **Example:** `CREATE TABLE sales_orc (id INT, amount DOUBLE) STORED AS ORC;`
    *   **When to use:** For analytical queries, reporting, and when you need fast query performance.
    *   **Why it's better:** Compresses data well, skips reading unnecessary columns, supports ACID transactions.
*   **Parquet:** Another columnar format, good for cross-platform use.
    *   **Example:** `CREATE TABLE sales_parquet (id INT, amount DOUBLE) STORED AS PARQUET;`
    *   **When to use:** When sharing data between Hive, Spark, and other tools.
*   **TextFile:** Default format, stores data as plain text.
    *   **Example:** `CREATE TABLE sales_text (id INT, amount DOUBLE) STORED AS TEXTFILE;`
    *   **When to use:** For initial data loading or when data needs to be human-readable.

### 2. Compression:
Reduces storage space and network transfer time.
*   **Example:**
    ```sql
    SET hive.exec.compress.output=true;
    SET mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
    ```
*   **When to use:** Always use compression for production tables.
*   **Simple explanation:** Like zipping files - takes less space and transfers faster.

### 3. Partitioning:
Divides a table into smaller parts based on column values.
*   **Example:** `CREATE TABLE sales (id INT, amount DOUBLE) PARTITIONED BY (year INT, month INT);`
*   **When to use:** When you frequently filter by columns like date, region, or category.
*   **Simple explanation:** Like organizing files in folders by date - you only search in relevant folders.

### 4. Bucketing:
Divides data into a fixed number of buckets based on the hash of column values.
*   **Example:** `CREATE TABLE users (id INT, name STRING) CLUSTERED BY (id) INTO 4 BUCKETS;`
*   **When to use:** For efficient joins (especially bucketed joins) and sampling.
*   **Simple explanation:** Like sorting books into numbered shelves - makes finding specific books easier.

### 5. Execution Engines:
*   **Tez:** Faster than MapReduce, good for interactive queries.
    *   **Example:** `SET hive.execution.engine=tez;`
*   **Spark:** Even faster, good for complex analytics and iterative algorithms.
    *   **Example:** `SET hive.execution.engine=spark;`
*   **When to use:** Always use Tez or Spark instead of default MapReduce for better performance.
*   **Simple explanation:** Like upgrading from a bicycle to a car - gets you there much faster.

### 6. Vectorization:
Processes multiple rows at once instead of one by one.
*   **Example:** `SET hive.vectorized.execution.enabled=true;`
*   **When to use:** With ORC format for analytical queries.
*   **Simple explanation:** Like processing a batch of letters together instead of one by one.

### 7. Cost-Based Optimizer (CBO):
Hive analyzes table statistics to choose the best query plan.
*   **Example:**
    ```sql
    ANALYZE TABLE sales COMPUTE STATISTICS;
    ANALYZE TABLE sales COMPUTE STATISTICS FOR COLUMNS id, amount;
    ```
*   **When to use:** After loading data and before running complex queries, to help Hive optimize.
*   **Simple explanation:** Like GPS finding the fastest route - analyzes traffic (data) to choose the best path.

### 8. Join Optimizations:
*   **Map Join:** Loads small table in memory for faster joins.
    *   **Example:** `SELECT /*+ MAPJOIN(small_table) */ * FROM large_table l JOIN small_table s ON l.id = s.id;`
*   **Bucket Map Join:** When both tables are bucketed on join keys.
    *   **Example:** `SET hive.optimize.bucketmapjoin=true;`
*   **When to use:** When one table is much smaller than the other (for Map Join) or both are bucketed on join keys.
*   **Simple explanation:** Keep the small table in your pocket (memory) instead of going back and forth to check it.

### 9. Parallel Execution:
Runs multiple operations simultaneously.
*   **Example:**
    ```sql
    SET hive.exec.parallel=true;
    SET hive.exec.parallel.thread.number=8;
    ```
*   **When to use:** For complex queries with multiple independent operations.
*   **Simple explanation:** Like having multiple workers doing different parts of a job at the same time.

### 10. Predicate Pushdown:
Applies filters early in the query process to reduce the amount of data processed.
*   **Example:** Automatically happens with `WHERE` clauses on partitioned columns.
*   **When to use:** Always use `WHERE` clauses to filter data early in your queries.
*   **Simple explanation:** Like filtering water at the source instead of after it travels through all pipes.

### 11. Strict Mode:
Prevents accidentally scanning entire large tables without specifying partitions or limits.
*   **Example:** `SET hive.mapred.mode=strict;`
*   **When to use:** In production environments to prevent costly and long-running queries due to accidental full table scans.
*   **Simple explanation:** Like safety locks that prevent you from accidentally doing expensive operations.

## 10. Hive Architecture (Overview)

### Components:
*   **CLI / JDBC / ODBC / WebUI:** Interfaces for users and applications to interact with Hive.
*   **Driver:** Manages the life cycle of a HiveQL query. It receives queries, compiles them, and sends them for execution.
*   **Compiler:** Parses the HiveQL query, performs semantic analysis, and generates an optimized execution plan (logical and physical plan). 
*   **Metastore:** A central repository for Hive metadata (table schemas, column types, partition information, HDFS locations, etc.). This is typically stored in a relational database (e.g., MySQL, PostgreSQL).
*   **Execution Engine:** Executes the physical plan generated by the compiler. This can be MapReduce, Apache Tez, or Apache Spark, which process the data stored in HDFS.
*   **Hadoop HDFS (Hadoop Distributed File System):** The underlying distributed storage system where the actual data for Hive tables resides.

### Flow:
1.  **User submits query:** A user or application submits a HiveQL query through one of the interfaces (CLI, JDBC, etc.).

2.  **Driver:** The Driver receives the query.

3.  **Compiler:** The Driver sends the query to the Compiler. The Compiler parses the query, checks syntax and semantics, and interacts with the Metastore to retrieve necessary metadata (e.g., table schemas, partitions). It then generates a logical plan, which is transformed into a physical execution plan (e.g., a series of MapReduce jobs, Tez tasks, or Spark stages).

4.  **Execution Engine:** The Driver submits the physical execution plan to the Execution Engine (MapReduce, Tez, or Spark).

5.  **Hadoop Cluster Interaction:** The Execution Engine interacts with the Hadoop cluster's YARN for resource management and HDFS for data storage and retrieval. It executes the various stages of the plan.

6.  **Results returned:** Once the execution is complete, the results are collected and returned through the Driver back to the user/application.