# SQL for Full-Stack Developers - Part 1: The Basics

## 1. Introduction to SQL in Full-Stack Development

SQL (Structured Query Language) is an essential skill for full-stack developers, serving as the bridge between your application and its data. Whether you're building a personal project or working on enterprise software, understanding SQL fundamentals will help you:

- Design efficient database schemas
- Write optimized queries
- Debug data-related issues
- Understand ORMs (Object-Relational Mappers) more deeply
- Create complex reports and analytics

### SQL vs. NoSQL - When to Use Each

While NoSQL databases have gained popularity, SQL databases remain essential for many applications:

| SQL Databases | NoSQL Databases |
|---------------|-----------------|
| Structured data with relationships | Unstructured or semi-structured data |
| Complex queries and transactions | High write throughput |
| Data integrity and consistency | Horizontal scalability |
| Financial applications, ERP, CRM | Content management, IoT, real-time analytics |
| MySQL, PostgreSQL, SQL Server | MongoDB, Redis, Cassandra |

Most full-stack applications benefit from understanding both paradigms, often using SQL for core business data and NoSQL for specific use cases like caching or logging.

## 2. Basic SQL Syntax and Operations

### Database and Table Creation

```sql
-- Creating a database
CREATE DATABASE ecommerce;

-- Using the database
USE ecommerce;

-- Creating a table with various data types
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    stock_quantity INT DEFAULT 0,
    category_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);
```

### Common Data Types

| Data Type | Description | Example Use Case |
|-----------|-------------|------------------|
| INT | Integer numbers | IDs, counts, ages |
| DECIMAL(p,s) | Precise decimal numbers | Prices, financial data |
| VARCHAR(n) | Variable-length strings | Names, titles, codes |
| TEXT | Longer text content | Descriptions, content |
| DATE | Date only | Birth dates, publication dates |
| TIMESTAMP | Date and time | Created/updated timestamps |
| BOOLEAN | True/false values | Status flags, toggles |
| JSON | JSON structured data | Flexible attributes, settings |

### CRUD Operations

CRUD stands for Create, Read, Update, and Delete - the four basic operations for persistent storage.

#### CREATE (INSERT)

```sql
-- Basic insert
INSERT INTO users (username, email, password_hash, created_at)
VALUES ('johndoe', 'john@example.com', 'hashed_password', NOW());

-- Multiple row insert
INSERT INTO categories (name, parent_id)
VALUES
    ('Electronics', NULL),
    ('Computers', 1),
    ('Smartphones', 1),
    ('Clothing', NULL);

-- Insert with select
INSERT INTO inactive_users (id, username, email)
SELECT id, username, email FROM users
WHERE last_login_date < DATE_SUB(NOW(), INTERVAL 1 YEAR);
```

#### READ (SELECT)

```sql
-- Basic select
SELECT id, username, email, created_at FROM users;

-- Select with conditions
SELECT * FROM products
WHERE price > 100 AND category_id = 3;

-- Select with pattern matching
SELECT id, name, price FROM products
WHERE name LIKE 'iPhone%';

-- Select with ordering
SELECT id, name, price FROM products
ORDER BY price DESC, name ASC;

-- Select with limits
SELECT id, name, price FROM products
ORDER BY created_at DESC
LIMIT 10;

-- Select with pagination
SELECT id, name, price FROM products
ORDER BY id
LIMIT 20 OFFSET 40;  -- Page 3 with 20 items per page
```

#### UPDATE

```sql
-- Basic update
UPDATE users
SET last_login_date = NOW()
WHERE id = 123;

-- Update with calculations
UPDATE products
SET stock_quantity = stock_quantity - 1,
    sold_count = sold_count + 1
WHERE id = 456;

-- Update with subquery
UPDATE products
SET price = price * 0.9
WHERE category_id IN (SELECT id FROM categories WHERE name = 'Electronics');
```

#### DELETE

```sql
-- Basic delete
DELETE FROM shopping_cart_items
WHERE user_id = 123;

-- Delete with joins
DELETE cart_items
FROM shopping_cart_items cart_items
JOIN shopping_carts carts ON cart_items.cart_id = carts.id
WHERE carts.created_at < DATE_SUB(NOW(), INTERVAL 30 DAY)
AND carts.status = 'abandoned';

-- Truncate (delete all rows efficiently)
TRUNCATE TABLE temporary_logs;
```

