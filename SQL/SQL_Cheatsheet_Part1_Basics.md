# FAANG-Ready SQL Cheat Sheet - Part 1: Fundamentals

## SQL Command Categories

SQL commands fall into several categories:

1. **DDL (Data Definition Language)** - Create, alter, and delete database objects
2. **DML (Data Manipulation Language)** - Insert, update, and delete data
3. **DQL (Data Query Language)** - Query and retrieve data
4. **DCL (Data Control Language)** - Control access to database
5. **TCL (Transaction Control Language)** - Control transactions

## DDL Commands

### CREATE

#### CREATE DATABASE

```sql
-- Basic syntax
CREATE DATABASE database_name;

-- With character set and collation
CREATE DATABASE database_name
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

-- Only if not exists
CREATE DATABASE IF NOT EXISTS database_name;
```

**Example:**
```sql
CREATE DATABASE employees_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

#### CREATE TABLE

```sql
-- Basic syntax
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    ...
);

-- With primary key
CREATE TABLE table_name (
    id INT PRIMARY KEY AUTO_INCREMENT,
    column1 datatype constraints,
    ...
);

-- With foreign key
CREATE TABLE table_name (
    id INT PRIMARY KEY,
    foreign_id INT,
    FOREIGN KEY (foreign_id) REFERENCES other_table(id)
);

-- With CHECK constraint
CREATE TABLE table_name (
    id INT PRIMARY KEY,
    age INT CHECK (age >= 18),
    ...
);

-- With DEFAULT values
CREATE TABLE table_name (
    id INT PRIMARY KEY,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'active',
    ...
);

-- Temporary table
CREATE TEMPORARY TABLE table_name (
    column1 datatype,
    column2 datatype,
    ...
);

-- Create table from another table
CREATE TABLE new_table AS 
SELECT column1, column2
FROM existing_table
WHERE condition;
```

**Comprehensive Example:**
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone_number VARCHAR(20),
    hire_date DATE NOT NULL,
    job_id INT NOT NULL,
    salary DECIMAL(10, 2) CHECK (salary > 0),
    commission_pct DECIMAL(4, 2),
    manager_id INT,
    department_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (job_id) REFERENCES jobs(job_id),
    FOREIGN KEY (department_id) REFERENCES departments(department_id),
    FOREIGN KEY (manager_id) REFERENCES employees(employee_id),
    INDEX idx_last_name (last_name),
    INDEX idx_department_id (department_id)
);
```

#### CREATE INDEX

```sql
-- Basic index
CREATE INDEX index_name
ON table_name (column1, column2, ...);

-- Unique index
CREATE UNIQUE INDEX index_name
ON table_name (column1, column2, ...);

-- Composite index
CREATE INDEX index_name
ON table_name (column1, column2);

-- Partial index (PostgreSQL)
CREATE INDEX index_name
ON table_name (column1)
WHERE condition;

-- Functional index (PostgreSQL)
CREATE INDEX index_name
ON table_name (LOWER(column1));

-- Descending index
CREATE INDEX index_name
ON table_name (column1 DESC);

-- Index with INCLUDE (SQL Server)
CREATE INDEX index_name
ON table_name (column1)
INCLUDE (column2, column3);
```

**Examples:**
```sql
-- Basic index for faster lookup by last_name
CREATE INDEX idx_employees_last_name
ON employees (last_name);

-- Composite index for queries filtering by department and job
CREATE INDEX idx_dept_job
ON employees (department_id, job_id);

-- Unique index for enforcing uniqueness
CREATE UNIQUE INDEX idx_unique_email
ON employees (email);

-- Functional index for case-insensitive searches
CREATE INDEX idx_last_name_lower
ON employees (LOWER(last_name));

-- Partial index only for active employees
CREATE INDEX idx_active_employees
ON employees (employee_id, last_name)
WHERE is_active = TRUE;
```

#### CREATE VIEW

```sql
-- Basic view
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition;

-- With check option (prevents updates that violate the view's condition)
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition
WITH CHECK OPTION;

-- Updatable view
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name;

-- Materialized view (PostgreSQL)
CREATE MATERIALIZED VIEW view_name AS
SELECT column1, column2, ...
FROM table_name;
```

