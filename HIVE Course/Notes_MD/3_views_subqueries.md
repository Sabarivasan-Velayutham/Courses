# Hive Subqueries, Set Operations, and Views - Notes

## 1. UNION and UNION ALL in Hive

*   Hive supports `UNION` and `UNION ALL`.
*   `UNION` removes duplicates (like `DISTINCT`).
*   `UNION ALL` keeps duplicates.

### Syntax:

```sql
SELECT col1, col2 FROM table1
UNION
SELECT col1, col2 FROM table2;
```

```sql
SELECT col1, col2 FROM table1
UNION ALL
SELECT col1, col2 FROM table2;
```

**Query:**
```sql
SELECT id FROM table1
UNION
SELECT id FROM table2;
```


## 2. INTERSECT and MINUS

*   Hive does not support `INTERSECT` and `MINUS` directly.
*   These operations can be achieved using joins or subqueries.

### Example for INTERSECT (common rows):

```sql
SELECT a.id
FROM table1 a
JOIN table2 b ON a.id = b.id;
```

### Example for MINUS (rows in `table1` but not in `table2`):

```sql
SELECT a.id
FROM table1 a
LEFT JOIN table2 b ON a.id = b.id
WHERE b.id IS NULL;
```

---

## 3. EXISTS, IN, NOT IN in Hive

*   `EXISTS`, `IN`, `NOT IN` are supported in Hive subqueries.
*   `IN` / `NOT IN` must return a single column.

### Example (IN):

```sql
SELECT id, name
FROM employees e
WHERE dept_id IN (SELECT id FROM departments WHERE location='Chennai');
```

### Example (NOT IN):

```sql
SELECT id, name
FROM employees e
WHERE dept_id NOT IN (SELECT id FROM departments WHERE location='Chennai');
```

### Example (EXISTS):

```sql
SELECT name
FROM employees e
WHERE EXISTS (SELECT 1 FROM departments d WHERE d.id = e.dept_id);
```

**Note:** Equality (`=`) with a subquery is not supported in Hive (i.e., `WHERE column = (SELECT ...)` where the subquery returns more than one row).

---

## 4. Subquery Restrictions in Hive

*   The referenced columns in subqueries should be in small letters (this might be specific to certain Hive versions or configurations, generally case-insensitivity applies to identifiers unless quotes are used).
*   Hive does not allow multiple subqueries in the same outer query (this applies to certain types of subqueries, e.g., correlated subqueries, or multiple non-correlated subqueries in the `WHERE` clause).
*   Nested subqueries are allowed.

### Example (Nested Subquery):

```sql
SELECT id
FROM (SELECT id FROM employees WHERE salary > 50000) sub
WHERE id > 100;
```

---

## 5. VIEWS in Hive

*   Views help simplify complex queries.
*   A view acts like a table but is read-only.
*   Data cannot be inserted/loaded into a view.
*   Metadata of a view can be changed using `ALTER VIEW`.
*   Views are shown in `SHOW TABLES` list.

### Syntax to create a view:

```sql
CREATE VIEW emp_high_salary AS
SELECT id, name, salary
FROM employees
WHERE salary > 50000;
```

### Querying a view:

```sql
SELECT * FROM emp_high_salary;
```

### Describe a view:

```sql
DESCRIBE EXTENDED emp_high_salary;
```

### Alter a view:

```sql
ALTER VIEW emp_high_salary AS
SELECT id, name
FROM employees
WHERE salary > 60000;
```

### Reasons to use views:
1.  **Reduce query complexity:** Simplify complex, frequently used queries by encapsulating them in a view.
2.  **Restrict access:** Expose only required columns/rows to specific users, enhancing security.
3.  **Construct multiple logical tables:** Create different logical representations of data from a single physical table without duplicating data.