### Filtering Data (WHERE Clause)

```sql
-- Basic comparison operators
SELECT * FROM products WHERE price > 100;
SELECT * FROM users WHERE created_at >= '2023-01-01';
SELECT * FROM orders WHERE status != 'completed';

-- Logical operators
SELECT * FROM products WHERE price > 100 AND category_id = 5;
SELECT * FROM orders WHERE status = 'pending' OR status = 'processing';
SELECT * FROM products WHERE NOT is_active;

-- Range operators
SELECT * FROM products WHERE price BETWEEN 20 AND 50;
SELECT * FROM orders WHERE created_at BETWEEN '2023-01-01' AND '2023-01-31';

-- Set membership
SELECT * FROM products WHERE category_id IN (1, 3, 5);
SELECT * FROM users WHERE country NOT IN ('US', 'CA', 'MX');

-- Pattern matching
SELECT * FROM products WHERE name LIKE 'Apple%';  -- Starts with "Apple"
SELECT * FROM products WHERE name LIKE '%Phone%';  -- Contains "Phone"
SELECT * FROM users WHERE email LIKE '%@gmail.com';  -- Ends with "@gmail.com"

-- NULL checks
SELECT * FROM users WHERE phone IS NULL;
SELECT * FROM products WHERE deleted_at IS NOT NULL;
```

### Sorting Data (ORDER BY)

```sql
-- Single column sorting
SELECT * FROM products ORDER BY price ASC;  -- Ascending (default)
SELECT * FROM users ORDER BY created_at DESC;  -- Descending

-- Multi-column sorting
SELECT * FROM orders ORDER BY status ASC, created_at DESC;

-- Using expressions in sorting
SELECT id, name, price, stock_quantity FROM products
ORDER BY price * stock_quantity DESC;  -- Sort by inventory value

-- Sorting with NULLS
SELECT * FROM users ORDER BY last_login_date DESC NULLS LAST;
```

## 3. Schema Design for Full-Stack Applications

### Database Normalization

Normalization is the process of organizing database tables to reduce redundancy and improve data integrity.

#### First Normal Form (1NF)
- Each table cell should contain a single value
- Each record needs to be unique

#### Second Normal Form (2NF)
- Table is in 1NF
- All non-key attributes depend on the entire primary key

#### Third Normal Form (3NF)
- Table is in 2NF
- No transitive dependencies (non-key attributes depend only on the primary key)

#### Example of Normalization

**Unnormalized Table:**
```
| OrderID | Customer | ProductName | Category    | Price | Quantity |
|---------|----------|-------------|-------------|-------|----------|
| 1       | John     | Laptop      | Electronics | 1200  | 1        |
| 1       | John     | Mouse       | Electronics | 25    | 2        |
| 2       | Sarah    | T-shirt     | Clothing    | 20    | 3        |
```

**Normalized Tables (3NF):**

Customers:
```
| CustomerID | Name  |
|------------|-------|
| 1          | John  |
| 2          | Sarah |
```

Products:
```
| ProductID | Name    | CategoryID | Price |
|-----------|---------|------------|-------|
| 1         | Laptop  | 1          | 1200  |
| 2         | Mouse   | 1          | 25    |
| 3         | T-shirt | 2          | 20    |
```

Categories:
```
| CategoryID | Name        |
|------------|-------------|
| 1          | Electronics |
| 2          | Clothing    |
```

Orders:
```
| OrderID | CustomerID | OrderDate  |
|---------|------------|------------|
| 1       | 1          | 2023-01-01 |
| 2       | 2          | 2023-01-02 |
```

OrderItems:
```
| OrderID | ProductID | Quantity |
|---------|-----------|----------|
| 1       | 1         | 1        |
| 1       | 2         | 2        |
| 2       | 3         | 3        |
```

### Common Database Relationships

#### One-to-One (1:1)
One record in table A relates to exactly one record in table B.

Example: User and UserProfile

