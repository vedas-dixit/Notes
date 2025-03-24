# SQL for Full-Stack Developers - Part 3: Transactions and Advanced Queries

## 1. Transactions and ACID Properties

Transactions are a fundamental concept in database systems that ensure data consistency and integrity, especially in concurrent environments like web applications.

### Understanding Transactions

A transaction is a sequence of operations treated as a single logical unit of work. Either all operations complete successfully (commit), or none of them do (rollback).

```sql
-- Basic transaction structure
BEGIN TRANSACTION;
    -- SQL operations here
COMMIT;
-- or ROLLBACK; in case of error
```

### ACID Properties

ACID is an acronym that describes the key guarantees of database transactions:

1. **Atomicity**: Transactions are all-or-nothing. If any part fails, the entire transaction fails and the database state is left unchanged.

2. **Consistency**: A transaction can only bring the database from one valid state to another, maintaining database invariants.

3. **Isolation**: Concurrent transactions execute as if they were sequential, preventing interference between transactions.

4. **Durability**: Once a transaction is committed, it remains permanent even in the event of system failure.

### Transaction Example

```sql
-- Transaction to transfer money between accounts
BEGIN TRANSACTION;

UPDATE accounts 
SET balance = balance - 100 
WHERE account_id = 123;

UPDATE accounts 
SET balance = balance + 100 
WHERE account_id = 456;

-- Check if any account would go negative
IF EXISTS (SELECT 1 FROM accounts WHERE balance < 0) THEN
    ROLLBACK;
ELSE
    COMMIT;
END IF;
```

### Transaction Isolation Levels

Different isolation levels provide tradeoffs between consistency and performance:

1. **Read Uncommitted**: Lowest isolation level. Allows dirty reads (reading uncommitted changes).

2. **Read Committed**: Prevents dirty reads. Each query sees only committed data as of when the query began.

3. **Repeatable Read**: In addition to read committed guarantees, ensures that if a row is read twice in the same transaction, it will have the same value.

4. **Serializable**: Highest isolation level. Ensures transactions execute as if they were sequential, preventing all concurrency issues.

```sql
-- Setting transaction isolation level (PostgreSQL)
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    -- Operations here
COMMIT;
```

### Common Concurrency Issues

1. **Dirty Reads**: Reading uncommitted changes made by another transaction.

2. **Non-repeatable Reads**: Re-reading a row gives a different value because another transaction modified it.

3. **Phantom Reads**: Re-running a query returns different rows because another transaction added or removed rows.

4. **Lost Updates**: Two transactions read and update the same row, causing one update to be lost.

### Optimistic vs. Pessimistic Locking

1. **Pessimistic Locking**: Locks resources explicitly before accessing them.

```sql
-- Pessimistic locking
BEGIN TRANSACTION;
SELECT * FROM products WHERE id = 101 FOR UPDATE;
-- Now other transactions can't update this row until this transaction ends
UPDATE products SET stock = stock - 1 WHERE id = 101;
COMMIT;
```

2. **Optimistic Locking**: Assumes conflicts are rare and checks at commit time.

```sql
-- Optimistic locking using a version column
BEGIN TRANSACTION;
SELECT id, stock, version FROM products WHERE id = 101;
-- Application code: decrement stock, increment version
UPDATE products 
SET stock = stock - 1, version = version + 1 
WHERE id = 101 AND version = [original_version];
-- If no rows affected, someone else modified it
COMMIT;
```

## 2. Indexes and Query Optimization

Indexes are data structures that improve the speed of data retrieval operations but add overhead for data modification operations.

### Types of Indexes

1. **B-tree (Balanced Tree)**: The default index type in most databases, good for equality and range queries.

2. **Hash**: Optimized for equality comparisons only, not for ranges.

3. **GiST (Generalized Search Tree)**: Useful for full-text search, geospatial data, etc.

4. **GIN (Generalized Inverted Index)**: Good for composite types and full-text search.

### Creating Indexes

