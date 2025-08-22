# Hive Windowing (Analytic) Functions

## What are Window Functions?

*   A window function performs a calculation across a set of rows related to the current row.
*   This “set of rows” is called a **window**.
*   Unlike aggregate functions (e.g., `SUM`, `AVG`, etc. used with `GROUP BY`), window functions do not collapse rows into a single result. Instead, they return a result for every row.
*   Think of them as “running aggregations” or “row-wise insights.”

---

## 1. Running Total (Simple Example)

We have a `Sales_Data` table:

| OrderID | StoreID | ProductID | OrderDate  | Revenue   |
| :------ | :------ | :-------- | :--------- | :-------- |
| 1       | 2       | 2         | 2016-01-17 | 1299.45   |
| 2       | 1       | 1         | 2016-01-17 | 2342.33   |
| 3       | 1       | 2         | 2016-01-17 | 4543.98   |
| 4       | 2       | 1         | 2016-01-18 | 1500.00   |
| 5       | 1       | 3         | 2016-01-18 | 3000.00   |

**Query: Running Total of Revenue**

```sql
SELECT OrderID, StoreID, ProductID, OrderDate, Revenue,
       SUM(Revenue) OVER (
           ORDER BY OrderID
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS Running_Total
FROM Sales_Data;
```

**Explanation:**

*   `OVER()` → Marks this as a window function.
*   `ORDER BY OrderID` → Defines the order of rows within the entire dataset for calculation.
*   `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` → Defines the "window frame" for the `SUM` calculation. It means sum all rows from the very first row up to the current row.
*   **Output** → A cumulative sum that updates row by row based on `OrderID`.

---

## 2. Partitioned Running Total (Reset by Day)

If you want the running total to reset for each `OrderDate`:

```sql
SELECT OrderID, StoreID, ProductID, OrderDate, Revenue,
       SUM(Revenue) OVER (
           PARTITION BY OrderDate
           ORDER BY OrderID
       ) AS Running_Total
FROM Sales_Data;
```

**Explanation:**

*   `PARTITION BY OrderDate` → Divides the dataset into separate, independent windows for each unique `OrderDate`.
*   The `ORDER BY OrderID` then applies within each `OrderDate` partition.
*   The running total starts fresh at the beginning of each `OrderDate` partition.

---

## 3. Types of Window Functions

### A) Ranking Functions

These functions assign a rank or sequential number to each row within its window.

#### 1. `ROW_NUMBER()`
Assigns unique, sequential row numbers starting from 1 within each partition.

**Example:**
```sql
SELECT OrderID, StoreID, Revenue,
       ROW_NUMBER() OVER (PARTITION BY StoreID ORDER BY Revenue DESC) AS row_num
FROM Sales_Data;
```

#### 2. `RANK()` (with gaps)
Assigns ranks with gaps when ties exist. If two rows have the same value, they get the same rank, and the next rank is skipped.

**Example:**
```sql
SELECT OrderID, StoreID, Revenue,
       RANK() OVER (PARTITION BY StoreID ORDER BY Revenue DESC) AS rank_val
FROM Sales_Data;
```

**Data Illustration for `RANK()`:**
Revenue   → Rank
1000      → 1
800       → 2
500       → 3
500       → 3
400       → 5   (gap: rank 4 is skipped)

#### 3. `DENSE_RANK()` (no gaps)
Assigns ranks without gaps. If two rows have the same value, they get the same rank, and the next rank is consecutive.

**Example:**
```sql
SELECT OrderID, StoreID, Revenue,
       DENSE_RANK() OVER (PARTITION BY StoreID ORDER BY Revenue DESC) AS dense_rank_val
FROM Sales_Data;
```

**Data Illustration for `DENSE_RANK()`:**
Revenue   → Dense_Rank
1000      → 1
800       → 2
500       → 3
500       → 3
400       → 4   (no gap)

---

### B) Value Functions

These functions retrieve a value from a row within the window, relative to the current row.

#### 1. `LAG()`
Retrieves a value from a row that is a specified number of physical rows before the current row within the partition.

**Example:** Get the previous order's revenue for each store.
```sql
SELECT OrderID, StoreID, Revenue,
       LAG(Revenue, 1) OVER (PARTITION BY StoreID ORDER BY OrderID) AS prev_revenue
FROM Sales_Data;
```

#### 2. `LEAD()`
Retrieves a value from a row that is a specified number of physical rows after the current row within the partition.

**Example:** Get the next order's revenue for each store.
```sql
SELECT OrderID, StoreID, Revenue,
       LEAD(Revenue, 1) OVER (PARTITION BY StoreID ORDER BY OrderID) AS next_revenue
FROM Sales_Data;
```

#### 3. `FIRST_VALUE()`
Returns the value of the expression from the first row in the window frame.

**Example:** Get the revenue of the first order for each store.
```sql
SELECT OrderID, StoreID, Revenue,
       FIRST_VALUE(Revenue) OVER (PARTITION BY StoreID ORDER BY OrderID) AS first_rev
FROM Sales_Data;
```

#### 4. `LAST_VALUE()`
Returns the value of the expression from the last row in the window frame.
*(By default, the window frame is from `UNBOUNDED PRECEDING` to `CURRENT ROW`. To get the true last value of the partition, the frame needs to be adjusted.)*

**Example:** Get the revenue of the last order for each store.
```sql
SELECT OrderID, StoreID, Revenue,
       LAST_VALUE(Revenue) OVER (PARTITION BY StoreID ORDER BY OrderID
                                 ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS last_rev
FROM Sales_Data;
```

---

## Key Takeaways

1.  **Window functions** perform row-wise aggregations or calculations over a defined frame of rows, without collapsing the result set.       
2.  `OVER (ORDER BY ...)` defines the order of rows within the window.
3.  `PARTITION BY ...` splits the data into independent subsets (partitions), and the window function operates separately within each partition.
4.  Window functions are powerful for tasks like calculating cumulative sums, moving averages, assigning ranks, and comparing values between rows (`LAG`, `LEAD`).