```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE user_profiles (
    user_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    bio TEXT,
    avatar_url VARCHAR(255),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

#### One-to-Many (1:N)
One record in table A relates to multiple records in table B.

Example: Category and Products

```sql
CREATE TABLE categories (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    description TEXT
);

CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    category_id INT,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);
```

#### Many-to-Many (N:M)
Multiple records in table A relate to multiple records in table B, requiring a junction table.

Example: Products and Orders

```sql
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);

CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

### Keys and Constraints

#### Primary Keys
Uniquely identify each record in a table.

```sql
-- Auto-incrementing integer (most common)
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL
);

-- Composite primary key
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    PRIMARY KEY (order_id, product_id)
);

-- UUID as primary key
CREATE TABLE documents (
    id CHAR(36) PRIMARY KEY,
    title VARCHAR(100) NOT NULL
);
```

#### Foreign Keys
Ensure referential integrity between tables.

```sql
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    category_id INT,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);

-- With cascade operations
CREATE TABLE comments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    post_id INT NOT NULL,
    content TEXT NOT NULL,
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE
);
```

#### Common Constraints

```sql
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    sku VARCHAR(20) UNIQUE,
    price DECIMAL(10, 2) NOT NULL CHECK (price > 0),
    stock_quantity INT DEFAULT 0 CHECK (stock_quantity >= 0),
    category_id INT,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);
```

## 4. Working with Dates and Times

### Date/Time Functions

```sql
-- Current date and time
SELECT CURRENT_DATE();
SELECT CURRENT_TIME();
SELECT NOW();  -- Date and time

-- Extracting parts
SELECT YEAR(created_at) FROM orders;
SELECT MONTH(order_date) FROM orders;
SELECT DAY(birth_date) FROM users;

-- Formatting
SELECT DATE_FORMAT(created_at, '%Y-%m-%d') FROM orders;
SELECT DATE_FORMAT(created_at, '%W, %M %D, %Y') FROM orders;

-- Date arithmetic
SELECT DATE_ADD(order_date, INTERVAL 14 DAY) AS delivery_due_date FROM orders;
SELECT DATE_SUB(NOW(), INTERVAL 30 DAY) AS thirty_days_ago;
SELECT DATEDIFF(return_date, checkout_date) AS rental_days FROM rentals;
```

### Common Date/Time Patterns

```sql
-- Find records from today
SELECT * FROM orders WHERE DATE(created_at) = CURRENT_DATE();

-- Find records from the last 7 days
SELECT * FROM orders WHERE created_at >= DATE_SUB(NOW(), INTERVAL 7 DAY);

-- Find records from current month
SELECT * FROM orders 
WHERE YEAR(created_at) = YEAR(CURRENT_DATE()) 
AND MONTH(created_at) = MONTH(CURRENT_DATE());

-- Group by day, month, year
SELECT 
    DATE(created_at) AS order_date,
    COUNT(*) AS orders,
    SUM(total_amount) AS revenue
FROM orders
GROUP BY DATE(created_at)
ORDER BY order_date;

-- Time component queries (business hours)
SELECT * FROM events
WHERE TIME(start_time) BETWEEN '09:00:00' AND '17:00:00';
```

## 5. String Operations and Pattern Matching

### String Functions

```sql
-- String manipulation
SELECT UPPER(username) FROM users;
SELECT LOWER(email) FROM users;
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;
SELECT LENGTH(description) AS desc_length FROM products;

-- Trimming whitespace
SELECT TRIM(comment_text) FROM comments;
SELECT LTRIM(user_input) FROM form_submissions;
SELECT RTRIM(code) FROM promo_codes;

-- Substring operations
SELECT SUBSTRING(description, 1, 100) AS excerpt FROM articles;
SELECT LEFT(phone, 3) AS area_code FROM customers;
SELECT RIGHT(file_name, 4) AS extension FROM uploads;

-- Replacement
SELECT REPLACE(description, 'old', 'new') FROM products;
```

### Pattern Matching

