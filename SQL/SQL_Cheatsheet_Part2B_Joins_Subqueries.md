# FAANG-Ready SQL Cheat Sheet - Part 2B: JOINs and Subqueries

## JOIN Operations

### INNER JOIN

```sql
-- Basic inner join
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1
INNER JOIN table2 t2 ON t1.id = t2.id;

-- Inner join with multiple conditions
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1
INNER JOIN table2 t2 
    ON t1.id = t2.id AND t1.column2 = t2.column2;

-- Inner join with USING (if column names are the same)
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1
INNER JOIN table2 t2 USING (id);

-- Multiple inner joins
SELECT t1.column1, t2.column2, t3.column3
FROM table1 t1
INNER JOIN table2 t2 ON t1.id = t2.id
INNER JOIN table3 t3 ON t2.id = t3.id;

-- Inner join with WHERE clause
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1
INNER JOIN table2 t2 ON t1.id = t2.id
WHERE t1.column3 > value;
```

**Examples:**
```sql
-- Employees with their department names
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name
FROM 
    employees e
INNER JOIN 
    departments d ON e.department_id = d.department_id;

-- Employees with their department and location
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name,
    l.city,
    l.country_id
FROM 
    employees e
INNER JOIN 
    departments d ON e.department_id = d.department_id
INNER JOIN 
    locations l ON d.location_id = l.location_id;

-- Orders with customer and product details
SELECT 
    o.order_id,
    o.order_date,
    c.customer_name,
    p.product_name,
    oi.quantity,
    oi.unit_price,
    (oi.quantity * oi.unit_price) AS line_total
FROM 
    orders o
INNER JOIN 
    customers c ON o.customer_id = c.customer_id
INNER JOIN 
    order_items oi ON o.order_id = oi.order_id
INNER JOIN 
    products p ON oi.product_id = p.product_id
WHERE 
    o.order_date >= '2023-01-01';
```

### LEFT JOIN (LEFT OUTER JOIN)

```sql
-- Basic left join
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1
LEFT JOIN table2 t2 ON t1.id = t2.id;

-- Left join with where clause
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1
LEFT JOIN table2 t2 ON t1.id = t2.id
WHERE t1.column3 = value;

-- Finding rows that exist only in the left table
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1
LEFT JOIN table2 t2 ON t1.id = t2.id
WHERE t2.id IS NULL;

-- Multiple left joins
SELECT t1.column1, t2.column2, t3.column3
FROM table1 t1
LEFT JOIN table2 t2 ON t1.id = t2.id
LEFT JOIN table3 t3 ON t1.id = t3.id;
```

**Examples:**
```sql
-- All departments with employees (including departments with no employees)
SELECT 
    d.department_id,
    d.department_name,
    e.employee_id,
    e.first_name,
    e.last_name
FROM 
    departments d
LEFT JOIN 
    employees e ON d.department_id = e.department_id
ORDER BY 
    d.department_id, e.employee_id;

-- Find departments with no employees
SELECT 
    d.department_id,
    d.department_name
FROM 
    departments d
LEFT JOIN 
    employees e ON d.department_id = e.department_id
WHERE 
    e.employee_id IS NULL;

-- All employees with their manager's name (if they have one)
SELECT 
    e1.employee_id,
    e1.first_name || ' ' || e1.last_name AS employee_name,
    e2.first_name || ' ' || e2.last_name AS manager_name
FROM 
    employees e1
LEFT JOIN 
    employees e2 ON e1.manager_id = e2.employee_id
ORDER BY 
    e1.employee_id;
```

### RIGHT JOIN (RIGHT OUTER JOIN)

```sql
-- Basic right join
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1
RIGHT JOIN table2 t2 ON t1.id = t2.id;

-- Finding rows that exist only in the right table
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1
RIGHT JOIN table2 t2 ON t1.id = t2.id
WHERE t1.id IS NULL;

-- Multiple right joins
SELECT t1.column1, t2.column2, t3.column3
FROM table1 t1
RIGHT JOIN table2 t2 ON t1.id = t2.id
RIGHT JOIN table3 t3 ON t2.id = t3.id;
```