```sql
-- Basic index
CREATE INDEX idx_users_email ON users(email);

-- Unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Composite index (multiple columns)
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- Partial index
CREATE INDEX idx_products_low_stock ON products(id, stock) WHERE stock < 10;

-- Expression index
CREATE INDEX idx_users_lower_email ON users(LOWER(email));
```

### When to Use Indexes

1. **Primary keys**: Automatically indexed.
2. **Foreign keys**: Often beneficial to index.
3. **Columns used in WHERE clauses**: Especially with high selectivity.
4. **Columns used in JOIN conditions**: Speed up join operations.
5. **Columns used in ORDER BY or GROUP BY**: Can eliminate sorting operations.

### Query Optimization Techniques

1. **Use EXPLAIN (or EXPLAIN ANALYZE)** to understand query execution plans:

```sql
EXPLAIN ANALYZE
SELECT u.username, COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.username;
```

2. **Apply filters early** to reduce the working set:

```sql
-- Less efficient
SELECT * FROM orders JOIN users ON orders.user_id = users.id
WHERE users.country = 'USA';

-- More efficient
SELECT * FROM orders 
JOIN (SELECT * FROM users WHERE country = 'USA') AS filtered_users
ON orders.user_id = filtered_users.id;
```

3. **Use appropriate JOIN types** and join order:

```sql
-- Joining the smaller table first is often more efficient
SELECT * FROM small_table s
JOIN large_table l ON s.id = l.small_id;
```

4. **Consider LIMIT and OFFSET for pagination**:

```sql
-- Efficient pagination
SELECT * FROM products
ORDER BY id
LIMIT 20 OFFSET 40;
```

5. **Use specific columns instead of ***:

```sql
-- More efficient
SELECT id, name, price FROM products WHERE category_id = 5;

-- Less efficient
SELECT * FROM products WHERE category_id = 5;
```

### Indexing Anti-patterns

1. **Over-indexing**: Too many indexes slow down writes.
2. **Indexing low-cardinality columns**: Columns with few distinct values.
3. **Not accounting for column order in composite indexes.
4. **Redundant indexes**: Having multiple indexes that serve similar purposes.

## 3. Advanced SQL Techniques

### Common Table Expressions (CTEs)

CTEs provide a way to write more readable and maintainable SQL by breaking complex queries into named temporary result sets.

```sql
-- Basic CTE syntax
WITH cte_name AS (
    SELECT column1, column2 FROM table1 WHERE condition
)
SELECT * FROM cte_name;

-- Example: Finding best customers
WITH order_totals AS (
    SELECT 
        user_id,
        COUNT(*) AS order_count,
        SUM(total_amount) AS total_spent
    FROM orders
    WHERE status = 'completed'
    GROUP BY user_id
)
SELECT 
    u.id,
    u.name,
    u.email,
    ot.order_count,
    ot.total_spent
FROM users u
JOIN order_totals ot ON u.id = ot.user_id
ORDER BY ot.total_spent DESC
LIMIT 10;
```

### Recursive CTEs

Recursive CTEs allow you to query hierarchical data like organizational structures or category trees.

```sql
-- Example: Employee hierarchy
WITH RECURSIVE employee_hierarchy AS (
    -- Base case (starting point)
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case
    SELECT 
        e.id,
        e.name,
        e.manager_id,
        eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM employee_hierarchy ORDER BY level, name;
```

### Window Functions

Window functions perform calculations across a set of rows related to the current row, without grouping rows into a single output row.

```sql
-- Basic window function syntax
SELECT column1, aggregate_function() OVER (PARTITION BY column2 ORDER BY column3) FROM table;

-- Example: Adding row numbers
SELECT 
    id,
    name,
    price,
    ROW_NUMBER() OVER (ORDER BY price DESC) AS price_rank
FROM products;

-- Example: Running totals
SELECT 
    order_date,
    total_amount,
    SUM(total_amount) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- Example: Moving averages
SELECT 
    date,
    daily_sales,
    AVG(daily_sales) OVER (
        ORDER BY date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS seven_day_average
FROM daily_sales_report;
```

