Hive Functions
==============

1. Standard Functions
----------------------
- Take rows/columns in a row as arguments.
Example:
    SELECT CONCAT(fname, " ", lname) FROM employees;

- Common string functions:
    LENGTH(), REVERSE(), REGEXP_REPLACE()

------------------------------------------------------------

2. Aggregate Functions
-----------------------
- Examples: SUM(), COUNT(), AVG()
- Take multiple rows as input and return a single result.
- Generally used with the GROUP BY clause.

------------------------------------------------------------

3. Table-Generating Functions
------------------------------
- Input: 1 row
- Output: Multiple rows
- Normally operate on collection data types like ARRAYS, MAPS, STRUCTS.
- One row is returned for each element in the collection.

Example:
    SELECT EXPLODE(ARRAY(1, 2, 3));
    Output:
        1
        2
        3

Example:
    SELECT EXPLODE(subordinates)
    FROM employeeDetails
    WHERE empName = "Vitthal";

    Output:
        Anuradha
        Arun
        Swetha

- EXPLODE() breaks up the array of subordinates into multiple rows.

Including Manager Name with EXPLODE():
--------------------------------------
    This will throw an error:
        SELECT empName, EXPLODE(subordinates)
        FROM employeeDetails
        WHERE empName = "Vitthal";

    Reason:
        empName has 1 row, but EXPLODE(subordinates) has 3 rows.

    Solution:
        Use LATERAL VIEW.

Example:
    SELECT empName, subordinates
    FROM employeeDetails
    LATERAL VIEW EXPLODE(subordinates) exp AS subordinates
    WHERE empName = "Vitthal";

    Output:
        Vitthal   Anuradha
        Vitthal   Arun
        Vitthal   Swetha

- LATERAL VIEW joins the exploded table back to the original table.
- By default, it is an INNER JOIN.
  If an employee has NULL in the subordinates column, they will be excluded.

------------------------------------------------------------

4. CASE..WHEN
--------------
Example:
    SELECT empname,
           CASE
               WHEN tenure < 2 THEN 0
               WHEN tenure >= 2 AND tenure <= 3 THEN 2
               WHEN tenure > 3 THEN 3
           END AS extra_vacation_days
    FROM employeeTenures;

------------------------------------------------------------

5. size()
---------
- Returns the number of elements in arrays and maps.
- Does not work with structs and unions.

Examples:
    SELECT SIZE(ARRAY(1, 2, 3));
    SELECT SIZE(MAP("name", "Swetha", "age", 30));

------------------------------------------------------------

6. CAST()
---------
- Converts from one data type to another.
- Hive enforces type safety — cannot insert a value of one type into a column of a different type.

Example:
    SELECT CAST("25" AS BIGINT);

- If a value cannot be converted, NULL is returned.

Example:
    SELECT CAST("abc" AS BIGINT);
    Output:
        NULL

------------------------------------------------------------

7. describe function
--------------------
- To get details of a function:
    DESCRIBE FUNCTION concat;

- To get description along with example:
    DESCRIBE EXTENDED FUNCTION concat;