**Examples:**
```sql
-- All employees with their department (even if department doesn't exist in departments table)
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name
FROM 
    departments d
RIGHT JOIN 
    employees e ON d.department_id = e.department_id
ORDER BY 
    e.employee_id;

-- Find employees with departments not in the departments table
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    e.department_id
FROM 
    departments d
RIGHT JOIN 
    employees e ON d.department_id = e.department_id
WHERE 
    d.department_id IS NULL;
```

### FULL JOIN (FULL OUTER JOIN)

```sql
-- Basic full join
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1
FULL JOIN table2 t2 ON t1.id = t2.id;

-- Finding rows that exist in only one table
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1
FULL JOIN table2 t2 ON t1.id = t2.id
WHERE t1.id IS NULL OR t2.id IS NULL;
```

**Examples:**
```sql
-- All departments and all employees (regardless of relationship)
SELECT 
    d.department_id,
    d.department_name,
    e.employee_id,
    e.first_name,
    e.last_name
FROM 
    departments d
FULL JOIN 
    employees e ON d.department_id = e.department_id
ORDER BY 
    d.department_id, e.employee_id;

-- Find departments with no employees and employees with no departments
SELECT 
    d.department_id,
    d.department_name,
    e.employee_id,
    e.first_name,
    e.last_name
FROM 
    departments d
FULL JOIN 
    employees e ON d.department_id = e.department_id
WHERE 
    d.department_id IS NULL OR e.employee_id IS NULL
ORDER BY 
    d.department_id, e.employee_id;
```

### CROSS JOIN (Cartesian Product)

```sql
-- Explicit cross join syntax
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1
CROSS JOIN table2 t2;

-- Implicit cross join syntax
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1, table2 t2;

-- Cross join with where condition (like an inner join)
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1
CROSS JOIN table2 t2
WHERE t1.id = t2.id;
```

**Examples:**
```sql
-- Generate all possible department-job combinations
SELECT 
    d.department_id,
    d.department_name,
    j.job_id,
    j.job_title
FROM 
    departments d
CROSS JOIN 
    jobs j
ORDER BY 
    d.department_id, j.job_id;

-- Generate date ranges (using a dates table)
SELECT 
    d1.date AS start_date,
    d2.date AS end_date
FROM 
    dates d1
CROSS JOIN 
    dates d2
WHERE 
    d1.date < d2.date
ORDER BY 
    d1.date, d2.date;
```

### SELF JOIN

```sql
-- Self join (joining a table to itself)
SELECT t1.column1, t1.column2, t2.column2
FROM table_name t1
JOIN table_name t2 ON t1.related_id = t2.id;
```

**Examples:**
```sql
-- Employee and their manager
SELECT 
    e.employee_id,
    e.first_name || ' ' || e.last_name AS employee_name,
    m.employee_id AS manager_id,
    m.first_name || ' ' || m.last_name AS manager_name
FROM 
    employees e
JOIN 
    employees m ON e.manager_id = m.employee_id;

-- Find employees who work in the same department
SELECT 
    e1.employee_id,
    e1.first_name || ' ' || e1.last_name AS employee_name,
    e2.employee_id AS colleague_id,
    e2.first_name || ' ' || e2.last_name AS colleague_name
FROM 
    employees e1
JOIN 
    employees e2 ON e1.department_id = e2.department_id
WHERE 
    e1.employee_id < e2.employee_id -- Avoid duplicates
ORDER BY 
    e1.employee_id, e2.employee_id;
```

### NATURAL JOIN

```sql
-- Natural join (joins based on all columns with same name)
SELECT column1, column2
FROM table1
NATURAL JOIN table2;
```

**Example:**
```sql
-- Natural join between employees and departments (if both have department_id)
SELECT 
    employee_id,
    first_name,
    last_name,
    department_id,
    department_name
FROM 
    employees
NATURAL JOIN 
    departments;
```