### Common Window Functions

1. **ROW_NUMBER()**: Assigns unique numbers to rows.
2. **RANK()**: Assigns ranks with gaps for ties.
3. **DENSE_RANK()**: Assigns ranks without gaps for ties.
4. **NTILE(n)**: Divides rows into n buckets.
5. **LEAD(col, n)**: Access a value n rows ahead.
6. **LAG(col, n)**: Access a value n rows behind.
7. **FIRST_VALUE(col)**: First value in the window.
8. **LAST_VALUE(col)**: Last value in the window.

```sql
-- Example: Ranking products by price within categories
SELECT 
    category_id,
    name,
    price,
    ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS row_num,
    RANK() OVER (PARTITION BY category_id ORDER BY price DESC) AS price_rank,
    DENSE_RANK() OVER (PARTITION BY category_id ORDER BY price DESC) AS dense_rank
FROM products;

-- Example: Finding price differences from the previous product
SELECT 
    id,
    name,
    price,
    price - LAG(price, 1, 0) OVER (ORDER BY price) AS price_diff_from_previous
FROM products;
```

### Materialized Views

Materialized views store the result of a query physically and can be refreshed periodically, providing faster access to complex query results.

```sql
-- Creating a materialized view (PostgreSQL)
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    SUM(total_amount) AS total_sales,
    COUNT(*) AS order_count
FROM orders
GROUP BY DATE_TRUNC('month', order_date);

-- Refreshing a materialized view
REFRESH MATERIALIZED VIEW monthly_sales;
```

### JSON Functions

Modern SQL databases provide robust support for JSON data, allowing flexible schema designs.

```sql
-- PostgreSQL JSON operations
-- Creating a table with JSON
CREATE TABLE user_profiles (
    user_id INT PRIMARY KEY,
    profile JSONB
);

-- Inserting JSON data
INSERT INTO user_profiles (user_id, profile)
VALUES (
    1, 
    '{"name": "John", "age": 30, "interests": ["hiking", "programming", "cooking"]}'
);

-- Querying JSON data
SELECT 
    user_id,
    profile->>'name' AS name,
    (profile->>'age')::INT AS age
FROM user_profiles
WHERE profile @> '{"interests": ["programming"]}';

-- Updating JSON data
UPDATE user_profiles
SET profile = profile || '{"verified": true}'::JSONB
WHERE user_id = 1;
```

### Full-Text Search

Full-text search capabilities allow natural language searching beyond simple pattern matching.

```sql
-- PostgreSQL full-text search
CREATE INDEX idx_products_fts ON products 
USING GIN (to_tsvector('english', name || ' ' || description));

SELECT id, name, description
FROM products
WHERE to_tsvector('english', name || ' ' || description) @@ 
      to_tsquery('english', 'modern & design');
```

## 4. Real-World Application Patterns

### Pagination with Complex Sorting

```sql
-- Keyset-based pagination (more efficient than OFFSET)
SELECT id, title, created_at
FROM posts
WHERE (created_at, id) < (?, ?) -- Use values from the last item of previous page
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

### Soft Delete Pattern

```sql
-- Instead of DELETE, mark rows as deleted
UPDATE users
SET deleted_at = CURRENT_TIMESTAMP
WHERE id = 123;

