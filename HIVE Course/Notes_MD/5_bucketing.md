# Hive Bucketing - Notes

## 0. Introduction to Bucketing

*   **Partitioning** splits large datasets into directories based on column values (e.g., `/country=US/`, `/country=CA/`).
*   **Bucketing** splits data within a partition (or an unpartitioned table) into a fixed number of files (buckets) based on the hash value of a specified column.
*   A Hive table can be partitioned, bucketed, or both.

## 1. Advantages of Bucketing

*   **Efficient sampling:** Allows for quick and accurate selection of a fraction of the data, as data within each bucket is guaranteed to be representative.
*   **Faster joins (Map-side join):** When two tables are bucketed on the same columns and have the same number of buckets, Hive can perform highly efficient bucketed map-side joins.
*   **Sorting inside buckets improves query execution:** If data within buckets is sorted, range queries and other operations become faster.
*   **Overall better performance:** By creating more manageable and structured data chunks.

## 2. How Tables are Bucketed

### Syntax:

```sql
CREATE TABLE table_name (
  col1 datatype,
  col2 datatype
)
CLUSTERED BY (column_name) INTO N BUCKETS;
```

*   Hive assigns rows to buckets using a hash function: `hash(bucket_column) % N`.
*   This ensures that the same column value always goes to the same bucket.

### Example: 3 buckets on `StudentID`

*   `1 % 3` -> bucket `1`
*   `2 % 3` -> bucket `2`
*   `3 % 3` -> bucket `0` (or `3 % 3 = 0`, mapping to the first bucket file)

### HDFS files:
If `N` is the number of buckets, Hive will create `N` files for the table (or per partition). These files are typically named `000000_0`, `000001_0`, `000002_0`, etc.

## 3. Using Bucketed Tables

### 3.1 Creating a Bucketed Table:

```sql
CREATE TABLE Reviews (
  MovieID INT,
  ReviewID BIGINT,
  Review VARCHAR(100),
  Rating TINYINT
)
CLUSTERED BY (ReviewID) INTO 4 BUCKETS;
```

### 3.2 Partitioning + Bucketing:

A table can be both partitioned and bucketed. Each partition will then contain its own set of buckets.

```sql
CREATE TABLE Sales_Data_NEW (
  ProductID INT,
  OrderDate DATE,
  Revenue DECIMAL(10,2)
)
PARTITIONED BY (StoreID INT)
CLUSTERED BY (ProductID) INTO 4 BUCKETS;
```

### HDFS Structure for Partitioned and Bucketed Tables:
/sales_data_new/storeid=1/000000_0
/sales_data_new/storeid=1/000001_0
/sales_data_new/storeid=1/000002_0
/sales_data_new/storeid=1/000003_0
/sales_data_new/storeid=2/000000_0
...
/sales_data_new/storeid=2/000003_0

### 3.3 Sorted Buckets:

You can also sort data within each bucket for further optimization.

```sql
CLUSTERED BY (ProductID) SORTED BY (OrderDate ASC) INTO 4 BUCKETS;
```
*(Note: `ASC` is default and optional)*

### 3.4 Inserting Data into Bucketed Tables:

When inserting data into a bucketed table, you need to set `hive.enforce.bucketing` to `true` for Hive to ensure data is written into the correct buckets.

```sql
-- For simple bucketed table (Reviews)
SET hive.enforce.bucketing = true;
INSERT INTO Reviews VALUES (1,1,'Amazing',5);
-- Or from another table using INSERT OVERWRITE/INTO SELECT

-- For partitioned and bucketed table (Sales_Data_NEW)
SET hive.enforce.bucketing = true;
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict; -- Or strict, depending on your needs

INSERT INTO Sales_Data_NEW PARTITION(StoreID)
SELECT ProductID, OrderDate, Revenue, StoreID
FROM Sales_Data;
```
*(Remember that for dynamic partitioning, the partition columns (`StoreID` in this case) must be the last columns in the `SELECT` statement.)*

## 4. Sampling in Hive

Sampling allows you to retrieve a subset of data from a table. This is particularly efficient with bucketed tables.

### 4.1 Sampling from a bucketed table:

```sql
SELECT * FROM reviews TABLESAMPLE(BUCKET 4 OUT OF 4 ON ReviewID);
```
*   **`TABLESAMPLE(BUCKET x OUT OF y ON col)`**: This syntax selects a specific `bucket x` out of `y` (total specified buckets, which can be a multiple or factor of the actual number of buckets).
    *   Example: `TABLESAMPLE(BUCKET 1 OUT OF 2 ON ReviewID)` would effectively combine buckets 1 and 3 (if there are 4 actual buckets), ensuring data is read only from those.

### 4.2 Sampling on a different column (less efficient than on bucketed column):

```sql
SELECT * FROM reviews TABLESAMPLE(BUCKET 1 OUT OF 4 ON MovieID);
```
*(While possible, this won't leverage the bucketing optimization unless `MovieID` is also the bucketing column.)*

### 4.3 Sampling from a non-bucketed table (approximate or row-based):

```sql
SELECT * FROM sales_data_new TABLESAMPLE(4 PERCENT); -- Approximate sampling based on percentage
SELECT * FROM sales_data_new TABLESAMPLE(2 ROWS);    -- Samples exact number of rows
```

### 4.4 Sampling vs. LIMIT

*   `LIMIT` scans the full table (or until enough rows are found) and then reduces the number of rows returned.
*   `TABLESAMPLE` queries only a portion of the data based on sampling criteria, making it much faster, especially when leveraging bucketing. 

## Summary

*   **Partitioning** = directory-level split of data.
*   **Bucketing** = file-level split within a table or partition, based on a hash function.
*   Combining **Partitioning + Bucketing** leads to maximum query efficiency and data organization.
*   **Advantages of Bucketing:** efficient sampling, faster joins (especially bucketed map joins), and overall better performance due to optimized data layout.
*   **`TABLESAMPLE`**: Used to fetch a subset of data directly from buckets, offering a performance advantage over `LIMIT` for sampling.  