### Alternative JOIN Syntax

```sql
-- Old-style join syntax (using WHERE for join condition)
SELECT t1.column1, t1.column2, t2.column1
FROM table1 t1, table2 t2
WHERE t1.id = t2.id;

-- Multi-table join with old syntax
SELECT t1.column1, t2.column2, t3.column3
FROM table1 t1, table2 t2, table3 t3
WHERE t1.id = t2.id AND t2.id = t3.id;
```

**Example:**
```sql
-- Old-style join syntax
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    d.department_name
FROM 
    employees e, departments d
WHERE 
    e.department_id = d.department_id
    AND e.salary > 50000;
```

## Subqueries

### Scalar Subqueries (Return a Single Value)

```sql
-- Subquery in SELECT clause
SELECT column1, (SELECT MAX(column2) FROM table2) AS max_value
FROM table1;

-- Subquery in WHERE clause with comparison
SELECT column1, column2
FROM table1
WHERE column1 > (SELECT AVG(column1) FROM table1);

-- Subquery in HAVING clause
SELECT column1, AVG(column2)
FROM table1
GROUP BY column1
HAVING AVG(column2) > (SELECT AVG(column2) FROM table1);
```

**Examples:**
```sql
-- Get employees with salary higher than average
SELECT 
    employee_id,
    first_name,
    last_name,
    salary
FROM 
    employees
WHERE 
    salary > (SELECT AVG(salary) FROM employees);

-- Display each department's average salary and deviation from company average
SELECT 
    d.department_name,
    AVG(e.salary) AS dept_avg_salary,
    AVG(e.salary) - (SELECT AVG(salary) FROM employees) AS diff_from_avg
FROM 
    employees e
JOIN 
    departments d ON e.department_id = d.department_id
GROUP BY 
    d.department_name
ORDER BY 
    diff_from_avg DESC;

-- Find the department with the highest average salary
SELECT 
    department_id,
    AVG(salary) AS avg_salary
FROM 
    employees
GROUP BY 
    department_id
HAVING 
    AVG(salary) = (
        SELECT MAX(dept_avg.avg_salary)
        FROM (
            SELECT department_id, AVG(salary) AS avg_salary
            FROM employees
            GROUP BY department_id
        ) dept_avg
    );
```

### Row Subqueries (Return a Single Row)

```sql
-- Comparison with row subquery
SELECT column1, column2
FROM table1
WHERE (column1, column2) = (SELECT column1, column2 FROM table2 WHERE condition);
```

**Examples:**
```sql
-- Find employee with same department and job as a specific employee
SELECT 
    employee_id,
    first_name,
    last_name,
    department_id,
    job_id
FROM 
    employees
WHERE 
    (department_id, job_id) = (
        SELECT department_id, job_id 
        FROM employees 
        WHERE employee_id = 101
    )
    AND employee_id != 101;

-- Get employee with highest salary in each department
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    e.department_id,
    e.salary
FROM 
    employees e
WHERE 
    (e.department_id, e.salary) IN (
        SELECT department_id, MAX(salary)
        FROM employees
        GROUP BY department_id
    );
```

### Table Subqueries (Return Multiple Rows)

```sql
-- Subquery in FROM clause
SELECT t.column1, t.column2
FROM (SELECT column1, column2 FROM table1 WHERE condition) t;

-- Subquery with IN operator
SELECT column1, column2
FROM table1
WHERE column1 IN (SELECT column1 FROM table2 WHERE condition);

-- Subquery with NOT IN operator
SELECT column1, column2
FROM table1
WHERE column1 NOT IN (SELECT column1 FROM table2 WHERE condition);

-- Subquery with ANY/SOME operator
SELECT column1, column2
FROM table1
WHERE column1 > ANY (SELECT column1 FROM table2 WHERE condition);

-- Subquery with ALL operator
SELECT column1, column2
FROM table1
WHERE column1 > ALL (SELECT column1 FROM table2 WHERE condition);

-- Subquery with EXISTS operator
SELECT column1, column2
FROM table1 t1
WHERE EXISTS (SELECT 1 FROM table2 t2 WHERE t2.id = t1.id);

-- Subquery with NOT EXISTS operator
SELECT column1, column2
FROM table1 t1
WHERE NOT EXISTS (SELECT 1 FROM table2 t2 WHERE t2.id = t1.id);
```

