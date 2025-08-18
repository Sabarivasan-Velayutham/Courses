===============================
HIVE PARTITIONING NOTES
===============================

--------------------------------
1. What is Partitioning in Hive?
--------------------------------
- Hive organizes tables into PARTITIONS.
- Partition = Splitting table data into sub-directories based on column values.
- Each partition corresponds to one or more values of PARTITION COLUMNS.
- Data in a partition is stored in separate HDFS directories.
- Multiple files can exist inside each partition directory.

Example Directory:
  /user/hive/warehouse/sales_table/
      /product=Bananas/
      /product=Nutella/
      /product=Milk/
      /product=PeanutButter/

--------------------------------
2. Why Partition Tables?
--------------------------------
- Logical organization of data.
- Improved query performance (prunes unnecessary partitions).
- Efficient storage structure in HDFS.

Example:
Query → "Total revenue from selling Milk on Jan 17"
- Unpartitioned table → Full table scan.
- Partitioned by Product → Only "product=Milk" partition is scanned → Faster!

NOTE:
- Partitioning works best if it matches common query filters.
- Too many partitions = overhead for NameNode (managing many directories).
- Do NOT partition on high cardinality columns (e.g., CustomerID).

--------------------------------
3. How to Create Partitioned Tables
--------------------------------
Syntax:
CREATE TABLE table_name
(
   col1 datatype,
   col2 datatype,
   ...
)
PARTITIONED BY (partition_col datatype);

Example 1: Partition by one column
CREATE TABLE Sales_Data_Product_Partition
(
   StoreLocation VARCHAR(30),
   OrderDate DATE,
   Revenue DECIMAL(10,2)
)
PARTITIONED BY (product VARCHAR(30));

Example 2: Partition by two columns
CREATE TABLE Sales_Data_Date_Product_Partition
(
   StoreLocation VARCHAR(30),
   Revenue DECIMAL(10,2)
)
PARTITIONED BY (OrderDate DATE, product VARCHAR(30));

NOTE:
- Partition column(s) are not part of the main CREATE TABLE columns list.
- They are declared only in PARTITIONED BY clause.

--------------------------------
4. Directory Structure for Partitions
--------------------------------
If table is partitioned by OrderDate and Product:

/user/hive/warehouse/sales_data_date_product_partition/
/OrderDate=2016-01-16/product=Milk/
/OrderDate=2016-01-16/product=Nutella/
/OrderDate=2016-01-17/product=Bananas/
/OrderDate=2016-01-17/product=PeanutButter/
/...

Each directory contains the files for that partition (file-01, file-02, etc.).

--------------------------------
5. Querying Partitioned Tables
--------------------------------
- Treat partition columns like normal columns.
- Queries with partition filters are faster.

Example:
SELECT * FROM Sales_Data_Product_Partition
WHERE product = 'Milk';

Faster because only partition "product=Milk" is scanned.

--------------------------------
6. Static Partition Insertion
--------------------------------
Syntax:
INSERT INTO table_name PARTITION (partition_col='value') VALUES (...);

Example:
CREATE TABLE Sales_Data_Date_Partition
(
   StoreLocation VARCHAR(30),
   Product VARCHAR(30),
   Revenue DECIMAL(10,2)
)
PARTITIONED BY (OrderDate DATE);

INSERT INTO Sales_Data_Date_Partition
PARTITION (OrderDate='2016-01-16')
VALUES
('Bellandur','Nutella',7455.67),
('Bellandur','Peanut Butter',5316.89),
('Bellandur','Milk',2433.76),
('Koramangala','Bananas',9456.01);

- Here, OrderDate is fixed (static).
- We must specify partition value.

--------------------------------
7. Problem with Static Partitioning
--------------------------------
- If table has many partitions (e.g., daily sales for years), inserting data one partition at a time is time-consuming.

--------------------------------
8. Dynamic Partitioning
--------------------------------
Solution = Hive auto-creates partitions at load time.

Enable Dynamic Partitioning (per session):
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;

Modes:
- STRICT → At least one static partition required.
- NONSTRICT → All partitions can be dynamic.

--------------------------------
9. Dynamic Partition Insert
--------------------------------
Suppose we have:
Source Table (unpartitioned):
CREATE TABLE Sales_Data_Without_Partition
(
   StoreLocation VARCHAR(30),
   Product VARCHAR(30),
   OrderDate DATE,
   Revenue DECIMAL(10,2)
);

Destination Table (partitioned by Product, OrderDate):
CREATE TABLE Sales_Data_Date_Product_Partition
(
   StoreLocation VARCHAR(30),
   Revenue DECIMAL(10,2)
)
PARTITIONED BY (Product VARCHAR(30), OrderDate DATE);

Insert with Dynamic Partitioning:
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;

INSERT INTO Sales_Data_Date_Product_Partition
PARTITION(Product, OrderDate)
SELECT StoreLocation, Revenue, Product, OrderDate
FROM Sales_Data_Without_Partition;

IMPORTANT:
- Partition columns must be at the end of SELECT statement.
- Hive automatically creates new partitions.

--------------------------------
10. Checking Existing Partitions
--------------------------------
Syntax:
SHOW PARTITIONS table_name;

Example:
SHOW PARTITIONS Sales_Data_Product_Partition;

Check HDFS directly:
hadoop fs -ls /user/hive/warehouse/sales_data_product_partition

--------------------------------
11. Summary
--------------------------------
- Partitioning splits tables into subdirectories.
- Use for performance optimization.
- Common query filters → Best candidates for partition columns.
- Avoid high cardinality columns for partitioning.
- Static partition → You specify partition manually.
- Dynamic partition → Hive creates automatically.
