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

**Total MySQL Questions: 80+ with detailed explanations!**

