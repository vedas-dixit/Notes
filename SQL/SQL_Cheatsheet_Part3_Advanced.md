# FAANG-Ready SQL Cheat Sheet - Part 3: Advanced Features

## Transaction Control Language (TCL)

Transactions allow you to group multiple SQL operations into a single unit of work that either completes entirely or not at all.

### COMMIT

```sql
-- Begin transaction (some databases)
BEGIN TRANSACTION;

-- Multiple SQL operations
INSERT INTO table1 (column1) VALUES (value1);
UPDATE table2 SET column1 = value1 WHERE condition;
DELETE FROM table3 WHERE condition;

-- Commit transaction
COMMIT;
```

**Example:**
```sql
-- Transfer money between accounts
BEGIN TRANSACTION;

UPDATE accounts 
SET balance = balance - 1000 
WHERE account_id = 1001;

UPDATE accounts 
SET balance = balance + 1000 
WHERE account_id = 1002;

-- Verify both operations succeeded
IF @@ROWCOUNT = 1 -- SQL Server syntax
    COMMIT;
ELSE
    ROLLBACK;
```

### ROLLBACK

```sql
-- Begin transaction
BEGIN TRANSACTION;

-- SQL operations that encounter errors
INSERT INTO table1 (column1) VALUES (value1);
-- If error occurs

-- Rollback transaction
ROLLBACK;
```

**Example:**
```sql
-- Try to transfer money but handle insufficient funds
BEGIN TRANSACTION;

-- Check balance first
DECLARE @balance DECIMAL(10,2);
SELECT @balance = balance FROM accounts WHERE account_id = 1001;

-- Only proceed if sufficient funds
IF @balance >= 1000
BEGIN
    UPDATE accounts SET balance = balance - 1000 WHERE account_id = 1001;
    UPDATE accounts SET balance = balance + 1000 WHERE account_id = 1002;
    COMMIT;
END
ELSE
BEGIN
    ROLLBACK;
    PRINT 'Insufficient funds';
END
```

### SAVEPOINT

```sql
-- Begin transaction
BEGIN TRANSACTION;

-- First operation
INSERT INTO table1 (column1) VALUES (value1);

-- Create savepoint
SAVEPOINT save1;

-- Second operation
UPDATE table2 SET column1 = value1 WHERE condition;

-- Rollback to savepoint (keeping first operation)
ROLLBACK TO save1;

-- Commit transaction (only the first operation)
COMMIT;
```

**Example:**
```sql
-- Complex transaction with savepoints
BEGIN TRANSACTION;

INSERT INTO orders (customer_id, order_date) 
VALUES (101, CURRENT_DATE);

-- Store the new order_id
DECLARE @order_id INT = SCOPE_IDENTITY();

SAVEPOINT order_created;

-- Try to add order items
INSERT INTO order_items (order_id, product_id, quantity) 
VALUES (@order_id, 1, 5);

-- Check if product is in stock
IF (SELECT stock_quantity FROM products WHERE product_id = 1) < 5
BEGIN
    ROLLBACK TO order_created;
    
    -- Instead, add a different product
    INSERT INTO order_items (order_id, product_id, quantity) 
    VALUES (@order_id, 2, 5);
END

COMMIT;
```

## Data Control Language (DCL)

### GRANT

```sql
-- Grant privileges on table
GRANT privilege1, privilege2 ON table_name TO user_name;

-- Grant all privileges
GRANT ALL PRIVILEGES ON table_name TO user_name;

-- Grant with grant option (user can grant privileges to others)
GRANT privilege1 ON table_name TO user_name WITH GRANT OPTION;

-- Grant role
GRANT role_name TO user_name;
```

**Examples:**
```sql
-- Grant specific privileges
GRANT SELECT, INSERT, UPDATE ON employees TO hr_staff;

-- Grant all privileges
GRANT ALL PRIVILEGES ON employees TO hr_manager;

-- Grant select on specific columns
GRANT SELECT (employee_id, first_name, last_name) ON employees TO hr_intern;

-- Grant access to all tables in schema
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analyst;

-- Grant role
GRANT hr_role TO new_employee;
```

### REVOKE

```sql
-- Revoke privileges
REVOKE privilege1, privilege2 ON table_name FROM user_name;

-- Revoke all privileges
REVOKE ALL PRIVILEGES ON table_name FROM user_name;

-- Revoke grant option
REVOKE GRANT OPTION FOR privilege1 ON table_name FROM user_name;

-- Revoke role
REVOKE role_name FROM user_name;
```