**Examples:**
```sql
-- View for employee details with department name
CREATE VIEW employee_details AS
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    e.email,
    e.salary,
    d.department_name,
    j.job_title
FROM 
    employees e
JOIN 
    departments d ON e.department_id = d.department_id
JOIN 
    jobs j ON e.job_id = j.job_id;

-- Department salary stats view
CREATE VIEW department_salary_stats AS
SELECT 
    d.department_id,
    d.department_name,
    COUNT(e.employee_id) AS employee_count,
    ROUND(AVG(e.salary), 2) AS avg_salary,
    MIN(e.salary) AS min_salary,
    MAX(e.salary) AS max_salary
FROM 
    departments d
LEFT JOIN 
    employees e ON d.department_id = e.department_id
GROUP BY 
    d.department_id, d.department_name;

-- View with check option
CREATE VIEW high_salary_employees AS
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE salary > 100000
WITH CHECK OPTION;
```

### ALTER

#### ALTER TABLE

```sql
-- Add column
ALTER TABLE table_name
ADD column_name datatype constraints;

-- Multiple columns
ALTER TABLE table_name
ADD column1 datatype constraints,
ADD column2 datatype constraints;

-- Drop column
ALTER TABLE table_name
DROP COLUMN column_name;

-- Modify column type
ALTER TABLE table_name
MODIFY column_name new_datatype new_constraints;

-- Rename column
ALTER TABLE table_name
RENAME COLUMN old_name TO new_name;

-- Add primary key
ALTER TABLE table_name
ADD PRIMARY KEY (column_name);

-- Add foreign key
ALTER TABLE table_name
ADD CONSTRAINT constraint_name
FOREIGN KEY (column_name)
REFERENCES referenced_table(referenced_column);

-- Drop foreign key
ALTER TABLE table_name
DROP FOREIGN KEY constraint_name;

-- Add constraint
ALTER TABLE table_name
ADD CONSTRAINT constraint_name CHECK (condition);

-- Drop constraint
ALTER TABLE table_name
DROP CONSTRAINT constraint_name;

-- Rename table
ALTER TABLE old_table_name
RENAME TO new_table_name;
```

**Examples:**
```sql
-- Add a new column for employee address
ALTER TABLE employees
ADD address VARCHAR(255);

-- Add multiple columns
ALTER TABLE employees
ADD emergency_contact VARCHAR(100),
ADD emergency_phone VARCHAR(20);

-- Modify salary column to allow higher values
ALTER TABLE employees
MODIFY salary DECIMAL(12, 2);

-- Rename column
ALTER TABLE employees
RENAME COLUMN phone_number TO contact_number;

-- Add index for performance
ALTER TABLE employees
ADD INDEX idx_hire_date (hire_date);

-- Add check constraint for email format
ALTER TABLE employees
ADD CONSTRAINT chk_email_format CHECK (email LIKE '%@%.%');

-- Add foreign key with specific delete behavior
ALTER TABLE employees
ADD CONSTRAINT fk_department
FOREIGN KEY (department_id)
REFERENCES departments(department_id)
ON DELETE CASCADE;
```

#### ALTER DATABASE

```sql
-- Rename database (MySQL)
ALTER DATABASE old_name
RENAME TO new_name;

-- Change character set
ALTER DATABASE database_name
CHARACTER SET character_set_name;

-- Change collation
ALTER DATABASE database_name
COLLATE collation_name;
```

**Example:**
```sql
ALTER DATABASE employees_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

### DROP

```sql
-- Drop table
DROP TABLE table_name;

-- Drop only if exists
DROP TABLE IF EXISTS table_name;

-- Drop multiple tables
DROP TABLE table1, table2, table3;

-- Drop view
DROP VIEW view_name;

-- Drop index
DROP INDEX index_name ON table_name;

-- Drop database
DROP DATABASE database_name;

-- Drop only if exists
DROP DATABASE IF EXISTS database_name;
```

**Examples:**
```sql
-- Drop a temporary table
DROP TABLE IF EXISTS temp_employee_salary;

-- Drop multiple tables
DROP TABLE employee_archive, department_archive;

-- Drop view
DROP VIEW IF EXISTS employee_details;

-- Drop an index
DROP INDEX idx_last_name ON employees;
```

### TRUNCATE

```sql
-- Remove all rows from a table (faster than DELETE)
TRUNCATE TABLE table_name;