```sql
-- Basic pattern matching with LIKE
SELECT * FROM products WHERE name LIKE 'iPhone%';
SELECT * FROM users WHERE email LIKE '%@gmail.com';

-- Case-insensitive search
SELECT * FROM products WHERE LOWER(name) LIKE '%wireless%';

-- Regex pattern matching (MySQL)
SELECT * FROM products WHERE name REGEXP '^[A-Z]';
SELECT * FROM phone_numbers WHERE number REGEXP '^\\+1-[0-9]{3}-[0-9]{3}-[0-9]{4}$';

-- Full-text search (MySQL)
CREATE FULLTEXT INDEX idx_product_search ON products(name, description);

SELECT * FROM products
WHERE MATCH(name, description) AGAINST('wireless headphones' IN NATURAL LANGUAGE MODE);
```

## 6. Transaction Control for Data Integrity

Transactions ensure that multiple SQL operations are treated as a single unit of work, maintaining data consistency.

### Basic Transaction Control

```sql
-- Start a transaction
BEGIN TRANSACTION;

-- Perform operations
UPDATE accounts SET balance = balance - 100 WHERE id = 123;
UPDATE accounts SET balance = balance + 100 WHERE id = 456;

-- Check for errors and commit or rollback
-- If successful:
COMMIT;

-- If an error occurred:
ROLLBACK;
```

### Implementing a Money Transfer

```sql
BEGIN TRANSACTION;

-- Deduct from source account
UPDATE accounts SET balance = balance - 100 WHERE id = 123;

-- Verify sufficient funds
IF @@ROWCOUNT = 0 OR (SELECT balance FROM accounts WHERE id = 123) < 0 THEN
    ROLLBACK;
    RAISERROR('Insufficient funds', 16, 1);
    RETURN;
END

-- Add to destination account
UPDATE accounts SET balance = balance + 100 WHERE id = 456;

-- Verify destination exists
IF @@ROWCOUNT = 0 THEN
    ROLLBACK;
    RAISERROR('Destination account not found', 16, 1);
    RETURN;
END

-- Create transaction log
INSERT INTO transfer_log (from_account, to_account, amount, timestamp)
VALUES (123, 456, 100, NOW());

COMMIT;
```

## 7. Real-World SQL Patterns for Full-Stack Apps

### Authentication and User Management

```sql
-- User registration
INSERT INTO users (username, email, password_hash, created_at)
VALUES ('newuser', 'user@example.com', 'hashed_password', NOW());

-- User login (check credentials)
SELECT id, username, role 
FROM users 
WHERE username = 'loginname' 
AND password_hash = 'hashed_input_password' 
AND is_active = TRUE;

-- User account lookup
SELECT u.id, u.username, u.email, u.created_at, p.first_name, p.last_name 
FROM users u
LEFT JOIN user_profiles p ON u.id = p.user_id
WHERE u.email = 'user@example.com';

-- Password reset
UPDATE users 
SET password_reset_token = 'generated_token', 
    password_reset_expires = DATE_ADD(NOW(), INTERVAL 1 HOUR)
WHERE email = 'user@example.com';
```

### Shopping Cart Implementation

```sql
-- Create a new cart
INSERT INTO carts (user_id, created_at) 
VALUES (123, NOW());

-- Add item to cart
INSERT INTO cart_items (cart_id, product_id, quantity, added_at)
VALUES (456, 789, 1, NOW())
ON DUPLICATE KEY UPDATE quantity = quantity + 1;

-- View cart with product details
SELECT 
    ci.cart_id,
    ci.product_id,
    p.name,
    p.price,
    ci.quantity,
    p.price * ci.quantity AS subtotal
FROM cart_items ci
JOIN products p ON ci.product_id = p.id
WHERE ci.cart_id = 456;

-- Calculate cart total
SELECT 
    cart_id,
    SUM(quantity) AS total_items,
    SUM(p.price * ci.quantity) AS total_amount
FROM cart_items ci
JOIN products p ON ci.product_id = p.id
WHERE cart_id = 456
GROUP BY cart_id;

-- Convert cart to order
BEGIN TRANSACTION;

-- Create order
INSERT INTO orders (user_id, total_amount, shipping_address_id, created_at)
SELECT c.user_id, SUM(p.price * ci.quantity), u.default_address_id, NOW()
FROM carts c
JOIN cart_items ci ON c.id = ci.cart_id
JOIN products p ON ci.product_id = p.id
JOIN users u ON c.user_id = u.id
WHERE c.id = 456
GROUP BY c.user_id, u.default_address_id;

SET @new_order_id = LAST_INSERT_ID();

-- Create order items from cart items
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
SELECT @new_order_id, ci.product_id, ci.quantity, p.price
FROM cart_items ci
JOIN products p ON ci.product_id = p.id
WHERE ci.cart_id = 456;

-- Clear the cart
DELETE FROM cart_items WHERE cart_id = 456;

COMMIT;
```

