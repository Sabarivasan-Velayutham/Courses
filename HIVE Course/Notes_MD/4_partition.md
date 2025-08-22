# Hive Partitioning - Notes

## 1. What is Partitioning in Hive?

*   Hive organizes tables into **PARTITIONS**.
*   **Partition** = Splitting table data into sub-directories based on column values.
*   Each partition corresponds to one or more values of **PARTITION COLUMNS**.
*   Data in a partition is stored in separate HDFS directories.
*   Multiple files can exist inside each partition directory.

### Example Directory Structure:

/user/hive/warehouse/sales_table/
    /product=Bananas/
    /product=Nutella/
    /product=Milk/
    /product=PeanutButter/

---

## 2. Why Partition Tables?

*   Logical organization of data.
*   Improved query performance (prunes unnecessary partitions).
*   Efficient storage structure in HDFS.

### Example:

**Query** → "Total revenue from selling Milk on Jan 17"

*   **Unpartitioned table** → Full table scan (reads all data).
*   **Partitioned by Product** → Only "product=Milk" partition is scanned → **Faster!**

### NOTE:

*   Partitioning works best if it matches common query filters.
*   Too many partitions can lead to overhead for NameNode (managing many directories and metadata).
*   Do NOT partition on high cardinality columns (e.g., `CustomerID`), as this can lead to an excessive number of small files and partitions, which is inefficient.

---

## 3. How to Create Partitioned Tables

### Syntax:

```sql
CREATE TABLE table_name
(
   col1 datatype,
   col2 datatype,
   ...
)
PARTITIONED BY (partition_col datatype);
```

### Example 1: Partition by one column

```sql
CREATE TABLE Sales_Data_Product_Partition
(
   StoreLocation VARCHAR(30),
   OrderDate DATE,
   Revenue DECIMAL(10,2)
)
PARTITIONED BY (product VARCHAR(30));
```

### Example 2: Partition by two columns

```sql
CREATE TABLE Sales_Data_Date_Product_Partition
(
   StoreLocation VARCHAR(30),
   Revenue DECIMAL(10,2)
)
PARTITIONED BY (OrderDate DATE, product VARCHAR(30));
```

### NOTE:

*   Partition column(s) are not part of the main `CREATE TABLE` columns list.
*   They are declared only in the `PARTITIONED BY` clause.

---

## 4. Directory Structure for Partitions

If a table is partitioned by `OrderDate` and `Product`:

/user/hive/warehouse/sales_data_date_product_partition/
/OrderDate=2016-01-16/product=Milk/
/OrderDate=2016-01-16/product=Nutella/
/OrderDate=2016-01-17/product=Bananas/
/OrderDate=2016-01-17/product=PeanutButter/
/...
Each directory contains the data files for that specific partition (e.g., `file-01`, `file-02`, etc.).

---

## 5. Querying Partitioned Tables

*   Treat partition columns like normal columns in your queries.
*   Queries with filters on partition columns will be faster due to partition pruning.

### Example:

```sql
SELECT * FROM Sales_Data_Product_Partition
WHERE product = 'Milk';
```
This query is faster because only the partition `/product=Milk/` is scanned, rather than the entire table.

---

## 6. Static Partition Insertion

In static partitioning, you manually specify the partition value(s) during data loading.

### Syntax:

```sql
INSERT INTO table_name PARTITION (partition_col='value') VALUES (...);
```

### Example:

```sql
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
```
Here, `OrderDate` is fixed (static) for all inserted rows in this statement. You must explicitly specify the partition value.

---

## 7. Problem with Static Partitioning

*   If a table has many partitions (e.g., daily sales data for several years), inserting data one partition at a time using static partitioning can be extremely time-consuming and cumbersome.

---

## 8. Dynamic Partitioning

**Solution** = Hive automatically creates and manages partitions at data load time, inferring the partition values from the data itself.      

### Enable Dynamic Partitioning (per session):

```sql
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;
```

### Modes for `hive.exec.dynamic.partition.mode`:

*   **`STRICT`**: Requires at least one static partition to be specified in the `INSERT` statement. This prevents accidental full table scans if no partition columns are provided.
*   **`NONSTRICT`**: All partitions can be dynamic. This is more flexible but requires caution, especially with large datasets, to avoid creating too many small partitions.

---

## 9. Dynamic Partition Insert

Suppose we have:

### Source Table (unpartitioned):

```sql
CREATE TABLE Sales_Data_Without_Partition
(
   StoreLocation VARCHAR(30),
   Product VARCHAR(30),
   OrderDate DATE,
   Revenue DECIMAL(10,2)
);
```

### Destination Table (partitioned by Product, OrderDate):

```sql
CREATE TABLE Sales_Data_Date_Product_Partition
(
   StoreLocation VARCHAR(30),
   Revenue DECIMAL(10,2)
)
PARTITIONED BY (Product VARCHAR(30), OrderDate DATE);
```

### Insert with Dynamic Partitioning:

```sql
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;

INSERT INTO Sales_Data_Date_Product_Partition
PARTITION(Product, OrderDate) -- These are the dynamic partition columns
SELECT StoreLocation, Revenue, Product, OrderDate -- Partition columns MUST be last in SELECT
FROM Sales_Data_Without_Partition;
```

### IMPORTANT:

*   The partition columns (`Product`, `OrderDate` in this example) must be the **last columns** in the `SELECT` statement in the `INSERT` query, and their order must match the order specified in the `PARTITION()` clause.
*   Hive automatically creates new partition directories based on the values in these columns as data is inserted.

---

## 10. Checking Existing Partitions

### Syntax:

```sql
SHOW PARTITIONS table_name;
```

### Example:

```sql
SHOW PARTITIONS Sales_Data_Product_Partition;
```

### Check HDFS directly:

```bash
hadoop fs -ls /user/hive/warehouse/sales_data_product_partition
```

---

## 11. Summary

*   **Partitioning** splits Hive tables into subdirectories on HDFS based on column values.
*   Use for **performance optimization**, especially for queries that filter on partition columns.
*   **Common query filters** are the best candidates for partition columns.
*   **Avoid high cardinality columns** for partitioning to prevent an excessive number of small partitions and files.
*   **Static partitioning**: You manually specify the partition value(s) during insertion.
*   **Dynamic partitioning**: Hive automatically determines and creates partitions based on the data values being inserted, making bulk loading easier.
