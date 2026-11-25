# SQL Practice Queries

---

## Quick Reference — RANK / DENSE_RANK / ROW_NUMBER

```sql
FUNCTION() OVER (PARTITION BY [group] ORDER BY [sort])
```

Think of it as answering: **"What's my rank within my group, sorted by this?"**

| Part | Maps to | Analogy |
|------|---------|---------|
| `FUNCTION()` | RANK / DENSE_RANK / ROW_NUMBER | "Which numbering style?" |
| `PARTITION BY` | GROUP BY equivalent for windows | "Start counting from 1 again per ___" |
| `ORDER BY` | What determines position | "Rank by ___" |

**When to use which:**

| Function | Ties | Gaps | Example ranks for scores 100, 90, 90, 80 |
|----------|------|------|------------------------------------------|
| `ROW_NUMBER` | Breaks ties arbitrarily | No | 1, 2, 3, 4 |
| `RANK` | Same rank for ties | Yes (skips) | 1, 2, 2, **4** |
| `DENSE_RANK` | Same rank for ties | No (consecutive) | 1, 2, 2, **3** |

**Mnemonic:** "**D**ense = **D**on't skip" — DENSE_RANK never leaves gaps.

**Minimal template:**

```sql
SELECT *,
       DENSE_RANK() OVER (PARTITION BY DeptID ORDER BY Salary DESC) AS Rnk
FROM   Employee;
```

If you don't need groups, just drop `PARTITION BY` — you get a single ranking across the whole table.

---

## Table of Contents

