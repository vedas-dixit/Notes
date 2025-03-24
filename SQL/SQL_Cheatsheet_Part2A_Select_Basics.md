# FAANG-Ready SQL Cheat Sheet - Part 2A: SELECT Basics

## SELECT Statement Fundamentals

### Basic SELECT

```sql
-- Select all columns
SELECT * FROM table_name;

-- Select specific columns
SELECT column1, column2 FROM table_name;

-- Select with alias
SELECT column1 AS alias1, column2 AS alias2 FROM table_name;

-- Select with calculation
SELECT column1, column2, column1 + column2 AS sum FROM table_name;

-- Select distinct values
SELECT DISTINCT column1 FROM table_name;

-- Select with limit
SELECT column1, column2 FROM table_name LIMIT n;

-- Select with offset
SELECT column1, column2 FROM table_name LIMIT n OFFSET m;

-- Select with pagination
SELECT column1, column2 FROM table_name LIMIT page_size OFFSET (page_number - 1) * page_size;
```

**Examples:**
```sql
-- Get all employee data
SELECT * FROM employees;

-- Get only name and email
SELECT first_name, last_name, email FROM employees;

-- Use column aliases for better readability
SELECT 
    first_name AS first, 
    last_name AS last, 
    first_name || ' ' || last_name AS full_name
FROM employees;

-- Calculate annual salary
SELECT 
    employee_id,
    first_name,
    last_name,
    salary AS monthly_salary,
    salary * 12 AS annual_salary
FROM employees;

-- Get distinct departments
SELECT DISTINCT department_id FROM employees;

-- Get first 10 employees
SELECT * FROM employees LIMIT 10;

-- Get employees 11-20
SELECT * FROM employees LIMIT 10 OFFSET 10;

-- Page 3 with 15 records per page
SELECT * FROM employees LIMIT 15 OFFSET (3-1) * 15;
```

### WHERE Clause

```sql
-- Basic where clause
SELECT column1, column2 FROM table_name WHERE condition;

-- Comparison operators
SELECT * FROM table_name WHERE column1 = value;
SELECT * FROM table_name WHERE column1 > value;
SELECT * FROM table_name WHERE column1 < value;
SELECT * FROM table_name WHERE column1 >= value;
SELECT * FROM table_name WHERE column1 <= value;
SELECT * FROM table_name WHERE column1 <> value; -- not equal
SELECT * FROM table_name WHERE column1 != value; -- not equal

-- Logical operators
SELECT * FROM table_name WHERE condition1 AND condition2;
SELECT * FROM table_name WHERE condition1 OR condition2;
SELECT * FROM table_name WHERE NOT condition;

-- BETWEEN operator
SELECT * FROM table_name WHERE column1 BETWEEN value1 AND value2;

-- IN operator
SELECT * FROM table_name WHERE column1 IN (value1, value2, ...);
SELECT * FROM table_name WHERE column1 IN (SELECT column1 FROM other_table);

-- LIKE operator (pattern matching)
SELECT * FROM table_name WHERE column1 LIKE 'pattern';
-- % matches zero or more characters
-- _ matches exactly one character

-- IS NULL / IS NOT NULL
SELECT * FROM table_name WHERE column1 IS NULL;
SELECT * FROM table_name WHERE column1 IS NOT NULL;

-- Combines multiple conditions
SELECT * FROM table_name 
WHERE (condition1 OR condition2) AND condition3;
```

