# Interview Quick Reference Cheat Sheet
### Last-Minute Review Guide

---

## 🚀 Quick Access

- [PHP Quick Reference](#php-quick-reference)
- [MySQL Quick Reference](#mysql-quick-reference)
- [Node.js Quick Reference](#nodejs-quick-reference)
- [MongoDB Quick Reference](#mongodb-quick-reference)
- [Common Algorithms](#common-algorithms)
- [Design Patterns](#design-patterns)
- [Time Complexity](#time-complexity)
- [Must-Know Concepts](#must-know-concepts)

---

## PHP Quick Reference

### Most Important Functions
```php
// Array Functions
array_map($callback, $array)          // Transform
array_filter($array, $callback)       // Filter
array_reduce($array, $callback, $init) // Reduce
in_array($needle, $haystack)          // Check existence
array_search($needle, $haystack)      // Find key
array_key_exists($key, $array)        // Check key
usort($array, $callback)              // Custom sort
array_merge($arr1, $arr2)             // Merge arrays
array_column($array, 'column')        // Extract column
array_unique($array)                  // Remove duplicates

// String Functions
strlen($string)                       // Length
substr($string, $start, $length)      // Substring
str_replace($search, $replace, $str)  // Replace
explode($delimiter, $string)          // Split to array
implode($glue, $array)                // Join array
trim($string)                         // Remove whitespace
strtolower($string)                   // Lowercase
strtoupper($string)                   // Uppercase
strpos($haystack, $needle)            // Find position

// Date/Time
date('Y-m-d H:i:s')                   // Current datetime
strtotime('2025-01-01')               // String to timestamp
time()                                // Current timestamp
```

### OOP Quick Reference
```php
// Class Definition
class ClassName {
    private $property;                 // Private property
    protected $property;               // Protected property
    public $property;                  // Public property
    
    public function __construct() {}   // Constructor
    public function __destruct() {}    // Destructor
    public function method() {}        // Method
    
    public static function static() {} // Static method
    
    private function private() {}      // Private method
}

// Inheritance
class Child extends Parent {
    public function method() {
        parent::method();              // Call parent
    }
}

// Interface
interface MyInterface {
    public function method();
}

class MyClass implements MyInterface {
    public function method() {}
}

// Trait
trait MyTrait {
    public function method() {}
}

class MyClass {
    use MyTrait;
}
```

---

## MySQL Quick Reference

### Common Queries
```sql
-- SELECT with JOIN
SELECT u.*, o.order_id, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE u.active = 1
ORDER BY o.created_at DESC
LIMIT 10 OFFSET 0;

-- Aggregate Functions
SELECT 
    category,
    COUNT(*) as count,
    SUM(price) as total,
    AVG(price) as average,
    MIN(price) as minimum,
    MAX(price) as maximum
FROM products
GROUP BY category
HAVING count > 10;

-- Subquery
SELECT * FROM products
WHERE price > (
    SELECT AVG(price) FROM products
);

-- UPDATE with JOIN
UPDATE orders o
INNER JOIN users u ON o.user_id = u.id
SET o.discount = 10
WHERE u.is_premium = 1;

-- Window Functions (MySQL 8+)
SELECT 
    product_name,
    price,
    RANK() OVER (ORDER BY price DESC) as rank,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) as row_num
FROM products;
```

### Index Types
```sql
-- Primary Key
ALTER TABLE users ADD PRIMARY KEY (id);

-- Unique Index
CREATE UNIQUE INDEX idx_email ON users(email);

-- Regular Index
CREATE INDEX idx_name ON users(name);

-- Composite Index
CREATE INDEX idx_category_price ON products(category, price);

-- Full-text Index
CREATE FULLTEXT INDEX ft_content ON posts(title, content);

-- Show Indexes
SHOW INDEX FROM table_name;
```

---

## Node.js Quick Reference

### Array Methods
```javascript
// Transform
arr.map(x => x * 2)                   // [1,2,3] → [2,4,6]
arr.filter(x => x > 2)                // [1,2,3,4] → [3,4]
arr.reduce((sum, x) => sum + x, 0)    // [1,2,3] → 6
arr.flatMap(x => [x, x*2])            // [1,2] → [1,2,2,4]
arr.flat()                            // [[1,2],[3,4]] → [1,2,3,4]

// Search
arr.find(x => x > 2)                  // First match or undefined
arr.findIndex(x => x > 2)             // Index or -1
arr.includes(2)                       // true/false
arr.indexOf(2)                        // Index or -1
arr.some(x => x > 2)                  // Any match?
arr.every(x => x > 2)                 // All match?

// Modify
arr.push(item)                        // Add to end
arr.pop()                             // Remove from end
arr.shift()                           // Remove from start
arr.unshift(item)                     // Add to start
arr.splice(index, count, ...items)    // Add/remove at position
arr.slice(start, end)                 // Extract portion

// Sort
arr.sort((a, b) => a - b)             // Ascending
arr.sort((a, b) => b - a)             // Descending
arr.reverse()                         // Reverse order
```

### Async Patterns
```javascript
// Promises
Promise.resolve(value)
Promise.reject(error)
Promise.all([p1, p2, p3])             // All must succeed
Promise.race([p1, p2, p3])            // First to complete
Promise.allSettled([p1, p2, p3])      // All complete (success/fail)
Promise.any([p1, p2, p3])             // First successful

// Async/Await
async function example() {
    try {
        const result = await asyncOperation();
        return result;
    } catch (error) {
        console.error(error);
        throw error;
    }
}

// Common Patterns
// Sequential
for (const item of items) {
    await processItem(item);
}

// Parallel
await Promise.all(items.map(item => processItem(item)));

// Batch Processing
const chunks = [];
for (let i = 0; i < items.length; i += batchSize) {
    chunks.push(items.slice(i, i + batchSize));
}
for (const chunk of chunks) {
    await Promise.all(chunk.map(processItem));
}
```

---

## MongoDB Quick Reference

### CRUD Operations
```javascript
// Create
db.collection.insertOne({ name: 'John', age: 30 })
db.collection.insertMany([{ }, { }])

// Read
db.collection.find({ age: { $gt: 25 } })
db.collection.findOne({ _id: id })
db.collection.find().limit(10).skip(20)
db.collection.find().sort({ age: -1 })

// Update
db.collection.updateOne(
    { _id: id },
    { $set: { name: 'Jane' } }
)
db.collection.updateMany(
    { status: 'pending' },
    { $set: { status: 'processed' } }
)

// Delete
db.collection.deleteOne({ _id: id })
db.collection.deleteMany({ status: 'old' })

// Operators
{ $eq: value }                        // Equal
{ $ne: value }                        // Not equal
{ $gt: value }                        // Greater than
{ $gte: value }                       // Greater than or equal
{ $lt: value }                        // Less than
{ $lte: value }                       // Less than or equal
{ $in: [val1, val2] }                // In array
{ $nin: [val1, val2] }               // Not in array
{ $exists: true }                     // Field exists
{ $regex: /pattern/ }                 // Regex match
```

### Aggregation Pipeline
```javascript
db.collection.aggregate([
    // Stage 1: Filter
    { $match: { status: 'active' } },
    
    // Stage 2: Lookup (join)
    { $lookup: {
        from: 'orders',
        localField: '_id',
        foreignField: 'user_id',
        as: 'orders'
    }},
    
    // Stage 3: Unwind array
    { $unwind: '$orders' },
    
    // Stage 4: Group and aggregate
    { $group: {
        _id: '$category',
        total: { $sum: '$orders.total' },
        count: { $sum: 1 },
        avg: { $avg: '$orders.total' }
    }},
    
    // Stage 5: Sort
    { $sort: { total: -1 } },
    
    // Stage 6: Limit
    { $limit: 10 },
    
    // Stage 7: Project
    { $project: {
        category: '$_id',
        total: 1,
        count: 1,
        avg: { $round: ['$avg', 2] }
    }}
])
```

---

## Common Algorithms

### 1. Two Sum (Array)
```javascript
function twoSum(nums, target) {
    const map = new Map();
    for (let i = 0; i < nums.length; i++) {
        const complement = target - nums[i];
        if (map.has(complement)) {
            return [map.get(complement), i];
        }
        map.set(nums[i], i);
    }
    return [];
}
// Time: O(n), Space: O(n)
```

### 2. Fibonacci
```javascript
// Recursive with memoization
function fibonacci(n, memo = {}) {
    if (n <= 1) return n;
    if (memo[n]) return memo[n];
    memo[n] = fibonacci(n-1, memo) + fibonacci(n-2, memo);
    return memo[n];
}
// Time: O(n), Space: O(n)
```

### 3. Binary Search
```javascript
function binarySearch(arr, target) {
    let left = 0;
    let right = arr.length - 1;
    
    while (left <= right) {
        const mid = Math.floor((left + right) / 2);
        if (arr[mid] === target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
// Time: O(log n), Space: O(1)
```

### 4. Quick Sort
```javascript
function quickSort(arr) {
    if (arr.length <= 1) return arr;
    
    const pivot = arr[Math.floor(arr.length / 2)];
    const left = arr.filter(x => x < pivot);
    const middle = arr.filter(x => x === pivot);
    const right = arr.filter(x => x > pivot);
    
    return [...quickSort(left), ...middle, ...quickSort(right)];
}
// Time: O(n log n) average, Space: O(n)
```

### 5. Reverse Linked List
```javascript
function reverseList(head) {
    let prev = null;
    let current = head;
    
    while (current) {
        const next = current.next;
        current.next = prev;
        prev = current;
        current = next;
    }
    
    return prev;
}
// Time: O(n), Space: O(1)
```

---

## Design Patterns

### 1. Singleton
```javascript
class Singleton {
    static instance;
    
    constructor() {
        if (Singleton.instance) {
            return Singleton.instance;
        }
        Singleton.instance = this;
    }
}
```

### 2. Factory
```javascript
class Factory {
    static create(type) {
        switch(type) {
            case 'A': return new ProductA();
            case 'B': return new ProductB();
            default: throw new Error('Invalid type');
        }
    }
}
```

### 3. Observer
```javascript
class Subject {
    constructor() {
        this.observers = [];
    }
    
    subscribe(observer) {
        this.observers.push(observer);
    }
    
    notify(data) {
        this.observers.forEach(obs => obs.update(data));
    }
}
```

### 4. Strategy
```javascript
class Context {
    constructor(strategy) {
        this.strategy = strategy;
    }
    
    execute() {
        return this.strategy.algorithm();
    }
}
```

---

## Time Complexity

### Common Complexities
```
O(1)         - Constant      - Hash table lookup
O(log n)     - Logarithmic   - Binary search
O(n)         - Linear        - Array iteration
O(n log n)   - Linearithmic  - Merge sort, Quick sort
O(n²)        - Quadratic     - Nested loops
O(2^n)       - Exponential   - Recursive fibonacci
O(n!)        - Factorial     - Permutations
```

### Array Operations
| Operation | Best | Average | Worst |
|-----------|------|---------|-------|
| Access    | O(1) | O(1)    | O(1)  |
| Search    | O(1) | O(n)    | O(n)  |
| Insert    | O(1) | O(n)    | O(n)  |
| Delete    | O(1) | O(n)    | O(n)  |

---

## Must-Know Concepts

### PHP
✅ OOP principles (Encapsulation, Inheritance, Polymorphism, Abstraction)  
✅ Design patterns (Singleton, Factory, Observer)  
✅ PHP 8 features (Named args, Union types, Attributes)  
✅ Composer and autoloading  
✅ PSR standards  

### MySQL
✅ JOIN types and when to use them  
✅ Index types and strategies  
✅ Query optimization  
✅ Transactions and ACID  
✅ Normalization  

### Node.js
✅ Event loop phases  
✅ Async patterns (Callbacks, Promises, Async/Await)  
✅ Error handling  
✅ Streams and buffers  
✅ Module systems  

### MongoDB
✅ CRUD operations  
✅ Aggregation pipeline  
✅ Indexing strategies  
✅ Schema design  
✅ Replication and sharding  

### General
✅ REST API design  
✅ Authentication (JWT, Session)  
✅ Security (XSS, CSRF, SQL Injection)  
✅ Caching strategies  
✅ Scalability principles  

---

## 🎯 Interview Day Checklist

### Before Interview:
- [ ] Get good sleep
- [ ] Eat healthy meal
- [ ] Test equipment (camera, mic)
- [ ] Have water nearby
- [ ] Have pen and paper
- [ ] Review this cheat sheet

### During Interview:
- [ ] Listen carefully
- [ ] Think aloud
- [ ] Ask clarifying questions
- [ ] Start with simple solution
- [ ] Optimize after it works
- [ ] Test your code mentally
- [ ] Discuss trade-offs

### Common Interview Questions:
1. Tell me about yourself
2. What's your biggest weakness?
3. Why are you leaving current job?
4. Describe a challenging project
5. How do you handle disagreements?
6. Where do you see yourself in 5 years?

---

## 🚀 Last Minute Tips

**30 Minutes Before:**
- Quick review of this cheat sheet
- Take deep breaths
- Stay positive
- Be yourself

**During Interview:**
- Smile and be friendly
- Make eye contact (for video)
- Take your time
- It's okay to say "I don't know, but..."
- Show your thought process
- Be honest

**After Interview:**
- Send thank you email within 24 hours
- Reflect on what went well
- Note areas for improvement
- Follow up after 1 week if no response

---

## 💪 Remember:

> "You've prepared for this. Trust your knowledge and skills!"

**Good luck! You've got this!** 🎉

---

**Print this cheat sheet and review it before your interview!** 📄

