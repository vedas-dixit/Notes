# SQL for Full-Stack Developers - Part 2: Joins and Aggregations

## 2. Joins (Handling Multiple Tables)

Joins allow you to combine data from multiple tables into meaningful results. They're essential for working with relational databases in full-stack applications.

### JOIN Types Visualized

```
Table A         Table B
+----+------+  +----+------+
| id | name |  | id | name |
+----+------+  +----+------+
| 1  | A1   |  | 1  | B1   |
| 2  | A2   |  | 3  | B3   |
| 3  | A3   |  | 4  | B4   |
+----+------+  +----+------+
```

### INNER JOIN

Returns only the rows that have matching values in both tables.

```sql
-- Basic INNER JOIN syntax
SELECT column1, column2, ...
FROM table1
INNER JOIN table2 ON table1.column_name = table2.column_name;

-- INNER JOIN example between users and orders
SELECT 
    u.id AS user_id,
    u.username,
    o.id AS order_id,
    o.order_date,
    o.total_amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```

**INNER JOIN Result:**
```
+----+------+----+------+
| id | name | id | name |
+----+------+----+------+
| 1  | A1   | 1  | B1   |
| 3  | A3   | 3  | B3   |
+----+------+----+------+
```

**Real-world use cases:**
- Finding all active rentals along with customer information
- Getting orders with their corresponding customer details
- Finding products with their category information

**Example: Fetching product details with categories**
```sql
SELECT 
    p.id,
    p.name,
    p.price,
    p.stock_quantity,
    c.name AS category_name
FROM products p
INNER JOIN categories c ON p.category_id = c.id
WHERE p.stock_quantity > 0
ORDER BY c.name, p.name;
```

### LEFT JOIN (LEFT OUTER JOIN)

Returns all rows from the left table and matching rows from the right table. If no match exists, NULL values appear for right table columns.

```sql
-- Basic LEFT JOIN syntax
SELECT column1, column2, ...
FROM table1
LEFT JOIN table2 ON table1.column_name = table2.column_name;

-- LEFT JOIN example between users and orders
SELECT 
    u.id AS user_id,
    u.username,
    o.id AS order_id,
    o.order_date,
    o.total_amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

**LEFT JOIN Result:**
```
+----+------+------+------+
| id | name | id   | name |
+----+------+------+------+
| 1  | A1   | 1    | B1   |
| 2  | A2   | NULL | NULL |
| 3  | A3   | 3    | B3   |
+----+------+------+------+
```

**Real-world use cases:**
- Showing all users and their orders (including users with no orders)
- Displaying all products and their reviews (including products with no reviews)
- Finding customers without a purchase (where the right side is NULL)

**Example: Finding users without orders**
```sql
SELECT 
    u.id,
    u.username,
    u.email,
    u.created_at
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

### RIGHT JOIN (RIGHT OUTER JOIN)

Returns all rows from the right table and matching rows from the left table. If no match exists, NULL values appear for left table columns.

```sql
-- Basic RIGHT JOIN syntax
SELECT column1, column2, ...
FROM table1
RIGHT JOIN table2 ON table1.column_name = table2.column_name;

-- RIGHT JOIN example between users and orders
SELECT 
    u.id AS user_id,
    u.username,
    o.id AS order_id,
    o.order_date,
    o.total_amount
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;
```

**RIGHT JOIN Result:**
```
+------+------+----+------+
| id   | name | id | name |
+------+------+----+------+
| 1    | A1   | 1  | B1   |
| 3    | A3   | 3  | B3   |
| NULL | NULL | 4  | B4   |
+------+------+----+------+
```

**Real-world use cases:**
- Finding orders placed by deleted accounts
- Identifying records in child tables with no parent (orphaned records)
- Auditing data integrity issues

**Example: Finding orders with missing user accounts**
```sql
SELECT 
    o.id AS order_id,
    o.order_date,
    o.total_amount,
    u.id AS user_id,
    u.username
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id
WHERE u.id IS NULL;
```

### FULL JOIN (FULL OUTER JOIN)

Returns all rows when there's a match in either the left or right table. If no match exists, NULL values appear for the columns from the table without a match.

```sql
-- Basic FULL JOIN syntax
SELECT column1, column2, ...
FROM table1
FULL JOIN table2 ON table1.column_name = table2.column_name;

-- FULL JOIN example between users and orders
SELECT 
    u.id AS user_id,
    u.username,
    o.id AS order_id,
    o.order_date,
    o.total_amount
FROM users u
FULL JOIN orders o ON u.id = o.user_id;
```

**FULL JOIN Result:**
```
+------+------+------+------+
| id   | name | id   | name |
+------+------+------+------+
| 1    | A1   | 1    | B1   |
| 2    | A2   | NULL | NULL |
| 3    | A3   | 3    | B3   |
| NULL | NULL | 4    | B4   |
+------+------+------+------+
```

