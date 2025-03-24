# FAANG SQL Interview Questions

This guide contains common SQL interview questions asked at FAANG (Facebook, Amazon, Apple, Netflix, Google) and other top tech companies.

## Data Analysis Questions

### 1. Find Duplicate Records

**Question:** Find all duplicate email addresses in a `users` table.

```sql
SELECT email, COUNT(*) as count 
FROM users 
GROUP BY email 
HAVING COUNT(*) > 1;
```

### 2. Nth Highest Salary

**Question:** Find the Nth highest salary from the `employees` table.

```sql
-- Using subquery (generic solution)
SELECT DISTINCT salary
FROM employees e1
WHERE N = (
    SELECT COUNT(DISTINCT salary)
    FROM employees e2
    WHERE e2.salary >= e1.salary
);

-- Using window functions
SELECT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rank
    FROM employees
) ranked_salaries
WHERE rank = N;

-- For 2nd highest salary specifically
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

### 3. Department Top 3 Salaries

**Question:** Find the top 3 highest paid employees in each department.

```sql
SELECT d.name AS Department, e.name AS Employee, e.salary AS Salary
FROM employees e
JOIN departments d ON e.department_id = d.id
WHERE (
    SELECT COUNT(DISTINCT e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id AND e2.salary >= e.salary
) <= 3
ORDER BY d.name, e.salary DESC;

-- Using window functions (cleaner)
SELECT Department, Employee, Salary
FROM (
    SELECT 
        d.name AS Department,
        e.name AS Employee,
        e.salary AS Salary,
        DENSE_RANK() OVER (PARTITION BY e.department_id ORDER BY e.salary DESC) as salary_rank
    FROM employees e
    JOIN departments d ON e.department_id = d.id
) ranked
WHERE salary_rank <= 3
ORDER BY Department, Salary DESC;
```

### 4. Cumulative Sum

**Question:** Calculate the cumulative sum of transactions per user, ordered by transaction date.

```sql
-- Using window functions
SELECT 
    user_id, 
    transaction_date, 
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id 
        ORDER BY transaction_date
        ROWS UNBOUNDED PRECEDING
    ) AS cumulative_sum
FROM transactions
ORDER BY user_id, transaction_date;

-- Without window functions (for older MySQL)
SELECT 
    t1.user_id, 
    t1.transaction_date, 
    t1.amount,
    SUM(t2.amount) AS cumulative_sum
FROM transactions t1
JOIN transactions t2 
    ON t1.user_id = t2.user_id 
    AND t2.transaction_date <= t1.transaction_date
GROUP BY t1.user_id, t1.transaction_date, t1.amount
ORDER BY t1.user_id, t1.transaction_date;
```

### 5. Active User Retention

**Question:** Calculate the month-over-month retention of users. A user is considered retained if they were active in both the current month and the previous month.

```sql
WITH monthly_active AS (
    SELECT 
        user_id,
        DATE_TRUNC('month', activity_date) AS activity_month
    FROM user_activities
    GROUP BY user_id, DATE_TRUNC('month', activity_date)
)

SELECT 
    current_month.activity_month,
    COUNT(DISTINCT previous_month.user_id) AS retained_users,
    COUNT(DISTINCT current_month.user_id) AS total_users,
    ROUND(COUNT(DISTINCT previous_month.user_id) * 100.0 / 
          COUNT(DISTINCT current_month.user_id), 2) AS retention_rate
FROM monthly_active current_month
LEFT JOIN monthly_active previous_month 
    ON current_month.user_id = previous_month.user_id
    AND previous_month.activity_month = current_month.activity_month - INTERVAL '1 month'