-- Multiple tables (SQL Server)
TRUNCATE TABLE table1, table2, table3;
```

**Example:**
```sql
-- Clear all records from a log table
TRUNCATE TABLE access_logs;
```

## DML Commands

### INSERT

```sql
-- Basic insert
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);

-- Multiple rows
INSERT INTO table_name (column1, column2, ...)
VALUES 
    (value1, value2, ...),
    (value1, value2, ...),
    (value1, value2, ...);

-- Insert from query
INSERT INTO table_name (column1, column2, ...)
SELECT column1, column2, ...
FROM source_table
WHERE condition;

-- Insert all columns
INSERT INTO table_name
VALUES (value1, value2, ...);

-- Insert with DEFAULT values
INSERT INTO table_name (column1, column2)
VALUES (value1, DEFAULT);

-- Insert returning (PostgreSQL)
INSERT INTO table_name (column1, column2)
VALUES (value1, value2)
RETURNING column1, column2;

-- Insert ignore (MySQL)
INSERT IGNORE INTO table_name (column1, column2)
VALUES (value1, value2);

-- Insert on duplicate key (MySQL)
INSERT INTO table_name (column1, column2)
VALUES (value1, value2)
ON DUPLICATE KEY UPDATE
    column1 = value1,
    column2 = value2;

-- Upsert (PostgreSQL)
INSERT INTO table_name (column1, column2)
VALUES (value1, value2)
ON CONFLICT (column1)
DO UPDATE SET column2 = EXCLUDED.column2;
```

**Examples:**
```sql
-- Simple insert
INSERT INTO employees (first_name, last_name, email, hire_date, job_id, department_id)
VALUES ('John', 'Doe', 'john.doe@example.com', '2023-01-15', 1, 2);

-- Multiple rows
INSERT INTO departments (department_name, location_id)
VALUES 
    ('Marketing', 1700),
    ('Finance', 1700),
    ('IT', 1400),
    ('Executive', 1700);

-- Insert from query (copy employees to archive)
INSERT INTO employee_archive (employee_id, first_name, last_name, email, hire_date, job_id, department_id)
SELECT employee_id, first_name, last_name, email, hire_date, job_id, department_id
FROM employees
WHERE hire_date < '2020-01-01';

-- Insert with on duplicate key update
INSERT INTO employee_skills (employee_id, skill_id, proficiency_level)
VALUES (101, 3, 'Advanced')
ON DUPLICATE KEY UPDATE
    proficiency_level = 'Advanced';

-- PostgreSQL upsert
INSERT INTO employee_contact (employee_id, phone_number, emergency_contact)
VALUES (101, '555-123-4567', 'Jane Doe')
ON CONFLICT (employee_id)
DO UPDATE SET 
    phone_number = EXCLUDED.phone_number,
    emergency_contact = EXCLUDED.emergency_contact;
```

### UPDATE

```sql
-- Basic update
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;

-- Update with subquery
UPDATE table_name
SET column1 = (SELECT column_x FROM other_table WHERE condition)
WHERE condition;

-- Update with case
UPDATE table_name
SET column1 = CASE
    WHEN condition1 THEN value1
    WHEN condition2 THEN value2
    ELSE value3
END
WHERE condition;

-- Update join (MySQL)
UPDATE table1 t1
JOIN table2 t2 ON t1.id = t2.id
SET t1.column1 = t2.column2
WHERE condition;

-- Update with limit (MySQL)
UPDATE table_name
SET column1 = value1
WHERE condition
LIMIT n;

-- Update returning (PostgreSQL)
UPDATE table_name
SET column1 = value1
WHERE condition
RETURNING column1, column2;
```

**Examples:**
```sql
-- Simple update
UPDATE employees
SET salary = 75000
WHERE employee_id = 101;

-- Update with calculation
UPDATE employees
SET salary = salary * 1.1
WHERE department_id = 3;

-- Update with subquery
UPDATE employees
SET manager_id = (SELECT employee_id FROM employees WHERE email = 'boss@example.com')
WHERE department_id = 5;

-- Update with CASE
UPDATE employees
SET 
    salary = CASE
        WHEN salary < 50000 THEN salary * 1.1
        WHEN salary BETWEEN 50000 AND 100000 THEN salary * 1.07
        ELSE salary * 1.05
    END
WHERE is_active = TRUE;

