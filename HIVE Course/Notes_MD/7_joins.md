# Hive Joins & Optimizations

---

## 1. General Join Rules & Optimizations

*   Always write the join query so that the **largest table comes last**.
*   Except for the last table, other tables are joined using **map-side operations** (in-memory). This is generally faster.
*   The last table is typically processed in the **reduce stage**, which is generally slower.
*   The `WHERE` clause is applied **after** the join is complete, not before the join operation itself. This is an important consideration for performance.

---

## 2. Types of Joins in Hive

### 2.1 Inner Join

*   Returns rows only when there is a match in both tables based on the join condition.

```sql
SELECT A.id, A.name, B.salary
FROM Employees A
INNER JOIN Salaries B
ON A.id = B.emp_id;
```

---

### 2.2 Left Outer Join

*   Returns all rows from the **left table**, and the matching rows from the right table.
*   If there is no match in the right table, `NULL` is returned for the columns from the right table.

```sql
SELECT A.id, A.name, B.salary
FROM Employees A
LEFT OUTER JOIN Salaries B
ON A.id = B.emp_id;
```

---

### 2.3 Right Outer Join

*   Returns all rows from the **right table**, and the matching rows from the left table.
*   If there is no match in the left table, `NULL` is returned for the columns from the left table.

```sql
SELECT A.id, A.name, B.salary
FROM Employees A
RIGHT OUTER JOIN Salaries B
ON A.id = B.emp_id;
```

---

### 2.4 Full Outer Join

*   Returns all rows from both tables.
*   If a row from one table does not have a match in the other table, the columns from the non-matching side will be filled with `NULL`s.     

```sql
SELECT A.id, A.name, B.salary
FROM Employees A
FULL OUTER JOIN Salaries B
ON A.id = B.emp_id;
```

---

### 2.5 Left Semi Join

*   This join type is equivalent to an `IN` or `EXISTS` subquery but is often more optimized in Hive.
*   It returns rows from the **left table** only where a match is found in the right table.
*   **Important:** Columns from the right table **cannot** be selected in the `SELECT` clause when using a `LEFT SEMI JOIN`. It's purely for existence checks.

**Subquery method (less efficient for large tables):**
```sql
SELECT NAMES.SYMBOL
FROM NAMES
WHERE NAMES.SYMBOL IN (SELECT SYMBOL FROM REVENUE);
```

**Equivalent Left Semi Join (more efficient):**
```sql
SELECT NAMES.SYMBOL
FROM NAMES
LEFT SEMI JOIN REVENUE
ON NAMES.SYMBOL = REVENUE.SYMBOL;
```

**Efficiency:**
A `LEFT SEMI JOIN` is efficient because Hive scans the right table only until a match is found for a given left-table row, unlike `IN`/`EXISTS` which might scan the entire subquery result.

**Difference from `IN`/`EXISTS`:**
*   `IN`/`EXISTS` typically only allow checking against a single column in the subquery result.
*   `LEFT SEMI JOIN` can use multiple columns in its `ON` clause for the join condition.

**Example (valid with `LEFT SEMI JOIN`, potentially invalid or less efficient with `IN`/`EXISTS`):**
```sql
SELECT NAMES.SYMBOL
FROM NAMES
LEFT SEMI JOIN REVENUE
ON (NAMES.SYMBOL = REVENUE.SYMBOL)
AND (NAMES.NAME = REVENUE.NAME);
```

---

## 3. Map-Side Joins

### What is a Map-Side Join?
A Map-Side Join is a type of join that executes entirely in the Map phase of a MapReduce job, thus **avoiding the use of reducers**. This makes them significantly faster for certain scenarios.

### Conditions for Map-Side Join:
*   All but one of the tables involved in the join must be **small** enough to fit into the memory of the mappers.
*   The small tables are loaded into in-memory hash tables via Hadoop's **distributed cache**.
*   The large table is split and processed by mappers. Each mapper can then join its portion of the large table with the in-memory small tables.

### When Map-Side Join Works?
The applicability of a Map-Side Join depends on the join type and the relative sizes of the tables.

*   **Inner Join:** Works if one table is small (either left or right).
*   **Left Outer Join:** Works if the **right table is small** (the large table is on the left).
*   **Right Outer Join:** Works if the **left table is small** (the large table is on the right).
*   **Full Outer Join:** Generally **not possible** with standard Map-Side Joins due to the need to handle unmatched rows from both sides.    

### Explicitly Enforcing Map Join
You can hint Hive to use a Map-Side Join using the `/*+ MAPJOIN(table_name) */` syntax.

```sql
SELECT /*+ MAPJOIN(TRADES) */
NAMES.SYMBOL, SERIES, HIGH
FROM NAMES
JOIN TRADES
ON NAMES.SYMBOL = TRADES.SYMBOL;
```
*(In this example, `TRADES` is expected to be the smaller table that will be loaded into memory.)*

---

## 4. When to Use Which Join?

*   **Inner Join:** Use when you need only the rows that have matching values in **both** tables.
*   **Left Outer Join:** Use when you need **all rows from the left table**, along with any matching rows from the right table.
*   **Right Outer Join:** Use when you need **all rows from the right table**, along with any matching rows from the left table.
*   **Full Outer Join:** Use when you need **all rows from both tables**, regardless of whether they have a match in the other table.
*   **Left Semi Join:** Use when you need to check for the **existence** of related records (similar to `IN` or `EXISTS` subqueries) but want a more performant alternative in Hive.
*   **Map-Side Join:** Consider enabling or hinting for a Map-Side Join when **one or more tables are significantly smaller** than the others, to drastically improve query performance.

---

## Key Takeaways

1.  **Table Order Matters:** Place the largest table last in join queries to optimize reducer-stage processing.
2.  **`WHERE` Clause Timing:** The `WHERE` clause applies after the join completes, which can impact performance if not planned well (consider pre-filtering data if possible).
3.  **`LEFT SEMI JOIN` for Existence:** This is the most efficient way to perform `IN`/`EXISTS` checks in Hive.
4.  **Map-Side Join for Performance:** Leverage Map-Side Joins when applicable (small tables fit in memory) to significantly speed up join operations by avoiding the reducer phase.
5.  **Choose Join Type Wisely:** Select the appropriate join type (`INNER`, `LEFT/RIGHT/FULL OUTER`, `LEFT SEMI`) based on whether you need unmatched rows, which side's rows must be preserved, and the relative sizes of your tables.