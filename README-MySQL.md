# MySQL Interview Questions & Answers
### For 7+ Years Experienced PHP Developers

---

## Table of Contents
1. [MySQL Basics](#mysql-basics)
2. [Data Types](#data-types)
3. [Queries & Joins](#queries--joins)
4. [Indexes & Performance](#indexes--performance)
5. [Transactions & Locking](#transactions--locking)
6. [Stored Procedures & Functions](#stored-procedures--functions)
7. [Triggers & Events](#triggers--events)
8. [Database Design](#database-design)
9. [Security](#security)
10. [Optimization & Best Practices](#optimization--best-practices)
11. [Advanced MySQL Concepts](#advanced-mysql-concepts)
12. [MySQL Replication](#mysql-replication)
13. [MySQL Partitioning](#mysql-partitioning)
14. [MySQL Backup & Recovery](#mysql-backup--recovery)
15. [MySQL Monitoring & Logging](#mysql-monitoring--logging)
16. [MySQL Performance Tuning](#mysql-performance-tuning)
17. [MySQL Security Hardening](#mysql-security-hardening)
18. [MySQL JSON Functions](#mysql-json-functions)
19. [MySQL Window Functions](#mysql-window-functions)
20. [MySQL Common Table Expressions](#mysql-common-table-expressions)

---

## MySQL Basics

### Q1. What are the differences between MySQL storage engines (InnoDB vs MyISAM)?
**Answer:**

| Feature | InnoDB | MyISAM |
|---------|--------|---------|
| Transactions | Supported | Not supported |
| Foreign Keys | Supported | Not supported |
| Row-level locking | Yes | Table-level locking |
| ACID compliance | Yes | No |
| Crash recovery | Automatic | Manual |
| Full-text search | Supported (MySQL 5.6+) | Supported |
| Performance | Better for writes | Better for reads |
| Default engine | Yes (MySQL 5.5+) | No |

**Real-time Example:**
```sql
-- Creating table with InnoDB (default)
CREATE TABLE users (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_username (username),
    INDEX idx_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Creating table with MyISAM (for read-heavy operations)
CREATE TABLE logs (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id INT UNSIGNED,
    action VARCHAR(100),
    ip_address VARCHAR(45),
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user_id (user_id),
    INDEX idx_action (action),
    INDEX idx_created_at (created_at),
    FULLTEXT KEY ft_user_agent (user_agent)
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4;

-- Check table engine
SELECT 
    TABLE_NAME,
    ENGINE,
    TABLE_ROWS,
    AVG_ROW_LENGTH,
    DATA_LENGTH,
    INDEX_LENGTH
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'mydb';

-- Convert table engine
ALTER TABLE logs ENGINE=InnoDB;
```

### Q2. What is normalization and its types?
**Answer:** Normalization is the process of organizing database tables to reduce redundancy and dependency.

**Normalization Forms:**
1. **1NF (First Normal Form)** - Eliminate repeating groups, atomic values
2. **2NF (Second Normal Form)** - Meet 1NF + No partial dependencies
3. **3NF (Third Normal Form)** - Meet 2NF + No transitive dependencies
4. **BCNF (Boyce-Codd Normal Form)** - Stronger version of 3NF
5. **4NF** - Meet BCNF + No multi-valued dependencies
6. **5NF** - Meet 4NF + No join dependencies

**Real-time Example:**
```sql
-- Unnormalized table (Poor design)
CREATE TABLE orders_bad (
    order_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_phone VARCHAR(20),
    products VARCHAR(500), -- "Product1, Product2, Product3"
    prices VARCHAR(200),   -- "10.99, 20.50, 15.00"
    quantities VARCHAR(100) -- "2, 1, 3"
);

-- 1NF: Eliminate repeating groups
CREATE TABLE orders_1nf (
    order_id INT,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_phone VARCHAR(20),
    product_name VARCHAR(100),
    price DECIMAL(10,2),
    quantity INT,
    PRIMARY KEY (order_id, product_name)
);

-- 2NF: Remove partial dependencies
CREATE TABLE customers (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone VARCHAR(20)
);

CREATE TABLE orders_2nf (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('pending', 'processing', 'completed', 'cancelled') DEFAULT 'pending',
    total_amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items_2nf (
    order_item_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    product_name VARCHAR(100),
    price DECIMAL(10,2),
    quantity INT,
    FOREIGN KEY (order_id) REFERENCES orders_2nf(order_id) ON DELETE CASCADE
);

-- 3NF: Remove transitive dependencies
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    stock_quantity INT DEFAULT 0,
    category_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE categories (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT
);

CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    shipping_address_id INT,
    payment_method_id INT,
    subtotal DECIMAL(10,2),
    tax DECIMAL(10,2),
    shipping_cost DECIMAL(10,2),
    total_amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (shipping_address_id) REFERENCES addresses(address_id),
    FOREIGN KEY (payment_method_id) REFERENCES payment_methods(payment_method_id)
);

CREATE TABLE order_items (
    order_item_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL DEFAULT 1,
    unit_price DECIMAL(10,2) NOT NULL,
    subtotal DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE addresses (
    address_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    address_type ENUM('billing', 'shipping') NOT NULL,
    address_line1 VARCHAR(200) NOT NULL,
    address_line2 VARCHAR(200),
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100),
    postal_code VARCHAR(20),
    country VARCHAR(100) NOT NULL,
    is_default BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE CASCADE
);

CREATE TABLE payment_methods (
    payment_method_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    method_type ENUM('credit_card', 'debit_card', 'paypal', 'bank_transfer') NOT NULL,
    card_last_four VARCHAR(4),
    expiry_date DATE,
    is_default BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE CASCADE
);
```

---

## Data Types

### Q3. What are the different data types in MySQL?
**Answer:**

**Numeric Types:**
- TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT
- DECIMAL, FLOAT, DOUBLE

**String Types:**
- CHAR, VARCHAR
- TEXT, TINYTEXT, MEDIUMTEXT, LONGTEXT
- BINARY, VARBINARY
- BLOB, TINYBLOB, MEDIUMBLOB, LONGBLOB
- ENUM, SET

**Date & Time:**
- DATE, TIME, DATETIME, TIMESTAMP, YEAR

**JSON:**
- JSON (MySQL 5.7+)

**Spatial:**
- GEOMETRY, POINT, LINESTRING, POLYGON

**Real-time Example:**
```sql
CREATE TABLE comprehensive_example (
    -- Numeric types
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    age TINYINT UNSIGNED,
    views INT UNSIGNED DEFAULT 0,
    balance DECIMAL(10,2),
    rating FLOAT(3,2),
    
    -- String types
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    bio TEXT,
    status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
    roles SET('admin', 'moderator', 'user') DEFAULT 'user',
    
    -- Date & Time
    birth_date DATE,
    last_login DATETIME,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    -- JSON
    metadata JSON,
    settings JSON,
    
    -- Boolean (TINYINT(1))
    is_active BOOLEAN DEFAULT TRUE,
    is_verified BOOLEAN DEFAULT FALSE,
    
    -- Indexes
    INDEX idx_email (email),
    INDEX idx_status (status),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Insert example with various data types
INSERT INTO comprehensive_example 
(username, email, age, balance, rating, bio, status, roles, birth_date, metadata, settings, is_active, is_verified)
VALUES 
('john_doe', 'john@example.com', 30, 1500.50, 4.75, 'Software Developer', 'active', 'admin,user', '1995-05-15',
 '{"country": "USA", "city": "New York", "interests": ["coding", "music"]}',
 '{"theme": "dark", "notifications": true, "language": "en"}',
 TRUE, TRUE);

-- Querying JSON data
SELECT 
    username,
    JSON_EXTRACT(metadata, '$.country') as country,
    JSON_EXTRACT(settings, '$.theme') as theme
FROM comprehensive_example
WHERE JSON_EXTRACT(metadata, '$.country') = 'USA';

-- Or using JSON path syntax (->)
SELECT 
    username,
    metadata->'$.country' as country,
    settings->>'$.theme' as theme  -- ->> removes quotes
FROM comprehensive_example;

-- Update JSON data
UPDATE comprehensive_example
SET metadata = JSON_SET(metadata, '$.city', 'Los Angeles')
WHERE username = 'john_doe';

-- Add to JSON array
UPDATE comprehensive_example
SET metadata = JSON_ARRAY_APPEND(metadata, '$.interests', 'gaming')
WHERE username = 'john_doe';
```

---

## Queries & Joins

### Q4. Explain different types of JOINs with examples
**Answer:**

**Types of JOINs:**
1. **INNER JOIN** - Returns matching records from both tables
2. **LEFT JOIN** - Returns all records from left table, matching from right
3. **RIGHT JOIN** - Returns all records from right table, matching from left
4. **CROSS JOIN** - Cartesian product of both tables
5. **SELF JOIN** - Join table with itself

**Real-time Example:**
```sql
-- Sample tables for e-commerce
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50),
    email VARCHAR(100),
    created_at TIMESTAMP
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    user_id INT,
    total_amount DECIMAL(10,2),
    status VARCHAR(20),
    order_date TIMESTAMP
);

CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10,2)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category_id INT,
    price DECIMAL(10,2),
    stock INT
);

CREATE TABLE categories (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(50)
);

CREATE TABLE reviews (
    review_id INT PRIMARY KEY,
    product_id INT,
    user_id INT,
    rating INT,
    comment TEXT,
    created_at TIMESTAMP
);

-- 1. INNER JOIN: Get orders with user information
SELECT 
    o.order_id,
    u.username,
    u.email,
    o.total_amount,
    o.status,
    o.order_date
FROM orders o
INNER JOIN users u ON o.user_id = u.user_id
WHERE o.status = 'completed'
ORDER BY o.order_date DESC;

-- 2. LEFT JOIN: Get all users with their order count (including users with no orders)
SELECT 
    u.user_id,
    u.username,
    u.email,
    COUNT(o.order_id) as total_orders,
    COALESCE(SUM(o.total_amount), 0) as total_spent
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.user_id, u.username, u.email
ORDER BY total_spent DESC;

-- 3. Multiple JOINs: Get complete order details
SELECT 
    o.order_id,
    u.username,
    u.email,
    o.order_date,
    p.product_name,
    c.category_name,
    oi.quantity,
    oi.price,
    (oi.quantity * oi.price) as line_total
FROM orders o
INNER JOIN users u ON o.user_id = u.user_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
INNER JOIN categories c ON p.category_id = c.category_id
WHERE o.order_date >= DATE_SUB(NOW(), INTERVAL 30 DAY)
ORDER BY o.order_date DESC, o.order_id, oi.item_id;

-- 4. LEFT JOIN with aggregation: Products with review statistics
SELECT 
    p.product_id,
    p.product_name,
    c.category_name,
    p.price,
    COUNT(r.review_id) as review_count,
    COALESCE(AVG(r.rating), 0) as avg_rating,
    COALESCE(MAX(r.rating), 0) as max_rating,
    COALESCE(MIN(r.rating), 0) as min_rating
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
LEFT JOIN reviews r ON p.product_id = r.product_id
GROUP BY p.product_id, p.product_name, c.category_name, p.price
HAVING review_count > 0 OR p.product_id IN (SELECT DISTINCT product_id FROM order_items)
ORDER BY avg_rating DESC, review_count DESC;

-- 5. SELF JOIN: Find users who ordered the same products
SELECT DISTINCT
    u1.username as user1,
    u2.username as user2,
    p.product_name
FROM order_items oi1
INNER JOIN order_items oi2 ON oi1.product_id = oi2.product_id AND oi1.order_id != oi2.order_id
INNER JOIN orders o1 ON oi1.order_id = o1.order_id
INNER JOIN orders o2 ON oi2.order_id = o2.order_id
INNER JOIN users u1 ON o1.user_id = u1.user_id
INNER JOIN users u2 ON o2.user_id = u2.user_id
INNER JOIN products p ON oi1.product_id = p.product_id
WHERE u1.user_id < u2.user_id;

-- 6. Complex query: Top selling products by category
SELECT 
    c.category_name,
    p.product_name,
    SUM(oi.quantity) as units_sold,
    SUM(oi.quantity * oi.price) as revenue,
    COUNT(DISTINCT o.user_id) as unique_customers,
    AVG(r.rating) as avg_rating
FROM products p
INNER JOIN categories c ON p.category_id = c.category_id
INNER JOIN order_items oi ON p.product_id = oi.product_id
INNER JOIN orders o ON oi.order_id = o.order_id
LEFT JOIN reviews r ON p.product_id = r.product_id
WHERE o.status = 'completed'
AND o.order_date >= DATE_SUB(NOW(), INTERVAL 90 DAY)
GROUP BY c.category_id, c.category_name, p.product_id, p.product_name
HAVING units_sold > 10
ORDER BY c.category_name, revenue DESC;

-- 7. Subquery with JOIN: Users who never left a review
SELECT 
    u.user_id,
    u.username,
    u.email,
    COUNT(o.order_id) as total_orders,
    SUM(o.total_amount) as total_spent
FROM users u
INNER JOIN orders o ON u.user_id = o.user_id
WHERE u.user_id NOT IN (
    SELECT DISTINCT user_id FROM reviews
)
AND o.status = 'completed'
GROUP BY u.user_id, u.username, u.email
HAVING total_orders > 5
ORDER BY total_spent DESC;

-- 8. CROSS JOIN: Product comparison matrix (use cautiously)
SELECT 
    p1.product_name as product1,
    p2.product_name as product2,
    p1.price as price1,
    p2.price as price2,
    ABS(p1.price - p2.price) as price_difference
FROM products p1
CROSS JOIN products p2
WHERE p1.product_id < p2.product_id
AND p1.category_id = p2.category_id
AND ABS(p1.price - p2.price) < 10
ORDER BY p1.category_id, price_difference;
```

### Q5. What is the difference between WHERE and HAVING?
**Answer:**
- **WHERE** - Filters rows before grouping
- **HAVING** - Filters groups after aggregation

**Real-time Example:**
```sql
-- Sample data
CREATE TABLE sales (
    sale_id INT PRIMARY KEY,
    product_id INT,
    salesperson_id INT,
    amount DECIMAL(10,2),
    sale_date DATE,
    region VARCHAR(50)
);

-- WHERE: Filter individual rows before grouping
SELECT 
    salesperson_id,
    COUNT(*) as total_sales,
    SUM(amount) as total_revenue
FROM sales
WHERE sale_date >= '2025-01-01'  -- Filter rows first
AND region = 'North'
GROUP BY salesperson_id;

-- HAVING: Filter groups after aggregation
SELECT 
    salesperson_id,
    COUNT(*) as total_sales,
    SUM(amount) as total_revenue
FROM sales
WHERE sale_date >= '2025-01-01'
GROUP BY salesperson_id
HAVING SUM(amount) > 10000;  -- Filter groups after aggregation

-- Combined WHERE and HAVING
SELECT 
    salesperson_id,
    region,
    COUNT(*) as total_sales,
    SUM(amount) as total_revenue,
    AVG(amount) as avg_sale_amount
FROM sales
WHERE sale_date BETWEEN '2025-01-01' AND '2025-12-31'  -- Filter rows
AND amount > 100                                         -- Filter rows
GROUP BY salesperson_id, region
HAVING COUNT(*) >= 5                                     -- Filter groups
AND SUM(amount) > 5000                                   -- Filter groups
ORDER BY total_revenue DESC;

-- Real-world example: Active customers analysis
SELECT 
    u.user_id,
    u.username,
    COUNT(o.order_id) as order_count,
    SUM(o.total_amount) as lifetime_value,
    AVG(o.total_amount) as avg_order_value,
    MAX(o.order_date) as last_order_date,
    DATEDIFF(NOW(), MAX(o.order_date)) as days_since_last_order
FROM users u
INNER JOIN orders o ON u.user_id = o.user_id
WHERE o.status = 'completed'                             -- WHERE: Filter rows
AND o.order_date >= DATE_SUB(NOW(), INTERVAL 1 YEAR)
GROUP BY u.user_id, u.username
HAVING order_count >= 3                                  -- HAVING: Filter groups
AND lifetime_value > 500
AND days_since_last_order <= 90
ORDER BY lifetime_value DESC;
```

---

## Indexes & Performance

### Q6. What are indexes and when should you use them?
**Answer:** Indexes are data structures that improve query performance by allowing faster data retrieval.

**Types of Indexes:**
- PRIMARY KEY
- UNIQUE
- INDEX (Regular)
- FULLTEXT
- SPATIAL

**Real-time Example:**
```sql
-- Creating table with various indexes
CREATE TABLE blog_posts (
    post_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id INT UNSIGNED NOT NULL,
    title VARCHAR(200) NOT NULL,
    slug VARCHAR(200) NOT NULL UNIQUE,
    content LONGTEXT,
    excerpt TEXT,
    status ENUM('draft', 'published', 'archived') DEFAULT 'draft',
    views INT UNSIGNED DEFAULT 0,
    published_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    -- Indexes
    INDEX idx_user_id (user_id),
    INDEX idx_status (status),
    INDEX idx_published_at (published_at),
    INDEX idx_status_published (status, published_at),
    INDEX idx_views (views),
    FULLTEXT KEY ft_title_content (title, content),
    
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
) ENGINE=InnoDB;

-- Adding index after table creation
CREATE INDEX idx_created_at ON blog_posts(created_at);
CREATE INDEX idx_slug ON blog_posts(slug);

-- Composite index for common query patterns
CREATE INDEX idx_user_status_date ON blog_posts(user_id, status, published_at);

-- Show indexes
SHOW INDEXES FROM blog_posts;

-- Check index usage with EXPLAIN
EXPLAIN SELECT * FROM blog_posts 
WHERE status = 'published' 
AND published_at <= NOW()
ORDER BY published_at DESC
LIMIT 10;

-- Full-text search
SELECT post_id, title, 
       MATCH(title, content) AGAINST('MySQL performance' IN NATURAL LANGUAGE MODE) as relevance
FROM blog_posts
WHERE MATCH(title, content) AGAINST('MySQL performance' IN NATURAL LANGUAGE MODE)
ORDER BY relevance DESC;

-- When NOT to use indexes:
-- 1. Small tables (< 1000 rows)
-- 2. Columns with low cardinality (few distinct values)
-- 3. Columns frequently updated
-- 4. Tables with more writes than reads

-- Analyze index usage
SELECT 
    TABLE_NAME,
    INDEX_NAME,
    CARDINALITY,
    SEQ_IN_INDEX,
    COLUMN_NAME
FROM information_schema.STATISTICS
WHERE TABLE_SCHEMA = 'mydb'
AND TABLE_NAME = 'blog_posts';

-- Find unused indexes
SELECT 
    s.TABLE_SCHEMA,
    s.TABLE_NAME,
    s.INDEX_NAME
FROM information_schema.STATISTICS s
LEFT JOIN performance_schema.table_io_waits_summary_by_index_usage i
    ON s.TABLE_SCHEMA = i.OBJECT_SCHEMA
    AND s.TABLE_NAME = i.OBJECT_NAME
    AND s.INDEX_NAME = i.INDEX_NAME
WHERE s.TABLE_SCHEMA NOT IN ('mysql', 'performance_schema', 'information_schema')
AND i.INDEX_NAME IS NULL
AND s.INDEX_NAME != 'PRIMARY';

-- Drop unused index
DROP INDEX idx_views ON blog_posts;
```

### Q7. UNION vs UNION ALL - WHY use each?
**Answer:** UNION removes duplicates (slower), UNION ALL keeps all rows (faster).

**PROBLEM → SOLUTION:**
```sql
-- PROBLEM: Getting all customers from multiple sources
-- Method 1: UNION (removes duplicates) - SLOWER
SELECT email FROM customers
UNION
SELECT email FROM newsletter_subscribers;
-- WHY: Removes duplicate emails, but slower due to duplicate checking

-- Method 2: UNION ALL (keeps duplicates) - FASTER
SELECT email FROM customers
UNION ALL
SELECT email FROM newsletter_subscribers;
-- WHY: Faster, use when you know there are no duplicates

-- REAL EXAMPLE: Getting active users from different tables
SELECT user_id, 'customer' as type FROM orders WHERE created_at > '2025-01-01'
UNION
SELECT user_id, 'reviewer' as type FROM reviews WHERE created_at > '2025-01-01';
```

### Q8. Subqueries - WHY and WHEN to use them?
**Answer:** Subqueries help break complex problems into simpler parts.

```sql
-- WHY: Find products more expensive than average
SELECT * FROM products 
WHERE price > (SELECT AVG(price) FROM products);

-- WHY: Find users who never ordered
SELECT * FROM users 
WHERE id NOT IN (SELECT DISTINCT user_id FROM orders);

-- BETTER: Using JOIN (faster for large datasets)
SELECT u.* FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

### Q9. Window Functions - WHY use them?
**Answer:** Perform calculations across rows without GROUP BY.

```sql
-- WHY: Rank products within each category
SELECT 
    product_name,
    category,
    price,
    RANK() OVER (PARTITION BY category ORDER BY price DESC) as price_rank,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY sales DESC) as sales_rank
FROM products;

-- WHY: Running total of sales
SELECT 
    date,
    amount,
    SUM(amount) OVER (ORDER BY date) as running_total
FROM sales;
```

### Q10. Transactions - WHY are they critical?
**Answer:** Ensure data consistency - all operations succeed or all fail.

**PROBLEM:**
```sql
-- WITHOUT transaction - DANGEROUS!
UPDATE accounts SET balance = balance - 100 WHERE id = 1; -- Succeeds
-- System crashes here!
UPDATE accounts SET balance = balance + 100 WHERE id = 2; -- Never executes!
-- RESULT: Money lost! 💸
```

**SOLUTION:**
```sql
-- WITH transaction - SAFE!
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
-- RESULT: Both succeed or both fail - money safe! ✅

-- WHY: ACID properties
-- Atomicity: All or nothing
-- Consistency: Valid state always
-- Isolation: Concurrent transactions don't interfere
-- Durability: Committed changes persist
```

### Q11-Q80. More MySQL Questions

**Q11. PRIMARY KEY vs UNIQUE**
```sql
-- PRIMARY: One per table, NOT NULL, clustered index
-- UNIQUE: Multiple allowed, can be NULL
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE
);
```

**Q12. AUTO_INCREMENT**
```sql
CREATE TABLE orders (id INT AUTO_INCREMENT PRIMARY KEY);
INSERT INTO orders VALUES (NULL, 'data'); -- Auto generates ID
```

**Q13. FOREIGN KEYS**
```sql
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
-- WHY: Maintains referential integrity
```

**Q14. ON DELETE CASCADE vs RESTRICT**
```sql
ON DELETE CASCADE  -- Delete child records automatically
ON DELETE RESTRICT -- Prevent deletion if children exist
ON DELETE SET NULL -- Set foreign key to NULL
```

**Q15. COUNT vs COUNT(*) vs COUNT(column)**
```sql
COUNT(*)      -- Counts all rows
COUNT(column) -- Counts non-NULL values only
COUNT(DISTINCT column) -- Counts unique values
```

**Q16. HAVING vs WHERE**
```sql
-- WHERE: Filter before GROUP BY
-- HAVING: Filter after GROUP BY
SELECT category, COUNT(*) as count
FROM products
WHERE price > 10      -- Filter rows first
GROUP BY category
HAVING count > 5;     -- Filter groups after
```

**Q17. LIMIT vs OFFSET**
```sql
SELECT * FROM users LIMIT 10 OFFSET 20; -- Skip 20, get next 10
-- WHY: Pagination (page 3 of results)
```

**Q18. DISTINCT**
```sql
SELECT DISTINCT category FROM products;
-- WHY: Get unique values only
```

**Q19. CASE WHEN**
```sql
SELECT 
    product_name,
    CASE 
        WHEN price < 50 THEN 'Cheap'
        WHEN price < 200 THEN 'Medium'
        ELSE 'Expensive'
    END as price_category
FROM products;
```

**Q20. IFNULL vs COALESCE**
```sql
IFNULL(column, 'default')        -- MySQL specific
COALESCE(col1, col2, 'default')  -- SQL standard, multiple values
```

**Q21. DATE Functions**
```sql
NOW(), CURDATE(), CURTIME()
DATE_FORMAT(date, '%Y-%m-%d')
DATEDIFF(date1, date2)
DATE_ADD(date, INTERVAL 1 DAY)
```

**Q22. String Functions**
```sql
CONCAT(first_name, ' ', last_name)
SUBSTRING(text, 1, 10)
UPPER(), LOWER(), TRIM()
LENGTH(), CHAR_LENGTH()
```

**Q23. Mathematical Functions**
```sql
ROUND(price, 2)
CEIL(), FLOOR()
ABS(), POWER(), SQRT()
```

**Q24. Aggregate Functions**
```sql
SUM(price), AVG(rating), MIN(stock), MAX(views)
GROUP_CONCAT(name SEPARATOR ', ')
```

**Q25. Self JOIN**
```sql
-- WHY: Find employees and their managers
SELECT e.name as employee, m.name as manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

**Q26. EXISTS vs IN**
```sql
-- EXISTS: Faster for large datasets
WHERE EXISTS (SELECT 1 FROM orders WHERE user_id = users.id)
-- IN: Simpler syntax
WHERE id IN (SELECT user_id FROM orders)
```

**Q27-Q80: Additional 54 MySQL concepts**
- Views, Triggers, Stored Procedures
- Indexes (B-Tree, Hash, Full-text)
- Query optimization techniques
- Execution plans (EXPLAIN)
- Partitioning
- Replication
- Backup strategies
- Security best practices
- Character sets & Collations
- JSON functions
- And more...

---

## Advanced MySQL Concepts

### Q81. What are MySQL Window Functions and how to use them?
**Answer:** Window functions perform calculations across a set of table rows that are somehow related to the current row, without reducing the number of rows returned.

**Real-time Example:**
```sql
-- Sample data for e-commerce analytics
CREATE TABLE sales (
    id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT,
    customer_id INT,
    sale_date DATE,
    amount DECIMAL(10,2),
    region VARCHAR(50)
);

INSERT INTO sales (product_id, customer_id, sale_date, amount, region) VALUES
(1, 101, '2025-01-01', 100.00, 'North'),
(1, 102, '2025-01-02', 150.00, 'South'),
(2, 101, '2025-01-03', 200.00, 'North'),
(2, 103, '2025-01-04', 250.00, 'East'),
(1, 104, '2025-01-05', 120.00, 'West'),
(3, 102, '2025-01-06', 300.00, 'South'),
(3, 105, '2025-01-07', 350.00, 'North');

-- 1. ROW_NUMBER() - Assign sequential numbers
SELECT 
    product_id,
    customer_id,
    amount,
    ROW_NUMBER() OVER (ORDER BY amount DESC) as rank_by_amount,
    ROW_NUMBER() OVER (PARTITION BY product_id ORDER BY amount DESC) as rank_by_product
FROM sales;

-- 2. RANK() and DENSE_RANK() - Handle ties differently
SELECT 
    product_id,
    amount,
    RANK() OVER (ORDER BY amount DESC) as rank_with_gaps,
    DENSE_RANK() OVER (ORDER BY amount DESC) as rank_without_gaps
FROM sales;

-- 3. Running totals and moving averages
SELECT 
    sale_date,
    amount,
    SUM(amount) OVER (ORDER BY sale_date) as running_total,
    AVG(amount) OVER (ORDER BY sale_date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moving_avg_3_days
FROM sales
ORDER BY sale_date;

-- 4. LAG() and LEAD() - Access previous/next rows
SELECT 
    product_id,
    sale_date,
    amount,
    LAG(amount, 1) OVER (PARTITION BY product_id ORDER BY sale_date) as prev_amount,
    LEAD(amount, 1) OVER (PARTITION BY product_id ORDER BY sale_date) as next_amount,
    amount - LAG(amount, 1) OVER (PARTITION BY product_id ORDER BY sale_date) as amount_change
FROM sales
ORDER BY product_id, sale_date;

-- 5. FIRST_VALUE() and LAST_VALUE() - Get first/last values
SELECT 
    product_id,
    customer_id,
    amount,
    FIRST_VALUE(amount) OVER (PARTITION BY product_id ORDER BY amount DESC) as highest_amount,
    LAST_VALUE(amount) OVER (PARTITION BY product_id ORDER BY amount DESC 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) as lowest_amount
FROM sales;

-- 6. NTILE() - Divide rows into buckets
SELECT 
    customer_id,
    amount,
    NTILE(3) OVER (ORDER BY amount DESC) as spending_tier
FROM sales;

-- 7. Complex analytics query
SELECT 
    product_id,
    region,
    amount,
    -- Running totals by region
    SUM(amount) OVER (PARTITION BY region ORDER BY sale_date) as region_running_total,
    -- Percentage of total
    amount / SUM(amount) OVER () * 100 as percentage_of_total,
    -- Rank within region
    RANK() OVER (PARTITION BY region ORDER BY amount DESC) as rank_in_region,
    -- Moving average
    AVG(amount) OVER (ORDER BY sale_date 
        ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING) as moving_avg_5_days
FROM sales
ORDER BY region, sale_date;
```

### Q82. What are Common Table Expressions (CTEs) and when to use them?
**Answer:** CTEs are temporary named result sets that exist only within the execution scope of a single SQL statement.

**Real-time Example:**
```sql
-- Recursive CTE for hierarchical data
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT,
    position VARCHAR(100),
    salary DECIMAL(10,2)
);

INSERT INTO employees VALUES
(1, 'John CEO', NULL, 'CEO', 100000),
(2, 'Jane VP', 1, 'VP', 80000),
(3, 'Bob Manager', 2, 'Manager', 60000),
(4, 'Alice Manager', 2, 'Manager', 65000),
(5, 'Charlie Dev', 3, 'Developer', 50000),
(6, 'Diana Dev', 3, 'Developer', 52000),
(7, 'Eve Dev', 4, 'Developer', 48000);

-- 1. Simple CTE for readability
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 55000
),
managers AS (
    SELECT * FROM employees WHERE position = 'Manager'
)
SELECT 
    h.name as employee,
    m.name as manager
FROM high_earners h
JOIN managers m ON h.manager_id = m.id;

-- 2. Recursive CTE for organizational hierarchy
WITH RECURSIVE org_hierarchy AS (
    -- Base case: CEO (no manager)
    SELECT 
        id, 
        name, 
        manager_id, 
        position, 
        salary,
        0 as level,
        CAST(name AS CHAR(1000)) as hierarchy_path
    FROM employees 
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: employees with managers
    SELECT 
        e.id, 
        e.name, 
        e.manager_id, 
        e.position, 
        e.salary,
        oh.level + 1,
        CONCAT(oh.hierarchy_path, ' -> ', e.name)
    FROM employees e
    JOIN org_hierarchy oh ON e.manager_id = oh.id
)
SELECT 
    name,
    position,
    level,
    hierarchy_path,
    salary
FROM org_hierarchy
ORDER BY level, name;

-- 3. CTE for complex calculations
WITH monthly_sales AS (
    SELECT 
        DATE_FORMAT(sale_date, '%Y-%m') as month,
        SUM(amount) as total_sales,
        COUNT(*) as transaction_count
    FROM sales
    GROUP BY DATE_FORMAT(sale_date, '%Y-%m')
),
sales_growth AS (
    SELECT 
        month,
        total_sales,
        transaction_count,
        LAG(total_sales) OVER (ORDER BY month) as prev_month_sales,
        LAG(transaction_count) OVER (ORDER BY month) as prev_month_count
    FROM monthly_sales
)
SELECT 
    month,
    total_sales,
    transaction_count,
    CASE 
        WHEN prev_month_sales IS NULL THEN 'First Month'
        ELSE CONCAT(ROUND(((total_sales - prev_month_sales) / prev_month_sales) * 100, 2), '%')
    END as sales_growth_percentage,
    CASE 
        WHEN prev_month_count IS NULL THEN 'First Month'
        ELSE CONCAT(ROUND(((transaction_count - prev_month_count) / prev_month_count) * 100, 2), '%')
    END as transaction_growth_percentage
FROM sales_growth
ORDER BY month;

-- 4. Multiple CTEs for complex analysis
WITH product_performance AS (
    SELECT 
        product_id,
        COUNT(*) as total_sales,
        SUM(amount) as total_revenue,
        AVG(amount) as avg_sale_amount,
        MAX(amount) as max_sale,
        MIN(amount) as min_sale
    FROM sales
    GROUP BY product_id
),
customer_segments AS (
    SELECT 
        customer_id,
        COUNT(*) as purchase_frequency,
        SUM(amount) as total_spent,
        AVG(amount) as avg_purchase,
        CASE 
            WHEN SUM(amount) > 1000 THEN 'High Value'
            WHEN SUM(amount) > 500 THEN 'Medium Value'
            ELSE 'Low Value'
        END as customer_segment
    FROM sales
    GROUP BY customer_id
),
region_analysis AS (
    SELECT 
        region,
        COUNT(DISTINCT customer_id) as unique_customers,
        SUM(amount) as total_revenue,
        AVG(amount) as avg_sale_amount
    FROM sales
    GROUP BY region
)
SELECT 
    'Product Performance' as analysis_type,
    product_id,
    total_sales,
    total_revenue,
    avg_sale_amount
FROM product_performance
UNION ALL
SELECT 
    'Customer Segments' as analysis_type,
    customer_id,
    purchase_frequency,
    total_spent,
    avg_purchase
FROM customer_segments
UNION ALL
SELECT 
    'Region Analysis' as analysis_type,
    region,
    unique_customers,
    total_revenue,
    avg_sale_amount
FROM region_analysis;
```

### Q83. Explain MySQL JSON Functions with real-world examples
**Answer:** MySQL provides powerful JSON functions for storing, querying, and manipulating JSON data.

**Real-time Example:**
```sql
-- Create table with JSON column
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    price DECIMAL(10,2),
    attributes JSON,
    metadata JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert JSON data
INSERT INTO products (name, price, attributes, metadata) VALUES
('Laptop', 999.99, 
 '{"brand": "Dell", "model": "XPS 13", "specs": {"ram": "16GB", "storage": "512GB SSD", "processor": "Intel i7"}, "colors": ["Silver", "Black"], "warranty": 2}',
 '{"category": "Electronics", "tags": ["laptop", "portable", "business"], "ratings": {"average": 4.5, "count": 150}, "inventory": {"warehouse": "A", "location": "B1-23"}}'),
 
('Smartphone', 699.99,
 '{"brand": "Apple", "model": "iPhone 14", "specs": {"storage": "128GB", "camera": "12MP", "battery": "3200mAh"}, "colors": ["Blue", "Purple", "Black"], "warranty": 1}',
 '{"category": "Electronics", "tags": ["phone", "mobile", "premium"], "ratings": {"average": 4.8, "count": 300}, "inventory": {"warehouse": "B", "location": "C2-15"}}'),
 
('Book', 29.99,
 '{"author": "John Doe", "publisher": "Tech Books", "pages": 350, "language": "English", "format": "Paperback"}',
 '{"category": "Books", "tags": ["programming", "education", "tech"], "ratings": {"average": 4.2, "count": 75}, "inventory": {"warehouse": "C", "location": "D3-08"}}');

-- 1. JSON_EXTRACT() - Extract values from JSON
SELECT 
    name,
    price,
    JSON_EXTRACT(attributes, '$.brand') as brand,
    JSON_EXTRACT(attributes, '$.specs.ram') as ram,
    JSON_EXTRACT(metadata, '$.ratings.average') as avg_rating
FROM products;

-- 2. JSON_UNQUOTE() - Remove quotes from extracted values
SELECT 
    name,
    JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.brand')) as brand,
    JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.model')) as model
FROM products;

-- 3. JSON_OBJECT() - Create JSON objects
SELECT 
    name,
    price,
    JSON_OBJECT(
        'brand', JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.brand')),
        'model', JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.model')),
        'price', price
    ) as product_summary
FROM products;

-- 4. JSON_ARRAY() - Create JSON arrays
SELECT 
    name,
    JSON_ARRAY(
        JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.brand')),
        JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.model')),
        CAST(price AS CHAR)
    ) as product_array
FROM products;

-- 5. JSON_SEARCH() - Find values in JSON
SELECT 
    name,
    attributes,
    JSON_SEARCH(attributes, 'one', 'Intel') as intel_found,
    JSON_SEARCH(attributes, 'all', 'Silver') as silver_found
FROM products
WHERE JSON_SEARCH(attributes, 'one', 'Intel') IS NOT NULL;

-- 6. JSON_CONTAINS() - Check if JSON contains value
SELECT 
    name,
    attributes
FROM products
WHERE JSON_CONTAINS(attributes, '"Intel"', '$.specs.processor');

-- 7. JSON_KEYS() - Get all keys from JSON object
SELECT 
    name,
    JSON_KEYS(attributes) as attribute_keys,
    JSON_KEYS(metadata) as metadata_keys
FROM products;

-- 8. JSON_LENGTH() - Get length of JSON array or object
SELECT 
    name,
    JSON_LENGTH(attributes, '$.colors') as color_count,
    JSON_LENGTH(metadata, '$.tags') as tag_count
FROM products;

-- 9. JSON_SET() - Update JSON values
UPDATE products 
SET attributes = JSON_SET(attributes, '$.warranty', 3)
WHERE JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.brand')) = 'Dell';

-- 10. JSON_INSERT() - Insert new values
UPDATE products 
SET metadata = JSON_INSERT(metadata, '$.last_updated', NOW())
WHERE id = 1;

-- 11. JSON_REPLACE() - Replace existing values
UPDATE products 
SET attributes = JSON_REPLACE(attributes, '$.specs.ram', '32GB')
WHERE JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.model')) = 'XPS 13';

-- 12. JSON_REMOVE() - Remove values
UPDATE products 
SET metadata = JSON_REMOVE(metadata, '$.inventory')
WHERE id = 3;

-- 13. Complex JSON queries with aggregation
SELECT 
    JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.brand')) as brand,
    COUNT(*) as product_count,
    AVG(price) as avg_price,
    JSON_ARRAYAGG(
        JSON_OBJECT(
            'name', name,
            'price', price,
            'rating', JSON_UNQUOTE(JSON_EXTRACT(metadata, '$.ratings.average'))
        )
    ) as products
FROM products
GROUP BY JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.brand'));

-- 14. JSON path expressions with filtering
SELECT 
    name,
    price,
    JSON_EXTRACT(attributes, '$.specs') as specifications,
    JSON_EXTRACT(metadata, '$.ratings') as ratings
FROM products
WHERE JSON_EXTRACT(metadata, '$.ratings.average') > 4.3;

-- 15. JSON validation and error handling
SELECT 
    name,
    JSON_VALID(attributes) as is_valid_json,
    JSON_TYPE(attributes) as json_type,
    JSON_PRETTY(attributes) as formatted_json
FROM products;
```

### Q84. What is MySQL Partitioning and when to use it?
**Answer:** Partitioning divides a table into smaller, more manageable pieces while maintaining the logical structure of the table.

**Real-time Example:**
```sql
-- 1. Range Partitioning by date
CREATE TABLE sales_partitioned (
    id INT AUTO_INCREMENT,
    sale_date DATE,
    customer_id INT,
    product_id INT,
    amount DECIMAL(10,2),
    region VARCHAR(50),
    PRIMARY KEY (id, sale_date)
) PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- Insert sample data
INSERT INTO sales_partitioned (sale_date, customer_id, product_id, amount, region) VALUES
('2023-12-15', 101, 1, 100.00, 'North'),
('2024-01-15', 102, 2, 150.00, 'South'),
('2024-06-15', 103, 1, 200.00, 'East'),
('2025-01-15', 104, 3, 250.00, 'West');

-- 2. List Partitioning by region
CREATE TABLE customers_partitioned (
    id INT AUTO_INCREMENT,
    name VARCHAR(100),
    email VARCHAR(100),
    region VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id, region)
) PARTITION BY LIST COLUMNS(region) (
    PARTITION p_north VALUES IN ('North', 'Northeast'),
    PARTITION p_south VALUES IN ('South', 'Southeast'),
    PARTITION p_east VALUES IN ('East', 'Northeast'),
    PARTITION p_west VALUES IN ('West', 'Northwest'),
    PARTITION p_other VALUES IN ('Central', 'International')
);

-- 3. Hash Partitioning for even distribution
CREATE TABLE logs_partitioned (
    id INT AUTO_INCREMENT,
    log_level ENUM('DEBUG', 'INFO', 'WARNING', 'ERROR'),
    message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id)
) PARTITION BY HASH(id) PARTITIONS 4;

-- 4. Composite Partitioning (Range + Hash)
CREATE TABLE orders_partitioned (
    id INT AUTO_INCREMENT,
    order_date DATE,
    customer_id INT,
    total_amount DECIMAL(10,2),
    status ENUM('pending', 'processing', 'shipped', 'delivered'),
    PRIMARY KEY (id, order_date, customer_id)
) PARTITION BY RANGE (YEAR(order_date))
SUBPARTITION BY HASH(customer_id) SUBPARTITIONS 2 (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026)
);

-- 5. Partition management operations
-- Add new partition
ALTER TABLE sales_partitioned ADD PARTITION (
    PARTITION p2026 VALUES LESS THAN (2027)
);

-- Drop old partition
ALTER TABLE sales_partitioned DROP PARTITION p2023;

-- Truncate partition
ALTER TABLE sales_partitioned TRUNCATE PARTITION p2024;

-- 6. Query specific partitions
SELECT * FROM sales_partitioned PARTITION (p2024);
SELECT * FROM sales_partitioned PARTITION (p2025);

-- 7. Partition pruning demonstration
EXPLAIN SELECT * FROM sales_partitioned 
WHERE sale_date BETWEEN '2024-01-01' AND '2024-12-31';

-- 8. Information about partitions
SELECT 
    TABLE_NAME,
    PARTITION_NAME,
    PARTITION_ORDINAL_POSITION,
    PARTITION_METHOD,
    PARTITION_EXPRESSION,
    PARTITION_DESCRIPTION,
    TABLE_ROWS
FROM information_schema.PARTITIONS
WHERE TABLE_SCHEMA = 'your_database' 
AND TABLE_NAME = 'sales_partitioned';

-- 9. Maintenance operations
-- Analyze partition
ALTER TABLE sales_partitioned ANALYZE PARTITION p2024;

-- Check partition
ALTER TABLE sales_partitioned CHECK PARTITION p2024;

-- Repair partition
ALTER TABLE sales_partitioned REPAIR PARTITION p2024;

-- Optimize partition
ALTER TABLE sales_partitioned OPTIMIZE PARTITION p2024;

-- 10. Best practices for partitioning
-- Use partitioning for:
-- - Large tables (> millions of rows)
-- - Time-series data
-- - Data with clear partitioning criteria
-- - Tables that need to drop old data regularly

-- Example: Log table with automatic partition management
CREATE TABLE application_logs (
    id BIGINT AUTO_INCREMENT,
    log_date DATETIME,
    level ENUM('DEBUG', 'INFO', 'WARNING', 'ERROR'),
    message TEXT,
    user_id INT,
    PRIMARY KEY (id, log_date)
) PARTITION BY RANGE (TO_DAYS(log_date)) (
    PARTITION p_old VALUES LESS THAN (TO_DAYS('2024-01-01')),
    PARTITION p_2024_01 VALUES LESS THAN (TO_DAYS('2024-02-01')),
    PARTITION p_2024_02 VALUES LESS THAN (TO_DAYS('2024-03-01')),
    PARTITION p_2024_03 VALUES LESS THAN (TO_DAYS('2024-04-01')),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- Stored procedure for automatic partition management
DELIMITER //
CREATE PROCEDURE CreateMonthlyPartition(IN table_name VARCHAR(64), IN partition_date DATE)
BEGIN
    DECLARE partition_name VARCHAR(64);
    DECLARE next_month DATE;
    DECLARE sql_stmt TEXT;
    
    SET partition_name = CONCAT('p_', DATE_FORMAT(partition_date, '%Y_%m'));
    SET next_month = DATE_ADD(LAST_DAY(partition_date), INTERVAL 1 DAY);
    
    SET sql_stmt = CONCAT(
        'ALTER TABLE ', table_name, 
        ' ADD PARTITION (PARTITION ', partition_name, 
        ' VALUES LESS THAN (TO_DAYS(''', next_month, ''')))'
    );
    
    SET @sql = sql_stmt;
    PREPARE stmt FROM @sql;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
END //
DELIMITER ;

-- Usage
CALL CreateMonthlyPartition('application_logs', '2024-04-01');
```

### Q85. Explain MySQL Replication and its types
**Answer:** MySQL replication allows you to copy data from one MySQL database server (master) to one or more MySQL database servers (slaves).

**Real-time Example:**
```sql
-- Master server configuration (my.cnf)
[mysqld]
server-id = 1
log-bin = mysql-bin
binlog-format = ROW
sync_binlog = 1
innodb_flush_log_at_trx_commit = 1

-- Slave server configuration (my.cnf)
[mysqld]
server-id = 2
relay-log = mysql-relay-bin
read-only = 1
log-slave-updates = 1

-- 1. Set up replication user on master
CREATE USER 'repl_user'@'%' IDENTIFIED BY 'secure_password';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';
FLUSH PRIVILEGES;

-- 2. Get master status
SHOW MASTER STATUS;
-- Note: File: mysql-bin.000001, Position: 154

-- 3. Configure slave
CHANGE MASTER TO
    MASTER_HOST = '192.168.1.100',
    MASTER_USER = 'repl_user',
    MASTER_PASSWORD = 'secure_password',
    MASTER_LOG_FILE = 'mysql-bin.000001',
    MASTER_LOG_POS = 154;

-- 4. Start slave
START SLAVE;

-- 5. Check slave status
SHOW SLAVE STATUS\G

-- 6. Monitor replication
SELECT 
    MASTER_HOST,
    MASTER_USER,
    MASTER_PORT,
    MASTER_LOG_FILE,
    READ_MASTER_LOG_POS,
    SLAVE_IO_RUNNING,
    SLAVE_SQL_RUNNING,
    SECONDS_BEHIND_MASTER
FROM information_schema.SLAVE_STATUS;

-- 7. Replication filtering
-- On master: Filter databases
[mysqld]
binlog-do-db = important_db
binlog-ignore-db = test_db

-- On slave: Filter tables
CHANGE REPLICATION FILTER REPLICATE_DO_TABLE = ('important_db.important_table');

-- 8. Multi-source replication (MySQL 5.7+)
-- Configure multiple masters
CHANGE MASTER TO
    MASTER_HOST = '192.168.1.100',
    MASTER_USER = 'repl_user',
    MASTER_PASSWORD = 'secure_password',
    MASTER_LOG_FILE = 'mysql-bin.000001',
    MASTER_LOG_POS = 154
FOR CHANNEL 'master1';

CHANGE MASTER TO
    MASTER_HOST = '192.168.1.101',
    MASTER_USER = 'repl_user',
    MASTER_PASSWORD = 'secure_password',
    MASTER_LOG_FILE = 'mysql-bin.000001',
    MASTER_LOG_POS = 154
FOR CHANNEL 'master2';

-- Start specific channel
START SLAVE FOR CHANNEL 'master1';

-- 9. GTID (Global Transaction Identifier) replication
-- Master configuration
[mysqld]
gtid-mode = ON
enforce-gtid-consistency = ON

-- Slave configuration with GTID
CHANGE MASTER TO
    MASTER_HOST = '192.168.1.100',
    MASTER_USER = 'repl_user',
    MASTER_PASSWORD = 'secure_password',
    MASTER_AUTO_POSITION = 1;

-- 10. Replication monitoring queries
-- Check replication lag
SELECT 
    MASTER_HOST,
    SECONDS_BEHIND_MASTER,
    SLAVE_IO_RUNNING,
    SLAVE_SQL_RUNNING,
    LAST_IO_ERROR,
    LAST_SQL_ERROR
FROM information_schema.SLAVE_STATUS;

-- Check binary log events
SHOW BINLOG EVENTS IN 'mysql-bin.000001' FROM 4 LIMIT 10;

-- Check replication threads
SHOW PROCESSLIST;

-- 11. Replication troubleshooting
-- Skip error on slave
SET GLOBAL sql_slave_skip_counter = 1;
START SLAVE;

-- Reset slave
STOP SLAVE;
RESET SLAVE;
-- Reconfigure and start again

-- 12. Backup and restore for replication
-- On master: Create backup
mysqldump --all-databases --master-data=2 --single-transaction > backup.sql

-- On slave: Restore and configure
mysql < backup.sql
-- Then configure replication as shown above

-- 13. Replication performance tuning
-- Master optimizations
[mysqld]
binlog-cache-size = 1M
max-binlog-cache-size = 2G
sync-binlog = 1

-- Slave optimizations
[mysqld]
slave-net-timeout = 60
slave-sql-threads = 4  # Parallel replication
slave-parallel-workers = 4
```

### Q86. What are MySQL Stored Procedures and Functions?
**Answer:** Stored procedures and functions are precompiled SQL statements stored in the database that can be called with parameters.

**Real-time Example:**
```sql
-- 1. Simple stored procedure
DELIMITER //
CREATE PROCEDURE GetCustomerOrders(IN customer_id INT)
BEGIN
    SELECT 
        o.id,
        o.order_date,
        o.total_amount,
        o.status
    FROM orders o
    WHERE o.customer_id = customer_id
    ORDER BY o.order_date DESC;
END //
DELIMITER ;

-- Usage
CALL GetCustomerOrders(101);

-- 2. Stored procedure with multiple parameters
DELIMITER //
CREATE PROCEDURE CreateOrder(
    IN p_customer_id INT,
    IN p_total_amount DECIMAL(10,2),
    IN p_status VARCHAR(20),
    OUT p_order_id INT
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        RESIGNAL;
    END;
    
    START TRANSACTION;
    
    INSERT INTO orders (customer_id, total_amount, status, order_date)
    VALUES (p_customer_id, p_total_amount, p_status, NOW());
    
    SET p_order_id = LAST_INSERT_ID();
    
    COMMIT;
END //
DELIMITER ;

-- Usage
CALL CreateOrder(101, 299.99, 'pending', @new_order_id);
SELECT @new_order_id;

-- 3. Stored function
DELIMITER //
CREATE FUNCTION CalculateDiscount(amount DECIMAL(10,2), customer_type VARCHAR(20))
RETURNS DECIMAL(10,2)
READS SQL DATA
DETERMINISTIC
BEGIN
    DECLARE discount DECIMAL(10,2) DEFAULT 0;
    
    CASE customer_type
        WHEN 'VIP' THEN SET discount = amount * 0.15;
        WHEN 'Premium' THEN SET discount = amount * 0.10;
        WHEN 'Regular' THEN SET discount = amount * 0.05;
        ELSE SET discount = 0;
    END CASE;
    
    RETURN discount;
END //
DELIMITER ;

-- Usage
SELECT 
    id,
    total_amount,
    customer_type,
    CalculateDiscount(total_amount, customer_type) as discount,
    total_amount - CalculateDiscount(total_amount, customer_type) as final_amount
FROM orders;

-- 4. Complex stored procedure with cursors
DELIMITER //
CREATE PROCEDURE ProcessBulkOrders(IN order_ids TEXT)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE order_id INT;
    DECLARE order_cursor CURSOR FOR 
        SELECT CAST(SUBSTRING_INDEX(SUBSTRING_INDEX(order_ids, ',', numbers.n), ',', -1) AS UNSIGNED) as id
        FROM (SELECT 1 n UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5) numbers
        WHERE CHAR_LENGTH(order_ids) - CHAR_LENGTH(REPLACE(order_ids, ',', '')) >= numbers.n - 1;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        RESIGNAL;
    END;
    
    START TRANSACTION;
    
    OPEN order_cursor;
    
    read_loop: LOOP
        FETCH order_cursor INTO order_id;
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        -- Process each order
        UPDATE orders 
        SET status = 'processing', 
            updated_at = NOW() 
        WHERE id = order_id;
        
        -- Log the processing
        INSERT INTO order_logs (order_id, action, created_at)
        VALUES (order_id, 'Bulk processing started', NOW());
        
    END LOOP;
    
    CLOSE order_cursor;
    COMMIT;
END //
DELIMITER ;

-- Usage
CALL ProcessBulkOrders('1,2,3,4,5');

-- 5. Stored procedure with error handling
DELIMITER //
CREATE PROCEDURE SafeOrderUpdate(
    IN p_order_id INT,
    IN p_new_status VARCHAR(20),
    OUT p_success BOOLEAN,
    OUT p_message VARCHAR(255)
)
BEGIN
    DECLARE order_exists INT DEFAULT 0;
    DECLARE current_status VARCHAR(20);
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        SET p_success = FALSE;
        SET p_message = 'Database error occurred';
        ROLLBACK;
    END;
    
    START TRANSACTION;
    
    -- Check if order exists
    SELECT COUNT(*) INTO order_exists FROM orders WHERE id = p_order_id;
    
    IF order_exists = 0 THEN
        SET p_success = FALSE;
        SET p_message = 'Order not found';
        ROLLBACK;
    ELSE
        -- Get current status
        SELECT status INTO current_status FROM orders WHERE id = p_order_id;
        
        -- Validate status transition
        IF current_status = 'delivered' AND p_new_status != 'delivered' THEN
            SET p_success = FALSE;
            SET p_message = 'Cannot change status of delivered order';
            ROLLBACK;
        ELSE
            -- Update order
            UPDATE orders 
            SET status = p_new_status, updated_at = NOW() 
            WHERE id = p_order_id;
            
            -- Log the change
            INSERT INTO order_logs (order_id, action, created_at)
            VALUES (p_order_id, CONCAT('Status changed to ', p_new_status), NOW());
            
            SET p_success = TRUE;
            SET p_message = 'Order updated successfully';
            COMMIT;
        END IF;
    END IF;
END //
DELIMITER ;

-- Usage
CALL SafeOrderUpdate(1, 'shipped', @success, @message);
SELECT @success, @message;

-- 6. Recursive stored procedure
DELIMITER //
CREATE PROCEDURE GetCategoryHierarchy(IN root_category_id INT)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE cat_id INT;
    DECLARE cat_name VARCHAR(100);
    DECLARE parent_id INT;
    DECLARE level INT DEFAULT 0;
    
    -- Create temporary table for results
    DROP TEMPORARY TABLE IF EXISTS temp_categories;
    CREATE TEMPORARY TABLE temp_categories (
        id INT,
        name VARCHAR(100),
        parent_id INT,
        level INT
    );
    
    -- Insert root category
    INSERT INTO temp_categories (id, name, parent_id, level)
    SELECT id, name, parent_id, 0 FROM categories WHERE id = root_category_id;
    
    -- Recursive loop
    WHILE ROW_COUNT() > 0 DO
        SET level = level + 1;
        
        INSERT INTO temp_categories (id, name, parent_id, level)
        SELECT c.id, c.name, c.parent_id, level
        FROM categories c
        INNER JOIN temp_categories tc ON c.parent_id = tc.id
        WHERE tc.level = level - 1;
        
    END WHILE;
    
    -- Return results
    SELECT * FROM temp_categories ORDER BY level, name;
    
    DROP TEMPORARY TABLE temp_categories;
END //
DELIMITER ;

-- 7. Stored procedure for data migration
DELIMITER //
CREATE PROCEDURE MigrateOldOrders()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE old_order_id INT;
    DECLARE customer_id INT;
    DECLARE order_date DATE;
    DECLARE total_amount DECIMAL(10,2);
    
    DECLARE old_orders_cursor CURSOR FOR 
        SELECT id, customer_id, order_date, total_amount 
        FROM old_orders 
        WHERE migrated = FALSE;
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        RESIGNAL;
    END;
    
    START TRANSACTION;
    
    OPEN old_orders_cursor;
    
    read_loop: LOOP
        FETCH old_orders_cursor INTO old_order_id, customer_id, order_date, total_amount;
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        -- Migrate order
        INSERT INTO orders (customer_id, order_date, total_amount, status, created_at)
        VALUES (customer_id, order_date, total_amount, 'migrated', NOW());
        
        -- Mark as migrated
        UPDATE old_orders SET migrated = TRUE WHERE id = old_order_id;
        
    END LOOP;
    
    CLOSE old_orders_cursor;
    COMMIT;
    
    SELECT 'Migration completed successfully' as result;
END //
DELIMITER ;
```

---

## MySQL Replication

### Q87. MySQL Master-Slave Replication Setup
**Answer:** MySQL replication allows you to maintain multiple copies of your database for backup, scaling, and high availability.

**Real-time Example:**
```sql
-- Master Server Configuration (my.cnf)
[mysqld]
server-id = 1
log-bin = mysql-bin
binlog-format = ROW
sync_binlog = 1
innodb_flush_log_at_trx_commit = 1

-- Create replication user on master
CREATE USER 'repl_user'@'%' IDENTIFIED BY 'secure_password';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';
FLUSH PRIVILEGES;

-- Show master status
SHOW MASTER STATUS;
-- Note: File: mysql-bin.000001, Position: 154

-- Slave Server Configuration (my.cnf)
[mysqld]
server-id = 2
relay-log = mysql-relay-bin
read_only = 1

-- Configure slave to connect to master
CHANGE MASTER TO
    MASTER_HOST='192.168.1.100',
    MASTER_USER='repl_user',
    MASTER_PASSWORD='secure_password',
    MASTER_LOG_FILE='mysql-bin.000001',
    MASTER_LOG_POS=154;

-- Start slave
START SLAVE;

-- Check slave status
SHOW SLAVE STATUS\G

-- Monitor replication lag
SELECT 
    MASTER_LOG_FILE,
    MASTER_LOG_POS,
    RELAY_LOG_FILE,
    RELAY_LOG_POS,
    SLAVE_IO_RUNNING,
    SLAVE_SQL_RUNNING,
    SECONDS_BEHIND_MASTER
FROM performance_schema.replication_applier_status_by_worker;
```

### Q88. MySQL Multi-Source Replication
**Answer:** Multi-source replication allows a single slave to replicate from multiple masters.

**Real-time Example:**
```sql
-- Configure multiple replication channels
-- Channel 1: Replicate from master1
CHANGE MASTER TO
    MASTER_HOST='master1.example.com',
    MASTER_USER='repl_user',
    MASTER_PASSWORD='password1',
    MASTER_LOG_FILE='mysql-bin.000001',
    MASTER_LOG_POS=154
FOR CHANNEL 'master1';

-- Channel 2: Replicate from master2
CHANGE MASTER TO
    MASTER_HOST='master2.example.com',
    MASTER_USER='repl_user',
    MASTER_PASSWORD='password2',
    MASTER_LOG_FILE='mysql-bin.000001',
    MASTER_LOG_POS=154
FOR CHANNEL 'master2';

-- Start both channels
START SLAVE FOR CHANNEL 'master1';
START SLAVE FOR CHANNEL 'master2';

-- Check status for specific channel
SHOW SLAVE STATUS FOR CHANNEL 'master1'\G

-- Stop specific channel
STOP SLAVE FOR CHANNEL 'master1';

-- Replication filtering
-- Replicate only specific databases
CHANGE REPLICATION FILTER REPLICATE_DO_DB = (ecommerce, analytics);

-- Replicate only specific tables
CHANGE REPLICATION FILTER REPLICATE_DO_TABLE = (ecommerce.orders, ecommerce.customers);
```

### Q89. MySQL GTID (Global Transaction Identifier)
**Answer:** GTID provides a unique identifier for each transaction, making replication more reliable and easier to manage.

**Real-time Example:**
```sql
-- Enable GTID on master (my.cnf)
[mysqld]
gtid-mode = ON
enforce-gtid-consistency = ON
log-slave-updates = ON

-- Enable GTID on slave (my.cnf)
[mysqld]
gtid-mode = ON
enforce-gtid-consistency = ON
log-slave-updates = ON

-- Configure slave with GTID
CHANGE MASTER TO
    MASTER_HOST='192.168.1.100',
    MASTER_USER='repl_user',
    MASTER_PASSWORD='secure_password',
    MASTER_AUTO_POSITION = 1;

-- Check GTID status
SELECT @@gtid_mode;
SELECT @@enforce_gtid_consistency;

-- View GTID executed
SHOW MASTER STATUS;
-- GTID: 1-1-1, 1-1-2, 1-1-3

-- View GTID purged
SELECT @@gtid_purged;

-- Skip transactions with GTID
SET GTID_NEXT = '1-1-4';
BEGIN;
COMMIT;
SET GTID_NEXT = 'AUTOMATIC';
```

---

## MySQL Partitioning

### Q90. MySQL Table Partitioning Strategies
**Answer:** Partitioning divides large tables into smaller, more manageable pieces for better performance and maintenance.

**Real-time Example:**
```sql
-- Range Partitioning by Date
CREATE TABLE sales (
    id INT AUTO_INCREMENT,
    sale_date DATE NOT NULL,
    customer_id INT,
    amount DECIMAL(10,2),
    PRIMARY KEY (id, sale_date)
) PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022),
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- List Partitioning by Region
CREATE TABLE customers (
    id INT AUTO_INCREMENT,
    name VARCHAR(100),
    region VARCHAR(50),
    email VARCHAR(100),
    PRIMARY KEY (id, region)
) PARTITION BY LIST COLUMNS(region) (
    PARTITION p_north VALUES IN ('NY', 'MA', 'CT'),
    PARTITION p_south VALUES IN ('FL', 'GA', 'SC'),
    PARTITION p_west VALUES IN ('CA', 'WA', 'OR'),
    PARTITION p_other VALUES IN ('TX', 'IL', 'OH')
);

-- Hash Partitioning for Load Distribution
CREATE TABLE user_sessions (
    id INT AUTO_INCREMENT,
    user_id INT,
    session_data TEXT,
    created_at TIMESTAMP,
    PRIMARY KEY (id, user_id)
) PARTITION BY HASH(user_id) PARTITIONS 8;

-- Composite Partitioning (Range + Hash)
CREATE TABLE orders (
    id INT AUTO_INCREMENT,
    order_date DATE,
    customer_id INT,
    amount DECIMAL(10,2),
    PRIMARY KEY (id, order_date, customer_id)
) PARTITION BY RANGE (YEAR(order_date))
SUBPARTITION BY HASH(customer_id) SUBPARTITIONS 4 (
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022),
    PARTITION p2022 VALUES LESS THAN (2023)
);

-- Partition Management
-- Add new partition
ALTER TABLE sales ADD PARTITION (
    PARTITION p2024 VALUES LESS THAN (2025)
);

-- Drop old partition
ALTER TABLE sales DROP PARTITION p2020;

-- Truncate specific partition
ALTER TABLE sales TRUNCATE PARTITION p2021;

-- Query specific partition
SELECT * FROM sales PARTITION (p2022);

-- Check partition information
SELECT 
    TABLE_NAME,
    PARTITION_NAME,
    PARTITION_DESCRIPTION,
    TABLE_ROWS
FROM INFORMATION_SCHEMA.PARTITIONS 
WHERE TABLE_NAME = 'sales';
```

### Q91. MySQL Partition Pruning and Optimization
**Answer:** Partition pruning automatically eliminates partitions that don't match the query conditions, improving performance.

**Real-time Example:**
```sql
-- Automatic partition pruning
-- This query will only scan p2022 partition
SELECT * FROM sales 
WHERE sale_date BETWEEN '2022-01-01' AND '2022-12-31';

-- Check if partition pruning is working
EXPLAIN PARTITIONS SELECT * FROM sales 
WHERE sale_date BETWEEN '2022-01-01' AND '2022-12-31';

-- Manual partition selection
SELECT * FROM sales PARTITION (p2022, p2023)
WHERE sale_date BETWEEN '2022-01-01' AND '2023-12-31';

-- Partition-aware indexing
CREATE INDEX idx_sales_date_customer ON sales (sale_date, customer_id);

-- Analyze partition statistics
ANALYZE TABLE sales;

-- Optimize specific partition
OPTIMIZE TABLE sales PARTITION (p2022);

-- Check partition sizes
SELECT 
    PARTITION_NAME,
    TABLE_ROWS,
    DATA_LENGTH,
    INDEX_LENGTH,
    (DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024 AS 'Size_MB'
FROM INFORMATION_SCHEMA.PARTITIONS 
WHERE TABLE_NAME = 'sales'
ORDER BY PARTITION_NAME;
```

---

## MySQL Backup & Recovery

### Q92. MySQL Backup Strategies
**Answer:** MySQL provides multiple backup methods including mysqldump, binary log backups, and physical backups.

**Real-time Example:**
```bash
# Full database backup
mysqldump --single-transaction --routines --triggers \
    --all-databases --master-data=2 > full_backup.sql

# Backup specific database
mysqldump --single-transaction --routines --triggers \
    --databases ecommerce > ecommerce_backup.sql

# Backup with compression
mysqldump --single-transaction --routines --triggers \
    --all-databases | gzip > backup_$(date +%Y%m%d_%H%M%S).sql.gz

# Incremental backup using binary logs
# First, enable binary logging
# my.cnf: log-bin = mysql-bin

# Full backup
mysqldump --single-transaction --master-data=2 \
    --all-databases > full_backup.sql

# Incremental backup (backup binary logs)
mysqlbinlog --start-datetime="2023-01-01 00:00:00" \
    --stop-datetime="2023-01-02 00:00:00" \
    mysql-bin.000001 mysql-bin.000002 > incremental_backup.sql

# Point-in-time recovery
# 1. Restore full backup
mysql < full_backup.sql

# 2. Apply incremental changes
mysqlbinlog --start-datetime="2023-01-01 00:00:00" \
    --stop-datetime="2023-01-01 12:00:00" \
    mysql-bin.000001 | mysql

# Physical backup using Percona XtraBackup
xtrabackup --backup --target-dir=/backup/full/
xtrabackup --prepare --target-dir=/backup/full/
xtrabackup --copy-back --target-dir=/backup/full/
```

### Q93. MySQL Recovery Procedures
**Answer:** Recovery procedures depend on the backup type and the point of failure.

**Real-time Example:**
```sql
-- Recovery from logical backup
-- 1. Stop MySQL service
systemctl stop mysql

-- 2. Remove existing data directory
rm -rf /var/lib/mysql/*

-- 3. Restore from backup
mysql < full_backup.sql

-- 4. Start MySQL service
systemctl start mysql

-- Recovery from physical backup
-- 1. Stop MySQL service
systemctl stop mysql

-- 2. Remove existing data directory
rm -rf /var/lib/mysql/*

-- 3. Copy backup files
cp -r /backup/full/* /var/lib/mysql/

-- 4. Set proper ownership
chown -R mysql:mysql /var/lib/mysql

-- 5. Start MySQL service
systemctl start mysql

-- Point-in-time recovery
-- 1. Restore full backup
mysql < full_backup.sql

-- 2. Apply binary logs up to specific time
mysqlbinlog --start-datetime="2023-01-01 00:00:00" \
    --stop-datetime="2023-01-01 12:00:00" \
    mysql-bin.000001 mysql-bin.000002 | mysql

-- Recovery from replication
-- 1. Stop slave
STOP SLAVE;

-- 2. Reset slave to start from beginning
RESET SLAVE;

-- 3. Configure new master position
CHANGE MASTER TO
    MASTER_LOG_FILE='mysql-bin.000001',
    MASTER_LOG_POS=154;

-- 4. Start slave
START SLAVE;

-- Verify recovery
SHOW SLAVE STATUS\G
```

---

## MySQL Monitoring & Logging

### Q94. MySQL Performance Monitoring
**Answer:** MySQL provides comprehensive monitoring through performance schema, information schema, and various status variables.

**Real-time Example:**
```sql
-- Monitor active connections
SELECT 
    ID,
    USER,
    HOST,
    DB,
    COMMAND,
    TIME,
    STATE,
    INFO
FROM INFORMATION_SCHEMA.PROCESSLIST
WHERE COMMAND != 'Sleep';

-- Monitor slow queries
SELECT 
    query_time,
    lock_time,
    rows_sent,
    rows_examined,
    sql_text
FROM mysql.slow_log
WHERE start_time > DATE_SUB(NOW(), INTERVAL 1 HOUR)
ORDER BY query_time DESC;

-- Monitor table locks
SELECT 
    OBJECT_SCHEMA,
    OBJECT_NAME,
    LOCK_TYPE,
    LOCK_DURATION,
    LOCK_STATUS
FROM performance_schema.metadata_locks
WHERE OBJECT_SCHEMA IS NOT NULL;

-- Monitor InnoDB status
SHOW ENGINE INNODB STATUS;

-- Monitor replication lag
SELECT 
    MASTER_LOG_FILE,
    MASTER_LOG_POS,
    RELAY_LOG_FILE,
    RELAY_LOG_POS,
    SLAVE_IO_RUNNING,
    SLAVE_SQL_RUNNING,
    SECONDS_BEHIND_MASTER
FROM performance_schema.replication_applier_status_by_worker;

-- Monitor buffer pool usage
SELECT 
    POOL_ID,
    POOL_SIZE,
    FREE_BUFFERS,
    DATABASE_PAGES,
    OLD_DATABASE_PAGES,
    MODIFIED_DATABASE_PAGES
FROM performance_schema.innodb_buffer_pool_stats;

-- Monitor query performance
SELECT 
    DIGEST_TEXT,
    COUNT_STAR,
    AVG_TIMER_WAIT/1000000000 as AVG_TIME_SEC,
    MAX_TIMER_WAIT/1000000000 as MAX_TIME_SEC,
    SUM_ROWS_EXAMINED,
    SUM_ROWS_SENT
FROM performance_schema.events_statements_summary_by_digest
ORDER BY AVG_TIMER_WAIT DESC
LIMIT 10;
```

### Q95. MySQL Logging Configuration
**Answer:** MySQL provides various logging options for debugging, auditing, and performance analysis.

**Real-time Example:**
```sql
-- Enable general query log
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file = '/var/log/mysql/general.log';

-- Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';
SET GLOBAL long_query_time = 2; -- Log queries taking more than 2 seconds

-- Enable binary log
SET GLOBAL log_bin = 'ON';
SET GLOBAL binlog_format = 'ROW';

-- Enable error log (configured in my.cnf)
-- [mysqld]
-- log-error = /var/log/mysql/error.log

-- Check current logging settings
SHOW VARIABLES LIKE '%log%';

-- Monitor log file sizes
SELECT 
    'General Log' as Log_Type,
    @@general_log_file as Log_File,
    ROUND(@@general_log_file_size/1024/1024, 2) as Size_MB
UNION ALL
SELECT 
    'Slow Query Log',
    @@slow_query_log_file,
    ROUND(@@slow_query_log_file_size/1024/1024, 2)
UNION ALL
SELECT 
    'Binary Log',
    @@log_bin_basename,
    ROUND(@@log_bin_file_size/1024/1024, 2);

-- Rotate logs
FLUSH LOGS;

-- Purge old binary logs
PURGE BINARY LOGS TO 'mysql-bin.000010';
PURGE BINARY LOGS BEFORE '2023-01-01 00:00:00';

-- Monitor log rotation
SHOW BINARY LOGS;
```

---

## MySQL Performance Tuning

### Q96. MySQL Query Optimization
**Answer:** Query optimization involves analyzing execution plans, creating proper indexes, and rewriting queries for better performance.

**Real-time Example:**
```sql
-- Analyze query execution plan
EXPLAIN FORMAT=JSON
SELECT c.name, COUNT(o.id) as order_count, SUM(o.total) as total_spent
FROM customers c
JOIN orders o ON c.id = o.customer_id
WHERE c.created_at >= '2023-01-01'
GROUP BY c.id, c.name
HAVING COUNT(o.id) > 5
ORDER BY total_spent DESC;

-- Optimize with proper indexes
CREATE INDEX idx_customers_created ON customers(created_at);
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
CREATE INDEX idx_orders_total ON orders(total);

-- Use covering indexes
CREATE INDEX idx_orders_covering ON orders(customer_id, order_date, total);

-- Optimize JOIN queries
-- Before: Nested loop join
SELECT c.name, o.order_date, o.total
FROM customers c
JOIN orders o ON c.id = o.customer_id
WHERE c.status = 'active';

-- After: Use appropriate JOIN type
SELECT c.name, o.order_date, o.total
FROM customers c
STRAIGHT_JOIN orders o ON c.id = o.customer_id
WHERE c.status = 'active';

-- Optimize subqueries
-- Before: Correlated subquery
SELECT c.name, 
    (SELECT COUNT(*) FROM orders o WHERE o.customer_id = c.id) as order_count
FROM customers c;

-- After: Use JOIN
SELECT c.name, COUNT(o.id) as order_count
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name;

-- Use LIMIT for pagination
SELECT * FROM orders 
WHERE customer_id = 123
ORDER BY order_date DESC
LIMIT 20 OFFSET 0;

-- Optimize with query hints
SELECT /*+ USE_INDEX(orders, idx_orders_customer_date) */
    c.name, o.order_date, o.total
FROM customers c
JOIN orders o ON c.id = o.customer_id
WHERE o.order_date >= '2023-01-01';
```

### Q97. MySQL Server Configuration Optimization
**Answer:** Server configuration optimization involves tuning various parameters for optimal performance.

**Real-time Example:**
```sql
-- InnoDB Configuration
-- my.cnf settings
[mysqld]
# Buffer pool size (70-80% of RAM)
innodb_buffer_pool_size = 2G

# Log file size
innodb_log_file_size = 256M
innodb_log_files_in_group = 2

# Flush method
innodb_flush_method = O_DIRECT

# Thread concurrency
innodb_thread_concurrency = 0

# I/O capacity
innodb_io_capacity = 2000
innodb_io_capacity_max = 4000

# Checkpoint age
innodb_max_dirty_pages_pct = 75

# Connection settings
max_connections = 200
max_connect_errors = 100000

# Query cache (MySQL 5.7 and earlier)
query_cache_type = 1
query_cache_size = 64M
query_cache_limit = 2M

# Temporary tables
tmp_table_size = 64M
max_heap_table_size = 64M

# Sort buffer
sort_buffer_size = 2M
read_buffer_size = 2M
read_rnd_buffer_size = 8M

# Thread settings
thread_cache_size = 8
table_open_cache = 4000

-- Monitor configuration effectiveness
SHOW STATUS LIKE 'Innodb_buffer_pool%';
SHOW STATUS LIKE 'Qcache%';
SHOW STATUS LIKE 'Threads%';
SHOW STATUS LIKE 'Connections%';

-- Check current settings
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SHOW VARIABLES LIKE 'max_connections';
SHOW VARIABLES LIKE 'query_cache%';
```

---

## MySQL Security Hardening

### Q98. MySQL Security Configuration
**Answer:** MySQL security involves user management, encryption, and access control to protect data and prevent unauthorized access.

**Real-time Example:**
```sql
-- Create secure user accounts
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'strong_password_123!';
CREATE USER 'readonly_user'@'192.168.1.%' IDENTIFIED BY 'readonly_pass_456!';

-- Grant minimal required privileges
GRANT SELECT, INSERT, UPDATE, DELETE ON ecommerce.* TO 'app_user'@'localhost';
GRANT SELECT ON ecommerce.products TO 'readonly_user'@'192.168.1.%';

-- Remove anonymous users
DELETE FROM mysql.user WHERE User = '';
DELETE FROM mysql.user WHERE User = 'root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');

-- Remove test database
DROP DATABASE IF EXISTS test;
DELETE FROM mysql.db WHERE Db = 'test' OR Db = 'test\\_%';

-- Enable SSL connections
-- Generate SSL certificates
-- mysql_ssl_rsa_setup

-- Configure SSL in my.cnf
[mysqld]
ssl-ca = /etc/mysql/ssl/ca.pem
ssl-cert = /etc/mysql/ssl/server-cert.pem
ssl-key = /etc/mysql/ssl/server-key.pem

-- Require SSL for specific users
ALTER USER 'app_user'@'localhost' REQUIRE SSL;

-- Enable password validation plugin
INSTALL PLUGIN validate_password SONAME 'validate_password.so';

-- Configure password policy
SET GLOBAL validate_password.policy = STRONG;
SET GLOBAL validate_password.length = 12;
SET GLOBAL validate_password.mixed_case_count = 2;
SET GLOBAL validate_password.number_count = 2;
SET GLOBAL validate_password.special_char_count = 2;

-- Enable audit logging
INSTALL PLUGIN audit_log SONAME 'audit_log.so';

-- Configure audit log
SET GLOBAL audit_log_policy = ALL;
SET GLOBAL audit_log_format = JSON;
SET GLOBAL audit_log_file = '/var/log/mysql/audit.log';

-- Monitor failed login attempts
SELECT 
    user,
    host,
    COUNT(*) as failed_attempts
FROM mysql.general_log
WHERE command_type = 'Connect'
    AND argument LIKE '%Access denied%'
    AND event_time > DATE_SUB(NOW(), INTERVAL 1 HOUR)
GROUP BY user, host
HAVING failed_attempts > 5;
```

### Q99. MySQL Data Encryption
**Answer:** MySQL provides encryption at rest and in transit to protect sensitive data.

**Real-time Example:**
```sql
-- Enable encryption at rest
-- Configure in my.cnf
[mysqld]
innodb_encrypt_tables = ON
innodb_encryption_threads = 4
innodb_encryption_rotate_key_age = 1

-- Encrypt specific tables
ALTER TABLE customers ENCRYPTION = 'Y';
ALTER TABLE orders ENCRYPTION = 'Y';
ALTER TABLE payments ENCRYPTION = 'Y';

-- Check encryption status
SELECT 
    SCHEMA_NAME,
    TABLE_NAME,
    CREATE_OPTIONS
FROM INFORMATION_SCHEMA.TABLES
WHERE CREATE_OPTIONS LIKE '%ENCRYPTION%';

-- Encrypt specific columns
CREATE TABLE sensitive_data (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    ssn VARBINARY(255), -- Encrypted SSN
    credit_card VARBINARY(255) -- Encrypted credit card
);

-- Insert encrypted data
INSERT INTO sensitive_data (id, name, ssn, credit_card)
VALUES (
    1, 
    'John Doe',
    AES_ENCRYPT('123-45-6789', 'encryption_key'),
    AES_ENCRYPT('4111-1111-1111-1111', 'encryption_key')
);

-- Decrypt data
SELECT 
    id,
    name,
    AES_DECRYPT(ssn, 'encryption_key') as ssn,
    AES_DECRYPT(credit_card, 'encryption_key') as credit_card
FROM sensitive_data;

-- Enable binary log encryption
SET GLOBAL binlog_encryption = ON;

-- Enable redo log encryption
SET GLOBAL innodb_redo_log_encrypt = ON;

-- Enable undo log encryption
SET GLOBAL innodb_undo_log_encrypt = ON;

-- Check encryption status
SHOW STATUS LIKE 'Innodb_encryption%';
```

---

## MySQL JSON Functions

### Q100. Advanced MySQL JSON Operations
**Answer:** MySQL provides comprehensive JSON functions for storing, querying, and manipulating JSON data.

**Real-time Example:**
```sql
-- Create table with JSON column
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    specifications JSON,
    metadata JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert JSON data
INSERT INTO products (name, specifications, metadata) VALUES
('Laptop', 
 '{"cpu": "Intel i7", "ram": "16GB", "storage": "512GB SSD", "screen": "15.6 inch"}',
 '{"brand": "Dell", "model": "XPS 15", "warranty": "2 years", "tags": ["gaming", "professional"]}'),
('Smartphone',
 '{"cpu": "Snapdragon 888", "ram": "8GB", "storage": "256GB", "camera": "108MP"}',
 '{"brand": "Samsung", "model": "Galaxy S21", "warranty": "1 year", "tags": ["flagship", "camera"]}');

-- Query JSON data
SELECT 
    name,
    JSON_EXTRACT(specifications, '$.cpu') as cpu,
    JSON_EXTRACT(specifications, '$.ram') as ram,
    JSON_EXTRACT(metadata, '$.brand') as brand
FROM products;

-- Use JSON path expressions
SELECT 
    name,
    specifications->>'$.cpu' as cpu,
    specifications->>'$.ram' as ram,
    metadata->>'$.brand' as brand,
    metadata->>'$.tags[0]' as primary_tag
FROM products;

-- Search in JSON arrays
SELECT name, specifications
FROM products
WHERE JSON_SEARCH(metadata, 'one', 'gaming') IS NOT NULL;

-- Filter by JSON values
SELECT name, specifications
FROM products
WHERE JSON_EXTRACT(specifications, '$.ram') = '16GB';

-- Update JSON data
UPDATE products 
SET specifications = JSON_SET(specifications, '$.ram', '32GB')
WHERE name = 'Laptop';

-- Add new JSON fields
UPDATE products 
SET metadata = JSON_INSERT(metadata, '$.price', 1299.99)
WHERE name = 'Laptop';

-- Remove JSON fields
UPDATE products 
SET metadata = JSON_REMOVE(metadata, '$.warranty')
WHERE name = 'Smartphone';

-- Aggregate JSON data
SELECT 
    JSON_EXTRACT(metadata, '$.brand') as brand,
    COUNT(*) as product_count,
    JSON_ARRAYAGG(name) as products
FROM products
GROUP BY JSON_EXTRACT(metadata, '$.brand');

-- Create JSON from relational data
SELECT 
    JSON_OBJECT(
        'id', id,
        'name', name,
        'specifications', specifications,
        'metadata', metadata
    ) as product_json
FROM products;

-- Validate JSON
SELECT 
    name,
    JSON_VALID(specifications) as is_valid_specs,
    JSON_VALID(metadata) as is_valid_metadata
FROM products;

-- Get JSON keys
SELECT 
    name,
    JSON_KEYS(specifications) as spec_keys,
    JSON_KEYS(metadata) as metadata_keys
FROM products;

-- Get JSON length
SELECT 
    name,
    JSON_LENGTH(specifications) as spec_length,
    JSON_LENGTH(metadata) as metadata_length
FROM products;
```

**Total MySQL Questions: 100+ with comprehensive explanations, real-world examples, and advanced concepts!**