**Examples:**
```sql
-- Equal condition
SELECT * FROM employees WHERE department_id = 3;

-- Greater than
SELECT * FROM employees WHERE salary > 50000;

-- Multiple conditions with AND
SELECT * FROM employees 
WHERE department_id = 3 AND salary > 50000;

-- Multiple conditions with OR
SELECT * FROM employees 
WHERE department_id = 3 OR department_id = 4;

-- Between range
SELECT * FROM employees 
WHERE salary BETWEEN 50000 AND 75000;

-- IN list
SELECT * FROM employees 
WHERE department_id IN (1, 3, 5);

-- IN subquery
SELECT * FROM employees 
WHERE department_id IN (
    SELECT department_id 
    FROM departments 
    WHERE location_id = 1700
);

-- LIKE pattern
SELECT * FROM employees WHERE last_name LIKE 'S%'; -- Starts with S
SELECT * FROM employees WHERE email LIKE '%@gmail.com'; -- Ends with @gmail.com
SELECT * FROM employees WHERE first_name LIKE '_a%'; -- Second letter is 'a'

-- NULL check
SELECT * FROM employees WHERE manager_id IS NULL;

-- Complex condition
SELECT * FROM employees 
WHERE (department_id = 3 OR department_id = 4)
AND salary > 50000 
AND manager_id IS NOT NULL;
```

### ORDER BY Clause

```sql
-- Basic order by
SELECT column1, column2 FROM table_name ORDER BY column1;

-- Ascending order (default)
SELECT column1, column2 FROM table_name ORDER BY column1 ASC;

-- Descending order
SELECT column1, column2 FROM table_name ORDER BY column1 DESC;

-- Multiple columns
SELECT column1, column2 FROM table_name ORDER BY column1 ASC, column2 DESC;

-- Order by column position
SELECT column1, column2 FROM table_name ORDER BY 1 ASC, 2 DESC;

-- Order by alias
SELECT column1 AS c1, column2 AS c2 FROM table_name ORDER BY c1;

-- Order by expression
SELECT column1, column2 FROM table_name ORDER BY column1 + column2;

-- NULLS FIRST/LAST (PostgreSQL)
SELECT column1, column2 FROM table_name ORDER BY column1 NULLS FIRST;
SELECT column1, column2 FROM table_name ORDER BY column1 NULLS LAST;

-- Case-insensitive ordering
SELECT column1, column2 FROM table_name ORDER BY LOWER(column1);
```

**Examples:**
```sql
-- Order by last name
SELECT * FROM employees ORDER BY last_name;

-- Highest salary first
SELECT * FROM employees ORDER BY salary DESC;

-- Order by department then salary (highest)
SELECT * FROM employees
ORDER BY department_id ASC, salary DESC;

-- Order by column position
SELECT first_name, last_name, salary FROM employees
ORDER BY 3 DESC, 2 ASC;

-- Order by calculated field
SELECT employee_id, first_name, last_name, salary, commission_pct,
       salary * (1 + COALESCE(commission_pct, 0)) AS total_compensation
FROM employees
ORDER BY total_compensation DESC;

-- NULL managers last
SELECT * FROM employees ORDER BY manager_id NULLS LAST;

-- Case-insensitive ordering
SELECT * FROM employees ORDER BY LOWER(last_name);
```

### FETCH / OFFSET (SQL Standard)

```sql
-- Fetch first n rows only
SELECT column1, column2 FROM table_name
FETCH FIRST n ROWS ONLY;

-- Fetch with offset
SELECT column1, column2 FROM table_name
OFFSET n ROWS
FETCH NEXT m ROWS ONLY;

-- Fetch with percentage
SELECT column1, column2 FROM table_name
FETCH FIRST n PERCENT ROWS ONLY;

-- Fetch with ties
SELECT column1, column2 FROM table_name
ORDER BY column1
FETCH FIRST n ROWS WITH TIES;
```

**Examples:**
```sql
-- Get first 5 rows 
SELECT * FROM employees 
FETCH FIRST 5 ROWS ONLY;

-- Skip 10 rows, get next 5
SELECT * FROM employees 
OFFSET 10 ROWS
FETCH NEXT 5 ROWS ONLY;

-- Get first 10% of total rows
SELECT * FROM employees
FETCH FIRST 10 PERCENT ROWS ONLY;

-- Get top earners (including ties)
SELECT employee_id, first_name, last_name, salary
FROM employees
ORDER BY salary DESC
FETCH FIRST 3 ROWS WITH TIES;
```

### TOP (SQL Server)