-- Update with JOIN
UPDATE employees e
JOIN performance_reviews p ON e.employee_id = p.employee_id
SET e.salary = e.salary * (1 + (p.rating * 0.05))
WHERE p.review_year = 2023;

-- PostgreSQL update with returning
UPDATE employees
SET last_login_date = CURRENT_TIMESTAMP
WHERE employee_id = 101
RETURNING employee_id, last_login_date;
```

### DELETE

```sql
-- Basic delete
DELETE FROM table_name
WHERE condition;

-- Delete all rows
DELETE FROM table_name;

-- Delete with subquery
DELETE FROM table_name
WHERE column_name IN (SELECT column_name FROM other_table WHERE condition);

-- Delete with join (MySQL)
DELETE t1
FROM table1 t1
JOIN table2 t2 ON t1.id = t2.id
WHERE condition;

-- Delete with limit (MySQL)
DELETE FROM table_name
WHERE condition
LIMIT n;

-- Delete returning (PostgreSQL)
DELETE FROM table_name
WHERE condition
RETURNING column1, column2;
```

**Examples:**
```sql
-- Delete a specific employee
DELETE FROM employees
WHERE employee_id = 101;

-- Delete employees who left the company
DELETE FROM employees
WHERE termination_date IS NOT NULL;

-- Delete with subquery
DELETE FROM employee_projects
WHERE project_id IN (SELECT project_id FROM projects WHERE status = 'Cancelled');

-- Delete with join
DELETE e
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE d.department_name = 'Temporary' AND e.hire_date < '2023-01-01';

-- PostgreSQL delete with returning
DELETE FROM access_logs
WHERE created_at < CURRENT_DATE - INTERVAL '1 year'
RETURNING id, user_id, created_at;
```

### MERGE (UPSERT)

```sql
-- SQL Server, Oracle syntax
MERGE INTO target_table t
USING source_table s
ON (t.key_column = s.key_column)
WHEN MATCHED THEN
    UPDATE SET t.column1 = s.column1, t.column2 = s.column2
WHEN NOT MATCHED THEN
    INSERT (column1, column2) VALUES (s.column1, s.column2);

-- MySQL syntax (INSERT ... ON DUPLICATE KEY UPDATE)
INSERT INTO target_table (key_column, column1, column2)
SELECT key_column, column1, column2 FROM source_table
ON DUPLICATE KEY UPDATE
    column1 = VALUES(column1),
    column2 = VALUES(column2);

-- PostgreSQL syntax
INSERT INTO target_table (key_column, column1, column2)
SELECT key_column, column1, column2 FROM source_table
ON CONFLICT (key_column)
DO UPDATE SET
    column1 = EXCLUDED.column1,
    column2 = EXCLUDED.column2;
```

**Examples:**
```sql
-- SQL Server MERGE
MERGE INTO employee_details t
USING (
    SELECT employee_id, first_name, last_name, email, phone_number
    FROM new_employee_data
) s
ON (t.employee_id = s.employee_id)
WHEN MATCHED THEN
    UPDATE SET 
        t.first_name = s.first_name,
        t.last_name = s.last_name,
        t.email = s.email,
        t.phone_number = s.phone_number
WHEN NOT MATCHED THEN
    INSERT (employee_id, first_name, last_name, email, phone_number)
    VALUES (s.employee_id, s.first_name, s.last_name, s.email, s.phone_number);

-- MySQL upsert
INSERT INTO salary_history (employee_id, year, month, salary)
SELECT employee_id, YEAR(CURRENT_DATE), MONTH(CURRENT_DATE), salary 
FROM employees
ON DUPLICATE KEY UPDATE
    salary = VALUES(salary);

-- PostgreSQL upsert
INSERT INTO employee_skills (employee_id, skill_id, proficiency_level)
SELECT e.employee_id, s.skill_id, 'Beginner'
FROM employees e, skills s
WHERE e.department_id = 3 AND s.skill_name = 'Python'
ON CONFLICT (employee_id, skill_id)
DO UPDATE SET
    proficiency_level = 
        CASE
            WHEN employee_skills.proficiency_level = 'Beginner' THEN 'Intermediate'
            WHEN employee_skills.proficiency_level = 'Intermediate' THEN 'Advanced'
            ELSE employee_skills.proficiency_level
        END;
``` 