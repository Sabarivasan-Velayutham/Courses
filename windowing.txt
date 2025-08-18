# Hive Windowing (Analytic) Functions

------------------------------------------------------------
🔹 What are Window Functions?
------------------------------------------------------------
* A window function performs a calculation across a set of rows related to the current row.
* This “set of rows” is called a window.
* Unlike aggregate functions (SUM, AVG, etc. with GROUP BY), window functions do not collapse rows into a single result. 
  Instead, they return a result for every row.
* Think of them as “running aggregations” or “row-wise insights.”

------------------------------------------------------------
1. Running Total (Simple Example)
------------------------------------------------------------
We have a Sales_Data table:

| OrderID | StoreID | ProductID | OrderDate  | Revenue |
| ------- | ------- | --------- | ---------- | ------- |
| 1       | 2       | 2         | 2016-01-17 | 1299.45 |
| 2       | 1       | 1         | 2016-01-17 | 2342.33 |
| 3       | 1       | 2         | 2016-01-17 | 4543.98 |

Query: Running Total of Revenue

SELECT OrderID, StoreID, ProductID, OrderDate, Revenue,
       SUM(Revenue) OVER (
           ORDER BY OrderID
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS Running_Total
FROM Sales_Data;

Explanation:
- OVER → marks it as a window function.
- ORDER BY OrderID → defines row order.
- ROWS BETWEEN … → sum from the first row up to the current row.
- Output → cumulative sum updated row by row.

------------------------------------------------------------
2. Partitioned Running Total (Reset by Day)
------------------------------------------------------------
SELECT OrderID, StoreID, ProductID, OrderDate, Revenue,
       SUM(Revenue) OVER (
           PARTITION BY OrderDate
           ORDER BY OrderID
       ) AS Running_Total
FROM Sales_Data;

Explanation:
- PARTITION BY OrderDate → creates a separate window for each date.
- Running total starts fresh at the beginning of each partition.

------------------------------------------------------------
3. Types of Window Functions
------------------------------------------------------------

A) Ranking Functions
---------------------

1. ROW_NUMBER()
Assigns unique row numbers.

Example:
SELECT OrderID, StoreID, Revenue,
       ROW_NUMBER() OVER (PARTITION BY StoreID ORDER BY Revenue DESC) AS row_num
FROM Sales_Data;

---

2. RANK()   (with gaps)
Assigns ranks with gaps when ties exist.

Example:
SELECT OrderID, StoreID, Revenue,
       RANK() OVER (PARTITION BY StoreID ORDER BY Revenue DESC) AS rank_val
FROM Sales_Data;

Data Illustration:
Revenue → Rank
1000    → 1
800     → 2
500     → 3
500     → 3
400     → 5   (gap: 4 skipped)

---

3. DENSE_RANK()   (no gaps)
Ranks without gaps.

Example:
SELECT OrderID, StoreID, Revenue,
       DENSE_RANK() OVER (PARTITION BY StoreID ORDER BY Revenue DESC) AS dense_rank_val
FROM Sales_Data;

Data Illustration:
Revenue → Dense_Rank
1000    → 1
800     → 2
500     → 3
500     → 3
400     → 4   (no gap)

------------------------------------------------------------

B) Value Functions
-------------------

1. LAG()
Shows the previous row's revenue per partition.

Example:
SELECT OrderID, StoreID, Revenue,
       LAG(Revenue, 1) OVER (PARTITION BY StoreID ORDER BY OrderID) AS prev_revenue
FROM Sales_Data;

---

2. LEAD()
Shows the next row's revenue per partition.

Example:
SELECT OrderID, StoreID, Revenue,
       LEAD(Revenue, 1) OVER (PARTITION BY StoreID ORDER BY OrderID) AS next_revenue
FROM Sales_Data;

---

3. FIRST_VALUE()
Shows the first value in a partition.

Example:
SELECT OrderID, StoreID, Revenue,
       FIRST_VALUE(Revenue) OVER (PARTITION BY StoreID ORDER BY OrderID) AS first_rev
FROM Sales_Data;

---

4. LAST_VALUE()
Shows the last value in a partition.

Example:
SELECT OrderID, StoreID, Revenue,
       LAST_VALUE(Revenue) OVER (PARTITION BY StoreID ORDER BY OrderID
                                 ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS last_rev
FROM Sales_Data;

------------------------------------------------------------
Key Takeaways
------------------------------------------------------------
1. Window functions = row-wise aggregations over a defined frame of rows.
2. OVER (ORDER BY …) defines row order.
3. PARTITION BY splits data into independent subsets.
4. Great for cumulative sums, moving averages, ranks, and row comparisons.