**Examples:**
```sql
-- Revoke specific privileges
REVOKE UPDATE, DELETE ON employees FROM hr_staff;

-- Revoke all privileges
REVOKE ALL PRIVILEGES ON salary_data FROM analyst;

-- Revoke role
REVOKE hr_role FROM former_employee;
```

## Advanced SQL Features

### Window Functions

Window functions perform calculations across a set of rows related to the current row.

```sql
-- Basic window function
SELECT
    column1,
    column2,
    aggregate_function(column3) OVER (
        PARTITION BY column4
        ORDER BY column5
        ROWS BETWEEN frame_start AND frame_end
    ) AS window_result
FROM table_name;

-- Common window functions:
-- ROW_NUMBER(), RANK(), DENSE_RANK(), NTILE()
-- LEAD(), LAG()
-- FIRST_VALUE(), LAST_VALUE()
-- SUM(), AVG(), MIN(), MAX(), COUNT()
```

**Examples:**
```sql
-- Row number within department
SELECT
    employee_id,
    first_name,
    last_name,
    department_id,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department_id
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;

-- Calculate running total of salary by department
SELECT
    employee_id,
    department_id,
    salary,
    SUM(salary) OVER (
        PARTITION BY department_id
        ORDER BY employee_id
        ROWS UNBOUNDED PRECEDING
    ) AS running_total
FROM employees;

-- Compare salary with next/previous employee in same department
SELECT
    employee_id,
    department_id,
    salary,
    LAG(salary) OVER (
        PARTITION BY department_id
        ORDER BY employee_id
    ) AS previous_employee_salary,
    LEAD(salary) OVER (
        PARTITION BY department_id
        ORDER BY employee_id
    ) AS next_employee_salary
FROM employees;

-- Calculate percentile within department
SELECT
    employee_id,
    department_id,
    salary,
    NTILE(4) OVER (
        PARTITION BY department_id
        ORDER BY salary
    ) AS salary_quartile
FROM employees;

-- Calculate moving average of salary for 3 consecutive employees
SELECT
    employee_id,
    department_id,
    salary,
    AVG(salary) OVER (
        PARTITION BY department_id
        ORDER BY employee_id
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) AS moving_avg_salary
FROM employees;
```

### Analytical Functions

```sql
-- ROLLUP (hierarchical subtotals)
SELECT column1, column2, SUM(column3)
FROM table_name
GROUP BY ROLLUP(column1, column2);

-- CUBE (all possible combinations of subtotals)
SELECT column1, column2, SUM(column3)
FROM table_name
GROUP BY CUBE(column1, column2);

-- GROUPING SETS (specific combinations)
SELECT column1, column2, SUM(column3)
FROM table_name
GROUP BY GROUPING SETS((column1, column2), (column1), (column2), ());
```

**Examples:**
```sql
-- ROLLUP example
SELECT
    COALESCE(department_id::TEXT, 'All Departments') AS department,
    COALESCE(job_id::TEXT, 'All Jobs') AS job,
    SUM(salary) AS total_salary
FROM employees
GROUP BY ROLLUP(department_id, job_id);

-- CUBE example
SELECT
    COALESCE(department_id::TEXT, 'All Departments') AS department,
    COALESCE(EXTRACT(YEAR FROM hire_date)::TEXT, 'All Years') AS hire_year,
    COUNT(*) AS employee_count
FROM employees
GROUP BY CUBE(department_id, EXTRACT(YEAR FROM hire_date));
```

### JSON Functions

```sql
-- PostgreSQL JSON functions
SELECT
    column1,
    json_column->>'property' AS json_property,
    json_column->'nested'->>'property' AS nested_property,
    jsonb_array_elements(json_column->'array') AS array_element,
    jsonb_object_keys(json_column) AS json_keys
FROM table_name;

-- MySQL JSON functions
SELECT
    column1,
    JSON_EXTRACT(json_column, '$.property') AS json_property,
    JSON_EXTRACT(json_column, '$.nested.property') AS nested_property,
    JSON_CONTAINS(json_column, '"value"', '$.property') AS contains_value
FROM table_name;
```

**Examples:**
```sql
-- PostgreSQL: Extract user preferences
SELECT
    user_id,
    preferences->>'theme' AS user_theme,
    preferences->>'language' AS user_language,
    preferences->'notifications'->>'email' AS email_notifications
FROM user_settings;

-- MySQL: Find users with specific preferences
SELECT
    user_id,
    JSON_EXTRACT(preferences, '$.theme') AS theme
FROM user_settings
WHERE JSON_CONTAINS(preferences, '"dark"', '$.theme');
```

