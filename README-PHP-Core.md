# PHP Core Interview Questions & Answers
### For 7+ Years Experienced PHP Developers

---

## Table of Contents
1. [PHP Basics](#php-basics)
2. [Data Types & Variables](#data-types--variables)
3. [Functions](#functions)
4. [Arrays](#arrays)
5. [String Manipulation](#string-manipulation)
6. [File Handling](#file-handling)
7. [Sessions & Cookies](#sessions--cookies)
8. [Error Handling & Exceptions](#error-handling--exceptions)
9. [Security](#security)
10. [Performance & Best Practices](#performance--best-practices)

---

## PHP Basics

### Q1. What are the differences between PHP 7 and PHP 8?
**Answer:** 

**PHP 8 Features:**
- JIT (Just-In-Time) Compiler
- Named Arguments
- Union Types
- Match Expression
- Nullsafe Operator
- Constructor Property Promotion
- Attributes (Annotations)
- Mixed Type
- Static Return Type

**Real-time Example:**
```php
// Named Arguments (PHP 8)
function create_user($name, $email, $role = 'user', $active = true) {
    return compact('name', 'email', 'role', 'active');
}

// PHP 7 way
$user = create_user('John', 'john@example.com', 'admin', true);

// PHP 8 way - can skip and reorder arguments
$user = create_user(
    email: 'john@example.com',
    name: 'John',
    active: false
);

// Union Types (PHP 8)
function process_id(int|string $id): int|string {
    return is_int($id) ? $id : (int) $id;
}

// Match Expression (PHP 8)
$status = 'pending';

// Old way - switch
switch ($status) {
    case 'pending':
        $message = 'Order is pending';
        break;
    case 'processing':
        $message = 'Order is processing';
        break;
    case 'completed':
        $message = 'Order is completed';
        break;
    default:
        $message = 'Unknown status';
}

// PHP 8 match expression
$message = match($status) {
    'pending' => 'Order is pending',
    'processing' => 'Order is processing',
    'completed' => 'Order is completed',
    default => 'Unknown status'
};

// Nullsafe Operator (PHP 8)
// Old way
$country = null;
if ($user !== null) {
    if ($user->getAddress() !== null) {
        $country = $user->getAddress()->getCountry();
    }
}

// PHP 8 way
$country = $user?->getAddress()?->getCountry();

// Constructor Property Promotion (PHP 8)
// Old way
class UserOld {
    private string $name;
    private string $email;
    
    public function __construct(string $name, string $email) {
        $this->name = $name;
        $this->email = $email;
    }
}

// PHP 8 way - much cleaner
class User {
    public function __construct(
        private string $name,
        private string $email,
        private string $role = 'user'
    ) {}
    
    public function getName(): string {
        return $this->name;
    }
}

// Attributes (PHP 8)
#[Route('/api/users', methods: ['GET', 'POST'])]
#[Authorize('admin')]
class UserController {
    #[Route('/api/users/{id}', methods: ['GET'])]
    public function show(int $id) {
        return User::find($id);
    }
}
```

### Q2. What is the difference between include, require, include_once, and require_once?
**Answer:**
- `include` - Includes file, warning on failure, script continues
- `require` - Includes file, fatal error on failure, script stops
- `include_once` - Includes file only once, warning on failure
- `require_once` - Includes file only once, fatal error on failure

**Real-time Example:**
```php
// config.php
<?php
define('DB_HOST', 'localhost');
define('DB_USER', 'root');
define('DB_PASS', 'password');
define('DB_NAME', 'myapp');

// functions.php
<?php
function sanitize_input($data) {
    return htmlspecialchars(strip_tags(trim($data)));
}

function redirect($url) {
    header("Location: $url");
    exit();
}

// Application structure example
// index.php
<?php
// Must have configuration - use require_once
require_once 'config.php';

// Must have database connection
require_once 'database.php';

// Must have functions - include only once
require_once 'functions.php';

// Optional analytics - won't break app if missing
include_once 'analytics.php';

// Optional debugging tools - only in development
if (ENVIRONMENT === 'development') {
    include 'debug_toolbar.php';
}

// Load page
require 'pages/home.php';

// Real-world autoloader example
spl_autoload_register(function ($class) {
    $file = str_replace('\\', '/', $class) . '.php';
    
    if (file_exists($file)) {
        require_once $file;
    }
});
```

### Q3. What are PHP superglobals?
**Answer:** Superglobals are built-in variables that are always available in all scopes.

**Superglobals:**
- `$GLOBALS` - References all variables in global scope
- `$_SERVER` - Server and execution environment information
- `$_GET` - HTTP GET variables
- `$_POST` - HTTP POST variables
- `$_FILES` - HTTP file upload variables
- `$_COOKIE` - HTTP cookies
- `$_SESSION` - Session variables
- `$_REQUEST` - HTTP request variables (GET, POST, COOKIE)
- `$_ENV` - Environment variables

**Real-time Example:**
```php
// Request handling class
class Request {
    
    // Get client IP address
    public static function getIpAddress() {
        if (!empty($_SERVER['HTTP_CLIENT_IP'])) {
            return $_SERVER['HTTP_CLIENT_IP'];
        } elseif (!empty($_SERVER['HTTP_X_FORWARDED_FOR'])) {
            return $_SERVER['HTTP_X_FORWARDED_FOR'];
        } else {
            return $_SERVER['REMOTE_ADDR'];
        }
    }
    
    // Get request method
    public static function getMethod() {
        return $_SERVER['REQUEST_METHOD'];
    }
    
    // Check if HTTPS
    public static function isSecure() {
        return (!empty($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off')
            || $_SERVER['SERVER_PORT'] == 443;
    }
    
    // Get current URL
    public static function getCurrentUrl() {
        $protocol = self::isSecure() ? 'https' : 'http';
        return $protocol . '://' . $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI'];
    }
    
    // Get user agent
    public static function getUserAgent() {
        return $_SERVER['HTTP_USER_AGENT'] ?? '';
    }
    
    // Get input from GET/POST
    public static function input($key, $default = null, $method = 'post') {
        $source = ($method === 'get') ? $_GET : $_POST;
        
        if (isset($source[$key])) {
            return self::sanitize($source[$key]);
        }
        
        return $default;
    }
    
    // Sanitize input
    private static function sanitize($data) {
        if (is_array($data)) {
            return array_map([self::class, 'sanitize'], $data);
        }
        
        return htmlspecialchars(strip_tags(trim($data)), ENT_QUOTES, 'UTF-8');
    }
    
    // Get all POST data
    public static function all() {
        return array_map([self::class, 'sanitize'], $_POST);
    }
    
    // Check if has file
    public static function hasFile($key) {
        return isset($_FILES[$key]) && $_FILES[$key]['error'] !== UPLOAD_ERR_NO_FILE;
    }
    
    // Get uploaded file
    public static function file($key) {
        if (self::hasFile($key)) {
            return $_FILES[$key];
        }
        return null;
    }
}

// Usage examples
$ip = Request::getIpAddress();
$method = Request::getMethod();
$isHttps = Request::isSecure();
$currentUrl = Request::getCurrentUrl();

if ($method === 'POST') {
    $username = Request::input('username');
    $email = Request::input('email');
    $allData = Request::all();
    
    if (Request::hasFile('avatar')) {
        $avatar = Request::file('avatar');
        // Process file upload
    }
}

// Session management example
class Session {
    
    public static function start() {
        if (session_status() === PHP_SESSION_NONE) {
            session_start();
        }
    }
    
    public static function set($key, $value) {
        self::start();
        $_SESSION[$key] = $value;
    }
    
    public static function get($key, $default = null) {
        self::start();
        return $_SESSION[$key] ?? $default;
    }
    
    public static function has($key) {
        self::start();
        return isset($_SESSION[$key]);
    }
    
    public static function remove($key) {
        self::start();
        unset($_SESSION[$key]);
    }
    
    public static function destroy() {
        self::start();
        session_destroy();
    }
    
    public static function flash($key, $value) {
        self::set($key, $value);
        self::set($key . '_flash', true);
    }
    
    public static function getFlash($key, $default = null) {
        $value = self::get($key, $default);
        
        if (self::get($key . '_flash')) {
            self::remove($key);
            self::remove($key . '_flash');
        }
        
        return $value;
    }
}

// Usage
Session::set('user_id', 123);
Session::flash('success', 'Login successful!');

$userId = Session::get('user_id');
$message = Session::getFlash('success');
```

---

## Data Types & Variables

### Q4. What are the data types in PHP?
**Answer:** 

**Scalar Types:**
- Boolean (bool)
- Integer (int)
- Float (float, double)
- String (string)

**Compound Types:**
- Array (array)
- Object (object)
- Callable (callable)
- Iterable (iterable)

**Special Types:**
- Resource (resource)
- NULL (null)

**Real-time Example:**
```php
// Type juggling and strict types
declare(strict_types=1);

class TypeExample {
    
    // Scalar types
    public function processPayment(float $amount, string $currency, bool $isRecurring): int {
        $transactionId = random_int(100000, 999999);
        
        // Type casting
        $amountCents = (int) ($amount * 100);
        $currencyCode = (string) $currency;
        $recurring = (bool) $isRecurring;
        
        return $transactionId;
    }
    
    // Array type
    public function processOrders(array $orders): array {
        $processed = [];
        
        foreach ($orders as $order) {
            $processed[] = $this->processOrder($order);
        }
        
        return $processed;
    }
    
    // Callable type
    public function filterData(array $data, callable $callback): array {
        return array_filter($data, $callback);
    }
    
    // Object type
    public function saveUser(object $user): bool {
        // Save user to database
        return true;
    }
    
    // Nullable types
    public function findUser(int $id): ?object {
        // May return User object or null
        return $id > 0 ? new stdClass() : null;
    }
    
    // Union types (PHP 8+)
    public function processId(int|string $id): int {
        return is_int($id) ? $id : (int) $id;
    }
    
    // Mixed type (PHP 8+)
    public function processValue(mixed $value): mixed {
        if (is_array($value)) {
            return count($value);
        } elseif (is_string($value)) {
            return strlen($value);
        } elseif (is_numeric($value)) {
            return $value * 2;
        }
        
        return $value;
    }
    
    // Type checking
    public function checkTypes($var) {
        $info = [
            'value' => $var,
            'type' => gettype($var),
            'is_bool' => is_bool($var),
            'is_int' => is_int($var),
            'is_float' => is_float($var),
            'is_string' => is_string($var),
            'is_array' => is_array($var),
            'is_object' => is_object($var),
            'is_null' => is_null($var),
            'is_numeric' => is_numeric($var),
            'is_scalar' => is_scalar($var),
            'is_callable' => is_callable($var),
        ];
        
        return $info;
    }
}

// Usage
$example = new TypeExample();

// Strict types enforce type checking
$transactionId = $example->processPayment(99.99, 'USD', true);

// Type casting examples
$strNumber = "123";
$intNumber = (int) $strNumber;  // 123
$floatNumber = (float) $strNumber;  // 123.0

$intValue = 100;
$stringValue = (string) $intValue;  // "100"

$truthyValue = 1;
$boolValue = (bool) $truthyValue;  // true

// Array casting
$object = new stdClass();
$object->name = "John";
$object->age = 30;
$array = (array) $object;  // ['name' => 'John', 'age' => 30]

// Variable type checking
$value = "123";
if (is_numeric($value)) {
    $numericValue = (int) $value;
}

// Callable examples
$result = $example->filterData(
    [1, 2, 3, 4, 5],
    function($n) { return $n % 2 === 0; }
);
```

### Q5. What is type hinting and type declaration?
**Answer:** Type hinting (declaration) specifies the expected data type of arguments and return values.

**Real-time Example:**
```php
// Type hinting examples
class ProductService {
    
    // Constructor with type hinting
    public function __construct(
        private PDO $database,
        private CacheInterface $cache,
        private LoggerInterface $logger
    ) {}
    
    // Parameter and return type hinting
    public function createProduct(
        string $name,
        float $price,
        int $quantity,
        ?string $description = null
    ): int {
        $data = [
            'name' => $name,
            'price' => $price,
            'quantity' => $quantity,
            'description' => $description,
            'created_at' => date('Y-m-d H:i:s')
        ];
        
        $stmt = $this->database->prepare(
            "INSERT INTO products (name, price, quantity, description, created_at) 
             VALUES (:name, :price, :quantity, :description, :created_at)"
        );
        
        $stmt->execute($data);
        
        return (int) $this->database->lastInsertId();
    }
    
    // Array type hint
    public function bulkCreate(array $products): array {
        $createdIds = [];
        
        foreach ($products as $product) {
            $createdIds[] = $this->createProduct(
                $product['name'],
                $product['price'],
                $product['quantity'],
                $product['description'] ?? null
            );
        }
        
        return $createdIds;
    }
    
    // Object type hint with nullable return
    public function findProduct(int $id): ?Product {
        $cacheKey = "product_$id";
        
        // Check cache
        if ($this->cache->has($cacheKey)) {
            return $this->cache->get($cacheKey);
        }
        
        // Query database
        $stmt = $this->database->prepare("SELECT * FROM products WHERE id = ?");
        $stmt->execute([$id]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if (!$data) {
            return null;
        }
        
        $product = new Product($data);
        
        // Store in cache
        $this->cache->set($cacheKey, $product, 3600);
        
        return $product;
    }
    
    // Callable type hint
    public function filterProducts(array $products, callable $filter): array {
        return array_filter($products, $filter);
    }
    
    // Void return type
    public function logActivity(string $action, array $data): void {
        $this->logger->info("Product $action", $data);
    }
    
    // Self return type (for fluent interface)
    public function setDatabase(PDO $database): self {
        $this->database = $database;
        return $this;
    }
    
    // Static return type
    public static function getInstance(): static {
        return new static();
    }
    
    // Union types (PHP 8+)
    public function processPrice(int|float|string $price): float {
        return (float) $price;
    }
    
    // Intersection types (PHP 8.1+)
    public function process(Countable&Iterator $collection): int {
        return count($collection);
    }
}

// Product class
class Product {
    public function __construct(
        private array $data
    ) {}
    
    public function getId(): int {
        return (int) $this->data['id'];
    }
    
    public function getName(): string {
        return (string) $this->data['name'];
    }
    
    public function getPrice(): float {
        return (float) $this->data['price'];
    }
    
    public function getQuantity(): int {
        return (int) $this->data['quantity'];
    }
}

// Interface examples
interface CacheInterface {
    public function get(string $key): mixed;
    public function set(string $key, mixed $value, int $ttl): bool;
    public function has(string $key): bool;
    public function delete(string $key): bool;
}

interface LoggerInterface {
    public function info(string $message, array $context = []): void;
    public function error(string $message, array $context = []): void;
}
```

---

## Functions

### Q6. What are anonymous functions and closures?
**Answer:** Anonymous functions (closures) are functions without a specified name. They can be stored in variables and passed as arguments.

**Real-time Example:**
```php
// Basic anonymous function
$greet = function($name) {
    return "Hello, $name!";
};

echo $greet('John');  // Hello, John!

// Closure with use keyword
$multiplier = 10;

$multiply = function($number) use ($multiplier) {
    return $number * $multiplier;
};

echo $multiply(5);  // 50

// Closure with reference
$counter = 0;

$increment = function() use (&$counter) {
    $counter++;
    return $counter;
};

echo $increment();  // 1
echo $increment();  // 2
echo $increment();  // 3

// Real-world examples

// 1. Array filtering
class ProductFilter {
    
    public function filterByPrice(array $products, float $minPrice, float $maxPrice): array {
        return array_filter($products, function($product) use ($minPrice, $maxPrice) {
            return $product['price'] >= $minPrice && $product['price'] <= $maxPrice;
        });
    }
    
    public function filterByCategory(array $products, string $category): array {
        return array_filter($products, function($product) use ($category) {
            return $product['category'] === $category;
        });
    }
    
    public function sortByPrice(array $products, string $direction = 'asc'): array {
        usort($products, function($a, $b) use ($direction) {
            if ($direction === 'asc') {
                return $a['price'] <=> $b['price'];
            }
            return $b['price'] <=> $a['price'];
        });
        
        return $products;
    }
}

// 2. Route handling with closures
class Router {
    private array $routes = [];
    
    public function get(string $path, callable $handler): void {
        $this->routes['GET'][$path] = $handler;
    }
    
    public function post(string $path, callable $handler): void {
        $this->routes['POST'][$path] = $handler;
    }
    
    public function dispatch(string $method, string $path): mixed {
        if (isset($this->routes[$method][$path])) {
            return call_user_func($this->routes[$method][$path]);
        }
        
        return null;
    }
}

// Usage
$router = new Router();

$router->get('/users', function() {
    return ['users' => User::all()];
});

$router->post('/users', function() {
    $data = json_decode(file_get_contents('php://input'), true);
    return User::create($data);
});

// 3. Middleware pattern
class Middleware {
    
    public static function auth(callable $next): callable {
        return function() use ($next) {
            if (!isset($_SESSION['user_id'])) {
                http_response_code(401);
                return ['error' => 'Unauthorized'];
            }
            
            return $next();
        };
    }
    
    public static function admin(callable $next): callable {
        return function() use ($next) {
            if ($_SESSION['role'] !== 'admin') {
                http_response_code(403);
                return ['error' => 'Forbidden'];
            }
            
            return $next();
        };
    }
    
    public static function rateLimit(int $limit, callable $next): callable {
        return function() use ($limit, $next) {
            $key = 'rate_limit_' . $_SERVER['REMOTE_ADDR'];
            
            if (!isset($_SESSION[$key])) {
                $_SESSION[$key] = ['count' => 0, 'reset' => time() + 60];
            }
            
            if (time() > $_SESSION[$key]['reset']) {
                $_SESSION[$key] = ['count' => 0, 'reset' => time() + 60];
            }
            
            $_SESSION[$key]['count']++;
            
            if ($_SESSION[$key]['count'] > $limit) {
                http_response_code(429);
                return ['error' => 'Too many requests'];
            }
            
            return $next();
        };
    }
}

// 4. Event handling
class EventEmitter {
    private array $listeners = [];
    
    public function on(string $event, callable $listener): void {
        $this->listeners[$event][] = $listener;
    }
    
    public function emit(string $event, ...$args): void {
        if (isset($this->listeners[$event])) {
            foreach ($this->listeners[$event] as $listener) {
                call_user_func_array($listener, $args);
            }
        }
    }
}

// Usage
$emitter = new EventEmitter();

$emitter->on('user.created', function($user) {
    // Send welcome email
    mail($user['email'], 'Welcome!', 'Thanks for signing up!');
});

$emitter->on('user.created', function($user) {
    // Log activity
    error_log("New user created: {$user['email']}");
});

$emitter->emit('user.created', ['email' => 'john@example.com']);

// 5. Array transformation
class ArrayHelper {
    
    public static function map(array $array, callable $callback): array {
        return array_map($callback, $array);
    }
    
    public static function reduce(array $array, callable $callback, $initial = null): mixed {
        return array_reduce($array, $callback, $initial);
    }
    
    public static function pipe(...$functions): callable {
        return function($value) use ($functions) {
            foreach ($functions as $function) {
                $value = $function($value);
            }
            return $value;
        };
    }
}

// Usage
$numbers = [1, 2, 3, 4, 5];

// Map
$doubled = ArrayHelper::map($numbers, fn($n) => $n * 2);
// [2, 4, 6, 8, 10]

// Reduce
$sum = ArrayHelper::reduce($numbers, fn($carry, $n) => $carry + $n, 0);
// 15

// Pipe
$process = ArrayHelper::pipe(
    fn($n) => $n * 2,
    fn($n) => $n + 10,
    fn($n) => $n / 2
);

echo $process(5);  // (5 * 2 + 10) / 2 = 10
```

---

## Arrays

### Q7. What are the different types of arrays in PHP?
**Answer:** PHP supports indexed arrays, associative arrays, and multidimensional arrays.

**Real-time Example:**
```php
// Indexed arrays
$fruits = ['apple', 'banana', 'orange'];
$numbers = array(1, 2, 3, 4, 5);

// Associative arrays
$person = [
    'name' => 'John Doe',
    'age' => 30,
    'email' => 'john@example.com'
];

// Multidimensional arrays
$users = [
    [
        'id' => 1,
        'name' => 'John Doe',
        'orders' => [
            ['id' => 101, 'amount' => 99.99],
            ['id' => 102, 'amount' => 149.99]
        ]
    ],
    [
        'id' => 2,
        'name' => 'Jane Smith',
        'orders' => [
            ['id' => 201, 'amount' => 79.99]
        ]
    ]
];

// Array operations
class ArrayOperations {
    
    // Array filtering
    public function filterActiveUsers(array $users): array {
        return array_filter($users, function($user) {
            return $user['status'] === 'active';
        });
    }
    
    // Array mapping
    public function getUserNames(array $users): array {
        return array_map(function($user) {
            return $user['name'];
        }, $users);
    }
    
    // Array reduction
    public function calculateTotalRevenue(array $orders): float {
        return array_reduce($orders, function($carry, $order) {
            return $carry + $order['amount'];
        }, 0);
    }
    
    // Array sorting
    public function sortUsersByAge(array $users, string $direction = 'asc'): array {
        usort($users, function($a, $b) use ($direction) {
            if ($direction === 'asc') {
                return $a['age'] <=> $b['age'];
            }
            return $b['age'] <=> $a['age'];
        });
        
        return $users;
    }
    
    // Array searching
    public function findUserById(array $users, int $id): ?array {
        foreach ($users as $user) {
            if ($user['id'] === $id) {
                return $user;
            }
        }
        return null;
    }
    
    // Array merging
    public function mergeUserData(array $users, array $profiles): array {
        $merged = [];
        
        foreach ($users as $user) {
            $profile = array_filter($profiles, function($p) use ($user) {
                return $p['user_id'] === $user['id'];
            });
            
            $merged[] = array_merge($user, $profile[0] ?? []);
        }
        
        return $merged;
    }
    
    // Array grouping
    public function groupUsersByRole(array $users): array {
        $grouped = [];
        
        foreach ($users as $user) {
            $grouped[$user['role']][] = $user;
        }
        
        return $grouped;
    }
    
    // Array chunking
    public function chunkUsers(array $users, int $size): array {
        return array_chunk($users, $size);
    }
    
    // Array flattening
    public function flattenOrders(array $users): array {
        $orders = [];
        
        foreach ($users as $user) {
            $orders = array_merge($orders, $user['orders']);
        }
        
        return $orders;
    }
    
    // Array unique values
    public function getUniqueRoles(array $users): array {
        $roles = array_column($users, 'role');
        return array_unique($roles);
    }
    
    // Array intersection
    public function getCommonRoles(array $group1, array $group2): array {
        $roles1 = array_column($group1, 'role');
        $roles2 = array_column($group2, 'role');
        
        return array_intersect($roles1, $roles2);
    }
    
    // Array difference
    public function getUniqueRoles(array $group1, array $group2): array {
        $roles1 = array_column($group1, 'role');
        $roles2 = array_column($group2, 'role');
        
        return array_diff($roles1, $roles2);
    }
}

// Usage examples
$operations = new ArrayOperations();

// Filter active users
$activeUsers = $operations->filterActiveUsers($users);

// Get user names
$names = $operations->getUserNames($users);

// Calculate total revenue
$totalRevenue = $operations->calculateTotalRevenue($orders);

// Sort users by age
$sortedUsers = $operations->sortUsersByAge($users, 'desc');

// Find specific user
$user = $operations->findUserById($users, 1);

// Group users by role
$groupedUsers = $operations->groupUsersByRole($users);

// Advanced array operations
class AdvancedArrayOperations {
    
    // Array walk with callback
    public function updateUserStatus(array &$users, string $status): void {
        array_walk($users, function(&$user) use ($status) {
            $user['status'] = $status;
        });
    }
    
    // Array walk recursive
    public function sanitizeArray(array &$data): void {
        array_walk_recursive($data, function(&$value) {
            if (is_string($value)) {
                $value = htmlspecialchars(trim($value), ENT_QUOTES, 'UTF-8');
            }
        });
    }
    
    // Array key exists
    public function validateUserData(array $user): bool {
        $requiredKeys = ['name', 'email', 'age'];
        
        foreach ($requiredKeys as $key) {
            if (!array_key_exists($key, $user)) {
                return false;
            }
        }
        
        return true;
    }
    
    // Array search
    public function searchUserByEmail(array $users, string $email): int|false {
        return array_search($email, array_column($users, 'email'));
    }
    
    // Array slice
    public function getUsersPage(array $users, int $page, int $perPage): array {
        $offset = ($page - 1) * $perPage;
        return array_slice($users, $offset, $perPage);
    }
    
    // Array splice
    public function removeUser(array &$users, int $userId): array {
        $index = array_search($userId, array_column($users, 'id'));
        
        if ($index !== false) {
            return array_splice($users, $index, 1);
        }
        
        return [];
    }
    
    // Array combine
    public function createUserLookup(array $users): array {
        $ids = array_column($users, 'id');
        $names = array_column($users, 'name');
        
        return array_combine($ids, $names);
    }
    
    // Array flip
    public function reverseUserLookup(array $lookup): array {
        return array_flip($lookup);
    }
    
    // Array pad
    public function padUserArray(array $users, int $size): array {
        return array_pad($users, $size, ['id' => null, 'name' => 'Empty Slot']);
    }
    
    // Array sum
    public function calculateTotalOrders(array $users): int {
        return array_sum(array_map(function($user) {
            return count($user['orders']);
        }, $users));
    }
    
    // Array product
    public function calculateTotalAmount(array $orders): float {
        return array_product(array_column($orders, 'amount'));
    }
    
    // Array random
    public function getRandomUsers(array $users, int $count): array {
        return array_rand($users, min($count, count($users)));
    }
    
    // Array shuffle
    public function shuffleUsers(array $users): array {
        $shuffled = $users;
        shuffle($shuffled);
        return $shuffled;
    }
    
    // Array count values
    public function countUserRoles(array $users): array {
        return array_count_values(array_column($users, 'role'));
    }
    
    // Array keys
    public function getUserKeys(array $user): array {
        return array_keys($user);
    }
    
    // Array values
    public function getUserValues(array $user): array {
        return array_values($user);
    }
}

// SPL ArrayObject usage
class UserCollection extends ArrayObject {
    
    public function __construct(array $users = []) {
        parent::__construct($users);
    }
    
    public function addUser(array $user): void {
        $this->append($user);
    }
    
    public function removeUser(int $id): bool {
        foreach ($this as $key => $user) {
            if ($user['id'] === $id) {
                unset($this[$key]);
                return true;
            }
        }
        return false;
    }
    
    public function findUser(int $id): ?array {
        foreach ($this as $user) {
            if ($user['id'] === $id) {
                return $user;
            }
        }
        return null;
    }
    
    public function sortBy(string $field, string $direction = 'asc'): void {
        $this->uasort(function($a, $b) use ($field, $direction) {
            if ($direction === 'asc') {
                return $a[$field] <=> $b[$field];
            }
            return $b[$field] <=> $a[$field];
        });
    }
    
    public function filter(callable $callback): UserCollection {
        $filtered = new UserCollection();
        
        foreach ($this as $user) {
            if ($callback($user)) {
                $filtered->addUser($user);
            }
        }
        
        return $filtered;
    }
    
    public function map(callable $callback): UserCollection {
        $mapped = new UserCollection();
        
        foreach ($this as $user) {
            $mapped->addUser($callback($user));
        }
        
        return $mapped;
    }
}

// Usage
$userCollection = new UserCollection($users);
$userCollection->addUser(['id' => 3, 'name' => 'Bob Johnson', 'age' => 25]);
$userCollection->sortBy('age', 'desc');
$activeUsers = $userCollection->filter(fn($user) => $user['age'] > 25);
```

### Q8. What is the difference between array_merge and array_merge_recursive?
**Answer:** 
- `array_merge` - Merges arrays, overwrites duplicate keys
- `array_merge_recursive` - Merges arrays recursively, creates arrays for duplicate keys

**Real-time Example:**
```php
$array1 = ['a' => 1, 'b' => 2, 'c' => 3];
$array2 = ['b' => 4, 'c' => 5, 'd' => 6];

// array_merge - overwrites duplicate keys
$merged = array_merge($array1, $array2);
// Result: ['a' => 1, 'b' => 4, 'c' => 5, 'd' => 6]

// array_merge_recursive - creates arrays for duplicate keys
$recursive = array_merge_recursive($array1, $array2);
// Result: ['a' => 1, 'b' => [2, 4], 'c' => [3, 5], 'd' => 6]

// Real-world example
$userSettings = [
    'theme' => 'dark',
    'notifications' => [
        'email' => true,
        'sms' => false
    ]
];

$defaultSettings = [
    'theme' => 'light',
    'notifications' => [
        'email' => false,
        'push' => true
    ],
    'language' => 'en'
];

// Merge with overwrite
$finalSettings = array_merge($defaultSettings, $userSettings);
// Notifications will be completely overwritten

// Merge recursively
$finalSettingsRecursive = array_merge_recursive($defaultSettings, $userSettings);
// Notifications will be merged, keeping both email and push settings
```

---

## String Manipulation

### Q9. What are the most important string functions in PHP?
**Answer:** PHP provides many string functions for manipulation, validation, and formatting.

**Real-time Example:**
```php
class StringOperations {
    
    // Basic string operations
    public function basicOperations(string $text): array {
        return [
            'length' => strlen($text),
            'word_count' => str_word_count($text),
            'uppercase' => strtoupper($text),
            'lowercase' => strtolower($text),
            'title_case' => ucwords($text),
            'first_letter_upper' => ucfirst($text),
            'trimmed' => trim($text),
            'reversed' => strrev($text)
        ];
    }
    
    // String searching
    public function searchOperations(string $text, string $search): array {
        return [
            'contains' => strpos($text, $search) !== false,
            'position' => strpos($text, $search),
            'last_position' => strrpos($text, $search),
            'starts_with' => str_starts_with($text, $search),
            'ends_with' => str_ends_with($text, $search),
            'count' => substr_count($text, $search)
        ];
    }
    
    // String replacement
    public function replaceOperations(string $text, string $search, string $replace): array {
        return [
            'simple_replace' => str_replace($search, $replace, $text),
            'case_insensitive_replace' => str_ireplace($search, $replace, $text),
            'first_occurrence' => str_replace($search, $replace, $text, 1),
            'substr_replace' => substr_replace($text, $replace, 0, strlen($search))
        ];
    }
    
    // String extraction
    public function extractOperations(string $text, int $start, int $length): array {
        return [
            'substring' => substr($text, $start, $length),
            'substring_from_end' => substr($text, -$length),
            'substring_to_end' => substr($text, $start),
            'words' => explode(' ', $text),
            'lines' => explode("\n", $text),
            'characters' => str_split($text)
        ];
    }
    
    // String formatting
    public function formatOperations(string $text): array {
        return [
            'padded_left' => str_pad($text, 20, ' ', STR_PAD_LEFT),
            'padded_right' => str_pad($text, 20, ' ', STR_PAD_RIGHT),
            'padded_both' => str_pad($text, 20, ' ', STR_PAD_BOTH),
            'wrapped' => wordwrap($text, 20, "\n"),
            'formatted_number' => number_format(1234567.89, 2, '.', ','),
            'formatted_currency' => '$' . number_format(1234.56, 2)
        ];
    }
    
    // String validation
    public function validateOperations(string $text): array {
        return [
            'is_numeric' => is_numeric($text),
            'is_email' => filter_var($text, FILTER_VALIDATE_EMAIL) !== false,
            'is_url' => filter_var($text, FILTER_VALIDATE_URL) !== false,
            'is_ip' => filter_var($text, FILTER_VALIDATE_IP) !== false,
            'is_alpha' => ctype_alpha($text),
            'is_digit' => ctype_digit($text),
            'is_alnum' => ctype_alnum($text),
            'is_printable' => ctype_print($text)
        ];
    }
    
    // String sanitization
    public function sanitizeOperations(string $text): array {
        return [
            'html_entities' => htmlentities($text, ENT_QUOTES, 'UTF-8'),
            'html_special_chars' => htmlspecialchars($text, ENT_QUOTES, 'UTF-8'),
            'strip_tags' => strip_tags($text),
            'trim_whitespace' => trim($text),
            'remove_slashes' => stripslashes($text),
            'add_slashes' => addslashes($text),
            'url_encode' => urlencode($text),
            'url_decode' => urldecode($text),
            'base64_encode' => base64_encode($text),
            'base64_decode' => base64_decode($text)
        ];
    }
    
    // Advanced string operations
    public function advancedOperations(string $text): array {
        return [
            'soundex' => soundex($text),
            'metaphone' => metaphone($text),
            'levenshtein_distance' => levenshtein($text, 'example'),
            'similarity' => similar_text($text, 'example'),
            'md5_hash' => md5($text),
            'sha1_hash' => sha1($text),
            'sha256_hash' => hash('sha256', $text),
            'crc32_checksum' => crc32($text)
        ];
    }
    
    // String parsing
    public function parseOperations(string $text): array {
        return [
            'parse_url' => parse_url($text),
            'parse_str' => parse_str($text, $result) ? $result : null,
            'parse_csv' => str_getcsv($text),
            'parse_json' => json_decode($text, true),
            'parse_xml' => simplexml_load_string($text)
        ];
    }
    
    // String comparison
    public function compareOperations(string $text1, string $text2): array {
        return [
            'strcmp' => strcmp($text1, $text2),
            'strcasecmp' => strcasecmp($text1, $text2),
            'strncmp' => strncmp($text1, $text2, 5),
            'strncasecmp' => strncasecmp($text1, $text2, 5),
            'strcoll' => strcoll($text1, $text2),
            'similar_text' => similar_text($text1, $text2, $percent),
            'similarity_percent' => $percent ?? 0
        ];
    }
}

// Real-world string processing examples
class StringProcessor {
    
    // Email validation and formatting
    public function processEmail(string $email): array {
        $email = trim(strtolower($email));
        
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException('Invalid email format');
        }
        
        return [
            'original' => $email,
            'local_part' => substr($email, 0, strpos($email, '@')),
            'domain' => substr($email, strpos($email, '@') + 1),
            'is_gmail' => str_ends_with($email, '@gmail.com'),
            'is_company' => !in_array(substr($email, strpos($email, '@') + 1), ['gmail.com', 'yahoo.com', 'hotmail.com'])
        ];
    }
    
    // Phone number formatting
    public function formatPhoneNumber(string $phone): string {
        // Remove all non-digits
        $phone = preg_replace('/\D/', '', $phone);
        
        // Format as (XXX) XXX-XXXX
        if (strlen($phone) === 10) {
            return '(' . substr($phone, 0, 3) . ') ' . substr($phone, 3, 3) . '-' . substr($phone, 6);
        }
        
        return $phone;
    }
    
    // Name formatting
    public function formatName(string $name): string {
        $name = trim($name);
        $name = preg_replace('/\s+/', ' ', $name); // Remove extra spaces
        $name = ucwords(strtolower($name)); // Title case
        
        return $name;
    }
    
    // Address formatting
    public function formatAddress(string $address): array {
        $address = trim($address);
        $parts = explode(',', $address);
        
        return [
            'street' => trim($parts[0] ?? ''),
            'city' => trim($parts[1] ?? ''),
            'state' => trim($parts[2] ?? ''),
            'zip' => trim($parts[3] ?? ''),
            'formatted' => implode(', ', array_filter($parts))
        ];
    }
    
    // Text analysis
    public function analyzeText(string $text): array {
        $text = trim($text);
        
        return [
            'character_count' => strlen($text),
            'word_count' => str_word_count($text),
            'sentence_count' => substr_count($text, '.') + substr_count($text, '!') + substr_count($text, '?'),
            'paragraph_count' => substr_count($text, "\n\n") + 1,
            'average_word_length' => $this->getAverageWordLength($text),
            'most_common_words' => $this->getMostCommonWords($text),
            'reading_time' => $this->getReadingTime($text)
        ];
    }
    
    private function getAverageWordLength(string $text): float {
        $words = str_word_count($text, 1);
        if (empty($words)) return 0;
        
        $totalLength = array_sum(array_map('strlen', $words));
        return round($totalLength / count($words), 2);
    }
    
    private function getMostCommonWords(string $text, int $limit = 5): array {
        $words = str_word_count(strtolower($text), 1);
        $wordCounts = array_count_values($words);
        arsort($wordCounts);
        
        return array_slice($wordCounts, 0, $limit, true);
    }
    
    private function getReadingTime(string $text): int {
        $wordCount = str_word_count($text);
        return ceil($wordCount / 200); // Average reading speed: 200 words per minute
    }
    
    // String encryption/decryption
    public function encryptString(string $text, string $key): string {
        $iv = random_bytes(16);
        $encrypted = openssl_encrypt($text, 'AES-256-CBC', $key, 0, $iv);
        return base64_encode($iv . $encrypted);
    }
    
    public function decryptString(string $encrypted, string $key): string {
        $data = base64_decode($encrypted);
        $iv = substr($data, 0, 16);
        $encrypted = substr($data, 16);
        return openssl_decrypt($encrypted, 'AES-256-CBC', $key, 0, $iv);
    }
    
    // String compression
    public function compressString(string $text): string {
        return base64_encode(gzcompress($text));
    }
    
    public function decompressString(string $compressed): string {
        return gzuncompress(base64_decode($compressed));
    }
}

// Usage examples
$stringOps = new StringOperations();
$processor = new StringProcessor();

// Basic operations
$text = "Hello World! This is a test string.";
$basicOps = $stringOps->basicOperations($text);

// Email processing
$email = "  JOHN.DOE@EXAMPLE.COM  ";
$emailInfo = $processor->processEmail($email);

// Phone number formatting
$phone = "1234567890";
$formattedPhone = $processor->formatPhoneNumber($phone);

// Name formatting
$name = "  john   doe  ";
$formattedName = $processor->formatName($name);

// Text analysis
$article = "This is a sample article. It contains multiple sentences! How many words are there?";
$analysis = $processor->analyzeText($article);
```

### Q10. How do you handle multibyte strings in PHP?
**Answer:** Use the `mb_*` functions for proper multibyte string handling.

**Real-time Example:**
```php
class MultibyteStringHandler {
    
    public function handleMultibyteStrings(string $text): array {
        return [
            'length' => mb_strlen($text, 'UTF-8'),
            'substring' => mb_substr($text, 0, 10, 'UTF-8'),
            'uppercase' => mb_strtoupper($text, 'UTF-8'),
            'lowercase' => mb_strtolower($text, 'UTF-8'),
            'title_case' => mb_convert_case($text, MB_CASE_TITLE, 'UTF-8'),
            'position' => mb_strpos($text, 'test', 0, 'UTF-8'),
            'last_position' => mb_strrpos($text, 'test', 0, 'UTF-8'),
            'width' => mb_strwidth($text, 'UTF-8'),
            'encoding' => mb_detect_encoding($text),
            'converted' => mb_convert_encoding($text, 'UTF-8', 'auto')
        ];
    }
    
    public function validateMultibyteString(string $text): bool {
        return mb_check_encoding($text, 'UTF-8');
    }
    
    public function truncateMultibyteString(string $text, int $length, string $suffix = '...'): string {
        if (mb_strlen($text, 'UTF-8') <= $length) {
            return $text;
        }
        
        return mb_substr($text, 0, $length - mb_strlen($suffix, 'UTF-8'), 'UTF-8') . $suffix;
    }
}

// Usage
$handler = new MultibyteStringHandler();
$unicodeText = "Hello 世界! 🌍";
$result = $handler->handleMultibyteStrings($unicodeText);
```

---

## File Handling

### Q11. How do you handle file operations in PHP?
**Answer:** PHP provides various functions for file operations including reading, writing, and managing files.

**Real-time Example:**
```php
class FileHandler {
    
    // File reading operations
    public function readFile(string $filepath): array {
        if (!file_exists($filepath)) {
            throw new FileNotFoundException("File not found: $filepath");
        }
        
        return [
            'content' => file_get_contents($filepath),
            'lines' => file($filepath, FILE_IGNORE_NEW_LINES),
            'size' => filesize($filepath),
            'modified' => filemtime($filepath),
            'permissions' => fileperms($filepath),
            'type' => filetype($filepath),
            'is_readable' => is_readable($filepath),
            'is_writable' => is_writable($filepath),
            'is_executable' => is_executable($filepath)
        ];
    }
    
    // File writing operations
    public function writeFile(string $filepath, string $content, int $flags = 0): bool {
        $directory = dirname($filepath);
        
        if (!is_dir($directory)) {
            mkdir($directory, 0755, true);
        }
        
        return file_put_contents($filepath, $content, $flags) !== false;
    }
    
    // Append to file
    public function appendToFile(string $filepath, string $content): bool {
        return file_put_contents($filepath, $content, FILE_APPEND | LOCK_EX) !== false;
    }
    
    // File copying and moving
    public function copyFile(string $source, string $destination): bool {
        $directory = dirname($destination);
        
        if (!is_dir($directory)) {
            mkdir($directory, 0755, true);
        }
        
        return copy($source, $destination);
    }
    
    public function moveFile(string $source, string $destination): bool {
        $directory = dirname($destination);
        
        if (!is_dir($directory)) {
            mkdir($directory, 0755, true);
        }
        
        return rename($source, $destination);
    }
    
    // File deletion
    public function deleteFile(string $filepath): bool {
        if (file_exists($filepath)) {
            return unlink($filepath);
        }
        return true; // File doesn't exist, consider it deleted
    }
    
    // Directory operations
    public function createDirectory(string $path, int $permissions = 0755): bool {
        if (!is_dir($path)) {
            return mkdir($path, $permissions, true);
        }
        return true;
    }
    
    public function deleteDirectory(string $path): bool {
        if (!is_dir($path)) {
            return true;
        }
        
        $files = array_diff(scandir($path), ['.', '..']);
        
        foreach ($files as $file) {
            $filePath = $path . DIRECTORY_SEPARATOR . $file;
            
            if (is_dir($filePath)) {
                $this->deleteDirectory($filePath);
            } else {
                unlink($filePath);
            }
        }
        
        return rmdir($path);
    }
    
    // File listing
    public function listFiles(string $directory, string $pattern = '*', bool $recursive = false): array {
        if (!is_dir($directory)) {
            throw new DirectoryNotFoundException("Directory not found: $directory");
        }
        
        $files = [];
        
        if ($recursive) {
            $iterator = new RecursiveIteratorIterator(
                new RecursiveDirectoryIterator($directory)
            );
            
            foreach ($iterator as $file) {
                if ($file->isFile() && fnmatch($pattern, $file->getFilename())) {
                    $files[] = $file->getPathname();
                }
            }
        } else {
            $files = glob($directory . DIRECTORY_SEPARATOR . $pattern);
        }
        
        return $files;
    }
    
    // File upload handling
    public function handleFileUpload(array $file, string $uploadDir, array $allowedTypes = []): array {
        if ($file['error'] !== UPLOAD_ERR_OK) {
            throw new UploadException("Upload error: " . $file['error']);
        }
        
        $filename = $file['name'];
        $tmpName = $file['tmp_name'];
        $size = $file['size'];
        $type = $file['type'];
        
        // Validate file type
        if (!empty($allowedTypes) && !in_array($type, $allowedTypes)) {
            throw new UploadException("File type not allowed: $type");
        }
        
        // Generate unique filename
        $extension = pathinfo($filename, PATHINFO_EXTENSION);
        $uniqueFilename = uniqid() . '.' . $extension;
        $uploadPath = $uploadDir . DIRECTORY_SEPARATOR . $uniqueFilename;
        
        // Create upload directory if it doesn't exist
        $this->createDirectory($uploadDir);
        
        // Move uploaded file
        if (!move_uploaded_file($tmpName, $uploadPath)) {
            throw new UploadException("Failed to move uploaded file");
        }
        
        return [
            'original_name' => $filename,
            'stored_name' => $uniqueFilename,
            'path' => $uploadPath,
            'size' => $size,
            'type' => $type,
            'extension' => $extension
        ];
    }
    
    // CSV file operations
    public function readCsvFile(string $filepath, string $delimiter = ','): array {
        $data = [];
        
        if (($handle = fopen($filepath, 'r')) !== false) {
            while (($row = fgetcsv($handle, 1000, $delimiter)) !== false) {
                $data[] = $row;
            }
            fclose($handle);
        }
        
        return $data;
    }
    
    public function writeCsvFile(string $filepath, array $data, string $delimiter = ','): bool {
        if (($handle = fopen($filepath, 'w')) !== false) {
            foreach ($data as $row) {
                fputcsv($handle, $row, $delimiter);
            }
            fclose($handle);
            return true;
        }
        
        return false;
    }
    
    // JSON file operations
    public function readJsonFile(string $filepath): array {
        $content = file_get_contents($filepath);
        $data = json_decode($content, true);
        
        if (json_last_error() !== JSON_ERROR_NONE) {
            throw new JsonException("JSON decode error: " . json_last_error_msg());
        }
        
        return $data;
    }
    
    public function writeJsonFile(string $filepath, array $data, int $flags = JSON_PRETTY_PRINT): bool {
        $json = json_encode($data, $flags);
        
        if (json_last_error() !== JSON_ERROR_NONE) {
            throw new JsonException("JSON encode error: " . json_last_error_msg());
        }
        
        return $this->writeFile($filepath, $json);
    }
    
    // File compression
    public function compressFile(string $source, string $destination): bool {
        $zip = new ZipArchive();
        
        if ($zip->open($destination, ZipArchive::CREATE) !== true) {
            return false;
        }
        
        $zip->addFile($source, basename($source));
        $zip->close();
        
        return true;
    }
    
    public function extractFile(string $source, string $destination): bool {
        $zip = new ZipArchive();
        
        if ($zip->open($source) !== true) {
            return false;
        }
        
        $zip->extractTo($destination);
        $zip->close();
        
        return true;
    }
    
    // File monitoring
    public function monitorFile(string $filepath, callable $callback): void {
        $lastModified = filemtime($filepath);
        
        while (true) {
            clearstatcache(true, $filepath);
            
            if (filemtime($filepath) > $lastModified) {
                $lastModified = filemtime($filepath);
                $callback($filepath);
            }
            
            sleep(1);
        }
    }
    
    // File locking
    public function readFileWithLock(string $filepath): string {
        $handle = fopen($filepath, 'r');
        
        if (flock($handle, LOCK_SH)) {
            $content = fread($handle, filesize($filepath));
            flock($handle, LOCK_UN);
            fclose($handle);
            return $content;
        }
        
        fclose($handle);
        throw new FileLockException("Could not acquire read lock on file: $filepath");
    }
    
    public function writeFileWithLock(string $filepath, string $content): bool {
        $handle = fopen($filepath, 'w');
        
        if (flock($handle, LOCK_EX)) {
            fwrite($handle, $content);
            flock($handle, LOCK_UN);
            fclose($handle);
            return true;
        }
        
        fclose($handle);
        throw new FileLockException("Could not acquire write lock on file: $filepath");
    }
}

// Custom exceptions
class FileNotFoundException extends Exception {}
class DirectoryNotFoundException extends Exception {}
class UploadException extends Exception {}
class JsonException extends Exception {}
class FileLockException extends Exception {}

// Usage examples
$fileHandler = new FileHandler();

// Read file
$fileInfo = $fileHandler->readFile('example.txt');

// Write file
$fileHandler->writeFile('output.txt', 'Hello World!');

// Handle file upload
$uploadedFile = $fileHandler->handleFileUpload($_FILES['file'], 'uploads');

// Read CSV
$csvData = $fileHandler->readCsvFile('data.csv');

// Write JSON
$fileHandler->writeJsonFile('config.json', ['setting' => 'value']);

// Monitor file changes
$fileHandler->monitorFile('log.txt', function($filepath) {
    echo "File changed: $filepath\n";
});
```

---

## Sessions & Cookies

### Q12. How do you manage sessions and cookies in PHP?
**Answer:** Sessions and cookies are essential for maintaining state in web applications.

**Real-time Example:**
```php
class SessionManager {
    
    public function __construct() {
        $this->startSession();
    }
    
    private function startSession(): void {
        if (session_status() === PHP_SESSION_NONE) {
            // Configure session settings
            ini_set('session.cookie_httponly', 1);
            ini_set('session.use_only_cookies', 1);
            ini_set('session.cookie_secure', isset($_SERVER['HTTPS']));
            
            session_start();
        }
    }
    
    // Session operations
    public function set(string $key, mixed $value): void {
        $_SESSION[$key] = $value;
    }
    
    public function get(string $key, mixed $default = null): mixed {
        return $_SESSION[$key] ?? $default;
    }
    
    public function has(string $key): bool {
        return isset($_SESSION[$key]);
    }
    
    public function remove(string $key): void {
        unset($_SESSION[$key]);
    }
    
    public function destroy(): void {
        session_destroy();
    }
    
    public function regenerateId(): void {
        session_regenerate_id(true);
    }
    
    // Flash messages
    public function flash(string $key, mixed $value): void {
        $_SESSION['_flash'][$key] = $value;
    }
    
    public function getFlash(string $key, mixed $default = null): mixed {
        $value = $_SESSION['_flash'][$key] ?? $default;
        unset($_SESSION['_flash'][$key]);
        return $value;
    }
    
    // Session data
    public function all(): array {
        return $_SESSION;
    }
    
    public function flush(): void {
        $_SESSION = [];
    }
    
    // Session security
    public function validateSession(): bool {
        // Check session timeout
        if (isset($_SESSION['last_activity']) && 
            (time() - $_SESSION['last_activity'] > 1800)) { // 30 minutes
            $this->destroy();
            return false;
        }
        
        // Update last activity
        $_SESSION['last_activity'] = time();
        
        // Check IP address
        if (isset($_SESSION['ip_address']) && 
            $_SESSION['ip_address'] !== $_SERVER['REMOTE_ADDR']) {
            $this->destroy();
            return false;
        }
        
        // Check user agent
        if (isset($_SESSION['user_agent']) && 
            $_SESSION['user_agent'] !== $_SERVER['HTTP_USER_AGENT']) {
            $this->destroy();
            return false;
        }
        
        return true;
    }
    
    public function setSecurityData(): void {
        $_SESSION['ip_address'] = $_SERVER['REMOTE_ADDR'];
        $_SESSION['user_agent'] = $_SERVER['HTTP_USER_AGENT'];
        $_SESSION['last_activity'] = time();
    }
}

class CookieManager {
    
    // Cookie operations
    public function set(string $name, string $value, int $expire = 0, string $path = '/', 
                       string $domain = '', bool $secure = false, bool $httponly = true): bool {
        return setcookie($name, $value, $expire, $path, $domain, $secure, $httponly);
    }
    
    public function get(string $name, string $default = ''): string {
        return $_COOKIE[$name] ?? $default;
    }
    
    public function has(string $name): bool {
        return isset($_COOKIE[$name]);
    }
    
    public function delete(string $name, string $path = '/', string $domain = ''): bool {
        return setcookie($name, '', time() - 3600, $path, $domain);
    }
    
    // Secure cookie operations
    public function setSecure(string $name, string $value, int $expire = 0): bool {
        $secure = isset($_SERVER['HTTPS']);
        return $this->set($name, $value, $expire, '/', '', $secure, true);
    }
    
    public function setRememberMe(string $name, string $value): bool {
        $expire = time() + (30 * 24 * 60 * 60); // 30 days
        return $this->setSecure($name, $value, $expire);
    }
    
    // Cookie encryption
    public function setEncrypted(string $name, string $value, string $key, int $expire = 0): bool {
        $encrypted = $this->encrypt($value, $key);
        return $this->setSecure($name, $encrypted, $expire);
    }
    
    public function getDecrypted(string $name, string $key, string $default = ''): string {
        $encrypted = $this->get($name, '');
        
        if (empty($encrypted)) {
            return $default;
        }
        
        return $this->decrypt($encrypted, $key) ?: $default;
    }
    
    private function encrypt(string $data, string $key): string {
        $iv = random_bytes(16);
        $encrypted = openssl_encrypt($data, 'AES-256-CBC', $key, 0, $iv);
        return base64_encode($iv . $encrypted);
    }
    
    private function decrypt(string $data, string $key): string {
        $data = base64_decode($data);
        $iv = substr($data, 0, 16);
        $encrypted = substr($data, 16);
        return openssl_decrypt($encrypted, 'AES-256-CBC', $key, 0, $iv);
    }
    
    // Cookie validation
    public function validateCookie(string $name, string $key): bool {
        $value = $this->getDecrypted($name, $key);
        return !empty($value);
    }
    
    // Cookie cleanup
    public function cleanupExpiredCookies(): void {
        foreach ($_COOKIE as $name => $value) {
            if (empty($value)) {
                $this->delete($name);
            }
        }
    }
}

class AuthenticationManager {
    
    private SessionManager $session;
    private CookieManager $cookie;
    private string $secretKey;
    
    public function __construct(string $secretKey) {
        $this->session = new SessionManager();
        $this->cookie = new CookieManager();
        $this->secretKey = $secretKey;
    }
    
    // User authentication
    public function login(array $user, bool $rememberMe = false): void {
        // Set session data
        $this->session->set('user_id', $user['id']);
        $this->session->set('user_email', $user['email']);
        $this->session->set('user_role', $user['role']);
        $this->session->set('logged_in', true);
        $this->session->setSecurityData();
        
        // Set remember me cookie
        if ($rememberMe) {
            $token = $this->generateRememberToken($user['id']);
            $this->cookie->setEncrypted('remember_token', $token, $this->secretKey);
            $this->storeRememberToken($user['id'], $token);
        }
        
        // Regenerate session ID for security
        $this->session->regenerateId();
    }
    
    public function logout(): void {
        // Clear remember me token
        if ($this->cookie->has('remember_token')) {
            $token = $this->cookie->getDecrypted('remember_token', $this->secretKey);
            $this->deleteRememberToken($token);
            $this->cookie->delete('remember_token');
        }
        
        // Destroy session
        $this->session->destroy();
    }
    
    public function isLoggedIn(): bool {
        // Check session
        if ($this->session->get('logged_in', false)) {
            return $this->session->validateSession();
        }
        
        // Check remember me token
        if ($this->cookie->has('remember_token')) {
            $token = $this->cookie->getDecrypted('remember_token', $this->secretKey);
            
            if ($this->validateRememberToken($token)) {
                $userId = $this->getUserIdFromToken($token);
                $user = $this->getUserById($userId);
                
                if ($user) {
                    $this->login($user, false);
                    return true;
                }
            }
        }
        
        return false;
    }
    
    public function getCurrentUser(): ?array {
        if (!$this->isLoggedIn()) {
            return null;
        }
        
        return [
            'id' => $this->session->get('user_id'),
            'email' => $this->session->get('user_email'),
            'role' => $this->session->get('user_role')
        ];
    }
    
    public function hasRole(string $role): bool {
        $user = $this->getCurrentUser();
        return $user && $user['role'] === $role;
    }
    
    public function requireAuth(): void {
        if (!$this->isLoggedIn()) {
            $this->session->flash('error', 'Please log in to access this page');
            header('Location: /login');
            exit;
        }
    }
    
    public function requireRole(string $role): void {
        $this->requireAuth();
        
        if (!$this->hasRole($role)) {
            $this->session->flash('error', 'Insufficient permissions');
            header('Location: /unauthorized');
            exit;
        }
    }
    
    // Remember me functionality
    private function generateRememberToken(int $userId): string {
        return bin2hex(random_bytes(32));
    }
    
    private function storeRememberToken(int $userId, string $token): void {
        $expire = time() + (30 * 24 * 60 * 60); // 30 days
        
        // Store in database or cache
        // Example: database query
        // INSERT INTO remember_tokens (user_id, token, expires_at) VALUES (?, ?, ?)
    }
    
    private function validateRememberToken(string $token): bool {
        // Check in database or cache
        // Example: database query
        // SELECT * FROM remember_tokens WHERE token = ? AND expires_at > NOW()
        return true; // Simplified for example
    }
    
    private function deleteRememberToken(string $token): void {
        // Delete from database or cache
        // Example: database query
        // DELETE FROM remember_tokens WHERE token = ?
    }
    
    private function getUserIdFromToken(string $token): int {
        // Get user ID from token
        // Example: database query
        // SELECT user_id FROM remember_tokens WHERE token = ?
        return 1; // Simplified for example
    }
    
    private function getUserById(int $id): ?array {
        // Get user from database
        // Example: database query
        // SELECT * FROM users WHERE id = ?
        return ['id' => $id, 'email' => 'user@example.com', 'role' => 'user']; // Simplified for example
    }
}

// Usage examples
$authManager = new AuthenticationManager('your-secret-key');

// Login user
$user = ['id' => 1, 'email' => 'user@example.com', 'role' => 'admin'];
$authManager->login($user, true); // Remember me

// Check authentication
if ($authManager->isLoggedIn()) {
    $currentUser = $authManager->getCurrentUser();
    echo "Welcome, " . $currentUser['email'];
}

// Check role
if ($authManager->hasRole('admin')) {
    echo "Admin access granted";
}

// Require authentication
$authManager->requireAuth();

// Require specific role
$authManager->requireRole('admin');

// Logout
$authManager->logout();
```

---

## Error Handling & Exceptions

### Q13. How do you handle errors and exceptions in PHP?
**Answer:** PHP provides various mechanisms for error handling including exceptions, error reporting, and custom error handlers.

**Real-time Example:**
```php
// Custom exception classes
class ValidationException extends Exception {
    private array $errors;
    
    public function __construct(array $errors, string $message = 'Validation failed') {
        $this->errors = $errors;
        parent::__construct($message);
    }
    
    public function getErrors(): array {
        return $this->errors;
    }
}

class DatabaseException extends Exception {
    private string $query;
    
    public function __construct(string $message, string $query = '', int $code = 0, Exception $previous = null) {
        $this->query = $query;
        parent::__construct($message, $code, $previous);
    }
    
    public function getQuery(): string {
        return $this->query;
    }
}

class FileException extends Exception {}

class AuthenticationException extends Exception {}

// Error handler class
class ErrorHandler {
    
    private static bool $debug = false;
    private static string $logFile = 'error.log';
    
    public static function setDebug(bool $debug): void {
        self::$debug = $debug;
    }
    
    public static function setLogFile(string $logFile): void {
        self::$logFile = $logFile;
    }
    
    public static function register(): void {
        set_error_handler([self::class, 'handleError']);
        set_exception_handler([self::class, 'handleException']);
        register_shutdown_function([self::class, 'handleShutdown']);
    }
    
    public static function handleError(int $level, string $message, string $file, int $line): bool {
        if (!(error_reporting() & $level)) {
            return false;
        }
        
        $error = [
            'type' => 'Error',
            'level' => self::getErrorLevelName($level),
            'message' => $message,
            'file' => $file,
            'line' => $line,
            'timestamp' => date('Y-m-d H:i:s'),
            'trace' => debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS)
        ];
        
        self::logError($error);
        
        if (self::$debug) {
            self::displayError($error);
        }
        
        return true;
    }
    
    public static function handleException(Throwable $exception): void {
        $error = [
            'type' => 'Exception',
            'class' => get_class($exception),
            'message' => $exception->getMessage(),
            'file' => $exception->getFile(),
            'line' => $exception->getLine(),
            'timestamp' => date('Y-m-d H:i:s'),
            'trace' => $exception->getTraceAsString()
        ];
        
        self::logError($error);
        
        if (self::$debug) {
            self::displayError($error);
        } else {
            self::displayGenericError();
        }
    }
    
    public static function handleShutdown(): void {
        $error = error_get_last();
        
        if ($error && in_array($error['type'], [E_ERROR, E_PARSE, E_CORE_ERROR, E_COMPILE_ERROR])) {
            $errorData = [
                'type' => 'Fatal Error',
                'level' => self::getErrorLevelName($error['type']),
                'message' => $error['message'],
                'file' => $error['file'],
                'line' => $error['line'],
                'timestamp' => date('Y-m-d H:i:s')
            ];
            
            self::logError($errorData);
            
            if (self::$debug) {
                self::displayError($errorData);
            } else {
                self::displayGenericError();
            }
        }
    }
    
    private static function getErrorLevelName(int $level): string {
        $levels = [
            E_ERROR => 'Error',
            E_WARNING => 'Warning',
            E_PARSE => 'Parse Error',
            E_NOTICE => 'Notice',
            E_CORE_ERROR => 'Core Error',
            E_CORE_WARNING => 'Core Warning',
            E_COMPILE_ERROR => 'Compile Error',
            E_COMPILE_WARNING => 'Compile Warning',
            E_USER_ERROR => 'User Error',
            E_USER_WARNING => 'User Warning',
            E_USER_NOTICE => 'User Notice',
            E_STRICT => 'Strict Notice',
            E_RECOVERABLE_ERROR => 'Recoverable Error',
            E_DEPRECATED => 'Deprecated',
            E_USER_DEPRECATED => 'User Deprecated'
        ];
        
        return $levels[$level] ?? 'Unknown';
    }
    
    private static function logError(array $error): void {
        $logEntry = json_encode($error) . "\n";
        file_put_contents(self::$logFile, $logEntry, FILE_APPEND | LOCK_EX);
    }
    
    private static function displayError(array $error): void {
        echo "<div style='background: #f8d7da; color: #721c24; padding: 20px; margin: 20px; border: 1px solid #f5c6cb; border-radius: 5px;'>";
        echo "<h3>{$error['type']}: {$error['message']}</h3>";
        echo "<p><strong>File:</strong> {$error['file']} on line {$error['line']}</p>";
        echo "<p><strong>Time:</strong> {$error['timestamp']}</p>";
        
        if (isset($error['trace'])) {
            echo "<h4>Stack Trace:</h4>";
            echo "<pre>" . htmlspecialchars($error['trace']) . "</pre>";
        }
        
        echo "</div>";
    }
    
    private static function displayGenericError(): void {
        http_response_code(500);
        echo "<h1>Internal Server Error</h1>";
        echo "<p>Something went wrong. Please try again later.</p>";
    }
}

// Validation class with exception handling
class Validator {
    
    public function validateUser(array $data): void {
        $errors = [];
        
        // Name validation
        if (empty($data['name'])) {
            $errors['name'] = 'Name is required';
        } elseif (strlen($data['name']) < 2) {
            $errors['name'] = 'Name must be at least 2 characters';
        }
        
        // Email validation
        if (empty($data['email'])) {
            $errors['email'] = 'Email is required';
        } elseif (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
            $errors['email'] = 'Invalid email format';
        }
        
        // Password validation
        if (empty($data['password'])) {
            $errors['password'] = 'Password is required';
        } elseif (strlen($data['password']) < 8) {
            $errors['password'] = 'Password must be at least 8 characters';
        }
        
        // Age validation
        if (!isset($data['age']) || !is_numeric($data['age'])) {
            $errors['age'] = 'Age must be a number';
        } elseif ($data['age'] < 18) {
            $errors['age'] = 'Age must be at least 18';
        }
        
        if (!empty($errors)) {
            throw new ValidationException($errors);
        }
    }
    
    public function validateEmail(string $email): void {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new ValidationException(['email' => 'Invalid email format']);
        }
    }
    
    public function validatePassword(string $password): void {
        $errors = [];
        
        if (strlen($password) < 8) {
            $errors[] = 'Password must be at least 8 characters';
        }
        
        if (!preg_match('/[A-Z]/', $password)) {
            $errors[] = 'Password must contain at least one uppercase letter';
        }
        
        if (!preg_match('/[a-z]/', $password)) {
            $errors[] = 'Password must contain at least one lowercase letter';
        }
        
        if (!preg_match('/[0-9]/', $password)) {
            $errors[] = 'Password must contain at least one number';
        }
        
        if (!preg_match('/[^A-Za-z0-9]/', $password)) {
            $errors[] = 'Password must contain at least one special character';
        }
        
        if (!empty($errors)) {
            throw new ValidationException($errors, 'Password validation failed');
        }
    }
}

// Database class with exception handling
class Database {
    
    private PDO $connection;
    
    public function __construct(string $dsn, string $username, string $password) {
        try {
            $this->connection = new PDO($dsn, $username, $password, [
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
            ]);
        } catch (PDOException $e) {
            throw new DatabaseException("Database connection failed: " . $e->getMessage());
        }
    }
    
    public function query(string $sql, array $params = []): array {
        try {
            $stmt = $this->connection->prepare($sql);
            $stmt->execute($params);
            return $stmt->fetchAll();
        } catch (PDOException $e) {
            throw new DatabaseException("Query failed: " . $e->getMessage(), $sql, 0, $e);
        }
    }
    
    public function execute(string $sql, array $params = []): int {
        try {
            $stmt = $this->connection->prepare($sql);
            $stmt->execute($params);
            return $stmt->rowCount();
        } catch (PDOException $e) {
            throw new DatabaseException("Execute failed: " . $e->getMessage(), $sql, 0, $e);
        }
    }
    
    public function transaction(callable $callback): mixed {
        try {
            $this->connection->beginTransaction();
            $result = $callback($this);
            $this->connection->commit();
            return $result;
        } catch (Exception $e) {
            $this->connection->rollBack();
            throw $e;
        }
    }
}

// File handler with exception handling
class FileHandler {
    
    public function readFile(string $filepath): string {
        if (!file_exists($filepath)) {
            throw new FileException("File not found: $filepath");
        }
        
        if (!is_readable($filepath)) {
            throw new FileException("File is not readable: $filepath");
        }
        
        $content = file_get_contents($filepath);
        
        if ($content === false) {
            throw new FileException("Failed to read file: $filepath");
        }
        
        return $content;
    }
    
    public function writeFile(string $filepath, string $content): void {
        $directory = dirname($filepath);
        
        if (!is_dir($directory)) {
            if (!mkdir($directory, 0755, true)) {
                throw new FileException("Failed to create directory: $directory");
            }
        }
        
        if (file_put_contents($filepath, $content) === false) {
            throw new FileException("Failed to write file: $filepath");
        }
    }
}

// Service class with comprehensive error handling
class UserService {
    
    private Database $database;
    private Validator $validator;
    private FileHandler $fileHandler;
    
    public function __construct(Database $database, Validator $validator, FileHandler $fileHandler) {
        $this->database = $database;
        $this->validator = $validator;
        $this->fileHandler = $fileHandler;
    }
    
    public function createUser(array $userData): array {
        try {
            // Validate user data
            $this->validator->validateUser($userData);
            
            // Check if user already exists
            $existingUser = $this->database->query(
                "SELECT id FROM users WHERE email = ?",
                [$userData['email']]
            );
            
            if (!empty($existingUser)) {
                throw new ValidationException(['email' => 'Email already exists']);
            }
            
            // Hash password
            $hashedPassword = password_hash($userData['password'], PASSWORD_DEFAULT);
            
            // Create user
            $userId = $this->database->execute(
                "INSERT INTO users (name, email, password, age, created_at) VALUES (?, ?, ?, ?, NOW())",
                [$userData['name'], $userData['email'], $hashedPassword, $userData['age']]
            );
            
            // Log user creation
            $this->logUserActivity($userId, 'user_created');
            
            return ['id' => $userId, 'message' => 'User created successfully'];
            
        } catch (ValidationException $e) {
            // Re-throw validation exceptions
            throw $e;
        } catch (DatabaseException $e) {
            // Log database errors
            error_log("Database error in createUser: " . $e->getMessage());
            throw new Exception("Failed to create user. Please try again later.");
        } catch (Exception $e) {
            // Log unexpected errors
            error_log("Unexpected error in createUser: " . $e->getMessage());
            throw new Exception("An unexpected error occurred. Please try again later.");
        }
    }
    
    public function getUser(int $userId): array {
        try {
            $users = $this->database->query(
                "SELECT id, name, email, age, created_at FROM users WHERE id = ?",
                [$userId]
            );
            
            if (empty($users)) {
                throw new Exception("User not found");
            }
            
            return $users[0];
            
        } catch (DatabaseException $e) {
            error_log("Database error in getUser: " . $e->getMessage());
            throw new Exception("Failed to retrieve user information.");
        }
    }
    
    public function updateUser(int $userId, array $userData): array {
        try {
            // Validate user data
            $this->validator->validateUser($userData);
            
            // Update user
            $affectedRows = $this->database->execute(
                "UPDATE users SET name = ?, email = ?, age = ?, updated_at = NOW() WHERE id = ?",
                [$userData['name'], $userData['email'], $userData['age'], $userId]
            );
            
            if ($affectedRows === 0) {
                throw new Exception("User not found or no changes made");
            }
            
            // Log user update
            $this->logUserActivity($userId, 'user_updated');
            
            return ['message' => 'User updated successfully'];
            
        } catch (ValidationException $e) {
            throw $e;
        } catch (DatabaseException $e) {
            error_log("Database error in updateUser: " . $e->getMessage());
            throw new Exception("Failed to update user. Please try again later.");
        } catch (Exception $e) {
            error_log("Unexpected error in updateUser: " . $e->getMessage());
            throw $e;
        }
    }
    
    public function deleteUser(int $userId): array {
        try {
            // Check if user exists
            $user = $this->getUser($userId);
            
            // Delete user
            $affectedRows = $this->database->execute(
                "DELETE FROM users WHERE id = ?",
                [$userId]
            );
            
            if ($affectedRows === 0) {
                throw new Exception("User not found");
            }
            
            // Log user deletion
            $this->logUserActivity($userId, 'user_deleted');
            
            return ['message' => 'User deleted successfully'];
            
        } catch (DatabaseException $e) {
            error_log("Database error in deleteUser: " . $e->getMessage());
            throw new Exception("Failed to delete user. Please try again later.");
        } catch (Exception $e) {
            error_log("Unexpected error in deleteUser: " . $e->getMessage());
            throw $e;
        }
    }
    
    private function logUserActivity(int $userId, string $action): void {
        try {
            $this->database->execute(
                "INSERT INTO user_activities (user_id, action, created_at) VALUES (?, ?, NOW())",
                [$userId, $action]
            );
        } catch (DatabaseException $e) {
            // Log error but don't throw exception for logging failures
            error_log("Failed to log user activity: " . $e->getMessage());
        }
    }
}

// Usage examples
// Register error handler
ErrorHandler::setDebug(true);
ErrorHandler::register();

// Initialize services
$database = new Database('mysql:host=localhost;dbname=test', 'username', 'password');
$validator = new Validator();
$fileHandler = new FileHandler();
$userService = new UserService($database, $validator, $fileHandler);

// Handle user creation
try {
    $userData = [
        'name' => 'John Doe',
        'email' => 'john@example.com',
        'password' => 'SecurePass123!',
        'age' => 25
    ];
    
    $result = $userService->createUser($userData);
    echo "Success: " . $result['message'];
    
} catch (ValidationException $e) {
    echo "Validation errors: " . implode(', ', $e->getErrors());
} catch (Exception $e) {
    echo "Error: " . $e->getMessage();
}
```

---

## Security

### Q14. What are the main security concerns in PHP and how do you address them?
**Answer:** PHP applications face various security threats that need to be addressed through proper coding practices and security measures.

**Real-time Example:**
```php
class SecurityManager {
    
    // SQL Injection Prevention
    public function safeQuery(PDO $pdo, string $sql, array $params = []): array {
        try {
            $stmt = $pdo->prepare($sql);
            $stmt->execute($params);
            return $stmt->fetchAll(PDO::FETCH_ASSOC);
        } catch (PDOException $e) {
            throw new Exception("Database query failed");
        }
    }
    
    // XSS Prevention
    public function sanitizeOutput(string $data): string {
        return htmlspecialchars($data, ENT_QUOTES | ENT_HTML5, 'UTF-8');
    }
    
    public function sanitizeInput(string $data): string {
        return htmlspecialchars(strip_tags(trim($data)), ENT_QUOTES, 'UTF-8');
    }
    
    // CSRF Protection
    public function generateCSRFToken(): string {
        if (session_status() === PHP_SESSION_NONE) {
            session_start();
        }
        
        $token = bin2hex(random_bytes(32));
        $_SESSION['csrf_token'] = $token;
        return $token;
    }
    
    public function validateCSRFToken(string $token): bool {
        if (session_status() === PHP_SESSION_NONE) {
            session_start();
        }
        
        return isset($_SESSION['csrf_token']) && hash_equals($_SESSION['csrf_token'], $token);
    }
    
    // Password Security
    public function hashPassword(string $password): string {
        return password_hash($password, PASSWORD_ARGON2ID, [
            'memory_cost' => 65536, // 64 MB
            'time_cost' => 4,       // 4 iterations
            'threads' => 3          // 3 threads
        ]);
    }
    
    public function verifyPassword(string $password, string $hash): bool {
        return password_verify($password, $hash);
    }
    
    // Input Validation
    public function validateEmail(string $email): bool {
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }
    
    public function validateURL(string $url): bool {
        return filter_var($url, FILTER_VALIDATE_URL) !== false;
    }
    
    public function validateIP(string $ip): bool {
        return filter_var($ip, FILTER_VALIDATE_IP) !== false;
    }
    
    public function sanitizeFilename(string $filename): string {
        // Remove dangerous characters
        $filename = preg_replace('/[^a-zA-Z0-9._-]/', '', $filename);
        
        // Remove multiple dots
        $filename = preg_replace('/\.{2,}/', '.', $filename);
        
        // Ensure filename doesn't start with dot
        $filename = ltrim($filename, '.');
        
        return $filename;
    }
    
    // File Upload Security
    public function validateFileUpload(array $file, array $allowedTypes = [], int $maxSize = 5242880): array {
        $errors = [];
        
        // Check for upload errors
        if ($file['error'] !== UPLOAD_ERR_OK) {
            $errors[] = 'File upload failed';
            return $errors;
        }
        
        // Check file size
        if ($file['size'] > $maxSize) {
            $errors[] = 'File size exceeds maximum allowed size';
        }
        
        // Check file type
        $finfo = finfo_open(FILEINFO_MIME_TYPE);
        $mimeType = finfo_file($finfo, $file['tmp_name']);
        finfo_close($finfo);
        
        if (!empty($allowedTypes) && !in_array($mimeType, $allowedTypes)) {
            $errors[] = 'File type not allowed';
        }
        
        // Check file extension
        $extension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
        $dangerousExtensions = ['php', 'phtml', 'php3', 'php4', 'php5', 'pl', 'py', 'jsp', 'asp', 'sh', 'cgi'];
        
        if (in_array($extension, $dangerousExtensions)) {
            $errors[] = 'Dangerous file extension not allowed';
        }
        
        // Scan file content for malicious code
        $content = file_get_contents($file['tmp_name']);
        $maliciousPatterns = [
            '/<\?php/i',
            '/<\?=/i',
            '/<script/i',
            '/javascript:/i',
            '/vbscript:/i',
            '/onload=/i',
            '/onerror=/i'
        ];
        
        foreach ($maliciousPatterns as $pattern) {
            if (preg_match($pattern, $content)) {
                $errors[] = 'File contains potentially malicious content';
                break;
            }
        }
        
        return $errors;
    }
    
    // Rate Limiting
    public function checkRateLimit(string $identifier, int $limit = 100, int $window = 3600): bool {
        $key = 'rate_limit_' . md5($identifier);
        
        if (session_status() === PHP_SESSION_NONE) {
            session_start();
        }
        
        $now = time();
        $requests = $_SESSION[$key] ?? [];
        
        // Remove old requests outside the window
        $requests = array_filter($requests, function($timestamp) use ($now, $window) {
            return ($now - $timestamp) < $window;
        });
        
        // Check if limit exceeded
        if (count($requests) >= $limit) {
            return false;
        }
        
        // Add current request
        $requests[] = $now;
        $_SESSION[$key] = $requests;
        
        return true;
    }
    
    // Session Security
    public function secureSession(): void {
        if (session_status() === PHP_SESSION_NONE) {
            // Configure secure session settings
            ini_set('session.cookie_httponly', 1);
            ini_set('session.use_only_cookies', 1);
            ini_set('session.cookie_secure', isset($_SERVER['HTTPS']));
            ini_set('session.cookie_samesite', 'Strict');
            
            session_start();
        }
        
        // Regenerate session ID periodically
        if (!isset($_SESSION['last_regeneration'])) {
            $_SESSION['last_regeneration'] = time();
        } elseif (time() - $_SESSION['last_regeneration'] > 300) { // 5 minutes
            session_regenerate_id(true);
            $_SESSION['last_regeneration'] = time();
        }
        
        // Validate session data
        if (isset($_SESSION['ip_address']) && $_SESSION['ip_address'] !== $_SERVER['REMOTE_ADDR']) {
            session_destroy();
            throw new Exception('Session hijacking detected');
        }
        
        if (isset($_SESSION['user_agent']) && $_SESSION['user_agent'] !== $_SERVER['HTTP_USER_AGENT']) {
            session_destroy();
            throw new Exception('Session hijacking detected');
        }
        
        // Set session data
        $_SESSION['ip_address'] = $_SERVER['REMOTE_ADDR'];
        $_SESSION['user_agent'] = $_SERVER['HTTP_USER_AGENT'];
    }
    
    // Encryption/Decryption
    public function encrypt(string $data, string $key): string {
        $iv = random_bytes(16);
        $encrypted = openssl_encrypt($data, 'AES-256-CBC', $key, 0, $iv);
        return base64_encode($iv . $encrypted);
    }
    
    public function decrypt(string $data, string $key): string {
        $data = base64_decode($data);
        $iv = substr($data, 0, 16);
        $encrypted = substr($data, 16);
        return openssl_decrypt($encrypted, 'AES-256-CBC', $key, 0, $iv);
    }
    
    // Secure Random String Generation
    public function generateSecureToken(int $length = 32): string {
        return bin2hex(random_bytes($length));
    }
    
    // Headers Security
    public function setSecurityHeaders(): void {
        // Prevent XSS
        header('X-Content-Type-Options: nosniff');
        header('X-Frame-Options: DENY');
        header('X-XSS-Protection: 1; mode=block');
        
        // Content Security Policy
        header("Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'");
        
        // Strict Transport Security
        if (isset($_SERVER['HTTPS'])) {
            header('Strict-Transport-Security: max-age=31536000; includeSubDomains');
        }
        
        // Referrer Policy
        header('Referrer-Policy: strict-origin-when-cross-origin');
    }
    
    // Input Sanitization
    public function sanitizeArray(array $data): array {
        return array_map(function($value) {
            if (is_array($value)) {
                return $this->sanitizeArray($value);
            }
            return $this->sanitizeInput($value);
        }, $data);
    }
    
    // Log Security Events
    public function logSecurityEvent(string $event, array $data = []): void {
        $logEntry = [
            'timestamp' => date('Y-m-d H:i:s'),
            'event' => $event,
            'ip' => $_SERVER['REMOTE_ADDR'] ?? 'unknown',
            'user_agent' => $_SERVER['HTTP_USER_AGENT'] ?? 'unknown',
            'data' => $data
        ];
        
        file_put_contents('security.log', json_encode($logEntry) . "\n", FILE_APPEND | LOCK_EX);
    }
}

// Security Middleware
class SecurityMiddleware {
    
    private SecurityManager $security;
    
    public function __construct(SecurityManager $security) {
        $this->security = $security;
    }
    
    public function handleRequest(): void {
        // Set security headers
        $this->security->setSecurityHeaders();
        
        // Secure session
        $this->security->secureSession();
        
        // Check rate limit
        $clientId = $_SERVER['REMOTE_ADDR'] ?? 'unknown';
        if (!$this->security->checkRateLimit($clientId)) {
            http_response_code(429);
            echo 'Rate limit exceeded';
            exit;
        }
        
        // Validate CSRF token for POST requests
        if ($_SERVER['REQUEST_METHOD'] === 'POST') {
            $token = $_POST['csrf_token'] ?? '';
            if (!$this->security->validateCSRFToken($token)) {
                http_response_code(403);
                echo 'CSRF token validation failed';
                exit;
            }
        }
    }
}

// Usage examples
$security = new SecurityManager();
$middleware = new SecurityMiddleware($security);

// Handle request with security middleware
$middleware->handleRequest();

// Secure password hashing
$password = 'SecurePass123!';
$hashedPassword = $security->hashPassword($password);
$isValid = $security->verifyPassword($password, $hashedPassword);

// File upload security
$uploadedFile = $_FILES['file'];
$errors = $security->validateFileUpload($uploadedFile, ['image/jpeg', 'image/png'], 2097152); // 2MB max

if (empty($errors)) {
    $filename = $security->sanitizeFilename($uploadedFile['name']);
    // Process file upload
} else {
    foreach ($errors as $error) {
        echo $error . "\n";
    }
}

// Data encryption
$sensitiveData = 'Credit card number: 1234-5678-9012-3456';
$encrypted = $security->encrypt($sensitiveData, 'your-secret-key');
$decrypted = $security->decrypt($encrypted, 'your-secret-key');

// Generate CSRF token
$csrfToken = $security->generateCSRFToken();

// Log security events
$security->logSecurityEvent('failed_login_attempt', ['email' => 'user@example.com']);
```

---

## Performance & Best Practices

### Q15. What are the best practices for PHP performance optimization?
**Answer:** PHP performance can be optimized through various techniques including caching, database optimization, and code improvements.

**Real-time Example:**
```php
// Caching implementation
class CacheManager {
    
    private array $cache = [];
    private string $cacheDir;
    
    public function __construct(string $cacheDir = 'cache') {
        $this->cacheDir = $cacheDir;
        
        if (!is_dir($this->cacheDir)) {
            mkdir($this->cacheDir, 0755, true);
        }
    }
    
    // Memory cache
    public function get(string $key): mixed {
        return $this->cache[$key] ?? null;
    }
    
    public function set(string $key, mixed $value, int $ttl = 3600): void {
        $this->cache[$key] = [
            'value' => $value,
            'expires' => time() + $ttl
        ];
    }
    
    // File cache
    public function getFile(string $key): mixed {
        $filepath = $this->cacheDir . '/' . md5($key) . '.cache';
        
        if (!file_exists($filepath)) {
            return null;
        }
        
        $data = unserialize(file_get_contents($filepath));
        
        if ($data['expires'] < time()) {
            unlink($filepath);
            return null;
        }
        
        return $data['value'];
    }
    
    public function setFile(string $key, mixed $value, int $ttl = 3600): void {
        $filepath = $this->cacheDir . '/' . md5($key) . '.cache';
        
        $data = [
            'value' => $value,
            'expires' => time() + $ttl
        ];
        
        file_put_contents($filepath, serialize($data), LOCK_EX);
    }
    
    // Cache cleanup
    public function cleanup(): void {
        $files = glob($this->cacheDir . '/*.cache');
        
        foreach ($files as $file) {
            $data = unserialize(file_get_contents($file));
            
            if ($data['expires'] < time()) {
                unlink($file);
            }
        }
    }
}

// Database optimization
class OptimizedDatabase {
    
    private PDO $connection;
    private CacheManager $cache;
    
    public function __construct(PDO $connection, CacheManager $cache) {
        $this->connection = $connection;
        $this->cache = $cache;
    }
    
    // Prepared statements with caching
    public function query(string $sql, array $params = [], int $cacheTtl = 300): array {
        $cacheKey = md5($sql . serialize($params));
        
        // Check cache first
        $cached = $this->cache->get($cacheKey);
        if ($cached !== null) {
            return $cached;
        }
        
        // Execute query
        $stmt = $this->connection->prepare($sql);
        $stmt->execute($params);
        $result = $stmt->fetchAll(PDO::FETCH_ASSOC);
        
        // Cache result
        $this->cache->set($cacheKey, $result, $cacheTtl);
        
        return $result;
    }
    
    // Batch operations
    public function batchInsert(string $table, array $data): int {
        if (empty($data)) {
            return 0;
        }
        
        $columns = array_keys($data[0]);
        $placeholders = '(' . implode(',', array_fill(0, count($columns), '?')) . ')';
        $values = str_repeat($placeholders . ',', count($data) - 1) . $placeholders;
        
        $sql = "INSERT INTO $table (" . implode(',', $columns) . ") VALUES $values";
        
        $params = [];
        foreach ($data as $row) {
            $params = array_merge($params, array_values($row));
        }
        
        $stmt = $this->connection->prepare($sql);
        $stmt->execute($params);
        
        return $stmt->rowCount();
    }
    
    // Connection pooling simulation
    public function getConnection(): PDO {
        static $connections = [];
        
        $key = md5($this->connection->getAttribute(PDO::ATTR_DSN));
        
        if (!isset($connections[$key])) {
            $connections[$key] = $this->connection;
        }
        
        return $connections[$key];
    }
}

// Performance monitoring
class PerformanceMonitor {
    
    private array $metrics = [];
    private float $startTime;
    
    public function __construct() {
        $this->startTime = microtime(true);
    }
    
    public function startTimer(string $name): void {
        $this->metrics[$name] = [
            'start' => microtime(true),
            'end' => null,
            'duration' => null
        ];
    }
    
    public function endTimer(string $name): float {
        if (!isset($this->metrics[$name])) {
            return 0;
        }
        
        $this->metrics[$name]['end'] = microtime(true);
        $this->metrics[$name]['duration'] = $this->metrics[$name]['end'] - $this->metrics[$name]['start'];
        
        return $this->metrics[$name]['duration'];
    }
    
    public function getMetrics(): array {
        $totalTime = microtime(true) - $this->startTime;
        
        return [
            'total_time' => $totalTime,
            'timers' => $this->metrics,
            'memory_usage' => memory_get_usage(true),
            'peak_memory' => memory_get_peak_usage(true)
        ];
    }
    
    public function logMetrics(): void {
        $metrics = $this->getMetrics();
        
        $logEntry = [
            'timestamp' => date('Y-m-d H:i:s'),
            'metrics' => $metrics
        ];
        
        file_put_contents('performance.log', json_encode($logEntry) . "\n", FILE_APPEND | LOCK_EX);
    }
}

// Code optimization examples
class OptimizedCode {
    
    // String concatenation optimization
    public function optimizedStringConcatenation(array $strings): string {
        // Use implode instead of concatenation in loops
        return implode('', $strings);
    }
    
    // Array operations optimization
    public function optimizedArrayOperations(array $data): array {
        // Use array functions instead of loops when possible
        return array_map(function($item) {
            return $item * 2;
        }, $data);
    }
    
    // Memory optimization
    public function memoryEfficientProcessing(array $largeArray): array {
        $result = [];
        
        // Process in chunks to avoid memory issues
        $chunks = array_chunk($largeArray, 1000);
        
        foreach ($chunks as $chunk) {
            $processedChunk = array_map(function($item) {
                return $item * 2;
            }, $chunk);
            
            $result = array_merge($result, $processedChunk);
        }
        
        return $result;
    }
    
    // Lazy loading
    public function lazyLoadData(string $key, callable $loader): mixed {
        static $cache = [];
        
        if (!isset($cache[$key])) {
            $cache[$key] = $loader();
        }
        
        return $cache[$key];
    }
    
    // Early returns
    public function optimizedValidation(array $data): bool {
        // Use early returns to avoid nested conditions
        if (empty($data)) {
            return false;
        }
        
        if (!isset($data['required_field'])) {
            return false;
        }
        
        if (strlen($data['required_field']) < 3) {
            return false;
        }
        
        return true;
    }
    
    // Use of static variables
    public function getExpensiveCalculation(): float {
        static $result = null;
        
        if ($result === null) {
            // Perform expensive calculation only once
            $result = $this->performExpensiveCalculation();
        }
        
        return $result;
    }
    
    private function performExpensiveCalculation(): float {
        // Simulate expensive calculation
        $sum = 0;
        for ($i = 0; $i < 1000000; $i++) {
            $sum += sqrt($i);
        }
        return $sum;
    }
}

// Autoloading optimization
class OptimizedAutoloader {
    
    private array $classMap = [];
    private string $cacheFile;
    
    public function __construct(string $cacheFile = 'classmap.cache') {
        $this->cacheFile = $cacheFile;
        $this->loadClassMap();
    }
    
    public function register(): void {
        spl_autoload_register([$this, 'autoload']);
    }
    
    public function autoload(string $className): void {
        if (isset($this->classMap[$className])) {
            require_once $this->classMap[$className];
        }
    }
    
    private function loadClassMap(): void {
        if (file_exists($this->cacheFile)) {
            $this->classMap = unserialize(file_get_contents($this->cacheFile));
        } else {
            $this->generateClassMap();
        }
    }
    
    private function generateClassMap(): void {
        $this->classMap = [];
        
        // Scan directories for classes
        $directories = ['src', 'vendor'];
        
        foreach ($directories as $dir) {
            if (is_dir($dir)) {
                $this->scanDirectory($dir);
            }
        }
        
        // Cache the class map
        file_put_contents($this->cacheFile, serialize($this->classMap));
    }
    
    private function scanDirectory(string $dir): void {
        $iterator = new RecursiveIteratorIterator(
            new RecursiveDirectoryIterator($dir)
        );
        
        foreach ($iterator as $file) {
            if ($file->getExtension() === 'php') {
                $className = $this->extractClassName($file->getPathname());
                if ($className) {
                    $this->classMap[$className] = $file->getPathname();
                }
            }
        }
    }
    
    private function extractClassName(string $filepath): ?string {
        $content = file_get_contents($filepath);
        
        if (preg_match('/class\s+(\w+)/', $content, $matches)) {
            return $matches[1];
        }
        
        return null;
    }
}

// Usage examples
$monitor = new PerformanceMonitor();
$cache = new CacheManager();
$optimized = new OptimizedCode();

// Start performance monitoring
$monitor->startTimer('total_execution');

// Use caching
$cache->set('expensive_data', $expensiveData, 3600);
$cachedData = $cache->get('expensive_data');

// Optimized string operations
$strings = ['Hello', ' ', 'World', '!'];
$result = $optimized->optimizedStringConcatenation($strings);

// Memory efficient processing
$largeArray = range(1, 100000);
$processed = $optimized->memoryEfficientProcessing($largeArray);

// Lazy loading
$data = $optimized->lazyLoadData('expensive_key', function() {
    return $optimized->getExpensiveCalculation();
});

// End monitoring and log metrics
$monitor->endTimer('total_execution');
$monitor->logMetrics();

// Autoloader optimization
$autoloader = new OptimizedAutoloader();
$autoloader->register();
```

This completes all the missing sections for the PHP Core interview questions and answers. The document now covers:

1. ✅ PHP Basics
2. ✅ Data Types & Variables  
3. ✅ Functions
4. ✅ Arrays
5. ✅ String Manipulation
6. ✅ File Handling
7. ✅ Sessions & Cookies
8. ✅ Error Handling & Exceptions
9. ✅ Security
10. ✅ Performance & Best Practices

Each section includes comprehensive examples, real-world scenarios, and best practices for 7+ years experienced PHP developers.