**Real-world use cases:**
- Comprehensive data auditing
- Finding data inconsistencies between tables
- Migration verification between old and new systems

**Example: Auditing data consistency**
```sql
SELECT 
    u.id AS user_id,
    u.username,
    o.id AS order_id,
    o.total_amount
FROM users u
FULL JOIN orders o ON u.id = o.user_id
WHERE u.id IS NULL OR o.user_id IS NULL;
```

### CROSS JOIN (Cartesian Product)

Returns the Cartesian product of both tables (all possible combinations of rows).

```sql
-- Basic CROSS JOIN syntax
SELECT column1, column2, ...
FROM table1
CROSS JOIN table2;

-- CROSS JOIN example
SELECT 
    p.name AS product_name,
    s.name AS size_name,
    c.name AS color_name
FROM products p
CROSS JOIN sizes s
CROSS JOIN colors c;
```

**CROSS JOIN Result:**
```
+----+------+----+------+
| id | name | id | name |
+----+------+----+------+
| 1  | A1   | 1  | B1   |
| 1  | A1   | 3  | B3   |
| 1  | A1   | 4  | B4   |
| 2  | A2   | 1  | B1   |
| 2  | A2   | 3  | B3   |
| 2  | A2   | 4  | B4   |
| 3  | A3   | 1  | B1   |
| 3  | A3   | 3  | B3   |
| 3  | A3   | 4  | B4   |
+----+------+----+------+
```

**Real-world use cases:**
- Generating all possible product variations (size, color, style)
- Creating a schedule with all possible time slots
- Calculating possible combinations for configuration options

**Example: Generating product variations**
```sql
SELECT 
    p.id AS product_id,
    p.name AS product_name,
    s.name AS size,
    c.name AS color,
    p.base_price + s.price_adjustment + c.price_adjustment AS final_price
FROM products p
CROSS JOIN sizes s
CROSS JOIN colors c
WHERE p.id = 101;
```

### SELF JOIN

Joins a table to itself, treating it as two separate tables. Useful for hierarchical data.

```sql
-- Basic SELF JOIN syntax
SELECT column1, column2, ...
FROM table1 AS t1
JOIN table1 AS t2 ON t1.column_name = t2.different_column;

-- SELF JOIN example with employees (manager relationship)
SELECT 
    e.id AS employee_id,
    e.name AS employee_name,
    m.id AS manager_id,
    m.name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

**Real-world use cases:**
- Employee-manager relationships
- Category and subcategory relationships
- Finding related products
- Social media connections (finding friends of friends)

**Example: Organization chart**
```sql
SELECT 
    e.id,
    e.name AS employee,
    e.title,
    m.name AS manager,
    m.title AS manager_title
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id
ORDER BY 
    CASE WHEN e.manager_id IS NULL THEN 0 ELSE 1 END,
    m.name,
    e.name;
```

### Multiple JOINs

Full-stack applications often need to join more than two tables to retrieve complete information.

```sql
-- Joining multiple tables
SELECT 
    o.id AS order_id,
    o.order_date,
    c.name AS customer_name,
    p.name AS product_name,
    oi.quantity,
    oi.unit_price,
    oi.quantity * oi.unit_price AS line_total
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE o.order_date >= '2023-01-01'
ORDER BY o.order_date DESC, o.id;
```

**Example: Blog post with comments and user information**
```sql
SELECT 
    p.id,
    p.title,
    p.content,
    p.created_at,
    u.username AS author,
    COUNT(c.id) AS comment_count,
    ARRAY_AGG(c.content) AS recent_comments
FROM posts p
JOIN users u ON p.user_id = u.id
LEFT JOIN (
    SELECT * FROM comments 
    ORDER BY created_at DESC 
    LIMIT 5
) c ON p.id = c.post_id
WHERE p.published = TRUE
GROUP BY p.id, p.title, p.content, p.created_at, u.username
ORDER BY p.created_at DESC;
```

## 3. Aggregations (Analytics Queries)

Aggregation functions calculate a single value from a set of input values. They're essential for analytics and reporting in full-stack applications.

### Basic Aggregation Functions

```sql
-- COUNT - counts the number of rows
SELECT COUNT(*) FROM orders;
SELECT COUNT(email) FROM users;  -- Excludes NULL values
SELECT COUNT(DISTINCT category_id) FROM products;

-- SUM - calculates the sum of numeric values
SELECT SUM(total_amount) FROM orders;
SELECT SUM(quantity * unit_price) FROM order_items;

-- AVG - calculates the average of numeric values
SELECT AVG(rating) FROM reviews;
SELECT AVG(COALESCE(discount, 0)) FROM orders;