**Examples:**
```sql
-- Table subquery in FROM
SELECT 
    dept_salary.department_name,
    dept_salary.avg_salary
FROM (
    SELECT 
        d.department_name,
        AVG(e.salary) AS avg_salary
    FROM 
        employees e
    JOIN 
        departments d ON e.department_id = d.department_id
    GROUP BY 
        d.department_name
) dept_salary
WHERE 
    dept_salary.avg_salary > 50000;

-- IN subquery
SELECT 
    employee_id,
    first_name,
    last_name
FROM 
    employees
WHERE 
    department_id IN (
        SELECT department_id
        FROM departments
        WHERE location_id = 1700
    );

-- NOT IN subquery
SELECT 
    department_id,
    department_name
FROM 
    departments
WHERE 
    department_id NOT IN (
        SELECT DISTINCT department_id
        FROM employees
        WHERE department_id IS NOT NULL
    );

-- ANY/SOME subquery
SELECT 
    employee_id,
    first_name,
    last_name,
    salary
FROM 
    employees
WHERE 
    salary > ANY (
        SELECT salary
        FROM employees
        WHERE department_id = 3
    );

-- ALL subquery
SELECT 
    employee_id,
    first_name,
    last_name,
    salary
FROM 
    employees
WHERE 
    salary > ALL (
        SELECT AVG(salary)
        FROM employees
        GROUP BY department_id
    );

-- EXISTS subquery
SELECT 
    d.department_id,
    d.department_name
FROM 
    departments d
WHERE 
    EXISTS (
        SELECT 1
        FROM employees e
        WHERE e.department_id = d.department_id
    );

-- NOT EXISTS subquery
SELECT 
    d.department_id,
    d.department_name
FROM 
    departments d
WHERE 
    NOT EXISTS (
        SELECT 1
        FROM employees e
        WHERE e.department_id = d.department_id
    );
```

### Correlated Subqueries

```sql
-- Correlated subquery in WHERE
SELECT column1, column2
FROM table1 t1
WHERE column1 > (SELECT AVG(column1) FROM table1 t2 WHERE t2.group_id = t1.group_id);

-- Correlated subquery with EXISTS
SELECT column1, column2
FROM table1 t1
WHERE EXISTS (SELECT 1 FROM table2 t2 WHERE t2.id = t1.id AND condition);
```

**Examples:**
```sql
-- Employees earning more than their department average
SELECT 
    e1.employee_id,
    e1.first_name,
    e1.last_name,
    e1.salary,
    e1.department_id
FROM 
    employees e1
WHERE 
    e1.salary > (
        SELECT AVG(e2.salary)
        FROM employees e2
        WHERE e2.department_id = e1.department_id
    )
ORDER BY 
    e1.department_id, e1.salary;

-- Find departments with at least one employee earning more than 50K
SELECT 
    d.department_id,
    d.department_name
FROM 
    departments d
WHERE 
    EXISTS (
        SELECT 1
        FROM employees e
        WHERE e.department_id = d.department_id
        AND e.salary > 50000
    );

-- Update with correlated subquery
UPDATE employees e
SET e.salary = e.salary * 1.1
WHERE e.performance_rating = 'Excellent'
AND e.salary < (
    SELECT AVG(e2.salary) * 1.2
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

## Advanced JOIN Techniques

### LATERAL Joins (PostgreSQL, SQL Server)

```sql
-- Lateral join with subquery
SELECT t1.column1, t2.column1
FROM table1 t1
CROSS JOIN LATERAL (
    SELECT column1, column2
    FROM table2
    WHERE table2.id = t1.id
    LIMIT n
) t2;
```

**Example:**
```sql
-- Get each department and its top 3 highest paid employees
SELECT 
    d.department_name,
    e.first_name,
    e.last_name,
    e.salary