-- Then filter deleted rows in queries
SELECT * FROM users WHERE deleted_at IS NULL;
```

### Audit Trail Implementation

```sql
-- Create audit table
CREATE TABLE audit_logs (
    id SERIAL PRIMARY KEY,
    table_name TEXT NOT NULL,
    record_id INTEGER NOT NULL,
    action TEXT NOT NULL, -- 'INSERT', 'UPDATE', 'DELETE'
    old_data JSONB,
    new_data JSONB,
    changed_by INTEGER REFERENCES users(id),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create trigger function (PostgreSQL)
CREATE OR REPLACE FUNCTION audit_trigger_function()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO audit_logs (table_name, record_id, action, new_data, changed_by)
        VALUES (TG_TABLE_NAME, NEW.id, 'INSERT', row_to_json(NEW), current_setting('app.user_id')::INTEGER);
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_logs (table_name, record_id, action, old_data, new_data, changed_by)
        VALUES (TG_TABLE_NAME, NEW.id, 'UPDATE', row_to_json(OLD), row_to_json(NEW), current_setting('app.user_id')::INTEGER);
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit_logs (table_name, record_id, action, old_data, changed_by)
        VALUES (TG_TABLE_NAME, OLD.id, 'DELETE', row_to_json(OLD), current_setting('app.user_id')::INTEGER);
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Attach trigger to table
CREATE TRIGGER users_audit_trigger
AFTER INSERT OR UPDATE OR DELETE ON users
FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();
```

### Time-Series Data Handling

```sql
-- Creating a partitioned table (PostgreSQL)
CREATE TABLE metrics (
    timestamp TIMESTAMP NOT NULL,
    device_id INTEGER NOT NULL,
    temperature NUMERIC,
    humidity NUMERIC
) PARTITION BY RANGE (timestamp);

-- Creating partitions by month
CREATE TABLE metrics_y2023m01 PARTITION OF metrics
FOR VALUES FROM ('2023-01-01') TO ('2023-02-01');

CREATE TABLE metrics_y2023m02 PARTITION OF metrics
FOR VALUES FROM ('2023-02-01') TO ('2023-03-01');

-- Query that benefits from partition pruning
SELECT AVG(temperature)
FROM metrics
WHERE timestamp BETWEEN '2023-01-15' AND '2023-01-20'
  AND device_id = 42;
```

### Multi-Tenant Data Architecture

```sql
-- Approach 1: Separate schema per tenant
CREATE SCHEMA tenant_123;

CREATE TABLE tenant_123.users (
    id SERIAL PRIMARY KEY,
    name TEXT,
    email TEXT
);

-- Approach 2: Tenant column in shared tables
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    tenant_id INTEGER NOT NULL,
    name TEXT,
    email TEXT,
    UNIQUE (tenant_id, email)
);

CREATE INDEX idx_users_tenant ON users(tenant_id);

-- Always filter by tenant_id in queries
SELECT * FROM users WHERE tenant_id = 123;
```

### Versioning and Concurrency Control

```sql
-- Optimistic concurrency control with version
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    title TEXT,
    content TEXT,
    version INTEGER DEFAULT 1,
    last_modified TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Update with version check
UPDATE documents
SET 
    title = 'New Title',
    content = 'Updated content',
    version = version + 1,
    last_modified = CURRENT_TIMESTAMP
WHERE id = 42 AND version = 3;

-- If 0 rows affected, someone else updated it first
```

## 5. SQL in Full-Stack Development

### Backend API Integration

```javascript
// Node.js with PostgreSQL example
const { Pool } = require('pg');
const pool = new Pool({
  connectionString: process.env.DATABASE_URL
});

// API endpoint for user dashboard data
app.get('/api/dashboard', async (req, res) => {
  const userId = req.user.id;
  
  try {
    // Using a transaction for consistent reads
    const client = await pool.connect();
    try {
      await client.query('BEGIN');
      
      // Get user profile
      const userResult = await client.query(
        'SELECT id, name, email FROM users WHERE id = $1',
        [userId]
      );
      
      // Get recent orders
      const ordersResult = await client.query(
        'SELECT id, order_date, total_amount, status FROM orders WHERE user_id = $1 ORDER BY order_date DESC LIMIT 5',
        [userId]
      );
      
      // Get spending summary
      const spendingResult = await client.query(
        `SELECT 
          SUM(total_amount) AS total_spent,
          COUNT(*) AS order_count,
          AVG(total_amount) AS average_order
         FROM orders
         WHERE user_id = $1 AND status = 'completed'`,
        [userId]
      );
      
      await client.query('COMMIT');
      
      res.json({
        profile: userResult.rows[0],
        recentOrders: ordersResult.rows,
        spending: spendingResult.rows[0]
      });
    } catch (e) {
      await client.query('ROLLBACK');
      throw e;
    } finally {
      client.release();
    }
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Database error' });
  }
});
```

### ORM vs. Raw SQL

**ORM (Object-Relational Mapping)**

```javascript
// Using Sequelize ORM (Node.js)
const Order = require('./models/Order');
const User = require('./models/User');