GROUP BY current_month.activity_month
ORDER BY current_month.activity_month;
```

## Facebook/Meta Specific Questions

### 1. Friend Recommendations

**Question:** Write a query to recommend friends for a user. Recommend friends of friends who aren't already their friends.

```sql
SELECT DISTINCT f1.user2_id AS recommended_friend
FROM friendships f1
JOIN friendships f2 ON f1.user2_id = f2.user1_id
WHERE f1.user1_id = [target_user]
AND f2.user2_id != [target_user]
AND NOT EXISTS (
    SELECT 1 FROM friendships f3
    WHERE f3.user1_id = [target_user] AND f3.user2_id = f2.user2_id
)
AND NOT EXISTS (
    SELECT 1 FROM friendships f4
    WHERE f4.user1_id = f2.user2_id AND f4.user2_id = [target_user]
);
```

### 2. Friendship Acceptance Rate

**Question:** Calculate the acceptance rate of friend requests by date.

```sql
SELECT 
    fr.request_date,
    COUNT(fa.acceptance_id) AS accepted_requests,
    COUNT(fr.request_id) AS total_requests,
    ROUND(COUNT(fa.acceptance_id) * 100.0 / COUNT(fr.request_id), 2) AS acceptance_rate
FROM friend_requests fr
LEFT JOIN friend_acceptances fa 
    ON fr.request_id = fa.request_id