-- MIN/MAX - finds minimum and maximum values
SELECT MIN(price) FROM products;
SELECT MAX(order_date) FROM orders;

-- Statistical functions (available in some databases)
SELECT 
    STDDEV(price) AS price_std_dev,
    VARIANCE(price) AS price_variance
FROM products;
```

### GROUP BY

The GROUP BY clause divides rows into groups, allowing aggregate functions to operate on each group independently.

```sql
-- Basic GROUP BY syntax
SELECT column1, column2, aggregate_function(column3)
FROM table_name
GROUP BY column1, column2;

-- Simple grouping example
SELECT 
    category_id,
    COUNT(*) AS product_count,
    AVG(price) AS average_price,
    MIN(price) AS min_price,
    MAX(price) AS max_price
FROM products
GROUP BY category_id;
```

**Example: Monthly sales report**
```sql
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    COUNT(*) AS order_count,
    COUNT(DISTINCT user_id) AS unique_customers,
    SUM(total_amount) AS revenue,
    AVG(total_amount) AS average_order_value
FROM orders
WHERE order_date >= '2023-01-01'
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

### HAVING

The HAVING clause filters groups based on aggregate results. It's like a WHERE clause but for grouped data.

```sql
-- Basic HAVING syntax
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1
HAVING condition;

-- Finding popular product categories
SELECT 
    category_id,
    COUNT(*) AS product_count
FROM products
GROUP BY category_id
HAVING COUNT(*) > 10;
```

**Example: Finding products with high price variance**
```sql
SELECT 
    category_id,
    COUNT(*) AS product_count,
    AVG(price) AS avg_price,
    STDDEV(price) AS price_stddev,
    STDDEV(price) / AVG(price) AS price_variation_coefficient
FROM products
GROUP BY category_id
HAVING COUNT(*) >= 5 AND STDDEV(price) / AVG(price) > 0.3
ORDER BY price_variation_coefficient DESC;
```

### Complete Workflow (WHERE → GROUP BY → HAVING → ORDER BY)

```sql
SELECT 
    u.country,
    EXTRACT(YEAR FROM o.order_date) AS year,
    COUNT(DISTINCT u.id) AS customer_count,
    COUNT(o.id) AS order_count,
    SUM(o.total_amount) AS total_revenue,
    AVG(o.total_amount) AS avg_order_value,
    SUM(o.total_amount) / COUNT(DISTINCT u.id) AS revenue_per_customer
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE 
    o.order_date BETWEEN '2020-01-01' AND '2023-12-31'
    AND o.status = 'completed'
GROUP BY 
    u.country, EXTRACT(YEAR FROM o.order_date)
HAVING 
    COUNT(o.id) >= 100 AND SUM(o.total_amount) > 10000
ORDER BY 
    year, total_revenue DESC;
```

### Aggregation with JOIN

```sql
-- Finding average rating per product with product details
SELECT 
    p.id,
    p.name,
    p.price,
    c.name AS category,
    COUNT(r.id) AS review_count,
    AVG(r.rating) AS average_rating
FROM products p
JOIN categories c ON p.category_id = c.id
LEFT JOIN reviews r ON p.id = r.product_id
GROUP BY p.id, p.name, p.price, c.name
HAVING COUNT(r.id) >= 5
ORDER BY average_rating DESC, review_count DESC;
```

### Grouping with Multiple Tables

```sql
-- Sales by category and country
SELECT 
    c.name AS category,
    u.country,
    COUNT(o.id) AS order_count,
    SUM(oi.quantity) AS total_items_sold,
    SUM(oi.quantity * oi.unit_price) AS total_revenue
FROM categories c
JOIN products p ON c.id = p.category_id
JOIN order_items oi ON p.id = oi.product_id
JOIN orders o ON oi.order_id = o.id
JOIN users u ON o.user_id = u.id
WHERE o.status = 'completed'
GROUP BY c.name, u.country
ORDER BY c.name, total_revenue DESC;
```

### Advanced Group By Techniques

```sql
-- Grouping sets (equivalent to multiple GROUP BY queries combined with UNION)
SELECT 
    COALESCE(category_name, 'All Categories') AS category,
    COALESCE(country, 'All Countries') AS country,
    SUM(revenue) AS total_revenue
FROM sales
GROUP BY GROUPING SETS (
    (category_name, country),
    (category_name),
    (country),
    ()
);

-- ROLLUP (hierarchical subtotals)
SELECT 
    COALESCE(year::TEXT, 'All Years') AS year,
    COALESCE(quarter::TEXT, 'All Quarters') AS quarter,
    COALESCE(month::TEXT, 'All Months') AS month,
    SUM(revenue) AS total_revenue
FROM sales
GROUP BY ROLLUP(year, quarter, month);

-- CUBE (all possible combinations of subtotals)
SELECT 
    COALESCE(category::TEXT, 'All Categories') AS category,
    COALESCE(region::TEXT, 'All Regions') AS region,
    COALESCE(country::TEXT, 'All Countries') AS country,
    SUM(revenue) AS total_revenue
FROM sales
GROUP BY CUBE(category, region, country);
```