async function getTopCustomers() {
  return await User.findAll({
    attributes: [
      'id', 
      'name', 
      'email',
      [sequelize.fn('COUNT', sequelize.col('Orders.id')), 'orderCount'],
      [sequelize.fn('SUM', sequelize.col('Orders.totalAmount')), 'totalSpent']
    ],
    include: [{
      model: Order,
      attributes: []
    }],
    where: {
      '$Orders.status$': 'completed'
    },
    group: ['User.id'],
    having: sequelize.literal('COUNT(Orders.id) >= 3'),
    order: [[sequelize.literal('totalSpent'), 'DESC']],
    limit: 10
  });
}
```

**Raw SQL**

```javascript
// Using a database client directly
async function getTopCustomers() {
  const { rows } = await pool.query(`
    SELECT 
      u.id, 
      u.name, 
      u.email, 
      COUNT(o.id) AS order_count,
      SUM(o.total_amount) AS total_spent
    FROM users u
    JOIN orders o ON u.id = o.user_id
    WHERE o.status = 'completed'
    GROUP BY u.id, u.name, u.email
    HAVING COUNT(o.id) >= 3
    ORDER BY total_spent DESC
    LIMIT 10
  `);
  
  return rows;
}
```

### Security Best Practices

1. **Parameterized Queries**: Prevent SQL injection

```javascript
// UNSAFE - vulnerable to SQL injection
const query = `SELECT * FROM users WHERE email = '${userInput}'`;

// SAFE - parameterized query
const query = 'SELECT * FROM users WHERE email = $1';
const values = [userInput];
await client.query(query, values);
```

2. **Least Privilege Principle**: Create database users with minimal permissions

```sql
-- Create application-specific database user
CREATE USER app_user WITH PASSWORD 'secure_password';

-- Grant only necessary permissions
GRANT SELECT, INSERT, UPDATE ON users, orders, products TO app_user;
GRANT SELECT ON analytics_views TO app_user;
```

3. **Connection Pooling**: Efficient management of database connections

```javascript
// Node.js PostgreSQL pool configuration
const pool = new Pool({
  user: 'app_user',
  password: 'secure_password',
  host: 'localhost',
  database: 'my_app',
  min: 2,          // Minimum pool size
  max: 10,         // Maximum pool size
  idleTimeoutMillis: 30000  // Close idle connections after 30s
});
```

## Key Takeaways for Full-Stack Developers

1. **Transactions are essential for data integrity**: Use them to ensure atomic operations, especially for critical business logic.

2. **Indexes can dramatically improve performance**: Place them on frequently queried columns but be mindful of write overhead.

3. **Analyze slow queries**: Use EXPLAIN to understand and optimize complex operations.

4. **Choose the right tool for each query**:
   - Simple CRUD: ORM is often sufficient
   - Complex reporting: Raw SQL or query builders provide more control
   - Real-time analytics: Consider materialized views or caching

5. **Security is non-negotiable**: Always use parameterized queries and follow least privilege principles.

6. **Consider application architecture**:
   - API-driven apps: Focus on efficient endpoints with minimal round trips
   - Microservices: Define clear data boundaries and service interfaces
   - Monoliths: Leverage transactions across related operations

7. **Plan for scale from the start**:
   - Design efficient schemas
   - Use appropriate indexes
   - Implement pagination
   - Consider read replicas for read-heavy workloads