```sql
-- Select top n rows
SELECT TOP n column1, column2 FROM table_name;

-- Top with percent
SELECT TOP n PERCENT column1, column2 FROM table_name;

-- Top with ties
SELECT TOP n WITH TIES column1, column2 
FROM table_name
ORDER BY column1;
```

**Examples:**
```sql
-- Get top 5 employees
SELECT TOP 5 * FROM employees;

-- Get top 10% of employees
SELECT TOP 10 PERCENT * FROM employees;

-- Get top 3 salaries including ties
SELECT TOP 3 WITH TIES employee_id, first_name, last_name, salary
FROM employees
ORDER BY salary DESC;
```

## Data Filtering and Processing

### GROUP BY Clause

```sql
-- Basic group by
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1;

-- Multiple columns
SELECT column1, column2, aggregate_function(column3)
FROM table_name
GROUP BY column1, column2;

-- Group by with order by
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1
ORDER BY aggregate_function(column2) DESC;

-- Group by position
SELECT column1, column2, COUNT(*)
FROM table_name
GROUP BY 1, 2;

-- Group by with filter
SELECT column1, aggregate_function(column2)
FROM table_name
WHERE condition
GROUP BY column1;
```

**Examples:**
```sql
-- Count employees per department
SELECT department_id, COUNT(*) AS employee_count
FROM employees
GROUP BY department_id;

-- Average salary by department and job
SELECT department_id, job_id, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id, job_id;

-- Highest total salary departments
SELECT department_id, SUM(salary) AS total_salary
FROM employees
GROUP BY department_id
ORDER BY total_salary DESC;

-- Department employee counts for specific locations
SELECT department_id, COUNT(*) AS employee_count
FROM employees
WHERE department_id IN (
    SELECT department_id 
    FROM departments 
    WHERE location_id IN (1700, 1800)
)
GROUP BY department_id;
```

### HAVING Clause

```sql
-- Basic having clause
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1
HAVING condition;

-- Having with complex condition
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1
HAVING condition1 AND condition2;

-- Where and having together
SELECT column1, aggregate_function(column2)
FROM table_name
WHERE condition1
GROUP BY column1
HAVING condition2;
```

**Examples:**
```sql
-- Departments with more than 10 employees
SELECT department_id, COUNT(*) AS employee_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 10;

-- Departments with average salary over 50K
SELECT department_id, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 50000;

-- Active departments with total salary over 500K
SELECT department_id, SUM(salary) AS total_salary
FROM employees
WHERE is_active = TRUE
GROUP BY department_id
HAVING SUM(salary) > 500000
ORDER BY total_salary DESC;

-- Departments where highest salary is at least 2x the average
SELECT department_id, 
       AVG(salary) AS avg_salary,
       MAX(salary) AS max_salary
FROM employees
GROUP BY department_id
HAVING MAX(salary) >= 2 * AVG(salary);
```

### Common SQL Aggregate Functions

```sql
-- COUNT
SELECT COUNT(*) FROM table_name;
SELECT COUNT(column1) FROM table_name;
SELECT COUNT(DISTINCT column1) FROM table_name;

-- SUM
SELECT SUM(column1) FROM table_name;
SELECT SUM(DISTINCT column1) FROM table_name;

-- AVG
SELECT AVG(column1) FROM table_name;
SELECT AVG(DISTINCT column1) FROM table_name;

-- MIN/MAX
SELECT MIN(column1), MAX(column1) FROM table_name;

-- Statistical functions
SELECT STDEV(column1) FROM table_name; -- Standard deviation
SELECT VAR(column1) FROM table_name; -- Variance

-- String aggregation (varies by database)
-- PostgreSQL
SELECT string_agg(column1, ',') FROM table_name;
-- MySQL
SELECT GROUP_CONCAT(column1 SEPARATOR ',') FROM table_name;
-- SQL Server
SELECT STRING_AGG(column1, ',') FROM table_name;
```