GROUP BY fr.request_date
ORDER BY fr.request_date;
```

### 3. Daily Active Users (DAU)

**Question:** Calculate the 7-day rolling average of daily active users.

```sql
SELECT 
    activity_date,
    COUNT(DISTINCT user_id) AS daily_active_users,
    AVG(COUNT(DISTINCT user_id)) OVER (
        ORDER BY activity_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS rolling_avg_7day
FROM user_activities
WHERE activity_date >= (SELECT MIN(activity_date) FROM user_activities) + INTERVAL '6 days'
GROUP BY activity_date
ORDER BY activity_date;
```

## Amazon Specific Questions

### 1. Product Recommendations

**Question:** For each product, find top 3 other products that are most frequently purchased together.

```sql
WITH product_pairs AS (
    SELECT 
        o1.product_id AS product1,
        o2.product_id AS product2,
        COUNT(*) AS purchase_frequency
    FROM order_items o1
    JOIN order_items o2 
        ON o1.order_id = o2.order_id
        AND o1.product_id < o2.product_id  -- Avoid duplicates
    GROUP BY o1.product_id, o2.product_id
)

SELECT 
    p.product_id,
    p.product_name,
    recommended.product_id AS recommended_product_id,
    recommended.product_name AS recommended_product_name,
    pp.purchase_frequency
FROM (
    SELECT 
        product1,
        product2,
        purchase_frequency,
        ROW_NUMBER() OVER (PARTITION BY product1 ORDER BY purchase_frequency DESC) AS rank
    FROM product_pairs
) ranked_pairs
JOIN products p ON ranked_pairs.product1 = p.product_id
JOIN products recommended ON ranked_pairs.product2 = recommended.product_id
WHERE ranked_pairs.rank <= 3
ORDER BY p.product_id, ranked_pairs.rank;
```

### 2. Customer Spending Analysis

**Question:** Identify customers who have consistently spent more each month for at least three consecutive months.

```sql
WITH monthly_spend AS (
    SELECT 
        customer_id,
        DATE_TRUNC('month', order_date) AS order_month,
        SUM(total_amount) AS monthly_spend
    FROM orders
    GROUP BY customer_id, DATE_TRUNC('month', order_date)
),
consecutive_increases AS (
    SELECT 
        ms1.customer_id,
        ms1.order_month
    FROM monthly_spend ms1
    JOIN monthly_spend ms2 
        ON ms1.customer_id = ms2.customer_id
        AND ms1.order_month = ms2.order_month + INTERVAL '1 month'
        AND ms1.monthly_spend > ms2.monthly_spend
    JOIN monthly_spend ms3
        ON ms2.customer_id = ms3.customer_id
        AND ms2.order_month = ms3.order_month + INTERVAL '1 month'
        AND ms2.monthly_spend > ms3.monthly_spend
)

SELECT DISTINCT c.customer_id, c.customer_name
FROM consecutive_increases ci
JOIN customers c ON ci.customer_id = c.customer_id;
```

## Google Specific Questions

### 1. Advertisers Performance

**Question:** Calculate the click-through rate (CTR) for each advertiser, by month.

```sql
SELECT 
    advertiser_id,
    DATE_TRUNC('month', impression_date) AS month,
    COUNT(DISTINCT CASE WHEN event_type = 'click' THEN impression_id END) AS clicks,
    COUNT(DISTINCT impression_id) AS impressions,
    ROUND(COUNT(DISTINCT CASE WHEN event_type = 'click' THEN impression_id END) * 100.0 / 
          NULLIF(COUNT(DISTINCT impression_id), 0), 2) AS ctr
FROM ad_impressions
GROUP BY advertiser_id, DATE_TRUNC('month', impression_date)
ORDER BY advertiser_id, month;
```

### 2. Search Query Analysis

**Question:** Find the top 10 search queries by volume in the past week, and compare their volume to the previous week.

```sql
WITH current_week AS (
    SELECT 
        search_query,
        COUNT(*) AS current_volume
    FROM search_logs
    WHERE search_date BETWEEN CURRENT_DATE - INTERVAL '7 days' AND CURRENT_DATE
    GROUP BY search_query
),
previous_week AS (
    SELECT 
        search_query,
        COUNT(*) AS previous_volume
    FROM search_logs
    WHERE search_date BETWEEN CURRENT_DATE - INTERVAL '14 days' AND CURRENT_DATE - INTERVAL '8 days'
    GROUP BY search_query
)

SELECT 
    cw.search_query,
    cw.current_volume,
    COALESCE(pw.previous_volume, 0) AS previous_volume,
    cw.current_volume - COALESCE(pw.previous_volume, 0) AS volume_change,
    CASE 
        WHEN pw.previous_volume IS NULL OR pw.previous_volume = 0 THEN NULL
        ELSE ROUND((cw.current_volume - pw.previous_volume) * 100.0 / pw.previous_volume, 2)
    END AS percent_change
FROM current_week cw
LEFT JOIN previous_week pw ON cw.search_query = pw.search_query
ORDER BY cw.current_volume DESC
LIMIT 10;
```

## Netflix Specific Questions

### 1. User Engagement

**Question:** Find the average watch time per user per day and identify users who have watched more than the average for 5 consecutive days.

```sql
WITH daily_watch_time AS (
    SELECT 
        user_id,
        watch_date,
        SUM(duration_minutes) AS total_watch_time
    FROM watch_history
    GROUP BY user_id, watch_date
),
avg_watch_time AS (
    SELECT 
        watch_date,
        AVG(total_watch_time) AS avg_watch_time
    FROM daily_watch_time
    GROUP BY watch_date
),
consecutive_days AS (
    SELECT 
        dwt.user_id,
        dwt.watch_date,
        dwt.total_watch_time,
        awt.avg_watch_time,
        COUNT(*) OVER (
            PARTITION BY dwt.user_id
            ORDER BY dwt.watch_date
            ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
        ) AS consecutive_count
    FROM daily_watch_time dwt
    JOIN avg_watch_time awt ON dwt.watch_date = awt.watch_date
    WHERE dwt.total_watch_time > awt.avg_watch_time
)

SELECT DISTINCT user_id
FROM consecutive_days
WHERE consecutive_count >= 5;
```

### 2. Content Recommendation

**Question:** For each user, identify the top 3 genres they watch most and the percentage of their total watch time for each genre.

```sql
WITH user_genre_watch_time AS (
    SELECT 
        w.user_id,
        c.genre,
        SUM(w.duration_minutes) AS genre_watch_time
    FROM watch_history w
    JOIN content c ON w.content_id = c.content_id
    GROUP BY w.user_id, c.genre
),
user_total_watch_time AS (
    SELECT 
        user_id,
        SUM(genre_watch_time) AS total_watch_time
    FROM user_genre_watch_time
    GROUP BY user_id
)

SELECT 
    ug.user_id,
    ug.genre,
    ug.genre_watch_time,
    ROUND(ug.genre_watch_time * 100.0 / ut.total_watch_time, 2) AS percentage_of_total,
    ROW_NUMBER() OVER (PARTITION BY ug.user_id ORDER BY ug.genre_watch_time DESC) AS genre_rank
FROM user_genre_watch_time ug
JOIN user_total_watch_time ut ON ug.user_id = ut.user_id
QUALIFY ROW_NUMBER() OVER (PARTITION BY ug.user_id ORDER BY ug.genre_watch_time DESC) <= 3
ORDER BY ug.user_id, genre_rank;
```

## Apple Specific Questions

### 1. App Store Analysis

**Question:** Find the correlation between app ratings and download counts.

```sql
SELECT 
    CORR(rating, download_count) AS correlation_coefficient,
    REGR_SLOPE(download_count, rating) AS regression_slope,
    REGR_INTERCEPT(download_count, rating) AS regression_intercept
FROM app_store_metrics;
```

### 2. Device Usage Patterns

**Question:** Identify users who use multiple Apple devices and analyze their usage patterns.

```sql
WITH multi_device_users AS (
    SELECT user_id, COUNT(DISTINCT device_type) as device_count
    FROM device_activities
    GROUP BY user_id
    HAVING COUNT(DISTINCT device_type) > 1
),
device_usage AS (
    SELECT 
        da.user_id,
        da.device_type,
        COUNT(*) AS activity_count,
        SUM(da.duration_minutes) AS total_usage,
        AVG(da.duration_minutes) AS avg_session_length
    FROM device_activities da
    JOIN multi_device_users mdu ON da.user_id = mdu.user_id
    GROUP BY da.user_id, da.device_type
)

SELECT 
    user_id,
    JSON_OBJECTAGG(device_type, 
        JSON_OBJECT(
            'activity_count', activity_count,
            'total_usage', total_usage,
            'avg_session_length', avg_session_length
        )
    ) AS device_usage_patterns
FROM device_usage
GROUP BY user_id;
```

## SQL Optimization Questions

### 1. Optimizing Slow Queries

**Question:** How would you optimize the following query?

```sql
-- Original slow query
SELECT 
    c.customer_name,
    o.order_id,
    o.order_date,
    SUM(oi.quantity * p.price) AS order_total
FROM customers c, orders o, order_items oi, products p
WHERE c.customer_id = o.customer_id
AND o.order_id = oi.order_id
AND oi.product_id = p.product_id
AND c.country = 'USA'
AND YEAR(o.order_date) = 2023
GROUP BY c.customer_name, o.order_id, o.order_date
ORDER BY order_total DESC;

-- Optimized query
SELECT 
    c.customer_name,
    o.order_id,
    o.order_date,
    SUM(oi.quantity * p.price) AS order_total
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE c.country = 'USA'
AND o.order_date BETWEEN '2023-01-01' AND '2023-12-31'
GROUP BY c.customer_name, o.order_id, o.order_date
ORDER BY order_total DESC;

-- Explanation of improvements:
-- 1. Used explicit JOIN syntax instead of comma-separated tables
-- 2. Changed YEAR() function to date range comparison (allows index usage)
-- 3. Added appropriate indexes:
--    - CREATE INDEX idx_customer_country ON customers(country);
--    - CREATE INDEX idx_order_date ON orders(order_date);
--    - CREATE INDEX idx_order_items_order ON order_items(order_id);
```

### 2. Index Selection

**Question:** For the following query, what indexes would you create to optimize its performance?

```sql
SELECT 
    u.username,
    COUNT(p.post_id) AS post_count
FROM users u
LEFT JOIN posts p ON u.user_id = p.user_id
WHERE u.account_status = 'active'
AND u.join_date >= '2023-01-01'
AND (p.post_date >= '2023-06-01' OR p.post_id IS NULL)
GROUP BY u.username
HAVING COUNT(p.post_id) < 5
ORDER BY post_count, u.username;

-- Optimal indexes:
-- 1. Composite index on users table:
--    CREATE INDEX idx_users_status_join ON users(account_status, join_date, user_id, username);
-- 
-- 2. Index on posts table:
--    CREATE INDEX idx_posts_user_date ON posts(user_id, post_date);
--
-- Explanation:
-- - The index on users covers the WHERE conditions and includes user_id for the join
-- - The index on posts helps with the join and the post_date filter
-- - Both indexes include columns needed for the rest of the query to create covering indexes
```

## Advanced SQL Concepts

### 1. Recursive CTEs

**Question:** Write a query to find all employees who report directly or indirectly to a specific manager.

```sql
WITH RECURSIVE subordinates AS (
    -- Base case: direct reports
    SELECT 
        employee_id, 
        first_name, 
        last_name, 
        manager_id, 
        1 AS level
    FROM employees
    WHERE manager_id = [target_manager_id]
    
    UNION ALL
    
    -- Recursive case: reports of reports
    SELECT 
        e.employee_id, 
        e.first_name, 
        e.last_name, 
        e.manager_id, 
        s.level + 1
    FROM employees e
    JOIN subordinates s ON e.manager_id = s.employee_id
)

SELECT 
    employee_id, 
    first_name, 
    last_name, 
    level
FROM subordinates
ORDER BY level, first_name, last_name;
```

### 2. Window Functions

**Question:** Calculate the rolling 3-month average revenue by product category.

```sql
SELECT 
    category_id,
    category_name,
    month_date,
    monthly_revenue,
    AVG(monthly_revenue) OVER (
        PARTITION BY category_id 
        ORDER BY month_date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS rolling_3month_avg
FROM (
    SELECT 
        p.category_id,
        c.category_name,
        DATE_TRUNC('month', o.order_date) AS month_date,
        SUM(oi.quantity * p.price) AS monthly_revenue
    FROM order_items oi
    JOIN products p ON oi.product_id = p.product_id
    JOIN categories c ON p.category_id = c.category_id
    JOIN orders o ON oi.order_id = o.order_id
    GROUP BY p.category_id, c.category_name, DATE_TRUNC('month', o.order_date)
) monthly_revenue
ORDER BY category_id, month_date;
```

### 3. Pivot and Unpivot

**Question:** Create a pivot table showing quarterly sales by product category.

```sql
-- PostgreSQL using crosstab
SELECT * FROM crosstab(
    'SELECT 
        c.category_name,
        EXTRACT(quarter FROM o.order_date) AS quarter,
        SUM(oi.quantity * p.price) AS quarterly_sales
     FROM order_items oi
     JOIN products p ON oi.product_id = p.product_id
     JOIN categories c ON p.category_id = c.category_id
     JOIN orders o ON oi.order_id = o.order_id
     WHERE EXTRACT(year FROM o.order_date) = 2023
     GROUP BY c.category_name, EXTRACT(quarter FROM o.order_date)
     ORDER BY c.category_name, quarter',
    'SELECT generate_series(1,4)'
) AS (
    category_name VARCHAR,
    "Q1" NUMERIC,
    "Q2" NUMERIC,
    "Q3" NUMERIC,
    "Q4" NUMERIC
);

-- SQL Server pivot
SELECT 
    category_name,
    [1] AS Q1,
    [2] AS Q2,
    [3] AS Q3,
    [4] AS Q4
FROM (
    SELECT 
        c.category_name,
        DATEPART(quarter, o.order_date) AS quarter,
        SUM(oi.quantity * p.price) AS quarterly_sales
    FROM order_items oi
    JOIN products p ON oi.product_id = p.product_id
    JOIN categories c ON p.category_id = c.category_id
    JOIN orders o ON oi.order_id = o.order_id
    WHERE YEAR(o.order_date) = 2023
    GROUP BY c.category_name, DATEPART(quarter, o.order_date)
) src
PIVOT (
    SUM(quarterly_sales)
    FOR quarter IN ([1], [2], [3], [4])
) AS pvt;
```

## Tips for SQL Interview Success

1. **Understand the Problem Completely**
   - Ask clarifying questions
   - Confirm expectations for the output
   - Discuss any assumptions you're making

2. **Start with Simple Cases**
   - Begin with the basic structure
   - Test with simple examples
   - Incrementally add complexity

3. **Think About Edge Cases**
   - What happens with NULL values?
   - How to handle duplicate records?
   - Are there date/time zone considerations?

4. **Optimize Your Solution**
   - Consider indexes that would help
   - Avoid unnecessary joins and subqueries
   - Explain your optimization choices

5. **Practice Common Patterns**
   - Window functions for analytical queries
   - CTEs for complex queries
   - Proper join techniques
   - Aggregation and filtering
   
6. **Verbalize Your Thought Process**
   - Explain your approach as you work
   - Discuss alternative solutions
   - Highlight trade-offs between different approaches 