- [1. Self-Join — Manager Lookup](#1-self-join--manager-lookup)
- [2. Age Filtering with DATEDIFF](#2-age-filtering-with-datediff)
- [3. Deleting Duplicate Records](#3-deleting-duplicate-records)
- [4. N'th Highest Salary (Correlated Subquery, TOP & OFFSET)](#4-nth-highest-salary-correlated-subquery-top--offset)
- [5. Conditional SUM — Negatives vs Positives](#5-conditional-sum--negatives-vs-positives)
- [6. Finding Duplicate Records with COUNT](#6-finding-duplicate-records-with-count)
- [7. Swap Gender Values with CASE](#7-swap-gender-values-with-case)
- [8. SalesPerson & Orders — Multi-Table Queries](#8-salesperson--orders--multi-table-queries)
- [9. PIVOT — Rows to Columns](#9-pivot--rows-to-columns)
- [10. Topper per Subject](#10-topper-per-subject)
- [11. JOIN Row Counts with Duplicate Keys](#11-join-row-counts-with-duplicate-keys)
- [12. Three-Table JOIN Row Counts](#12-three-table-join-row-counts)
- [13. Year with Maximum Hires](#13-year-with-maximum-hires)
- [14. Top N per Department (RANK)](#14-top-n-per-department-rank)
- [15. Employees Above Department Average](#15-employees-above-department-average)
- [16. Period of Service (Years & Months)](#16-period-of-service-years--months)
- [17. Even / Odd Row Selection (Oracle)](#17-even--odd-row-selection-oracle)
- [18. Clone Table Structure](#18-clone-table-structure)
- [19. First / Last N Characters](#19-first--last-n-characters)
- [20. Total Salary by Department (HAVING)](#20-total-salary-by-department-having)
- [21. Companies with < 5 Employees](#21-companies-with--5-employees)
- [22. Recursive / Hierarchical Queries (CTE)](#22-recursive--hierarchical-queries-cte)
- [23. Window Functions (SUM OVER PARTITION BY) — Sales Percentage](#23-window-functions-sum-over-partition-by--sales-percentage)

---

## 1. Self-Join — Manager Lookup

> Find ManagerID and ManagerName for every employee.

**Table — EMPMGR:**

| ID | Name  | MGRID | DOB        |
|---:|-------|------:|------------|
|  1 | Dave  |  NULL | 1982-01-12 |
|  2 | Scott |     1 | 1985-05-22 |
|  3 | Mike  |     1 | 1990-09-10 |
|  4 | Nick  |     3 | 1978-02-02 |
|  5 | Jack  |     1 | 1995-05-22 |
|  6 | Tim   |     3 | 1964-03-23 |
|  7 | Chris |     6 | 1985-07-29 |

```sql
SELECT t1.ID,
       t1.Name,
       t2.ID   AS ManagerID,
       t2.Name AS ManagerName
FROM   dbo.EMPMGR t1
LEFT JOIN dbo.EMPMGR t2 ON t1.MGRID = t2.ID;
```

---

## 2. Age Filtering with DATEDIFF

> From the EMPMGR table, find employees older than 35.

```sql
SELECT DATEDIFF(HOUR, DOB, GETDATE()) / 8766 AS Age,
       Name
FROM   dbo.EMPMGR
WHERE  DATEDIFF(HOUR, DOB, GETDATE()) / 8766 > 35;
```

---

## 3. Deleting Duplicate Records

> Delete duplicates where Name + City are repeated; keep the row with the highest ID.

**Table — DuplicateTable:**

| ID | Name  | City      |
|---:|-------|-----------|
|  1 | Jack  | NewYork   |
|  2 | Toad  | NewJersey |
|  3 | Perry | London    |
|  4 | Joe   | NewYork   |
|  5 | Jack  | NewYork   |
|  6 | Perry | London    |
|  7 | Scott | NewJersey |

```sql
DELETE FROM [dbo].[DuplicateTable]
WHERE ID NOT IN (
    SELECT MAX(ID)
    FROM   [dbo].[DuplicateTable]
    GROUP BY Name, City
);
```

---

## 4. N'th Highest Salary (Correlated Subquery, TOP & OFFSET)

> Find the N'th highest salary.
>
> Source: [programmerinterview.com](http://www.programmerinterview.com/index.php/database-sql/find-nth-highest-salary-sql/)

**Table — EmployeeSalary:**

| Name   | Salary |
|--------|-------:|
| Susan  |   5200 |
| Mike   |   2000 |
| Jordan |   3500 |
| Maria  |   2800 |
| Sunny  |   5000 |
| Tom    |   1800 |
| Adam   |   3400 |
| Sam    |   4500 |

### Approach A — Correlated Subquery

```sql
SELECT *
FROM   Employee Emp1
WHERE  (N - 1) = (
           SELECT COUNT(DISTINCT Emp2.Salary)
           FROM   Employee Emp2
           WHERE  Emp2.Salary > Emp1.Salary
       );
```

`Emp1` appears in **both** the outer query and the subquery — making this a **correlated subquery**. It re-evaluates for every row, giving O(n²) performance.

**Key Insight:**

> The N'th highest salary has exactly **N − 1** distinct salaries greater than it.
>
> - 2nd highest → 1 salary above it
> - 4th highest → 3 salaries above it

**Step-by-Step Walkthrough (N = 2, simplified data):**

| EmployeeID | Salary |
|:----------:|-------:|
| 3          |    200 |
| 4          |    800 |
| 7          |    450 |

With N = 2 the `WHERE` clause becomes `(1) = (...)`:

- **Salary = 200:** salaries > 200 are {450, 800} → COUNT = 2 → 2 ≠ 1 → skipped
- **Salary = 800:** salaries > 800 are {} → COUNT = 0 → 0 ≠ 1 → skipped
- **Salary = 450:** salaries > 450 are {800} → COUNT = 1 → 1 = 1 → **returned**

**Why DISTINCT?** Duplicate salaries should only count once — prevents two employees earning 800 from inflating the count.

### Approach B — TOP (SQL Server)

```sql
SELECT TOP 1 Salary
FROM (
    SELECT DISTINCT TOP N Salary
    FROM   Employee
    ORDER BY Salary DESC
) AS Emp
ORDER BY Salary ASC;
```

**How it works:**

1. Inner query selects the top N distinct salaries in descending order.
2. Outer query re-sorts ascending — the N'th highest is now first.
3. `TOP 1` returns that single value.

### Approach C — OFFSET / FETCH (SQL Server 2012+)

No subquery needed — best performance on large tables.

```sql
SELECT *
FROM   Employee
ORDER BY Salary DESC
OFFSET N - 1 ROWS
FETCH NEXT 1 ROWS ONLY;
```

**Example** for the 2nd highest salary (N = 2):

```sql
SELECT *
FROM   Employee
ORDER BY Salary DESC
OFFSET 1 ROWS
FETCH NEXT 1 ROWS ONLY;
```

Skips the first row (highest salary) and returns the next one.

### Approach D — CTE with DENSE_RANK()

Handles duplicate salaries correctly and reads the table only once.

```sql
WITH RankedSalaries AS (
    SELECT *,
           DENSE_RANK() OVER (ORDER BY Salary DESC) AS Rnk
    FROM   Employee
)
SELECT *
FROM   RankedSalaries
WHERE  Rnk = N;
```

`DENSE_RANK` assigns consecutive ranks with no gaps — two employees earning 5000 both get rank 2, and the next distinct salary gets rank 3. Use `DENSE_RANK` over `RANK` here to avoid skipping positions when duplicates exist.

---

## 5. Conditional SUM — Negatives vs Positives

> Sum negative and positive numbers into separate columns.

**Table — SignedInteger:**

| Number |
|-------:|
|     10 |
|    -15 |
|    -33 |
|    -58 |
|     32 |
|     -8 |
|    -62 |
|     69 |
|     78 |
|    -81 |

**Expected output:** Negative = −257, Positive = 189

**Approach A — Subqueries:**

```sql
SELECT (SELECT SUM(Number) FROM [dbo].SignedInteger WHERE Number < 0) AS Negative,
       (SELECT SUM(Number) FROM [dbo].SignedInteger WHERE Number > 0) AS Positive;
```

**Approach B — Conditional aggregation (SUM + CASE):**

```sql
SELECT SUM(CASE WHEN Number < 0 THEN Number END) AS Negative,
       SUM(CASE WHEN Number > 0 THEN Number END) AS Positive
FROM   [dbo].SignedInteger;
```

Single table scan instead of two subqueries. `CASE` rows that don't match evaluate to `NULL`, which `SUM` ignores.

---

## 6. Finding Duplicate Records with COUNT

> Show duplicate names and their occurrence count.

**Table — PersonNames:**

| Name  |
|-------|
| Tim   |
| Mike  |
| Susan |
| Tom   |
| Mike  |
| Kevin |
| Susan |
| Mike  |
| Joi   |

```sql
SELECT   Name, COUNT(*) AS OccurrenceCount
FROM     dbo.PersonNames
GROUP BY Name
HAVING   COUNT(*) > 1;
```

**Output:**

| Name  | OccurrenceCount |
|-------|----------------:|
| Mike  |               3 |
| Susan |               2 |

---

## 7. Swap Gender Values with CASE

> Change Male to Female and Female to Male in a single statement.

```sql
UPDATE dbo.PersonNames
SET    Gender = CASE
                   WHEN Gender = 'Male'   THEN 'Female'
                   WHEN Gender = 'Female' THEN 'Male'
                   ELSE NULL
               END;
```

---

## 8. SalesPerson & Orders — Multi-Table Queries

**Table — SalesPerson:**

| ID | Name  | Age | Salary |
|---:|-------|----:|-------:|
|  1 | Nick  |  45 |   8000 |
|  2 | Bob   |  32 |   5000 |
|  3 | Tim   |  54 |   8500 |
|  4 | Mike  |  28 |   4200 |
|  5 | Joe   |  35 |   4800 |
|  6 | Chris |  40 |   5200 |
|  7 | Toad  |  34 |   4800 |

**Table — Orders:**

| OrderID | OrderDate  | CustID | SalesPersonID | Amount |
|--------:|------------|-------:|--------------:|-------:|
|       1 | 2014-12-10 |     45 |             4 |    340 |
|       2 | 2014-02-10 |     56 |             1 |    200 |
|       3 | 2014-04-12 |     89 |             4 |    450 |
|       4 | 2014-06-02 |     34 |             6 |    890 |
|       5 | 2014-04-23 |     21 |             4 |    340 |
|       6 | 2014-01-10 |     34 |             7 |    650 |
|       7 | 2014-04-26 |     76 |             6 |    900 |
|       8 | 2014-06-28 |     88 |             1 |    230 |

### 8a. SalesPersons with more than 1 order

| Name  |
|-------|
| Nick  |
| Mike  |
| Chris |

```sql
SELECT S.Name
FROM   dbo.SalesPerson S
INNER JOIN (
    SELECT   SalesPersonID
    FROM     dbo.Orders
    GROUP BY SalesPersonID
    HAVING   COUNT(*) > 1
) O ON S.ID = O.SalesPersonID;
```

### 8b. SalesPersons with total order amount > 1000

| Name  | Amount |
|-------|-------:|
| Chris |   1790 |
| Mike  |   1130 |

```sql
SELECT S.ID,
       S.Name,
       O.OrderCount,
       O.TotalAmount
FROM   dbo.SalesPerson S
INNER JOIN (
    SELECT   SalesPersonID,
             COUNT(*)    AS OrderCount,
             SUM(Amount) AS TotalAmount
    FROM     dbo.Orders
    GROUP BY SalesPersonID
    HAVING   COUNT(*) > 1
) O ON S.ID = O.SalesPersonID
WHERE  O.TotalAmount > 1000;
```

---

## 9. PIVOT — Rows to Columns

> Transform subject marks from rows into columns.

**Table — MarkReports:**

| Name    | Subject | Marks |
|---------|---------|------:|
| GEORGE  | ECO     |    77 |
| GEORGE  | HIS     |    99 |
| GEORGE  | MAT     |    64 |
| GEORGE  | GEO     |    85 |
| GEORGE  | SCI     |    98 |
| ROBERT  | ECO     |    71 |
| ROBERT  | HIS     |    90 |
| ROBERT  | MAT     |    84 |
| ROBERT  | GEO     |    95 |
| ROBERT  | SCI     |    58 |
| TIMOTHY | ECO     |    56 |
| TIMOTHY | HIS     |    55 |
| TIMOTHY | MAT     |    67 |
| TIMOTHY | GEO     |    54 |
| TIMOTHY | SCI     |    45 |

**Expected output:**

| NAME    | ECO | HIS | MAT | GEO | SCI |
|---------|----:|----:|----:|----:|----:|
| GEORGE  |  77 |  99 |  64 |  85 |  98 |
| ROBERT  |  71 |  90 |  84 |  95 |  58 |
| TIMOTHY |  56 |  55 |  67 |  54 |  45 |

**Approach A — Conditional aggregation (portable):**

```sql
SELECT Name,
       MAX(CASE WHEN Subject = 'ECO' THEN Marks END) AS ECO,
       MAX(CASE WHEN Subject = 'HIS' THEN Marks END) AS HIS,
       MAX(CASE WHEN Subject = 'MAT' THEN Marks END) AS MAT,
       MAX(CASE WHEN Subject = 'GEO' THEN Marks END) AS GEO,
       MAX(CASE WHEN Subject = 'SCI' THEN Marks END) AS SCI
FROM   dbo.MarkReports
GROUP BY Name;
```

**Approach B — PIVOT operator (SQL Server):**

```sql
SELECT Name, ECO, HIS, MAT, GEO, SCI
FROM   dbo.MarkReports
PIVOT (
    MAX(Marks)
    FOR Subject IN (ECO, HIS, MAT, GEO, SCI)
) AS PivotTable;
```

---

## 10. Topper per Subject

> Select the highest scorer in each subject.

**Expected output:**

| Marks | Subject | Name    |
|------:|---------|---------|
|    77 | ECO     | GEORGE  |
|    99 | HIS     | GEORGE  |
|    84 | MAT     | ROBERT  |
|    95 | GEO     | ROBERT  |
|    98 | SCI     | GEORGE  |

**Approach A — Self-join with MAX:**

```sql
SELECT M.Name, M.[Subject], T.Score
FROM   dbo.MarkReports M
JOIN (
    SELECT   [Subject], MAX(Marks) AS Score
    FROM     dbo.MarkReports
    GROUP BY [Subject]
) T ON T.[Subject] = M.[Subject]
   AND T.Score     = M.Marks;
```

**Approach B — CTE with DENSE_RANK:**

```sql
WITH Ranked AS (
    SELECT Name, [Subject], Marks,
           DENSE_RANK() OVER (PARTITION BY [Subject] ORDER BY Marks DESC) AS Rnk
    FROM   dbo.MarkReports
)
SELECT Marks, [Subject], Name
FROM   Ranked
WHERE  Rnk = 1;
```

---

## 11. JOIN Row Counts with Duplicate Keys

> Both Table1 and Table2 have three rows, all with Col1 = 1. How many rows does each JOIN return?

**Table1:**

| Col1 |
|-----:|
|    1 |
|    1 |
|    1 |

**Table2:**

| Col1 |
|-----:|
|    1 |
|    1 |
|    1 |

| Query | Rows |
|-------|-----:|
| `SELECT * FROM Table1 INNER JOIN Table2 ON Table1.Col1 = Table2.Col1` | 9 |
| `SELECT * FROM Table1 LEFT OUTER JOIN Table2 ON Table1.Col1 = Table2.Col1` | 9 |
| `SELECT * FROM Table1 RIGHT OUTER JOIN Table2 ON Table1.Col1 = Table2.Col1` | 9 |

---

## 12. Three-Table JOIN Row Counts

> Tab1 (Col1 = 1,1,1), Tab2 (Col1 = 2,2,2), Tab3 (Col1 = 3,3,3) — no overlapping values.

| Query | Rows |
|-------|-----:|
| `INNER JOIN Tab2 ... LEFT OUTER JOIN Tab3 ...` | 0 |
| `LEFT OUTER JOIN Tab2 ... LEFT OUTER JOIN Tab3 ...` | 3 |
| `FULL OUTER JOIN Tab2 ... LEFT OUTER JOIN Tab3 ...` | 6 |

---

## 13. Year with Maximum Hires

> Find the year with the most new employees and the count.

```sql
SELECT   TO_CHAR(Emp.HireDate, 'YYYY') AS Year,
         COUNT(Emp.EmployeeID)          AS TotalEmployees
FROM     Employee Emp
GROUP BY TO_CHAR(Emp.HireDate, 'YYYY')
HAVING   COUNT(Emp.EmployeeID) = (
             SELECT MAX(COUNT(Emp2.EmployeeID))
             FROM   Employee Emp2
             GROUP BY TO_CHAR(Emp2.HireDate, 'YYYY')
         );
```

---

## 14. Top N per Department (RANK)

> Select the top 3 earners within each department.

```sql
SELECT Employee.*
FROM (
    SELECT Employee.*,
           RANK() OVER (PARTITION BY Employee.DeptID
                        ORDER BY Employee.Salary DESC) AS Rnk
    FROM   Employee
) Employee
WHERE Rnk <= 3;

-- For top N per department, replace 3 with N.
```

---

## 15. Employees Above Department Average

> Find employees earning more than their department's average salary.

This is a **correlated subquery** — the inner query runs once **per row** in the outer query:

1. For each employee (`OuterEmployee`), the subquery calculates the average salary of **that employee's department** (`WHERE OuterEmployee.DeptID = InnerEmployee.DeptID`).
2. The outer `WHERE` keeps only employees whose salary exceeds their own department's average.

**Example walkthrough** (DeptID = 10, avg salary = 4000):

| Name | Salary | Dept Avg | Kept? |
|------|-------:|---------:|-------|
| Alice | 5000 | 4000 | Yes (5000 > 4000) |
| Bob | 3000 | 4000 | No (3000 ≤ 4000) |

**Approach A — Correlated subquery (O(n²)):**

```sql
SELECT OuterEmployee.*
FROM   Employee OuterEmployee
WHERE  OuterEmployee.Salary > (
           SELECT AVG(InnerEmployee.Salary)
           FROM   Employee InnerEmployee
           WHERE  OuterEmployee.DeptID = InnerEmployee.DeptID
       );
```

**Approach B — Window function (single pass):**

```sql
SELECT *
FROM (
    SELECT Employee.*,
           AVG(Salary) OVER (PARTITION BY DeptID) AS DeptAvg
    FROM   Employee
) E
WHERE Salary > DeptAvg;
```

---

## 16. Period of Service (Years & Months)

```sql
SELECT Employee.*,
       TO_CHAR(TRUNC(MONTHS_BETWEEN(SYSDATE, Employee.HireDate) / 12))          AS Years,
       TO_CHAR(TRUNC(MOD(MONTHS_BETWEEN(SYSDATE, Employee.HireDate), 12)))      AS Months
FROM   Employee;
```

---

## 17. Even / Odd Row Selection (Oracle)

**Even rows:**

```sql
SELECT Employee.*
FROM   Employee
WHERE  (ROWID, 0) IN (SELECT ROWID, MOD(ROWNUM, 2) FROM Employee);
```

**Odd rows:**

```sql
SELECT Employee.*
FROM   Employee
WHERE  (ROWID, 1) IN (SELECT ROWID, MOD(ROWNUM, 2) FROM Employee);
```

---

## 18. Clone Table Structure

> Create an empty copy of an existing table.

```sql
SELECT * INTO StudentCopy FROM Student WHERE 1 = 0;
```

---

## 19. First / Last N Characters

```sql
-- First 5 characters
SELECT SUBSTRING(StudentName, 1, 5) AS StudentName FROM Student;

-- Last 5 characters
SELECT RIGHT(StudentName, 5) AS StudentName FROM Student;
```

---

## 20. Total Salary by Department (HAVING)

> Sum salaries per department where more than 2 employees exist.

```sql
SELECT   DeptNo, SUM(Salary) AS TotalSalary
FROM     Employee
GROUP BY DeptNo
HAVING   COUNT(EmpNo) > 2;
```

---

## 21. Companies with < 5 Employees

**Tables:**

```
Company  (CompanyID, CompanyName, City)
Employee (FirstName, LastName, CompanyID)
```

> Include companies with zero employees.

```sql
SELECT   C.CompanyName,
         COUNT(E.CompanyID) AS EmployeeCount
FROM     Company C
LEFT OUTER JOIN Employee E ON C.CompanyID = E.CompanyID
GROUP BY C.CompanyID, C.CompanyName
HAVING   COUNT(E.CompanyID) < 5;
```

---

## 22. Recursive / Hierarchical Queries (CTE)

> Display hierarchical levels for each employee with their manager's name.

**Schema:**

```sql
CREATE TABLE OrgEmployee (
    Id        INT PRIMARY KEY,
    Name      VARCHAR(50),
    ManagerId INT NULL REFERENCES OrgEmployee(Id)
);

INSERT INTO OrgEmployee (Id, Name, ManagerId) VALUES
(1, 'Dayanand',  NULL),
(2, 'Rahul',     1),
(3, 'Pankaj',    1),
(4, 'Shivendra', 3),
(5, 'Suyog',     1),
(6, 'Mandar',    4),
(7, 'Nitin',     3),
(8, 'Pragnesh',  5),
(9, 'Nilesh',    7);
```

**Expected output:**

| Id | Name      | ManagerId | Manager   | Level |
|---:|-----------|----------:|-----------|------:|
|  1 | Dayanand  |           | BOSS      |     1 |
|  2 | Rahul     |         1 | Dayanand  |     2 |
|  3 | Pankaj    |         1 | Dayanand  |     2 |
|  5 | Suyog     |         1 | Dayanand  |     2 |
|  4 | Shivendra |         3 | Pankaj    |     3 |
|  7 | Nitin     |         3 | Pankaj    |     3 |
|  8 | Pragnesh  |         5 | Suyog     |     3 |
|  6 | Mandar    |         4 | Shivendra |     4 |
|  9 | Nilesh    |         7 | Nitin     |     4 |

**Solution:**

```sql
WITH EmployeeHierarchy AS (
    -- Anchor: the boss (no manager)
    SELECT Id, Name, ManagerId,
           'BOSS' AS Manager,
           1      AS [Level]
    FROM   OrgEmployee
    WHERE  ManagerId IS NULL

    UNION ALL

    -- Recursive: employees with a manager
    SELECT EMP.Id, EMP.Name, EMP.ManagerId,
           EH.Name        AS Manager,
           EH.[Level] + 1
    FROM   OrgEmployee EMP
    INNER JOIN EmployeeHierarchy EH ON EH.Id = EMP.ManagerId
    WHERE  EMP.ManagerId IS NOT NULL
)
SELECT * FROM EmployeeHierarchy;
```

---

## 23. Window Functions (SUM OVER PARTITION BY) — Sales Percentage

> Calculate each order's percentage of the day's total sales.

**Schema:**

```sql
CREATE TABLE StoreSales (
    OrderDate    DATETIME,
    SalesOrderId INT,
    TotalDue     INT
);

INSERT INTO StoreSales (OrderDate, SalesOrderId, TotalDue) VALUES
('2022-01-01', 101, 1000),
('2022-01-01', 102, 2000),
('2022-01-01', 103, 3000),
('2022-01-01', 104, 4000),
('2022-01-02', 106, 6000),
('2022-01-02', 107, 7000),
('2022-01-02', 108, 8000),
('2022-01-03', 105, 5000);
```

**Example:** Total for 2022-01-01 = 10,000. Order 102 = 2,000 / 10,000 x 100 = **20%**.

**Solution:**

```sql
SELECT OrderDate,
       SalesOrderId,
       TotalDue,
       SUM(TotalDue) OVER (PARTITION BY OrderDate)                          AS SalesTotal,
       (TotalDue * 100) / SUM(TotalDue) OVER (PARTITION BY OrderDate)       AS PercentTotal
FROM   StoreSales
ORDER BY OrderDate, SalesOrderId;
```

**Output:**

| OrderDate  | SalesOrderId | TotalDue | SalesTotal | PercentTotal |
|------------|-------------:|---------:|-----------:|-------------:|
| 2022-01-01 |          101 |     1000 |      10000 |           10 |
| 2022-01-01 |          102 |     2000 |      10000 |           20 |
| 2022-01-01 |          103 |     3000 |      10000 |           30 |
| 2022-01-01 |          104 |     4000 |      10000 |           40 |
| 2022-01-02 |          106 |     6000 |      21000 |           28 |
| 2022-01-02 |          107 |     7000 |      21000 |           33 |
| 2022-01-02 |          108 |     8000 |      21000 |           38 |
| 2022-01-03 |          105 |     5000 |       5000 |          100 |