**Examples:**
```sql
-- Basic counts
SELECT COUNT(*) AS total_employees FROM employees;
SELECT COUNT(manager_id) AS employees_with_managers FROM employees;
SELECT COUNT(DISTINCT department_id) AS unique_departments FROM employees;

-- Salary statistics
SELECT 
    SUM(salary) AS total_salary,
    AVG(salary) AS average_salary,
    MIN(salary) AS lowest_salary,
    MAX(salary) AS highest_salary,
    STDEV(salary) AS salary_stdev,
    VAR(salary) AS salary_variance
FROM employees;

-- String aggregation: employee list by department
-- PostgreSQL
SELECT department_id, 
       string_agg(first_name || ' ' || last_name, ', ' ORDER BY last_name) AS employees
FROM employees
GROUP BY department_id;

-- MySQL
SELECT department_id, 
       GROUP_CONCAT(CONCAT(first_name, ' ', last_name) ORDER BY last_name SEPARATOR ', ') AS employees
FROM employees
GROUP BY department_id;

-- SQL Server
SELECT department_id, 
       STRING_AGG(CONCAT(first_name, ' ', last_name), ', ') WITHIN GROUP (ORDER BY last_name) AS employees
FROM employees
GROUP BY department_id;
```

### Common SQL Scalar Functions

```sql
-- String functions
SELECT UPPER(column1) FROM table_name; -- Uppercase
SELECT LOWER(column1) FROM table_name; -- Lowercase
SELECT LENGTH(column1) FROM table_name; -- String length
SELECT SUBSTR(column1, start, length) FROM table_name; -- Substring
SELECT TRIM(column1) FROM table_name; -- Remove whitespace
SELECT REPLACE(column1, search, replace) FROM table_name; -- Replace
SELECT CONCAT(column1, column2) FROM table_name; -- Concatenate

-- Numeric functions
SELECT ROUND(column1, decimal_places) FROM table_name; -- Round
SELECT CEIL(column1) FROM table_name; -- Ceiling
SELECT FLOOR(column1) FROM table_name; -- Floor
SELECT ABS(column1) FROM table_name; -- Absolute value
SELECT POWER(column1, power) FROM table_name; -- Power
SELECT MOD(column1, divisor) FROM table_name; -- Modulo
SELECT SQRT(column1) FROM table_name; -- Square root

-- Date functions
SELECT CURRENT_DATE FROM table_name; -- Current date
SELECT CURRENT_TIMESTAMP FROM table_name; -- Current date and time
SELECT EXTRACT(part FROM column1) FROM table_name; -- Extract part (year, month, day)
SELECT DATE_TRUNC('part', column1) FROM table_name; -- Truncate to specified part
SELECT column1 + INTERVAL '1 day' FROM table_name; -- Date arithmetic
SELECT DATEDIFF(column1, column2) FROM table_name; -- Date difference

-- Conditional functions
SELECT CASE WHEN condition THEN result1 ELSE result2 END FROM table_name;
SELECT COALESCE(column1, column2, default_value) FROM table_name; -- First non-null
SELECT NULLIF(column1, column2) FROM table_name; -- NULL if equal
```

**Examples:**
```sql
-- String manipulations
SELECT 
    employee_id,
    UPPER(first_name) AS first_upper,
    LOWER(last_name) AS last_lower,
    LENGTH(email) AS email_length,
    SUBSTR(last_name, 1, 3) AS last_name_short,
    TRIM(phone_number) AS phone_clean,
    REPLACE(email, '@example.com', '@company.com') AS new_email,
    CONCAT(first_name, ' ', last_name) AS full_name
FROM employees;

-- Numeric calculations
SELECT 
    salary,
    ROUND(salary, -3) AS rounded_thousands,
    CEIL(salary / 1000) * 1000 AS ceiling_thousands,
    FLOOR(salary / 1000) * 1000 AS floor_thousands,
    ABS(salary - 50000) AS salary_gap,
    POWER(salary, 0.5) AS square_root_salary,
    MOD(employee_id, 10) AS employee_id_mod,
    SQRT(salary) AS salary_sqrt
FROM employees;

-- Date manipulations
SELECT 
    hire_date,
    CURRENT_DATE AS today,
    EXTRACT(YEAR FROM hire_date) AS hire_year,
    EXTRACT(MONTH FROM hire_date) AS hire_month,
    DATE_TRUNC('month', hire_date) AS hire_month_start,
    hire_date + INTERVAL '90 days' AS probation_end,
    DATEDIFF(CURRENT_DATE, hire_date) AS days_employed
FROM employees;

-- Conditional logic
SELECT 
    employee_id,
    salary,
    CASE 
        WHEN salary < 30000 THEN 'Low'
        WHEN salary BETWEEN 30000 AND 70000 THEN 'Medium'
        WHEN salary > 70000 THEN 'High'
        ELSE 'Unknown'
    END AS salary_bracket,
    COALESCE(commission_pct, 0) AS commission,
    NULLIF(department_id, 10) AS dept_if_not_admin
FROM employees;
```