## Real-World Aggregation Use Cases for Full-Stack Apps

### User Analytics Dashboard

```sql
-- Daily active users (DAU)
SELECT 
    DATE(login_timestamp) AS date,
    COUNT(DISTINCT user_id) AS daily_active_users
FROM user_sessions
WHERE login_timestamp >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY DATE(login_timestamp)
ORDER BY date;

-- Weekly active users (WAU)
SELECT 
    DATE_TRUNC('week', login_timestamp) AS week,
    COUNT(DISTINCT user_id) AS weekly_active_users
FROM user_sessions
WHERE login_timestamp >= CURRENT_DATE - INTERVAL '12 weeks'
GROUP BY DATE_TRUNC('week', login_timestamp)
ORDER BY week;
```

### E-commerce Analytics

```sql
-- Customer lifetime value (CLTV)
SELECT 
    u.id,
    u.email,
    COUNT(o.id) AS order_count,
    SUM(o.total_amount) AS total_spent,
    MIN(o.order_date) AS first_order_date,
    MAX(o.order_date) AS last_order_date,
    AVG(o.total_amount) AS average_order_value,
    SUM(o.total_amount) / NULLIF(
        EXTRACT(YEAR FROM AGE(CURRENT_DATE, MIN(o.order_date))), 0
    ) AS yearly_value
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed'
GROUP BY u.id, u.email
HAVING COUNT(o.id) > 0
ORDER BY total_spent DESC
LIMIT 100;

-- Product performance analysis
SELECT 
    p.id,
    p.name,
    COUNT(oi.order_id) AS order_count,
    SUM(oi.quantity) AS units_sold,
    SUM(oi.quantity * oi.unit_price) AS revenue,
    AVG(oi.unit_price) AS average_selling_price,
    (SUM(oi.quantity * oi.unit_price) / SUM(oi.quantity)) AS weighted_average_price
FROM products p
LEFT JOIN order_items oi ON p.id = oi.product_id
LEFT JOIN orders o ON oi.order_id = o.id
WHERE o.status = 'completed' AND o.order_date >= CURRENT_DATE - INTERVAL '90 days'
GROUP BY p.id, p.name
ORDER BY revenue DESC;
```

### Content Platform Analytics

```sql
-- Content engagement metrics
SELECT 
    c.id,
    c.title,
    COUNT(v.id) AS view_count,
    COUNT(DISTINCT v.user_id) AS unique_viewers,
    AVG(EXTRACT(EPOCH FROM (v.end_time - v.start_time)) / 60) AS avg_view_minutes,
    SUM(CASE WHEN v.completed = TRUE THEN 1 ELSE 0 END) AS completion_count,
    SUM(CASE WHEN v.completed = TRUE THEN 1 ELSE 0 END)::FLOAT / COUNT(v.id) AS completion_rate,
    COUNT(l.id) AS like_count,
    COUNT(s.id) AS share_count
FROM content c
LEFT JOIN content_views v ON c.id = v.content_id
LEFT JOIN content_likes l ON c.id = l.content_id
LEFT JOIN content_shares s ON c.id = s.content_id
GROUP BY c.id, c.title
ORDER BY view_count DESC;
```

## Key Takeaways for Joins and Aggregations

1. **Choose the right JOIN type**: 
   - INNER JOIN: When you need matching rows from both tables
   - LEFT JOIN: When you need all rows from the first table
   - RIGHT JOIN: When you need all rows from the second table (rare)
   - FULL JOIN: When you need all rows from both tables (rare)

2. **Optimize JOIN performance**:
   - Always join on indexed columns
   - Filter tables before joining when possible
   - Be careful with multiple joins (each join multiplies the potential result set)

3. **Aggregation best practices**:
   - Use GROUP BY with specific columns to avoid unexpected results
   - Filter with WHERE before grouping for better performance
   - Use HAVING only for conditions involving aggregate functions
   - Ensure all non-aggregated columns in SELECT are in the GROUP BY clause

4. **Common JOIN patterns for full-stack apps**:
   - User → Profile (1:1)
   - Order → Order Items (1:Many)
   - Product → Categories (Many:1)
   - Users → Roles (Many:Many through junction table)

Joins and aggregations are fundamental for building data-driven features in full-stack applications. They allow you to transform raw data into meaningful insights and provide users with relevant information spanning multiple related entities. 