### Content Management System

```sql
-- Create a new blog post with tags
BEGIN TRANSACTION;

INSERT INTO posts (user_id, title, content, status, published_at)
VALUES (123, 'SQL for Beginners', 'This is a blog post about SQL...', 'published', NOW());

SET @new_post_id = LAST_INSERT_ID();

-- Insert tags
INSERT INTO post_tags (post_id, tag_id)
VALUES (@new_post_id, 1), (@new_post_id, 5), (@new_post_id, 9);

COMMIT;

-- Get published posts with tags and author information
SELECT 
    p.id,
    p.title,
    p.slug,
    SUBSTRING(p.content, 1, 200) AS excerpt,
    p.published_at,
    u.username AS author,
    GROUP_CONCAT(t.name SEPARATOR ', ') AS tags
FROM posts p
JOIN users u ON p.user_id = u.id
LEFT JOIN post_tags pt ON p.id = pt.post_id
LEFT JOIN tags t ON pt.tag_id = t.id
WHERE p.status = 'published' AND p.published_at <= NOW()
GROUP BY p.id, p.title, p.slug, p.content, p.published_at, u.username
ORDER BY p.published_at DESC
LIMIT 10;
```

## 8. Enhancing Performance with Indexes

Indexes speed up data retrieval but can slow down writes. Use them strategically for the best performance.

### Types of Indexes

```sql
-- Primary key (automatic index)
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50)
);

-- Simple index
CREATE INDEX idx_users_email ON users(email);

-- Unique index
CREATE UNIQUE INDEX idx_products_sku ON products(sku);

-- Composite index (multiple columns)
CREATE INDEX idx_orders_user_date ON orders(user_id, order_date);

-- Partial index
CREATE INDEX idx_products_active_price ON products(price) WHERE is_active = TRUE;

-- Full-text index
CREATE FULLTEXT INDEX idx_product_search ON products(name, description);
```

### When to Use Indexes

```sql
-- Add indexes on columns used in WHERE clauses
CREATE INDEX idx_products_category ON products(category_id);

-- Add indexes on columns used in JOIN conditions
CREATE INDEX idx_order_items_product ON order_items(product_id);

-- Add indexes on columns used in ORDER BY
CREATE INDEX idx_users_created_at ON users(created_at);

-- Add indexes on columns used in GROUP BY
CREATE INDEX idx_orders_status ON orders(status);
```

### Index Performance Considerations

1. **Don't over-index**: Each index adds overhead to write operations
2. **Consider column cardinality**: Indexes work best on columns with many unique values
3. **Index order matters** in composite indexes
4. **Update statistics** regularly for optimal query execution plans

## Key Takeaways for Full-Stack Developers

1. **Learn the basics thoroughly**: Understanding SQL fundamentals will help you with any database system.

2. **Think about data integrity**: Use constraints, transactions, and proper schema design to ensure data correctness.

3. **Optimize for common access patterns**: Design your schema and queries based on how your application will use the data.

4. **Balance normalization and performance**: While normalization reduces redundancy, some denormalization might be needed for performance.

5. **Use parameters for dynamic queries**: Never concatenate user input directly into SQL strings to avoid SQL injection.

6. **Learn your specific database**: Each database system (MySQL, PostgreSQL, SQL Server) has unique features worth exploring.

7. **Monitor and optimize**: Use query plans and execution times to identify and fix performance bottlenecks.

Building effective database skills takes practice. By mastering these concepts, you'll be able to create robust data layers for your full-stack applications that are both efficient and maintainable. 