### Selecting Random Records

```sql
-- MySQL
SELECT column1, column2 FROM table_name
ORDER BY RAND() LIMIT n;

-- PostgreSQL
SELECT column1, column2 FROM table_name
ORDER BY RANDOM() LIMIT n;

-- SQL Server
SELECT TOP n column1, column2 FROM table_name
ORDER BY NEWID();

-- Oracle
SELECT column1, column2 FROM table_name
ORDER BY DBMS_RANDOM.VALUE FETCH FIRST n ROWS ONLY;
```

**Examples:**
```sql
-- MySQL: Get 5 random employees
SELECT * FROM employees
ORDER BY RAND() LIMIT 5;

-- PostgreSQL: Get 5 random employees
SELECT * FROM employees
ORDER BY RANDOM() LIMIT 5;

-- SQL Server: Get 5 random employees
SELECT TOP 5 * FROM employees
ORDER BY NEWID();

-- Oracle: Get 5 random employees
SELECT * FROM employees
ORDER BY DBMS_RANDOM.VALUE
FETCH FIRST 5 ROWS ONLY;
```

### WITH Clause (Common Table Expressions)

```sql
-- Basic CTE
WITH cte_name AS (
    SELECT column1, column2
    FROM table_name
    WHERE condition
)
SELECT * FROM cte_name;

-- Multiple CTEs
WITH 
cte1 AS (
    SELECT * FROM table1 WHERE condition1
),
cte2 AS (
    SELECT * FROM table2 WHERE condition2
)
SELECT * FROM cte1 JOIN cte2 ON cte1.id = cte2.id;

-- Recursive CTE
WITH RECURSIVE cte_name AS (
    -- Base case
    SELECT column1, column2 FROM table_name WHERE condition
    UNION ALL
    -- Recursive case
    SELECT t.column1, t.column2
    FROM table_name t JOIN cte_name c ON t.parent_id = c.id
)
SELECT * FROM cte_name;
```

**Examples:**
```sql
-- CTE for high salary employees
WITH high_salary_employees AS (
    SELECT * FROM employees
    WHERE salary > 75000
)
SELECT e.*, d.department_name
FROM high_salary_employees e
JOIN departments d ON e.department_id = d.department_id;

-- Multiple CTEs
WITH 
dept_stats AS (
    SELECT department_id, COUNT(*) AS emp_count, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
),
high_avg_depts AS (
    SELECT department_id FROM dept_stats
    WHERE avg_salary > 60000
)
SELECT e.*, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.department_id IN (SELECT department_id FROM high_avg_depts);

-- Recursive CTE to get employee hierarchy
WITH RECURSIVE emp_hierarchy AS (
    -- Base case: employees with no manager (top level)
    SELECT employee_id, first_name, last_name, manager_id, 0 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: employees with managers from the previous level
    SELECT e.employee_id, e.first_name, e.last_name, e.manager_id, h.level + 1
    FROM employees e
    JOIN emp_hierarchy h ON e.manager_id = h.employee_id
)
SELECT 
    employee_id,
    CONCAT(REPEAT('    ', level), first_name, ' ', last_name) AS employee_hierarchy
FROM emp_hierarchy
ORDER BY level, first_name, last_name;
``` 