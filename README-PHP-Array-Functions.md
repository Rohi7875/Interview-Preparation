# PHP Array Functions - Complete Guide
### Real-World Examples for Interview Preparation

---

## Table of Contents
1. [Array Creation & Manipulation](#array-creation--manipulation)
2. [Array Traversal Functions](#array-traversal-functions)
3. [Array Search & Filter](#array-search--filter)
4. [Array Sorting](#array-sorting)
5. [Array Merging & Splitting](#array-merging--splitting)
6. [Array Transformation](#array-transformation)
7. [Array Comparison](#array-comparison)
8. [Array Keys & Values](#array-keys--values)
9. [Array Stack & Queue Operations](#array-stack--queue-operations)
10. [Real-World Use Cases](#real-world-use-cases)

---

## Array Creation & Manipulation

### Q1. array() vs [] - Array Creation
**Answer:** Both create arrays, but [] is the modern short syntax (PHP 5.4+).

**Real-time Example:**
```php
// E-commerce Product Management System

class ProductManager {
    private $products;
    
    public function __construct() {
        // Method 1: Using array()
        $this->products = array();
        
        // Method 2: Using [] (preferred)
        $this->products = [];
    }
    
    public function initializeProducts() {
        // Creating indexed array
        $categories = ['Electronics', 'Clothing', 'Books', 'Food'];
        
        // Creating associative array
        $product = [
            'id' => 1,
            'name' => 'Laptop',
            'price' => 999.99,
            'stock' => 50,
            'category' => 'Electronics'
        ];
        
        // Multi-dimensional array
        $products = [
            [
                'id' => 1,
                'name' => 'Laptop',
                'price' => 999.99,
                'specs' => [
                    'ram' => '16GB',
                    'storage' => '512GB SSD',
                    'processor' => 'Intel i7'
                ]
            ],
            [
                'id' => 2,
                'name' => 'Mouse',
                'price' => 29.99,
                'specs' => [
                    'type' => 'Wireless',
                    'dpi' => '1600',
                    'buttons' => 5
                ]
            ]
        ];
        
        return $products;
    }
}

// Usage
$manager = new ProductManager();
$products = $manager->initializeProducts();
print_r($products);
```

---

## Array Traversal Functions

### Q2. foreach vs array_walk vs array_map
**Answer:** Different ways to iterate arrays with different purposes.

**Real-time Example:**
```php
// User Processing System

class UserProcessor {
    
    // Method 1: foreach - Most common, flexible
    public function processUsersForEach($users) {
        $processed = [];
        
        foreach ($users as $key => $user) {
            $processed[$key] = [
                'id' => $user['id'],
                'name' => strtoupper($user['name']),
                'email' => strtolower($user['email']),
                'status' => $user['active'] ? 'Active' : 'Inactive'
            ];
        }
        
        return $processed;
    }
    
    // Method 2: array_walk - Modifies array by reference
    public function processUsersWalk($users) {
        array_walk($users, function(&$user, $key) {
            $user['name'] = strtoupper($user['name']);
            $user['email'] = strtolower($user['email']);
            $user['processed_at'] = date('Y-m-d H:i:s');
        });
        
        return $users;
    }
    
    // Method 3: array_map - Returns new array
    public function processUsersMap($users) {
        return array_map(function($user) {
            return [
                'id' => $user['id'],
                'full_name' => $user['name'],
                'email' => $user['email'],
                'display_name' => strtoupper($user['name']),
                'joined' => date('Y-m-d', strtotime($user['created_at']))
            ];
        }, $users);
    }
    
    // Real-world example: Process orders with different methods
    public function processOrders($orders) {
        // Calculate totals using foreach
        $totalRevenue = 0;
        foreach ($orders as $order) {
            $totalRevenue += $order['total'];
        }
        
        // Add shipping cost using array_walk
        array_walk($orders, function(&$order) {
            $order['shipping'] = $order['total'] > 100 ? 0 : 10;
            $order['final_total'] = $order['total'] + $order['shipping'];
        });
        
        // Extract order IDs using array_map
        $orderIds = array_map(function($order) {
            return $order['id'];
        }, $orders);
        
        return [
            'total_revenue' => $totalRevenue,
            'orders' => $orders,
            'order_ids' => $orderIds
        ];
    }
}

// Usage
$users = [
    ['id' => 1, 'name' => 'John Doe', 'email' => 'JOHN@EXAMPLE.COM', 'active' => true, 'created_at' => '2024-01-01'],
    ['id' => 2, 'name' => 'jane smith', 'email' => 'Jane@Example.com', 'active' => false, 'created_at' => '2024-02-01']
];

$processor = new UserProcessor();

echo "=== foreach Method ===\n";
print_r($processor->processUsersForEach($users));

echo "\n=== array_walk Method ===\n";
print_r($processor->processUsersWalk($users));

echo "\n=== array_map Method ===\n";
print_r($processor->processUsersMap($users));
```

---

## Array Search & Filter

### Q3. in_array vs array_search vs array_key_exists
**Answer:** Different methods to search arrays with different return values.

**Real-time Example:**
```php
// E-commerce Cart System

class ShoppingCart {
    private $cart = [];
    private $products = [
        'PROD001' => ['id' => 'PROD001', 'name' => 'Laptop', 'price' => 999.99, 'stock' => 10],
        'PROD002' => ['id' => 'PROD002', 'name' => 'Mouse', 'price' => 29.99, 'stock' => 50],
        'PROD003' => ['id' => 'PROD003', 'name' => 'Keyboard', 'price' => 79.99, 'stock' => 30]
    ];
    
    // in_array - Check if value exists (returns boolean)
    public function isProductInCart($productId) {
        $cartProductIds = array_column($this->cart, 'product_id');
        
        if (in_array($productId, $cartProductIds)) {
            return true;
        }
        
        return false;
    }
    
    // array_search - Find key/index of value (returns key or false)
    public function findProductIndex($productId) {
        $cartProductIds = array_column($this->cart, 'product_id');
        
        $index = array_search($productId, $cartProductIds);
        
        if ($index !== false) {
            return $index;
        }
        
        return null;
    }
    
    // array_key_exists - Check if key exists (returns boolean)
    public function productExists($productId) {
        return array_key_exists($productId, $this->products);
    }
    
    // Real-world example: Add product to cart
    public function addToCart($productId, $quantity = 1) {
        // Check if product exists using array_key_exists
        if (!array_key_exists($productId, $this->products)) {
            throw new Exception("Product not found");
        }
        
        $product = $this->products[$productId];
        
        // Check stock
        if ($product['stock'] < $quantity) {
            throw new Exception("Insufficient stock");
        }
        
        // Check if product already in cart using in_array
        $cartProductIds = array_column($this->cart, 'product_id');
        
        if (in_array($productId, $cartProductIds)) {
            // Find and update quantity using array_search
            $index = array_search($productId, $cartProductIds);
            $this->cart[$index]['quantity'] += $quantity;
        } else {
            // Add new item
            $this->cart[] = [
                'product_id' => $productId,
                'name' => $product['name'],
                'price' => $product['price'],
                'quantity' => $quantity
            ];
        }
        
        return $this->cart;
    }
    
    // Update quantity
    public function updateQuantity($productId, $quantity) {
        $index = $this->findProductIndex($productId);
        
        if ($index !== null) {
            $this->cart[$index]['quantity'] = $quantity;
            return true;
        }
        
        return false;
    }
    
    // Remove from cart
    public function removeFromCart($productId) {
        $index = $this->findProductIndex($productId);
        
        if ($index !== null) {
            array_splice($this->cart, $index, 1);
            return true;
        }
        
        return false;
    }
    
    // Get cart total
    public function getTotal() {
        $total = 0;
        
        foreach ($this->cart as $item) {
            $total += $item['price'] * $item['quantity'];
        }
        
        return $total;
    }
    
    public function getCart() {
        return $this->cart;
    }
}

// Usage
$cart = new ShoppingCart();

// Add products
$cart->addToCart('PROD001', 1);
$cart->addToCart('PROD002', 2);
$cart->addToCart('PROD001', 1); // Increases quantity

echo "Cart Contents:\n";
print_r($cart->getCart());

echo "\nTotal: $" . $cart->getTotal();

// Check if product in cart
echo "\nIs PROD001 in cart? " . ($cart->isProductInCart('PROD001') ? 'Yes' : 'No');

// Update quantity
$cart->updateQuantity('PROD002', 5);

// Remove product
$cart->removeFromCart('PROD001');

echo "\n\nUpdated Cart:\n";
print_r($cart->getCart());
```

### Q4. array_filter - Filter Array Elements
**Answer:** Filters elements of an array using a callback function.

**Real-time Example:**
```php
// Product Filtering System

class ProductFilter {
    private $products;
    
    public function __construct($products) {
        $this->products = $products;
    }
    
    // Filter by price range
    public function filterByPrice($minPrice, $maxPrice) {
        return array_filter($this->products, function($product) use ($minPrice, $maxPrice) {
            return $product['price'] >= $minPrice && $product['price'] <= $maxPrice;
        });
    }
    
    // Filter by category
    public function filterByCategory($category) {
        return array_filter($this->products, function($product) use ($category) {
            return $product['category'] === $category;
        });
    }
    
    // Filter in stock products
    public function filterInStock() {
        return array_filter($this->products, function($product) {
            return $product['stock'] > 0;
        });
    }
    
    // Filter by rating
    public function filterByRating($minRating) {
        return array_filter($this->products, function($product) use ($minRating) {
            return $product['rating'] >= $minRating;
        });
    }
    
    // Complex filtering with multiple criteria
    public function filterAdvanced($criteria) {
        return array_filter($this->products, function($product) use ($criteria) {
            $match = true;
            
            // Price range
            if (isset($criteria['min_price'])) {
                $match = $match && ($product['price'] >= $criteria['min_price']);
            }
            
            if (isset($criteria['max_price'])) {
                $match = $match && ($product['price'] <= $criteria['max_price']);
            }
            
            // Category
            if (isset($criteria['category'])) {
                $match = $match && ($product['category'] === $criteria['category']);
            }
            
            // In stock
            if (isset($criteria['in_stock']) && $criteria['in_stock']) {
                $match = $match && ($product['stock'] > 0);
            }
            
            // Rating
            if (isset($criteria['min_rating'])) {
                $match = $match && ($product['rating'] >= $criteria['min_rating']);
            }
            
            // Featured
            if (isset($criteria['featured']) && $criteria['featured']) {
                $match = $match && $product['is_featured'];
            }
            
            return $match;
        });
    }
    
    // Filter and preserve keys (using ARRAY_FILTER_USE_BOTH)
    public function filterWithKeys($callback) {
        return array_filter($this->products, $callback, ARRAY_FILTER_USE_BOTH);
    }
}

// Sample products
$products = [
    ['id' => 1, 'name' => 'Laptop', 'price' => 999.99, 'category' => 'Electronics', 'stock' => 10, 'rating' => 4.5, 'is_featured' => true],
    ['id' => 2, 'name' => 'Mouse', 'price' => 29.99, 'category' => 'Electronics', 'stock' => 50, 'rating' => 4.0, 'is_featured' => false],
    ['id' => 3, 'name' => 'Book', 'price' => 19.99, 'category' => 'Books', 'stock' => 0, 'rating' => 4.8, 'is_featured' => true],
    ['id' => 4, 'name' => 'Shirt', 'price' => 39.99, 'category' => 'Clothing', 'stock' => 25, 'rating' => 3.5, 'is_featured' => false],
    ['id' => 5, 'name' => 'Keyboard', 'price' => 79.99, 'category' => 'Electronics', 'stock' => 30, 'rating' => 4.2, 'is_featured' => true]
];

$filter = new ProductFilter($products);

// Filter by price range
echo "=== Products between $20 and $100 ===\n";
print_r($filter->filterByPrice(20, 100));

// Filter by category
echo "\n=== Electronics ===\n";
print_r($filter->filterByCategory('Electronics'));

// Filter in stock products
echo "\n=== In Stock Products ===\n";
print_r($filter->filterInStock());

// Advanced filtering
echo "\n=== Advanced Filter (Electronics, In Stock, Rating >= 4.0, Featured) ===\n";
$results = $filter->filterAdvanced([
    'category' => 'Electronics',
    'in_stock' => true,
    'min_rating' => 4.0,
    'featured' => true
]);
print_r($results);
```

---

## Array Sorting

### Q5. sort vs asort vs ksort vs usort
**Answer:** Different sorting functions for different sorting needs.

**Real-time Example:**
```php
// Product Sorting System

class ProductSorter {
    
    // sort() - Sort by values, reindex keys (ascending)
    public function sortByValuesReindex($array) {
        sort($array);
        return $array;
    }
    
    // rsort() - Sort by values descending, reindex keys
    public function sortByValuesReverseReindex($array) {
        rsort($array);
        return $array;
    }
    
    // asort() - Sort by values, maintain keys (ascending)
    public function sortByValuesMaintainKeys($array) {
        asort($array);
        return $array;
    }
    
    // arsort() - Sort by values descending, maintain keys
    public function sortByValuesReverseMaintainKeys($array) {
        arsort($array);
        return $array;
    }
    
    // ksort() - Sort by keys (ascending)
    public function sortByKeys($array) {
        ksort($array);
        return $array;
    }
    
    // krsort() - Sort by keys (descending)
    public function sortByKeysReverse($array) {
        krsort($array);
        return $array;
    }
    
    // usort() - Custom sort by values with callback
    public function customSortByValues($array, $callback) {
        usort($array, $callback);
        return $array;
    }
    
    // uasort() - Custom sort by values, maintain keys
    public function customSortByValuesMaintainKeys($array, $callback) {
        uasort($array, $callback);
        return $array;
    }
    
    // uksort() - Custom sort by keys
    public function customSortByKeys($array, $callback) {
        uksort($array, $callback);
        return $array;
    }
    
    // Real-world example: Sort products
    public function sortProducts($products, $sortBy = 'price', $order = 'asc') {
        switch ($sortBy) {
            case 'price':
                usort($products, function($a, $b) use ($order) {
                    if ($order === 'asc') {
                        return $a['price'] <=> $b['price'];
                    }
                    return $b['price'] <=> $a['price'];
                });
                break;
                
            case 'name':
                usort($products, function($a, $b) use ($order) {
                    if ($order === 'asc') {
                        return strcmp($a['name'], $b['name']);
                    }
                    return strcmp($b['name'], $a['name']);
                });
                break;
                
            case 'rating':
                usort($products, function($a, $b) use ($order) {
                    if ($order === 'asc') {
                        return $a['rating'] <=> $b['rating'];
                    }
                    return $b['rating'] <=> $a['rating'];
                });
                break;
                
            case 'stock':
                usort($products, function($a, $b) use ($order) {
                    if ($order === 'asc') {
                        return $a['stock'] <=> $b['stock'];
                    }
                    return $b['stock'] <=> $a['stock'];
                });
                break;
                
            default:
                // Sort by popularity (complex calculation)
                usort($products, function($a, $b) use ($order) {
                    $popularityA = ($a['rating'] * 0.6) + ($a['sales'] * 0.4);
                    $popularityB = ($b['rating'] * 0.6) + ($b['sales'] * 0.4);
                    
                    if ($order === 'asc') {
                        return $popularityA <=> $popularityB;
                    }
                    return $popularityB <=> $popularityA;
                });
        }
        
        return $products;
    }
    
    // Multi-level sorting
    public function sortProductsMultiLevel($products) {
        usort($products, function($a, $b) {
            // First by category
            $categoryCompare = strcmp($a['category'], $b['category']);
            if ($categoryCompare !== 0) {
                return $categoryCompare;
            }
            
            // Then by price (descending)
            $priceCompare = $b['price'] <=> $a['price'];
            if ($priceCompare !== 0) {
                return $priceCompare;
            }
            
            // Finally by name
            return strcmp($a['name'], $b['name']);
        });
        
        return $products;
    }
}

// Sample data
$products = [
    ['id' => 1, 'name' => 'Laptop', 'price' => 999.99, 'rating' => 4.5, 'stock' => 10, 'category' => 'Electronics', 'sales' => 150],
    ['id' => 2, 'name' => 'Mouse', 'price' => 29.99, 'rating' => 4.0, 'stock' => 50, 'category' => 'Electronics', 'sales' => 500],
    ['id' => 3, 'name' => 'Book', 'price' => 19.99, 'rating' => 4.8, 'stock' => 100, 'category' => 'Books', 'sales' => 200],
    ['id' => 4, 'name' => 'Keyboard', 'price' => 79.99, 'rating' => 4.2, 'stock' => 30, 'category' => 'Electronics', 'sales' => 300]
];

$sorter = new ProductSorter();

echo "=== Sort by Price (Ascending) ===\n";
print_r($sorter->sortProducts($products, 'price', 'asc'));

echo "\n=== Sort by Rating (Descending) ===\n";
print_r($sorter->sortProducts($products, 'rating', 'desc'));

echo "\n=== Sort by Popularity ===\n";
print_r($sorter->sortProducts($products, 'popularity', 'desc'));

echo "\n=== Multi-level Sort (Category > Price > Name) ===\n";
print_r($sorter->sortProductsMultiLevel($products));
```

---

## Array Merging & Splitting

### Q6. array_merge vs array_merge_recursive vs + operator
**Answer:** Different ways to combine arrays with different behaviors for duplicate keys.

**Syntax:**
```php
array_merge(array1, array2, ...) : array
array_merge_recursive(array1, array2, ...) : array
array1 + array2 : array
```

**Real-time Example:**
```php
// E-commerce Product Management System
class ProductManager {
    
    // array_merge - Overwrites duplicate keys
    public function mergeProductData($baseData, $additionalData) {
        return array_merge($baseData, $additionalData);
    }
    
    // array_merge_recursive - Preserves duplicate keys as arrays
    public function mergeProductSpecs($specs1, $specs2) {
        return array_merge_recursive($specs1, $specs2);
    }
    
    // + operator - Keeps first array values for duplicate keys
    public function mergeWithPriority($defaults, $userData) {
        return $userData + $defaults;
    }
    
    // Real-world example: Product configuration
    public function buildProductConfig($productId, $userPreferences = []) {
        // Default configuration
        $defaultConfig = [
            'display_mode' => 'grid',
            'items_per_page' => 20,
            'sort_by' => 'name',
            'filters' => [
                'price_range' => [0, 1000],
                'brands' => [],
                'categories' => []
            ],
            'features' => [
                'wishlist' => true,
                'compare' => true,
                'reviews' => true
            ]
        ];
        
        // User preferences
        $userConfig = [
            'display_mode' => 'list',
            'items_per_page' => 50,
            'filters' => [
                'price_range' => [100, 500],
                'brands' => ['Apple', 'Samsung']
            ],
            'features' => [
                'wishlist' => false
            ]
        ];
        
        // Merge with array_merge_recursive to preserve nested arrays
        $mergedConfig = array_merge_recursive($defaultConfig, $userConfig);
        
        // Clean up duplicate values in arrays
        $mergedConfig['filters']['brands'] = array_unique($mergedConfig['filters']['brands']);
        
        return $mergedConfig;
    }
    
    // Advanced merging with custom logic
    public function smartMerge($arrays) {
        $result = [];
        
        foreach ($arrays as $array) {
            foreach ($array as $key => $value) {
                if (isset($result[$key])) {
                    if (is_array($result[$key]) && is_array($value)) {
                        $result[$key] = $this->smartMerge([$result[$key], $value]);
                    } elseif (is_numeric($result[$key]) && is_numeric($value)) {
                        $result[$key] = $result[$key] + $value;
                    } else {
                        $result[$key] = $value; // Overwrite
                    }
                } else {
                    $result[$key] = $value;
                }
            }
        }
        
        return $result;
    }
}

// Usage examples
$manager = new ProductManager();

// Basic merging
$baseData = ['name' => 'Laptop', 'price' => 999.99];
$additionalData = ['price' => 899.99, 'discount' => 100];
$merged = $manager->mergeProductData($baseData, $additionalData);
// Result: ['name' => 'Laptop', 'price' => 899.99, 'discount' => 100]

// Recursive merging
$specs1 = ['ram' => '8GB', 'colors' => ['black', 'white']];
$specs2 = ['storage' => '256GB', 'colors' => ['red', 'blue']];
$recursiveMerged = $manager->mergeProductSpecs($specs1, $specs2);
// Result: ['ram' => '8GB', 'colors' => ['black', 'white', 'red', 'blue'], 'storage' => '256GB']

// Priority merging
$defaults = ['theme' => 'light', 'language' => 'en'];
$userData = ['language' => 'es'];
$priorityMerged = $manager->mergeWithPriority($defaults, $userData);
// Result: ['language' => 'es', 'theme' => 'light']
```

### Q7. array_slice vs array_splice - Array Extraction
**Answer:** Different methods to extract portions of arrays with different behaviors.

**Syntax:**
```php
array_slice(array, offset, length, preserve_keys) : array
array_splice(array, offset, length, replacement) : array
```

**Real-time Example:**
```php
// Pagination and Data Processing System
class DataProcessor {
    
    // array_slice - Extract without modifying original
    public function getPageData($data, $page, $perPage) {
        $offset = ($page - 1) * $perPage;
        return array_slice($data, $offset, $perPage, true);
    }
    
    // array_splice - Extract and modify original array
    public function removeAndGetItems(&$data, $start, $count) {
        return array_splice($data, $start, $count);
    }
    
    // Real-world example: Pagination system
    public function paginateResults($results, $currentPage = 1, $perPage = 10) {
        $totalItems = count($results);
        $totalPages = ceil($totalItems / $perPage);
        $offset = ($currentPage - 1) * $perPage;
        
        $pageData = array_slice($results, $offset, $perPage, true);
        
        return [
            'data' => $pageData,
            'pagination' => [
                'current_page' => $currentPage,
                'per_page' => $perPage,
                'total_items' => $totalItems,
                'total_pages' => $totalPages,
                'has_next' => $currentPage < $totalPages,
                'has_prev' => $currentPage > 1,
                'next_page' => $currentPage < $totalPages ? $currentPage + 1 : null,
                'prev_page' => $currentPage > 1 ? $currentPage - 1 : null
            ]
        ];
    }
    
    // Chunk processing for large datasets
    public function processInChunks($data, $chunkSize, $callback) {
        $results = [];
        $totalChunks = ceil(count($data) / $chunkSize);
        
        for ($i = 0; $i < $totalChunks; $i++) {
            $chunk = array_slice($data, $i * $chunkSize, $chunkSize);
            $processedChunk = $callback($chunk, $i);
            $results = array_merge($results, $processedChunk);
        }
        
        return $results;
    }
    
    // Remove items and get them
    public function extractAndRemove(&$data, $condition) {
        $extracted = [];
        $remaining = [];
        
        foreach ($data as $key => $item) {
            if ($condition($item)) {
                $extracted[$key] = $item;
            } else {
                $remaining[$key] = $item;
            }
        }
        
        $data = $remaining;
        return $extracted;
    }
}

// Usage
$processor = new DataProcessor();

// Pagination
$products = range(1, 100); // Simulate 100 products
$page1 = $processor->getPageData($products, 1, 10);
$page2 = $processor->getPageData($products, 2, 10);

// Full pagination with metadata
$pagination = $processor->paginateResults($products, 2, 15);
print_r($pagination);

// Chunk processing
$largeDataset = range(1, 1000);
$processed = $processor->processInChunks($largeDataset, 100, function($chunk, $index) {
    return array_map(function($item) {
        return $item * 2; // Process each item
    }, $chunk);
});
```

### Q8. array_chunk - Split Array into Chunks
**Answer:** Divides an array into smaller arrays of specified size.

**Syntax:**
```php
array_chunk(array, size, preserve_keys) : array
```

**Real-time Example:**
```php
// Batch Processing System
class BatchProcessor {
    
    public function processBatch($items, $batchSize, $processor) {
        $chunks = array_chunk($items, $batchSize, true);
        $results = [];
        
        foreach ($chunks as $index => $chunk) {
            echo "Processing batch " . ($index + 1) . " of " . count($chunks) . "\n";
            $batchResult = $processor($chunk);
            $results = array_merge($results, $batchResult);
        }
        
        return $results;
    }
    
    // Real-world: Email sending in batches
    public function sendBulkEmails($recipients, $batchSize = 50) {
        $chunks = array_chunk($recipients, $batchSize);
        $sentCount = 0;
        
        foreach ($chunks as $chunk) {
            $this->sendEmailBatch($chunk);
            $sentCount += count($chunk);
            
            // Add delay between batches to avoid rate limiting
            sleep(1);
        }
        
        return $sentCount;
    }
    
    private function sendEmailBatch($recipients) {
        // Simulate email sending
        foreach ($recipients as $recipient) {
            echo "Sending email to: {$recipient['email']}\n";
        }
    }
    
    // Database batch operations
    public function batchInsert($table, $data, $batchSize = 1000) {
        $chunks = array_chunk($data, $batchSize);
        $insertedCount = 0;
        
        foreach ($chunks as $chunk) {
            $this->insertBatch($table, $chunk);
            $insertedCount += count($chunk);
        }
        
        return $insertedCount;
    }
    
    private function insertBatch($table, $data) {
        // Simulate database insert
        echo "Inserting " . count($data) . " records into $table\n";
    }
}

// Usage
$processor = new BatchProcessor();

// Process large dataset in batches
$largeData = range(1, 10000);
$results = $processor->processBatch($largeData, 100, function($chunk) {
    return array_map(function($item) {
        return $item * 2;
    }, $chunk);
});

// Email sending
$recipients = [
    ['email' => 'user1@example.com', 'name' => 'User 1'],
    ['email' => 'user2@example.com', 'name' => 'User 2'],
    // ... more recipients
];
$sentCount = $processor->sendBulkEmails($recipients, 10);
```

---

## Array Transformation

### Q9. array_map vs array_walk vs array_reduce
**Answer:** Different array transformation functions with different purposes and return values.

**Syntax:**
```php
array_map(callback, array1, array2, ...) : array
array_walk(array, callback, userdata) : bool
array_reduce(array, callback, initial) : mixed
```

**Real-time Example:**
```php
// Data Processing and Analytics System
class DataAnalytics {
    
    // array_map - Transform each element, returns new array
    public function calculateTaxes($products) {
        return array_map(function($product) {
            return [
                'id' => $product['id'],
                'name' => $product['name'],
                'price' => $product['price'],
                'tax' => $product['price'] * 0.08,
                'total' => $product['price'] * 1.08
            ];
        }, $products);
    }
    
    // array_walk - Modify array by reference, returns boolean
    public function applyDiscount(&$products, $discountPercent) {
        array_walk($products, function(&$product) use ($discountPercent) {
            $product['price'] = $product['price'] * (1 - $discountPercent / 100);
            $product['discount_applied'] = $discountPercent;
        });
    }
    
    // array_reduce - Reduce array to single value
    public function calculateTotal($products) {
        return array_reduce($products, function($carry, $product) {
            return $carry + $product['price'];
        }, 0);
    }
    
    // Real-world example: E-commerce analytics
    public function analyzeSalesData($salesData) {
        // Calculate total revenue
        $totalRevenue = array_reduce($salesData, function($carry, $sale) {
            return $carry + $sale['amount'];
        }, 0);
        
        // Calculate average sale
        $averageSale = $totalRevenue / count($salesData);
        
        // Find top products
        $productTotals = [];
        array_walk($salesData, function($sale) use (&$productTotals) {
            $productId = $sale['product_id'];
            if (!isset($productTotals[$productId])) {
                $productTotals[$productId] = 0;
            }
            $productTotals[$productId] += $sale['amount'];
        });
        
        arsort($productTotals);
        $topProducts = array_slice($productTotals, 0, 5, true);
        
        // Calculate monthly totals
        $monthlyData = [];
        array_walk($salesData, function($sale) use (&$monthlyData) {
            $month = date('Y-m', strtotime($sale['date']));
            if (!isset($monthlyData[$month])) {
                $monthlyData[$month] = 0;
            }
            $monthlyData[$month] += $sale['amount'];
        });
        
        return [
            'total_revenue' => $totalRevenue,
            'average_sale' => $averageSale,
            'top_products' => $topProducts,
            'monthly_breakdown' => $monthlyData,
            'total_sales' => count($salesData)
        ];
    }
    
    // Complex data transformation
    public function transformUserData($users) {
        // Step 1: Filter active users
        $activeUsers = array_filter($users, function($user) {
            return $user['status'] === 'active';
        });
        
        // Step 2: Transform user data
        $transformedUsers = array_map(function($user) {
            return [
                'id' => $user['id'],
                'full_name' => $user['first_name'] . ' ' . $user['last_name'],
                'email' => strtolower($user['email']),
                'age' => date_diff(date_create($user['birth_date']), date_create('now'))->y,
                'membership_years' => date_diff(date_create($user['created_at']), date_create('now'))->y,
                'is_premium' => $user['subscription_type'] === 'premium'
            ];
        }, $activeUsers);
        
        // Step 3: Sort by membership years
        usort($transformedUsers, function($a, $b) {
            return $b['membership_years'] <=> $a['membership_years'];
        });
        
        return $transformedUsers;
    }
    
    // Advanced reduce example
    public function calculateStatistics($numbers) {
        return array_reduce($numbers, function($carry, $number) {
            $carry['sum'] += $number;
            $carry['count']++;
            $carry['min'] = min($carry['min'], $number);
            $carry['max'] = max($carry['max'], $number);
            return $carry;
        }, [
            'sum' => 0,
            'count' => 0,
            'min' => PHP_FLOAT_MAX,
            'max' => PHP_FLOAT_MIN
        ]);
    }
}

// Usage
$analytics = new DataAnalytics();

// Sample sales data
$salesData = [
    ['product_id' => 1, 'amount' => 100, 'date' => '2025-01-01'],
    ['product_id' => 2, 'amount' => 150, 'date' => '2025-01-02'],
    ['product_id' => 1, 'amount' => 200, 'date' => '2025-01-03'],
];

$analysis = $analytics->analyzeSalesData($salesData);
print_r($analysis);

// Transform user data
$users = [
    [
        'id' => 1,
        'first_name' => 'John',
        'last_name' => 'Doe',
        'email' => 'JOHN@EXAMPLE.COM',
        'birth_date' => '1990-01-01',
        'created_at' => '2020-01-01',
        'status' => 'active',
        'subscription_type' => 'premium'
    ],
    // ... more users
];

$transformedUsers = $analytics->transformUserData($users);
print_r($transformedUsers);
```

### Q10. array_column - Extract Column Values
**Answer:** Returns the values from a single column of the input array.

**Syntax:**
```php
array_column(array, column_key, index_key) : array
```

**Real-time Example:**
```php
// User Management System
class UserManager {
    
    public function extractUserEmails($users) {
        return array_column($users, 'email');
    }
    
    public function createUserLookup($users) {
        return array_column($users, 'name', 'id');
    }
    
    // Real-world example: Generate reports
    public function generateUserReport($users) {
        // Extract specific columns
        $userEmails = array_column($users, 'email');
        $userNames = array_column($users, 'name', 'id');
        $userRoles = array_column($users, 'role');
        
        // Count by role
        $roleCounts = array_count_values($userRoles);
        
        // Get unique roles
        $uniqueRoles = array_unique($userRoles);
        
        // Create email list for newsletter
        $newsletterList = array_filter($userEmails, function($email) {
            return filter_var($email, FILTER_VALIDATE_EMAIL);
        });
        
        return [
            'total_users' => count($users),
            'user_emails' => $userEmails,
            'user_lookup' => $userNames,
            'role_distribution' => $roleCounts,
            'unique_roles' => $uniqueRoles,
            'newsletter_recipients' => $newsletterList
        ];
    }
    
    // Advanced column extraction
    public function extractNestedData($users) {
        // Extract from nested arrays
        $addresses = array_column($users, 'address');
        $cities = array_column($addresses, 'city');
        $countries = array_column($addresses, 'country');
        
        // Create lookup tables
        $userToCity = array_column($users, 'address', 'id');
        $cityToUsers = [];
        
        foreach ($userToCity as $userId => $address) {
            $city = $address['city'];
            if (!isset($cityToUsers[$city])) {
                $cityToUsers[$city] = [];
            }
            $cityToUsers[$city][] = $userId;
        }
        
        return [
            'cities' => $cities,
            'countries' => $countries,
            'city_to_users' => $cityToUsers
        ];
    }
}

// Usage
$userManager = new UserManager();

$users = [
    [
        'id' => 1,
        'name' => 'John Doe',
        'email' => 'john@example.com',
        'role' => 'admin',
        'address' => [
            'city' => 'New York',
            'country' => 'USA'
        ]
    ],
    [
        'id' => 2,
        'name' => 'Jane Smith',
        'email' => 'jane@example.com',
        'role' => 'user',
        'address' => [
            'city' => 'London',
            'country' => 'UK'
        ]
    ]
];

$report = $userManager->generateUserReport($users);
print_r($report);
```

---

## Array Comparison

### Q11. array_diff vs array_intersect - Array Comparison
**Answer:** Different methods to compare arrays and find differences or similarities.

**Syntax:**
```php
array_diff(array1, array2, ...) : array
array_intersect(array1, array2, ...) : array
array_diff_assoc(array1, array2, ...) : array
array_intersect_assoc(array1, array2, ...) : array
```

**Real-time Example:**
```php
// Inventory Management System
class InventoryManager {
    
    // Find products that need restocking
    public function findLowStockProducts($currentStock, $minimumStock) {
        $lowStock = [];
        
        foreach ($currentStock as $productId => $quantity) {
            if (isset($minimumStock[$productId]) && $quantity < $minimumStock[$productId]) {
                $lowStock[$productId] = [
                    'current' => $quantity,
                    'minimum' => $minimumStock[$productId],
                    'needed' => $minimumStock[$productId] - $quantity
                ];
            }
        }
        
        return $lowStock;
    }
    
    // Compare inventory changes
    public function compareInventory($oldInventory, $newInventory) {
        $added = array_diff_assoc($newInventory, $oldInventory);
        $removed = array_diff_assoc($oldInventory, $newInventory);
        $changed = [];
        
        foreach ($added as $productId => $quantity) {
            if (isset($oldInventory[$productId])) {
                $changed[$productId] = [
                    'old' => $oldInventory[$productId],
                    'new' => $quantity,
                    'difference' => $quantity - $oldInventory[$productId]
                ];
                unset($added[$productId]);
            }
        }
        
        return [
            'added' => $added,
            'removed' => $removed,
            'changed' => $changed
        ];
    }
    
    // Find common products between warehouses
    public function findCommonProducts($warehouse1, $warehouse2) {
        return array_intersect_key($warehouse1, $warehouse2);
    }
    
    // Find products only in one warehouse
    public function findUniqueProducts($warehouse1, $warehouse2) {
        $onlyIn1 = array_diff_key($warehouse1, $warehouse2);
        $onlyIn2 = array_diff_key($warehouse2, $warehouse1);
        
        return [
            'only_in_warehouse1' => $onlyIn1,
            'only_in_warehouse2' => $onlyIn2
        ];
    }
    
    // Real-world example: Order processing
    public function processOrder($orderItems, $availableStock) {
        $processedItems = [];
        $unavailableItems = [];
        
        foreach ($orderItems as $productId => $requestedQuantity) {
            if (isset($availableStock[$productId])) {
                if ($availableStock[$productId] >= $requestedQuantity) {
                    $processedItems[$productId] = $requestedQuantity;
                    $availableStock[$productId] -= $requestedQuantity;
                } else {
                    $unavailableItems[$productId] = [
                        'requested' => $requestedQuantity,
                        'available' => $availableStock[$productId],
                        'shortage' => $requestedQuantity - $availableStock[$productId]
                    ];
                }
            } else {
                $unavailableItems[$productId] = [
                    'requested' => $requestedQuantity,
                    'available' => 0,
                    'shortage' => $requestedQuantity
                ];
            }
        }
        
        return [
            'processed' => $processedItems,
            'unavailable' => $unavailableItems,
            'updated_stock' => $availableStock
        ];
    }
}

// Usage
$inventoryManager = new InventoryManager();

// Sample data
$currentStock = [
    'PROD001' => 50,
    'PROD002' => 25,
    'PROD003' => 100,
    'PROD004' => 5
];

$minimumStock = [
    'PROD001' => 30,
    'PROD002' => 50,
    'PROD003' => 20,
    'PROD004' => 10
];

$lowStock = $inventoryManager->findLowStockProducts($currentStock, $minimumStock);
print_r($lowStock);

// Order processing
$orderItems = [
    'PROD001' => 20,
    'PROD002' => 30,
    'PROD005' => 10
];

$orderResult = $inventoryManager->processOrder($orderItems, $currentStock);
print_r($orderResult);
```

---

## Array Keys & Values

### Q12. array_keys vs array_values vs array_flip
**Answer:** Different functions to work with array keys and values.

**Syntax:**
```php
array_keys(array, search_value, strict) : array
array_values(array) : array
array_flip(array) : array
array_reverse(array, preserve_keys) : array
```

**Real-time Example:**
```php
// User Data Management System
class UserDataManager {
    
    // array_keys - Get all keys from array
    public function getUserIds($users) {
        return array_keys($users);
    }
    
    // array_values - Get all values from array
    public function getUserNames($users) {
        return array_values($users);
    }
    
    // array_flip - Exchange keys and values
    public function createUserLookup($users) {
        return array_flip($users);
    }
    
    // array_reverse - Reverse array order
    public function getRecentUsers($users) {
        return array_reverse($users, true);
    }
    
    // Real-world example: User analytics
    public function analyzeUserData($users) {
        // Get user IDs
        $userIds = array_keys($users);
        
        // Get user names
        $userNames = array_values($users);
        
        // Create reverse lookup (name => id)
        $nameToId = array_flip($users);
        
        // Get unique names
        $uniqueNames = array_unique($userNames);
        
        // Count name occurrences
        $nameCounts = array_count_values($userNames);
        
        // Find duplicate names
        $duplicateNames = array_filter($nameCounts, function($count) {
            return $count > 1;
        });
        
        return [
            'user_ids' => $userIds,
            'user_names' => $userNames,
            'name_to_id_lookup' => $nameToId,
            'unique_names' => $uniqueNames,
            'name_counts' => $nameCounts,
            'duplicate_names' => $duplicateNames,
            'total_users' => count($users)
        ];
    }
    
    // Advanced key/value manipulation
    public function transformUserRoles($users) {
        // Extract roles
        $roles = array_column($users, 'role', 'id');
        
        // Create role-based groups
        $roleGroups = [];
        foreach ($roles as $userId => $role) {
            $roleGroups[$role][] = $userId;
        }
        
        // Get unique roles
        $uniqueRoles = array_unique(array_values($roles));
        
        // Create role hierarchy
        $roleHierarchy = [
            'admin' => 3,
            'moderator' => 2,
            'user' => 1
        ];
        
        // Sort users by role hierarchy
        $sortedUsers = [];
        foreach ($roleGroups as $role => $userIds) {
            $sortedUsers[$role] = $userIds;
        }
        
        return [
            'roles' => $roles,
            'role_groups' => $roleGroups,
            'unique_roles' => $uniqueRoles,
            'role_hierarchy' => $roleHierarchy,
            'sorted_by_role' => $sortedUsers
        ];
    }
}

// Usage
$userManager = new UserDataManager();

$users = [
    1 => 'John Doe',
    2 => 'Jane Smith', 
    3 => 'John Doe', // Duplicate name
    4 => 'Bob Wilson',
    5 => 'Jane Smith' // Duplicate name
];

$analysis = $userManager->analyzeUserData($users);
print_r($analysis);

// Role-based example
$usersWithRoles = [
    ['id' => 1, 'name' => 'John', 'role' => 'admin'],
    ['id' => 2, 'name' => 'Jane', 'role' => 'user'],
    ['id' => 3, 'name' => 'Bob', 'role' => 'moderator'],
    ['id' => 4, 'name' => 'Alice', 'role' => 'user']
];

$roleAnalysis = $userManager->transformUserRoles($usersWithRoles);
print_r($roleAnalysis);
```

### Q13. array_unique vs array_count_values
**Answer:** Functions to handle duplicate values and count occurrences.

**Real-time Example:**
```php
// Product Catalog System
class ProductCatalog {
    
    // Remove duplicate products
    public function removeDuplicateProducts($products) {
        return array_unique($products, SORT_REGULAR);
    }
    
    // Count product occurrences
    public function countProductOccurrences($products) {
        return array_count_values($products);
    }
    
    // Find most popular products
    public function findPopularProducts($products, $limit = 5) {
        $counts = array_count_values($products);
        arsort($counts);
        return array_slice($counts, 0, $limit, true);
    }
    
    // Real-world example: E-commerce analytics
    public function analyzeProductViews($viewHistory) {
        // Get unique products viewed
        $uniqueProducts = array_unique($viewHistory);
        
        // Count views per product
        $viewCounts = array_count_values($viewHistory);
        
        // Sort by popularity
        arsort($viewCounts);
        
        // Get top 10 most viewed
        $topProducts = array_slice($viewCounts, 0, 10, true);
        
        // Find products with single view
        $singleViewProducts = array_filter($viewCounts, function($count) {
            return $count === 1;
        });
        
        // Calculate statistics
        $totalViews = array_sum($viewCounts);
        $averageViews = $totalViews / count($uniqueProducts);
        
        return [
            'unique_products' => $uniqueProducts,
            'view_counts' => $viewCounts,
            'top_products' => $topProducts,
            'single_view_products' => $singleViewProducts,
            'total_views' => $totalViews,
            'average_views' => $averageViews,
            'total_unique_products' => count($uniqueProducts)
        ];
    }
}

// Usage
$catalog = new ProductCatalog();

$viewHistory = [
    'PROD001', 'PROD002', 'PROD001', 'PROD003', 'PROD001',
    'PROD002', 'PROD004', 'PROD001', 'PROD002', 'PROD005'
];

$analysis = $catalog->analyzeProductViews($viewHistory);
print_r($analysis);
```

---

## Array Stack & Queue Operations

### Q14. array_push vs array_pop vs array_shift vs array_unshift
**Answer:** Different functions for stack (LIFO) and queue (FIFO) operations.

**Syntax:**
```php
array_push(array, value1, value2, ...) : int
array_pop(array) : mixed
array_shift(array) : mixed
array_unshift(array, value1, value2, ...) : int
```

**Real-time Example:**
```php
// Task Management System
class TaskManager {
    private $taskQueue = [];
    private $completedTasks = [];
    
    // Stack operations (LIFO)
    public function addTaskToStack($task) {
        array_push($this->taskQueue, $task);
        return count($this->taskQueue);
    }
    
    public function processLastTask() {
        if (empty($this->taskQueue)) {
            return null;
        }
        
        $task = array_pop($this->taskQueue);
        $this->completedTasks[] = $task;
        return $task;
    }
    
    // Queue operations (FIFO)
    public function addTaskToQueue($task) {
        array_unshift($this->taskQueue, $task);
        return count($this->taskQueue);
    }
    
    public function processFirstTask() {
        if (empty($this->taskQueue)) {
            return null;
        }
        
        $task = array_shift($this->taskQueue);
        $this->completedTasks[] = $task;
        return $task;
    }
    
    // Real-world example: Priority task system
    public function addPriorityTask($task, $priority = 'normal') {
        $taskWithPriority = [
            'task' => $task,
            'priority' => $priority,
            'added_at' => time()
        ];
        
        if ($priority === 'high') {
            // Add to front of queue
            array_unshift($this->taskQueue, $taskWithPriority);
        } else {
            // Add to back of queue
            array_push($this->taskQueue, $taskWithPriority);
        }
        
        return count($this->taskQueue);
    }
    
    public function processNextTask() {
        if (empty($this->taskQueue)) {
            return null;
        }
        
        // Process first task (FIFO)
        $task = array_shift($this->taskQueue);
        $this->completedTasks[] = $task;
        
        return $task;
    }
    
    public function getQueueStatus() {
        return [
            'pending_tasks' => count($this->taskQueue),
            'completed_tasks' => count($this->completedTasks),
            'next_task' => !empty($this->taskQueue) ? $this->taskQueue[0] : null,
            'last_completed' => !empty($this->completedTasks) ? end($this->completedTasks) : null
        ];
    }
    
    public function getTaskQueue() {
        return $this->taskQueue;
    }
    
    public function getCompletedTasks() {
        return $this->completedTasks;
    }
}

// Usage
$taskManager = new TaskManager();

// Add tasks with different priorities
$taskManager->addPriorityTask('Fix critical bug', 'high');
$taskManager->addPriorityTask('Update documentation', 'normal');
$taskManager->addPriorityTask('Code review', 'normal');
$taskManager->addPriorityTask('Security patch', 'high');

echo "Initial Queue:\n";
print_r($taskManager->getTaskQueue());

// Process tasks
echo "\nProcessing tasks:\n";
while ($task = $taskManager->processNextTask()) {
    echo "Processed: " . $task['task'] . " (Priority: " . $task['priority'] . ")\n";
}

echo "\nCompleted Tasks:\n";
print_r($taskManager->getCompletedTasks());

echo "\nQueue Status:\n";
print_r($taskManager->getQueueStatus());
```

### Q15. Array as Stack Implementation
**Real-time Example:**
```php
// Undo/Redo System
class UndoRedoManager {
    private $undoStack = [];
    private $redoStack = [];
    
    public function performAction($action) {
        // Add to undo stack
        array_push($this->undoStack, $action);
        
        // Clear redo stack when new action is performed
        $this->redoStack = [];
        
        return $action;
    }
    
    public function undo() {
        if (empty($this->undoStack)) {
            return null;
        }
        
        $action = array_pop($this->undoStack);
        array_push($this->redoStack, $action);
        
        return $action;
    }
    
    public function redo() {
        if (empty($this->redoStack)) {
            return null;
        }
        
        $action = array_pop($this->redoStack);
        array_push($this->undoStack, $action);
        
        return $action;
    }
    
    public function getStackStatus() {
        return [
            'undo_count' => count($this->undoStack),
            'redo_count' => count($this->redoStack),
            'can_undo' => !empty($this->undoStack),
            'can_redo' => !empty($this->redoStack)
        ];
    }
}

// Usage
$undoManager = new UndoRedoManager();

// Perform actions
$undoManager->performAction('Create document');
$undoManager->performAction('Add text');
$undoManager->performAction('Format text');

echo "After actions:\n";
print_r($undoManager->getStackStatus());

// Undo actions
echo "\nUndoing:\n";
echo "Undone: " . $undoManager->undo() . "\n";
echo "Undone: " . $undoManager->undo() . "\n";

echo "\nAfter undos:\n";
print_r($undoManager->getStackStatus());

// Redo actions
echo "\nRedoing:\n";
echo "Redone: " . $undoManager->redo() . "\n";

echo "\nFinal status:\n";
print_r($undoManager->getStackStatus());
```

---

## Real-World Use Cases

### Q16. E-commerce Shopping Cart System
**Complete implementation using multiple array functions:**

```php
class ShoppingCartSystem {
    private $cart = [];
    private $products = [];
    private $discounts = [];
    
    public function __construct() {
        $this->initializeProducts();
        $this->initializeDiscounts();
    }
    
    private function initializeProducts() {
        $this->products = [
            'PROD001' => [
                'id' => 'PROD001',
                'name' => 'Laptop',
                'price' => 999.99,
                'category' => 'Electronics',
                'stock' => 10,
                'rating' => 4.5
            ],
            'PROD002' => [
                'id' => 'PROD002', 
                'name' => 'Mouse',
                'price' => 29.99,
                'category' => 'Electronics',
                'stock' => 50,
                'rating' => 4.0
            ],
            'PROD003' => [
                'id' => 'PROD003',
                'name' => 'Book',
                'price' => 19.99,
                'category' => 'Books',
                'stock' => 100,
                'rating' => 4.8
            ]
        ];
    }
    
    private function initializeDiscounts() {
        $this->discounts = [
            'BULK10' => ['type' => 'percentage', 'value' => 10, 'min_quantity' => 5],
            'FREESHIP' => ['type' => 'shipping', 'value' => 0, 'min_total' => 100],
            'SAVE20' => ['type' => 'percentage', 'value' => 20, 'min_total' => 200]
        ];
    }
    
    // Add product to cart
    public function addToCart($productId, $quantity = 1) {
        if (!array_key_exists($productId, $this->products)) {
            throw new Exception("Product not found");
        }
        
        $product = $this->products[$productId];
        
        // Check stock
        if ($product['stock'] < $quantity) {
            throw new Exception("Insufficient stock");
        }
        
        // Check if product already in cart
        $cartProductIds = array_column($this->cart, 'product_id');
        
        if (in_array($productId, $cartProductIds)) {
            // Update quantity
            $index = array_search($productId, $cartProductIds);
            $this->cart[$index]['quantity'] += $quantity;
        } else {
            // Add new item
            array_push($this->cart, [
                'product_id' => $productId,
                'name' => $product['name'],
                'price' => $product['price'],
                'quantity' => $quantity,
                'category' => $product['category']
            ]);
        }
        
        return $this->cart;
    }
    
    // Remove from cart
    public function removeFromCart($productId) {
        $cartProductIds = array_column($this->cart, 'product_id');
        $index = array_search($productId, $cartProductIds);
        
        if ($index !== false) {
            array_splice($this->cart, $index, 1);
            return true;
        }
        
        return false;
    }
    
    // Apply discount
    public function applyDiscount($discountCode) {
        if (!array_key_exists($discountCode, $this->discounts)) {
            throw new Exception("Invalid discount code");
        }
        
        $discount = $this->discounts[$discountCode];
        $total = $this->getSubtotal();
        
        // Check minimum requirements
        if (isset($discount['min_total']) && $total < $discount['min_total']) {
            throw new Exception("Minimum total not met");
        }
        
        if (isset($discount['min_quantity'])) {
            $totalQuantity = array_sum(array_column($this->cart, 'quantity'));
            if ($totalQuantity < $discount['min_quantity']) {
                throw new Exception("Minimum quantity not met");
            }
        }
        
        return $discount;
    }
    
    // Calculate totals
    public function getSubtotal() {
        return array_reduce($this->cart, function($carry, $item) {
            return $carry + ($item['price'] * $item['quantity']);
        }, 0);
    }
    
    public function getTotal($discountCode = null) {
        $subtotal = $this->getSubtotal();
        $discount = 0;
        $shipping = $this->calculateShipping();
        
        if ($discountCode && array_key_exists($discountCode, $this->discounts)) {
            $discountData = $this->discounts[$discountCode];
            
            if ($discountData['type'] === 'percentage') {
                $discount = $subtotal * ($discountData['value'] / 100);
            } elseif ($discountData['type'] === 'shipping') {
                $shipping = 0;
            }
        }
        
        return $subtotal - $discount + $shipping;
    }
    
    private function calculateShipping() {
        $total = $this->getSubtotal();
        return $total >= 100 ? 0 : 10;
    }
    
    // Filter cart by category
    public function filterCartByCategory($category) {
        return array_filter($this->cart, function($item) use ($category) {
            return $item['category'] === $category;
        });
    }
    
    // Sort cart by price
    public function sortCartByPrice($order = 'asc') {
        if ($order === 'asc') {
            usort($this->cart, function($a, $b) {
                return $a['price'] <=> $b['price'];
            });
        } else {
            usort($this->cart, function($a, $b) {
                return $b['price'] <=> $a['price'];
            });
        }
        
        return $this->cart;
    }
    
    // Get cart summary
    public function getCartSummary() {
        $subtotal = $this->getSubtotal();
        $shipping = $this->calculateShipping();
        $total = $subtotal + $shipping;
        
        // Group by category
        $categoryGroups = [];
        foreach ($this->cart as $item) {
            $categoryGroups[$item['category']][] = $item;
        }
        
        // Get product statistics
        $productCounts = array_count_values(array_column($this->cart, 'name'));
        $totalItems = array_sum(array_column($this->cart, 'quantity'));
        
        return [
            'items' => $this->cart,
            'subtotal' => $subtotal,
            'shipping' => $shipping,
            'total' => $total,
            'category_groups' => $categoryGroups,
            'product_counts' => $productCounts,
            'total_items' => $totalItems,
            'item_count' => count($this->cart)
        ];
    }
    
    // Clear cart
    public function clearCart() {
        $this->cart = [];
        return true;
    }
}

// Usage
$cart = new ShoppingCartSystem();

// Add products
$cart->addToCart('PROD001', 1);
$cart->addToCart('PROD002', 2);
$cart->addToCart('PROD003', 1);

echo "Cart Summary:\n";
print_r($cart->getCartSummary());

// Apply discount
try {
    $discount = $cart->applyDiscount('BULK10');
    echo "\nApplied discount: " . $discount['value'] . "% off\n";
} catch (Exception $e) {
    echo "\nDiscount error: " . $e->getMessage() . "\n";
}

// Filter by category
echo "\nElectronics in cart:\n";
print_r($cart->filterCartByCategory('Electronics'));

// Sort by price
echo "\nCart sorted by price (descending):\n";
print_r($cart->sortCartByPrice('desc'));
```

### Q17. Data Processing Pipeline
**Complete data transformation pipeline:**

```php
class DataProcessingPipeline {
    
    public function processUserData($rawData) {
        // Step 1: Clean and validate data
        $cleanedData = $this->cleanData($rawData);
        
        // Step 2: Transform data
        $transformedData = $this->transformData($cleanedData);
        
        // Step 3: Filter active users
        $activeUsers = $this->filterActiveUsers($transformedData);
        
        // Step 4: Calculate statistics
        $statistics = $this->calculateStatistics($activeUsers);
        
        // Step 5: Generate reports
        $reports = $this->generateReports($activeUsers, $statistics);
        
        return [
            'processed_data' => $activeUsers,
            'statistics' => $statistics,
            'reports' => $reports,
            'total_processed' => count($activeUsers)
        ];
    }
    
    private function cleanData($data) {
        return array_filter($data, function($user) {
            return !empty($user['email']) && 
                   filter_var($user['email'], FILTER_VALIDATE_EMAIL) &&
                   !empty($user['name']);
        });
    }
    
    private function transformData($data) {
        return array_map(function($user) {
            return [
                'id' => $user['id'],
                'name' => trim(strtoupper($user['name'])),
                'email' => strtolower(trim($user['email'])),
                'age' => $this->calculateAge($user['birth_date']),
                'status' => $user['status'] ?? 'inactive',
                'created_at' => $user['created_at'],
                'last_login' => $user['last_login'] ?? null
            ];
        }, $data);
    }
    
    private function filterActiveUsers($data) {
        return array_filter($data, function($user) {
            return $user['status'] === 'active';
        });
    }
    
    private function calculateAge($birthDate) {
        return date_diff(date_create($birthDate), date_create('now'))->y;
    }
    
    private function calculateStatistics($users) {
        $ages = array_column($users, 'age');
        $emails = array_column($users, 'email');
        $domains = array_map(function($email) {
            return substr(strrchr($email, "@"), 1);
        }, $emails);
        
        return [
            'total_users' => count($users),
            'average_age' => array_sum($ages) / count($ages),
            'min_age' => min($ages),
            'max_age' => max($ages),
            'domain_distribution' => array_count_values($domains),
            'unique_domains' => count(array_unique($domains))
        ];
    }
    
    private function generateReports($users, $statistics) {
        // Group by age ranges
        $ageGroups = [];
        foreach ($users as $user) {
            $ageRange = $this->getAgeRange($user['age']);
            $ageGroups[$ageRange][] = $user;
        }
        
        // Sort by creation date
        $sortedUsers = $users;
        usort($sortedUsers, function($a, $b) {
            return strtotime($a['created_at']) <=> strtotime($b['created_at']);
        });
        
        return [
            'age_groups' => $ageGroups,
            'recent_users' => array_slice($sortedUsers, -10),
            'oldest_users' => array_slice($sortedUsers, 0, 10),
            'statistics' => $statistics
        ];
    }
    
    private function getAgeRange($age) {
        if ($age < 18) return 'Under 18';
        if ($age < 25) return '18-24';
        if ($age < 35) return '25-34';
        if ($age < 50) return '35-49';
        return '50+';
    }
}

// Usage
$pipeline = new DataProcessingPipeline();

$rawData = [
    ['id' => 1, 'name' => 'John Doe', 'email' => 'john@example.com', 'birth_date' => '1990-01-01', 'status' => 'active', 'created_at' => '2020-01-01'],
    ['id' => 2, 'name' => 'Jane Smith', 'email' => 'jane@test.com', 'birth_date' => '1985-05-15', 'status' => 'active', 'created_at' => '2019-06-01'],
    ['id' => 3, 'name' => 'Bob Wilson', 'email' => 'bob@company.org', 'birth_date' => '1995-12-10', 'status' => 'inactive', 'created_at' => '2021-03-15']
];

$result = $pipeline->processUserData($rawData);
print_r($result);
```

**Total PHP Array Functions: 50+ with comprehensive syntax definitions, real-world examples, and advanced use cases!**