### Common Table Expressions (CTEs)

```sql
-- Basic CTE
WITH cte_name AS (
    SELECT column1, column2
    FROM table_name
    WHERE condition
)
SELECT * FROM cte_name;

-- Recursive CTE
WITH RECURSIVE cte_name AS (
    -- Base case
    SELECT column1, column2
    FROM table_name
    WHERE condition

    UNION ALL

    -- Recursive case
    SELECT t.column1, t.column2
    FROM table_name t
    JOIN cte_name c ON t.parent_id = c.id
)
SELECT * FROM cte_name;
```

**Examples:**
```sql
-- Find employees who earn more than their managers
WITH emp_manager_salary AS (
    SELECT
        e.employee_id,
        e.first_name,
        e.last_name,
        e.salary,
        e.manager_id,
        m.salary AS manager_salary
    FROM employees e
    LEFT JOIN employees m ON e.manager_id = m.employee_id
)
SELECT
    employee_id,
    first_name,
    last_name,
    salary,
    manager_salary
FROM emp_manager_salary
WHERE salary > manager_salary OR manager_salary IS NULL;

-- Find the organization hierarchy using recursive CTE
WITH RECURSIVE org_hierarchy AS (
    -- Base case: CEO (no manager)
    SELECT
        employee_id,
        first_name,
        last_name,
        manager_id,
        0 AS level,
        first_name || ' ' || last_name AS path
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case: employees with managers
    SELECT
        e.employee_id,
        e.first_name,
        e.last_name,
        e.manager_id,
        h.level + 1,
        h.path || ' > ' || e.first_name || ' ' || e.last_name
    FROM employees e
    JOIN org_hierarchy h ON e.manager_id = h.employee_id
)
SELECT
    employee_id,
    first_name,
    last_name,
    level,
    path
FROM org_hierarchy
ORDER BY path;
```

## SQL Optimization Techniques

### Indexing Strategies

```sql
-- Create proper indexes
CREATE INDEX idx_name ON table_name(column1, column2);

-- Covering index (includes all columns needed by the query)
CREATE INDEX idx_covering ON table_name(column1, column2) INCLUDE (column3, column4);

-- Index for range queries (order matters)
CREATE INDEX idx_range ON table_name(equality_column, range_column);

-- Partial index
CREATE INDEX idx_partial ON table_name(column1) WHERE condition;
```

**Best Practices:**
- Index columns used in WHERE, JOIN, ORDER BY, and GROUP BY clauses
- Put equality filter columns before range filter columns in composite indexes
- Avoid indexing columns with low cardinality (few distinct values)
- Use covering indexes for high-performance queries
- Consider adding filtered/partial indexes for queries that target a subset of data
- Regularly review and maintain indexes based on query patterns

### Query Optimization

```sql
-- Avoid SELECT *
SELECT specific_columns FROM table_name;

-- Use appropriate JOINs
SELECT t1.column1, t2.column2
FROM table1 t1
INNER JOIN table2 t2 ON t1.id = t2.id;

-- Filter early
SELECT column1, column2
FROM table_name
WHERE high_selectivity_column = value;

-- Use EXISTS instead of IN for large subqueries
SELECT column1
FROM table1 t1
WHERE EXISTS (SELECT 1 FROM table2 t2 WHERE t2.id = t1.id);

-- Use EXPLAIN/EXPLAIN ANALYZE to review query plans
EXPLAIN SELECT * FROM table_name WHERE condition;
```

**Best Practices:**
- Select only the columns you need
- Filter rows early in query processing
- Use appropriate join types based on the data
- Prefer EXISTS over IN for large subqueries
- Use UNION ALL instead of UNION when duplicates are acceptable
- Break down complex queries into simpler ones using CTEs
- Use appropriate data types for columns
- Avoid functions on indexed columns in WHERE clauses

### Database Design Patterns

#### Normalization

- **First Normal Form (1NF):** Eliminate duplicate columns, create separate tables for related data
- **Second Normal Form (2NF):** Meet 1NF requirements, move columns dependent on part of the primary key to separate tables
- **Third Normal Form (3NF):** Meet 2NF requirements, eliminate columns not dependent on the primary key

#### Denormalization

- Combine tables to reduce joins
- Add redundant data to improve read performance
- Use materialized views for frequent complex queries
- Consider denormalization when read performance is critical

