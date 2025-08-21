
Hive Bucketing - Notes
======================

0. Introduction to Bucketing
----------------------------
- Partitioning splits large datasets into directories based on column values.
- Bucketing splits data into a fixed number of files (buckets).
- A table can be partitioned, bucketed, or both.

1. Advantages of Bucketing
--------------------------
- Efficient sampling: quickly select fraction of data.
- Faster joins (map-side join).
- Sorting inside buckets improves query execution.
- Overall better performance.

2. How Tables are Bucketed
--------------------------
- Syntax:
  CREATE TABLE table_name (
    col1 datatype,
    col2 datatype
  )
  CLUSTERED BY (column_name) INTO N BUCKETS;

- Hive assigns rows using: hash(col) % N.
- Same column value always goes to same bucket.

Example: 3 buckets on StudentID:
  1 % 3 -> bucket 1
  2 % 3 -> bucket 2
  3 % 3 -> bucket 0

HDFS files:
  000000_0, 000001_0, 000002_0

3. Using Bucketed Tables
------------------------
3.1 Creating Bucketed Table:
  CREATE TABLE Reviews (
    MovieID INT,
    ReviewID BIGINT,
    Review VARCHAR(100),
    Rating TINYINT
  )
  CLUSTERED BY (ReviewID) INTO 4 BUCKETS;

3.2 Partition + Bucketing:
  CREATE TABLE Sales_Data_NEW (
    ProductID INT,
    OrderDate DATE,
    Revenue DECIMAL(10,2)
  )
  PARTITIONED BY (StoreID INT)
  CLUSTERED BY (ProductID) INTO 4 BUCKETS;

HDFS:
  /sales_data_new/storeid=1/000000_0 … 000003_0
  /sales_data_new/storeid=2/000000_0 … 000003_0

3.3 Sorted Buckets:
  CLUSTERED BY (ProductID) SORTED BY (OrderDate) INTO 4 BUCKETS;

3.4 Inserting Data:
  INSERT INTO Reviews VALUES (1,1,'Amazing',5);
  SET hive.enforce.bucketing = true;

  For partitioned buckets:
  SET hive.exec.dynamic.partition = true;
  SET hive.exec.dynamic.partition.mode = nonstrict;

  INSERT INTO Sales_Data_NEW PARTITION(StoreID)
  SELECT ProductID, OrderDate, Revenue, StoreID
  FROM Sales_Data;

4. Sampling in Hive
-------------------
4.1 From bucketed table:
  SELECT * FROM reviews TABLESAMPLE(BUCKET 4 OUT OF 4 ON ReviewID);

- bucket x out of y on col
  Example: bucket 1 out of 2 -> merges 1 & 3.

4.2 On different column:
  SELECT * FROM reviews TABLESAMPLE(BUCKET 1 OUT OF 4 ON MovieID);

4.3 Non-bucketed table:
  SELECT * FROM sales_data_new TABLESAMPLE(4 PERCENT);
  SELECT * FROM sales_data_new TABLESAMPLE(2 ROWS);

4.4 Sampling vs LIMIT
- LIMIT scans full table then reduces rows.
- TABLESAMPLE queries only a portion (faster with buckets).

Summary
-------
- Partitioning = directory-level split.
- Bucketing = file-level split, hash-based.
- Partition + Bucketing = maximum efficiency.
- Advantages: efficient sampling, faster joins, better performance.
- TABLESAMPLE: fetch subset directly from buckets.
