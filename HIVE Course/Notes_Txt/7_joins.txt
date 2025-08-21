Hive Joins & Optimizations

------------------------------------------------------------------------

1. General Join Rules & Optimizations

-   Always write the join query so that the largest table comes last.
-   Except the last table, other tables are joined using map-side
    operations (in-memory).
-   The last table is processed in the reduce stage (slower).
-   WHERE clause is applied after the join is complete (not before).

------------------------------------------------------------------------

2. Types of Joins in Hive

2.1 Inner Join

-   Returns rows when there is a match in both tables.

    SELECT A.id, A.name, B.salary
    FROM Employees A
    INNER JOIN Salaries B
    ON A.id = B.emp_id;

------------------------------------------------------------------------

2.2 Left Outer Join

-   Returns all rows from the left table, and matching rows from the
    right table.
-   If no match, NULL is returned for right table columns.

    SELECT A.id, A.name, B.salary
    FROM Employees A
    LEFT OUTER JOIN Salaries B
    ON A.id = B.emp_id;

------------------------------------------------------------------------

2.3 Right Outer Join

-   Returns all rows from the right table, and matching rows from the
    left table.

    SELECT A.id, A.name, B.salary
    FROM Employees A
    RIGHT OUTER JOIN Salaries B
    ON A.id = B.emp_id;

------------------------------------------------------------------------

2.4 Full Outer Join

-   Returns all rows from both tables; unmatched rows are filled with
    NULLs.

    SELECT A.id, A.name, B.salary
    FROM Employees A
    FULL OUTER JOIN Salaries B
    ON A.id = B.emp_id;

------------------------------------------------------------------------

2.5 Left Semi Join

-   Equivalent to IN or EXISTS but optimized.
-   Returns rows from the left table where a match is found in the right
    table.
-   Right table columns cannot be selected.

    -- Subquery method
    SELECT NAMES.SYMBOL
    FROM NAMES
    WHERE NAMES.SYMBOL IN (SELECT SYMBOL FROM REVENUE);

    -- Equivalent Left Semi Join (efficient)
    SELECT NAMES.SYMBOL
    FROM NAMES
    LEFT SEMI JOIN REVENUE
    ON NAMES.SYMBOL = REVENUE.SYMBOL;

Efficient because Hive scans the right table only until a match is
found, unlike IN/EXISTS which scans entire table.

Difference: - IN/EXISTS only allows one column in subquery. -
LEFT SEMI JOIN can use multiple columns in ON clause.

Example (valid with SEMI JOIN, invalid with IN/EXISTS):

    SELECT NAMES.SYMBOL
    FROM NAMES
    LEFT SEMI JOIN REVENUE
    ON (NAMES.SYMBOL = REVENUE.SYMBOL)
    AND (NAMES.NAME = REVENUE.NAME);

------------------------------------------------------------------------

3. Map-Side Joins

What is a Map-Side Join?

-   A join that avoids reducers (faster execution).
-   Conditions:
    -   All but one table must be small.
    -   Small tables are loaded into in-memory hash tables via Hadoop
        distributed cache.
    -   The large table is split and processed by mappers.

When Map-Side Join Works?

-   Depends on join type and table sizes.

Case 1: Small table LEFT, large table RIGHT - Inner Join → works 
Left Outer Join → incorrect (can’t handle unmatched left rows) 
Right Outer Join → works 
Full Outer Join → not possible

Case 2: Small table RIGHT, large table LEFT → Reverse of above.

🔹 Explicitly Enforcing Map Join

    SELECT /*+ MAPJOIN(TRADES) */
    NAMES.SYMBOL, SERIES, HIGH
    FROM NAMES
    JOIN TRADES
    ON NAMES.SYMBOL = TRADES.SYMBOL;

------------------------------------------------------------------------

4. When to Use Which Join?

-   Inner Join → when only matched rows are needed.
-   Left Outer Join → when all rows from left + matched right rows are
    needed.
-   Right Outer Join → when all rows from right + matched left rows are
    needed.
-   Full Outer Join → when all rows from both tables are needed (with
    NULLs for unmatched).
-   Left Semi Join → when checking existence (IN/EXISTS) but faster.
-   Map-Side Join → when most tables are small and one is large (for
    performance).

------------------------------------------------------------------------

Key Takeaways

1.  Place the largest table last in join queries.
2.  WHERE clause applies after join, not before.
3.  Left Semi Join is best for existence checks (IN/EXISTS replacement).
4.  Map-Side Joins drastically improve performance when applicable.
5.  Choose join type based on:
    -   Whether you need unmatched rows,
    -   Which side’s rows must be preserved,
    -   Table sizes.