**Example:**
```sql
-- Normalization: Separate orders and customers
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(100),
    address TEXT
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id),
    order_date DATE,
    total_amount DECIMAL(10,2)
);

-- Denormalization: Include customer data in orders report view
CREATE MATERIALIZED VIEW orders_report AS
SELECT
    o.order_id,
    o.order_date,
    o.total_amount,
    c.customer_id,
    c.customer_name,
    c.email
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

## Database Specific Features

### PostgreSQL Specific

```sql
-- Array type
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    tags TEXT[]
);

INSERT INTO events (name, tags) VALUES ('Conference', ARRAY['tech', 'networking', 'career']);

-- Query array
SELECT * FROM events WHERE 'tech' = ANY(tags);
SELECT * FROM events WHERE tags @> ARRAY['tech', 'career'];

-- JSON/JSONB type
CREATE TABLE config (
    id SERIAL PRIMARY KEY,
    data JSONB
);

INSERT INTO config (data) VALUES ('{"theme": "dark", "notifications": {"email": true, "sms": false}}');

-- Query JSON
SELECT * FROM config WHERE data->>'theme' = 'dark';
SELECT * FROM config WHERE data @> '{"notifications": {"email": true}}';

-- Full-text search
CREATE INDEX idx_fts ON articles USING gin(to_tsvector('english', title || ' ' || content));

SELECT * FROM articles
WHERE to_tsvector('english', title || ' ' || content) @@ to_tsquery('english', 'postgresql & database');
```

### MySQL Specific

```sql
-- Full-text search
CREATE FULLTEXT INDEX idx_fulltext ON articles(title, content);

SELECT * FROM articles
WHERE MATCH(title, content) AGAINST('database optimization' IN BOOLEAN MODE);

-- GROUP_CONCAT
SELECT 
    department_id,
    GROUP_CONCAT(first_name ORDER BY first_name SEPARATOR ', ') AS employees
FROM employees
GROUP BY department_id;

-- On Duplicate Key Update
INSERT INTO metrics (date, count)
VALUES (CURRENT_DATE, 1)
ON DUPLICATE KEY UPDATE count = count + 1;
```

### SQL Server Specific

```sql
-- Table variables
DECLARE @TempTable TABLE (
    id INT,
    name VARCHAR(100)
);

INSERT INTO @TempTable
SELECT employee_id, first_name FROM employees WHERE department_id = 3;

-- OUTPUT clause
INSERT INTO archive_employees
OUTPUT inserted.employee_id, inserted.termination_date INTO termination_log
SELECT * FROM employees
WHERE status = 'Terminated';

-- TOP with TIES
SELECT TOP 5 WITH TIES *
FROM employees
ORDER BY salary DESC;
```

### Oracle Specific

```sql
-- ROWNUM for pagination
SELECT *
FROM (
    SELECT a.*, ROWNUM rnum
    FROM (
        SELECT * FROM employees ORDER BY employee_id
    ) a
    WHERE ROWNUM <= 20
)
WHERE rnum >= 11;

-- Connect By (hierarchical queries)
SELECT
    employee_id,
    first_name,
    last_name,
    LEVEL as depth,
    CONNECT_BY_ROOT first_name as top_manager,
    SYS_CONNECT_BY_PATH(first_name, '/') as path
FROM employees
START WITH manager_id IS NULL
CONNECT BY PRIOR employee_id = manager_id;
```

## Interview Tips: SQL Optimization

1. **Always analyze the query plan**
   - Use EXPLAIN / EXPLAIN ANALYZE to identify inefficient operations
   - Look for sequential scans on large tables
   - Watch for expensive operations (hash joins, sorts)

2. **Index strategy**
   - Identify columns in WHERE, JOIN, GROUP BY, ORDER BY clauses
   - Consider column order in composite indexes
   - Be aware of when indexes won't be used (functions on columns)

3. **Join optimization**
   - Use the right join types (INNER, LEFT, RIGHT)
   - Join order matters in some databases
   - Filter tables before joining when possible

4. **Rewrite queries**
   - Replace subqueries with joins when appropriate
   - Use EXISTS instead of IN for large result sets
   - Break down complex queries into CTEs

5. **Handle large result sets**
   - Implement pagination
   - Only select needed columns
   - Filter data as early as possible in the query

6. **Common anti-patterns to avoid**
   - SELECT *
   - Functions in WHERE clauses on indexed columns
   - Correlated subqueries that run for each row
   - Cursors when set-based operations would work
   - Implicit conversions between data types 