FROM 
    departments d
CROSS JOIN LATERAL (
    SELECT first_name, last_name, salary
    FROM employees
    WHERE department_id = d.department_id
    ORDER BY salary DESC
    LIMIT 3
) e;
```

### PIVOT and UNPIVOT (SQL Server, Oracle)

```sql
-- SQL Server PIVOT
SELECT column1, [value1], [value2], [value3]
FROM (
    SELECT column1, column2, column3
    FROM table1
) src
PIVOT (
    aggregate_function(column3)
    FOR column2 IN ([value1], [value2], [value3])
) pvt;

-- SQL Server UNPIVOT
SELECT column1, column2, column3
FROM table1
UNPIVOT (
    column3
    FOR column2 IN (column_a, column_b, column_c)
) unpvt;
```

**Example:**
```sql
-- SQL Server PIVOT example: Employee count by department and job
SELECT department_name, [Manager], [Developer], [Analyst], [Assistant]
FROM (
    SELECT d.department_name, j.job_title, e.employee_id
    FROM employees e
    JOIN departments d ON e.department_id = d.department_id
    JOIN jobs j ON e.job_id = j.job_id
) src
PIVOT (
    COUNT(employee_id)
    FOR job_title IN ([Manager], [Developer], [Analyst], [Assistant])
) pvt;
```

### ROLLUP, CUBE, and GROUPING SETS

```sql
-- ROLLUP (hierarchical subtotals)
SELECT column1, column2, SUM(column3)
FROM table1
GROUP BY ROLLUP(column1, column2);

-- CUBE (all possible subtotals)
SELECT column1, column2, SUM(column3)
FROM table1
GROUP BY CUBE(column1, column2);

-- GROUPING SETS (specific subtotal combinations)
SELECT column1, column2, SUM(column3)
FROM table1
GROUP BY GROUPING SETS(
    (column1, column2),
    (column1),
    (column2),
    ()
);
```

**Examples:**
```sql
-- ROLLUP: Department and job salary analysis with subtotals
SELECT 
    COALESCE(d.department_name, 'All Departments') AS department,
    COALESCE(j.job_title, 'All Jobs') AS job,
    COUNT(*) AS employee_count,
    SUM(e.salary) AS total_salary
FROM 
    employees e
JOIN 
    departments d ON e.department_id = d.department_id
JOIN 
    jobs j ON e.job_id = j.job_id
GROUP BY 
    ROLLUP(d.department_name, j.job_title);

-- CUBE: Department and location salary analysis with all combinations
SELECT 
    COALESCE(d.department_name, 'All Departments') AS department,
    COALESCE(l.city, 'All Locations') AS location,
    COUNT(*) AS employee_count,
    SUM(e.salary) AS total_salary
FROM 
    employees e
JOIN 
    departments d ON e.department_id = d.department_id
JOIN 
    locations l ON d.location_id = l.location_id
GROUP BY 
    CUBE(d.department_name, l.city);

-- GROUPING SETS: Specific subtotal combinations
SELECT 
    COALESCE(d.department_name, 'All Departments') AS department,
    COALESCE(j.job_title, 'All Jobs') AS job,
    COALESCE(TO_CHAR(e.hire_date, 'YYYY'), 'All Years') AS hire_year,
    COUNT(*) AS employee_count
FROM 
    employees e
JOIN 
    departments d ON e.department_id = d.department_id
JOIN 
    jobs j ON e.job_id = j.job_id
GROUP BY 
    GROUPING SETS (
        (d.department_name, j.job_title, TO_CHAR(e.hire_date, 'YYYY')),
        (d.department_name, j.job_title),
        (d.department_name, TO_CHAR(e.hire_date, 'YYYY')),
        (j.job_title, TO_CHAR(e.hire_date, 'YYYY')),
        (d.department_name),
        (j.job_title),
        (TO_CHAR(e.hire_date, 'YYYY')),
        ()
    );
``` 