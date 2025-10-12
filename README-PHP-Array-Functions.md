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

This is the first part of the PHP Array Functions guide. Would you like me to continue with more sections including Array Merging, Transformation, and the Node.js array methods guide?
